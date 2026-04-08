# Listen Notes API Setup Guide

## Overview
Listen Notes is the best podcast search database. We use it to find where guests have appeared across the podcast ecosystem.

**Cost:** Free tier (300 requests/month) → Paid tier ($49-149/mo)
**Time to complete:** ~5 minutes

---

## Step 1: Create Listen Notes API Account

1. Go to [Listen Notes API](https://www.listennotes.com/api/)
2. Click **"Get Started for Free"**
3. Sign up with your email (marcusawakuni@gmail.com or business email)
4. Verify your email address
5. Log in to the [Listen Notes API Dashboard](https://www.listennotes.com/api/dashboard/)

---

## Step 2: Choose API Plan

### Free Tier (Start Here)
- **Cost:** $0/month
- **Quota:** 300 requests per month
- **Best for:** Testing and validation (first 10-30 guests)
- **Limitations:**
  - ~10 guests per day if each guest requires 1-2 searches
  - Good for Week 3 testing phase

### Basic Plan (Recommended for Production)
- **Cost:** $49/month
- **Quota:** 10,000 requests per month
- **Best for:** Production use (can process ~300 guests per day)
- **Upgrade after:** Week 3 when you've validated data quality

### Pro Plan (If Scaling Further)
- **Cost:** $149/month
- **Quota:** 40,000 requests per month
- **Best for:** High-volume operations or multiple projects

**Recommendation:** Start with **Free tier** for testing, upgrade to **Basic ($49/mo)** once validated.

---

## Step 3: Get Your API Key

1. In the [API Dashboard](https://www.listennotes.com/api/dashboard/), you'll see your API key
2. Copy the **API Key** (looks like: `abc123def456ghi789...`)
3. This is your authentication token for all API requests

**⚠️ Important:** Keep your API key private. Never share it publicly or commit it to git.

---

## Step 4: Understand API Endpoints We'll Use

### 1. **Search Endpoint** (Main one for our automation)
```
GET /api/v2/search
```
**Purpose:** Find podcast episodes by guest name
**Example:**
```bash
curl -X GET \
  'https://listen-api.listennotes.com/api/v2/search?q=Dennis%20McKenna&type=episode' \
  -H 'X-ListenAPI-Key: YOUR_API_KEY'
```

**Parameters:**
- `q`: Guest name (e.g., "Dennis McKenna")
- `type`: "episode" (we're searching for episodes)
- `sort_by_date`: 0 (relevance) or 1 (date)
- `offset`: Pagination offset

### 2. **Podcast Lookup Endpoint**
```
GET /api/v2/podcasts/{id}
```
**Purpose:** Get full podcast details (description, email, website)
**Example:**
```bash
curl -X GET \
  'https://listen-api.listennotes.com/api/v2/podcasts/{PODCAST_ID}' \
  -H 'X-ListenAPI-Key: YOUR_API_KEY'
```

---

## Step 5: Test API Connection

### Test 1: Search for a Known Guest

```bash
# Replace YOUR_API_KEY with your actual key
curl -X GET \
  'https://listen-api.listennotes.com/api/v2/search?q=Dennis%20McKenna&type=episode' \
  -H 'X-ListenAPI-Key: YOUR_API_KEY'
```

**Expected Response:** JSON with podcast episodes featuring Dennis McKenna

### Test 2: Get Podcast Details

Find a podcast ID from the search results, then:

```bash
curl -X GET \
  'https://listen-api.listennotes.com/api/v2/podcasts/{PODCAST_ID}' \
  -H 'X-ListenAPI-Key: YOUR_API_KEY'
```

**Expected Response:** Podcast details including title, description, email (if available)

If both tests work, your API is ready!

---

## Step 6: Add Credentials to Configuration File

1. Open `/Users/marcusawakuni/Desktop/LW/podcast-outreach/api-credentials.env`
2. Fill in your Listen Notes credentials:

```bash
LISTEN_NOTES_API_KEY=abc123def456ghi789...
LISTEN_NOTES_TIER=free  # Change to "basic" or "pro" when you upgrade
```

---

## Step 7: Configure in N8N

When you import the N8N workflows:

1. Open workflow `3_Podcast_Discovery_Engine.json`
2. Find the **"Listen Notes Search"** HTTP Request node
3. Click **"Credentials"** → **"Create New"**
4. Select **"Header Auth"**
5. Configure:
   - **Name:** `X-ListenAPI-Key`
   - **Value:** Your API key from api-credentials.env
6. Save credentials as "Listen Notes API"

---

## API Limits & Rate Limiting

### Free Tier
- **Quota:** 300 requests/month
- **Rate Limit:** Unlimited requests per second (within monthly quota)
- **Usage for testing:**
  - Test with 10 guests: ~10-20 requests
  - Test with 30 guests: ~30-60 requests (if each guest needs 1-2 searches)

### Basic Plan ($49/mo)
- **Quota:** 10,000 requests/month
- **Daily equivalent:** ~333 requests/day
- **Usage for production:**
  - Process 20-30 guests/day: ~40-60 requests/day (well within limits)
  - Can discover podcasts for all ~200 guests in one week if needed

### Pro Plan ($149/mo)
- **Quota:** 40,000 requests/month
- **Daily equivalent:** ~1,333 requests/day
- **Best for:** Multi-project use or rapid scaling

---

## Step 8: Plan Your Quota Usage

**Week 3 - Testing Phase (Free Tier):**
```
Day 1-2: Test with 5 guests = ~10 requests
Day 3-5: Test with 10 more guests = ~20 requests
Day 6-7: Test with 15 more guests = ~30 requests
Total: ~60 requests out of 300 (plenty of buffer)
```

**Week 3 End - Validation Decision:**
- If data quality is good (relevant podcasts found): ✅ Upgrade to Basic ($49/mo)
- If data quality is poor: ❌ Adjust approach or try alternative APIs

**Week 4+ - Production (Basic Tier):**
```
Daily: Process 20-30 guests = ~40-60 requests/day
Monthly: ~1,200-1,800 requests out of 10,000 (safe margin)
```

---

## Understanding Search Results

When you search for a guest (e.g., "Dennis McKenna"), the API returns:

```json
{
  "results": [
    {
      "podcast": {
        "id": "abc123",
        "title": "Psychedelics Today",
        "publisher": "Psychedelics Today",
        "email": "info@psychedelicstoday.com",  // ← Contact info!
        "website": "https://psychedelicstoday.com"
      },
      "title": "Episode Title with Guest Name",
      "description": "Episode description..."
    }
  ]
}
```

**What we extract:**
- `podcast.id` → Unique identifier
- `podcast.title` → Podcast name
- `podcast.email` → Contact email (if available)
- `podcast.website` → For scraping more contact info

---

## Data Quality Expectations

**Search Accuracy:**
- **High-profile guests** (Dennis McKenna, Wade Davis): Excellent results
- **Mid-tier guests:** Good results (80-90% relevant)
- **Lesser-known guests:** Mixed results (may need manual review)

**Deduplication:**
- Same podcast may appear multiple times (different episodes)
- N8N workflow will deduplicate by podcast ID

**Contact Info Availability:**
- ~50-60% of podcasts have email in Listen Notes metadata
- Remaining contacts: scrape from website or use Instagram

---

## Troubleshooting

### Issue: "Invalid API key"
**Solution:**
- Check that you copied the full API key (no spaces)
- Verify you're using the header name: `X-ListenAPI-Key`
- Try regenerating the key in the dashboard

### Issue: "Quota exceeded"
**Solution:**
- Check usage in [API Dashboard](https://www.listennotes.com/api/dashboard/)
- Wait for monthly reset (1st of each month)
- Upgrade to Basic plan if consistently hitting limits

### Issue: "No results for guest name"
**Solution:**
- Guest may not be prominent enough in podcast ecosystem
- Try variations of the name (nickname, full name, etc.)
- Some guests may have very few appearances (expected)

### Issue: "Too many duplicate results"
**Solution:**
- Use podcast ID to deduplicate
- Filter by podcast language (English)
- Filter by region (US if targeting US podcasts)

---

## Advanced Search Tips

### 1. **Improve Search Accuracy**
```
# More specific query
q=Dennis McKenna psychedelic

# Filter by language
q=Dennis McKenna&language=English

# Recent episodes only
q=Dennis McKenna&published_after=1609459200000  # Jan 1, 2021
```

### 2. **Pagination for Popular Guests**
```
# First page
offset=0

# Second page
offset=10

# Automatically handled by N8N workflow
```

### 3. **Exclude LaWayra & Seed Podcasts**
```
# In N8N workflow, filter out:
- podcast.id = LaWayra's ID
- podcast.id in [seed_podcasts_ids]
```

---

## Cost Optimization Strategies

1. **Start with Free Tier:**
   - Test with 30 guests before committing to paid plan
   - Validate data quality and relevance

2. **Batch Processing:**
   - Process 20-30 guests per day (don't rush)
   - Spreads cost over time

3. **Cache Results:**
   - Store discovered podcasts in Google Sheets
   - Don't re-search the same guest multiple times

4. **Upgrade Only When Needed:**
   - Free tier → Basic ($49): When you're processing more than 10 guests/day
   - Basic → Pro ($149): Only if running multiple projects or scaling beyond 30 guests/day

---

## Alternative APIs (If Needed)

If Listen Notes doesn't meet your needs:

1. **Podchaser API** - Similar to Listen Notes, different pricing
2. **Podcast Index API** - Open-source podcast database (cheaper, less features)
3. **Spotify API** - Limited search (no cross-podcast guest tracking)

**Recommendation:** Stick with Listen Notes—it's the industry standard for podcast discovery.

---

## Next Steps

✅ Listen Notes API account created
✅ API Key obtained (free tier)
✅ API connection tested successfully
✅ Plan to upgrade to Basic after validation

**Continue to:** [AI_SETUP.md](AI_SETUP.md)
