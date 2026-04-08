# GHL & Drip Integration Setup Guide

## Overview
Since you already have GoHighLevel (GHL) and Drip for email marketing, we'll integrate them with the N8N automation for outreach.

**Cost:** Existing infrastructure (no additional cost)
**Time to complete:** ~15 minutes

---

## Part 1: GoHighLevel (GHL) API Setup

### Step 1: Access GHL API Settings

1. Log in to your [GoHighLevel account](https://app.gohighlevel.com/)
2. Go to **Settings** (gear icon)
3. Navigate to **"Integrations"** or **"API"**
4. Look for **"API Keys"** section

### Step 2: Create New API Key

1. Click **"Create API Key"** or **"Generate New Key"**
2. Name it: `Podcast Outreach Automation`
3. Set permissions (check all needed):
   - ✅ **Contacts** - Read/Write (to create prospect records)
   - ✅ **Conversations** - Read (to track email responses)
   - ✅ **Emails** - Send (to send outreach emails)
   - ✅ **Opportunities** - Write (to track podcast booking opportunities)
4. **Copy the API Key** (looks like a long alphanumeric string)

### Step 3: Get Your GHL Location ID

1. In GHL, go to **Settings** → **Business Profile**
2. Look for **"Location ID"** or check the URL:
   - URL format: `https://app.gohighlevel.com/location/{LOCATION_ID}/...`
   - Copy the `{LOCATION_ID}` part
3. Alternative: Use the API to list locations (if you have multiple)

### Step 4: Test GHL API Connection

```bash
# Replace YOUR_API_KEY and YOUR_LOCATION_ID
curl -X GET \
  'https://rest.gohighlevel.com/v1/contacts/' \
  -H 'Authorization: Bearer YOUR_API_KEY' \
  -H 'Content-Type: application/json'
```

**Expected Response:** JSON with list of contacts (or empty array if no contacts yet)

### Step 5: Understand GHL Email Sending

GHL has two ways to send emails:

#### Option A: Via Campaigns (Recommended for Warmup)
- Create an email campaign in GHL
- N8N adds contacts to the campaign
- GHL sends emails on schedule
- **Pros:** Built-in warmup, tracking, follow-ups
- **Cons:** Less real-time control

#### Option B: Via API (Direct Send)
- N8N sends emails directly via GHL API
- Full control over timing and content
- **Pros:** More flexible, real-time sending
- **Cons:** Need to manage warmup manually

**Recommendation:** Use **Option A (Campaigns)** for first month (built-in warmup), then switch to **Option B** for full automation.

---

## Part 2: Drip API Setup

### Step 1: Access Drip API Settings

1. Log in to your [Drip account](https://www.getdrip.com/)
2. Go to **Settings** → **User Settings** (top right, your profile)
3. Click on **"API Tokens"** or **"Integrations"**

### Step 2: Generate API Token

1. Click **"Create New Token"** or **"Generate Token"**
2. Name it: `Podcast Outreach N8N`
3. **Copy the API Token** (looks like a long string)
4. **⚠️ Important:** Save it immediately—you may not be able to view it again

### Step 3: Get Your Drip Account ID

1. In Drip, check the URL when you're in your account:
   - URL format: `https://www.getdrip.com/{ACCOUNT_ID}/...`
   - Copy the `{ACCOUNT_ID}` part (usually a number)
2. Alternative: Check **Settings** → **Account** → **Account ID**

### Step 4: Test Drip API Connection

```bash
# Replace YOUR_API_TOKEN and YOUR_ACCOUNT_ID
curl -X GET \
  'https://api.getdrip.com/v2/{YOUR_ACCOUNT_ID}/subscribers' \
  -H 'Authorization: Bearer YOUR_API_TOKEN' \
  -H 'Content-Type: application/json'
```

**Expected Response:** JSON with list of subscribers

---

## Part 3: Choose Email Platform for Outreach

### Decision: GHL vs Drip?

| Feature | GHL | Drip |
|---------|-----|------|
| **Warmup Support** | ✅ Built-in campaigns | ⚠️ Manual |
| **Response Tracking** | ✅ Conversations inbox | ✅ Reply tracking |
| **Follow-up Sequences** | ✅ Automated workflows | ✅ Automated workflows |
| **Personalization** | ✅ Merge tags | ✅ Liquid templating |
| **N8N Integration** | ✅ Direct API support | ✅ Direct API support |
| **Cost** | Existing | Existing |

**Recommendation for Podcast Outreach:**
- **Use GHL** if you prefer all-in-one CRM (contacts, opportunities, emails in one place)
- **Use Drip** if you already have email sequences/workflows set up there

**For this guide, we'll set up BOTH** and you can choose which to use in the N8N workflows.

---

## Part 4: Add Credentials to Configuration

```bash
# In api-credentials.env

# GHL Configuration
GHL_API_KEY=your_ghl_api_key_here
GHL_LOCATION_ID=your_ghl_location_id_here

# Drip Configuration
DRIP_API_KEY=your_drip_api_key_here
DRIP_ACCOUNT_ID=your_drip_account_id_here

# Email Sending Settings
OUTREACH_FROM_EMAIL=sam@lawayra.com
OUTREACH_FROM_NAME=Sam from LaWayra
```

---

## Part 5: Configure in N8N

### For GHL:

1. Open workflow `5_Outreach_Automation.json`
2. Find the **"GHL Send Email"** HTTP Request node
3. Configure:
   - **Method:** POST
   - **URL:** `https://rest.gohighlevel.com/v1/conversations/messages`
   - **Authentication:** Header Auth
     - Header Name: `Authorization`
     - Header Value: `Bearer {{$env.GHL_API_KEY}}`
   - **Body (JSON):**
     ```json
     {
       "type": "Email",
       "contactId": "{{$json.contact_id}}",
       "message": "{{$json.pitch_text}}",
       "subject": "{{$json.email_subject}}"
     }
     ```

### For Drip:

1. Open workflow `5_Outreach_Automation.json`
2. Find the **"Drip Send Email"** HTTP Request node
3. Configure:
   - **Method:** POST
   - **URL:** `https://api.getdrip.com/v2/{{$env.DRIP_ACCOUNT_ID}}/campaigns/{CAMPAIGN_ID}/subscribers`
   - **Authentication:** Bearer Token
     - Token: `{{$env.DRIP_API_KEY}}`
   - **Body (JSON):**
     ```json
     {
       "subscribers": [{
         "email": "{{$json.podcast_email}}",
         "custom_fields": {
           "podcast_name": "{{$json.podcast_name}}",
           "pitch_text": "{{$json.pitch_text}}"
         }
       }]
     }
     ```

---

## Part 6: Email Warmup Strategy

**Why Warmup Matters:**
- Sending 30 cold emails/day from a new or low-volume sender can trigger spam filters
- Gradual ramp-up builds sender reputation
- Reduces bounce rates and spam complaints

### Week-by-Week Warmup Plan:

| Week | Daily Emails | Strategy |
|------|--------------|----------|
| **Week 1** | 5-10 | Test emails, verify deliverability |
| **Week 2** | 10-15 | Gradual increase, monitor responses |
| **Week 3** | 15-20 | Continue scaling |
| **Week 4** | 20-25 | Approaching target volume |
| **Week 5+** | 25-30 | Full production volume |

### Implementation in N8N:

In the `5_Outreach_Automation.json` workflow:

```javascript
// Week 1: Send 5-10/day
const currentWeek = 1; // Update manually each week
const dailyLimits = {
  1: 10,
  2: 15,
  3: 20,
  4: 25,
  5: 30
};

const dailyLimit = dailyLimits[currentWeek] || 30;

// In Google Sheets, only fetch {dailyLimit} enriched podcasts
```

### Warmup Best Practices:

1. **Authenticate Your Domain:**
   - Set up SPF record for sam@lawayra.com
   - Set up DKIM record
   - Set up DMARC policy
   - GHL/Drip usually help with this

2. **Monitor Bounce Rates:**
   - Keep bounce rate < 3%
   - Remove invalid emails immediately

3. **Track Spam Complaints:**
   - Keep complaint rate < 0.1%
   - Include clear unsubscribe link

4. **Send at Natural Times:**
   - Schedule sends: 9 AM - 5 PM local time
   - Avoid weekends (lower engagement)

---

## Part 7: Response Tracking Setup

### Option A: GHL Conversations (Recommended)

GHL automatically tracks email replies in the **Conversations** inbox.

**N8N Webhook Setup:**
1. In GHL, go to **Settings** → **Integrations** → **Webhooks**
2. Create webhook for **"New Conversation Message"** event
3. Webhook URL: Your N8N webhook URL (from `6_Response_Tracking.json`)
4. When someone replies, GHL sends data to N8N → Updates Google Sheet

### Option B: Drip Reply Tracking

Drip tracks replies but doesn't have direct webhooks.

**Alternative:**
1. N8N polls Drip API daily for new replies
2. Or manually check Drip inbox and update Google Sheet

---

## Part 8: Create Email Templates

### Template 1: Initial Outreach (AI-Generated Pitch)

**Subject Line Options** (AI-generated per podcast):
- "Shared guest: [Guest Name]"
- "LaWayra + [Podcast Name] crossover?"
- "Psychedelic podcast collaboration?"

**Body:**
```
Hi [Host Name],

{{AI_GENERATED_PITCH}}

Looking forward to connecting!

Sam
Founder, LaWayra Ayahuasca Retreats
https://ayahuascapodcast.com

P.S. If this isn't the right fit, I totally understand—just let me know!
```

### Template 2: Follow-Up #1 (Day 5)

**Subject:** Re: [Original Subject]

```
Hi [Host Name],

Just bumping this to the top of your inbox in case you missed it.

Would love to chat about potentially appearing on [Podcast Name]—we're clearly covering similar ground in the psychedelic space.

Best,
Sam
```

### Template 3: Follow-Up #2 (Day 12 - Final)

**Subject:** Re: [Original Subject]

```
Hi [Host Name],

Last quick note! If now's not the right time or it's not a fit, no worries at all.

But if you're open to it, I'd love to share [specific topic] with your audience.

Cheers,
Sam
```

---

## Part 9: Testing Email Deliverability

### Before Going Live:

1. **Send Test Emails to Yourself:**
   ```
   Test 1: Send to your Gmail
   Test 2: Send to your Outlook/Hotmail
   Test 3: Send to a business email
   ```

2. **Check Spam Folder:**
   - Do test emails land in inbox or spam?
   - If spam: Review SPF/DKIM/DMARC settings

3. **Use Email Testing Tools:**
   - [Mail-Tester.com](https://www.mail-tester.com/) - Tests spam score
   - [GlockApps](https://glockapps.com/) - Tests deliverability

4. **Verify Links:**
   - Make sure https://ayahuascapodcast.com link works
   - Include unsubscribe link (GHL/Drip add automatically)

---

## Part 10: Compliance & Best Practices

### CAN-SPAM Compliance:

✅ **Include unsubscribe link** (GHL/Drip add automatically)
✅ **Use real sender address** (sam@lawayra.com)
✅ **Accurate subject lines** (no deceptive subjects)
✅ **Include physical address** (your business address in footer)

### GDPR Considerations (if targeting EU):

- Add consent tracking (for EU podcast hosts)
- Provide data deletion option
- Include privacy policy link

### Ethical Outreach:

- Don't send to same podcast multiple times
- Respect "not interested" responses
- Keep emails genuine and personalized
- No spammy tactics

---

## Troubleshooting

### Issue: Emails going to spam
**Solution:**
- Check SPF/DKIM/DMARC records
- Warm up slower (5/day for 2 weeks)
- Improve email content (less salesy, more conversational)
- Use a dedicated sending domain

### Issue: High bounce rate
**Solution:**
- Validate email addresses before sending (use email verification tool)
- Remove invalid emails from list
- Check for typos in scraped emails

### Issue: No responses
**Solution:**
- A/B test subject lines
- Improve pitch personalization
- Send at different times (Tuesday-Thursday, 10 AM)
- Follow up (but max 2 follow-ups)

### Issue: GHL/Drip API errors
**Solution:**
- Check API key is valid and has correct permissions
- Verify account/location ID is correct
- Check API rate limits (usually very high)

---

## Next Steps

✅ GHL API key obtained and tested
✅ Drip API key obtained and tested
✅ Email templates created
✅ Warmup strategy planned
✅ Compliance checklist complete

**You're now ready to build the N8N workflows!**

**Continue to:** Build Week 1 Foundation (see main README.md)
