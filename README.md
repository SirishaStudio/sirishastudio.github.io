# Sirisha Studio — Document Submission Portal

A single-file (`index.html`) vanilla JS app for GitHub Pages + Firebase, built to run
entirely on Firebase's **free Spark plan** — no credit card, no Cloud Storage bucket.
The Admin builds one or more forms (PAN Card, Voter ID, anything else) from a
Manage Forms nav; customers pick which form to fill from their own nav; every file
(photo, signature, PDF) is cropped/resized/compressed in the browser and stored as
a Base64 string right inside the submission document.

## What's in this folder

- `index.html` — the entire app (HTML, CSS, JS).
- `icon.svg` — the app's icon/favicon (also used by the manifest).
- `manifest.webmanifest` — makes the site installable as an app.
- `sw.js` — a minimal service worker (required for installability).
- `firestore.rules` — security rules blueprint for Firestore.
- `firestore.indexes.json` — the one composite index the admin feed needs.

There is no Storage bucket and no `storage.rules` — see "Why no Cloud Storage?" below.

## 1. Create your Firebase project

1. [Firebase console](https://console.firebase.google.com/) → **Add project**.
2. **Build → Authentication → Get started** → enable **Email/Password** and **Google**
   sign-in methods (Sign-in method tab → click each → Enable → Save; Google asks for a support email).
3. **Build → Firestore Database → Create database** (production mode — `firestore.rules` locks it down).
4. You do **not** need Storage at all.

## 2. Configuration is already wired up

`index.html` already contains your live config and constants:

```js
const firebaseConfig = { apiKey: "AIzaSyDOdr6Zv0RgZBzPGvjMMea9WJ8tH62UT9s", ... };
const ADMIN_UID = "5R2061iDtwUIfXp1aacA378fhza2";
const ADMIN_EMAIL = "sirishastudio9@gmail.com";
```

Sign in once as `sirishastudio9@gmail.com` (email/password or Google) so that account
exists in **Authentication → Users**, and confirm its UID matches `ADMIN_UID`. The
app checks both the UID and the email, so either sign-in method works for the admin.

## 3. Publish the security rules + index

- **Firestore Database → Rules** → paste in `firestore.rules` → Publish.
- **Firestore Database → Indexes → Composite** → you can add the index from
  `firestore.indexes.json` manually (collection `user_submissions`, fields
  `templateId` Ascending + `createdAt` Ascending), **or** just open the admin
  **Submissions** tab once — Firestore will show an error in the browser console
  with a direct "Create index" link; click it, wait about a minute, and reload.

## 4. Deploy to GitHub Pages

1. Push this whole folder to a GitHub repo (keep `index.html`, `icon.svg`,
   `manifest.webmanifest`, and `sw.js` together at the same level — they reference each other by relative path).
2. Repo → **Settings → Pages** → Source = your branch, folder `/ (root)`.
3. Live at `https://<username>.github.io/<repo>/`.
4. Firebase console → **Authentication → Settings → Authorized domains** → add your
   GitHub Pages domain.

## 5. How the app is organized now

### Admin: Manage Forms → Builder → Submissions
- **Manage Forms** is the home tab: every form you've created shows as a card with
  a Draft/Published badge, plus **Edit**, **Submissions**, **Publish/Unpublish**,
  and **Delete** buttons. Nothing opens automatically — you choose what to look at.
- **+ New blank form** starts an empty form; **⚡ PAN Card quick start** instantly
  generates a ready-made PAN Card form (see below) that you can review and tweak.
- **Form Builder** is now compact: clicking "+ Text/Choice/Document field" adds the
  field, scrolls straight to it, and pre-selects its label text so you can just
  start typing the new name immediately — no more hunting for it further down the
  page. Document-field settings (format, size, width) are tucked behind an
  "⚙ Upload & compression settings" toggle so the list stays tidy.
- **Submissions** is scoped to one form at a time (pick it from Manage Forms) —
  no more giant combined table.

### Customers: pick a form, don't get dropped into one
- After signing in, customers land on an **Available forms** gallery — nothing
  opens by itself. A sticky **forms nav bar** underneath the header also lists
  every published form as a pill, so switching between multiple forms (e.g. PAN
  Card and Voter ID) never requires scrolling to find them.
- Text inputs are compact (not full-width) so the form doesn't feel oversized;
  they only stretch full-width where it makes sense (e.g. long descriptions).

## 6. The PAN Card quick start, in detail

Clicking **⚡ PAN Card quick start** creates a form with:

| Field | Behavior |
|---|---|
| Applicant's Full Name | Splits into First / Middle / Last inputs; First & Last are required |
| Phone Number | Exactly 10 digits, auto-spaced as `XXXXX XXXXX` on screen; stored/copied without the space |
| Email Address | Validated email format |
| Aadhaar Number | Exactly 12 digits, auto-spaced in groups of 4; stored/copied without spaces |
| Application Type | New PAN Card / Correction / Change |
| Upload Photo | JPEG, exactly 2.5×3.5cm at 200 DPI (197×276px), max 50KB |
| Upload Signature | JPEG, exactly 4.5×2cm at 200 DPI (354×157px), max 50KB |
| Upload Supporting Document | PDF, max 300KB — label switches automatically between "Proof of Date of Birth" (New) and "PAN Copy / Acknowledgement Receipt" (Correction) based on the Application Type answer |

**No stretching:** the Photo and Signature crop boxes are locked to the exact
aspect ratio of their required output size, so whatever the user crops is resized
to precisely 197×276px / 354×157px without distorting it — the same guarantee
tools like pancardresizer.com give, just built in.

**The supporting document is a PDF, not an image** — there's no cropping step for
it. The app checks it's actually a PDF and under 300KB, then stores it directly;
there's no client-side PDF compression library included, so if a file is too big
the user is asked to pick a smaller one.

**Voter ID / anything else:** just use "+ New blank form" and add whatever text
and document fields you need — those stay free-crop with whatever size/format you
configure, exactly like before.

## 7. Status tracking (replaces the old Approve/Reject)

Each submission now tracks two independent things:
- **Payment** — `Mark Paid` / `Mark Unpaid` toggle.
- **Application status** — `Applied` (successfully applied with the authority) or
  `Reject` (prompts you for a short reason, which is shown directly to the
  customer on their status widget so they know exactly what to fix).

Customers see both badges live, plus a banner explaining a rejection or confirming
an applied status — no need to guess what's going on.

## 8. Mandatory fields & formatting

Every field is required. Predefined field types handle their own formatting and
validation automatically:
- **Full name** → First/Middle/Last, First & Last required.
- **Phone** → must be exactly 10 digits (space shown after the 5th digit on
  screen only; the stored value and anything copied via the ⧉ button is the
  raw 10-digit number with no space).
- **Aadhaar number** → must be exactly 12 digits (shown grouped in 4s; stored/copied raw).
- **Email** → standard email format check.
- **Custom text** → just required, no extra formatting.

If anything is missing or invalid, the field is highlighted with an inline error
message and the form scrolls to the first problem when Submit is pressed.

## 9. Installing the app / favicon

The site now has a proper "SS" monogram favicon (`icon.svg`) and is installable:
- **Android/desktop Chrome:** look for the install icon in the address bar, or the
  browser's "Install app" / "Add to Home screen" menu option.
- **iPhone/iPad Safari:** Share → Add to Home Screen (uses `icon.svg` via the
  apple-touch-icon tag).

This works because of `manifest.webmanifest` + `sw.js` sitting alongside
`index.html` — make sure all four files stay in the same folder when you deploy.

## Notes & limitations to know about

- This is a client-only app — the real security boundary is `firestore.rules`.
  Double-check the UID/email inside it match your admin account.
- Firestore caps each document at 1MiB; Base64 inflates the original file size by
  about 33%. Keep "Max size (KB)" conservative on any form with several document
  fields (the built-in PAN spec — two 50KB images + one 300KB PDF — comfortably fits).
- There's no PDF compression on the client (no library included for that), so
  oversized PDFs are rejected with a message rather than silently shrunk.
- The Admin UID/email check is client-side for routing convenience only — access
  is actually enforced by `firestore.rules`.
