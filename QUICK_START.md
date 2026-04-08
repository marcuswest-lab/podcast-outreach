# Quick Start Guide - Podcast Guest Outreach Automation

## 🚀 Get Started in 3 Steps

### Step 1: Complete Prerequisites (Week 1)

#### 1.1 API Credentials Setup (~30-45 minutes total)

Follow these guides in order:

1. **[Spotify API Setup](SPOTIFY_SETUP.md)** (~5 min)
   - Create Spotify Developer account
   - Get Client ID & Secret
   - Find LaWayra's Spotify Show ID

2. **[YouTube API Setup](YOUTUBE_SETUP.md)** (~10 min)
   - Enable YouTube Data API in Google Cloud
   - Get API key
   - Find LaWayra's YouTube Channel ID

3. **[Listen Notes API Setup](LISTENNOTES_SETUP.md)** (~5 min)
   - Sign up for free tier
   - Get API key
   - Plan to upgrade after testing

4. **[AI API Setup](AI_SETUP.md)** (~5 min)
   - Get Claude API key (recommended) OR OpenAI API key
   - For guest extraction & pitch generation

5. **[GHL/Drip Setup](GHL_DRIP_SETUP.md)** (~15 min)
   - Get GHL API key & Location ID
   - Get Drip API key & Account ID
   - Choose which platform to use for outreach

#### 1.2 Configuration Files (~10 minutes)

1. **Copy credentials template:**
   ```bash
   cd /Users/marcusawakuni/Desktop/LW/podcast-outreach
   cp api-credentials.env.template api-credentials.env
   ```

2. **Fill in ALL credentials** in `api-credentials.env`
   - Spotify Client ID & Secret
   - YouTube API Key
   - Listen Notes API Key
   - Claude or OpenAI API Key
   - GHL or Drip API Keys
   - Google Sheet IDs (see Step 1.3)

3. **Provide Sam's pitch prompt:**
   - Open `sam-pitch-prompt.txt`
   - Paste Sam's Gemini prompt that generates personalized pitches

4. **List seed podcasts:**
   - Edit `seed-podcasts.json`
   - Add 5-10 psychedelic podcasts (beyond Psychedelics Today)
   - Find their Spotify Show IDs

#### 1.3 Google Sheets Setup (~15 minutes)

1. **Import templates:**
   - Go to [Google Drive](https://drive.google.com/)
   - Sign in as marcusawakuni@gmail.com
   - Follow instructions in `/google-sheets-templates/README.md`

2. **Create these 3 sheets:**
   - LaWayra Guest Database
   - Discovered Podcasts
   - Outreach Analytics Dashboard

3. **Add Sheet IDs to `api-credentials.env`**

---

### Step 2: Build & Test N8N Workflows (Weeks 2-5)

#### Week 2: Guest Extraction

1. Import `1_LaWayra_Guest_Extraction.json` to N8N
2. Configure API credentials in nodes
3. Test with manual trigger
4. Run full extraction → Should get ~80-90 guests

5. Import `2_External_Podcast_Guest_Extraction.json` to N8N
6. Test with Psychedelics Today first
7. Run for all seed podcasts → Should get hundreds of guests

8. Manual review pass in Google Sheets (verify guest names)

#### Week 3: Podcast Discovery

1. Import `3_Podcast_Discovery_Engine.json` to N8N
2. Test with first 10 guests (free Listen Notes tier)
3. Review results - are podcasts relevant?
4. If quality is good: Upgrade to Listen Notes Basic ($49/mo)
5. Run discovery for all guests → Should get 500-2000 podcasts

#### Week 4: Enrichment & Pitches

1. Import `4_Podcast_Enrichment_And_Pitch.json` to N8N
2. Configure AI node with Sam's pitch prompt
3. Generate first batch of 20-30 pitches
4. Manual review: Are pitches personalized and natural?
5. Refine AI prompt if needed

#### Week 5: Outreach

1. Import `5_Outreach_Automation.json` to N8N
2. Configure GHL or Drip integration
3. **Week 5 Day 1:** Send 5 test emails (warmup)
4. **Week 5 Day 2-7:** Send 5-10 emails/day
5. Monitor deliverability (spam folder? bounces?)

#### Week 6: Automation & Optimization

1. Import `6_Response_Tracking.json` to N8N
2. Set up follow-up sequences (Day 5, Day 12)
3. Scale to 15-20 emails/day
4. Monitor analytics dashboard
5. Optimize based on response rates

---

### Step 3: Go Live & Scale (Week 7+)

1. **Ramp up to 20-30 emails/day**
2. **Track key metrics:**
   - Response rate (target: 5-15%)
   - Booking rate (target: 1-3 bookings/week)
3. **Optimize:**
   - A/B test subject lines
   - Refine pitch templates
   - Focus on high-performing seed podcasts
4. **Scale:**
   - Add more seed podcasts (Funnel B)
   - Expand to related niches (wellness, spirituality)

---

## ✅ Success Checklist

### Week 1: Foundation ✅
- [ ] All API credentials obtained and tested
- [ ] `api-credentials.env` fully filled in
- [ ] Sam's pitch prompt provided
- [ ] Seed podcasts list complete (5-10 podcasts)
- [ ] 3 Google Sheets created and configured
- [ ] Google Sheets accessible from N8N

### Week 2: Guest Extraction ✅
- [ ] LaWayra guest extraction works (~90 guests)
- [ ] External guest extraction works (Psychedelics Today)
- [ ] All seed podcasts processed
- [ ] Manual review pass completed (85%+ accuracy)
- [ ] Guest Database populated in Google Sheets

### Week 3: Podcast Discovery ✅
- [ ] Podcast discovery works for test guests
- [ ] Listen Notes free tier validated
- [ ] Upgraded to paid tier if needed
- [ ] Discovery run for all guests
- [ ] 500+ unique podcasts discovered

### Week 4: Enrichment ✅
- [ ] Contact scraping finds 50%+ emails
- [ ] AI pitch generation works
- [ ] First 20-30 pitches reviewed (high quality)
- [ ] Warm vs discovery lead logic works
- [ ] Discovered Podcasts sheet populated

### Week 5: Outreach ✅
- [ ] GHL/Drip integration working
- [ ] Test emails delivered successfully
- [ ] Warmup schedule started (5-10/day)
- [ ] No spam flags or bounces
- [ ] Response tracking configured

### Week 6: Optimization ✅
- [ ] Follow-up sequences working
- [ ] Analytics dashboard updating
- [ ] First responses received
- [ ] Scaled to 20-30 emails/day
- [ ] System running autonomously

---

## 🎯 Expected Outcomes (6 Weeks)

| Metric | Target |
|--------|--------|
| Guests Extracted | 200-300 |
| Podcasts Discovered | 500-2000 |
| Emails Sent | 300-500 |
| Response Rate | 5-15% |
| Podcast Bookings | 3-5 |
| Time Saved | 15-20 hours/week |

---

## 📞 What to Do If You Get Stuck

1. **API Setup Issues:**
   - Re-read the specific setup guide (SPOTIFY_SETUP.md, etc.)
   - Check API keys have no spaces or typos
   - Verify API services are enabled

2. **N8N Workflow Issues:**
   - Check credentials are configured in each node
   - Test nodes individually (not just full workflow)
   - Review N8N execution logs for error messages

3. **Data Quality Issues:**
   - Guest extraction not accurate? → Adjust AI prompt, lower temperature
   - Podcast discovery not relevant? → Add filters (language, region)
   - Pitches too generic? → Refine Sam's prompt, add more context

4. **Email Deliverability Issues:**
   - Going to spam? → Check SPF/DKIM, slow down warmup
   - High bounce rate? → Validate emails before sending
   - No responses? → A/B test subject lines, improve pitches

---

## 💡 Pro Tips

1. **Start Small:**
   - Test with 10 guests before processing all 200
   - Send 5 emails before ramping to 30/day
   - Validate each phase before moving to next

2. **Manual Review is Critical:**
   - Week 2: Review guest extractions (catch AI errors)
   - Week 4: Review generated pitches (ensure quality)
   - Week 5: Monitor first emails (deliverability check)

3. **Optimize Based on Data:**
   - Track which seed podcasts lead to best bookings
   - Note which pitch styles get most responses
   - Double down on what works

4. **Don't Rush:**
   - Better to warmup slowly (avoid spam flags)
   - Better to review quality (avoid embarrassing pitches)
   - Better to iterate (improve over time)

---

## 🚀 You're Ready!

Everything is set up. Just follow the steps above in order, and you'll have a fully automated podcast outreach system in 6 weeks.

**Start with Week 1** → Complete all prerequisite tasks → Then move to Week 2!

**Questions?** Review the detailed guides:
- [Main README](README.md)
- [API Setup Guides](SPOTIFY_SETUP.md)
- [Google Sheets Templates](../google-sheets-templates/README.md)
- [Implementation Plan](../.claude/plans/rosy-wondering-micali.md)

Let's get Sam booked on 3-5 new podcasts this month! 🎙️
