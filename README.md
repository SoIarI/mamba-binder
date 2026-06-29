# Binder

A personal project bookmark manager — single HTML file, no build step, no framework. Backed by Supabase for auth, storage, and realtime sync.

---

## Supabase Setup

### 1. Create a Supabase project

Go to [supabase.com](https://supabase.com), create a new project, and wait for it to provision.

### 2. Run this SQL in the SQL Editor

In your Supabase dashboard → **SQL Editor** → **New query**, paste and run:

```sql
create table projects (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users(id) on delete cascade not null,
  name text not null,
  description text,
  url text not null,
  tags text[] default '{}',
  pinned boolean default false,
  created_at timestamptz default now()
);

alter table projects enable row level security;

create policy "Users can only access their own projects"
  on projects for all
  using (auth.uid() = user_id)
  with check (auth.uid() = user_id);
```

### 3. Get your URL and anon key

In your Supabase dashboard → **Project Settings** → **API**:

- Copy **Project URL** (e.g. `https://abcxyz.supabase.co`)
- Copy **anon / public** key

### 4. Paste them into `binder.html`

Open `binder.html` in a text editor and find these two lines near the top of the `<script>` block:

```js
const SUPABASE_URL = 'YOUR_SUPABASE_URL';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

Replace both placeholder strings with your actual values.

---

## How to Use

Open `binder.html` directly in any modern browser — no server required.

To host it permanently, drop the file on any static host (GitHub Pages, Vercel, Netlify, Cloudflare Pages, etc.).

**First launch:** Sign up for an account. On your first login with zero projects, a set of seed projects will be inserted automatically.

### Managing projects

| Action | How |
|--------|-----|
| Add    | Click the **+** button (bottom-right) |
| Edit   | Click **Edit** on any card |
| Delete | Click **Delete** on any card (confirmation required) |
| Open   | Click **Open** to open the project URL in a new tab |
| Pin    | Click **Pin / Unpin** on any card |

### Search & filter

- **Search bar** (header): filters by name, description, and tags instantly
- **Tag pills** (below header): click any tag to filter; multi-select uses OR logic; click **All** to clear

---

## Export / Import

### Export

Click **Export** in the header. Downloads `binder-export.json` containing all your projects as a JSON array.

### Import

Click **Import** in the header and select a `.json` file (must be a JSON array).

- If a project with the same **name + URL** already exists → it is **updated**
- Otherwise → it is **inserted**

Use export/import to sync your project list across devices or back it up locally.

---

## Realtime

Changes you make in one browser tab are reflected in all other open tabs automatically via Supabase Realtime — no refresh needed.
