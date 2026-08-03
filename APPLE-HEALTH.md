# Getting runs in from Apple Health

Bridge syncs the Galaxy Watch into Apple Health. This app can read what lands
there, so runs do not have to be typed in.

**What you get:** date, distance, duration, average heart rate — and from those,
pace and the VO<sub>2</sub> estimate.

**What you do not get: splits.** Apple Health has nowhere to store per-kilometre
splits for a third-party workout, so they are gone before this app ever sees the
data. No import route recovers them. Splits come from the watch app in
`samsung-watch-sync/`, or from reading them off Samsung Health and typing them
into the run's Splits box.

There are two routes in. Try the Shortcut first; it is faster and produces a
small file. The full export always works but is heavy.

---

## Route 1 — a Shortcut (fast, for routine use)

Open the **Shortcuts** app and build this. It reads recent runs out of Health,
turns them into JSON, and saves the result to Files.

1. **Find Health Samples**
   - Type: **Workouts** — or **Find Workouts**, if your iOS lists it as its own
     action
   - Filter: *Workout Type* **is** *Running*
   - Filter: *Start Date* **is in the last** *7 days*
   - Sort by *Start Date*
2. **Repeat with Each** over the result
3. Inside the loop, **Text**, containing exactly one line:
   ```
   {"date":"[Start Date]","distance":[Distance],"duration":[Duration],"heartRate":[Average Heart Rate]},
   ```
   Each bracketed item is a *magic variable* — tap the Repeat Item and pick the
   property, do not type the name.
4. After the loop, **Combine Text** with *New Lines*
5. **Text**: `[` then the combined text then `]`
6. **Save File**, destination **Files**, and let it ask where — name it
   `runs.json`

Run it, save the file, then in the tracker: **Import from a file** → *Choose a
file* → pick `runs.json`.

### If the numbers come out wrong

Units are inferred, and the import screen shows you every value before anything
is written, so check it there rather than trusting it blind.

- Distance is read as kilometres under 200 and metres above it. If Shortcuts
  hands over metres, a 7 km run arrives as 7000 and is read correctly.
- Duration is read as minutes under 600 and seconds above it. A 35-minute run
  can therefore be given as `35` or as `2100`; both land on 2100 seconds. The
  ambiguity only bites for a run of 600+ minutes, which is not a training run.
- If a field arrives empty, drop it from the Text action rather than sending an
  empty string.

Nothing is written until you confirm, and already-logged slots are flagged and
left unticked.

### Which actions exist on your iOS

Health actions in Shortcuts have moved around across iOS versions, particularly
for whole *workouts* as opposed to raw samples like heart rate. If you cannot
find a workout action at all, use route 2 — it does not depend on any of this.

---

## Route 2 — the full Health export (always works)

1. **Health** app → your **profile picture**, top right → **Export All Health
   Data** → **Export**
2. It produces `export.zip`. Save it to Files.
3. In Files, tap the zip to unpack it. Inside is **`export.xml`**.
4. In the tracker: **Import from a file** → *Choose a file* → pick `export.xml`

The app scans it for running workouts and ignores everything else.

**It is a very big file.** The export holds your entire Health history — every
heart rate reading the phone has ever taken — and routinely passes a gigabyte.
Files that size cannot be read into memory in one go; the tab dies first. So the
app reads it in slices and keeps only the `<Workout>` blocks, showing progress
as it goes. A 1.3 GB export has been tested end to end.

Keep the app open while it reads. Generating the export in Health is itself
slow, often several minutes. Fine occasionally; not something to do weekly —
that is what the Shortcut is for.

Both older and newer export layouts are handled — Apple moved distance from an
attribute on the workout into a nested `WorkoutStatistics` element, and both
shapes are still out in the wild.

---

## Backfilling everything you have already run

Route 2 is also how you get your history in. The export contains every running
workout Health has ever held, and the importer takes the lot in one go.

Runs from before the plan began have no slot to attach to — the plan only has
36 of them, on fixed dates — so they are kept as **history** instead. The import
screen labels them, and Stats grows a **History** card listing them with a total.

History counts towards:

- the month-by-month table
- the VO<sub>2</sub> max trend
- average heart rate

History deliberately does **not** count towards:

- any plan week's actual distance
- the 110% over-volume warning
- the recovery-week check

A run from last year should not make this week look overshot, and should not
argue with a recovery week. Adherence is measured against the plan; history is
just a record of running.

Re-importing an export that overlaps one you already loaded is fine. Anything
already held for the same date and distance is flagged as a duplicate and
arrives unticked, so the usual case — exporting again a month later — does not
double anything up. You can remove any single history entry, or all of them, in
the History card; runs logged against the plan are untouched by that.

## What this does not solve

The two-step shuffle. There is no one-tap route from Health into this app,
because a page served over HTTPS cannot read HealthKit — there is no web API for
it, on any browser, and Safari is no exception. A file is the only way across.

Splits, as above. If they matter to you, that is the argument for building the
watch app; it is the only route that produces them.
