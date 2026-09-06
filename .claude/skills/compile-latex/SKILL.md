---
name: compile-latex
description: Compile a LaTeX document (Beamer research-presentation deck under `slides/`, or the paper draft under `paper/`) with XeLaTeX (3 passes + bibtex). Use when user says "compile", "build the slides", "build the paper", "rebuild the PDF", "run latex", "render the tex", or asks why a `.tex` file isn't producing a PDF.
argument-hint: "[slides/<name> or paper/<name>, without .tex extension]"
allowed-tools: ["Read", "Bash", "Glob"]
---

# Compile LaTeX (Beamer slides or paper draft)

Compile a `.tex` file using XeLaTeX with full citation resolution. `$ARGUMENTS` is a path
relative to the repo root, without the `.tex` extension (e.g. `slides/results_talk` or
`paper/chapter1`). If no directory prefix is given, default to `slides/`.

## Steps

1. **Determine the directory and filename** from `$ARGUMENTS`, then compile with the 3-pass sequence:

```bash
DIR=$(dirname "$ARGUMENTS")     # slides or paper
FILE=$(basename "$ARGUMENTS")
cd "$DIR"
TEXINPUTS=../preambles:$TEXINPUTS xelatex -interaction=nonstopmode "$FILE.tex"
BIBINPUTS=..:$BIBINPUTS bibtex "$FILE"
TEXINPUTS=../preambles:$TEXINPUTS xelatex -interaction=nonstopmode "$FILE.tex"
TEXINPUTS=../preambles:$TEXINPUTS xelatex -interaction=nonstopmode "$FILE.tex"
```

**Alternative (latexmk):**
```bash
cd "$DIR"
TEXINPUTS=../preambles:$TEXINPUTS BIBINPUTS=..:$BIBINPUTS latexmk -xelatex -interaction=nonstopmode "$FILE.tex"
```

2. **Check for warnings:**
   - Grep output for `Overfull \\hbox` warnings
   - Grep for `undefined citations` or `Label(s) may have changed`
   - Report any issues found

3. **Open the PDF** for visual verification:
   ```bash
   open "$DIR/$FILE.pdf"          # macOS
   # xdg-open "$DIR/$FILE.pdf"    # Linux
   ```

4. **Report results:**
   - Compilation success/failure
   - Number of overfull hbox warnings
   - Any undefined citations
   - PDF page count

## Why 3 passes?
1. First xelatex: Creates `.aux` file with citation keys
2. bibtex: Reads `.aux`, generates `.bbl` with formatted references
3. Second xelatex: Incorporates bibliography
4. Third xelatex: Resolves all cross-references with final page numbers

## Important
- **Always use XeLaTeX**, never pdflatex
- **TEXINPUTS** is required: the shared preamble lives in `preambles/`
- **BIBINPUTS** is required: `Bibliography_base.bib` lives in the repo root
- `slides/` decks are Beamer; `paper/` drafts are typically `article` class — the same 3-pass
  sequence applies to both, `preambles/header.tex` guards its Beamer-only content with
  `\@ifundefined{beamertemplate}` so it's safe to `\input{header}` from either.
