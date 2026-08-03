# Run Plan Tracker

A single-file PWA for tracking a 12-week run block. Add it to the iPhone home
screen and it behaves like an app: opens offline, no account, no network calls,
no analytics. Everything is stored in `localStorage` on the device.

Runs are Tue / Thu / Sun, with Sunday as the long run.

| Week | Starts | Tue / Thu / Sun | Total |
|-----:|--------|-----------------|------:|
| 1  | Aug 3, 2026  | 7 / 7 / 8    | 22 km |
| 2  | Aug 10       | 7 / 8 / 9    | 24 km |
| 3  | Aug 17       | 8 / 8 / 10   | 26 km |
| 4  | Aug 24       | 6 / 6 / 7    | 19 km — recovery |
| 5  | Aug 31       | 8 / 8 / 10   | 26 km |
| 6  | Sep 7        | 8 / 9 / 11   | 28 km |
| 7  | Sep 14       | 9 / 10 / 11  | 30 km |
| 8  | Sep 21       | 7 / 7 / 8    | 22 km — recovery |
| 9  | Sep 28       | 9 / 10 / 11  | 30 km |
| 10 | Oct 5        | 10 / 10 / 12 | 32 km |
| 11 | Oct 12       | 11 / 11 / 12 | 34 km — peak |
| 12 | Oct 19       | 8 / 8 / 9    | 25 km — recovery |

318 km across the block, finishing Sunday Oct 25, 2026.

## Install

Open the page in Safari on iPhone, then **Share → Add to Home Screen**. It
launches full screen with its own icon and works with no signal.

## Deploy

Push to GitHub and turn on Pages (Settings → Pages → deploy from branch,
`main`, root). Every path in the app is relative, so it works from a project
subpath like `/Run-Plan-Tracker-Bridge/` without changes.

To try it locally:

```
npx http-server -p 8099 -c-1 .
```

Service workers need `http://localhost` or HTTPS — opening `index.html` as a
`file://` URL will not register one.

## What it tracks

Per run: completed or skipped, actual distance and duration, effort
(easy / moderate / hard), and any pain with a body-area tag and a note. Pace is
derived and shown as you type.

Per week: actual against planned with a percentage, rolling 4-week volume, and
a streak of consecutive completed runs.

## The parts that push back

- **Over-reaching.** Logging more than 110% of a week's planned volume raises a
  warning on that week.
- **Recovery weeks.** Drawn in blue throughout. Saving a run longer than planned
  during one asks you to confirm first, and says by how much.
- **Repeated pain.** Two logs against the same body area inside 14 days raises a
  banner suggesting a cutback. It cannot be dismissed — it clears itself once
  the older entry falls outside the window.
- **A missed week.** If a week passes with nothing completed, the app offers to
  shift that week and everything after it forward to the next Monday, rather
  than letting you rejoin at a volume you have not built up to.

## Your data

**Stats → Export JSON** downloads a backup; **Copy** puts the same JSON on the
clipboard when the download route is awkward on iOS. **Import** restores from
either. Clearing Safari website data erases everything, so export occasionally.

The plan itself is hardcoded in `PLAN` at the top of the script in
`index.html`, alongside `WEEK1_MONDAY`. Editing those changes the block; logged
runs are keyed by week and day, so they survive a distance edit but not a
reordering of the plan.

## Files

```
index.html     the whole app — markup, styles, logic
manifest.json  PWA metadata
sw.js          service worker, offline shell cache
icon.svg       source icon
icon-180.png   apple-touch-icon (iOS ignores SVG here)
icon-512.png   manifest icon
```

Vanilla JS, no build step, no dependencies, no CDN.
