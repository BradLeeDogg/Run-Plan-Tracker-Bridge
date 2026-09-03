# Run Plan Tracker

A single-file PWA for tracking a run block. Add it to the iPhone home screen and
it behaves like an app: opens offline, no account, no network calls, no
analytics. Everything is stored in `localStorage` on the device.

It ships with **Marathon Build — May 30, 2027**: 42 weeks, 1,630 km, from
Monday 10 August 2026 to the race on Sunday 30 May 2027. The plan is editable
in the app — distances, which day each run falls on, and how many weeks there
are.

| Phase | Weeks | Focus |
|---|---|---|
| 1 Base Building | 1–16 | All easy. No speed work. |
| 2 Winter Base | 17–29 | Cross-training mandatory. One tempo per week. |
| 3 Marathon Specific | 30–39 | Long runs and marathon-pace work. |
| 4 Taper | 40–42 | Volume down. Long runs rehearse race day. |

Tue / Thu / Sun, Sunday long, weeks start Monday. Sessions are typed: easy,
long, tempo and race.
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
calories, effort (easy / moderate / hard), an RPE out of ten, how it felt, which
shoes, and any pain with a body-area tag and a note. Pace and the VO<sub>2</sub> estimate are derived and shown as you type.

Distances take **two decimal places**, entered with either separator — `9.42`
or `9,42`. iOS puts the decimal separator its *locale* uses on the numeric
keypad, so a phone set to most of Europe offers a comma and no full stop at all,
and an `<input type="number">` accepts only a full stop: the HTML value
sanitiser discards anything else and the field reads back empty. Every field
that takes a decimal is therefore a plain text input with a decimal keypad, and
both separators are read. Whole-number fields keep `type="number"`, which has no
such problem.

Two decimals, because a watch says 9.42 km and not 9.4,
and a number you typed should be the number you are shown. Trailing zeros are
trimmed, so 9.42, 9.4 and 9 all read as themselves. Planned distances take them
too; only the bulk *scale* operation rounds, to the half-kilometre, since a plan
reading 12.37 km is a plan nobody follows.

Duration is entered as **hours, minutes and seconds** — a three-hour long run is
3 h 12, not 192 minutes. The hours box is left blank for anything under an hour.
Everything is stored as seconds, so old logs read back into the three boxes
unchanged.

Calories are read from whatever you import — TCX laps, Apple Health's
`totalEnergyBurned`, or a calories column in JSON or CSV — and can be typed in
alongside heart rate.

**Notes and conditions** — weather, temperature, surface and route on each run,
plus a free-text note. None of it reaches the plan maths; it is there so a slow
week still has an explanation attached to it a year from now. The note field
stays available on a skipped run, since why it did not happen is exactly the
kind of thing worth a sentence. Conditions and the first line of the note show
on the run row, and everything is searchable — see **Browse by time** below.

Both survive an import: a TCX carries none of it, so what was written by hand is
kept rather than overwritten.

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

**Strides** — 4–6 × 20 seconds at the end of the Tuesday run, from **week 8**,
where the supplied plan starts them. Relaxed fast, not a sprint, with a full
walking recovery between each.

They are conditional on the weeks before them having felt genuinely
comfortable, and that condition is asked rather than assumed. Nothing appears
until the first strides week, when the app puts the question and takes either
answer. Declining is recorded so it stops asking; the decision can be reversed
from Stats at any time, in either direction. The trigger week is read from the
plan, so changing the plan moves it.

**Cross-training** — Wednesday and Friday, between the runs. Optional through
the base phase and **required from phase 2**, which is the plan's own wording.
Pick what you did — bike, swim, row, elliptical or other — and how long. Like
strength, it adds no running distance and cannot trip the volume guards.

**Morning readings** — resting heart rate, hours slept and body weight, one
number each. The card asks at the top of Today on a morning with nothing
recorded and steps down the page once answered, because a daily prompt that
stays put after it has been answered stops being read.

Resting rate carries most of the value. A sustained rise over *your own*
baseline — the mean of the last 14 mornings, once there are at least five of
them — is the oldest cheap sign that the build is outrunning the recovery, and
it is flagged at 5 bpm with the plain warning that one morning is noise. It also
anchors the VO<sub>2</sub> estimate; see below.

**Session RPE and training load** — every guard in this app counts kilometres,
which means a 14 km tempo and a 14 km easy run are indistinguishable to all of
them, and from week 22 the plan has you doing one of each most weeks. Rating a
session 1–10 and multiplying by its minutes gives a number that works for a run,
a bike hour and twenty minutes of calf raises alike, so total load can be seen
rather than inferred from mileage. Sessions logged before RPE existed fall back
to a stand-in from their effort rating and are marked as estimated wherever the
figures appear.

Stats shows the last 7 days against the weekly average of the last 28, load per
week, and the split across running, cross-training and strength — a split
distance cannot show, since it counts only the first of the three. The ratio
stays blank until there are four weeks behind it rather than reporting a number
that only means the week is being compared with itself, and it is framed as a
trend to read rather than a rule: the method has had fair criticism, and the
sound part of it is what the plan's own recovery weeks already say.

Paired with it, a separate **how it felt** tap. The gap between the two is the
signal — hard that felt strong is fitness arriving, easy that felt rough is
fatigue banking up.

**Fuelling practice** — *"Begin fueling practice: 30-60g carb/hr"* is the only
numeric prescription in the plan that had nowhere to be answered. Long runs of
16 km and over from phase 3 now carry a carbohydrate field: you enter the total
grams, because gels are countable and a rate is not, and the app derives g/hr
from the run's own duration and marks it against the plan's band. A gut-tolerance
tap goes with it. Read from the plan's phase and distance rather than a fixed
week, so reshaping the plan moves which runs ask.

**See a doctor before Phase 2** — the plan's other untracked instruction, said
once in one week's note and then never again, which is exactly how a sentence
like that gets missed. A banner appears four weeks before phase 2 starts and
escalates once phase 2 has begun without it; ticking it records the date and an
optional note, and it shows as done in Readiness. The week comes from the plan's
phase field, so it moves with edits.

**Shoes** — several pairs can be in service at once, each with its own name and
mileage. The plan replaces at 700 km, with a warning at 85% and again past 700,
per pair.

Mileage keys on the pair a run names, not on dates. With one pair the service
window told you everything; with a rotation two windows overlap and the dates
decide nothing. Runs logged before rotation existed were assigned to their pair
once, at load, using the old window rule — so every existing total reads exactly
as it did rather than resetting to zero on an update. Runs that end up naming no
pair are counted and surfaced rather than having their distance quietly vanish
from every total.

The log sheet only offers a picker when more than one pair is in service, and
one pair is marked default so a single-pair setup carries on needing no thought.

Per week: actual against planned with a percentage, rolling 4-week volume, and
a streak of consecutive completed runs.

The VO<sub>2</sub> estimate scales by heart-rate reserve, so it needs a resting
rate. It used to read one global setting for every run, which meant the whole
historical trend was scaled by whatever that setting said *today* — correcting a
stale 55 down to 48 silently lifted every VO<sub>2</sub> the app had ever shown.
Over a 42-week base build the resting rate is supposed to fall, so the value now
travels with the run: each estimate uses the most recent morning reading on or
before that run's date, falling back to the setting when there is none that old.

Stats is in four sections — **Training**, **Body**, **Records** and
**History** — because fourteen cards on one scroll is not a page anybody reads
to the end. Training is how the block is going, Body is what you are doing,
Records is what you have achieved, History is what is stored.

Stats breaks the block down **week by week** — planned against done, run count,
total time, average pace, heart rate and calories, with the current week
highlighted.

Tapping a column in the weekly chart fills in a readout below it — that week's
distance against plan, run count, time, pace, heart rate and calories. There is
no hover on a phone, so the numbers behind a bar need somewhere to land.

**Race day** gets three panels of its own.

A **countdown** sits at the top of Today: days and weeks to the race, the race
distance, and how much of the block is behind you. Race day is read from the
plan — the last run typed `race`, not the constant at the top of the file — so
editing the plan moves it.

**Readiness** answers whether the block is actually being run. It measures only
against runs that have already come due, so weeks still ahead do not count
against you, and it leads on volume rather than run count: three half-length
runs are three ticks and not the week that was planned. The verdict runs from
*On plan* through *Behind* to *Well behind*, with *Ahead of plan* treated as the
warning it is. Underneath, a check specific to the phase you are in — no hard
runs during base building, cross-training done during phase 2, long runs done
during phase 3.

**Race day prediction** projects a finish time two independent ways. The
headline is [Riegel][riegel] — `T₂ = T₁ × (D₂/D₁)^1.06` — from your longest
genuine effort, because a marathon extrapolated from a single fast kilometre is
arithmetic rather than evidence. Beside it, the same target worked out from the
VO<sub>2</sub> estimate: the app's own oxygen-cost equation run backwards, at
the fraction of maximum that is holdable for a race that long ([Daniels'][dan]
curve — about 81% for three and a half hours). Every distance you have a best
for gets its own row, and the spread between the fastest and slowest read is
shown rather than hidden, because where two methods disagree the spread is the
honest answer. A confidence badge grades the projection on how far the longest
effort behind it actually went.

[riegel]: https://en.wikipedia.org/wiki/Peter_Riegel
[dan]: https://en.wikipedia.org/wiki/Jack_Daniels_(coach)

**Personal bests** builds a ladder — 1 km, 1 mile, 2 miles, 5 km, 10 km, half,
marathon — from two kinds of evidence: a whole run at about that distance, and
the fastest stretch *inside* a longer one, so a quick 5 km buried in a Sunday
long run still counts. Candidates are ranked on pace rather than elapsed time,
since their distances are not identical and ranking on time would hand the
record to whichever was shortest. Every row says the exact distance its time
covers and opens the run behind it. Below the ladder: longest run, biggest week
and month, most runs in a week, best VO<sub>2</sub>.

The ladder is in real race distances — a mile is 1.609 km, the half 21.0975,
the marathon 42.195 — none of which land on a kilometre mark. For those, the
pace inside each kilometre is taken as even, which makes cumulative time a
straight line between marks and the window time a piecewise-linear function of
where the window starts. Such a function takes its minimum at a corner, and the
corners are the kilometre marks plus the points one window-width before them, so
checking those two families finds the true best rather than sampling along the
run and hoping to land near it. On a whole number of kilometres it reduces to
the exact sum of consecutive splits, which is checked against a brute-force
slide over randomised runs.

Whole-run candidates get half a percent of slack underneath the distance. That
is for GPS, not generosity: a measured half marathon comes back from a watch as
21.05 or 21.08 km as often as not, and refusing to count a real race because the
satellites cut the tangents would be precision in the wrong direction. Nothing
is overstated by it — the row reports the distance the watch recorded, and
ranking is on pace, so a slightly short run gains nothing.

**Browse by time** drills down: all time → a year → a month → a week → the runs
themselves. A **search box** sits above it: type anything and it matches across
notes, route, weather, surface, effort, where it hurt and the date, all terms
having to match. Only the results redraw, so the keyboard stays put mid-word. Every level shows full stats for that period — distance, runs, time,
pace, heart rate, calories, longest run and best VO<sub>2</sub> — and a
breadcrumb walks back up. Tapping a run opens it: the log sheet if it belongs to
the plan, a detail view if it is history.

Its weeks run Monday to Sunday on the calendar, independent of the plan's weeks,
so runs from before the block still land in one. A week that straddles two
months is shown whole and flagged, so the row and the week it opens always
agree.

Tapping a logged run **opens it to be read** — distance, time, pace, heart
rate, calories, VO<sub>2</sub>, effort, strides, conditions, your note, and
every split with the fastest and slowest marked. Editing is a second,
deliberate step behind an **Edit this run** button, so glancing at a run cannot
end in an accidental overwrite. A run with nothing logged yet goes straight to
the form, since there is nothing to look at.

**The last part-kilometre** is shown too. A 5.4 km run has five splits and then
400 m, and that tail used to vanish: the watch reports it as a trailing lap, and
this app's split list is whole kilometres. It is now worked out from the
distance, the duration and the splits — so it needs nothing stored and works for
splits typed by hand as well as imported ones — and drawn dashed, apart from the
splits. Kept out of the list on purpose: a 400 m split is not comparable with a
1 km one, and letting it in would hand "fastest split" to whatever fraction the
run happened to finish on.

## Logging a run

The sheet opens short: status, distance and time, pain, Save. Everything
else — heart rate, splits, effort, RPE, how it felt, shoes, conditions, the
note — is behind **Add detail**, which opens itself when the run already
carries any of it, or when the plan is asking that particular run for strides
or fuelling. Pain stays out of the fold on purpose: something you bury is
something you stop logging, and it is the one field that exists to catch a
pattern. Save is pinned to the bottom of the sheet however long it gets.

A skipped run shows pain and the note in plain sight instead, since why it did
not happen is the whole of what is worth recording.

Every question the app asks is its own — no browser `confirm()`, `prompt()` or
`alert()` anywhere. They are unstyleable, and in a home-screen app visibly not
part of it. The replacements can also ask for more than one thing at a time, so
naming a pair of shoes and saying what they have already run is one dialog
rather than two prompts back to back, and a rejected value keeps the panel open
with what you typed still in it.

## Changing the plan

- **A run you cannot make.** Tap it and pick another day of that week. Moving a
  run is limited to its own week on purpose: weekly totals stay meaningful and
  the volume guards keep working. To move further, shift the whole week.
- **Distances, and how the week is shaped.** Plan → *Edit this week* sets each
  run's distance and day, adds or removes runs, and marks the week normal,
  recovery or peak. Week totals are always summed from the runs, never stored,
  so they cannot drift.
- **Switching plans.** When the plan in the app is newer than the one on your
  device, it offers to load it — and runs you have already logged **come with
  you**. Logs key on run ids, which is what makes editing a plan safe; replacing
  one gives every run a new id, so the old behaviour swept every log into
  history. Safe, but it emptied the week table, the streak and every adherence
  figure of a block you had actually run.

  A date is the thing that survives a plan change. A run done on 25 August is
  still a run done on 25 August whatever the new plan calls that day, so logs
  are re-pointed onto whichever run in the new plan falls on their date, within
  three days. Strength and cross-training sessions move the same way. Only what
  genuinely has no home goes to history, and you are told the split — how many
  move across, how many do not — before anything happens.

  If an earlier plan change already stranded runs in history, Stats offers to
  **re-attach** them: the same matcher, run against the plan you are on now.

- **Undo.** A plan change, a restore, a reset or a history wipe takes a snapshot
  first, and Stats offers one step back for as long as it is there. If storage
  is too tight to hold both, the snapshot is dropped rather than the save
  failing — losing the ability to go back beats losing the thing itself.
- **Length of the block.** *Add a week at the end* extends it, copying the shape
  of the final week; *Delete week* shortens it and pulls the later weeks back so
  the calendar stays tight.

- **Many weeks at once.** Plan → *Reshape several weeks* does over a range what
  editing does to one: scale the distances by a percentage, or move a weekday —
  every long run to Saturday, say — across weeks 12 to 30 in a single action. It
  edits in place, so run ids never change and not one log is touched. Distances
  round to the half-kilometre, because a plan reading 12.37 km is a plan nobody
  follows, and a week that already has a run on the target day is left alone
  rather than made to hold two, with the count reported back.

- **A plan from somewhere else.** Plan → *Paste a plan* or *From a file* reads
  one in and swaps it for the block you are on, keeping every run the dates can
  account for. Three shapes are accepted, because a plan arrives in whatever
  form its author had to hand:

  | Shape | What it needs |
  |---|---|
  | CSV or TSV | A `date` column and a `km` column, one row per run. `type`, `note`, `tag`, `phase`, `cap`, `tempo`, `mp` and `strides` are read if present. |
  | JSON plan | The array this app stores, or a whole backup with a `plan` in it. |
  | JSON seed | The `{s, t, p, n, r}` shape at the top of `index.html`. |

  Columns are found by header name, so their order does not matter and a
  coach's extra columns are ignored. Rows missing a date or a distance are
  skipped and counted rather than being fatal. Week starts snap to the Monday,
  since the whole app reckons days as offsets from one. Nothing is written until
  the file has parsed, and a refusal says which part failed.

- **Putting the shipped plan back.** *Restore Marathon Build* returns the
  original 42 weeks without erasing a run. It used to be that **Erase logs and
  reset the plan** was the only route back, and it took everything with it.
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
