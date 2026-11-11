# Block Together: Minecraft Education Multiplayer Community

A static website (HTML/CSS/JS) hosted on **GitHub Pages**, with **Supabase** (Auth + PostgreSQL + Row Level Security) as the backend. Players sign up, post Minecraft Education multiplayer join codes, browse other players' worlds and copy codes to join.

Pages: Landing, Sign Up, Login, Dashboard, Browse Worlds, Post a World, My Profile.

## Project structure

```
index.html  signup.html  login.html  dashboard.html
browse.html post.html    profile.html
css/styles.css
js/config.js     <- your public Supabase URL + anon key go here
js/app.js        <- shared: Supabase client, auth guard, navbar, toasts, cards
js/pages.js      <- per-page logic
supabase/schema.sql  <- tables, trigger, RLS policies
.nojekyll
```

No build step is needed. GitHub Pages serves these files as they are.

## 1. Set up Supabase

1. Create a free account at <https://supabase.com> and click **New project**. Pick a name, a database password (save it somewhere; the website never needs it) and a region.
2. When the project is ready, open **SQL Editor -> New query**, paste the entire contents of [`supabase/schema.sql`](supabase/schema.sql) and click **Run**. This creates the `profiles` and `worlds` tables, the sign-up trigger, and all RLS policies.
3. Open **Project Settings -> API** and copy:
   - **Project URL** (looks like `https://abcdxyz.supabase.co`)
   - **anon / publishable key** (the `anon` `public` key)
4. Paste both into [`js/config.js`](js/config.js):
   ```js
   window.MCE_CONFIG = {
     SUPABASE_URL: "https://abcdxyz.supabase.co",
     SUPABASE_ANON_KEY: "eyJ...your anon key...",
   };
   ```
   These two values are meant to be public. Your data is protected by the RLS policies, not by hiding the key.
   **Never** paste the `service_role` key (or any "secret" key) anywhere in this project.
5. Configure Auth redirects. Go to **Authentication -> URL Configuration**:
   - **Site URL**: your published URL, e.g. `https://YOUR-USERNAME.github.io/YOUR-REPO/`
   - **Redirect URLs**: add the same URL with `**` on the end, e.g. `https://YOUR-USERNAME.github.io/YOUR-REPO/**` (and `http://localhost:8000/**` if you test locally).
6. Email confirmation (your choice), under **Authentication -> Providers -> Email**:
   - **Confirm email ON** (the Supabase default): after signing up, users get an email link and must confirm before logging in. The site shows a "check your email" message. Supabase's built-in email sender is rate-limited, so for a real community configure your own SMTP under Authentication -> SMTP Settings.
   - **Confirm email OFF**: users are logged in immediately after sign-up. Easiest for testing.

## 2. Test locally (optional)

From the project folder:

```bash
python -m http.server 8000
```

Open <http://localhost:8000>. (Opening the HTML files directly by double-click also mostly works, but a local server is closer to GitHub Pages.)

## 3. Publish to GitHub Pages

1. Create a repository on GitHub (for example `minecraft-edu-community`). Public is required for Pages on a free account.
2. In this project folder, run:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPO.git
   git push -u origin main
   ```
3. On GitHub open the repo -> **Settings -> Pages**. Under **Build and deployment**, set **Source** to **Deploy from a branch**, choose branch **main** and folder **/ (root)**, then **Save**.
4. After a minute or two your site is live at `https://YOUR-USERNAME.github.io/YOUR-REPO/`.
5. Make sure that exact URL is set as the **Site URL** in Supabase (step 1.5), otherwise email-confirmation links will point to the wrong place.

All links in the site are relative, so it works under the `/YOUR-REPO/` sub-path. To update the site later, commit and push; Pages redeploys automatically.

## How security works

- **Auth**: Supabase Auth handles passwords (hashed server-side) and issues JWTs; the session persists in the browser and refreshes automatically. Pages other than landing/login/sign-up redirect to Login when there is no session.
- **RLS** (see `supabase/schema.sql`):
  - Anonymous visitors cannot read or write anything.
  - Signed-in users can read all worlds and profiles (needed to show creator usernames).
  - Users can insert worlds only as themselves, and update/delete only rows where `user_id = auth.uid()`. This is enforced by the database, not just by the hidden buttons in the UI.
  - Profiles can't be modified by clients; they are created by a database trigger at sign-up.
- The only key in the frontend is the public anon key. The browser-side route guards are a convenience; RLS is the real protection.

## Notes

- The site is for **Minecraft Education** only. Users enter their own real join codes; there is no demo data.
- Usernames are 3-20 characters (letters, numbers, underscore) and unique case-insensitively.
- Join codes are stored as plain text exactly as the user pastes them.
