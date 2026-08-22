# Markbench

Laser marking template designer for lockout/tagout locks. Drag-and-drop label
editor, batch roster, and direct export to LightBurn (`.lbrn`) and SVG for the
Haas HL-50E 50 W fiber marker.

One static HTML file. No build step, no bundler, no server. Firebase is
optional — without it the app is fully functional and stores work in the
browser.

---

## Contents

| File | What it is |
|---|---|
| `index.html` | The whole application |
| `firestore.rules` | Security rules to paste into the Firebase console |
| `.nojekyll` | Stops GitHub Pages from running Jekyll over the files |
| `.github/workflows/pages.yml` | Optional Actions-based Pages deploy |

---

## 1. Put it on GitHub Pages

```bash
git init
git add .
git commit -m "Markbench"
git branch -M main
git remote add origin https://github.com/<you>/markbench.git   # or your Enterprise host
git push -u origin main
```

Then in the repository: **Settings → Pages → Source: Deploy from a branch →
`main` / `(root)` → Save.**

The site appears at `https://<you>.github.io/markbench/` (or your Enterprise
Pages host) after a minute or so.

If your organisation blocks branch deployment, leave `.github/workflows/pages.yml`
in place and choose **Source: GitHub Actions** instead. If branch deployment
works, you can delete that file.

`index.html` at the repository root is what makes the bare URL work — don't
rename it.

### On a phone

Open the Pages URL in Safari or Chrome and add it to the home screen. It runs
full-screen and remembers your work between sessions.

---

## 2. Wire up Firebase (optional)

Skip this entirely if you only want a local tool. Add it when you want the
shop sharing one design library, one people roster, and one set of proven
laser recipes.

### Create the project

1. <https://console.firebase.google.com> → **Add project**.
2. **Build → Firestore Database → Create database → Production mode.**
   Pick a region near you; `nam5` is fine for the US.
3. **Build → Authentication → Get started → Sign-in method →** enable
   **Anonymous**. (Or **Google** if you want named users — see below.)
4. **Project settings → Your apps → Web (`</>`)** → register an app.
   Copy the `firebaseConfig` object it shows you.

### Publish the rules

**Firestore Database → Rules →** paste the contents of `firestore.rules` →
**Publish.** Do this before you connect, or every write is rejected.

### Authorise the domain

**Authentication → Settings → Authorized domains → Add domain** and enter your
Pages host (for example `you.github.io`, or your Enterprise Pages hostname).

### Connect the app

Open the app, click the **Local** chip in the header (or **Cloud** in the right
panel), paste **Project ID**, **API key** and **App ID**, and press
**Connect**. There's a *Paste whole config* button that accepts the whole
`firebaseConfig` object as-is.

The config is stored in your browser. To connect everyone automatically
instead, fill in `FIREBASE_BUILTIN` near the top of the `<script>` block in
`index.html` and commit it:

```js
const FIREBASE_BUILTIN = {
  apiKey: "AIza…",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project",
  appId: "1:123456789:web:abc123",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "123456789",
  workspace: "shop",
  authMode: "anonymous"
};
```

A Firebase web `apiKey` is an identifier, not a credential. It is meant to be
public. Your Firestore rules are what actually protect the data — which is why
publishing them properly matters more than hiding the key.

### Workspaces

Everything lives under `workspaces/<name>`. Change **Workspace** in the setup
form to keep separate libraries — say `plant-1` and `plant-2` — inside one
Firebase project.

---

## 3. What lives in the database

```
workspaces/{workspace}/
  designs/{id}                 name, size, face count, who saved it, when
  designs/{id}/body/main       the design payload (JSON)
  people/{id}                  first, last, dept — the shared roster
  presets/{id}                 speed, power, frequency, interval, passes
```

The list view only reads the small metadata records, so opening the app doesn't
pull down every design. The payload is fetched only when you open one.

Firestore caps a single record at 1 MB. A design with a 1-bit logo at 600 dpi
is normally 20–80 KB. The Cloud panel shows the current size and warns you
before you hit the ceiling; drop the graphic to 300 dpi if you ever do.

---

## 4. Named users instead of anonymous

Anonymous sign-in means anyone who can load the page can read and write the
library. That's usually right for an internal Enterprise Pages site and wrong
for a public one.

To use named accounts:

1. **Authentication → Sign-in method →** enable **Google**.
2. Set `authMode` to `"google"` in the setup form or in `FIREBASE_BUILTIN`.
3. In `firestore.rules`, delete Option A and uncomment Option B, changing the
   email domain to yours. Publish.

Saves are then stamped with the actual person's email rather than a device ID.

---

## 5. If the network blocks the CDN

The Firebase SDK loads from `https://www.gstatic.com/firebasejs/…`. On a locked
down corporate network that may be blocked — the app will tell you so plainly
and keep working locally.

Two ways around it:

- Ask IT to allow `www.gstatic.com` and `*.googleapis.com`.
- Vendor the SDK: download `firebase-app.js`, `firebase-auth.js` and
  `firebase-firestore.js` from that URL into a `vendor/` folder in this repo,
  then change `FB_CDN` in `index.html` to `"./vendor"`.

---

## 6. Using it

**Stock** — lock body size, label size, padding, units. Padding defines the
safe area; elements are clamped inside it and reflow when you resize.

**Build** — add text, graphic, symbol or rule elements. Drag, resize, snap.
Each element anchors to one of nine points on the safe area, so it holds its
position when the label changes size.

**Element** — per-element properties. Text auto-shrinks or wraps to fit.
Graphics resample to their printed size at 300/600/1000 dpi and convert to
1-bit black-and-clear, which is what the fiber laser actually marks.

**Batch** — the roster. Text tokens `{first}` `{last}` `{dept}` `{n}` `{date}`
fill per lock, so one design covers the whole batch.

**Laser** — material presets with a too-light / too-dark trim ladder, manual
override, and a 4×4 parameter test grid you can mark on a scrap body.

**Export** — `.lbrn` with your material layer already configured and each face
grouped, or `.svg`. Both are written in millimetres at true size.

Raster logos travel in the SVG, not in the `.lbrn` — the LightBurn file
reserves a guide box for them on a no-output layer instead. Import the SVG, or
place the logo in LightBurn separately.

Check the first face on screen in LightBurn before running a batch: text
anchoring shifts slightly between LightBurn versions.
