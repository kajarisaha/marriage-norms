---
paths:
  - "slides/**/*.tex"
  - "paper/**/*.tex"
  - "scripts/**/*.R"
---

# Quality Review & Scoring Rubrics

> **Framing:** Thresholds are **advisory at the harness level**. The `/commit` skill runs `quality_score.py` and halts on failure until the user fixes or explicitly overrides. A **real git pre-commit hook** (`.githooks/pre-commit`, activated once via `./scripts/install-hooks.sh`) extends the same gates to *every* commit, so bypassing the skill no longer bypasses the review — unless you opt out with `SKIP_QUALITY_GATE=1` / `--no-verify`. "Gate" here means "checkpoint enforced by a skill or the pre-commit hook," not an unconditional harness-level block.

## Thresholds

- **80/100 = Commit** -- good enough to save
- **90/100 = PR** -- ready for deployment
- **95/100 = Excellence** -- aspirational

## R Scripts (.R) — dormant in this project (no R pipeline)

| Severity | Issue | Deduction |
|----------|-------|-----------|
| Critical | Syntax errors | -100 |
| Critical | Domain-specific bugs | -30 |
| Critical | Hardcoded absolute paths | -20 |
| Major | Missing set.seed() | -10 |
| Major | Missing figure generation | -5 |

## Beamer Slides (.tex)

| Severity | Issue | Deduction |
|----------|-------|-----------|
| Critical | XeLaTeX compilation failure | -100 |
| Critical | Undefined citation | -15 |
| Critical | Overfull hbox > 10pt | -10 |

## Enforcement (the /commit skill + an optional pre-commit hook)

- **Score < 80:** Halt within `/commit`. List blocking issues. User may override with an explicit natural-language signal ("commit anyway" / "skip quality gate") and a reason — the override is logged in the commit body.
- **Score < 90:** Allow commit within `/commit`, warn. List recommendations.
- **Direct `git commit`:** unenforced *until* you run `./scripts/install-hooks.sh`, which points `core.hooksPath` at the version-controlled `.githooks/pre-commit`. After that, every commit (skill or not) runs the full backtest gate suite plus the quality (≥80) gate. Bypass sparingly with `SKIP_QUALITY_GATE=1` (quality only) or `git commit --no-verify` (all hooks); record the reason in the commit body.

## Quality Reports

Generated **only at merge time**. Use `templates/quality-report.md` for format.
Save to `quality_reports/merges/YYYY-MM-DD_[branch-name].md`.

## Tolerance Thresholds (Research)

Mirrors `replication-protocol.md`'s passport tolerances — used by `/audit-reproducibility` when
checking a numeric claim in `paper/` or `slides/` against `output/tables/`:

| Quantity | Tolerance | Rationale |
|----------|-----------|-----------|
| Integers (N, counts) | Exact match | No reason for any difference |
| Point estimates | < 0.01 | Rounding in display |
| Standard errors | < 0.05 | Clustering/df-adjustment variation |
| P-values | Same significance level | Exact p may differ slightly |
| Percentages | < 0.1pp | Display rounding |
