# Contributing

Thanks for helping keep this list useful! Everything happens through pull requests (PRs) and issues.

## What belongs here

A paper fits if its main contribution is one of the following:

- **Growing a model**: initializing a larger network from a trained smaller one (width, depth, experts, sequence length, ...).
- **Training after growth**: learning rate, optimizer state, schedules or hyperparameter transfer for grown models.
- **Theory or analysis** of growth: symmetry, subspace traps, scaling laws, when and how much to grow.
- **Initialization** methods that growth methods build on (these go in the *Foundations* section at the end).

Pruning, distillation into *smaller* models and model merging are out of scope, unless the paper uses them to grow a model.

## How to add a paper

1. Choose the most fitting section in [README.md](./README.md). If none fits, put it in the closest one and say so in the PR.
2. Add one row to that section's table, **sorted newest first** (by year, then roughly by date).
3. Use this format:

```markdown
| [Paper Title (Method Name)](https://arxiv.org/abs/XXXX.XXXXX) | Venue | Year | W / D / W+D | Architecture 🚀 | One-sentence TL;DR. | [GitHub](https://github.com/org/repo) |
```

### Column guide

| Column | What to write |
| ------ | ------------- |
| **Paper** | The full title, linked to arXiv (abstract page, not the PDF) or the proceedings page. Put the method's short name in parentheses if it has one. |
| **Venue** | Conference or journal (e.g. `ICLR`, `NeurIPS`, `ACL`). Use `arXiv` for preprints, and update it once the paper is accepted. |
| **Year** | Year of the venue (or of the arXiv version, for preprints). |
| **Growth** | `W` (width), `D` (depth), `W+D` (both) or `–` if the paper doesn't grow weights directly (e.g. distillation-based guidance). |
| **Architecture** | The model families tested (e.g. `BERT`, `GPT-2`, `LLaMA-style`, `ViT`, `CNN`). Add 🚀 if the grown model is **≥ 1B parameters**. |
| **TL;DR** | One sentence (two at most) on *what the method does* or *what the paper finds*. Avoid marketing language such as "state of the art". |
| **Code** | `[GitHub](link)` to the official code, or leave empty. |

The *Foundations* table uses only `Paper | Venue | Year | TL;DR`.

## Other contributions

- **Fix a TL;DR or venue**: just edit the row and explain the fix in the PR.
- **Suggest a new section**: open an issue describing it and listing at least 3 papers that would go there.
- **Report a broken link**: open an issue or a PR with the fixed link.

## Checklist before opening a PR

- [ ] The paper is not already listed (search the README for its title).
- [ ] The link opens the paper's abstract or proceedings page.
- [ ] The row is in the right section and in newest-first order.
- [ ] The TL;DR is one or two sentences.
