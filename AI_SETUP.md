# AI API Setup Guide (Claude / OpenAI)

## Overview
We use AI for two critical tasks:
1. **Guest Extraction:** Extract guest names from podcast episode titles/descriptions
2. **Pitch Generation:** Create personalized outreach messages using Sam's Gemini prompt

**Recommended:** Anthropic Claude (better at nuanced, conversational writing)
**Alternative:** OpenAI GPT-4 (also works well)

**Cost:** ~$10-30/month (usage-based)
**Time to complete:** ~5 minutes

---

## Option A: Anthropic Claude API (Recommended)

### Why Claude?
- **Better at conversational writing** (more natural pitch generation)
- **Follows instructions precisely** (better guest extraction accuracy)
- **Cost-effective** for our use case
- **Longer context window** (can analyze more episode data at once)

### Step 1: Create Anthropic Account

1. Go to [Anthropic Console](https://console.anthropic.com/)
2. Click **"Sign Up"**
3. Enter your email (marcusawakuni@gmail.com or business email)
4. Verify your email
5. Complete account setup

### Step 2: Add Payment Method

1. Go to [Billing Settings](https://console.anthropic.com/settings/billing)
2. Click **"Add Payment Method"**
3. Enter credit card information
4. Set a **spending limit** (recommended: $50/month to avoid surprises)

### Step 3: Get API Key

1. Go to [API Keys](https://console.anthropic.com/settings/keys)
2. Click **"Create Key"**
3. Name it: `LaWayra Podcast Outreach`
4. **Copy the API key** (starts with `sk-ant-...`)
5. **⚠️ Important:** Save it immediately—you can't view it again

### Step 4: Test API Connection

```bash
# Replace YOUR_API_KEY with your actual key
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-3-5-sonnet-20241022",
    "max_tokens": 1024,
    "messages": [
      {"role": "user", "content": "Extract the guest name from this podcast title: Episode 42: Dennis McKenna on Plant Medicine. Return only the name."}
    ]
  }'
```

**Expected Response:** JSON with the guest name extracted

### Step 5: Add to Configuration

```bash
# In api-credentials.env
ANTHROPIC_API_KEY=sk-ant-api03-...
AI_PROVIDER=anthropic
```

---

## Option B: OpenAI API (Alternative)

### Why OpenAI?
- **Widely supported** in N8N (easier integration)
- **Good performance** for guest extraction and pitch generation
- **Well-documented** APIs

### Step 1: Create OpenAI Account

1. Go to [OpenAI Platform](https://platform.openai.com/)
2. Click **"Sign Up"**
3. Enter your email and create password
4. Verify your email
5. Complete account setup

### Step 2: Add Payment Method

1. Go to [Billing Settings](https://platform.openai.com/account/billing/overview)
2. Click **"Add payment details"**
3. Enter credit card information
4. Set a **usage limit** (recommended: $50/month)

### Step 3: Get API Key

1. Go to [API Keys](https://platform.openai.com/api-keys)
2. Click **"Create new secret key"**
3. Name it: `LaWayra Podcast Outreach`
4. **Copy the API key** (starts with `sk-...`)
5. **⚠️ Important:** Save it immediately—you can't view it again

### Step 4: Test API Connection

```bash
# Replace YOUR_API_KEY with your actual key
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "gpt-4",
    "messages": [
      {"role": "user", "content": "Extract the guest name from this podcast title: Episode 42: Dennis McKenna on Plant Medicine. Return only the name."}
    ],
    "temperature": 0.3
  }'
```

**Expected Response:** JSON with the guest name extracted

### Step 5: Add to Configuration

```bash
# In api-credentials.env
OPENAI_API_KEY=sk-...
AI_PROVIDER=openai
```

---

## Cost Estimates

### Anthropic Claude Pricing (Claude 3.5 Sonnet)
- **Input:** $3 per million tokens (~$0.003 per 1K tokens)
- **Output:** $15 per million tokens (~$0.015 per 1K tokens)

**Our Usage Estimate:**
- **Guest Extraction:** ~100 tokens input, ~10 tokens output per episode
  - Cost per extraction: ~$0.0003 + ~$0.0002 = **~$0.0005/episode**
  - 100 episodes = **~$0.05**
- **Pitch Generation:** ~500 tokens input, ~300 tokens output per pitch
  - Cost per pitch: ~$0.0015 + ~$0.0045 = **~$0.006/pitch**
  - 100 pitches = **~$0.60**

**Monthly Estimate:**
- Extract 200 guests: ~$0.10
- Generate 500 pitches: ~$3.00
- **Total: ~$5-10/month** (very affordable)

### OpenAI Pricing (GPT-4)
- **Input:** $30 per million tokens (~$0.03 per 1K tokens)
- **Output:** $60 per million tokens (~$0.06 per 1K tokens)

**Our Usage Estimate:**
- **Guest Extraction:** ~$0.003/episode
- **Pitch Generation:** ~$0.03/pitch

**Monthly Estimate:**
- Extract 200 guests: ~$0.60
- Generate 500 pitches: ~$15.00
- **Total: ~$20-30/month**

**Recommendation:** Use **Claude** for 3-5x cost savings with better output quality.

---

## Configure AI Prompts

### 1. Guest Extraction Prompt (Built into N8N Workflow)

```
Extract the guest name(s) from this podcast episode title and description.

Rules:
- Return ONLY the guest's full name (first and last name)
- If there are multiple guests, separate with commas
- If this is a solo episode (no guest), return "SOLO"
- Do not include titles (Dr., PhD, etc.) unless part of stage name
- Do not include any other text or explanation

Episode Title: {{title}}
Episode Description: {{description}}

Guest Name:
```

### 2. Pitch Generation Prompt (Sam's Custom Prompt)

**You need to provide Sam's Gemini prompt.**

The prompt should accept these variables:
- `{{podcast_name}}` - Target podcast name
- `{{podcast_description}}` - From Listen Notes
- `{{recent_episodes}}` - Recent episode titles
- `{{source_guest}}` - Guest that led to discovery
- `{{guest_source_type}}` - "own_guest" or "external_guest"

**Template structure:**
```
Generate a personalized podcast pitch for Sam from LaWayra to appear on {{podcast_name}}.

[If guest_source_type == "own_guest"]
Warm approach: Reference that we both recently hosted {{source_guest}}.

[If guest_source_type == "external_guest"]
Discovery approach: Reference our shared interest in psychedelic/plant medicine topics.

Podcast Description: {{podcast_description}}
Recent Episodes: {{recent_episodes}}

Requirements:
- Match the podcast's tone and style
- Keep it conversational and authentic
- Highlight relevant topics Sam can discuss
- Include a clear call-to-action
- Length: 150-250 words

Pitch:
```

**Action Required:** Copy Sam's actual Gemini prompt into `sam-pitch-prompt.txt`

---

## Configure in N8N

### For Anthropic Claude:

1. Open any workflow with an AI node
2. Click the **AI node** → **"Credentials"**
3. Select **"Anthropic API"**
4. Enter:
   - **API Key:** From api-credentials.env
5. In the AI node settings:
   - **Model:** `claude-3-5-sonnet-20241022` (recommended)
   - **Temperature:** `0.3` (for guest extraction), `0.7` (for pitch generation)
   - **Max Tokens:** `100` (guest extraction), `500` (pitch generation)

### For OpenAI:

1. Open any workflow with an AI node
2. Click the **AI node** → **"Credentials"**
3. Select **"OpenAI API"**
4. Enter:
   - **API Key:** From api-credentials.env
5. In the AI node settings:
   - **Model:** `gpt-4` or `gpt-4-turbo`
   - **Temperature:** `0.3` (for guest extraction), `0.7` (for pitch generation)
   - **Max Tokens:** `100` (guest extraction), `500` (pitch generation)

---

## Temperature Settings Explained

**Temperature = How Creative/Random the AI Output Is**

- **0.0-0.3 (Low):** Deterministic, factual, consistent
  - **Use for:** Guest extraction (we want the exact name, no creativity)

- **0.5-0.7 (Medium):** Balanced, natural, varied
  - **Use for:** Pitch generation (conversational but not too random)

- **0.8-1.0 (High):** Creative, diverse, unpredictable
  - **Avoid for our use case** (too inconsistent)

---

## Monitoring Usage & Costs

### Anthropic Console
- Check usage: [Anthropic Console - Usage](https://console.anthropic.com/settings/usage)
- Set alerts for spending thresholds
- View per-request costs

### OpenAI Console
- Check usage: [OpenAI Usage Dashboard](https://platform.openai.com/usage)
- Set monthly budget limits
- View API call logs

**Recommendation:** Check weekly during first month to validate cost estimates.

---

## Troubleshooting

### Issue: "Invalid API key"
**Solution:**
- Check that you copied the full key (starts with `sk-ant-` for Claude or `sk-` for OpenAI)
- Verify no extra spaces or line breaks
- Try regenerating the key

### Issue: "Rate limit exceeded"
**Solution:**
- Anthropic/OpenAI have generous rate limits (60 requests/min)
- Add delays in N8N workflow if hitting limits (unlikely for our use case)

### Issue: "Insufficient credits/quota"
**Solution:**
- Add payment method in billing settings
- Increase spending limit
- Check that card is valid and has funds

### Issue: AI returns incorrect guest names
**Solution:**
- Adjust temperature (lower = more conservative)
- Improve prompt clarity
- Add examples to the prompt (few-shot learning)

### Issue: Pitches sound too generic
**Solution:**
- Provide more context in the prompt
- Increase temperature slightly (0.7 → 0.8)
- Add more specific instructions about tone/style

---

## Best Practices

1. **Use Caching:**
   - Don't re-generate pitches for same podcast
   - Store extracted guest names in Google Sheets

2. **Prompt Engineering:**
   - Be specific about desired output format
   - Provide examples when possible
   - Iterate based on results

3. **Cost Optimization:**
   - Use Claude instead of OpenAI (3-5x cheaper)
   - Limit max_tokens to what you actually need
   - Batch requests when possible (N8N handles this)

4. **Quality Control:**
   - Manual review of first 20 guest extractions
   - Manual review of first 20 generated pitches
   - Adjust prompts based on feedback

---

## Next Steps

✅ AI API account created (Claude or OpenAI)
✅ API key obtained and secured
✅ API connection tested successfully
✅ Sam's pitch prompt added to `sam-pitch-prompt.txt`

**Continue to:** [GHL_DRIP_SETUP.md](GHL_DRIP_SETUP.md)
