# LaWayra Podcast Guest Outreach Automation

## Project Goal
Automate podcast guest discovery and outreach to book Sam on 20-30 new podcasts per month, scaling from manual 1-2 pitches/day to systematic 20-30/day.

## How It Works
Two-funnel approach to reverse-engineer the podcast guest circuit:
- **Funnel A (Warm Leads)**: Find where LaWayra's ~136 guests have appeared → warm outreach ("We both hosted Dennis McKenna")
- **Funnel B (Discovery Leads)**: Find guests from other psychedelic/wellness podcasts → find where they've appeared

**Pipeline:** Guest Extraction → Podcast Discovery → Enrichment → Multi-Step AI Pitch Generation (Gemini + Claude) → Automated Email Outreach

**Why it works:** Shared guest connections dramatically increase response rates. Instead of cold outreach, Sam references mutual connections.

**Source Podcast:** LaWayra's Ayahuasca Podcast (YouTube channel `UCs1CblyDBVJ6_vvLAgvHqsg`, ~136 guests). WF#1 extracts Sam's own guests only.

## Current Status

### COMPLETED
- All API credentials configured in `api-credentials.env`
- Consolidated Google Spreadsheet with 4 tabs (LW Guests, Discovered Podcasts, External Guests, Analytics)
- YouTube episode fetching works (1,036 episodes fetched)
- Claude AI guest extraction works (370 guests identified from 1,036 episodes)
- PodcastIndex.org replaces Listen Notes (Listen Notes rejected our use case)
- Workflows #1-5 rebuilt with real API credentials, PodcastIndex migration complete
- All If nodes removed from all workflows (broken on v1.123.7, replaced with Code nodes)
- Workflow order corrected: WF#2=Podcast Discovery, WF#3=External Guest Extraction
- Google Sheets credentials set in n8n UI for all workflows
- **WF#1 tested and working** — guest extraction with dedup
- **WF#2 tested and working** — podcast discovery with AI relevance scoring + dedup
- **WF#3 tested and working** — external guest extraction with dedup, batched Claude calls
- **WF#4 tested and working** — enrichment & pitch generation with HTTP timeouts
- **WF#5 updated and ready** — day check bypass for testing, pitch text in Drip custom fields
- Deduplication added to WF#1, WF#2, WF#3 (prevents duplicate entries across runs)
- AI relevance scoring added to WF#2 (1-10 score, configurable threshold)
- HTTP timeouts (30s) added to all API calls in WF#3, WF#4, WF#5
- Skip Empty nodes added before all Append nodes (prevents empty rows from `_skip` items)

### UPDATED (2026-03-15)
- **WF#1 reverted to Ayahuasca Podcast** — extracts Sam's own guests only (own_guest). External podcasts like Psychedelics Today go through WF#3 instead.
- **WF#2 relevance scoring expanded** — now includes entheogens, biohacking, health, wellness, men's work; threshold lowered to 4
- **WF#2 exclusion list updated** — Sam's 37 past podcast appearances added to prevent pitching shows he's already been on
- **WF#5 daily limit set to 10** — flat 10 emails/day (replaces warmup schedule)
- **Sam's reference data saved** — `sam-bio.txt`, `sam-guest-list.txt`, `sam-past-appearances.txt`

### PENDING
- **WF#4 testing** — needs Google Sheets credential re-set, then test with real data
- **WF#5 testing** — awaiting Sam's approval of generated pitches before sending real emails
- Set `BYPASS_DAY_CHECK = false` in WF#5 Check Sending Day node before production use
- Set up Drip automation triggered by `send-podcast-pitch` tag
- Re-set Google Sheets credentials in n8n UI for WF#1, WF#2, WF#4, WF#5 (API updates overwrote them)

## Tech Stack
- **n8n**: Workflow automation platform (6 main workflows)
  - **Instance URL**: https://server.lawayraserver.com
  - **Version**: 1.123.7 (Self Hosted) — see Known Issues below
- **n8n-mcp**: MCP server with access to 1,084 nodes + 2,646 real-world workflow examples
- **n8n-skills**: 7 Claude Code skills for building production-ready workflows with validation
- **APIs**: Spotify, YouTube Data, PodcastIndex.org, Claude AI (Anthropic), Gemini AI (Google, pending key), GHL/Drip
- **Storage**: Google Sheets (1 spreadsheet with 4 tabs: LW Guests, Discovered Podcasts, External Guests, Analytics)

## n8n Instance — CRITICAL Known Issues (v1.123.7)

### If Node v2 is BROKEN
The If node (typeVersion 2) on this n8n instance routes ALL items to output[1] (FALSE branch) regardless of the condition. This has been tested with:
- boolean `notEqual`
- string `notEquals`
- string `isNotEmpty`

**All fail the same way** — every item goes to the FALSE branch. **Do NOT use If nodes in any workflow on this instance.** Use Code nodes with filtering logic instead.

### Split In Batches Output Mapping
- **output[0]** = "done" — fires once AFTER all batches are processed
- **output[1]** = "loop" — fires for EACH batch with current batch items

Processing nodes must connect to output[1]. This is counter-intuitive and the n8n-mcp API tends to reset connections to output[0].

### TypeVersion Compatibility
Do NOT upgrade node typeVersions beyond these maximums:
- HTTP Request: **4.2** (not 4.4+)
- If: **2** (not 2.3+)
- Google Sheets: **4.6** (not 4.7+)

Upgrading beyond these causes: `"Cannot read properties of undefined (reading 'execute')"`

### Google Sheets Credential via API
The n8n-mcp API cannot reliably set Google Sheets credentials. When you update a workflow via API, it overwrites the credential the user selected in the UI. The workaround:
1. Make all API updates first
2. Then have the user open the workflow in n8n UI
3. Select the credential in each Google Sheets node
4. Save with Ctrl+S
5. Do NOT push any more API updates after this

### Empty Google Sheets Read Returns 0 Items
When a Google Sheets Read node reads a sheet with only headers (no data rows), it returns 0 items. In n8n, 0 output items = downstream nodes never execute. This creates a chicken-and-egg problem for dedup checks on first run.

**Workaround**: Use Code nodes that handle the empty case, or skip dedup on first run. For dedup, reference Read nodes with `try { $('ReadNode').all() } catch(e) { [] }` to gracefully handle empty results.

### getWorkflowStaticData is NOT Available in Code Nodes
`this.getWorkflowStaticData('global')` throws `TypeError: this.getWorkflowStaticData is not a function` in Code nodes on v1.123.7. This function is only available in Function nodes (legacy). **Do NOT use it in Code nodes.** Use alternative patterns like reading from sheets + within-batch Set tracking instead.

### setTimeout is Unreliable in Code Nodes
`setTimeout` and other timer functions may not work reliably in the Code node sandbox environment. Use n8n's built-in Wait node for delays between operations instead.

### HTTP Requests Without Timeouts Can Hang Indefinitely
API calls made via `this.helpers.httpRequest()` in Code nodes have no default timeout. Long-running requests (especially Claude AI calls) can hang forever, eventually causing the task runner to disconnect with `InternalTaskRunnerDisconnectAnalyzer.toDisconnectError`.

**Fix**: Always add `timeout: 30000` (30 seconds) to all `this.helpers.httpRequest()` calls:
```javascript
const response = await this.helpers.httpRequest({
  method: 'POST',
  url: 'https://api.anthropic.com/v1/messages',
  headers: { ... },
  body: { ... },
  json: true,
  timeout: 30000  // ALWAYS add this
});
```

### Batching Claude AI Calls is Essential
Making many individual Claude API calls in a loop (e.g., one per episode title) causes task runner disconnects. Batch multiple items per Claude call instead:
- **WF#3**: Sends 10 episode titles per Claude call (reduced from ~100 individual calls to ~2-5 batched calls)
- **WF#2 Score Relevance**: Sends all podcast names in one batched Claude call

## n8n Workflow IDs

| # | Workflow | n8n ID | Status |
|---|----------|--------|--------|
| 1 | LaWayra_Guest_Extraction | `f9sEhiNep0ISobCO` | Tested & working (10 nodes, includes dedup). Source: Ayahuasca Podcast channel |
| 2 | Podcast_Discovery | `WIctYIYrw0YpCPXU` | Tested & working (12 nodes, includes relevance scoring + dedup) |
| 3 | External_Guest_Extraction | `NTm9HMU2TD2KBZ6L` | Tested & working (11 nodes, batched Claude calls + dedup) |
| 4 | Enrichment_And_MultiStep_Pitch | `4IDSEo7inrGtNWs9` | Rebuilt (9 nodes). Gemini + Claude multi-step pitch. Needs Google Sheets credential re-set. |
| 5 | Outreach_Via_Drip | `fZmpONlaDqAG2TrS` | Ready, awaiting Sam's pitch approval (12 nodes) |

### Pipeline Flow
```
WF#1: LaWayra Guest Extraction
  Ayahuasca Podcast YouTube → Claude AI → Guest names → "LW Guests" tab

WF#2: Podcast Discovery
  "LW Guests" tab → PodcastIndex search → "Discovered Podcasts" tab

WF#3: External Guest Extraction
  "Discovered Podcasts" tab → Fetch episodes → Claude AI → "External Guests" tab

WF#4: Enrichment & Multi-Step Pitch (Gemini + Claude)
  "Discovered Podcasts" tab (pending) → PodcastIndex enrichment → Gemini podcast analysis → Claude persona match + pitch → Updates "Discovered Podcasts" tab

WF#5: Outreach via Drip
  "Discovered Podcasts" tab (enriched) → Drip API → Updates "Discovered Podcasts" tab
```

## Workflow #1 — LaWayra Guest Extraction (10 nodes)

```
Manual Trigger → Fetch YouTube Episodes → Batch 5 Episodes
           ↘                                    ↓ (output[1] = "loop")
     Read Existing Guests              Claude Extract Guest
     (parallel branch)                        ↓
                                        Parse Guest Names
                                          ↙           ↘
                                    Wait 1s        Filter Guests
                                      ↓                  ↓
                               (back to Batch)    Dedup Guests
                                                       ↓
                                                Append to Guest DB
```

- **Source**: Ayahuasca Podcast YouTube channel (`UCs1CblyDBVJ6_vvLAgvHqsg`). Host: Sam Believ. All guests labeled `own_guest`.
- Writes to: "LW Guests" tab in consolidated spreadsheet
- **Dedup**: Read Existing Guests runs on a parallel branch from Manual Trigger; Dedup Guests checks incoming names (case-insensitive) against existing sheet data + within-batch Set tracking
- Google Sheets credentials set in n8n UI

## Workflow #2 — Podcast Discovery (12 nodes)

```
Manual Trigger → Read Guest Database → Filter Unprocessed Guests
           ↘                            → One Guest at a Time (batch=1)
     Read Existing Podcasts                → [output 1] Search PodcastIndex
     (parallel branch)                     → Filter & Dedup Podcasts (checks sheet + within-batch)
                                           → Skip Empty Podcasts
                                           → Score Relevance (Claude AI, 1-10)
                                           → Append Discovered Podcasts
                                           → Mark Guest Processed
                                           → Wait 2s → loop
```

- Reads from: "LW Guests" tab (unprocessed guests)
- Writes to: "Discovered Podcasts" tab
- Updates: "LW Guests" tab (marks guest as Processed=TRUE)
- **Dedup**: Read Existing Podcasts runs on a parallel branch; Filter & Dedup checks Podcast ID against existing sheet data + within-batch Set tracking
- **AI Relevance Scoring**: Score Relevance node sends all podcast names to Claude in one batched call, returns 1-10 scores. Threshold: `RELEVANCE_THRESHOLD = 4` (lowered from 5 to capture biohacking, health, wellness, men's work, combat sports). Only podcasts scoring >= threshold are appended.
- **Exclusion List**: Search PodcastIndex node excludes Sam's 37 past podcast appearances (prevents pitching shows he's already been on)
- **Relevance Score** is the first column in the Discovered Podcasts tab

## Workflow #3 — External Guest Extraction (11 nodes)

```
Manual Trigger → Read Discovered Podcasts → Filter Unprocessed Podcasts
           ↘                                → One Podcast at a Time (batch=1)
     Read Existing External Guests            → [output 1] Fetch Episodes & Extract Guests
     (parallel branch)                        → Filter Guests (checks sheet + within-batch)
                                              → Skip Empty Guests
                                              → Append to External Guests
                                              → Mark Podcast Processed
                                              → Wait 2s → loop
```

- Reads from: "Discovered Podcasts" tab (unprocessed podcasts)
- Writes to: "External Guests" tab
- Updates: "Discovered Podcasts" tab (marks podcast as Processed=TRUE)
- Uses combined Code node (fetch + Claude) to avoid nested batch loops
- **Batched Claude calls**: Sends 10 episode titles per Claude call (MAX_EPISODES=20, TITLES_PER_CALL=10), with 30s HTTP timeouts on all requests
- **Dedup**: Read Existing External Guests runs on parallel branch; Filter Guests checks guest names (case-insensitive) against existing sheet data + within-batch Set

## Workflow #4 — Enrichment & Multi-Step Pitch (9 nodes)

```
Manual Trigger → Read Discovered Podcasts → Filter Pending Podcasts
  → Process One at a Time (batch=1)
    → [output 1] PodcastIndex Enrichment (get details, extract email/instagram)
    → Gemini Analyze Podcast (Google Search grounding — real-time web data)
    → Claude Generate Pitch (persona match + social proof + pitch in one call)
    → Update Discovered Podcasts
    → Wait 2s → loop back
```

- **PodcastIndex Enrichment** (Code node): Gets podcast details + 10 recent episode titles via PodcastIndex API. Extracts email, website, description, host name, Instagram.
- **Gemini Analyze Podcast** (Code node): Uses `tools: [{ google_search: {} }]` for web grounding. Finds host background, audience type, podcast tone, contact info, booking URL, recent notable guests, pitch angle suggestion. 45s timeout.
- **Claude Generate Pitch** (Code node): Single call that handles persona selection (5 modes: Engineer/Frontier Founder/Warrior-Healer/Healer/Storyteller), social proof matching (114 Sam guests embedded), and pitch writing. Embeds Sam's full profile + all guest names + tone rules. 45s timeout.
- **Tone rules**: Speak as peer, never gush, never imply shared guests mentioned Sam, use bullet points for topics, include required links (sambeliev.com, ayahuascaincolombia.com, ayahuascapodcast.com).
- **No If nodes**: Filter Pending returns empty array if no pending podcasts (stops execution naturally).
- Reads from: "Discovered Podcasts" tab (status=pending)
- Updates: "Discovered Podcasts" tab (status→enriched, adds pitch, email, contact info, persona mode, shared guests)
- Google Sheets credentials need re-setting in n8n UI after API update

## Workflow #5 — Outreach via Drip (12 nodes)

```
Schedule Tue-Thu 9AM → Check Sending Day → Read Discovered Podcasts
Manual Trigger ↗      → Apply Warmup Limit → Send One at a Time (batch=1)
                        → Prepare Drip Payload → Create Drip Subscriber → Apply Send Tag
                        → Mark as Contacted → Calculate Random Delay → Wait 2-4 min → loop back
```

- Reads from: "Discovered Podcasts" tab (status=enriched, has email)
- Updates: "Discovered Podcasts" tab (status→contacted, adds contacted date)
- **Daily limit**: Flat 10 emails/day (no warmup schedule — Sam's preference)
- Random 2-4 min delay between sends for natural pattern
- **HTTP timeouts**: 30s timeout on all Drip API calls
- **BYPASS_DAY_CHECK**: Set to `true` in Check Sending Day node for testing on non Tue/Wed/Thu days. **Set to `false` before production use.**
- **Drip custom fields**: Includes `pitch_body` (plain text) and `pitch_html` (HTML formatted) so Drip automation can use the generated pitch content
- **Status**: NOT YET TESTED — awaiting Sam's approval of generated pitches

## API Credentials (all configured)

All credentials are in `api-credentials.env`. Key details:

| API | Status | Notes |
|-----|--------|-------|
| Spotify | Configured | Client ID + Secret + Show ID |
| YouTube Data | Configured | API key + Channel ID |
| PodcastIndex.org | Configured | Replaces Listen Notes (free, no limits) |
| Anthropic Claude | Configured | For guest extraction + pitch generation |
| Google Gemini | Configured | gemini-2.0-flash with Google Search grounding (for WF#4 podcast analysis) |
| GHL | Configured | API key + Location ID |
| Drip | Configured | API key + Account ID |

## Google Sheets — Consolidated Spreadsheet

**One spreadsheet, four tabs.** All workflows read/write to this single spreadsheet.

**Spreadsheet ID**: `16l7E6Cox0fT5ktCXLjdJA6IWkY7P3EEWGg2hViYKwIY`
**Spreadsheet Name**: "LaWayra Podcast Outreach"
**URL**: [LaWayra Podcast Outreach](https://docs.google.com/spreadsheets/d/16l7E6Cox0fT5ktCXLjdJA6IWkY7P3EEWGg2hViYKwIY/edit)

| Tab Name | Used By | Description |
|----------|---------|-------------|
| LW Guests | WF#1 writes, WF#2 reads | Sam's Ayahuasca Podcast guests (own_guest only) |
| Discovered Podcasts | WF#2 writes, WF#3/4/5 read/update | Podcasts found via guest search |
| External Guests | WF#3 writes | Guests from discovered podcasts |
| Analytics | Dashboard metrics | Tracking outreach performance |

**Google account**: admin@lawayra.com (shared with marcusawakuni@gmail.com for n8n OAuth)

**Old spreadsheets** (deprecated, kept as backup):
- `1_ZEzGuSpzDYBVupqJAgXByA5mbok0LrurIqYMp5jLvc` — Old Discovered Podcasts
- `1P-dPQdzZfdV0fRT_aoUPxQhfddTLqP8b37IRNiM_UuA` — Old Analytics Dashboard

## Development Setup

### 1. Install n8n-mcp Server (Required First)
```bash
npx n8n-mcp
```

### 2. Configure Claude Desktop/Code
Edit `~/.claude/.mcp.json` (or Claude Desktop settings):
```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["n8n-mcp"],
      "env": {
        "MCP_MODE": "stdio",
        "N8N_API_URL": "https://server.lawayraserver.com",
        "N8N_API_KEY": "YOUR_N8N_API_KEY"
      }
    }
  }
}
```

### 3. Install n8n-skills
```bash
/plugin install czlonkowski/n8n-skills
```

### 4. Access n8n Instance
n8n instance: https://server.lawayraserver.com

## Building n8n Workflows

### ALWAYS Use n8n-mcp + n8n-skills

**Search for nodes:**
```
You: "Find the Spotify node in n8n"
Claude: [Returns full Spotify node documentation with examples]
```

**Check workflow examples:**
```
You: "Show me workflow examples for API pagination"
Claude: [Returns patterns from 2,646 real-world workflows]
```

### Critical n8n Patterns for THIS Instance

**NEVER use If nodes** — they are broken on v1.123.7. Use Code nodes instead:
```javascript
// Instead of If node, use a Code node to filter:
const results = [];
for (const item of $input.all()) {
  if (item.json.guest_name && item.json.guest_name !== 'SOLO') {
    results.push(item);
  }
}
return results;
```

**Code Node Return Format:**
```javascript
// CORRECT: Always return array of objects with json property
return [{json: {result: "success", data: processedData}}];

// WRONG:
return {result: "success"};  // Missing array wrapper and json property
```

**HTTP Request vs Code Node for API calls:**
```javascript
// Use Code node with this.helpers.httpRequest for API calls that need
// dynamic JSON bodies (avoids expression serialization issues):
// ALWAYS include timeout: 30000 to prevent hanging!
const response = await this.helpers.httpRequest({
  method: 'POST',
  url: 'https://api.anthropic.com/v1/messages',
  headers: { 'x-api-key': 'your-key', 'content-type': 'application/json' },
  body: { model: "claude-sonnet-4-20250514", messages: [...] },
  json: true,
  timeout: 30000  // CRITICAL: prevents task runner disconnects
});
```

**Batch Claude API Calls — NEVER call Claude in a tight loop:**
```javascript
// WRONG: One Claude call per item (causes task runner disconnect)
for (const title of titles) {
  await callClaude(title);  // 100 calls = hang + disconnect
}

// CORRECT: Batch multiple items per call
const BATCH_SIZE = 10;
for (let i = 0; i < titles.length; i += BATCH_SIZE) {
  const batch = titles.slice(i, i + BATCH_SIZE);
  const prompt = `Extract guest names from these episodes:\n${batch.join('\n')}`;
  await callClaude(prompt);  // 2-5 calls total
}
```

**Dedup Pattern — Read existing data on parallel branch:**
```javascript
// In a dedup Code node, reference the parallel Read node:
let existing = [];
try { existing = $('Read Existing Items').all(); } catch (e) {}
const existingSet = new Set(
  existing.map(i => (i.json['Key Column'] || '').toString().toLowerCase().trim())
);

// Filter incoming items
const seen = new Set(); // within-batch dedup
const results = [];
for (const item of $input.all()) {
  const key = (item.json.key_field || '').toLowerCase().trim();
  if (!key || existingSet.has(key) || seen.has(key)) continue;
  seen.add(key);
  results.push(item);
}
return results;
```

**Split In Batches — ALWAYS use output[1] for processing:**
```
output[0] = "done" — fires AFTER all batches complete
output[1] = "loop" — fires for EACH batch (this is what you want!)
```

**Test Nodes Individually:**
```
1. Build one node
2. Test it (Execute node button)
3. Verify output
4. Build next node
5. Repeat
Don't build entire workflow before testing!
```

## Google Sheets Schema

### Tab 1: LW Guests
| Column | Type | Description |
|--------|------|-------------|
| Guest Name | Text | Full name extracted by AI |
| Source Type | Enum | "own_guest" (all entries in this tab) |
| Original Episode Title | Text | For reference/verification |
| Original Podcast | Text | "Ayahuasca Podcast" |
| Extraction Date | Date | When guest was added |
| Verified | Checkbox | Manual review flag |
| Processed | Checkbox | Has podcast discovery (WF#2) run for this guest? |
| Notes | Text | Additional context |

### Tab 2: Discovered Podcasts
| Column | Type | Description |
|--------|------|-------------|
| Relevance Score | Number | AI-generated 1-10 score (higher = better fit for Ayahuasca Podcast) |
| Podcast Name | Text | Show title |
| Podcast ID | Text | PodcastIndex unique ID |
| Source Guest | Text | Which guest led to this discovery |
| Guest Source Type | Enum | "own_guest" (warm) or "external_guest" (discovery) |
| Discovery Date | Date | When podcast was found |
| Status | Enum | "pending" → "enriched" → "contacted" → "responded" |
| PodcastIndex URL | URL | Direct link to podcast page |
| Website | URL | Podcast's website (if found) |
| Email | Email | Contact email (if found) |
| Instagram | Text | Handle like @podcast_name |
| Pitch Generated | Checkbox | Has AI created personalized pitch? |
| Pitch Text | Text | The generated pitch content |
| Email Subject | Text | AI-generated subject line |
| Contacted Date | Date | When outreach was sent |
| Contact Method | Enum | "email" or "instagram" |
| Response Received | Checkbox | Did they reply? |
| Response Date | Date | When they replied |
| Response Type | Enum | "Interested", "Not Interested", "No Response", "Bounce" |
| Booking Status | Enum | "Scheduled", "Completed", "Declined", "In Discussion" |
| Follow-Up Count | Number | 0, 1, or 2 |
| Notes | Text | Additional context |
| Processed | Checkbox | Has WF#3 (guest extraction) run for this podcast? |

### Tab 3: External Guests
| Column | Type | Description |
|--------|------|-------------|
| Guest Name | Text | Full name extracted by AI |
| Source Podcast | Text | Which discovered podcast this guest is from |
| Source Podcast ID | Text | PodcastIndex feed ID |
| Original Episode Title | Text | Episode they appeared on |
| Extraction Date | Date | When guest was added |
| Verified | Checkbox | Manual review flag |
| Processed | Checkbox | Has further processing run? |
| Notes | Text | Additional context |

### Tab 4: Analytics
| Metric | Value Type | Description |
|--------|------------|-------------|
| Total Guests Extracted (Funnel A) | Number | LaWayra's own guests |
| Total Guests Extracted (Funnel B) | Number | External podcast guests |
| Total Unique Guests | Number | After deduplication |
| Total Podcasts Discovered | Number | Unique podcasts found |
| Podcasts with Email Found | Number | Contact info hit rate |
| Podcasts with Instagram Found | Number | Social media hit rate |
| Total Emails Sent | Number | Cumulative outreach |
| Emails Sent This Week | Number | Weekly tracking |
| Total Responses Received | Number | Replies to outreach |
| Response Rate (%) | Percentage | % of emails that got replies |
| Podcast Appearances Booked | Number | Scheduled interviews |
| Podcast Appearances Completed | Number | Finished interviews |
| Warm Lead Performance (own_guest) | Percentage | Response rate for Funnel A |
| Discovery Lead Performance (external_guest) | Percentage | Response rate for Funnel B |
| Average Days to Response | Number | How long before they reply |
| Current Week Number | Number | For warmup tracking (1-5+) |
| Daily Email Limit | Number | Current sending limit (10 → 30) |

## Important Notes

### Email Sending (10/day flat)
Sam requested a flat 10 emails/day limit (no warmup ramp). Monitor deliverability and bounce rates. Can increase later if needed.

### API Rate Limits
- **PodcastIndex.org**: Free, no published rate limits (much better than Listen Notes)
- **YouTube Data API**: 10,000 units/day quota (playlist items = 1 unit each)
- **Anthropic Claude**: Pay-per-use, ~$0.003-0.01 per guest extraction
- **Strategy**: PodcastIndex has no limits, so discovery is essentially unlimited

### Manual Review Requirements
- **Guest Extraction**: Check first 20 extractions (expected 85-90% accuracy)
- **Pitch Generation**: Review first 10 pitches for quality and tone
- **Email Sending**: Monitor first week for spam/bounce issues

### Security
- **Never commit**: `api-credentials.env` to git (already in .gitignore)
- **API Keys**: Store in environment variables, never hardcode in workflow JSON
- **n8n Credentials**: Use n8n's credential system for Google Sheets OAuth

## Reference Data Files (for WF#4 pitch generation)

| File | Description |
|------|-------------|
| `sam-bio.txt` | Detailed profile with 5 persona modes, biographical background, discussion hooks, notable guest names, required links, tone rules |
| `sam-guest-list.txt` | 114 guest names from Sam's Ayahuasca Podcast (for social proof matching) |
| `sam-past-appearances.txt` | 37 podcasts Sam has appeared on (for exclusion / dedup) |
| `sam-pitch-prompt.txt` | Pitch strategy with 3 personas, 2 email templates, IG DM versions, workflow integration notes |

## Gemini API Integration (for WF#4 Step 1)

**Purpose**: Real-time web search grounding to analyze target podcasts before generating pitches.

**Setup** (Sam needs to do this):
1. Go to https://aistudio.google.com/apikey
2. Sign in as admin@lawayra.com
3. Click "Create API Key"
4. Add to `api-credentials.env` as `GEMINI_API_KEY=...`

**API Pattern** (used in WF#4 Code node):
```javascript
const response = await this.helpers.httpRequest({
  method: 'POST',
  url: `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${GEMINI_API_KEY}`,
  headers: { 'Content-Type': 'application/json' },
  body: {
    contents: [{ parts: [{ text: prompt }] }],
    tools: [{ google_search: {} }]  // enables real-time web search
  },
  json: true,
  timeout: 30000
});
```

## Resuming Work — Quick Start

### Current State (as of 2026-03-15):
- **WF#1**: Ayahuasca Podcast guest extraction (own_guest only). Needs Google Sheets credential re-set after API update.
- **WF#2**: Updated with expanded relevance scoring + Sam's past appearances exclusion list. Needs Google Sheets credential re-set.
- **WF#3**: No changes this session. Working.
- **WF#4**: Rebuilt with Gemini + Claude multi-step pitch (9 nodes). Needs Google Sheets credential re-set.
- **WF#5**: Updated to flat 10 emails/day. Needs Google Sheets credential re-set. NOT tested — awaiting pitch approval.
- **After any API update**: You MUST re-set Google Sheets credentials in the n8n UI (API updates overwrite them).

### IMMEDIATE NEXT STEPS:
1. **Re-set Google Sheets credentials** in n8n UI for WF#1, WF#2, WF#4, and WF#5 (API updates on 2026-03-15 overwrote them)
2. **Test WF#1** (Ayahuasca Podcast guest extraction)
5. **Run full pipeline** WF#1 → WF#2 → WF#3 → WF#4
6. **Sam reviews pitches** in Discovered Podcasts tab
7. **Test WF#5** with 1-2 emails, then enable schedule trigger

### Set Google Sheets Credentials (REQUIRED after API updates):
For workflows #1, #2, #4, and #5 (updated on 2026-03-15):
1. Open the workflow in n8n at https://server.lawayraserver.com
2. Click on EVERY Google Sheets node in the workflow
3. Select "Google Sheets account" from the credential dropdown
4. Verify all Split In Batches → processing connections are on output[1] ("loop")
5. Save with Ctrl+S
6. **DO NOT push API updates after saving** (overwrites credentials)

### Before WF#5 Production:
1. Have Sam review generated pitches in the "Discovered Podcasts" tab
2. Set `BYPASS_DAY_CHECK = false` in Check Sending Day node
3. Set up Drip automation triggered by `send-podcast-pitch` tag
4. Test with 1-2 emails first, then enable schedule trigger

### PodcastIndex API (used in WF#2, #3, #4):
```
Base URL: https://api.podcastindex.org/api/1.0
Auth: Custom headers (X-Auth-Key, X-Auth-Date, Authorization with SHA-1 HMAC)
Key endpoints:
  /search/byperson?q={guest_name} — Find podcasts where a person appeared
  /search/byterm?q={term} — Find podcasts by keyword
  /podcasts/byfeedid?id={id} — Get podcast details
  /episodes/byfeedid?id={id} — Get episodes for a podcast
```

## Resources

- **n8n Instance**: https://server.lawayraserver.com
- **n8n-mcp GitHub**: https://github.com/czlonkowski/n8n-mcp
- **n8n-skills GitHub**: https://github.com/czlonkowski/n8n-skills
- **n8n API Docs**: https://docs.n8n.io/api/
- **LaWayra Podcast**: https://ayahuascapodcast.com
- **PodcastIndex API Docs**: https://podcastindex-org.github.io/docs-api/
- **Google Sheets**: [LaWayra Podcast Outreach (all tabs)](https://docs.google.com/spreadsheets/d/16l7E6Cox0fT5ktCXLjdJA6IWkY7P3EEWGg2hViYKwIY/edit)

## Common Tasks

### Add a New Seed Podcast (Funnel B)
1. Find podcast on Spotify
2. Get Spotify Show ID: Share → Copy Show Link → Extract ID from URL
3. Edit `seed-podcasts.json`:
   ```json
   {
     "name": "The Third Wave Podcast",
     "spotify_show_id": "abc123xyz",
     "notes": "Psychedelic wellness focus",
     "priority": 3
   }
   ```
4. Open n8n → External_Podcast_Guest_Extraction workflow
5. Run manual trigger for new podcast
6. Verify guests added to Google Sheet

### Debug Workflow Issues

**Step 1: Check n8n Execution Logs**
```
n8n UI → Executions tab → Find failed execution → Click to see error details
```

**Step 2: Test Individual Nodes**
```
Click on failing node → Click "Execute node" button → Check output in bottom panel
```

**Step 3: Common Issues on THIS Instance:**
- "Cannot read properties of undefined (reading 'execute')" → typeVersion too high, revert
- If node routing everything to FALSE → BROKEN on this version, use Code node instead
- "JSON parameter needs to be valid JSON" → Use Code node with this.helpers.httpRequest
- Google Sheets credential error → Must set credential manually in UI, not via API
- Batch node dead-ending → Check connection is on output[1] not output[0]
- Read node returns 0 items → Sheet is empty, downstream nodes won't fire
- "this.getWorkflowStaticData is not a function" → NOT available in Code nodes, use alternatives
- Task runner disconnect / Node execution failed → Add `timeout: 30000` to all HTTP requests, batch Claude API calls
- Node hangs indefinitely → Missing HTTP timeout, add `timeout: 30000`
- Empty rows in Google Sheets → `_skip` items leaking to Append node, add Skip Empty filter node before Append

---

Last Updated: 2026-03-15
Project Status: All 5 workflows updated. WF#1 back to Ayahuasca Podcast (own_guest only), WF#2 expanded relevance + exclusion list, WF#4 rebuilt with Gemini + Claude multi-step pitch, WF#5 set to 10/day. Tab renamed back to "LW Guests". Google Sheets credentials need re-setting in n8n UI for WF#1, #2, #4, #5. Next: Re-set credentials → test full pipeline → Sam reviews pitches → test WF#5.
