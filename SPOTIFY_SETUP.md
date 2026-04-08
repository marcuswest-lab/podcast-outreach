# Spotify API Setup Guide

## Overview
The Spotify API allows us to pull all episodes from LaWayra's podcast and seed podcasts for guest extraction.

**Cost:** Free
**Time to complete:** ~5 minutes

---

## Step 1: Create Spotify Developer Account

1. Go to [Spotify for Developers Dashboard](https://developer.spotify.com/dashboard)
2. Log in with your Spotify account (or create one if needed)
3. Accept the Terms of Service

---

## Step 2: Create New Application

1. Click **"Create App"** button
2. Fill in the application details:
   - **App Name:** `LaWayra Podcast Outreach Automation`
   - **App Description:** `Automated podcast guest outreach system for LaWayra`
   - **Website:** `https://ayahuascapodcast.com` (optional)
   - **Redirect URI:** `http://localhost:5678/rest/oauth2-credential/callback` (for N8N)
     - Click **"Add"** after entering the redirect URI
   - **API/SDKs:** Check "Web API"
3. Accept Spotify's Developer Terms of Service
4. Click **"Save"**

---

## Step 3: Get Your Credentials

After creating the app, you'll be taken to the app dashboard:

1. Click on **"Settings"** button
2. Copy your credentials:
   - **Client ID** - Copy this (looks like: `abc123def456...`)
   - **Client Secret** - Click "View client secret", then copy

**⚠️ Important:** Keep your Client Secret private. Never share it publicly or commit it to git.

---

## Step 4: Find LaWayra's Spotify Show ID

### Option A: Via Spotify Web Player (Easiest)

1. Go to [Spotify Web Player](https://open.spotify.com)
2. Search for "Ayahuasca Podcast" or "LaWayra"
3. Click on your podcast
4. Click the **"..." menu** → **"Share"** → **"Copy Show Link"**
5. The URL looks like: `https://open.spotify.com/show/1A2B3C4D5E`
6. The Show ID is the part after `/show/`: `1A2B3C4D5E`

### Option B: Via Spotify API (If podcast not on Spotify yet)

If LaWayra isn't on Spotify yet, you'll need to:
1. Submit the podcast RSS feed to Spotify
2. Or use YouTube API only for now (skip Spotify in workflows)

---

## Step 5: Find Seed Podcasts' Spotify Show IDs

Repeat the process from Step 4 for each seed podcast:

1. Search for "Psychedelics Today" on Spotify
2. Copy Show Link → Extract Show ID
3. Add to `seed-podcasts.json`

**Example Seed Podcasts:**
- Psychedelics Today
- The Third Wave Podcast
- MAPS Podcast
- Plant Medicine Podcast

---

## Step 6: Add Credentials to Configuration File

1. Open `/Users/marcusawakuni/Desktop/LW/podcast-outreach/api-credentials.env`
2. Fill in your Spotify credentials:

```bash
SPOTIFY_CLIENT_ID=abc123def456...
SPOTIFY_CLIENT_SECRET=xyz789uvw012...
SPOTIFY_LAWAYRA_SHOW_ID=1A2B3C4D5E
```

---

## Step 7: Test API Connection (Optional but Recommended)

### Test with curl command:

```bash
# Get access token
curl -X POST "https://accounts.spotify.com/api/token" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "grant_type=client_credentials&client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET"

# You should receive a JSON response with "access_token"
```

### Test getting podcast episodes:

```bash
# Replace {ACCESS_TOKEN} and {SHOW_ID} with your values
curl -X GET "https://api.spotify.com/v1/shows/{SHOW_ID}/episodes" \
     -H "Authorization: Bearer {ACCESS_TOKEN}"

# You should see a list of episodes
```

If both commands return data successfully, your Spotify API setup is complete!

---

## Step 8: Configure in N8N

When you import the N8N workflows:

1. Open workflow `1_LaWayra_Guest_Extraction.json`
2. Find the **"Spotify"** node
3. Click **"Credentials"** → **"Create New"**
4. Select **"Spotify OAuth2 API"**
5. Enter:
   - **Client ID:** From api-credentials.env
   - **Client Secret:** From api-credentials.env
6. Click **"Connect my account"** and authorize
7. Save credentials as "Spotify - LaWayra Podcast Outreach"

---

## API Limits & Quotas

**Spotify Web API:**
- **Rate Limit:** ~180 requests per minute (generous)
- **Cost:** Free
- **Quota:** No daily limits for our use case

For our automation:
- Fetching all LaWayra episodes: ~1-5 API calls (depending on pagination)
- Fetching seed podcast episodes: ~1-5 calls each
- **Total:** ~10-50 calls per day (well within limits)

---

## Troubleshooting

### Issue: "Invalid redirect URI"
**Solution:** Make sure you added exactly `http://localhost:5678/rest/oauth2-credential/callback` in your Spotify app settings.

### Issue: "Invalid client"
**Solution:** Double-check your Client ID and Client Secret. They should have no spaces or extra characters.

### Issue: "Show not found"
**Solution:** Verify the Show ID is correct. Try searching on Spotify web player and copying the link again.

### Issue: LaWayra podcast not on Spotify yet
**Solution:**
1. Submit podcast RSS feed to Spotify for Podcasters
2. Or skip Spotify in Funnel A workflow and use YouTube only
3. Can still use Spotify for Funnel B seed podcasts

---

## Next Steps

✅ Spotify API credentials obtained
✅ LaWayra Show ID found (or noted if not on Spotify)
✅ Seed podcast Show IDs added to `seed-podcasts.json`

**Continue to:** [YOUTUBE_SETUP.md](YOUTUBE_SETUP.md)
