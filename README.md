# Adscom — Version 2 (accounts + database connection)

This package contains the Adscom responsive homepage plus a Supabase-ready account and listing system.

## 1. Preview the homepage
Open `index.html` in a browser. Without Supabase configuration, it runs in demo mode:
- Search and category filtering work.
- Sample advertisements appear.
- The post-ad form adds a listing for the current page session only.
- Login and signup show setup instructions.

## 2. Create the free backend project
1. Go to https://supabase.com/ and create an account.
2. Create a new project. Choose a strong database password and save it somewhere private.
3. In the Supabase project, open **Project Settings → API** (or **Data API / API Keys**, depending on the dashboard version).
4. Copy the **Project URL** and the **anon/public key**. Do not use the `service_role` or secret key in website code.
5. Open `index.html` in a code editor and replace:
   - `PASTE_YOUR_SUPABASE_PROJECT_URL_HERE`
   - `PASTE_YOUR_SUPABASE_ANON_PUBLIC_KEY_HERE`

## 3. Create the ads table
In Supabase, open **SQL Editor → New query**, paste this SQL, and run it:

```sql
create table if not exists public.ads (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references auth.users(id) on delete cascade,
  title text not null check (char_length(title) between 5 and 70),
  category text not null,
  price text,
  location text not null,
  contact text not null,
  description text not null check (char_length(description) between 10 and 500),
  status text not null default 'pending' check (status in ('pending', 'approved', 'rejected')),
  created_at timestamptz not null default now()
);

alter table public.ads enable row level security;

drop policy if exists "Anyone can read approved ads" on public.ads;
create policy "Anyone can read approved ads"
on public.ads for select
using (status = 'approved');

drop policy if exists "Users can submit their own ads" on public.ads;
create policy "Users can submit their own ads"
on public.ads for insert
to authenticated
with check (auth.uid() = user_id and status = 'pending');
```

This policy means new advertisements are pending and are not publicly visible until reviewed and approved. The admin approval screen is a later development step. Do not disable row-level security.

## 4. Test accounts
1. Host the files on a local development server or a static host (for example, Vercel or Netlify). Authentication redirects work best when the site is hosted over HTTPS.
2. In Supabase, open **Authentication → URL Configuration** and add your website URL to the Site URL / Redirect URLs.
3. Create a test account through Adscom.
4. Log in and submit an advertisement.
5. In Supabase **Table Editor → ads**, check that the new record is present with status `pending`.

## Important security and launch notes
- The browser code uses only the Supabase anon/public key. Never put a Supabase service-role key or database password in `index.html`.
- The current demo contact data is not displayed in public listing cards. Do not publish personal phone numbers or email addresses without consent.
- This is an early MVP, not yet ready to accept payments or unrestricted public advertising.
- Before launch, add admin moderation tools, image upload with file-size/type limits, spam protection, terms/privacy pages, abuse reporting, and backups.
- For worldwide payments, select a payment provider that supports the countries and currencies you intend to serve. Payment credentials must be handled server-side, not in browser JavaScript.
