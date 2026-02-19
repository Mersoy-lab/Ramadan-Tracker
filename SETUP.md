# 🌙 Ramadan Tracker 1447 — Setup Guide

> Deploy in ~15 minutes. Your users sign in with Google and their 30-day data syncs across every device.

---

## Overview

| Layer | Service | Cost |
|---|---|---|
| Frontend + Hosting | Vercel | Free |
| Database + Auth | Supabase | Free (up to 50k users) |
| Login | Google OAuth | Free |

---

## Step 1 — Set up Supabase (5 min)

### 1a. Create project
1. Go to **https://supabase.com** → Sign up → New Project
2. Name it `ramadan-tracker-2026`
3. Set a strong database password (save it)
4. Choose a region close to your users
5. Click **Create new project** — wait ~2 minutes

### 1b. Create the database table
Once your project loads, go to **SQL Editor** (left sidebar) and run this:

```sql
-- Create the main tracking table
CREATE TABLE ramadan_days (
  id           UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id      UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  year         INTEGER NOT NULL DEFAULT 2026,
  day_number   INTEGER NOT NULL CHECK (day_number BETWEEN 1 AND 30),
  data         JSONB NOT NULL DEFAULT '{}',
  updated_at   TIMESTAMPTZ DEFAULT now(),
  UNIQUE(user_id, year, day_number)
);

-- Enable Row Level Security (users can only see their own data)
ALTER TABLE ramadan_days ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can read their own days"
  ON ramadan_days FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert their own days"
  ON ramadan_days FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update their own days"
  ON ramadan_days FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete their own days"
  ON ramadan_days FOR DELETE
  USING (auth.uid() = user_id);

-- Enable real-time updates
ALTER PUBLICATION supabase_realtime ADD TABLE ramadan_days;
```

Click **Run** ✅

### 1c. Get your API keys
Go to **Settings → API** and copy:
- **Project URL** (looks like `https://xxxxxxxxxxxx.supabase.co`)
- **anon public key** (long string starting with `eyJ...`)

---

## Step 2 — Set up Google OAuth (5 min)

### 2a. Create Google OAuth credentials
1. Go to **https://console.cloud.google.com**
2. Create a new project (or use existing)
3. Go to **APIs & Services → Credentials**
4. Click **Create Credentials → OAuth Client ID**
5. Application type: **Web application**
6. Name it `Ramadan Tracker`
7. Under **Authorized redirect URIs**, add:
   ```
   https://YOUR_SUPABASE_PROJECT_REF.supabase.co/auth/v1/callback
   ```
   (Replace `YOUR_SUPABASE_PROJECT_REF` with your project ref — visible in your Supabase URL)
8. Click **Create** — copy the **Client ID** and **Client Secret**

### 2b. Add Google to Supabase
1. In Supabase, go to **Authentication → Providers**
2. Find **Google** and enable it
3. Paste in your **Client ID** and **Client Secret**
4. Click **Save**

---

## Step 3 — Deploy to Vercel (5 min)

### 3a. Put code on GitHub
1. Create a new repository at **https://github.com/new**
2. Name it `ramadan-tracker-2026`
3. Upload all the project files (drag & drop or use git)

### 3b. Deploy on Vercel
1. Go to **https://vercel.com** → Sign up with GitHub
2. Click **Add New Project**
3. Import your `ramadan-tracker-2026` repository
4. Under **Environment Variables**, add:
   ```
   NEXT_PUBLIC_SUPABASE_URL     = https://your-project.supabase.co
   NEXT_PUBLIC_SUPABASE_ANON_KEY = eyJ...your-anon-key...
   ```
5. Click **Deploy** — Vercel builds and gives you a URL like:
   `https://ramadan-tracker-2026.vercel.app`

### 3c. Add your Vercel URL to Google & Supabase
1. Back in **Google Cloud Console → OAuth Credentials**, add to **Authorized JavaScript Origins**:
   ```
   https://ramadan-tracker-2026.vercel.app
   ```
   And to **Authorized redirect URIs**:
   ```
   https://ramadan-tracker-2026.vercel.app/auth/callback
   ```

2. In **Supabase → Authentication → URL Configuration**, add:
   - **Site URL**: `https://ramadan-tracker-2026.vercel.app`
   - **Redirect URLs**: `https://ramadan-tracker-2026.vercel.app/auth/callback`

---

## Step 4 — Share with your community 🌙

Your app is live! Share the Vercel URL:
```
https://ramadan-tracker-2026.vercel.app
```

Users:
1. Visit the link on any device (phone, laptop, tablet)
2. Tap **Continue with Google**
3. Start tracking their prayers, adhkar, and good deeds
4. Open on any other device — data is already there ✅

---

## Optional: Custom domain

If you want `ramadan.yourschool.org` instead of the Vercel URL:
1. In Vercel → your project → **Settings → Domains**
2. Add your domain and follow the DNS instructions

---

## How the sync works

```
User logs in on iPhone
  → Supabase stores their Day 1 prayer data
  
Same user opens laptop
  → App fetches all 30 days from Supabase
  → Sees Day 1 data instantly ✅

User taps Fajr on phone while laptop is open
  → Real-time subscription pushes update to laptop
  → Both devices show the same data in <1 second ✅
```

---

## App Features Summary

| Tab | What it tracks |
|---|---|
| 🕌 Salah | 5 obligatory + Duha + Tarawih, with 5-level Namaz Rubric quality rating |
| 📿 Adhkar | 10 adhkar counted to 100× each |
| 📖 Soul | Daily Juz + kindness self-score + Laylatul Qadr dua (last 10 days) |
| 📅 Calendar | Full 30-day grid with color-coded quality bars, last 10 nights highlighted |
| ⭐ Score | Daily score 0–100, breakdown, shareable text card |

---

## Troubleshooting

**"Invalid redirect URI" error on Google sign-in**
→ Check that your Vercel URL is in both Google OAuth and Supabase redirect URLs

**Data not syncing**
→ Make sure `supabase_realtime` publication was added in the SQL step

**Build fails on Vercel**
→ Check that both environment variables are set correctly (no extra spaces)

---

*Ramadan Mubarak — may Allah accept your fasts and good deeds* 🤲
