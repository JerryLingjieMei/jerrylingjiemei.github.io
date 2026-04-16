---
name: update-portfolio
description: >-
  End-to-end pipeline that refreshes the user's CV, personal website, and LinkedIn from gathered achievements, then commits and pushes to GitHub. Chains gather-achievements → export-cv → update-website → update-linkedin, then creates a single commit per changed file group and pushes. Use when the user wants to do a full portfolio refresh in one go.
---

# `update-portfolio` Skill

Orchestrates the four downstream skills and finishes with a git commit + push.

## Usage

```bash
/update-portfolio [--dry-run] [--skip linkedin] [--no-push]
```

Flags:
- `--dry-run` — do gather + diff only; show what WOULD change; touch no files.
- `--skip <skill>` — skip `cv`, `website`, or `linkedin`.
- `--no-push` — commit locally but don't push to origin.

## Implementation

### Step 1: Gather

Invoke `/gather-achievements`. Expect `.cache/achievements.json` and `.cache/achievements_diff.json`. If it fails hard (e.g., no network), abort the pipeline and surface the error.

If the diff is empty (nothing new in Scholar/GitHub vs CV/website), report "No changes to apply" and exit success without touching files.

### Step 2: Preview

Before making any edits, print a preview:

```
Pipeline plan:
  Publications to add: N  (titles…)
  Projects to add:     M  (repo names…)
  LinkedIn sections:   K  (About / Pubs / Featured)
```

If `--dry-run`, stop here.

### Step 3: Export CV

Invoke `/export-cv`. On failure (e.g., compile error), stop. Do NOT run the remaining steps — website and LinkedIn updates should reflect the same state that's in the CV.

### Step 4: Update website

Invoke `/update-website`.

### Step 5: Update LinkedIn

Invoke `/update-linkedin` unless `--skip linkedin`.

### Step 6: Commit and push

Group changes into logical commits (do NOT use `git add -A`):

```bash
# CV commit
git add cv/main.tex cv/main.pdf cv/sample.bib
git commit -m "Update CV with <N> new publications and <M> projects"

# Website commit
git add index.html
git commit -m "Add <N> new publications to website"
```

Skip a commit if its file group has no staged changes.

If `--no-push` is not set:
```bash
git push origin main
git push pages main        # the GitHub Pages remote (see: git remote -v)
```

The website deploys from the `pages` remote (`jerrylingjiemei.github.io.git`) — both remotes must be pushed for the change to go live.

### Step 7: Final report

```
✓ Gathered:  <N> pubs, <M> repos
✓ CV:        cv/main.pdf (<kb> KB) — pushed to Overleaf project "CV"
✓ Website:   index.html updated — deployed to pages
✓ LinkedIn:  .cache/linkedin_update.md ready, 3 edit tabs opened in Chrome
✓ Git:       2 commits pushed to origin + pages
```

## Safety

- Always confirm with the user before pushing if any single commit touches more than 5 files or the commit diff exceeds 200 lines.
- Never force-push. Never push to a non-main branch without asking.
- Back up each edited file to `/tmp/*.bak.<timestamp>` at the start of its step.
- If any step fails, stop — do not try to recover. Leave backups in place and tell the user where they are.

## Non-goals

- Does NOT open a pull request — commits go straight to main because this is a personal portfolio repo.
- Does NOT run website build/tests (the site is static).
- Does NOT invent publications, repos, or LinkedIn content beyond what the diff contains.
