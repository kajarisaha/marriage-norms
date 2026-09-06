---
paths:
  - "**/*.do"
  - "scripts/stata/**"
---

# Stata Code Conventions

**Reproducibility is the default, not a feature.** A `.do` file should be runnable from a clean shell with no manual intervention. Every script ends in the same state regardless of how many times it runs.

> This rule mirrors [`r-code-conventions.md`](r-code-conventions.md) for users whose pipelines are Stata-first. Forkers who use both R and Stata in the same project: both rules apply on their respective files.

## 1. Reproducibility scaffolding

Every `.do` file starts with the same header:

```stata
/*------------------------------------------------------------
File:       NN_descriptive_name.do
Purpose:    [one-sentence description]
Inputs:     [path to inputs]
Outputs:    [path to outputs]
Run order:  Standalone | After NN_prior.do
------------------------------------------------------------*/

version 18                        // pin Stata semantics
clear all
set more off
set seed 12345                    // pin RNG seed for any random ops
set sortseed 12345                // pin sort stability across versions
cap log close
cap log close _all                // belt-and-suspenders
log using "scripts/stata/_outputs/NN_log.smcl", replace
```

Why each line:

- **`version 18`** — explicit semantics. New Stata versions can silently change defaults (e.g., `reghdfe` clustering df-adjustment); pinning is the only defence.
- **`clear all`** — no leftover state from a prior session.
- **`set more off`** — scripts shouldn't pause for keystrokes.
- **`set seed` + `set sortseed`** — every random op + every `sort` is deterministic.
- **`cap log close _all`** — pre-emptively close any logs the previous session left open.
- **`log using ... , replace`** — capture stdout; `replace` ensures re-runs don't append.

End every `.do` file with:

```stata
log close
exit, clear STATA      // explicit clean exit; useful in batch mode
```

## 2. Numbered pipeline

Scripts live in `scripts/stata/` and are numbered for run order:

```
scripts/stata/
├── 00_install.do        # ssc install packages, set globals, paths
├── 01_clean.do          # raw → cleaned panel
├── 02_descriptive.do    # summary tables, balance, attrition
├── 03_analyze.do        # main regression specs
├── 04_robustness.do     # alt specs, sensitivity
├── 05_tables_figures.do # estout/esttab + graph export
└── 99_run_all.do        # do "01_clean.do" / do "02_..." / ...
```

The 99-script is the **one-command reproduction**: `do scripts/stata/99_run_all.do` from the repo root produces every output the paper cites. AEA Data Editor checks this exact shape.

## 3. Outputs convention

All outputs land in `scripts/stata/_outputs/`:

```
scripts/stata/_outputs/
├── 01_log.smcl                 # captured stdout per script
├── clean_panel.dta             # cleaned data
├── descriptives.csv            # summary stats
├── main_results.tex            # esttab → .tex for direct \input{} in paper
├── balance_table.tex
├── fig_eventstudy.pdf          # graph export, vector
└── sessionInfo.txt             # capture stata version + installed pkg versions
```

`sessionInfo.txt` is mandatory. Generate via:

```stata
* At end of 00_install.do (or via a dedicated sessioninfo subroutine):
log using "scripts/stata/_outputs/sessionInfo.txt", text replace
which estout
which reghdfe
which ivreg2
about
log close
```

This gives the AEA / referee / future-you the package versions actually used.

## 4. Tables (estout / esttab)

Use `esttab` for any table that appears in the paper. **Never hand-format a table in LaTeX** — the `\input{}` pattern means the table cell values come from the actual estimation:

```stata
quietly: reghdfe y x1 x2, absorb(unit time) cluster(unit)
eststo m1
quietly: reghdfe y x1 x2 controls, absorb(unit time) cluster(unit)
eststo m2

esttab m1 m2 using "scripts/stata/_outputs/tab_main.tex", replace ///
    booktabs label                              /// use the variable labels you set
    se(2) b(3)                                  /// SE in parens, 3-decimal coeffs
    star(* 0.10 ** 0.05 *** 0.01)              /// significance convention
    stats(N r2, fmt(%9.0fc %9.3f) labels("Observations" "R²")) ///
    nonotes addnote("Robust SEs clustered at unit level.")
```

Then in the manuscript: `\input{scripts/stata/_outputs/tab_main.tex}` — table values update mechanically every time the .do file runs.

## 5. Significance-stars convention

The default is `* 0.10 ** 0.05 *** 0.01` (the econ convention; matches AER / QJE / JPE / ECMA defaults). Political science journals (APSR / AJPS / JOP) often use the same. Document the convention in the table note even though it's "obvious" — referees read the notes.

For one-tailed contexts (rare in published work), use `* 0.05 ** 0.025 *** 0.005` and explain why in the note. Default to two-tailed.

## 6. Clustering and SE conventions

- **This project's specification, always:** individual FE + year FE + governorate×year FE,
  clustered at the individual level, regardless of outcome or specification:

  ```stata
  reghdfe y treatment, absorb(individual_id year governorate#year) vce(cluster individual_id)
  ```

  Do not substitute `, cluster()` for `, vce(cluster ...)`, and do not drop the
  `governorate#year` term for "a simpler spec" without flagging it explicitly as a robustness
  check, not the main spec.
- **`reghdfe` defaults to df-adjusted clustering** but check — Stata's `, cluster()` and `reghdfe ... , cluster()` use different df adjustments in some edge cases. The version pin at top of file is partial defence; explicit `, dofadj() ` is the rest.
- **Bootstrap clustering** for very small clusters (< 50 groups): use `cluster bootstrap` not `bootstrap, cluster()`. Not expected to bind here (ELMPS individual-level clusters are large), but keep the discipline for any subgroup analysis with a small number of governorates.

## 6b. Data availability by round — check before writing any code

ELMPS covers rounds **1998, 2006, 2012, 2018, 2023**. Outcome availability is round-specific,
not universal — the single most common way a `.do` file in this project goes wrong is
estimating an outcome on a round it doesn't exist in:

| Outcome | 1998 | 2006 | 2012 | 2018 | 2023 |
|---|:-:|:-:|:-:|:-:|:-:|
| Labor force participation | ✓ | ✓ | ✓ | ✓ | ✓ |
| Domestic work hours | | | ✓ | ✓ | |
| Gender role attitudes — women | ✓ | ✓ | ✓ | ✓ | ✓ |
| Gender role attitudes — men | | | | ✓ | ✓ |

Before estimating any spec on one of these outcomes:

1. Restrict the sample to the rounds where the outcome is actually collected — `keep if inlist(year, 2012, 2018)` for domestic work hours, `keep if inlist(year, 2018, 2023) & sex == 1` for men's gender-role attitudes, etc.
2. State the round restriction in the `.do` file's header comment and in any table note — a
   reader of `output/tables/tab_domestic_work.tex` should not have to guess it's a 2-round panel.
3. Never pool rounds where an outcome is missing as though it were legitimately absent
   (attrition) rather than not-asked (survey design) — Stata will happily run a regression on
   whatever non-missing rows remain, silently dropping the rounds where the question wasn't
   fielded, and the resulting N will look plausible without the sample actually meaning what the
   table implies.

## 6c. Kleven hybrid pseudo-panel + event-study checklist

Following Kleven's "The Child Penalty Atlas" methodology for the marriage/childbirth event
study:

- **Event time is anchored to the individual event** (first marriage date, or first birth date)
  — not calendar year. Construct `event_time = year - marriage_year` (or `- first_birth_year`)
  before collapsing to cohort/event-time cells.
- **Reference period is normalized to exactly one period**, conventionally `event_time == -1`
  (the last pre-event observation) — state explicitly which period is omitted, since the
  estimating equation's coefficients are relative to it, not to zero.
- **Pseudo-cohort construction** (when true panel linkage is broken by attrition or by
  ELMPS's round spacing not aligning with event time) groups individuals into cohorts by
  birth-year/marriage-year bins, and the pseudo-panel outcome is the **cohort-period mean**, not
  the individual value — verify the collapse (`collapse (mean) y, by(cohort event_time)`)
  happens before the event-study regression, not after.
- **Report whether a given event-study result is estimated on the true panel or the
  pseudo-panel** — they have different standard-error structures (pseudo-panel SEs need a
  grouped/cluster correction for the number of underlying individuals per cell, not just the
  number of cells).

## 7. Balance / attrition discipline

For every RCT or quasi-experimental design:

- **Balance table** via `iebaltab` (World Bank `ietoolkit` package). Don't reinvent.
- **Attrition table** — fraction missing per round, balanced by treatment status. Same `iebaltab` invocation with different outcome.
- **Manipulation checks** (survey experiments) — pass rate per arm. Document in the paper, not just the .do.

## 8. Figures (graph export)

```stata
graph export "scripts/stata/_outputs/fig_eventstudy.pdf", replace as(pdf)
graph export "scripts/stata/_outputs/fig_eventstudy.png", replace as(png) width(2000)
```

Both vector (PDF for the paper) and raster (PNG for slides). Don't rely on the auto-generated `.gph` — it's not portable across Stata versions.

## 8b. coefplot conventions (recurring source of malformed figures)

Four hard rules, all learned the hard way — verify each before shipping an event-study or
coefficient-plot figure:

- **Never `noalphabetical`.** It silently reorders coefficients away from the order you specified
  in the plot command, which is exactly backwards for an event-study plot where order = time.
- **Degree symbol is `{char 176}`, never `{&deg}`.** `{&deg}` is a Stata Markup and Control
  Language SMCL directive that does not render in graph text; `{char 176}` is the actual degree
  glyph. This matters anywhere an axis label needs a degree symbol.
- **`ciopts(lcolor(...))` colors are assigned per model, not per coefficient.** When plotting
  multiple `eststo` models on one coefplot, `ciopts(lcolor(color1 color2 ...))` supplies one
  color **per model**, in model order — it is not a per-coefficient palette. Supplying N
  coefficient colors when you have M models (N ≠ M) either errors or silently mis-assigns colors.
- **`levels(95 90)` needs two colors in `ciopts lcolor()`.** Plotting two confidence levels means
  `ciopts(lcolor(color1 color2))` must supply exactly two colors — one per level, not per model —
  when `levels()` itself has two values. (This and the previous rule both consume `ciopts
  lcolor()`, so combining multi-model **and** multi-level in one plot needs care: check
  `coefplot`'s own documentation for how the two axes of color assignment combine before assuming
  either rule alone.)

## 9. Common Stata → R / Stata → AEA traps

| Trap | Fix |
|---|---|
| `reg y x, cluster(id)` without explicit df-adjustment | Use `reghdfe` for explicit df; document the adjustment in the table note |
| Bootstrap reps inconsistent across runs | `set seed` + `set sortseed` at top of file; use `bootstrap, reps(N) seed(X)` |
| `replace` modifying observed data | Always `gen new_var = ... ` and inspect before `drop` |
| `merge 1:1` without `assert` | `merge 1:1 id using foo, assert(3)` — fail loud on mismatched keys |
| `if` on missing values | Stata treats `.` as `+∞` in inequality comparisons. `if x > 5 & x != .` for non-missing-and-greater-than-5 |
| `egen sum(x)` deprecated | `egen total(x)` is the modern form; `egen sum()` still works but `bysort id: egen y = total(x)` is the safer pattern |

## 10. AEA Data Editor compliance

The [AEA Data Editor checklist](https://aeadataeditor.github.io/) requires:

- `README.md` at repo root describing data source, computational requirements, run instructions.
- A single command that reproduces all results (`do scripts/stata/99_run_all.do`).
- All scripts numbered and ordered.
- A separate `requirements.txt`-equivalent — for Stata, that's the `sessionInfo.txt` from §3 plus a list of `ssc install` commands in `00_install.do`.
- License (MIT / GPL / similar).
- No hard-coded paths — use globals or relative paths from repo root.

## Enforcement

- [`/stata-replication`](../skills/stata-replication/SKILL.md) is the analogue of `/data-analysis` for Stata. It emits .do files conforming to this convention.
- [`/audit-reproducibility`](../skills/audit-reproducibility/SKILL.md) handles Stata `.dta` outputs alongside R `.rds` (via `haven` or `pyreadstat`).
- [`/review-r`](../skills/review-r/SKILL.md) is R-specific; a Stata-equivalent is on the v2.0-backlog.

## Cross-references

- [`r-code-conventions.md`](r-code-conventions.md) — analogous discipline for R-first pipelines.
- [`replication-protocol.md`](replication-protocol.md) — tolerance contract that applies across R / Stata / Python.
- [`../references/release-engineering.md`](../references/release-engineering.md) — shipping an `.ado` package or a replication package as a versioned artifact: message and silent-resolution censuses, preflight archives, generated status contracts, downstream pinning.
- [stata-mcp on GitHub](https://github.com/SepineTam/stata-mcp) — the MCP server that lets Claude Code execute Stata `.do` files. Install via `claude mcp add stata-mcp --scope user -- uvx stata-mcp`.
