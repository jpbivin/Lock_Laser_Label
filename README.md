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
  designs/{id}                 metadata — see below
  designs/{id}/body/main       the design payload (JSON)
  people/{id}                  first, last, dept — the shared roster
  presets/{id}                 shared materials, stamped with the machine
                               they were proved on
```

Each design record carries enough to identify it without opening it: name,
lock body and label size, padding, fixture rows/columns/pitch/field and how
many cells are rotated, skipped or moved, machine and lens, material with its
proven flag and full laser settings, element census, text mode, who saved it
and when. The Cloud panel shows that as an expandable detail line, so you can
tell two similar locks apart at a glance.

Name each design in the Cloud tab — "Red thermoplastic 1.5in", "Aluminium tag
2in" — and **Save as new** keeps them separate. Opening one restores the whole
document: geometry, fixture layout including per-cell rotations, roster,
machine, material and laser settings.

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

## 6. Screen sizes

Three layouts, picked automatically:

| Width | Layout |
|---|---|
| under 780 px | Phone — full-bleed canvas, bottom nav, panels as sheets |
| 780–1179 px | Canvas plus a docked properties column; elements slide in from the left |
| 1180 px and up | Three columns, nothing hidden |

Touch and mouse are handled separately too — coarse pointers get larger drag
handles and hit targets, fine pointers get a denser panel.

If you keep a browser window narrow on a desktop (a portrait monitor, or a
half-screen split) you'll get the phone layout, which is intentional — it's the
one that works in that space.

---

## 7. Using it

**Stock** — lock body size, label size, padding, units. Padding defines the
safe area; elements are clamped inside it and reflow when you resize.

**Build** — add text, graphic, symbol or rule elements. Drag, resize, snap.
Each element anchors to one of nine points on the safe area, so it holds its
position when the label changes size.

**Element** — per-element properties. Text auto-shrinks or wraps to fit.

Every element has an **Output** setting: *Mark this* or *Reference only*.
Reference elements are drawn in the editor as a dashed violet ghost and are
never written to the `.lbrn` or the `.svg`. Use them to stand in for whatever
is already on the lock — a stamped serial number, a cast logo, a hole — and lay
the real marks around it. Because they record what exists rather than what you
are adding, they are allowed outside the padding line and never raise a
safe-area warning. There's a per-element toggle in the Elements list too, and
the Export tab reports how many are held back so nothing goes missing quietly.

Graphics have two output modes:

- **Vector outlines** (the default). The image is thresholded to 1-bit and
  contour-traced into closed paths, holes and counters included. It exports as
  real geometry in *both* the `.svg` and the `.lbrn`, marks faster than an
  image fill, and has crisp edges at any size. *Simplify* trades point count
  against fidelity; *Drop specks* clears stray pixels. Right for logos, badges,
  symbols, and anything flat.
- **Raster image**. Resamples to the printed size at 300/600/1000 dpi and
  converts to 1-bit black-and-clear or greyscale. Right for photographs and
  shading. Embeds into the `.lbrn` as a Bitmap shape, or you can switch it to
  a guide box and import the `.svg` instead.

Small sources are scaled up before tracing — interpolating the edge first puts
the traced contour much closer to the true curve than the original pixel
corners would.

**Batch** — the roster. Text tokens `{first}` `{last}` `{dept}` `{n}` `{date}`
fill per lock, so one design covers the whole batch.

**Fixture** — where the locks physically sit on the bed. Switch the canvas to
**Fixture** with the toggle at the top-left of the stage. Set rows, columns and
**pitch** — centre to centre, the number you measure on the jig, not an edge
gap. The bed shows the lens field, a crosshair on every lock centre, and the
pitch dimensioned between the first two.

- Drag a lock to move that one off pitch; the rest of the array stays put.
- Tap a lock to skip that pocket — broken clamp, missing fixture insert.
- **Which face goes where.** Leave *Assignment* on Automatic and faces fill in
  the order set under Sheet layout. Switch to **By pocket** and you get a
  miniature of the bed: every pocket shows a lock slot and a face, and tapping
  one lets you set both. *Face per column* and *Face per row* fill the whole
  grid in one press; *Match automatic* copies the automatic layout as a
  starting point.

  A pocket's **slot** is which lock of a load it holds, so the pattern repeats
  every run — set "column 1 marks side 1, column 2 marks side 2" once and it
  holds for every batch of locks you feed it. Leave a pocket empty and nothing
  is marked there.
- Rotate a cell 90/180/270 when the fixture holds that lock the other way up.
  **Flip rows 2, 4, 6…** does alternating rows in one press. The artwork turns
  with the lock, so the fixture never has to change.
- **Marked area** reports the overall centre and size exactly as LightBurn
  reports a selection. Type your fixture's X and Y into *Place centre at* and
  the whole array moves there — set once, keep forever.
- Arrow keys nudge the selected cell by one grid step.
- *Centre in field* and *Fill the field* set the array up in one press.
- Stagger offsets odd rows for a brick-pattern fixture.

If the roster needs more locks than the fixture holds, the panel splits the job
into runs. Step through them, or use **All runs** in Export to write one
`.lbrn` per load.

Exports are referenced to the marking field, not to the artwork, so the
coordinates in LightBurn line up with the physical fixture.

**Machine zero** in the Bed section sets whether bed coordinates read from the
bottom-left (the usual fibre setup, and the default) or the top-left. It changes
what the panel, ruler and status bar display — the exported file is written
Y-up either way and does not change.

**Machine** — a library of the machines you own. Name, wattage, source type,
lens field, and hard limits for speed, power, frequency and line interval.
Every setting anywhere in the app is clamped to the active machine, so a recipe
proved on a 60 W MOPA can't quietly ask a 50 W Q-switch for speeds it doesn't
have. The machine name is written into the `.lbrn` as the device, and MOPA
machines get pulse width as a real parameter.

**Material** — your recipe library. Each material stores speed, power,
frequency, interval, passes, mode, pulse width, which machine it was proved on,
and notes on what to watch for. Add materials, edit them, mark them proven.

The too-light / too-dark ladder trims the active recipe without touching what's
stored; when the mark is right, press **Save trim into material** and the tuned
numbers become the stored ones. **Capture current** forks a tuned variant into
a new material instead. A 4×4 parameter test grid is one press away.

**Export** — `.lbrn` with your material layer already configured and each face
grouped, or `.svg`. Both are written in millimetres at true size.

Text is converted to **outlines** by default. LightBurn's text shape anchors
vertically by the top of the character box rather than the baseline, and its
file format doesn't document that, so handing over geometry removes the guess
entirely — and removes the need for the font to be installed on the laser PC.
What the editor shows is what lands on the part. Switch to *Live text* in the
Export tab if you'd rather keep it editable in LightBurn; a vertical nudge is
provided there for fine tuning.

Knocked-out text over a filled bar is folded into the bar as holes in a single
even-odd shape, so it cuts out correctly regardless of the layer's fill
grouping setting.

Traced graphics need nothing special — they are ordinary paths in both files.
Raster graphics embed into the `.lbrn` as a Bitmap shape; open the first one
and confirm it landed at the right size and place before running a batch, and
if it hasn't, switch that element to *Guide box only* and import the `.svg`.

Check the first face on screen in LightBurn before running a batch: text
anchoring shifts slightly between LightBurn versions.
