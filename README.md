# Awesome Model Growth: Methods, Theories & Applications

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](./CONTRIBUTING.md)

A curated list of papers on **model growth** (also called *model expansion*, *upscaling*, *warm-starting* or *progressive training*): reusing a trained smaller network to initialize and speed up training of a larger one. It also covers the **initialization** methods these papers build on.

---

> [!IMPORTANT]
> Contributions welcome:
>
> Open a pull request (see [CONTRIBUTING.md](./CONTRIBUTING.md)) to add a missing paper, fix a TL;DR, move a paper to a better category, or update the venue once a paper is accepted. Thank you!

---

## 💥 News 💥

- **2026-10**: Repository created.
- 🚀 Papers with experiments at **≥ 1B parameters** (for the grown model) are marked with 🚀.

---

## What is model growth?

Training large models from scratch is expensive, yet we usually already have smaller trained models of the same family. Model growth asks:

- **How to grow?** Which operator maps small weights to large weights: copying, stacking, interpolation, learned maps or zero-padding? Should the grown model compute the same function as the small one?
- **When to grow?** At which point in the small model's training, and by how much (the growth factor)?
- **How to train after growing?** Learning rate, schedule, optimizer state and hyperparameter transfer.
- **Why does it work or fail?** Symmetry, subspace traps, training dynamics and scaling laws.

**Growth dimensions** used in the tables: **W** = width (hidden size, heads, FFN), **D** = depth (layers), **W+D** = both.

---

## Contents

- [Growth Methods](#growth-methods)
  - [Function-Preserving Expansion](#function-preserving-expansion)
  - [Depth Growth and Stacking](#depth-growth-and-stacking)
  - [Learned Growth Operators](#learned-growth-operators)
  - [Progressive and Multi-Stage Growth](#progressive-and-multi-stage-growth)
  - [Neuron Splitting and Adaptive Growth](#neuron-splitting-and-adaptive-growth)
  - [Knowledge-Transfer-Based Growth](#knowledge-transfer-based-growth)
  - [Non-Function-Preserving Warm Starts](#non-function-preserving-warm-starts)
- [Training Dynamics and Hyperparameters after Growth](#training-dynamics-and-hyperparameters-after-growth)
- [Theory and Analysis](#theory-and-analysis)
- [Benchmarks and Empirical Studies](#benchmarks-and-empirical-studies)
- [Applications](#applications)
- [Foundations: Neural Network Initialization](#foundations-neural-network-initialization)
- [Contributing](#contributing)
- [Citation](#citation)

---

## Growth Methods

### Function-Preserving Expansion

The grown model computes exactly the same function as the small model at initialization.

| Paper | Venue | Year | Growth | Architecture | TL;DR | Code |
| ----- | :---: | :--: | :----: | :----------: | ----- | :--: |
| [Lossless Model Expansion (LEMON)](https://arxiv.org/abs/2310.07999) | ICLR | 2024 | W+D | BERT, ViT | Lossless expansion that adds a parameter to bert2BERT-style neuron splitting to break weight symmetry, plus an LR scheduler tailored to expanded models. | |
| [Scaling Smart: Accelerating Large Language Model Pre-Training with Small Model Initialization (HyperCloning)](https://arxiv.org/abs/2409.12903) | arXiv | 2024 | W | Decoder LLMs 🚀 | Clones a small LM's weights into a wider LM so the large model starts with the small model's accuracy, then trains it further. | |
| [Staged Training for Transformer Language Models (StagedGrow)](https://arxiv.org/abs/2203.06211) | ICML | 2022 | W+D | GPT-2 | Growth operators that preserve both loss and training dynamics; width doubling concatenates two copies of each layer and halves the output. | |
| [bert2BERT: Towards Reusable Pretrained Language Models](https://arxiv.org/abs/2110.07143) | ACL | 2022 | W+D | BERT, GPT | Function-preserving initialization (FPI) for width, stacking for depth, plus Advanced Knowledge Initialization (AKI) that also borrows weights from the next layer. | |
| [Network Morphism](https://arxiv.org/abs/1603.01670) | ICML | 2016 | W+D | CNN | Morphs a parent network into a child network (wider, deeper, new kernels) that preserves its function, including through non-linearities. | |
| [Net2Net: Accelerating Learning via Knowledge Transfer](https://arxiv.org/abs/1511.05641) | ICLR | 2016 | W+D | CNN | The foundational paper. Net2WiderNet copies random neurons and divides their outgoing weights; Net2DeeperNet inserts identity-initialized layers. | |

### Depth Growth and Stacking

Grow the number of layers, typically by duplicating or stacking trained blocks.

| Paper | Venue | Year | Growth | Architecture | TL;DR | Code |
| ----- | :---: | :--: | :----: | :----------: | ----- | :--: |
| [On the Inductive Bias of Stacking Towards Improving Reasoning (MIDAS)](https://arxiv.org/abs/2409.19044) | NeurIPS | 2024 | D | Decoder LLMs 🚀 | A gradual-stacking variant that speeds up training and improves reasoning beyond what equal perplexity would predict. | |
| [Stacking Your Transformers: A Closer Look at Model Growth for Efficient LLM Pre-Training (G<sub>stack</sub>)](https://arxiv.org/abs/2405.15319) | NeurIPS | 2024 | D | LLaMA-style 🚀 | Compares growth operators (direct, learned, random, zero) for LLMs; depth-wise stacking (G<sub>stack</sub>) wins and scales to 7B with ~50% speedup. | |
| [LLaMA Pro: Progressive LLaMA with Block Expansion](https://arxiv.org/abs/2401.02415) | ACL | 2024 | D | LLaMA-2 🚀 | Inserts copied blocks with zero-initialized output projections (identity at init) and trains only the new blocks for domain adaptation without forgetting. | [GitHub](https://github.com/TencentARC/LLaMA-Pro) |
| [SOLAR 10.7B: Scaling Large Language Models with Simple yet Effective Depth Up-Scaling](https://arxiv.org/abs/2312.15166) | NAACL (Industry) | 2024 | D | Mistral 🚀 | Depth up-scaling: duplicate a 7B model, drop middle layers from each copy, concatenate and continue pre-training. | |
| [Progressively Stacking 2.0: A Multi-stage Layerwise Training Method for BERT Training Speedup](https://arxiv.org/abs/2011.13635) | arXiv | 2020 | D | BERT | Stacks new layers and trains only the newly added top layers at each stage, reducing per-step cost further. | |
| [Efficient Training of BERT by Progressively Stacking](https://proceedings.mlr.press/v97/gong19a.html) | ICML | 2019 | D | BERT | Observes similar attention patterns across layers and doubles depth by copying trained layers on top; no width growth. | |

### Learned Growth Operators

Learn the mapping from small weights to large weights.

| Paper | Venue | Year | Growth | Architecture | TL;DR | Code |
| ----- | :---: | :--: | :----: | :----------: | ----- | :--: |
| [Reusing Pretrained Models by Multi-linear Operators for Efficient Training (Mango)](https://arxiv.org/abs/2310.10699) | NeurIPS | 2023 | W+D | BERT, GPT, ViT | Replaces LiGO's separate width/depth maps with a multi-linear (tensor-ring) operator that captures correlations among all weights. | |
| [Learning to Grow Pretrained Models for Efficient Transformer Training (LiGO)](https://arxiv.org/abs/2303.00980) | ICLR | 2023 | W+D | BERT, RoBERTa, GPT-2, ViT | Learns a linear growth operator M = LR (L for depth, R for width) in a few steps; generalizes Net2Net and stacking. | [GitHub](https://github.com/VITA-Group/LiGO) |

### Progressive and Multi-Stage Growth

Grow repeatedly during training, often along several dimensions on a schedule.

| Paper | Venue | Year | Growth | Architecture | TL;DR | Code |
| ----- | :---: | :--: | :----: | :----------: | ----- | :--: |
| [Masked Structural Growth for 2x Faster Language Model Pre-training (MSG)](https://arxiv.org/abs/2305.02869) | ICLR | 2024 | W+D | BERT, GPT | Masks newly added structures and gradually unmasks them, making growth strictly function-preserving regardless of how new weights are initialized. | |
| [Efficient Stagewise Pretraining via Progressive Subnetworks (RaPTr)](https://arxiv.org/abs/2402.05913) | arXiv | 2024 | D | BERT, UL2 | Keeps the full model but trains random subnetworks of growing depth, as an alternative to stacking. | |
| [FLM-101B: An Open LLM and How to Train It with $100K Budget](https://arxiv.org/abs/2309.03852) | arXiv | 2023 | W+D | Decoder LLM 🚀 | Grows a model 16B → 51B → 101B during pre-training to cut cost. | |
| [Automated Progressive Learning for Efficient Training of Vision Transformers (AutoProg)](https://arxiv.org/abs/2203.14509) | CVPR | 2022 | D | ViT | Automatically decides when and how much to grow, using momentum growth (MoGrow) to initialize new layers. | |
| [On the Transformer Growth for Progressive BERT Training (CompoundGrow)](https://arxiv.org/abs/2010.12562) | NAACL | 2021 | W+D | BERT | Grows several dimensions together (depth, width and input length) rather than one at a time. | |

### Neuron Splitting and Adaptive Growth

Decide *where* and *when* to add neurons or layers, often from gradient information.

| Paper | Venue | Year | Growth | Architecture | TL;DR | Code |
| ----- | :---: | :--: | :----: | :----------: | ----- | :--: |
| [GradMax: Growing Neural Networks using Gradient Information](https://arxiv.org/abs/2201.05125) | ICLR | 2022 | W | MLP, CNN | Initializes new neurons so the function is unchanged but the gradient norm is maximized. | |
| [Firefly Neural Architecture Descent: a General Approach for Growing Neural Networks](https://arxiv.org/abs/2102.08574) | NeurIPS | 2020 | W+D | CNN | Grows wider and deeper by picking, from a large candidate set, the new neurons that most reduce the loss. | |
| [Splitting Steepest Descent for Growing Neural Architectures](https://arxiv.org/abs/1910.03103) | NeurIPS | 2019 | W | CNN | Splits neurons when the loss is at a saddle in the splitting direction, chosen via a "splitting matrix". | |

### Knowledge-Transfer-Based Growth

Use the small model as a teacher or guide instead of (or as well as) copying its weights.

| Paper | Venue | Year | Growth | Architecture | TL;DR | Code |
| ----- | :---: | :--: | :----: | :----------: | ----- | :--: |
| [Late-to-Early Training: LET LLMs Learn Earlier, So Faster and Better](https://arxiv.org/abs/2602.05393) | ICLR | 2026 | – | Decoder LLMs 🚀 | Aligns the early layers of a large LLM with the late-layer representations of a 10x smaller pretrained model early in training; up to 1.6x speedup at 1.4B. | |
| [Knowledge Inheritance for Pre-trained Language Models (KI)](https://arxiv.org/abs/2105.13880) | NAACL | 2022 | – | BERT, RoBERTa | Uses a smaller trained model as a teacher, distilling into the larger model during pre-training. | |

### Non-Function-Preserving Warm Starts

Initialize from a small model without exactly preserving its function.

| Paper | Venue | Year | Growth | Architecture | TL;DR | Code |
| ----- | :---: | :--: | :----: | :----------: | ----- | :--: |
| [When is Warmstarting Effective for Scaling Language Models?](https://arxiv.org/abs/2605.13405) | NeurIPS | 2026 | W | Decoder LLMs | Shrink-Zero-Perturb (SZP) as a simple baseline; function preservation is not needed; growth factor 2 is most reliable; gains are largest below ~20 tokens/parameter. | |
| [Warmstarting for Scaling Language Models](https://arxiv.org/abs/2411.07340) | arXiv | 2024 | W | Decoder LLMs | Shrinks small-model weights, zero-pads, and perturbs with μP-scaled noise so warm-starting works with μTransfer. | |

---

## Training Dynamics and Hyperparameters after Growth

How to set the learning rate, schedule and optimizer state for a grown model.

| Paper | Venue | Year | Growth | Architecture | TL;DR | Code |
| ----- | :---: | :--: | :----: | :----------: | ----- | :--: |
| [μpscaling Small Models: Principled Warm Starts and Hyperparameter Transfer](https://arxiv.org/abs/2602.10545) | ICML | 2026 | W | MLP, ResNet, GPT-2 | μP-based width upscaling with "dynamic equivalence" (same function throughout training); small symmetry-breaking noise lets the model use its new capacity, and μTransfer carries hyperparameters to the upscaled model. | |
| [Understanding Warmup-Stable-Decay Learning Rates: A River Valley Loss Landscape Perspective](https://arxiv.org/abs/2410.05192) | ICLR | 2025 | – | Decoder LLMs | Explains why WSD schedules work (a "river valley" loss landscape); relevant to scheduling LR when continuing training after growth. | |
| [Accelerated Training via Incrementally Growing Neural Networks using Variance Transfer and Learning Rate Adaptation](https://arxiv.org/abs/2306.12700) | NeurIPS | 2023 | W | CNN, ViT | Transfers variance to new neurons and adapts the learning rate per subnetwork so the growing network trains stably. | |

---

## Theory and Analysis

| Paper | Venue | Year | Growth | Architecture | TL;DR | Code |
| ----- | :---: | :--: | :----: | :----------: | ----- | :--: |
| [How Model Growth, Recursion, and Boundary Operators Influence Scaling Exponents](https://arxiv.org/abs/2609.19107) | arXiv | 2026 | D | Looped transformers 🚀 | Model growth (with or without weight sharing) changes the scaling *exponent*, not just the offset; a grown 7.4B model matches GPT-3 13B with ~20x less compute. | |
| [Escaping the Subspace Trap: The Role of Optimizer Geometry in Model Width Expansion](https://proceedings.mlr.press/v306/chen26eq.html) | ICML | 2026 | W | Decoder LLMs | After function-preserving width expansion, updates stay in a low-dimensional "subspace trap"; element-wise adaptive optimizers stay trapped while isotropic ones escape. | [GitHub](https://github.com/A-PolarBear/Model-Width-Expansion) |
| [When and How to Grow? On Efficient Pre-training via Model Growth](https://proceedings.mlr.press/v260/wang25a.html) | ACML | 2024 | W+D | Transformer LMs | Estimates when to grow theoretically (even after convergence works), with progressive width growth and function-preserving depth growth from adjacent layers; ~1.5x speedup. | |

---

## Benchmarks and Empirical Studies

| Paper | Venue | Year | Growth | Architecture | TL;DR | Code |
| ----- | :---: | :--: | :----: | :----------: | ----- | :--: |
| [No Train No Gain: Revisiting Efficient Training Algorithms For Transformer-based Language Models](https://arxiv.org/abs/2307.06440) | NeurIPS | 2023 | D | BERT, T5 | Under a fixed compute budget measured fairly, gains from layer stacking and other efficient-training methods mostly vanish. | |

See also [Stacking Your Transformers](#depth-growth-and-stacking) and [When is Warmstarting Effective?](#non-function-preserving-warm-starts), which are large empirical comparisons.

---

## Applications

| Paper | Venue | Year | Growth | Architecture | TL;DR | Code |
| ----- | :---: | :--: | :----: | :----------: | ----- | :--: |
| [Small Pre-trained Language Models Can be Fine-tuned as Large Models via Over-Parameterization](https://aclanthology.org/2023.acl-long.212) | ACL | 2023 | W | BERT, GPT-2 | Over-parameterizes a small model only during fine-tuning (matrix product operators), then merges back so inference cost is unchanged. | [GitHub](https://github.com/zfgao66/OPF) |
| [Sparse Upcycling: Training Mixture-of-Experts from Dense Checkpoints](https://arxiv.org/abs/2212.05055) | ICLR | 2023 | W (experts) | T5, ViT 🚀 | Grows a dense checkpoint into a mixture of experts by copying FFNs into experts. | |

---

## Foundations: Neural Network Initialization

Classic and modern initialization schemes that growth methods build on.

| Paper | Venue | Year | TL;DR |
| ----- | :---: | :--: | ----- |
| [Initializing Models with Larger Ones (Weight Selection)](https://arxiv.org/abs/2311.18823) | ICLR | 2024 | The reverse of growth: initializes a small model by selecting a subset of a larger pretrained model's weights. |
| [Tensor Programs VI: Feature Learning in Infinite-Depth Neural Networks (Depth-μP)](https://arxiv.org/abs/2310.02244) | ICLR | 2024 | Extends μP to depth so hyperparameters transfer across the number of layers. |
| [Mimetic Initialization of Self-Attention Layers](https://arxiv.org/abs/2305.09828) | ICML | 2023 | Initializes attention weights to mimic patterns seen in pretrained transformers. |
| [DeepNet: Scaling Transformers to 1,000 Layers](https://arxiv.org/abs/2203.00555) | arXiv | 2022 | DeepNorm residual scaling with matching initialization for very deep transformers. |
| [Tensor Programs V: Tuning Large Neural Networks via Zero-Shot Hyperparameter Transfer (μP)](https://arxiv.org/abs/2203.03466) | NeurIPS | 2021 | μP parametrization lets hyperparameters tuned on a small model transfer to wider models. |
| [GradInit: Learning to Initialize Neural Networks for Stable and Efficient Training](https://arxiv.org/abs/2102.08098) | NeurIPS | 2021 | Learns per-layer init scales so the first optimizer step reduces the loss most. |
| [Tensor Programs IV: Feature Learning in Infinite-Width Neural Networks](https://arxiv.org/abs/2011.14522) | ICML | 2021 | Introduces the maximal update parametrization (μP), which keeps feature learning at infinite width. |
| [ReZero is All You Need: Fast Convergence at Large Depth](https://arxiv.org/abs/2003.04887) | UAI | 2021 | Zero-initialized residual gates make every block start as the identity. |
| [Improving Transformer Optimization Through Better Initialization (T-Fixup)](https://proceedings.mlr.press/v119/huang20f.html) | ICML | 2020 | Initialization that lets transformers train without warmup or layer norm. |
| [Fixup Initialization: Residual Learning Without Normalization](https://arxiv.org/abs/1901.09321) | ICLR | 2019 | Rescales residual branches at init so deep ResNets train without normalization. |
| [All You Need is a Good Init (LSUV)](https://arxiv.org/abs/1511.06422) | ICLR | 2016 | Orthogonal init followed by layer-wise rescaling to unit output variance on data. |
| [Data-dependent Initializations of Convolutional Neural Networks](https://arxiv.org/abs/1511.06856) | ICLR | 2016 | Sets weights from data statistics so all layers train at similar rates. |
| [Delving Deep into Rectifiers (He / Kaiming init)](https://arxiv.org/abs/1502.01852) | ICCV | 2015 | Variance-preserving initialization for ReLU networks. |
| [Exact Solutions to the Nonlinear Dynamics of Learning in Deep Linear Neural Networks (Orthogonal init)](https://arxiv.org/abs/1312.6120) | ICLR | 2014 | Analyzes deep linear network dynamics and motivates orthogonal initialization. |
| [Understanding the Difficulty of Training Deep Feedforward Neural Networks (Xavier / Glorot init)](https://proceedings.mlr.press/v9/glorot10a.html) | AISTATS | 2010 | Variance-preserving initialization for tanh/sigmoid networks. |

---

## Contributing

Missing a paper, or found a mistake? See [CONTRIBUTING.md](./CONTRIBUTING.md). Pull requests and issues are welcome.

## Citation

If you find this list useful, please consider citing it:

```bibtex
@misc{awesome-model-growth,
  author       = {Maurya, Akansh},
  title        = {Awesome Model Growth: Methods, Theories \& Applications},
  year         = {2026},
  howpublished = {\url{https://github.com/akansh12/Awesome-Model-Growth-Methods-Theories-Applications}}
}
```

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=akansh12/Awesome-Model-Growth-Methods-Theories-Applications&type=Date)](https://star-history.com/#akansh12/Awesome-Model-Growth-Methods-Theories-Applications&Date)
