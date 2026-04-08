# Podcast Guest Outreach Automation

Automated podcast guest discovery and outreach system for LaWayra (Ayahuasca Podcast). Scales from manual 1-2 pitches/day to systematic 20-30/day using n8n workflows, PodcastIndex API, and Claude AI.

## System Overview

**Two-Funnel Approach:**
- **Funnel A (Warm Leads)**: Find where LaWayra's ~90 podcast guests have appeared → reference shared guest in pitch
- **Funnel B (Discovery Leads)**: Extract guests from discovered podcasts → find where they've appeared

**Pipeline:** Guest Extraction → Podcast Discovery → Enrichment → Pitch Generation → Automated Outreach

**Why it works:** Shared guest connections dramatically increase response rates. Instead of cold outreach, Sam references mutual connections.

## Current Status

| Workflow | Status | Description |
|----------|--------|-------------|
| WF#1 — Guest Extraction | Tested & working | Extracts guests from YouTube episodes via Claude AI |
| WF#2 — Podcast Discovery | Tested & working | Finds podcasts via PodcastIndex + AI relevance scoring |
| WF#3 — External Guest Extraction | Tested & working | Extracts guests from discovered podcasts |
| WF#4 — Enrichment & Pitch | Tested & working | Enriches podcast data + generates personalized pitches |
| WF#5 — Outreach via Drip | Ready, not tested | Sends emails via Drip API (awaiting Sam's pitch approval) |

**Next Steps:**
1. Sam reviews generated pitches in the Discovered Podcasts sheet
2. Test WF#5 with 1-2 emails
3. Begin email warmup schedule (5/day → 30/day over 5 weeks)

## Tech Stack

- **n8n** v1.123.7 (Self-Hosted) — workflow automation at https://server.lawayraserver.com
- **PodcastIndex.org** — podcast discovery API (free, unlimited)
- **Claude AI (Anthropic)** — guest extraction + pitch generation + relevance scoring
- **YouTube Data API** — LaWayra episode fetching
- **Drip** — email outreach automation
- **Google Sheets** — centralized data storage (1 spreadsheet, 4 tabs)

## n8n Workflows

### WF#1 — LaWayra Guest Extraction (`f9sEhiNep0ISobCO`)
Fetches LaWayra YouTube episodes, uses Claude AI to extract guest names, and appends to "LW Guests" tab. Includes deduplication against existing entries.

### WF#2 — Podcast Discovery (`WIctYIYrw0YpCPXU`)
For each unprocessed guest, searches PodcastIndex to find podcasts they've appeared on. Includes:
- **AI Relevance Scoring**: Claude rates each podcast 1-10 for fit with Ayahuasca Podcast
- **Configurable threshold**: `RELEVANCE_THRESHOLD` constant (default: 5) in Score Relevance node
- **Deduplication**: Checks Podcast ID against existing entries

### WF#3 — External Guest Extraction (`NTm9HMU2TD2KBZ6L`)
For each unprocessed discovered podcast, fetches episodes from PodcastIndex and uses Claude AI to extract guest names. Includes:
- **Batched Claude calls**: 10 episode titles per API call (prevents task runner timeouts)
- **Deduplication**: Checks guest names (case-insensitive) against existing entries

### WF#4 — Enrichment & Pitch (`4IDSEo7inrGtNWs9`)
For each pending podcast, enriches with PodcastIndex details (website, email, social) and generates personalized pitches via Claude AI.

### WF#5 — Outreach via Drip (`fZmpONlaDqAG2TrS`)
Sends personalized outreach emails via Drip API with warmup schedule:
- Week 1: 5-10/day → Week 5+: 25-30/day
- Runs Tue/Wed/Thu at 9AM (configurable)
- Random 2-4 min delay between sends

## Google Sheets

**Spreadsheet**: [LaWayra Podcast Outreach](https://docs.google.com/spreadsheets/d/16l7E6Cox0fT5ktCXLjdJA6IWkY7P3EEWGg2hViYKwIY/edit)

| Tab | Used By | Description |
|-----|---------|-------------|
| LW Guests | WF#1 writes, WF#2 reads | LaWayra's podcast guests |
| Discovered Podcasts | WF#2 writes, WF#3/4/5 read/update | Podcasts found via guest search |
| External Guests | WF#3 writes | Guests from discovered podcasts |
| Analytics | Dashboard | Outreach performance metrics |

## Key Features

### Deduplication (WF#1, WF#2, WF#3)
Each workflow reads existing sheet data on a parallel branch at startup, then filters incoming items against existing entries before appending. Prevents duplicate rows when workflows are run multiple times or when multiple guests share the same podcast appearances.

### AI Relevance Scoring (WF#2)
Claude AI scores each discovered podcast 1-10 based on relevance to the Ayahuasca Podcast's topics (psychedelics, consciousness, wellness, indigenous wisdom). Only podcasts meeting the threshold are added to the sheet. Adjust `RELEVANCE_THRESHOLD` in the Score Relevance node.

### HTTP Timeout Protection (WF#3, WF#4, WF#5)
All API calls include 30-second timeouts to prevent indefinite hanging and task runner disconnects.

## Cost Breakdown

| Service | Cost | Purpose |
|---------|------|---------|
| PodcastIndex.org | Free | Podcast discovery (unlimited) |
| YouTube Data API | Free | Episode data (10K units/day) |
| Claude AI (Anthropic) | ~$10-30/mo | Guest extraction + pitches + scoring |
| Drip | Existing | Email campaigns |
| **Total** | **~$10-30/mo** | |

## Development

See [CLAUDE.md](CLAUDE.md) for detailed technical documentation including:
- Known n8n v1.123.7 issues and workarounds
- Workflow architecture diagrams
- Google Sheets schema
- API credential setup
- Common debugging patterns

### Quick Dev Setup
```bash
# 1. Install n8n-mcp server
npx n8n-mcp

# 2. Access n8n instance
open https://server.lawayraserver.com
```

## Important Notes

- **Never commit** `api-credentials.env` to git
- **Google Sheets credentials** must be set manually in the n8n UI after any API-based workflow update
- **If nodes are broken** on n8n v1.123.7 — all workflows use Code nodes for filtering instead
- **WF#5 requires Sam's approval** before testing (it sends real emails)
- **Email warmup is critical** — do not skip or you'll trigger spam filters

---

Last Updated: 2026-02-22
