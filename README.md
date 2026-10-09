# Wedding Management Application

A wedding planning dashboard — budget, payment ledger, funding & cash
requirement, guest lists, checklist, day-of agenda, user accounts, and
(as of this version) multiple wedding projects — in a single `index.html`.
No build step, no backend to run yourself.

## Deploy to GitHub Pages

1. On GitHub, click **New repository**. Name it anything (e.g. `wedding-app`).
   Keep it **Public** — GitHub Pages on the free plan requires a public repo.
2. Upload `index.html` ("Add file → Upload files"), or push it with git.
3. In that repo, go to **Settings → Pages**. Under "Build and deployment,"
   set **Source** to `Deploy from a branch`, branch `main`, folder
   `/ (root)`. Save.
4. Your live link appears within a minute or two:
   `https://<your-username>.github.io/wedding-app/`

Updating later: just re-upload `index.html` with the same filename to
overwrite it — Pages redeploys automatically within a minute or two.

## Multiple projects (new)

The app now supports more than one wedding under one login system — useful
for a couple with two separate functions (e.g. the wedding and a homecoming
at a different hotel with a separate budget), or for running more than one
couple's wedding from a single Super Admin account.

- **Super Admin** logs in and lands on a **Projects** tab (always first).
  From there they create new projects (a title, the Bride and Groom's
  names, and new login credentials for each), and open any project to see
  its full dashboard.
- **Bride/Groom/Task Access** accounts are each tied to one or more specific
  projects. With access to exactly one, they log in and land straight in
  it — no project list to navigate. With access to more than one, they get
  a simple picker, and can jump between them later via the profile menu's
  **"Switch project."**
- An existing user (say, a helper who already has a Task Access login on
  one project) can be added to a *different* project too, instead of
  creating a duplicate account — from **Manage Users** (profile menu),
  there's an "Add existing user to this project" option alongside creating
  a brand new one.
- Each project has its own completely separate budget, guest list,
  checklist, agenda, and activity history. Export and Import (footer) act
  on whichever project is currently open, never on your other projects.
- **Resetting a project** is done by the Super Admin only, from the
  **Projects** tab: each project card has its own **Reset** button. It clears
  that one project's budget, payments, guests, checklist, agenda and history
  and starts it fresh (title, couple's names, venue and date are kept). Every
  other project and every login is left exactly as it was. A copy of the
  data is kept as a local backup first (see "Recover data found on this
  device").

Your existing wedding data was carried over automatically into a project
called **"Project 01"** the first time this version loads — nothing about
that data changed, it just moved one level deeper.

## Cloud sync (optional — for changes to appear across devices)

Without this, each device/browser keeps its own separate copy of the data.
With it, everyone sees the same live data. This uses Firebase (Google's),
specifically Firestore — free for this scale of use.

**Setup:**
1. **console.firebase.google.com** → sign in → **Add project** (any name,
   skip Analytics).
2. **Build → Firestore Database → Create database** → **Start in test
   mode** → pick a region → Create.
3. **Build → Authentication → Get started → Sign-in method** → enable
   **Anonymous**.
4. Gear icon → **Project settings** → "Your apps" → **`</>`** (web) icon →
   register an app → copy the `firebaseConfig` object it shows you.
5. In `index.html`, search for `FIREBASE_CONFIG` (near the top of the
   `<script>` block) and paste your six values in over the `"YOUR_..."`
   placeholders. Re-upload the file.
6. Reload the live page — the footer should show a **"Live — synced across
   devices"** badge.

**Lock down the database** (do this before sharing the link widely — test
mode is wide open to anyone who reaches it directly): Firebase console →
**Firestore Database → Rules** → replace with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /weddingApp/{docId} {
      allow read, write: if request.auth != null;
    }
  }
}
```

Click **Publish**. This means only requests coming from the app itself
(which signs in anonymously and automatically) can read or write the data.
It does not distinguish a Super Admin from a random visitor who has the
link — the app's own login screen is what does that day-to-day.

**Backups (free plan, nothing to switch on):** Firebase's own backup
features (scheduled backups, point-in-time recovery) need the paid Blaze plan
and only protect from the moment you enable them, so this app keeps its own.
In the **Projects** tab, the Super Admin has a **Backups** panel:
- **Download full backup** — one file with every project and login. Do this
  now and then, and before any big change. Keep it somewhere safe.
- **Restore from file** — either *add as new projects* (never touches what you
  have) or *replace everything* (full backups only).
- **Daily cloud snapshots** — when cloud sync is on, the first save of each
  day copies the shared data (as it was just before that save) into one of 14
  rotating slots, so about two weeks stay available. A blank/untouched state
  is never snapshotted, so a reset can't push a good snapshot out. Use
  **Show cloud snapshots** to browse them and restore one. **Back up to the
  cloud now** takes one on demand.
- Cloud snapshots are stored as extra documents beside the main one, which is
  why the rule above is `/weddingApp/{docId}` rather than a single document.
  If you still have the older single-document rule, normal saving keeps
  working but snapshots are blocked, and the Backups panel will say so.

**Test-mode rules expire.** Firebase's "test mode" rules stop working 30
days after the database was created, and sync silently stops (the footer
badge turns red: "Sync blocked by Firestore rules"). Replace them with the
rule above well before then.

**Safeguards (added after a data-loss scare):**
- A device that hasn't yet heard back from the server never writes to the
  shared data, so a blank phone / new browser / flaky connection can no
  longer overwrite it with empty starter data.
- Saves are revision-checked: if someone else saved since this device last
  synced, the newer shared data wins, and this device's own copy is kept as
  a local backup instead of being thrown away.
- Before this device's data is ever replaced by shared data, up to three
  snapshots are kept in the browser.
- **If data ever goes missing:** log in as Super_Admin (the password may have
  reverted to `Super_Admin_26`), open the **Projects** tab, and look for
  "Recover data found on this device". It lists any older copies still stored
  in that browser and restores them as new projects without touching what you
  have. Try the device you used most. Free Firebase keeps no version history,
  so a device copy or an exported backup (footer → Export) is the only way
  back — export a project now and then.
- After deploying a new version, reload every device once (Ctrl+F5, or close
  and reopen the tab) so nobody keeps running an older cached copy.

**Behavior worth knowing:** edits sync roughly a second after you stop
typing, not on every keystroke. If a remote change arrives while you're
actively typing in a field, the app holds it back until you click away from
that field, so it can't overwrite what you're mid-typing. If two people
edit the exact same field at the exact same moment, whichever save reaches
the database last wins — there's no merge.

## First login (either way)

- Username: `Super_Admin`
- Password: `Super_Admin_26`

Change this password from the profile menu (top-left icon) once you're set
up, then create your Bride and Groom logins from the Projects tab.

## Honest limits

This is a client-side app with no real server-side security. Anyone who
opens a browser's dev tools on this page can see the underlying data
regardless of which account they're logged in as, and Firestore's rule
above stops outside scanning but not someone who has the link. If that
matters for your situation, don't publish the live link anywhere public,
and treat the login system as a courtesy barrier for your family/wedding
party rather than a real security boundary.
