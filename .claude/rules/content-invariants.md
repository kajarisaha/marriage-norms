---
paths:
  - "slides/**/*.tex"
  - "paper/**/*.tex"
  - "preambles/header.tex"
---

# Content Invariants (INV-1 through INV-4)

Numbered non-negotiable rules for content produced in this repository. Critic agents, reviewers,
and audit agents should cite invariants by number (e.g., "violates INV-2") when flagging issues.
Adapted from clo-author's enforcement pattern.

- **INV-1: Single bibliography.** `Bibliography_base.bib` is the canonical bibliography. No
  per-deck or per-draft `.bib` files. All citations must resolve against this one file.
- **INV-2: Numbers come from `.do` output, not from memory.** Every coefficient, SE, N, or
  p-value shown in `slides/` or `paper/` traces to `output/tables/` (esttab) or
  `output/figures/` (graph export) — see [`single-source-of-truth.md`](single-source-of-truth.md).
  Hand-typed numbers are a critical bug even if currently correct, because nothing re-checks them
  when the `.do` file changes.
- **INV-3: Max 2 colored boxes per slide.** Overusing `keybox`, `definitionbox`, or callout
  environments creates "box fatigue." Two per slide maximum.
- **INV-4: Motivation before formalism.** Every definition or identifying assumption should be
  preceded by intuition or a concrete example — a research talk audience needs the "why" before
  the notation as much as a classroom does.

R script invariants (set.seed discipline, transparent-background ggsave, project theme on plots)
live dormant in [`r-code-conventions.md`](r-code-conventions.md) — this project has no R
pipeline, so they don't apply here, but they're not deleted in case a future coauthor adds one.
