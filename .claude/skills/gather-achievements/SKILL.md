---
name: gather-achievements
description: >-
  Gather a structured snapshot of the user's recent academic/engineering achievements from Google Scholar, GitHub, and local description files (CV, website). Produces a single JSON with publications, repos, and service entries. Use when the user wants to refresh their CV/website/LinkedIn from current state.
---

# `gather-achievements` Skill

Collects fresh achievement data from three sources and merges it into one JSON:

- **Google Scholar** — publications list (titles, authors, venues, years, URLs)
- **GitHub** — repos owned or authored by the user (`gh repo list`)
- **Existing descriptions** — parses [cv/main.tex](cv/main.tex) and [index.html](index.html) to capture what is already reflected, so downstream skills can diff.

Output written to `.cache/achievements.json` (relative to repo root).

## Usage

```bash
/gather-achievements [--scholar-id <id>] [--github-user <user>]
```

Defaults are read from `cv/main.tex` (Scholar `user=` query parameter) and `cv/main.tex` Github URL. For this repo: Scholar ID `NBGAr-0AAAAJ`, Github user `JerryLingjieMei`.

## Implementation

### Step 1: Scholar

Use a `uv run` one-shot with `scholarly` (no API key needed, but rate-limited — fail softly):

```python
# /// script
# dependencies = ["scholarly"]
# ///
import json, sys
from scholarly import scholarly
author = scholarly.search_author_id(sys.argv[1])
scholarly.fill(author, sections=["publications", "basics", "indices"])
out = {
    "name": author.get("name"),
    "affiliation": author.get("affiliation"),
    "h_index": author.get("hindex"),
    "i10": author.get("i10index"),
    "citedby": author.get("citedby"),
    "publications": [],
}
for p in author["publications"]:
    bib = p.get("bib", {})
    out["publications"].append({
        "title": bib.get("title"),
        "authors": bib.get("author"),
        "venue": bib.get("citation") or bib.get("venue"),
        "year": bib.get("pub_year"),
        "num_citations": p.get("num_citations"),
        "url": p.get("pub_url") or p.get("eprint_url"),
        "scholar_id": p.get("author_pub_id"),
    })
print(json.dumps(out, indent=2))
```

If Scholar returns a CAPTCHA or times out, log a warning, write `scholar: null` to the output JSON, and continue. Do NOT retry.

### Step 2: GitHub

Use `gh` CLI (already authenticated):

```bash
gh repo list <user> --limit 200 --json name,description,isFork,isArchived,stargazerCount,primaryLanguage,url,updatedAt,pushedAt
gh api users/<user>/events/public --paginate -q '.[] | select(.type=="PushEvent") | {repo: .repo.name, time: .created_at}' | head -200
```

Filter out forks and archived repos. Keep name, description, URL, stars, primary language, last pushed-to date.

### Step 3: Local descriptions

Parse [cv/main.tex](cv/main.tex): extract `\item` entries in `PUBLICATIONS`, `SERVICES`, `OTHER MISC PROJECTS`, `ACTIVITIES`. Use simple regex — don't try to fully parse LaTeX.

Parse [index.html](index.html): extract `<div class="pub-card">` entries with their `data-year`, `data-topic`, `data-selected`, title, authors, conference text, and links.

### Step 4: Merge and write

Produce `.cache/achievements.json`:

```json
{
  "gathered_at": "2026-04-15T01:45:00Z",
  "scholar": { ... },
  "github": { "repos": [...], "recent_pushes": [...] },
  "cv": { "publications": [...], "services": [...], "misc": [...], "activities": [...] },
  "website": { "pub_cards": [...] }
}
```

### Step 5: Compute deltas

Also emit `.cache/achievements_diff.json` listing items that appear in Scholar/GitHub but not in the CV/website, and vice versa. Match publications by fuzzy title (lowercase, strip punctuation; require ≥ 0.85 sequence ratio).

## Non-goals

- No fetching of full-text PDFs.
- No automatic deduplication across the three sources — surface candidates in the diff and let the user confirm.
- No LinkedIn scraping (requires authenticated browser automation; out of scope).
