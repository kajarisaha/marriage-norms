---
paths:
  - "slides/**/*.tex"
  - "paper/**/*.tex"
  - "scripts/stata/**"
  - "output/**/*"
---

# Single Source of Truth: Enforcement Protocol

There are three authoritative sources in this project, each for a different kind of content.
Nothing else is authoritative — everything else is derived and must never be hand-edited
independently of its source.

```
scripts/stata/*.do      (SOURCE OF TRUTH for every number)
  └── output/tables/*.tex, output/figures/*      (derived — esttab / graph export)
        └── \input{} into slides/**/*.tex and paper/**/*.tex

slides/**/*.tex          (SOURCE OF TRUTH for slide content/wording)
paper/**/*.tex           (SOURCE OF TRUTH for paper prose)
Bibliography_base.bib    (shared across slides/ and paper/)
```

**Never hand-type a coefficient, standard error, N, or p-value into a slide or the paper.**
If a number appears in `slides/` or `paper/`, it must arrive via `\input{output/tables/...}`
(esttab output) or reference a figure produced by `graph export` — never be typed by hand from
looking at Stata's console output. This is the practical form of "verify after": a hand-typed
number is a number nobody re-checked against the `.do` file's actual output.

## Numbers-from-do-files checklist

```
[ ] Every number in slides/paper/ traces to a specific scripts/stata/NN_*.do line
[ ] Every table in output/tables/ was produced by esttab, not typed
[ ] Every figure in output/figures/ was produced by graph export, not typed/drawn
[ ] No two artifacts (slide + paper, or two slides) show the same number differently
    (see .claude/rules/replication-protocol.md's "horizontal check" if a passport exists)
```

## Slide-content changes

Beamer `.tex` in `slides/` is authoritative for what a given deck says. There is no derived
HTML mirror to keep in sync (no Quarto) — a slide edit is done once it compiles.

## Paper-content changes

Paper `.tex` in `paper/` is authoritative for prose, structure, and argument. Slides are not
required to match the paper's exact wording — a talk is not obligated to say things the same way
a chapter does — but a **number** shown in both places must be the same number, sourced the
same way (see the checklist above).

## Bibliography

`Bibliography_base.bib` is the canonical bibliography for both `slides/` and `paper/`. No
per-deck or per-draft `.bib` files. All citations resolve against this one file.
