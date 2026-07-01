# CLAUDE.md , sge-web

Rules for any Claude Code session working in this repo. Read this before editing.

## What this repo is

A PUBLIC host for SGE's embeddable web tools. Files here are served to
Squarespace pages via the jsDelivr CDN. This is an asset host: no secrets, no
backend, no build step, no private code. Never add API keys or private logic.

Each tool is two files in `tools/`:
- `NAME.html` , the self-contained tool (HTML + inline CSS + inline JS).
- `NAME.loader.html` , the tiny snippet pasted into a Squarespace Code Block.

## HARD RULES , do not break these

1. **No em-dashes anywhere.** Not in copy, UI text, comments, or this file's
   edits. Use commas or rewrite. After any edit, verify zero U+2014 characters.
   This is a non-negotiable SGE style rule.

2. **Bump the version stamp on EVERY change to a tool file.** Each tool has a
   version in two places: an HTML comment near the top
   (`<!-- SGE ... BUILD vN -->`) and a faint `vN` in the footer. Increment both
   (v2 to v3 to v4 ...) whenever you change the file, so the live footer tells
   Matthew whether his update went live. Never leave the version unchanged after
   an edit.

3. **The Squarespace spacing trap. Do NOT "clean up" these.** Each tool scopes
   everything under a wrapper (e.g. `.sge-ds`) and opens with a universal reset
   `* { margin:0; padding:0 }`. Because of that reset, every intentional margin
   or padding MUST carry `!important` or it gets eaten on the live Squarespace
   page (even though it looks fine in preview). Where two `<p>` elements need a
   gap and `!important` still loses to Squarespace's own `<p>` rules, a
   fixed-height spacer `<div class="ds-gap">` is used instead (a div's height
   cannot be collapsed). The `!important`s and spacer divs are LOAD-BEARING.
   Removing them silently breaks spacing live. Leave them.

4. **Share text must fit X's 280-char limit.** The share builder appends
   " Demand it." plus a bare link (~35 chars on X, where the link counts as 23).
   So each issue's `data-share` should stay ~240 chars or under. A `data-share`
   that already ends in "Demand it." is not doubled (so do not hand-append it;
   let the builder add it). When editing any share text, re-check the X length.

5. **"I demand" is for politicians, not journalists.** The rep/candidate/
   conversation letters keep "I demand". The media-outlet path
   (`buildMediaLetter`) strips every "I demand" and reframes it as a coverage
   request. If you edit demands, do not reintroduce "I demand" into media
   letters.

6. **Each tool stays ONE self-contained file.** All CSS and JS inline. No
   external build, no imports that need compiling. That is what lets it load via
   the fetch-and-inject loader and (as a fallback) paste into a Code Block.

7. **Demand letters are auto-paragraphed.** `paragraphizeDemand()` inserts a
   blank line before each "I demand ..." clause so the compose box and the
   mailto/email bodies read as paragraphs, not a wall of text. Keep demand
   action clauses starting with "I demand ..." so they break cleanly. For the
   rare letter with no "I demand" clause (e.g. corporate-monsters), put a real
   blank line inside the `data-demand` value where you want a break; the parser
   preserves it and the function leaves it intact.

## How deploys work (so you push correctly)

- Editing a tool = edit `tools/NAME.html`, bump its version, commit, push to
  `main`. That is all. Do NOT touch Squarespace; the loader there fetches
  whatever the file currently says.
- After pushing, Claude purges jsDelivr automatically so the change goes live
  immediately (do not leave this to the human). Hit the purge endpoint for each
  changed file:
  `curl -s "https://purge.jsdelivr.net/gh/matthewcooke-sge/sge-web@main/tools/NAME.html"`
  A `"status": "finished"` response means it cleared. Then verify the CDN serves
  the new version, e.g.
  `curl -s "https://cdn.jsdelivr.net/gh/matthewcooke-sge/sge-web@main/tools/NAME.html" | grep -oE "BUILD v[0-9]+"`
- Tell the human it is live and remind them to hard-refresh the Squarespace page
  and confirm the footer version.

## Naming

Lowercase, hyphens, no spaces, no capitals: `demand-solutions`, not
`Demand_Solutions`. CDN URLs are case-sensitive. One tool = `NAME.html` +
`NAME.loader.html`, nothing else. Version via git history, never filenames
(no `tool-v2.html`).

## Not in this repo

The future SGE orchestration system and standalone site rebuild are separate,
mostly private projects with their own repos. Keep this repo scoped to public
embeddable tools so it can safely stay public.
