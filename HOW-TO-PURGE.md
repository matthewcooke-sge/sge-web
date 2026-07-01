# How to purge jsDelivr (make an update go live now)

Quick reference for the one manual step after pushing a change.

## Why this exists

When someone loads the tool on the Squarespace page, it does NOT fetch the file
straight from GitHub. It fetches through jsDelivr, a CDN that sits in the middle
and keeps a cached (saved) copy of the file so it loads fast.

The catch: after you push a new version to GitHub, jsDelivr keeps handing out its
OLD cached copy for up to 24 hours, until its cache naturally expires. So your
change is on GitHub but visitors still see the old version.

Purging tells jsDelivr: "throw away your cached copy and grab the fresh one from
GitHub now." It forces the update through immediately instead of waiting a day.

Mental model: GitHub holds the real file. jsDelivr is a fast messenger holding a
photocopy. When you change the original, the messenger does not notice on its
own. Purging is tapping the messenger and saying "get the new copy."

## How to do it

1. Go to https://www.jsdelivr.com/tools/purge
2. Paste the full file URL into the box:
   https://cdn.jsdelivr.net/gh/matthewcooke-sge/sge-web@main/tools/demand-solutions.html
3. Click Purge.
4. Wait about a minute, then reload the live page (e.g. /demand) and check the
   footer version number.

## When you need it

Every time you push a change to a tool and want it live right away.

- Push to GitHub = the change exists.
- Purge jsDelivr = the change is visible to everyone now.

If you push and do NOT purge, the change still goes live on its own eventually
(within 24 hours), whenever jsDelivr's cache expires. Purging just skips the wait.

## Checking it worked (the version stamp)

Each tool shows a small version number in its footer. Bump that number on every
edit. After pushing and purging:

- Footer shows the NEW version = the update is live. Done.
- Footer still shows the OLD version = either jsDelivr has not finished purging
  (wait a minute, reload), or your own browser is showing a cached page
  (hard-refresh with Ctrl+Shift+R, or open a private/incognito window).

The version number is the truth-teller. If it matches what you pushed, the file
is live and anything stale is just your browser. If it does not match, the new
file is not being served yet, purge again and wait.

## The purge URLs for each tool

Purge the specific file you changed. Current tools:

- demand-solutions:
  https://cdn.jsdelivr.net/gh/matthewcooke-sge/sge-web@main/tools/demand-solutions.html

When you add a new tool, add its purge URL to this list.
