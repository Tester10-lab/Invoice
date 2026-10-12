# Lalitpur Holidays — Invoice Generator

A single-page invoice app for a travel company. It works in two modes:

- **Offline** — double-click `index.html`. Everything is stored in the browser (localStorage). No internet needed.
- **Online (shared database)** — the same page, deployed on Vercel, syncing invoices, the company profile and the package catalog to a Supabase Postgres database, with one login per staff member.

The app detects the mode automatically: if `config.js` has Supabase keys, a sync chip and a **Sign in** button appear in the toolbar; if the keys are empty it behaves exactly like the offline file.

---

## 1. Set up the database (Supabase) — ~5 minutes

1. Go to [supabase.com](https://supabase.com) and open your project (or create one — the free tier is enough).
2. In the left sidebar open **SQL Editor** → **New query**, paste the whole contents of [`supabase-setup.sql`](supabase-setup.sql) and click **Run**. This creates the tables and security rules.
3. Create staff accounts: **Authentication → Users → Add user → Create new user**. Enter each staff member's email and a password, and tick **Auto Confirm User**. Repeat for every staff member.
4. Stop public sign-ups: **Authentication → Sign In / Providers → Email** and turn **off** "Allow new users to sign up" (name may vary slightly in the dashboard). Now only the accounts you created can log in.
5. Copy the connection values: **Project Settings (⚙) → API**:
   - **Project URL**
   - **Project API keys → `anon` / `public`**

## 2. Connect the app — 1 minute

Open [`config.js`](config.js) in this folder and paste the two values:

```js
window.SUPABASE_URL = 'https://YOUR-PROJECT.supabase.co';
window.SUPABASE_ANON_KEY = 'eyJhbGciOi...';
```

(The `anon` key is safe to publish — the database rules allow access only to signed-in staff.)

## 3. Put it online (GitHub → Vercel) — ~5 minutes

1. Create a repository on [github.com](https://github.com) (e.g. `lalitpur-invoices`) and upload the four files: `index.html`, `config.js`, `supabase-setup.sql`, `README.md`. (On github.com use **Add file → Upload files** and drag the folder contents in.)
2. Go to [vercel.com](https://vercel.com) → **Add New → Project** → **Import** your new repository.
3. Framework preset: **Other** (no build settings needed) → **Deploy**.
4. Open the URL Vercel gives you (e.g. `lalitpur-invoices.vercel.app`), click **Sign in**, and log in with one of the staff accounts. The toolbar chip should turn green — **Synced**.

## 4. Bring your existing invoices over (optional)

1. Open the old **offline** file (`index.html` by double-click) → **Export** — this downloads a JSON backup.
2. On the live site, click **Import** and choose that file. Because you're signed in, everything is pushed straight into the database and appears on every staff member's device.

---

## How syncing works (short version)

- **While signed in**: every Save / Delete / company-setting / package change is written to the database immediately (the chip flashes *Syncing…* then *Synced*).
- **On sign-in / page load**: the app merges data — per invoice, the most recently updated copy wins; deletions propagate through tombstone records so nothing resurrects.
- **Signed out / no internet**: the app keeps working on this device's local copy and syncs the next time you sign in.
- The company profile and package catalog follow the database (server wins) once they exist there.

## Notes

- Staff accounts are managed in Supabase only — there is deliberately no self-signup.
- Changing a staff password: Supabase → Authentication → Users → select user.
- The JSON **Export** button always works as a manual backup of what's on the device.
- Costs: Supabase free tier and Vercel free tier (Hobby) are sufficient for a small office.
