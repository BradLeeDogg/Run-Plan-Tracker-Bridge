# Run Plan Tracker

A single-file PWA for tracking a run block. Add it to the iPhone home screen and
it behaves like an app: opens offline, no account, no network calls, no
analytics. Everything is stored in `localStorage` on the device.

It ships with the 12-week block below, and the plan is editable in the app —
distances, which day each run falls on, and how many weeks there are.

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

Stats also breaks the block down **week by week** and **month by month** —
planned against done, run count, total time, average pace and average heart
rate. Months are calendar months taken from the date each run happened, so a
week straddling the turn of a month is split across both.

## Changing the plan

- **A run you cannot make.** Tap it and pick another day of that week. Moving a
  run is limited to its own week on purpose: weekly totals stay meaningful and
  the volume guards keep working. To move further, shift the whole week.
- **Distances, and how the week is shaped.** Plan → *Edit this week* sets each
  run's distance and day, adds or removes runs, and marks the week normal,
  recovery or peak. Week totals are always summed from the runs, never stored,
  so they cannot drift.
- **Length of the block.** *Add a week at the end* extends it, copying the shape
  of the final week; *Delete week* shortens it and pulls the later weeks back so
  the calendar stays tight.
- **Slipping the whole thing.** The per-week controls move that week and
  everything after it a week earlier or later.

Editing the plan never disturbs runs you have already logged — see below.

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

### How logs stay attached

Every run carries an id, and logs are stored against that id rather than against
the run's position in the plan. This is what makes the plan safe to edit:
deleting week 2 shifts every later week up one, and under position-based keying
each log after the edit would silently re-point to a different run.

`DEFAULT_PLAN` and `WEEK1_MONDAY` at the top of the script are only the starting
point, used on a first run and by **Erase logs and reset the plan**. After that
the plan lives in storage next to the logs, and an exported backup carries both.

Backups written by the previous version — logs keyed `"week-day"`, with a
separate `weekStarts` array — are migrated on load and on restore, so older
exports still work.

## Getting runs in without typing

Bridge already syncs the Galaxy Watch into Apple Health, and this app can read
what lands there — via a Shortcut for routine use, or Health's own "Export All
Health Data" as the always-works fallback. Both give distance, duration and
average heart rate, and therefore pace and the VO<sub>2</sub> estimate.

Neither gives splits. Apple Health has nowhere to keep per-kilometre splits for
a third-party workout, so they are lost before this app sees the data. That is
the one thing the watch app below exists to provide.

See **[APPLE-HEALTH.md](APPLE-HEALTH.md)** for both recipes. The full export also
backfills everything you have already run: pre-plan runs are kept as **history**,
counted in the monthly table, VO<sub>2</sub> trend and average heart rate, but never
against a plan week.

## The watch half

`samsung-watch-sync/` holds the other end of this: a Wear OS app that records a
run with GPS and heart rate, derives the splits, writes a TCX and serves it over
local Wi-Fi on port 8787. Samsung Health has no per-run file export, so a custom
watch app is the only way to get splits off a Galaxy Watch at all.

The flow is two steps, because a page served over HTTPS cannot fetch `http://`
from the watch: Safari to `http://<watch-ip>:8787/runs`, save the `.tcx` to
Files, then **Import from a file** here.

Its TCX output and this app's importer are verified against each other — the
Kotlin that computes splits has no Android imports, so it runs on a plain JVM
and its output can be fed straight to the parser in `index.html`. See
`samsung-watch-sync/README.md`.

## Files

```
index.html          the whole app — markup, styles, logic
manifest.json       PWA metadata
sw.js               service worker, offline shell cache
icon.svg            source icon
icon-180.png        apple-touch-icon (iOS ignores SVG here)
icon-512.png        manifest icon
.nojekyll           serve files verbatim, no Jekyll pass
samsung-watch-sync/ the Wear OS recorder and its iOS/Shortcuts clients
```

Vanilla JS, no build step, no dependencies, no CDN.
