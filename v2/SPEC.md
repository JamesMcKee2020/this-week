# This Week v2 - Spec

A phone-first personal task app. One self-contained web page, no server, no
account, no build step. Everything lives on the device it runs on.

v2 is a redesign driven by use: the front door is **capture**, not the
calendar. It lives in the `v2/` folder so it can be tried side by side with
v1 - both read and write the same on-device data (same IndexedDB database,
same record shape; v2 only *adds* optional fields, which v1 ignores but
preserves).

## What changed from v1

- **Intake first.** The opening screen is a capture box (text, photo, or a
  pasted list) above the unsorted Inbox. The week calendar is a collapsed
  section inside the Now screen - a view of what you assigned to days, not
  the front door.
- **Tags.** Any number of lowercase tags per task ("groceries",
  "home-project", "job-search"). The Tags screen retrieves everything open
  under a tag, independent of dates. The `groceries` tag renders as a
  tap-to-cross-off checklist. Tags are separate from priority and from
  coloured streams (streams still exist, optional).
- **Mental effort.** Each task can be rated Mindless / Some thought /
  Deep focus. Combined with rough size, the Now screen answers "I have an
  hour and want something mindless": pick time + effort, get the tasks that
  fit. Unrated tasks are listed separately so nothing hides.
- **Must do this week.** A flag stamped with the week it was set for
  (`mustWeekOf`), not a floating forever-flag. Flagged tasks sit at the top
  of the Now screen until done or deliberately given a day. When the week
  rolls over, unfinished musts move to a "Carried over from last week"
  section that asks for a fresh decision - "Still must this week" re-stamps
  it, "Let it float" clears the flag. When the time/effort filter excludes
  a flagged task, it stays visible in a "must do but doesn't fit" notice -
  necessary work never quietly disappears because tonight called for
  something small.
- **Due date vs planned day.** A due date is the hard deadline; a planned
  day is a rough plan. Changing the plan never moves the deadline. Due
  dates show as pills (amber within 3 days, red past), and anything due
  within 3 days or passed is named in a notice at the top of the Now
  screen, so a hard deadline never depends on remembering to open the
  collapsed week view.
- **Completion is a record.** Marking done asks when (defaults to today)
  and an optional outcome. The Done screen is a dated log - no relying on
  memory. Chores now write a done record and spawn their next parked
  occurrence (v1 silently re-parked the same task).
- **EI activity log bridge.** Completing a task tagged with "job" offers to
  write one entry into the Job tracker app's EI activity log
  (`localStorage['jobTrackerActivitiesV1']` - same site, same browser
  storage, no network). Entries are prefixed `tw-`. The owner picks the
  activity type (the same list the Job tracker already offers), date and
  outcome before anything is written - the type starts unchosen on purpose,
  and saving with the EI box ticked but no type picked is refused rather
  than guessed. A task that already has an EI entry says so and cannot be
  logged twice; the save button disables on tap so a double-tap cannot
  double-write. Caveat: this only works where both apps share browser
  storage (tested: two tabs of the same desktop browser). Home-screen icons
  on a phone may keep separate storage; if the entry does not appear in the
  Job tracker, it still lives in this app's Done log, and the Done screen
  can download a job-search CSV with the same columns as the Job tracker's
  own CSV export.
- **Paste a list, with review.** The inbox accepts pasted lines (one task
  per line, `#tags` pulled out) and shows them for review before adding -
  lines ending in a colon start unticked as likely headings. Pasting from
  Notes brings the words only: ticks, crossings-out and formatting do not
  survive the clipboard, so a finished item looks like any other line and
  the review step is where it gets unticked.
- **Photos, screenshots, and pasted images.** Photo buttons accept the
  camera, the photo library, or the file picker (where screenshots saved to
  Files live). An image copied anywhere can be pasted straight onto the
  page - it becomes a new photo task, ready to name. Clipboard image paste
  is verified in desktop Chrome; phone browser support for pasting images
  is expected but untested on an actual iPhone.
- **Restore asks first.** Restoring a backup opens a choice instead of
  acting silently: merge (backup items are added or updated, device-only
  items stay) or replace (the device is wiped to match the backup), with
  counts from both sides shown before anything changes.

## Task record (v2)

```json
{
  "id": "t…", "title": "…", "streamId": "t…|null",
  "kind": "task|chore|course",
  "size": 60,
  "col": "inbox|now|next|later",
  "order": 1761…,
  "tags": ["groceries"],
  "effort": "low|med|high|null",
  "dueDate": "YYYY-MM-DD|null",
  "mustWeek": false,
  "mustWeekOf": "YYYY-MM-DD|null",  // week start the must flag was set for
  "notBefore": "YYYY-MM-DD|null",
  "notes": "…",
  "day": "YYYY-MM-DD|null",
  "done": false,
  "doneDate": "YYYY-MM-DD|null",
  "outcome": "…",
  "ei": {"id": "tw-…", "type": "Application submitted"} | null,
  "photos": ["p…"],
  "course": {"totalText": "10-12 hours", "totalMin": 660, "loggedMin": 60} | null,
  "chore": {"everyN": 2, "unit": "days|weeks|months", "lastDone": null} | null,
  "created": "YYYY-MM-DD"
}
```

v1 tasks load unchanged: missing fields get defaults on read (doneDate
stays null - those completions show as "date not recorded"). Backups are
`version: 2` and v2 restores v1 backups.

## Unchanged from v1

Storage (IndexedDB `this-week`, stores tasks/streams/photos/meta, in-memory
fallback with a visible warning), photo downscale to ~1200px JPEG, no
network requests, no accounts/sync/analytics, backup = full-state JSON
download, restore = merge or replace by explicit choice, not-before
parking, recurring chores,
finite courses with countdown, week planning Sunday-Saturday with day
totals, explicit move controls.

## Known limits (v2)

- Same-device only; moving devices = export, restore.
- The EI bridge depends on shared browser storage (see above); phone home-screen storage separation is untested.
- Clipboard image paste and the photo/file picker menu are verified on desktop only; exact iPhone Safari behaviour (which sources the picker offers, whether paste prompts for permission) is untested.
- A standalone, fully-offline page cannot read photo contents (recognition needs a service, and this app's data never leaves the device); a photo attaches to a task as-is.
- Week view is the current week only.
