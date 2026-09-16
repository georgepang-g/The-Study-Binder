# The Binder — IB Grade 11

A personal study binder for the IB Diploma. Notes, tasks, projects, and PDF
files for each subject, with SL/HL levels and deadlines.

**Features**
- **Installable app (PWA)** — add it to your home screen for a real app icon,
  full-screen launch, and an offline app-shell.
- **Grades** — log assessment scores per subject, see each subject's average and
  a trend sparkline, set a predicted 1–7 grade, and track your predicted total /45
  (plus TOK/EE core points).
- **Flashcards** — build revision decks per subject and study them in a flip-card
  mode ("Got it" / "Again" re-queues cards until you know them all).
- **Study timer** — a focus countdown (15 / 25 / 50 min or a custom length) that
  logs each finished session to a subject, plus stats: last-7-days focus time,
  minutes per subject, minutes per day, and a recent-sessions list. You can also
  log time by hand.
- **Calendar** view of every deadline, and a **search** bar across all notes,
  tasks, projects, exam items, and files.
- Four sections per subject: **Notes, Tasks, Projects, and Exam prep** (past papers).
- **PDF files organized by section** — upload a PDF straight into Notes, Tasks,
  Projects, or Exam prep; they sync everywhere and render in-app.
- **Deadlines on any task, project, or exam item**, editable anytime, shown in a
  combined "Upcoming deadlines" list.
- **SL/HL per class** — pick your level once and it's remembered on every device.
- **Settings** — color theme (Paper / Light / Dark), class sort order, and a
  hide-completed-tasks option, all synced.

**`index.html` is the whole app.** It signs each person in and syncs their binder
across every device — phone, iPad, and computer — in real time. Everyone who
signs in gets their **own private binder**; nobody can see anyone else's.

It uses [Firebase](https://firebase.google.com) (Google's free backend) for
sign-in and storage. You need to do a one-time setup, then host the file.

---

## How the syncing works

- Each signed-in user has one document at `binders/{their-user-id}` in Firestore,
  plus a private `files` sub-collection for uploaded PDFs.
- Security rules lock all of it to that user only.
- The app listens for live changes, so editing on your phone updates your iPad
  and laptop within a second — as long as you're signed in with the same account
  on each.
- **Offline-ready:** Firestore's local cache (IndexedDB, multi-tab) is enabled, so
  the app opens instantly, works with no connection, and anything you change while
  offline is saved on the device and syncs automatically when you reconnect. The
  sidebar shows an "Offline" note while you're disconnected.

---

## One-time setup (about 5 minutes)

### 1. Create a Firebase project
1. Go to <https://console.firebase.google.com> and sign in with a Google account.
2. Click **Add project**, give it a name (e.g. `the-binder`), and finish. You can
   skip Google Analytics.

### 2. Add a Web app and copy the config
1. In your project, click the **`</>`** (Web) icon to "Add an app to get started".
2. Give it a nickname and register it (you do **not** need Firebase Hosting here).
3. Firebase shows a `firebaseConfig` object. Copy those values.
4. Open `index.html`, find the **`FIREBASE_CONFIG`** block near the bottom (inside
   `<script type="module">`), and paste your real values over the `PASTE_…`
   placeholders.

> These web config values are **not secrets** — they're meant to ship in the
> browser. What actually protects each binder is the security rules in step 4.

### 3. Turn on sign-in methods
In the console: **Build → Authentication → Get started**, then under
**Sign-in method** enable:
- **Email/Password** (works everywhere, no extra setup), and/or
- **Google** (one-tap on phones; if you use it, see "Authorized domains" below).

### 4. Create the database and lock it down
1. **Build → Firestore Database → Create database.** Start in **production mode**
   and pick a location.
2. Open the **Rules** tab, paste the following, and **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Each user can read and write only their own binder document...
    match /binders/{uid} {
      allow read, write: if request.auth != null && request.auth.uid == uid;

      // ...and only their own uploaded PDF files, which live in a
      // sub-collection under their binder. (Firestore rules do NOT cascade
      // to sub-collections, so this inner rule is required for PDF uploads.)
      match /files/{fileId} {
        allow read, write: if request.auth != null && request.auth.uid == uid;
      }
    }
  }
}
```

That rule is what makes every person's binder — and every PDF they upload —
private to them.

> **Upgrading from the first version?** If you already published the old rules
> (which had no `files` block), re-paste the rules above and **Publish** again, or
> PDF uploads will be denied.

#### Optional: stricter rules (defense-in-depth)

The rules above are already secure — each user can only ever touch their own data.
If you want to also reject malformed writes (stray fields, wrong shapes), you can
publish this hardened version instead. It matches exactly what the app writes, so
nothing breaks; it just rejects anything unexpected:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /binders/{uid} {
      allow read: if request.auth != null && request.auth.uid == uid;
      allow write: if request.auth != null && request.auth.uid == uid
                   && request.resource.data.keys().hasOnly(
                        ['entries','dismissed','courses','files','settings','updatedAt']);

      match /files/{fileId} {
        allow read, delete: if request.auth != null && request.auth.uid == uid;
        allow create, update: if request.auth != null && request.auth.uid == uid
                              && request.resource.data.keys().hasOnly(
                                   ['i','data','name','type','size','createdAt']);
      }
    }
  }
}
```

---

## Host it so all your devices can reach it

Pick any one of these. All are free.

### Option A — GitHub Pages (easiest, this repo)
1. Push this repo to GitHub (it already is if you're reading this there).
2. Repo **Settings → Pages → Build and deployment → Source: Deploy from a branch**,
   choose the `main` branch and `/root`, then **Save**.
3. After a minute your site is live at
   `https://<your-username>.github.io/<repo-name>/`.
4. Open that URL on your phone, iPad, and computer; sign in with the same account
   on each. Done.

### Option B — Firebase Hosting
```
npm install -g firebase-tools
firebase login
firebase init hosting     # public dir: . (or move index.html into public/)
firebase deploy
```

### Option C — Netlify / Vercel
Drag the folder onto <https://app.netlify.com/drop>, or import the repo in Vercel.

### Authorized domains (only if you use Google sign-in)
In **Authentication → Settings → Authorized domains**, add the domain you host on
(e.g. `your-username.github.io`). `localhost` is already allowed for testing.
Email/password sign-in does not need this.

---

## Spotify "now playing" overlay (optional)

Shows the song you're currently playing on Spotify in the bottom-left corner. It
uses Spotify's Authorization Code + PKCE flow, so it works entirely in the
browser with no server. Your tokens are stored only in that browser; nothing
about Spotify is written to Firestore. Leave `SPOTIFY_CLIENT_ID` on its
placeholder to keep this feature off.

1. Go to <https://developer.spotify.com/dashboard>, log in, and click **Create app**.
2. Fill in any name/description. For **Redirect URI**, enter your site's exact URL
   including the trailing slash, e.g. `https://your-username.github.io/your-repo/`.
   Under **APIs used**, tick **Web API**. Save.
3. Open the app's **Settings** and copy the **Client ID**. Paste it into the
   `SPOTIFY_CLIENT_ID` value near the top of `index.html` (just below
   `FIREBASE_CONFIG`), then commit.
4. Open your site, click **🎵 Connect Spotify** in the sidebar, and authorize.
   Start playing something on Spotify (any device) and it appears in the corner.

The overlay shows album art, track/artist, a progress bar, **play/pause and skip
controls**, and a **volume slider**. **Drag it** by the album art or title to move
it anywhere; it remembers where you put it (default position is the top-right
corner). Playback and volume controls require Spotify **Premium**.

Notes:
- Reading "now playing" works on **free** Spotify accounts. The **play/pause,
  skip, and volume** controls require **Premium** (Spotify only allows apps to
  control playback for Premium accounts); on a free account they show a short
  "needs Premium" note.
- It shows a song **only while Spotify is actively playing** somewhere; otherwise
  it stays hidden.
- The Client ID is not a secret (it ships in every Spotify web app). The redirect
  URI must match your live URL exactly, or Spotify will reject the login.

---

## Using it day to day
- **Install it:** on iPhone/iPad use Share → **Add to Home Screen**; on desktop
  Chrome/Edge use the install icon in the address bar. It launches full-screen
  with its own icon and works offline (a service worker caches the app shell;
  `manifest.webmanifest`, `sw.js`, and `icon-*.png` are part of the repo).
- **Calendar** (sidebar) shows all your deadlines on a month grid; **Search**
  finds anything across your binder.
- Sign in once per device; you'll stay signed in.
- Everything you file syncs automatically. There's also a **Download study
  guide (.txt)** button in the sidebar for an offline copy.
- **Viewing PDFs:** tapping **Open** renders the PDF right inside the app (using
  the bundled PDF.js in `vendor/`), so it works on iPhone/iPad where the browser
  won't reliably open PDFs on its own. A **Download** button is there too.
- **Uploading PDFs:** open a subject, then use **+ Upload PDF** in its Files
  section. PDFs are stored in Firestore. Because Firestore caps any single
  document at ~1 MB, each PDF is base64-encoded and split into chunk documents
  (`binders/{uid}/files/{id}_c0`, `_c1`, …) that are stitched back together when
  you open it — all inside the same `files` sub-collection, so no extra rules are
  needed. Each PDF can be up to **15 MB**. This all stays on the free (Spark)
  plan; the plan's real ceiling is **1 GiB of total storage** shared across all
  files. Need more (lots of large files, or many users)? Upgrade the project to
  the Blaze plan and switch uploads to Firebase Storage — ask and it can be added.
- **Your level (SL/HL):** each subject page has an SL/HL toggle next to "+ Add".
  Tap it once; your choice is saved to that course and follows you everywhere.
- **Settings** live in the sidebar (theme, class sorting, completed tasks).

## Testing locally before hosting
Open a terminal in this folder and run `python3 -m http.server 8000`, then visit
`http://localhost:8000`. (Opening the file directly with `file://` can block
sign-in, so use a local server.)
