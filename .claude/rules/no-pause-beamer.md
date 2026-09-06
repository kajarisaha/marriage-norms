---
paths:
  - "slides/**/*.tex"
---

# Overlays in Beamer Slides — used sparingly, on purpose

The template default for *taught* courses bans `\pause`/`\onslide`/`\only`/`\uncover` outright,
because a recorded lecture has no live pacing to protect and overlays just add edit friction.
That rationale doesn't hold for a **live research talk**: progressive reveals are normal there —
building up a DAG node by node, or revealing regression-table columns one at a time while you
talk through them — and reviewers who default to "add `\pause`" for classroom decks are giving
advice for the wrong medium.

**Guideline for this project:**

- Overlays are fine for a genuine progressive-disclosure need tied to what you're saying out
  loud in the moment (revealing one coefficient column, building one edge of a DAG at a time).
- Overlays are **not** a substitute for splitting two genuinely separate points into two slides.
  If you'd keep both halves on screen simultaneously once revealed, it's one slide with an
  overlay; if you're really making two different points, make two slides instead.
- Don't let a review agent add `\pause` reflexively "for pacing" — that's the classroom-deck
  reflex this rule exists to override. An overlay should trace to a specific line you'll be
  saying when you click forward.

If you find overlays are creating more clutter than clarity, tighten this back to the hard ban —
just say so and this file reverts.
