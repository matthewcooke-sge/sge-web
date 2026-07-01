# sge-web

Public host for Survivors Guide to Earth's embeddable web tools.

These are self-contained HTML tools that get pulled into Squarespace pages via
the jsDelivr CDN. This repo is a public asset host, nothing more. It holds no
secrets, no backend, no build step. Do not put private code or API keys here.

## How a tool gets onto the site

Each tool is two files in `tools/`:

- `NAME.html`, the actual tool (HTML + inline CSS + inline JS).
- `NAME.loader.html`, a tiny snippet pasted into a Squarespace Code Block.

The loader is small enough to fit Squarespace's Code Block character limit
(~65,536). It fetches `NAME.html` from jsDelivr and injects it into the page,
so the tool runs *on* the real Squarespace page (it can read the page URL,
share to your domain, and see Outseta for member gating).

### First-time setup for a tool

1. This repo must be **public** (jsDelivr only serves public repos).
2. Put `NAME.html` and `NAME.loader.html` in `tools/`.
3. Open `NAME.loader.html`, set `TOOL_URL` to:
   `https://cdn.jsdelivr.net/gh/matthewcooke-sge/sge-web@main/tools/NAME.html`
4. In the Squarespace page's Code Block: select all, delete, paste the loader.
   Save, then **Publish** the site.

## How to update a tool (the common case)

You do not touch Squarespace to update a tool. The loader in the Code Block
never changes; it just fetches whatever the hosted file currently says. So the
whole update loop is:

1. **Edit the file on GitHub.** Either edit `tools/NAME.html` directly in
   GitHub's web editor (open the file, click the pencil, edit, commit), or
   upload a new version (Add file, Upload files, GitHub asks to replace the
   existing one, commit). For small text tweaks the web editor is fine; for a
   rebuilt file, upload-and-replace.
2. **Commit.** That saves and pushes it to `main`.
3. **Purge jsDelivr so it goes live now.** jsDelivr caches for up to 24 hours,
   so without this the change appears eventually but not immediately. Go to
   https://www.jsdelivr.com/tools/purge and paste the full file URL:
   `https://cdn.jsdelivr.net/gh/matthewcooke-sge/sge-web@main/tools/NAME.html`
   Click purge. Within a minute or two the live tool reflects the change.

That is the entire loop. Squarespace is touched only once, when a tool is first
added.

### Verifying a change actually went live (the version stamp)

Each tool carries a small version number, both as an HTML comment near the top
(`<!-- SGE ... BUILD vN -->`) and as a faint `vN` in the tool's footer. Bump
this number every time you change the file. Then, after committing and purging:

1. Load the live page (e.g. `/demand`) and scroll to the tool's footer.
2. If the version number matches the version you just deployed, the update is
   live.
3. If it still shows the old number: the commit did not save, or the purge has
   not propagated yet. Wait a minute, then hard-refresh (Cmd/Ctrl+Shift+R) or
   check in a private/incognito window. Re-purge if needed.

The version stamp removes all guessing about whether a paste/upload "took." A
mismatch always means the new file is not being served yet, never a CSS quirk.

### If a change still will not show

- Hard-refresh the page (Cmd/Ctrl+Shift+R) or open it in a private window. That
  separates *your browser's* cache from jsDelivr's. The version stamp tells you
  which one is stale.
- Re-purge the jsDelivr URL and wait a minute.
- Confirm you committed to `main` and edited the file under `tools/`.

### Optional: instant updates without the purge step

`@main` plus a manual purge is the simplest reliable method; stick with it
unless purging becomes annoying. If you later want changes live instantly, pin
a specific commit hash in the jsDelivr URL (`@COMMITHASH` instead of `@main`)
and update the loader's `TOOL_URL` to the new hash each release. That trades one
manual step (purge) for another (bump the hash in the loader, which does mean
re-pasting the loader into Squarespace), so it is usually not worth it.

## Adding a NEW tool later

Same pattern, no new decisions:

1. Add `tools/newtool.html` and `tools/newtool.loader.html`.
2. Point the new loader's `TOOL_URL` at `.../tools/newtool.html`.
3. Paste that loader into the relevant Squarespace page.

## Naming rules (keep these consistent)

- Lowercase, hyphens, no spaces, no capitals: `demand-solutions`, not
  `Demand_Solutions`. CDN URLs are case-sensitive; this avoids 404s.
- One tool = `NAME.html` + `NAME.loader.html`. Nothing else.
- Version with git history, not filenames. No `tool-v2.html` or
  `tool-final.html`. To freeze a version live, pin a commit hash in the
  jsDelivr URL (`@COMMITHASH` instead of `@main`) instead of renaming files.

## Editorial / build rules for the tools themselves

- **No em-dashes anywhere** (copy, UI, comments). This is a hard SGE style rule.
- Each tool is one self-contained `.html`: all CSS and JS inline, no external
  build. That is what lets it drop into a Squarespace Code Block or load via the
  fetch-and-inject loader.
- **The Squarespace spacing trap:** the tools scope everything under a wrapper
  (e.g. `.sge-ds`) and open with a `* { margin:0; padding:0 }` reset. Because of
  that reset, every intentional margin/padding must carry `!important` or it gets
  eaten live. Where two `<p>` elements need a gap and `!important` still loses to
  Squarespace, use a fixed-height spacer `<div>` instead (geometry can't be
  collapsed). Do not "clean up" the `!important`s or spacer divs; they are load-
  bearing.

## What does NOT belong in this repo

The future SGE orchestration system and the standalone site rebuild
(analytics, design, project management, scheduler, orchestrator) are separate,
mostly private projects. They get their own repos. This repo stays scoped to
public embeddable tools so it can safely remain public on a CDN.
