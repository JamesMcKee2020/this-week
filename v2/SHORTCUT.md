# "Send to This Week" - iPhone Shortcut

Turns a photo, screenshot or Note into a list of tasks. The iPhone reads the
text itself (Apple's on-device text recognition - the same thing behind
Live Text), so the app still makes no network requests and the image never
leaves the phone. The text lands in the app's normal paste review: every
line is shown, ticked or not, before anything becomes a task.

## Build it (about 5 minutes, once)

Action names below are as they appear in the Shortcuts app's search box on
recent iOS; if one is worded slightly differently, search for the key word.

1. Open **Shortcuts**, tap **+**, name it **Send to This Week**.
2. Tap the **(i)** / settings button: turn on **Show in Share Sheet**. Set
   the share-sheet types to **Images, Text, Rich Text** (and **Files**, for
   screenshots saved there). Set "If there's no input" to
   **Get Clipboard** - then running it from the Shortcuts app reads
   whatever you last copied (a screenshot or text).
3. Add **Get Images from Input** (input: Shortcut Input).
4. Add **If** - *Images* - **has any value**.
   - Inside the If: add **Extract Text from Image** (input: Images), then
     **Set Variable** `Captured` to *Text from Image*.
   - In **Otherwise**: add **Get Text from Input** (input: Shortcut Input),
     then **Set Variable** `Captured` to *Text*.
5. After **End If**: add **Copy to Clipboard** (`Captured`).
6. Add **URL Encode** (`Captured`).
7. Add **Open URLs** with:
   `https://jamesmckee2020.github.io/this-week/v2/#capture=` followed by the
   *URL Encoded Text* variable (no space between them).

## Use it

Share a photo, screenshot or Note -> **Send to This Week**. The app opens on
the Inbox with the lines listed for review: untick junk, mark group labels
as headings (they become #tags), then **Add the ticked tasks**. Nothing is
added until you tap that.

## The home-screen catch (untested - please check)

On iPhone, an app added to the home screen can keep its own storage,
separate from Safari. Step 7 opens Safari, so if you normally use the
home-screen icon, the tasks may land in Safari's copy instead of yours.

Test once: run the Shortcut, add a task, then open the home-screen icon.

- **Task is there** - storage is shared; keep step 7.
- **Task is missing** - delete it in Safari, remove step 7 from the
  Shortcut, and use it this way instead: run the Shortcut (it copies the
  text), open the home-screen app, tap **Paste a list**, long-press the box,
  **Paste**. The review opens by itself as soon as text is pasted.

## No-Shortcut fallback

Open the photo, long-press its text, **Select All**, **Copy**, then paste
into **Paste a list**. Same review step.

## How it works

The text travels in the address's `#fragment`, which browsers never send to
the web server. The app reads it on load (or when it changes while open),
removes it from the address bar so a reload cannot import it twice, and
fills the paste review. An empty or unreadable capture says so instead of
adding anything.

Limits: one image per run reads reliably; very long text (pages of it)
may hit URL length limits - the clipboard copy in step 5 is the backup.
Handwriting accuracy is whatever the iPhone's own recognition manages.
