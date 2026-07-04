# FormFlow — Dynamic Form Builder & Base64 Submission Portal

A single-file (`index.html`) vanilla JS app for GitHub Pages + Firebase, built to run
entirely on Firebase's **free Spark plan** — no credit card, no Cloud Storage bucket.

The Admin designs form fields (text + document/image fields with free-style crop,
compression, and format-conversion rules) inside the Admin workspace; the client site
renders that exact form, crops/compresses images fully in the browser, and stores the
result as a Base64 text string directly inside the submission's Firestore document.

## What's in this folder

- `index.html` — the entire app (HTML, CSS, JS). This is the only file GitHub Pages needs to serve.
- `firestore.rules` — security rules blueprint for Firestore.

There is no `storage.rules` file — Cloud Storage isn't used anywhere in this app.

## Why no Cloud Storage?

Firebase Cloud Storage requires the paid **Blaze** plan to activate, even for tiny
amounts of usage. To keep this project at $0/month, files are never uploaded to a
bucket at all. Instead:

1. The user crops an image (free-style, any shape) with Cropper.js.
2. The Canvas API resizes it and iteratively lowers JPEG quality (or shrinks
   dimensions for PNG) until the resulting `canvas.toDataURL(...)` string fits
   under the Admin's configured size budget.
3. That Base64 string is saved as a normal text value inside the submission's
   `user_submissions/{uid}` document, right alongside the text field answers.

The trade-off: **Firestore caps every document at 1MiB total.** Base64 also inflates
the original image size by about 33%. So keep each document field's "Max file size
(KB)" modest — the Form Builder defaults to 60KB and warns you about this — especially
if a form has several document fields. The app also does a rough client-side check
before writing and will warn the user if a submission would be too large.

## 1. Create your Firebase project

1. Go to the [Firebase console](https://console.firebase.google.com/) → **Add project**.
2. Go to **Build → Authentication → Get started** and enable both the **Email/Password** and **Google** sign-in methods (Sign-in method tab → click each → Enable → Save). For Google, pick a support email when prompted.
3. Go to **Build → Firestore Database → Create database** (start in production mode — `firestore.rules` below locks it down).
4. You do **not** need to open or activate Storage at all.

## 2. Configuration is already wired up

`index.html` already contains the live Firebase config and hardcoded constants:

```js
const firebaseConfig = {
  apiKey: "AIzaSyDOdr6Zv0RgZBzPGvjMMea9WJ8tH62UT9s",
  authDomain: "sirishastudio-41d56.firebaseapp.com",
  projectId: "sirishastudio-41d56",
  storageBucket: "sirishastudio-41d56.firebasestorage.app",
  messagingSenderId: "430197596841",
  appId: "1:430197596841:web:f46b36e1b07c2de727ffb8",
  measurementId: "G-K3GB6LGVS3"
};

const ADMIN_UID = "5R2061iDtwUIfXp1aacA378fhza2";
const ADMIN_EMAIL = "sirishastudio9@gmail.com";
```

Make sure the account with UID `5R2061iDtwUIfXp1aacA378fhza2` actually exists in
**Authentication → Users** for `sirishastudio9@gmail.com` — sign up once with that
email/password (or sign in with that Google account) from the app's login screen if
you haven't already, and confirm the UID Firebase assigns it matches. The app checks
both the UID and the email against `ADMIN_EMAIL`, so signing in as the admin works
the same way whether you use email/password or the "Continue with Google" button.
If your project ever changes (new Firebase project, different admin account), update
these three values here **and** the matching UID inside `firestore.rules`.

## 3. Publish the security rules

Firestore Database → **Rules** tab → paste in the contents of `firestore.rules` → Publish.

These rules ensure:
- Only the Admin can create/edit form templates and settings.
- Any signed-in user can read the active template (so the form can render).
- Each customer can only read/write their own submission document.
- Only the Admin can change a submission's `status` (approve/reject).

## 4. Deploy to GitHub Pages

1. Push this folder to a GitHub repository.
2. Repo → **Settings → Pages** → set **Source** to your branch (e.g. `main`) and folder `/ (root)`.
3. Your app will be live at `https://<username>.github.io/<repo>/`.
4. In the Firebase console, go to **Authentication → Settings → Authorized domains** and add your GitHub Pages domain so sign-in works there.

## 5. Build your form as the Admin

1. Sign in with the Admin account — you'll land on the **Admin workspace**.
2. Under **Form Builder**, set a tool name/description, then add **Text fields** and **Document fields**.
3. For each document field, choose the output format (`.jpg`/`.png`), a max stored
   size in KB, and a target width in px. Cropping itself is always free-style — the
   customer can drag every edge of the crop box to any shape.
4. Reorder fields with the ↑ / ↓ buttons — order is saved exactly as arranged.
5. Click **Save & Publish Template**. This becomes the live form every customer sees.
6. Switch to the **Live Submissions** tab to watch entries arrive in real time.
   Document fields render as inline image thumbnails right in the table — click a
   thumbnail to open it full-size in a new tab. Text fields get a one-click Copy
   button. Approve/Reject buttons update status instantly for the customer.

## 6. What customers see

- After signing up/in, customers get the dynamically rendered form.
- Clicking a document field's **Choose file** button opens a free-style crop modal
  (Cropper.js) — drag any corner or edge to select any shape or size.
- After cropping, the Canvas API compresses the image in-browser until it's under
  the configured size limit, converts it to the chosen format, and shows a live
  "Ready — ~NNKB" preview — all before anything is saved.
- A **live status widget** at the top of the form updates instantly (via Firestore
  `onSnapshot`) when the Admin approves or rejects the submission.

## Theming

Four built-in themes (Light Pastel, Cyber Dark, Minimal Cream, Cute Pink) are
switchable from the 🎨 **Theme** button in the top bar and persist per-browser via
`localStorage`. Add a new one with another `[data-theme="..."]` block in
`index.html`'s `<style>` section plus an entry in the `THEMES` array in the script.

## Notes & limitations to know about

- This is a client-only app — there's no server, so all validation happens in the
  browser. The real security boundary is `firestore.rules`, so double-check the
  UID inside it matches your Admin account.
- Every field's Firestore document is capped at 1MiB total. With several document
  fields on one form, keep each one's Max KB conservative (well under 200KB per
  field is a safe rule of thumb) so a submission with multiple images doesn't hit
  that ceiling.
- The Admin UID is checked client-side to decide which UI to show, purely for
  routing convenience — access is actually enforced by `firestore.rules`.
