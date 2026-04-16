---
name: update-website
description: >-
  Add new publications (from .cache/achievements_diff.json) to index.html as new `<div class="pub-card">` entries in the right year/topic, matching the existing markup. Use when the user wants to reflect newly-gathered achievements on their personal website.
---

# `update-website` Skill

Updates [index.html](index.html) to add publication cards for items in the diff that are not already rendered.

## Usage

```bash
/update-website [--diff .cache/achievements_diff.json]
```

## Implementation

### Step 1: Load diff

Read `.cache/achievements_diff.json` → take `publications.missing_on_website`.

### Step 2: Determine insertion point

The pub list lives in `<div id="main-pub-card-container" class="hide">` in [index.html](index.html). Cards are ordered newest-first. For each new pub:

- Classify `data-topic` (`vision-graphics` | `scene-understanding` | `concept-learning` | other — ask the user if ambiguous).
- Use `data-year` from the publication year.
- Default `data-selected="true"` for first-author or equal-contribution papers, `"false"` otherwise.

### Step 3: Generate card markup

Template (preserve indentation from the existing file — 20 spaces for inner content):

```html
                    <div id="<slug>" class="pub-card" data-topic="<topic>" data-year="<year>"
                         data-selected="<true|false>">
                        <div class="row">
                            <div class="col-l col-xs-12 col-lg-3">
                                <img src="imgs/<slug>.png" width="100%"/>
                            </div>
                            <div class="col-r col-xs-12 col-lg-9">
                                <div class="pub-card-body">
                                    <h5 class="title"><TITLE></h5>
                                    <h6 class="authors">
                                        <AUTHORS with <u>Lingjie Mei</u> and * for equal>
                                    </h6>
                                    <p class="info">
                                        <a class="conference"><VENUE> <YEAR></a>
                                        <a href="<PAPER_URL>"> Paper</a>.
                                        <a href="<CODE_URL>"> Code</a>.
                                    </p>
                                </div>
                            </div>
                        </div>
                    </div>
```

### Step 4: Insert

Insert each new card **just after** the opening `<div id="main-pub-card-container" class="hide">` line, so new pubs appear first visually. Preserve every other card.

### Step 5: Image placeholder

If `imgs/<slug>.png` doesn't exist, leave the `<img>` reference in place but emit a TODO in the report: "Add imgs/<slug>.png". Do NOT generate images.

### Step 6: Validate

After editing, parse the file with `python -c 'from html.parser import HTMLParser; ...'` or simply check that opening/closing `<div>` tags balance in the pub-card-container block. If they don't, restore from backup.

## Safety

- Back up [index.html](index.html) to `/tmp/index.html.bak.<timestamp>` before editing.
- Never touch `<head>`, the bio section, or footer. Only the `main-pub-card-container` block.
- Do NOT delete existing cards (even if they look outdated) — that's a separate manual curation task.

## Non-goals

- Does NOT change CSS or JS.
- Does NOT create images (only references them).
- Does NOT rewrite the `Research Interest` paragraph. If the user wants that updated, ask.
