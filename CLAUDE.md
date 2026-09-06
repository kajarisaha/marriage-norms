# CLAUDE.MD -- Dissertation Chapter Development with Claude Code

**Project:** Marriage, Parenthood, and Gender Role Attitudes in Egypt
**Institution:** UC Santa Barbara, Department of Economics (5th-year PhD)
**Branch:** main

---

## Research Overview

**Central question:** Does the transition to marriage and parenthood shift the gender-role
attitudes of men and women in Egypt?

**Data:** Egypt Labor Market Panel Survey (ELMPS), rounds 1998, 2006, 2012, 2018, 2023.

**Primary empirical strategy:** two-way fixed effects (individual + year FE), plus a hybrid
pseudo-panel + panel event-study design following Kleven's "The Child Penalty Atlas"
methodology.

**Mechanisms:** gender identity (Akerlof & Kranton) and cognitive dissonance. Evidence that
marriage activates a more traditional female identity comes from labor force participation
(all 5 rounds) and domestic work hours (2012, 2018 only).

### Data availability by outcome and round — check this before writing any code or claim

| Outcome | 1998 | 2006 | 2012 | 2018 | 2023 |
|---|:-:|:-:|:-:|:-:|:-:|
| Labor force participation | ✓ | ✓ | ✓ | ✓ | ✓ |
| Domestic work hours | | | ✓ | ✓ | |
| Gender role attitudes — women | ✓ | ✓ | ✓ | ✓ | ✓ |
| Gender role attitudes — men | | | | ✓ | ✓ |

**Never write a regression, table, or claim touching one of these outcomes without checking
this table for the rounds it actually covers.** This is the single most common way a spec goes
wrong in this project, and it is a data-availability fact, not a modeling choice.

### Fixed effects, clustering, and other locked-in conventions

- **FE structure:** individual FE + year FE + governorate×year FE.
- **Clustering:** always at the individual level, regardless of specification.
  `reghdfe y x, absorb(individual_id year governorate#year) vce(cluster individual_id)`
- **coefplot gotchas** (recurring source of malformed figures — see
  [`stata-code-conventions.md`](.claude/rules/stata-code-conventions.md) for the full writeup):
  - never `noalphabetical`
  - degree symbol is `{char 176}`, never `{&deg}`
  - `ciopts(lcolor(...))` colors are assigned **per model**, not per coefficient
  - `levels(95 90)` requires **two** colors in `ciopts lcolor()`, one per level

---

## Scope Discipline

**Do exactly what was asked — nothing adjacent.** Do not add README files, build scripts,
`.gitignore` edits, helper utilities, or extra tooling that was not requested. If an addition
looks valuable, **list it as a suggestion at the end** and let the user decide.

**Before adding anything not named in the request, ask.** One line is cheaper than a revert.

---

## Core Principles

- **Plan first** -- enter plan mode before non-trivial tasks; save plans to `quality_reports/plans/`
- **Verify after** -- compile/render and confirm output at the end of every task
- **Single source of truth** -- Beamer `.tex` is authoritative for slide content, the `.do` file
  is authoritative for every number, the paper `.tex` is authoritative for prose (see
  [`single-source-of-truth.md`](.claude/rules/single-source-of-truth.md))
- **Quality gates** -- nothing ships below 80/100
- **[LEARN] tags** -- when corrected, save `[LEARN:category] wrong → right` to [MEMORY.md](MEMORY.md)

Cross-session context lives in [MEMORY.md](MEMORY.md); past plans, specs, and session logs are in [quality_reports/](quality_reports/).

**How we verify** — the references and rules that carry the verification discipline:

- [`verification-ladder.md`](.claude/references/verification-ladder.md) — the seven rungs, from *qualify the checker* to the external oracle, and how the review loop converges.
- [`external-oracle-process.md`](.claude/references/external-oracle-process.md) — running an independent frontier-model referee (Claude Code → GPT-5.6 Sol Pro) and adjudicating what it returns.
- [`provenance-and-ground-truth.md`](.claude/references/provenance-and-ground-truth.md) — naming and pinning your oracles, classifying divergence, and the clean-room boundary.
- [`review-fencing.md`](.claude/rules/review-fencing.md) — reviewer independence is a property of the environment, not an instruction: a neutral copy outside the checkout, prior verdicts withheld, and the answer keys the repo already commits fenced off.
- [`release-engineering.md`](.claude/references/release-engineering.md) — shipping research software: message and silent-resolution censuses, frozen feature matrices for ports, hash-claimed inherited tests, and downstream consumers pinned by commit SHA.

**How we write** — [`writing-with-ai.md`](.claude/rules/writing-with-ai.md): internal vs external-facing documents, why a model cannot make its own output stop reading as model output, and the human-readable standard for anything with your name on it.

**Confidential data** — [`confidential-data.md`](.claude/rules/confidential-data.md): raw ELMPS
microdata never gets committed (`data/raw/`, `*.dta` are gitignored); disclosure-avoidance
review applies to any table/figure built on it. **TODO (you):** fill in the actual ELMPS/ERF
data-use-agreement thresholds (cell-suppression minimum, any IRB protocol number) in that file —
Claude will not fabricate them.

**The laws** — [`research-agent-laws.md`](.claude/references/research-agent-laws.md): 21 laws for running agents on research infrastructure, each paid for by a real incident.

**How we remember** — the record lives in the repo, not the transcript:

- [`progress-reports.md`](.claude/rules/progress-reports.md) — GitHub issues as defect memory, `quality_reports/` as work memory, `MEMORY.md` as lesson memory.
- [`issue-ledger.md`](.claude/rules/issue-ledger.md) — the evidence standard an issue must meet, and the seven-section closure comment.
- [`repo-hygiene.md`](.claude/rules/repo-hygiene.md) — **scratch must not become main.** Enforced by `check-repo-hygiene.py` on every commit.

Nothing clears work until it has a row in [`quality_reports/qualification/LEDGER.md`](quality_reports/qualification/LEDGER.md) — run [`/vaccinate`](.claude/skills/vaccinate/SKILL.md) to put one there.

---

## Folder Structure

```
marriage-norms/
├── CLAUDE.md
├── .claude/                     # Rules, skills, agents, hooks
├── Bibliography_base.bib        # Centralized bibliography
├── preambles/header.tex         # Shared LaTeX/Beamer preamble
├── data/                        # ELMPS rounds (raw/ is gitignored — never commit microdata)
├── scripts/
│   └── stata/                   # Numbered .do pipeline (00_install ... 99_run_all)
├── output/
│   ├── tables/                  # esttab .tex fragments, \input{} into slides/paper
│   └── figures/                 # graph export (PDF + PNG)
├── slides/                      # Beamer .tex research-presentation decks (from Overleaf)
├── paper/                       # Dissertation chapter / paper draft .tex
├── explorations/                # Research sandbox (see rules)
├── quality_reports/             # Plans, session logs, merge reports, decision records
├── templates/                   # Session log, quality report templates
└── scripts/R/                   # Dormant — not used in this project (Stata-only pipeline)
```

---

## Commands

```bash
# LaTeX (3-pass, XeLaTeX only) — for slides/ (Beamer) or paper/ (article class)
cd slides   # or: cd paper
TEXINPUTS=../preambles:$TEXINPUTS xelatex -interaction=nonstopmode file.tex
BIBINPUTS=..:$BIBINPUTS bibtex file
TEXINPUTS=../preambles:$TEXINPUTS xelatex -interaction=nonstopmode file.tex
TEXINPUTS=../preambles:$TEXINPUTS xelatex -interaction=nonstopmode file.tex

# Stata replication pipeline (one-command reproduction)
do scripts/stata/99_run_all.do
# Or via /stata-replication, which scaffolds + executes through the stata-mcp MCP server

# Quality score
python scripts/quality_score.py slides/file.tex

# Backtest: is the repo internally consistent and currently true?
# (surface-sync + skill-integrity + model-versions + links + spec-conformance + staleness + repo-hygiene + derived-counts + ledger-coverage + hook-battery)
# Run this after ANY change. Also runs in pre-commit and CI.
./scripts/backtest.sh
```

---

## Quality Thresholds (advisory)

| Score | Checkpoint | Meaning |
|-------|------|---------|
| 80 | Commit | Good enough to save |
| 90 | PR | Ready for deployment |
| 95 | Excellence | Aspirational |

Enforced by `/commit` (halts + asks for override) **and** — once you run `./scripts/install-hooks.sh` — by a real git pre-commit hook (`.githooks/pre-commit`) that runs the full backtest gate suite plus the quality (≥80) gate on every commit. Bypass sparingly with `SKIP_QUALITY_GATE=1` or `--no-verify`.

---

## Skills Quick Reference

The full table of all skills lives in [README.md](README.md#skills-claudeskills). Most-used, by workflow:

- **Slides (research presentations):** `/compile-latex` `/visual-audit` `/slide-excellence` `/proofread`
- **Papers / dissertation chapter:** `/review-paper` (`--peer`) `/seven-pass-review` `/respond-to-referees` `/verify-claims` `/proofread` `/humanize` `/submission-disclosures`
- **Data / reproducibility (Stata):** `/stata-replication` `/audit-reproducibility` `/diagnose` `/replication-package` `/capture-environment` `/power-analysis` `/disclosure-check`
- **Research / writing:** `/interview-me` `/lit-review` `/research-ideation` `/preregister` `/coauthor-brief`
- **Verification / rigor:** `/vaccinate` `/challenge` `/oracle-review` `/adjudicate-review` `/differential-audit` `/blast-radius` `/verify-artifact` `/credible-claims` `/deep-audit`
- **Meta / workflow:** `/commit` `/learn` `/new-skill` `/checkpoint` `/context-status` `/triage-inbox`

`/data-analysis`, `/review-r`, `/r-package-check`, `/simulation-study` are R-pipeline skills —
**dormant** in this project (no R pipeline; kept on disk in case a future coauthor needs them).
TikZ tooling (`/extract-tikz`, `/new-diagram`) is likewise dormant unless you draw diagrams
(e.g. a causal DAG) directly inside a Beamer slide.

---

## Beamer Palette (`preambles/header.tex`)

| Color | Hex | Use |
| --- | --- | --- |
| `primary-blue` | `#012169` | headings, accents |
| `primary-gold` | `#B9975B` | emphasis, borders |
| `highlight-yellow` | `#F2A900` | markers, alerts |
| `positive` | `#15803D` | good / observed / identified |
| `negative` | `#B91C1C` | bad / problematic / bias |
| `neutral` | `#525252` | reference / context |

No `\newtcolorbox` environments are defined yet in `preambles/header.tex` — add one here when
you create it (e.g. a results-highlight box), rather than inventing table rows for boxes that
don't exist.

---

## Current Project State

| Artifact | File(s) | Status |
| --- | --- | --- |
| HelloWorld *(sample — delete when ready)* | `slides/HelloWorld.tex` | Minimal deck to verify LaTeX setup |
| Dissertation chapter draft | `paper/` *(empty — first draft not yet started)* | — |
| Research presentation slides | `slides/` *(empty — first deck not yet started)* | — |
| Stata pipeline | `scripts/stata/` *(empty — scaffold via `/stata-replication`)* | — |
