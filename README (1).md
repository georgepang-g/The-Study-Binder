# The Binder — IB Grade 11

A personal study binder for the IB Diploma. Notes, tasks, and projects for each
subject, with SL/HL levels and deadlines.

**`index.html` is the whole app.** It signs each person in and syncs their binder
across every device — phone, iPad, and computer — in real time. Everyone who
signs in gets their **own private binder**; nobody can see anyone else's.

It uses [Firebase](https://firebase.google.com) (Google's free backend) for
sign-in and storage. You need to do a one-time setup, then host the file.

---

## How the syncing works

- Each signed-in user has one document at `binders/{their-user-id}` in Firestore.
- Security rules lock that document to that user only.
- The app listens for live changes, so editing on your phone updates your iPad
  and laptop within a second — as long as you're signed in with the same account
  on each.

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
    // Each user can read and write only their own binder document.
    match /binders/{uid} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```

That single rule is what makes every person's binder private to them.

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

## Using it day to day
- Add it to your home screen on iPhone/iPad (Share → **Add to Home Screen**) and
  it behaves like an app.
- Sign in once per device; you'll stay signed in.
- Everything you file syncs automatically. There's also a **Download study
  guide (.txt)** button in the sidebar for an offline copy.

## Testing locally before hosting
Open a terminal in this folder and run `python3 -m http.server 8000`, then visit
`http://localhost:8000`. (Opening the file directly with `file://` can block
sign-in, so use a local server.)
