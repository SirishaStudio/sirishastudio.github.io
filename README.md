# FormFlow — Dynamic Form Builder & Asset Portal

A single-file (`index.html`) vanilla JS app for GitHub Pages + Firebase. The Admin
designs form fields (text + document/image fields with crop, compress, and
format-conversion rules) inside the Admin workspace; the client site renders that
exact form for regular users, handles cropping/compression in the browser, and
uploads the results to Firebase Storage.

No build tools, no frameworks — everything runs from CDN imports inside `index.html`.

## What's in this folder

- `index.html` — the entire app (HTML, CSS, JS). This is the only file GitHub Pages needs to serve.
- `firestore.rules` — security rules blueprint for Firestore.
- `storage.rules` — security rules blueprint for Cloud Storage.

## 1. Create your Firebase project

1. Go to the [Firebase console](https://console.firebase.google.com/) → **Add project**.
2. Once created, go to **Build → Authentication → Get started** and enable the **Email/Password** sign-in method.
3. Go to **Build → Firestore Database → Create database** (start in production mode — the rules file below will lock it down properly).
4. Go to **Build → Storage → Get started** (also production mode).

## 2. Get your Firebase config

In the Firebase console: **Project settings → General → Your apps → Add app → Web (</>)**.
Copy the `firebaseConfig` object it gives you, then open `index.html` and paste it into
the `firebaseConfig` constant near the top of the `<script type="module">` block:

```js
const firebaseConfig = {
  apiKey: "...",
  authDomain: "...",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "..."
};
```

## 3. Create your Admin account and set ADMIN_UID

1. Open `index.html` in a browser (or deploy it first — see step 5) and sign up with
   the email/password you want to use as the Admin account.
2. In the Firebase console, go to **Authentication → Users** and copy that user's **UID**.
3. Paste it into `index.html`:
   ```js
   const ADMIN_UID = "paste-the-uid-here";
   ```
4. Paste the **same UID** into `firestore.rules` and `storage.rules`, replacing
   `"ADMIN_UID"` in both files' `isAdmin()` functions.

## 4. Publish the security rules

- Firestore: **Firestore Database → Rules** tab → paste in the contents of `firestore.rules` → Publish.
- Storage: **Storage → Rules** tab → paste in the contents of `storage.rules` → Publish.

These rules ensure:
- Only the Admin can create/edit form templates and settings.
- Any signed-in user can read the active template (so the form can render).
- Each customer can only read/write their own submission document and their own
  files under `submissions/{their-uid}/...`.
- Only the Admin can change a submission's `status` (approve/reject).

## 5. Deploy to GitHub Pages

1. Push this folder to a GitHub repository.
2. Repo → **Settings → Pages** → set **Source** to your branch (e.g. `main`) and
   folder `/ (root)` — or `/docs` if you place `index.html` there instead.
3. Your app will be live at `https://<username>.github.io/<repo>/`.
4. In the Firebase console, go to **Authentication → Settings → Authorized domains**
   and add your GitHub Pages domain so sign-in works there.

## 6. Build your form as the Admin

1. Sign in with your Admin account — you'll land on the **Admin workspace**.
2. Under **Form Builder**, set a tool name/description, then add **Text fields**
   and **Document fields**. For each document field choose:
   - Crop shape (1:1 square, 3:1 rectangle, or free crop)
   - Output format (`.jpg` or `.png`)
   - Max file size in KB
   - Target width in px
3. Reorder fields with the ↑ / ↓ buttons — order is saved exactly as arranged.
4. Click **Save & Publish Template**. This becomes the live form every customer sees.
5. Switch to the **Live Submissions** tab to watch entries arrive in real time,
   copy text answers, open uploaded files, and Approve/Reject each submission.

## 7. What customers see

- After signing up/in, customers get the dynamically rendered form.
- Clicking a document field's **Choose file** button opens a crop modal
  (powered by Cropper.js) locked to the aspect ratio you configured.
- After cropping, the app resizes and compresses the image in-browser via the
  Canvas API until it's under your configured size limit, then converts it to
  your chosen format — all before it ever reaches Firebase Storage.
- A **live status widget** at the top of the form updates instantly (via
  Firestore `onSnapshot`) when the Admin approves or rejects the submission.

## Theming

Four built-in themes (Light Pastel, Cyber Dark, Minimal Cream, Cute Pink) are
switchable from the 🎨 **Theme** button in the top bar and persist per-browser
via `localStorage`. Themes are implemented purely with CSS custom properties —
add a new one by adding another `[data-theme="..."]` block in `index.html`'s
`<style>` section and an entry in the `THEMES` array in the script.

## Notes & limitations to know about

- This is a client-only app — there's no server, so all validation (file size,
  crop shape, required fields) happens in the browser and is enforced as best
  as possible by the Storage/Firestore rules, not by a backend.
- The Admin UID is checked client-side to decide which UI to show; the real
  security boundary is the rules files, so make sure those are published with
  the correct UID before relying on this in production.
- Multiple form templates can technically exist in `form_templates`, but the
  UI here always edits/publishes to whichever one is currently marked
  `activeTemplateId` in `settings/app` — this keeps the beginner workflow to
  "one live form at a time." You can extend it to a template picker later.
