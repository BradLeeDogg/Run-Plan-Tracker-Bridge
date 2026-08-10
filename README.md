# Run Plan Tracker

A single-file PWA for tracking a run block. Add it to the iPhone home screen and
it behaves like an app: opens offline, no account, no network calls, no
analytics. Everything is stored in `localStorage` on the device.

It ships with **Marathon Build — May 29, 2027**: 42 weeks, 1,630 km, from
Monday 10 August 2026 to the race on Saturday 29 May 2027. The plan is editable
in the app — distances, which day each run falls on, and how many weeks there
are.

| Phase | Weeks | Focus |
|---|---|---|
| 1 Base Building | 1–16 | All easy. No speed work. |
| 2 Winter Base | 17–29 | Cross-training mandatory. One tempo per week. |
| 3 Marathon Specific | 30–39 | Long runs and marathon-pace work. |
| 4 Taper | 40–42 | Volume down. Long run moves to Saturday. |

Tue / Thu / Sun, Sunday long, weeks start Monday — except the taper, where the
long run moves to Saturday. Sessions are typed: easy, long, tempo and race.
Strides begin week 8, the Thursday tempo begins week 22, marathon-pace finishes
begin week 35. Week 33 is a half-marathon tune-up, week 39 the 30 km peak.

Long runs carry a 3½-hour time cap, and weeks that came with a note keep it —
*"Do not make up missed distance"*, *"See a doctor before Phase 2"*, *"Taper
will feel wrong — this is normal"*.

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

Per run: completed or skipped, actual distance and duration, average heart rate,
calories, effort (easy / moderate / hard), and any pain with a body-area tag and
a note. Pace and the VO<sub>2</sub> estimate are derived and shown as you type.

Calories are read from whatever you import — TCX laps, Apple Health's
`totalEnergyBurned`, or a calories column in JSON or CSV — and can be typed in
alongside heart rate.

**Strength sessions** — twice a week, 20–25 minutes, after a run. Tuesday and
Thursday by default, after the two shorter runs, leaving the Sunday long run
alone. Five exercises, each tickable:

| | |
|---|---|
| Calf raises | 3 × 12, straight-leg and bent-knee — progress to single-leg, then load |
| Single-leg glute bridges | 3 × 10 per side |
| Split squats or step-ups | 3 × 8 per side |
| Side-lying hip abduction | 3 × 12, or banded lateral walks |
| Plank + side plank | 3 × 30–45 seconds |

Strength never counts towards weekly distance, so it cannot trip the
over-volume warning or argue with a recovery week. It has its own totals in
Stats: sessions done against planned, and hours.

**Strides** — 4–6 × 20 seconds at the end of the Tuesday run, from week 9.
Relaxed fast, not a sprint, with a full walking recovery between each.

They are conditional on the first eight weeks having felt genuinely
comfortable, and that condition is asked rather than assumed. Nothing appears
until week 9, when the app puts the question and takes either answer. Declining
is recorded so it stops asking; the decision can be reversed from Stats at any
time, in either direction.

Per week: actual against planned with a percentage, rolling 4-week volume, and
a streak of consecutive completed runs.

Stats breaks the block down **week by week** — planned against done, run count,
total time, average pace, heart rate and calories, with the current week
highlighted.

Tapping a column in the weekly chart fills in a readout below it — that week's
distance against plan, run count, time, pace, heart rate and calories. There is
no hover on a phone, so the numbers behind a bar need somewhere to land.

**Browse by time** drills down: all time → a year → a month → a week → the runs
themselves. Every level shows full stats for that period — distance, runs, time,
pace, heart rate, calories, longest run and best VO<sub>2</sub> — and a
breadcrumb walks back up. Tapping a run opens it: the log sheet if it belongs to
the plan, a detail view if it is history.

Its weeks run Monday to Sunday on the calendar, independent of the plan's weeks,
so runs from before the block still land in one. A week that straddles two
months is shown whole and flagged, so the row and the week it opens always
agree.

Tapping a logged run **opens it to be read** — distance, time, pace, heart
rate, calories, VO<sub>2</sub>, effort, strides, and every split with the
fastest and slowest marked. Editing is a second, deliberate step behind an
**Edit this run** button, so glancing at a run cannot end in an accidental
overwrite. A run with nothing logged yet goes straight to the form, since there
is nothing to look at.

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
counted in the time browser, VO<sub>2</sub> trend and average heart rate, but never
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
