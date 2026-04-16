---
name: export-cv
description: >-
  Apply new achievements (from .cache/achievements_diff.json) to cv/main.tex, then compile via Overleaf and save cv/main.pdf. Use when the user wants to push gathered achievements into their CV and produce an updated PDF.
---

# `export-cv` Skill

Writes achievement deltas into [cv/main.tex](cv/main.tex) in the correct sections, then delegates to [overleaf-compile](../overleaf-compile/SKILL.md) to produce [cv/main.pdf](cv/main.pdf).

## Usage

```bash
/export-cv [--diff .cache/achievements_diff.json]
```

## Implementation

### Step 1: Load diff

Read `.cache/achievements_diff.json`. If missing, run `/gather-achievements` first and abort with a message if that also fails.

### Step 2: Apply publication additions

For each publication in `diff.publications.scholar_only` (present in Scholar but not CV):

1. Locate the `\mysection{PUBLICATIONS}` `itemize` block in `cv/main.tex`.
2. Insert the new item at the top of the list (newest first), using the same `\item` + `\textbf{Lingjie Mei}` author-bolding convention:

   ```latex
   \item <authors with Lingjie Mei bolded>.
   \href{<url>}{<title>}.
   \textit{<venue>}, <year>.
   ```

3. Preserve the blank line between items.

### Step 3: Apply GitHub additions

For each significant repo (stars ≥ 3 OR description non-empty AND isFork=false) not yet mentioned in CV `OTHER MISC PROJECTS`, append a new `\item` to that section's `itemize` block with the repo description (one line).

Do NOT touch `ACTIVITIES` or `SERVICES` unless the diff explicitly flags one.

### Step 4: Preserve formatting

- Match the existing indentation exactly (4 spaces inside `itemize`).
- Do NOT reorder existing items.
- Do NOT change `\newcommand` definitions, preamble, or section structure.
- After editing, run `diff` against the backup and show a preview to the user before compiling.

### Step 5: Compile via Overleaf

Invoke the `overleaf-compile` skill:

```
/overleaf-compile cv/main.tex "CV"
```

This pushes the updated source to the "CV" project, compiles on Overleaf, and saves the PDF at `cv/main.pdf`.

### Step 6: Report

Show a short summary: number of publications added, number of projects added, PDF size, and the paths to both `cv/main.tex` and `cv/main.pdf`.

## Safety

- Always back up `cv/main.tex` to `/tmp/main.tex.bak.<timestamp>` before editing.
- If the post-edit file fails to compile (overleaf-compile reports errors), restore the backup and surface the LaTeX log.

## Non-goals

- Does NOT invent content. Only inserts items present in the diff.
- Does NOT prettify or restructure existing sections.
- Does NOT commit or push to git (the pipeline skill handles that).
