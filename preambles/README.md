# Preambles

Shared LaTeX/Beamer preamble for the research-presentation slides (`slides/`) and paper draft
(`paper/`) in this project.

## Usage in a deck or the paper

```latex
\documentclass{beamer}
\input{header}   % resolves via TEXINPUTS=../preambles:$TEXINPUTS

\title{Your Talk}
\author{You}
\date{\today}

\begin{document}
\frame{\titlepage}
% ...
\end{document}
```

Compile with `/compile-latex <file>` — the skill sets `TEXINPUTS` for you. For manual compilation:

```bash
cd slides
TEXINPUTS=../preambles:$TEXINPUTS xelatex -interaction=nonstopmode YourDeck.tex
```

## The palette

Named colors live in `header.tex` only — there is no second copy to keep in sync (no Quarto/SCSS
in this project). See `CLAUDE.md`'s "Beamer Palette" table for the current hex values:
`primary-blue`, `primary-gold`, `highlight-yellow`, `light-bg`, `jet`, `positive`, `negative`,
`neutral`, `hi-slate`, `hi-green`, `hi-red`.

## What's inside

- **Palette** — 11 named colors matching the SCSS.
- **Beamer theme assignments** — structure, titles, itemize, alert, blocks, minimal footer. Applied only under Beamer (`\@ifundefined{beamertemplate}`).
- **TikZ libraries** — `arrows.meta, positioning, calc, decorations.pathreplacing, fit, shapes.geometric, backgrounds`.
- **Shared TikZ styles** — `dag-node`, `decision-node`, `observed-edge`, `counterfactual-edge`, `confound-edge`, `observed-dot`, `counterfactual-dot`. Used by `templates/tikz-snippets/` and reusable in hand-written diagrams.
- **Convenience macros** — `\muted{...}`, `\key{...}`, `\good{...}`, `\bad{...}`, `\transitionslide{...}`.

## Extending

Add packages a specific deck or the paper needs *after* `\input{header}` in that file, not in
this file — that keeps the preamble small and auditable. Only add to `header.tex` if you are
certain every deck/draft needs it.

For a deck-specific preamble addon (rare), create `preambles/<name>-addon.tex` and `\input` it
after `header.tex`.
