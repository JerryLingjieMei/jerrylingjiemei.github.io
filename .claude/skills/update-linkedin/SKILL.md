---
name: update-linkedin
description: >-
  Generate an updated "About" summary and "Experience"/"Publications" block for LinkedIn, based on .cache/achievements_diff.json. Saves to .cache/linkedin_update.md and opens the user's LinkedIn profile edit page in the browser for paste. Use when the user wants to refresh LinkedIn from gathered achievements.
---

# `update-linkedin` Skill

LinkedIn does not provide a public write API for personal profiles (only Marketing Developer Platform approval grants such access, which requires partnership review). This skill therefore:

1. Generates polished LinkedIn-formatted text for the user to paste.
2. Opens the relevant profile-edit URLs in Chrome so the paste takes one click.

## Usage

```bash
/update-linkedin [--diff .cache/achievements_diff.json] [--profile-url <url>]
```

Default profile URL is read from [index.html](index.html): `https://www.linkedin.com/in/jerry-lingjie-mei-125378161/`.

## Implementation

### Step 1: Load diff

Read `.cache/achievements_diff.json`. Focus on:

- `scholar.h_index`, `scholar.citedby`, `scholar.i10` for the headline summary.
- `publications.missing_on_website` (and any recent pubs) for the Publications section.
- `github.repos` filtered to stars ≥ 5 or ones the user has recently contributed to — for an optional Projects block.

### Step 2: Generate sections

Produce `.cache/linkedin_update.md` with three clearly-labeled sections, each ready to paste:

```markdown
## About (paste into the About section)

<2–4 sentences. Current role, research focus, citation counts if > 200, notable recent milestones.>

## Publications (add each as a new Publications entry)

For each pub:
- **Title**
- Publication: <Venue>, <Year>
- URL: <arxiv/conference URL>
- Authors: <comma-separated, bolded Lingjie Mei>
- Description (2 lines max)

## Featured Projects (optional — add to Featured section)

For each GitHub repo ≥ 5 stars or flagship:
- **Repo name** — one-line tagline — <URL>
```

### Step 3: Open edit pages

Open the relevant edit URLs in Chrome (macOS `open` command):

```bash
open -a "Google Chrome" "https://www.linkedin.com/in/<handle>/edit/intro/"           # About
open -a "Google Chrome" "https://www.linkedin.com/in/<handle>/details/publications/" # Publications
open -a "Google Chrome" "https://www.linkedin.com/in/<handle>/details/featured/"     # Featured
```

The `<handle>` is parsed from the profile URL.

### Step 4: Report

Tell the user:

- Where the generated text is saved (`.cache/linkedin_update.md`).
- Which three tabs were opened.
- That LinkedIn requires manual paste + save for each section (no API path is available).

## Why not fully automate?

LinkedIn's `/v2/me` profile write endpoints require an approved Marketing Developer Platform application. Browser automation (Playwright) can work but violates LinkedIn's User Agreement §8.2 and risks account suspension. The paste-assist flow is the cleanest reliable option for a personal profile.

## Non-goals

- No browser automation clicking into LinkedIn on the user's behalf.
- No posting to the LinkedIn feed.
- No scraping of LinkedIn — the diff source is Scholar + GitHub + local CV.
