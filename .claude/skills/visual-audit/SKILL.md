---
name: visual-audit
description: Adversarial visual-layout audit of a Beamer `.tex` research-presentation deck. Flags overflow, font inconsistency, box fatigue, spacing, and alignment issues. Use when user says "visual audit", "check the layout", "does this overflow?", "look for visual issues", "audit the slides", or after reworking a deck's appearance. Does NOT check writing quality — pair with `/proofread`.
argument-hint: "[TEX filename under slides/]"
allowed-tools: ["Read", "Grep", "Glob", "Write", "Agent", "Task"]
disallowed-tools: ["Edit", "MultiEdit"]
---

# Visual Audit of Slide Deck

Perform a thorough visual layout audit of a Beamer slide deck.

## Steps

1. **Read the slide file** specified in `$ARGUMENTS` (under `slides/`).

2. **Compile and check for overfull hbox warnings** (see `/compile-latex`).

3. **Audit every slide for:**

   **OVERFLOW:** Content exceeding slide boundaries
   **FONT CONSISTENCY:** Inline font-size overrides, inconsistent sizes
   **BOX FATIGUE:** 2+ colored boxes on one slide (INV-3 in `content-invariants.md`)
   **SPACING:** Missing negative margins, tables/figures without explicit width
   **LAYOUT:** Missing transitions, missing framing sentences before identifying assumptions,
   overlay usage that doesn't trace to a genuine progressive-disclosure need

4. **Produce a report** organized by slide with severity and recommendations

5. **Follow the spacing-first principle:**
   1. Reduce vertical spacing with negative margins
   2. Consolidate lists
   3. Move displayed equations inline
   4. Reduce image/figure size
   5. Last resort: font size reduction (never below 0.85em)
