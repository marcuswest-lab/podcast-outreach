# YouTube Data API Setup Guide

## Overview
The YouTube Data API allows us to pull all videos from LaWayra's YouTube channel to extract podcast guest names.

**Cost:** Free (with daily quotas)
**Time to complete:** ~10 minutes

---

## Step 1: Access Google Cloud Console

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Sign in with **marcusawakuni@gmail.com** (same account as N8N Google Sheets integration)
3. If this is your first time, accept the Terms of Service

---

## Step 2: Create a New Project (or Use Existing)

### Option A: Create New Project

1. Click the **project dropdown** at the top (says "Select a project")
2. Click **"NEW PROJECT"**
3. Project details:
   - **Project Name:** `LaWayra Podcast Automation`
   - **Organization:** Leave as "No organization" (or select if you have one)
4. Click **"CREATE"**
5. Wait for the project to be created (~30 seconds)
6. Select the new project from the dropdown

### Option B: Use Existing Project

If you already have a Google Cloud project for LaWayra, select it from the dropdown.

---

## Step 3: Enable YouTube Data API v3

1. In the Google Cloud Console, open the **navigation menu** (☰ icon, top left)
2. Go to **"APIs & Services"** → **"Library"**
3. In the search box, type: `YouTube Data API v3`
4. Click on **"YouTube Data API v3"**
5. Click **"ENABLE"** button
6. Wait for the API to be enabled (~10 seconds)

---

## Step 4: Create API Credentials

1. After enabling, you'll see "API enabled" confirmation
2. Click **"CREATE CREDENTIALS"** button (top right)
3. In the "Which API are you using?" dropdown: **YouTube Data API v3**
4. "What data will you be accessing?": Select **Public data**
5. Click **"NEXT"**
6. Your **API Key** will be generated automatically
7. **Copy the API key** (looks like: `AIzaSyAbc123Def456Ghi789...`)
8. Click **"DONE"**

**⚠️ Important:**
- Keep your API key private
- You can restrict the key to only YouTube Data API for security (recommended)

---

## Step 5: Restrict API Key (Recommended for Security)

1. Go to **"APIs & Services"** → **"Credentials"**
2. Find your API key in the list and click on it
3. Under **"API restrictions"**:
   - Select **"Restrict key"**
   - Check **"YouTube Data API v3"**
4. Under **"Application restrictions"** (optional):
   - Select **"IP addresses"** if you have a static IP for N8N
   - Or leave as "None" for flexibility
5. Click **"SAVE"**

---

## Step 6: Find LaWayra's YouTube Channel ID

### Option A: Via YouTube Studio (If you own the channel)

1. Go to [YouTube Studio](https://studio.youtube.com/)
2. Log in with the LaWayra channel account
3. Click on **"Settings"** (gear icon, bottom left)
4. Click **"Channel"** → **"Advanced settings"**
5. Copy the **"YouTube Channel ID"** (looks like: `UC1A2B3C4D5E6F7G8H9I0J`)

### Option B: Via Channel URL (Public method)

1. Go to the LaWayra YouTube channel: https://ayahuascapodcast.com (find YouTube link)
2. Look at the URL in your browser:
   - **New format:** `https://www.youtube.com/@ayahuascapodcast`
   - **Old format:** `https://www.youtube.com/channel/UC1A2B3C4D5E6F7G8H9I0J`
3. If you see the **@handle format**:
   - Right-click on the page → "View Page Source"
   - Search for `"channelId"` (Ctrl/Cmd + F)
   - Copy the ID (looks like: `UC1A2B3C4D5E6F7G8H9I0J`)

### Option C: Via API Test (If you can't find it)

Use this curl command with your API key:

```bash
# Replace {YOUR_API_KEY} and search for "ayahuasca podcast"
curl "https://www.googleapis.com/youtube/v3/search?part=snippet&q=ayahuasca%20podcast&type=channel&key={YOUR_API_KEY}"

# Look for "channelId" in the response
```

---

## Step 7: Test API Connection

Test that your API key works:

```bash
# Replace {YOUR_API_KEY} and {CHANNEL_ID}
curl "https://www.googleapis.com/youtube/v3/channels?part=snippet,statistics&id={CHANNEL_ID}&key={YOUR_API_KEY}"

# You should see channel info including title, description, subscriber count
```

Test fetching videos from the channel:

```bash
# Get playlist ID for "Uploads" (usually UC → UU)
# If channel ID is: UC1A2B3C4D5E6F7G8H9I0J
# Uploads playlist is: UU1A2B3C4D5E6F7G8H9I0J

curl "https://www.googleapis.com/youtube/v3/playlistItems?part=snippet&playlistId=UU{REST_OF_CHANNEL_ID}&maxResults=10&key={YOUR_API_KEY}"

# You should see a list of recent videos
```

If both commands return data successfully, your YouTube API setup is complete!

---

## Step 8: Add Credentials to Configuration File

1. Open `/Users/marcusawakuni/Desktop/LW/podcast-outreach/api-credentials.env`
2. Fill in your YouTube credentials:

```bash
YOUTUBE_API_KEY=AIzaSyAbc123Def456Ghi789...
YOUTUBE_LAWAYRA_CHANNEL_ID=UC1A2B3C4D5E6F7G8H9I0J
```

---

## Step 9: Configure in N8N

When you import the N8N workflows:

1. Open workflow `1_LaWayra_Guest_Extraction.json`
2. Find the **"YouTube"** node (HTTP Request node)
3. Click **"Credentials"** → **"Create New"**
4. Select **"Header Auth"** or create a custom credential
5. Or simply use the HTTP Request node with the API key in the URL parameter:
   ```
   https://www.googleapis.com/youtube/v3/playlistItems?part=snippet&playlistId={UPLOADS_PLAYLIST_ID}&key={YOUR_API_KEY}
   ```
6. Save credentials as "YouTube - LaWayra Channel"

---

## API Limits & Quotas

**YouTube Data API v3 Quotas:**
- **Daily Quota:** 10,000 units per day (generous for our use case)
- **Cost per request:**
  - List videos: 1 unit
  - Search: 100 units
- **Our usage:**
  - Fetching all LaWayra videos: ~5-50 units (depending on pagination)
  - **Daily total:** ~50-100 units (well within limits)

**Quota Reset:** Daily at midnight Pacific Time

**Important Notes:**
- Each page of results costs 1 unit
- If you have 500 videos and fetch 50 per page, that's 10 requests = 10 units
- Very unlikely to hit the 10,000 unit daily limit with this automation

---

## Understanding YouTube Playlists

Every YouTube channel has an automatic "Uploads" playlist that contains all videos:

- **Channel ID format:** `UC1A2B3C4D5E6F7G8H9I0J`
- **Uploads Playlist ID:** Replace `UC` with `UU` → `UU1A2B3C4D5E6F7G8H9I0J`

This is what we'll use in the N8N workflow to fetch all LaWayra podcast episodes.

---

## Troubleshooting

### Issue: "API key not valid"
**Solution:**
- Check that you copied the full API key (no spaces)
- Verify the API is enabled in Google Cloud Console
- Make sure API restrictions allow YouTube Data API v3

### Issue: "Daily limit exceeded"
**Solution:**
- Wait for quota to reset (midnight Pacific Time)
- Reduce batch sizes in N8N workflows
- Consider requesting quota increase (rarely needed)

### Issue: "Channel not found"
**Solution:**
- Verify the Channel ID is correct
- Make sure the channel is public (not private/unlisted)
- Try searching via API with channel name instead

### Issue: "Videos not showing up"
**Solution:**
- Use the Uploads playlist ID (UU instead of UC)
- Check that videos are public (not private/unlisted)
- Verify the API response pagination

---

## Security Best Practices

1. **Restrict API Key:** Only allow YouTube Data API v3
2. **Don't commit to git:** Keep api-credentials.env in .gitignore
3. **Regenerate if exposed:** If you accidentally share the key, regenerate it in Google Cloud Console
4. **Monitor usage:** Check quota usage in Google Cloud Console

---

## Next Steps

✅ YouTube Data API enabled
✅ API Key created and secured
✅ LaWayra Channel ID found
✅ API connection tested successfully

**Continue to:** [LISTENNOTES_SETUP.md](LISTENNOTES_SETUP.md)
