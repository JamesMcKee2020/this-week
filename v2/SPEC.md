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
- **Must do this week.** A flag, not a day. Flagged tasks sit at the top of
  the Now screen until done or deliberately given a day. When the
  time/effort filter excludes a flagged task, it stays visible in a
  "must do but doesn't fit" notice - necessary work never quietly
  disappears because tonight called for something small.
- **Due date vs planned day.** A due date is the hard deadline; a planned
  day is a rough plan. Changing the plan never moves the deadline. Due
  dates show as pills (amber within 3 days, red past).
- **Completion is a record.** Marking done asks when (defaults to today)
  and an optional outcome. The Done screen is a dated log - no relying on
  memory. Chores now write a done record and spawn their next parked
  occurrence (v1 silently re-parked the same task).
- **EI activity log bridge.** Completing a task tagged with "job" offers to
  write one entry into the Job tracker app's EI activity log
  (`localStorage['jobTrackerActivitiesV1']` - same site, same browser
  storage, no network). Entries are prefixed `tw-`. The owner picks the
  activity type (the same list the Job tracker already offers), date and
  outcome before anything is written - nothing is inferred from the tag
  alone. Caveat: this only works where both apps share browser storage
  (tested: two tabs of the same desktop browser). Home-screen icons on a
  phone may keep separate storage; if the entry does not appear in the Job
  tracker, it still lives in this app's Done log, and the Done screen can
  download a job-search CSV with the same columns as the Job tracker's own
  CSV export.
- **Paste a list.** The inbox accepts pasted lines (one task per line,
  `#tags` pulled out). This is the photo-to-task bridge in this build: a
  standalone, fully-offline page cannot read photos (recognition needs a
  service, and this app's data never leaves the device), so the photo goes
  to Instinct in chat, and the draft list it returns is pasted here for
  review - every line is checkable before and after import.

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
download, restore merges by id, not-before parking, recurring chores,
finite courses with countdown, week planning Sunday-Saturday with day
totals, explicit move controls.

## Known limits (v2)

- Same-device only; moving devices = export, restore.
- The EI bridge depends on shared browser storage (see above); phone home-screen storage separation is untested.
- Photo-to-task is a chat + paste flow, not in-app recognition.
- Week view is the current week only.
