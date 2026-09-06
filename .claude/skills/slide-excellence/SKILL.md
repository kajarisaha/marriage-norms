---
name: slide-excellence
description: Multi-agent comprehensive slide review (visual + proofreading + substance, plus TikZ conditionally). Use when user says "full review", "excellence pass", "comprehensive check", "review everything", "pre-release review", "slide excellence", or before presenting a deck. Fanout wrapper — for a single lens, use `/visual-audit` or `/proofread` directly.
argument-hint: "[TEX filename under slides/] [--fast] [--skip-substance]"
allowed-tools: ["Read", "Grep", "Glob", "Write", "Bash", "Agent", "Task"]
context: fork
---

# Slide Excellence Review

Run a comprehensive multi-dimensional review of a Beamer research-presentation deck. Multiple
agents analyze the file independently, then results are synthesized.

> **Which slide-review skill do I want?**
>
> - **`/slide-excellence`** (this skill) — multi-agent fanout (visual + proofread + substance,
>   plus TikZ conditionally). Best for **pre-presentation** checks.
> - **`/visual-audit`** — single lens, layout/overflow/font/spacing only. Fast.
> - **`/proofread`** — single lens, grammar/typos/overflow/terminology.

**Important:** this orchestrator does **conditional** dispatch — it only spawns the subagents
that can actually produce useful output for the given file. No running `tikz-reviewer` on a
file with zero TikZ.

## Step 1: Identify the File

Parse `$ARGUMENTS` for the filename. Resolve path in `slides/` (or `paper/` if a paper draft is
being reviewed as prose — but for paper drafts prefer `/review-paper`, which is the dedicated
manuscript reviewer).

## Step 2: Pre-flight — Detect Conditions

Before spawning any agent, probe the file to determine which reviews make sense:

```bash
FILE="$resolved_path"

# Has TikZ diagrams?
has_tikz=$(grep -c '\\begin{tikzpicture}' "$FILE" 2>/dev/null); has_tikz=${has_tikz:-0}
```

Report the detection:

```
File:         slides/results_talk.tex
TikZ blocks:  1
```

## Step 3: Domain-reviewer sanity check (MANDATORY)

`.claude/agents/domain-reviewer.md` is already configured for this project's empirical-micro /
causal-identification content (TWFE, Kleven event-study, round-availability checks) — this step
is a lighter sanity check than the original template's, since customization is done. Verify the
file being reviewed hasn't drifted from that configuration (e.g., a stray
`AUTO-DETECT-TEMPLATE-MARKER` reappearing would mean someone reverted it) before spawning it.

## Step 4: Run Review Agents in Parallel

Spawn only the agents whose conditions hold:

**Always-on:**

- **Agent A: Visual Audit** (`slide-auditor`)
  Overflow, font consistency, box fatigue, spacing, images.
  Save: `quality_reports/[FILE]_visual_audit.md`.

- **Agent B: Proofreading** (`proofreader`)
  Grammar, typos, consistency, academic quality, citations.
  Save: `quality_reports/[FILE]_proofread_report.md`.

- **Agent C: Substance Review** (`domain-reviewer`)
  TWFE identification, Kleven event-study construction, round-availability, citation fidelity,
  code-spec alignment via the 6-lens framework.
  Save: `quality_reports/[FILE]_substance_review.md`.

**Conditional:**

- **Agent D: TikZ Review** (`tikz-reviewer`) — only if `has_tikz > 0`.
  Measurement-based collision audit (Bézier, gaps, boundaries, margins).
  Save: `quality_reports/[FILE]_tikz_review.md`.

**De-duplication:** if the user has already run one of these skills on this file in the current session (e.g., ran `/proofread` first, now running `/slide-excellence`), ask whether to reuse the existing report or re-run. Default: reuse (saves tokens).

## Step 5: Synthesize Combined Summary (reduce typed findings)

This is **fan-out → reduce** ([`orchestrator-protocol.md`](../../rules/orchestrator-protocol.md)): each agent returns `FINDING`s + a `SCORECARD` in the shared schema ([`orchestration-schemas.md`](../../references/orchestration-schemas.md)), and this step **stacks the typed scorecards** rather than re-reading each report by eye. The Overall Quality Score is the gate predicate over summed CRITICAL/MAJOR/MINOR counts. (Conditional dispatch means a skipped lens contributes no findings, not zeros to average.)

Only include sections for agents that actually ran.

```markdown
# Slide Excellence Review: [Filename]

**File:** [path]
**Detected:** TikZ=N
**Agents spawned:** [A, B, C, D] (skipped: D [no TikZ])

## Overall Quality Score: [EXCELLENT / GOOD / NEEDS WORK / POOR]

| Dimension | Critical | Medium | Low |
|-----------|----------|--------|-----|
| Visual/Layout | | | |
| Proofreading | | | |
| Substance | | | |
| TikZ (if ran) | | | |

### Critical Issues (Immediate Action Required)
### Medium Issues (Next Revision)
### Recommended Next Steps
```

## Step 6: Report Token/Time Budget

After completion, print an estimate:

```
Spawned N agents; approx token usage ~XXk. Sequential fallback
(one agent at a time) would cost ~XXk but take ~5× longer. For
cost-conscious reviews, run individual subagent skills directly
(/proofread, /visual-audit).
```

## Flag Reference

| Flag | Effect |
|---|---|
| `--skip-substance` | Don't spawn Agent C (domain-reviewer). |
| `--fast` | Spawn a single synthesis agent reading the file directly, rather than parallel subagents. Cheaper (~8k vs ~50k tokens) but less thorough. |

## Quality Score Rubric

| Score | Critical | Medium | Meaning |
|-------|----------|--------|---------|
| Excellent | 0-2 | 0-5 | Ready to present |
| Good | 3-5 | 6-15 | Minor refinements |
| Needs Work | 6-10 | 16-30 | Significant revision |
| Poor | 11+ | 31+ | Major restructuring |

## Why conditional dispatch matters

Running `tikz-reviewer` on a TikZ-free deck produces an empty report (wasted tokens). Conditional
dispatch cuts token cost and avoids running a reviewer that can't produce useful output.

## Findings are validated, not just written (v2.5)

This skill's reviewers emit findings under the machine-checked contract in
[`finding-schema.json`](../../references/finding-schema.json). Reports are JSON **arrays**.

**Smoke-test the harness before spending review effort** — a run that fans out reviewers and
then cannot write a valid report has wasted the whole pass:

```bash
echo '[]' | python3 scripts/validate-findings.py
```

Then, before presenting any summary:

```bash
python3 scripts/validate-findings.py <report>.json   # exit 0 required
```

What the contract forces, and why:

- **`rule`** — the documented rule or standard violated. A finding citing no rule is an
  opinion, and opinions do not gate a commit.
- **`failing_case`** — a concrete configuration under which the claim breaks, or the exact
  missing hypothesis. *"This could be clearer"* does not validate.
- **`id = sha1("<file>:<line>:<locus>")`** — deterministic, so dedup across rounds is
  exact and the two-strikes rule is checkable rather than eyeballed.
- **`mechanical`** — `true` only for fixes that cannot change a result (typo, cross-reference,
  formatting, label). **Never** for an estimand, assumption, specification, inference
  procedure, sample definition, or reporting language: those return to the researcher.

Apply the **per-lens evidence burdens** and the **"does NOT count" filters** in
[`orchestration-schemas.md` §7](../../references/orchestration-schemas.md) *before*
verification, so known false alarms never reach the judge. The verifier pass is
**refute-biased**: only `verdict: "confirmed"` findings ship; anything it cannot ground is
dropped, not downgraded to a warning.
