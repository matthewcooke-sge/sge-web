# Demand the Solutions: release handoff for a second machine (ROG)

For working on `tools/demand-solutions.html` away from the Bizon (e.g. the ROG
Strix while traveling). Read the repo's `CLAUDE.md` first; this doc adds only
the second-machine specifics and the release ritual. This file lives in `docs/`
because `tools/` is reserved for exactly two files per tool (see Naming in
CLAUDE.md).

House style applies to THIS file too: no em-dashes anywhere, ever.

## 1. Fresh-clone setup

This repo is PUBLIC and contains ZERO secrets by design (no API keys, no
tokens, no credential files). There is nothing to copy from the Bizon. A clone
is the whole setup:

```
git clone https://github.com/matthewcooke-sge/sge-web.git
cd sge-web
git config user.name "Matthew Cooke"
git config user.email "mc@matthewcooke.com"
```

- First push will trigger a GitHub sign-in via Git Credential Manager. Sign in
  as an account with write access to `matthewcooke-sge/sge-web`.
- No build step, no npm, no dependencies. The tool is one self-contained HTML
  file with inline CSS and JS.
- To eyeball changes locally, any static server works:
  `python -m http.server 8137` then open
  `http://localhost:8137/tools/demand-solutions.html`.
- Local-preview caveat: the Action Network embed (US letter form) often shows
  blank locally because their anti-bot dislikes odd origins. The tool then
  shows its "Compose Your Letter" fallback button after a few seconds. That is
  expected and is NOT a breakage; the live Squarespace page behaves normally.

## 2. The release ritual (every change, no exceptions)

Deploys are pinned to immutable git tags, never `@main`. Reason: jsDelivr's
branch alias has served stale or mixed versions for 18+ minutes even after
purges. A tag is immutable, cached forever, and live the instant Squarespace
points at it. Never move or reuse a tag; always mint the next number.

1. Edit `tools/demand-solutions.html`.
2. Bump the version stamp in ALL THREE places (same number):
   - the CSS header comment near line 3: `SGE DEMAND SOLUTIONS TOOL vN`
   - the HTML comment: `<!-- SGE Demand Solutions BUILD vN -->`
   - the faint footer div: `>vN</div>`
3. Style gate before committing (both must return nothing):
   ```
   grep -P "\x{2014}" tools/demand-solutions.html
   ```
   (no em-dashes; also no `--` in prose). If share text changed, re-check the
   X limit: share line + " Demand it." + 23 chars for the link must be <= 280.
4. Commit and push to `main`:
   ```
   git add tools/demand-solutions.html
   git commit -m "demand-solutions vN: what changed"
   git push
   ```
5. Tag the release and push the tag:
   ```
   git tag demand-vN
   git push origin demand-vN
   ```
6. Verify the CDN serves the new tag BEFORE touching anything else:
   ```
   curl -s "https://cdn.jsdelivr.net/gh/matthewcooke-sge/sge-web@demand-vN/tools/demand-solutions.html" | grep -oE "BUILD v[0-9]+"
   ```
   Must print `BUILD vN`. Tags are usually live within seconds.
7. Repoint the loader IN THE REPO so it stays truthful: edit `TOOL_URL` in
   `tools/demand-solutions.loader.html` to `@demand-vN`, then commit and push
   that too ("Point demand loader to demand-vN").
8. Go live: in the Squarespace `/demand` Code Block, change the one
   `@demand-v(N-1)` to `@demand-vN` and save. This is the only step that
   requires Matthew's Squarespace login; it is what actually flips the live
   site.

The "LF will be replaced by CRLF" git warning is harmless; ignore it.

## 3. Verify checklist (after step 8)

- [ ] Hard-refresh `survivorsguidetoearth.com/demand` (mobile: pull to reload).
- [ ] The tiny footer version at the very bottom of the tool reads `vN`. If it
      still shows the old number you are on a cached copy; refresh again.
- [ ] Optional remote check of what the live page references:
      `curl -s -A "Mozilla/5.0" "https://www.survivorsguidetoearth.com/demand" | grep -o "sge-web@[^/]*/tools/demand-solutions.html"`
- [ ] Click the changed thing (the edited card, button, or copy) and confirm.
- [ ] On a phone: buttons respond on FIRST tap and none stick dark (hover
      styles are gated behind `@media (hover: hover)`; keep it that way).
- [ ] US flow: either the Action Network letter form renders, or its fallback
      "Compose Your Letter" button appears. Blank-box-with-no-fallback is a bug.

## 4. Current state and parked items (as of 2026-07-19)

- Live release: **demand-v16** (loader + Squarespace both point at it).
- v2 to v16 shipped 2026-07-01 to 2026-07-18, mostly editorial: new cards
  (Reclaim the Supreme Court; Return Social Media to Society), the immigration
  card retitled to "Immigration Enforcement Crisis" and rewritten twice, media
  outlet addresses corrected (NYT letters desk; Guardian; CNN and Fox are form
  links, not emails), paragraph-broken letters, "Demand it." CTA, how-to video
  lightbox, iOS touch hardening, Action Network fallback.
- Parked (safe to do in any future release):
  - Immigration wording mismatch: long letter says "world's most powerful
    nations"; the short share/summary still say "wealthiest nations". Align
    whenever the card is next touched.
  - The static compose-box heading "This is a proven solution. Demand it." is
    intentionally KEPT (Matthew's call). Do not remove.
  - The Squarespace page SEO title contains an em-dash; that lives in
    Squarespace settings, not this repo, and is Matthew's to fix.
- Bigger picture: an SGEv2 (survivorsguide.earth) integration is planned that
  replaces this embed with a Sanity-backed route; see the report in the SGE-OS
  bus (`bus/inbox/2026-07-19/demand-solutions.integration-report.md` on the
  Bizon). Until that lands, this repo remains the live tool's home and this
  ritual remains the deploy path.
