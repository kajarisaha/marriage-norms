---
description: Paper/slides ↔ code cross-artifact review — when /review-paper runs, auto-invoke /audit-reproducibility (and /review-r only if R scripts are referenced) on the pair. Surface cross-artifact findings alongside the paper review.
paths: ["paper/**/*.tex", "slides/**/*.tex", "*.tex"]
---

# Cross-Artifact Review Protocol

A paper is not an island. Its claims depend on the code that produced them. Reviewing the paper without reviewing the code is reviewing half the artifact.

## The Dependency Graph

```
paper.tex        ──cites──> Table 2
Table 2          ──from──> output/tables/tab_main.tex
tab_main.tex     ──by──> scripts/stata/03_analyze.do
03_analyze.do    ──uses──> scripts/stata/_outputs/clean_panel.dta
clean_panel.dta  ──by──> scripts/stata/01_clean.do
01_clean.do      ──reads──> data/raw/elmps_2023.dta
```

A bug in `01_clean.do` invalidates Table 2. Reviewing `paper.tex` without touching the code misses this class of error entirely. **No `/review-r`-equivalent code-quality reviewer exists for Stata yet** (`stata-code-conventions.md` §Enforcement notes this gap) — cross-artifact review for this project runs `/audit-reproducibility` only, unless a referenced script is actually R.

## When to apply

Applies when `/review-paper` runs on a manuscript that references analysis scripts. Detection is **pattern-based** — if the manuscript has none of the signals below, no cross-artifact work happens (and `--no-cross-artifact` is a no-op). To force invocation on a paper without these detection signals, point `/review-paper` at a manuscript that `\input{}`s the script outputs, or invoke `/review-r` and `/audit-reproducibility` directly alongside `/review-paper`.

Detection signals:

- `\input{output/tables/...}` or `\input{scripts/stata/_outputs/...}`
- `%% source: scripts/stata/03_analyze.do` comments
- Numeric claims in text (ATT, coefficients, N, p-values) **combined with** the `scripts/stata/` directory
- Table labels in the paper that match filenames under `output/tables/` or `scripts/stata/_outputs/`

Detection is intentionally conservative — a theory paper with no code should not trigger the protocol, even if it lives in a repo that has scripts for other work.

## The protocol

When `/review-paper` detects any of the above:

### 1. Identify referenced scripts

Scan the manuscript for:

- `\input{path}` commands (tables, figures pulled from files)
- Line comments `%% from: scripts/...`
- Table labels that match filenames in `output/tables/` or `scripts/stata/_outputs/` (e.g., `Table:main_ATT` ↔ `tab_main.tex`)

Build a list of scripts that produced content in this paper.

### 2. Auto-invoke `/review-r` (only if an identified script is R)

For each identified script that is actually R (not the expected case in this project), launch
`/review-r` in a forked subagent (`context: fork`). Save reports to
`quality_reports/cross_artifact_[paper]/review_r_[script].md`. For Stata `.do` scripts, skip
this step — there is no code-quality reviewer for Stata yet — and rely on `/audit-reproducibility`
(step 3) to catch numeric drift.

### 3. Auto-invoke `/audit-reproducibility`

Run `/audit-reproducibility $manuscript scripts/stata/_outputs/` once (it also reads `output/tables/`). Save to `quality_reports/cross_artifact_[paper]/reproducibility.md`.

### 4. Surface cross-artifact findings

In the paper review report, add a new section:

```markdown
## Cross-Artifact Findings

**Scripts reviewed:** N (see `quality_reports/cross_artifact_[paper]/`)
**Reproducibility:** PASS / FAIL — k of m claims within tolerance
**Code quality (merged from /review-r reports):** C critical, M major, L minor

### Critical cross-artifact issues (paper + code together)
| Paper claim | Code location | Issue |
|---|---|---|

### Code-only issues (won't block paper, but file a follow-up)
…

### Paper-only issues (code is clean)
[Rest of the paper review goes here]
```

### 5. Exit behavior

- Any CRITICAL from `/audit-reproducibility` (FAIL on tolerance) → escalate to CRITICAL in paper review.
- Code CRITICAL bugs that affect paper claims → escalate in paper review.
- Code CRITICAL bugs unrelated to paper claims → file as separate action item.

## Opt-out

- `/review-paper --no-cross-artifact` skips the dependency graph. Useful for theory papers, comments, or preprints without code.

## Cross-references

- `.claude/skills/review-paper/SKILL.md` — the orchestrator.
- `.claude/skills/review-r/SKILL.md` — code reviewer.
- `.claude/skills/audit-reproducibility/SKILL.md` — numeric claims verifier.
- `.claude/rules/replication-protocol.md` — tolerance contract.

## What this rule does NOT require

- Running R / Stata / Python (that's `/audit-reproducibility`'s job, and it reads existing outputs).
- Git-blame archaeology — we review current state.
- Judging whether a paper's authors wrote good code vs. whether their *results* are defensible. We care about the latter first.

## `--peer` mode ordering

In `/review-paper --peer [journal]` mode, cross-artifact review runs **before** the editor's desk review (as Phase 0). This gives the editor reproducibility evidence — any FAIL on load-bearing claims is desk-reject-worthy. The editor's desk review will cite specific `/audit-reproducibility` findings in the desk_review.md when relevant.

In default and `--adversarial` modes, cross-artifact still runs at Step 6b (after the paper review). Both orderings are valid; the `--peer` pre-flight ordering exists because editors make desk-reject decisions based on evidence of data errors.

