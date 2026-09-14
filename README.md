# Wedding Management Application

A wedding planning dashboard — budget, payment ledger, funding & cash
requirement, guest lists, checklist, day-of agenda, and user accounts — in a
single `index.html`. No build step, no backend to run yourself.

It works two ways:

- **As-is (out of the box):** each visitor's browser keeps its own local
  copy of the data. Nobody's changes show up for anyone else.
- **With cloud sync turned on (optional, instructions below):** everyone who
  opens the link sees the same live data, and changes made by one person
  appear for everyone else within about a second.

## Part 1 — Deploy to GitHub Pages

1. On GitHub, click **New repository**. Name it anything (e.g. `wedding-app`).
   Keep it **Public** — GitHub Pages on the free plan requires a public repo.
2. Upload `index.html` from this folder ("Add file → Upload files"), or push
   it with git (see below).
3. In that repo, go to **Settings → Pages**. Under "Build and deployment",
   set **Source** to `Deploy from a branch`, branch `main`, folder
   `/ (root)`. Save.
4. Your live link appears within a minute or two, shaped like:
   `https://<your-username>.github.io/wedding-app/`

Want it at the shorter `https://<your-username>.github.io/` address instead?
Name the repository exactly `<your-username>.github.io` — Pages turns on
automatically for a repo with that exact name.

```bash
# command-line alternative to the web upload
cd wedding-management-app
git init
git add index.html README.md .gitignore
git commit -m "Wedding management app"
git branch -M main
git remote add origin https://github.com/<your-username>/<repo-name>.git
git push -u origin main
```

## Part 2 — Turn on cloud sync (optional but recommended for multiple users)

This uses Firebase (Google's), specifically its Firestore database — free
for this scale of use, no server for you to run, called directly from the
browser.

### Create the project (about 5 minutes)

1. Go to **console.firebase.google.com**, sign in with any Google account,
   click **Add project**. Name it anything. You can skip Google Analytics.
2. Left sidebar → **Build → Firestore Database → Create database** → choose
   **Start in test mode** → pick any region close to you → Create.
   (We'll lock this down properly in step 5 below.)
3. Left sidebar → **Build → Authentication → Get started → Sign-in method**
   → click **Anonymous** → enable it → Save.
   (This is a separate, invisible technical login just so Firestore knows
   requests are coming from this app — it has nothing to do with the app's
   own Super Admin / Bride / Groom login screen, which is unchanged.)
4. Gear icon (top-left, next to "Project Overview") → **Project settings**
   → scroll to "Your apps" → click the **`</>`** (web) icon → give it any
   nickname → **Register app**. It shows a code block like:

   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "wedding-app-xxxxx.firebaseapp.com",
     projectId: "wedding-app-xxxxx",
     storageBucket: "wedding-app-xxxxx.appspot.com",
     messagingSenderId: "...",
     appId: "..."
   };
   ```

### Paste it into `index.html`

Open `index.html` in any text editor and search for `FIREBASE_CONFIG` (it's
near the very top of the `<script>` block, clearly marked). Replace the six
placeholder values with the six values from your own config. Save, then
re-upload/re-push `index.html` to your GitHub repo the same way you did in
Part 1.

That's it — reload the live page and you should see a small **"Live —
synced across devices"** badge in the footer at the bottom of the page.

### Lock down the database (do this — test mode is wide open)

"Test mode" from step 2 allows anyone to read/write your database for 30
days, then locks everyone out entirely. Replace it with a real rule:

1. In the Firebase console: **Build → Firestore Database → Rules** tab.
2. Replace the contents with:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /weddingApp/sharedState {
         allow read, write: if request.auth != null;
       }
     }
   }
   ```

3. Click **Publish**.

This means only requests coming from the app itself (which signs in
anonymously and automatically) can read or write the data — a random script
hitting Firestore's API directly without going through your page can't.

**The honest limit of this, worth knowing:** anyone who has your live link
and opens it in a browser *is* "the app" as far as these rules are
concerned — the rule doesn't (and can't, without a much bigger rebuild)
distinguish a Super Admin from a total stranger at the database level. The
app's own login screen is still what's actually keeping casual visitors out
day-to-day; these Firestore rules mainly stop automated scanning and direct
API access that bypasses the page entirely. Don't publish the live link
anywhere public if that's not an acceptable risk for your situation.

### A few things worth knowing about how sync behaves

- Changes sync roughly a second after you stop typing in a field, not on
  every keystroke — this keeps it efficient and avoids fighting with
  whoever else is typing at the same moment.
- If two people edit the *exact same field* in the *exact same moment*,
  whoever's change reaches the database last wins — there's no merge. For a
  family planning tool this essentially never comes up in practice.
- If a remote change arrives while you're actively typing in a field, the
  app holds it back until you click away from that field, so it can't yank
  your cursor or overwrite what you're mid-typing.
- The Export/Import backup buttons in the footer still work as a manual
  safety net regardless of whether cloud sync is on.
- **"Reset to blank"** and **"Import backup"** now warn you explicitly when
  cloud sync is on, since those actions replace the data for everyone
  currently using the link, not just your own device.

### Turning it back off

Just restore the six `FIREBASE_CONFIG` values to their original
`"YOUR_..."` placeholders and re-upload. The app falls back to
browser-local storage exactly as it worked before this was ever set up.

## First login after deploying (same either way)

- Username: `Super_Admin`
- Password: `Super_Admin_26`

Log in, open the profile icon (top-left) → **Manage users**, and create
accounts for Bride, Groom, and anyone else who needs access. Change the
Super Admin password from the profile menu once you're set up.
