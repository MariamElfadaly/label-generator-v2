# Employee Label Registry

A single-file, browser-only tool for creating, storing, and printing bilingual
(Arabic/English) employee ID labels (50mm × 25.4mm) with a QR code. Built to
follow the same no-build, GitHub Pages workflow as Locker Room and Employee
File Tracker.

## What it does

- **Add employee** — one-by-one entry form, saves straight to Firestore.
- **Bulk import** — upload an Excel/CSV file to add or update many employees
  at once, with a preview table before committing.
- **Registry** — every saved label in one searchable table. Preview, edit,
  delete, or download any label as a PDF.
- **Download all as PDF** — combines every (or every filtered) label into one
  multi-page, print-ready PDF, sized exactly to each label.

No build step, no npm, no terminal — it's one `index.html` file. Open it
locally to test, or push it to a GitHub Pages repo the same way as your other
lab tools.

## One-time setup (Firebase)

You can reuse your existing Locker Room Firebase project — see the note at
the bottom. Steps below assume a fresh setup.

1. **Open your Firebase project.**
   Go to [console.firebase.google.com](https://console.firebase.google.com)
   and open the project you want to use.

2. **Enable Firestore.**
   Build → Firestore Database → Create database. Choose **Production mode**
   and a region close to Egypt.

3. **Enable Email/Password authentication.**
   Build → Authentication → Get started → Sign-in method tab → enable
   **Email/Password**.

4. **Create one shared login user.**
   Authentication → Users tab → Add user. This one email/password is what
   everyone in the lab uses to sign into the app.

5. **Get your web app config.**
   Project settings (gear icon) → scroll to "Your apps" → click the web icon
   (`</>`) to register a new app (or open the existing one) → copy the
   `firebaseConfig` object shown.

6. **Paste the config into `index.html`.**
   Open the file, find the `firebaseConfig` object near the top of the
   `<script type="module">` section, and replace the `YOUR_...` placeholders.

7. **Set the Firestore security rule.**
   Firestore Database → Rules:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /labels/{id} {
         allow read, write: if request.auth != null;
       }
     }
   }
   ```

   Click **Publish**.

8. **Restrict the API key (recommended).**
   Google Cloud Console → APIs & Services → Credentials → your Firebase API
   key → set Application restrictions to **HTTP referrers**, and add your
   GitHub Pages URL, e.g. `mariamelfadaly.github.io/*`.

9. **Deploy and test.**
   Push `index.html` to your GitHub Pages repo. Open the live URL, sign in,
   add a test employee, confirm it saves and the PDF downloads correctly.

### Reusing the Locker Room Firebase project instead

Skip step 1. In step 7, **add** the `labels` match block next to your
existing rules — don't replace the whole rules file, or Locker Room will
lose its access:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /lockers/{id} {              // your existing Locker Room rule
      allow read, write: if request.auth != null;
    }
    match /labels/{id} {               // new — for this app
      allow read, write: if request.auth != null;
    }
  }
}
```

Check Firestore's **Data** tab first if you're not sure what your existing
collection is actually called. You can reuse the same shared login user for
both tools, or add a second one in step 4 if you want separate access.

## Bulk import format

Excel or CSV, header row required, columns in any order:

| ID  | Name EN      | Name AR                          | Title EN    | Title AR      | QR Link                    |
|-----|--------------|-----------------------------------|-------------|---------------|-----------------------------|
| 181 | Mahmoud Atia | محمود احمد محمود عطيه قاعود        | Lab Chemist | كميائى معمل    | https://drive.google.com/...|

Existing IDs are **updated**; new IDs are **added**. Rows missing any field
are skipped and flagged in the preview before you confirm the import.

## Notes / limitations

- QR codes are generated via `api.qrserver.com` at export time, so PDF
  generation (single or "download all") needs an internet connection.
- "Download all as PDF" respects the current search filter — filter first if
  you only want a subset.
- Firestore's free tier allows 50K reads / 20K writes per day, shared across
  every collection in the project (including Locker Room if reused). Normal
  day-to-day use here is far below that.
