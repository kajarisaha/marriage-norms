# Workflow Quick Reference

**Model:** Contractor (you direct, Claude orchestrates)

---

## The Loop

```
Your instruction
    ↓
[PLAN] (if multi-file or unclear) → Show plan → Your approval
    ↓
[EXECUTE] Implement, verify, done
    ↓
[REPORT] Summary + what's ready
    ↓
Repeat
```

---

## I Ask You When

- **Design forks:** "Option A (fast) vs. Option B (robust). Which?"
- **Code ambiguity:** "Spec unclear on X. Assume Y?"
- **Replication edge case:** "Just missed tolerance. Investigate?"
- **Scope question:** "Also refactor Y while here, or focus on X?"

---

## I Just Execute When

- Code fix is obvious (bug, pattern application)
- Verification (tolerance checks, tests, compilation)
- Documentation (logs, commits)
- Plotting (per established standards)
- Deployment (after you approve, I ship automatically)

---

## Quality Gates (No Exceptions)

| Score | Action |
|-------|--------|
| >= 80 | Ready to commit |
| < 80  | Fix blocking issues |

---

## Non-Negotiables

- **Paths:** all relative from repo root; LaTeX resolves via `TEXINPUTS=../preambles`.
- **FE/clustering:** individual + year + governorate×year FE, `vce(cluster individual_id)` — always, no `, robust` substitutions.
- **Round availability:** check `CLAUDE.md`'s outcome-by-round table before any code touching LFP / domestic work hours / gender-role attitudes.
- **Figure standards:** `graph export` both PDF (vector, paper) and PNG (raster, slides); never rely on the `.gph` binary format.
- **Color palette:** `primary-blue #012169`, `primary-gold #B9975B`, `highlight-yellow #F2A900`, `positive #15803D`, `negative #B91C1C`, `neutral #525252` (`preambles/header.tex`).
- **Tolerance thresholds:** point estimates < 0.01, SEs < 0.05, N exact — see `quality-gates.md` / `replication-protocol.md`.

---

## Preferences

**Visual:** figures via Stata `graph export` only — never hand-drawn or typed into a table.
**Reporting:** concise; this is a working research session, not a teaching narrative.
**Session logs:** Always (post-plan, incremental, end-of-session)
**Replication:** strict — flag near-misses as EXPLAINED only with a named alternative spec, never silently.
**Check-ins:** more frequent during the first few sessions while the user learns the workflow — confirm before large/irreversible steps rather than batching silently.

---

## Exploration Mode

For experimental work, use the **Fast-Track** workflow:
- Work in `explorations/` folder
- 60/100 quality threshold (vs. 80/100 for production)
- No plan needed — just a research value check (2 min)
- See `.claude/rules/exploration-fast-track.md`

---

## Next Step

You provide task → I plan (if needed) → Your approval → Execute → Done.
