# TinkerHub JCET — Setup Guide

This is your complete walkthrough: setting up the free backend (Supabase), deploying the site, and using the admin dashboard to add events and photos afterward.

---

## Part 1: Create your Supabase project

1. Go to **supabase.com** → sign up (Google login works fine) → click **New project**
2. Give it a name (e.g. `tinkerhub-jcet`)
3. Set a database password — just note it somewhere safe, you won't need it day-to-day
4. Pick a region close to you
5. Click **Create new project** — takes about 2 minutes to spin up

---

## Part 2: Connect the site to your project

1. Once your project is ready, go to **Project Settings → API**
2. Copy the **Project URL** and the **anon public** key
3. Open the file `assets/js/supabase-config.js` in your site folder
4. Replace the two placeholder values with what you copied:
   ```js
   export const supabaseUrl = "https://your-project-id.supabase.co";
   export const supabaseAnonKey = "your-anon-key-here";
   ```
5. Save the file

---

## Part 3: Create your admin login

1. Left sidebar → **Authentication → Users** → **Add user**
2. Enter your email and a strong password
3. Toggle **"Auto Confirm User"** ON (so you don't need to click a confirmation email)
4. Click **Create user**

This is what you'll log in with at `/admin` — keep it safe, it's the real front door now.

---

## Part 4: Create the database tables

1. Left sidebar → **SQL Editor** → **New query**
2. Paste this and click **Run**:

```sql
create table events (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  category text not null,
  mode text not null,
  date date not null,
  description text,
  created_at timestamptz default now()
);
alter table events enable row level security;
create policy "Public can read events" on events for select using (true);
create policy "Authenticated can insert events" on events for insert with check (auth.role() = 'authenticated');
create policy "Authenticated can delete events" on events for delete using (auth.role() = 'authenticated');

create table gallery (
  id uuid primary key default gen_random_uuid(),
  src text not null,
  storage_path text,
  caption text,
  created_at timestamptz default now()
);
alter table gallery enable row level security;
create policy "Public can read gallery" on gallery for select using (true);
create policy "Authenticated can insert gallery" on gallery for insert with check (auth.role() = 'authenticated');
create policy "Authenticated can delete gallery" on gallery for delete using (auth.role() = 'authenticated');
```

---

## Part 4b: Team & Impact tables (for the Team and Community Impact sections)

Run this in the same **SQL Editor**:

```sql
create table team (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  role text not null,
  photo text not null,
  storage_path text,
  linkedin text,
  created_at timestamptz default now()
);
alter table team enable row level security;
create policy "Public can read team" on team for select using (true);
create policy "Authenticated can insert team" on team for insert with check (auth.role() = 'authenticated');
create policy "Authenticated can delete team" on team for delete using (auth.role() = 'authenticated');

create table impact (
  id uuid primary key default gen_random_uuid(),
  icon text,
  title text not null,
  description text,
  tag text,
  created_at timestamptz default now()
);
alter table impact enable row level security;
create policy "Public can read impact" on impact for select using (true);
create policy "Authenticated can insert impact" on impact for insert with check (auth.role() = 'authenticated');
create policy "Authenticated can delete impact" on impact for delete using (auth.role() = 'authenticated');
```

Then create a second storage bucket for team photos:
1. **Storage → New bucket** → name it exactly `team` → toggle **Public bucket** ON → **Create bucket**
2. Back in **SQL Editor**, run:
```sql
create policy "Public can view team images" on storage.objects for select using (bucket_id = 'team');
create policy "Authenticated can upload team images" on storage.objects for insert with check (bucket_id = 'team' and auth.role() = 'authenticated');
create policy "Authenticated can delete team images" on storage.objects for delete using (bucket_id = 'team' and auth.role() = 'authenticated');
```

**Note:** the site shows the original founding team and default impact cards automatically until you add real entries via `/admin` — nothing breaks or looks empty in the meantime. Once you add even one team member or impact stat through the dashboard, it replaces the defaults on the live site.

---

## Part 4c: Materials & Projects tables (Study Materials + Featured Projects)

Run this in the **SQL Editor**:

```sql
create table materials (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  link text not null,
  description text,
  created_at timestamptz default now()
);
alter table materials enable row level security;
create policy "Public can read materials" on materials for select using (true);
create policy "Authenticated can insert materials" on materials for insert with check (auth.role() = 'authenticated');
create policy "Authenticated can delete materials" on materials for delete using (auth.role() = 'authenticated');

create table projects (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  description text,
  link text,
  icon text,
  created_at timestamptz default now()
);
alter table projects enable row level security;
create policy "Public can read projects" on projects for select using (true);
create policy "Authenticated can insert projects" on projects for insert with check (auth.role() = 'authenticated');
create policy "Authenticated can delete projects" on projects for delete using (auth.role() = 'authenticated');
```

No storage bucket needed for these two — materials are just links (Google Drive, PDFs hosted elsewhere, etc.) and projects link out to GitHub/demos rather than uploading files here.

---

## Part 5: Create the photo storage bucket

1. Left sidebar → **Storage** → **New bucket**
2. Name it exactly `gallery` (lowercase, must match)
3. Toggle **Public bucket** ON
4. Click **Create bucket**
5. Back in **SQL Editor**, run this:

```sql
create policy "Public can view gallery images" on storage.objects for select using (bucket_id = 'gallery');
create policy "Authenticated can upload gallery images" on storage.objects for insert with check (bucket_id = 'gallery' and auth.role() = 'authenticated');
create policy "Authenticated can delete gallery images" on storage.objects for delete using (bucket_id = 'gallery' and auth.role() = 'authenticated');
```

---

## Part 6: Deploy the site

Your site is a folder of files — no build step needed. Pick one:

**Vercel (recommended, drag-and-drop, no account setup needed beyond signing in)**
1. Go to **vercel.com/drop**
2. Drag your whole `tinkerhub-jcet` folder onto the page
3. Choose a team + project name → **Deploy**
4. You get a live URL in seconds

**Netlify (also drag-and-drop)**
1. Go to **netlify.com**, sign up, go to your dashboard
2. Find **"Deploy manually"** / drag-and-drop zone
3. Drag your `tinkerhub-jcet` folder in
4. Live URL appears immediately

Either one works the same way going forward — whenever you have updated files, just drag the folder in again to redeploy.

---

## Part 7: Test the admin dashboard

1. Visit `yoursite.com/admin`
2. Log in with the email + password from **Part 3**
3. Try adding a test event — fill the form, click **Add Event**
4. Open your actual site's Events page in another tab — the event should appear within seconds, no redeploy needed
5. Same idea for **Gallery** — upload a photo, check it shows up on the homepage Gallery section

---

## Your two editing workflows, going forward

**Adding/removing events or gallery photos** → just use `/admin`. Live immediately, no code, no redeploy.

**Changing anything else** (wording, sections, team members, colors, links, layout) → this lives directly in the HTML/CSS files, not the database. Either:
- Ask me to make the change and hand you updated files, or
- Edit the files yourself in VS Code

Either way, after a code change, **redeploy** by dragging the updated folder onto Vercel or Netlify again — takes seconds, no downtime.

---

## Quick troubleshooting

| Problem | Likely cause |
|---|---|
| `/admin` shows a "Setup needed" warning | `supabase-config.js` still has placeholder values — check Part 2 |
| Login fails | Double check the email/password from Part 3, or that "Auto Confirm User" was toggled on |
| "Error saving" when adding an event | RLS policies from Part 4 weren't run, or ran with an error — check the SQL Editor for errors |
| Photo upload fails | Bucket name isn't exactly `gallery`, or the Part 5 storage policies weren't run |
| Site was fine, now Events/Gallery are empty | Your Supabase project may have auto-paused after 7 days of no activity — open your Supabase dashboard and click "Restore" |
