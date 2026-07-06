# Deploying the RFF (Captain Q) chat widget

## How the live site loads it
The client's **Squarespace** site (rottnestfastferries.com.au) embeds the widget with a
**fixed** script tag that we **cannot edit** (no website access):

```html
<script src="https://cdn.jsdelivr.net/gh/tradesolutionsai-dot/quincy-chatwidget@main/quincy-widget-final.js" defer></script>
```

Key consequences:
- The deployed file is **`quincy-widget-final.js`** — NOT `quincy-widget.js`.
  Always edit `quincy-widget-final.js` for anything that must go live.
- It's served by **jsDelivr from GitHub `@main`** (not Vercel — a Vercel redeploy does nothing here).
- Because the embed is pinned to `@main`, deploying = commit to `main` + **purge jsDelivr**
  (we can't switch to a pinned commit hash, since that needs an embed edit we can't make).

`quincy-widget.js` is a near-identical sibling (points at `mobile-widget-iframe.html` instead
of `mobile-widget-iframe-final.html`). **Keep both files in sync** — apply every change to both.

## Deploy steps
1. Edit `quincy-widget-final.js` (and mirror the change into `quincy-widget.js`).
2. `node --check` both files.
3. Commit to `main` and push (from VS Code: ⌘⇧P → **Git: Push**; the CLI sandbox has no GitHub creds).
4. **Purge the jsDelivr cache** — open this URL in a browser (returns JSON = success):
   ```
   https://purge.jsdelivr.net/gh/tradesolutionsai-dot/quincy-chatwidget@main/quincy-widget-final.js
   ```
5. Verify in an **incognito** window:
   - Open the CDN URL and ⌘F for a new-code marker (e.g. `quincy-widget-host`).
   - Load the RFF site: desktop (header/font/buttons) + a real phone (mobile teaser after ~5s).

## Rollback
`@main` can't roll back by embed. Instead:
```
git revert <bad-commit> --no-edit
git push          # via VS Code
```
then **purge again** (same URL).

## Caching note
`@main` is cached by jsDelivr (~12h edge) and browsers (~7-day max-age). After a purge,
**new / incognito** visitors get the update immediately; returning visitors may keep the old
file until their browser cache expires. Nothing we can do about that without embed access.

## Architecture quick facts
- Desktop widget mounts in a **shadow root** (`#quincy-widget-host`) to block Squarespace CSS.
- Font is **Rubik** (matches the Squarespace theme), loaded via Google Fonts + set as `--font`.
- Mobile is a separate **iframe** widget (`mobile-widget-iframe-final.html`) + a floating bubble
  and a proactive **teaser** nudge; mobile intentionally stays on DM Sans.
