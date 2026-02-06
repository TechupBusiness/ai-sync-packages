---
name: apify
description: >-
  Use the Apify platform to run cloud actors for data collection: scrape social media
  (Twitter/X, Instagram, TikTok, LinkedIn, YouTube, Reddit, Facebook), Google Search/Maps,
  websites, and 2,500+ other services. Searches the Apify Store to find the cheapest actor
  for the user's goal, compares prices, and asks for confirmation before running.
  Use when the user wants to scrape, collect, or extract data from websites or social media.
allowed-tools: Bash(apify:*), Bash(curl:*)
triggers:
  - "apify"
  - "scrape"
  - "collect tweets"
  - "scrape instagram"
  - "social media data"
  - "web scraping"
  - "get data from"
---

# Apify Skill

Run cloud actors on the Apify platform for data collection tasks. Always find the cheapest option and confirm cost before running.

## Prerequisites

- **Apify CLI**: `npm install -g apify-cli`
- **Auth** (any of these):
  - `apify login` — opens browser for interactive login (easiest)
  - `apify login -t <token>` — direct token
  - `APIFY_TOKEN` env var
- Token from: Apify Console > Account Settings > Integrations
- Credentials stored in `~/.apify/auth.json`

If CLI is not installed, fall back to `curl` with REST API.

## Core Workflow

### 1. Parse User Intent

Identify: **platform/source**, **what to collect**, **how much** (maxItems), **filters** (date range, keywords, handles).

### 2. Search Store for Actors

```bash
curl -s "https://api.apify.com/v2/store?search=KEYWORDS&limit=10" | \
  jq '[.data.items[] | {
    id: (.username + "/" + .name),
    title: .title,
    pricing: .currentPricingInfo.pricingModel,
    price_per_1k: (if .currentPricingInfo.pricePerUnitUsd then (.currentPricingInfo.pricePerUnitUsd * 1000) else null end),
    unit: .currentPricingInfo.unitName,
    users_30d: .stats.totalUsers30Days,
    total_runs: .stats.totalRuns,
    rating: .stats.actorReviewRating
  }] | sort_by(.price_per_1k // 999)'
```

Filter by pricing model for easier comparison:
```bash
curl -s "https://api.apify.com/v2/store?search=KEYWORDS&pricingModel=PRICE_PER_DATASET_ITEM&limit=10"
```

### 3. Compare & Recommend

Present top 3 actors to user in a table:

```
| Actor | Price/1K results | Users (30d) | Rating |
|-------|-----------------|-------------|--------|
| ... | $X.XX | ... | .../5 |
```

Include:
- **Estimated total cost**: `maxItems * pricePerUnit`
- **Reliability signal**: prefer actors with >1K users_30d and >4.0 rating
- **Warning** if user is on Free tier (prices are 10-100x higher)

**Always ask for confirmation before running.**

### 4. Run the Actor

```bash
# Synchronous (waits for completion, prints results)
apify call ACTOR_ID -s -o --input='JSON_INPUT'

# For large jobs (>5 min), use async
apify actors start ACTOR_ID --input='JSON_INPUT'
# Then check: apify runs info RUN_ID --json
# Get results: apify datasets get-items DATASET_ID --format json
```

Without CLI installed:
```bash
# Sync run (returns data directly, max 5 min)
curl -s -X POST "https://api.apify.com/v2/acts/ACTOR_ID/run-sync-get-dataset-items" \
  -H "Authorization: Bearer $APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d 'JSON_INPUT'
```

### 5. Return Results

- Summarize key findings (count, highlights)
- Offer to save full results: `> results.json` or `--format csv > results.csv`

## Pricing Models

| Model | How It Works | Compare? |
|-------|-------------|----------|
| **PPR** (`PRICE_PER_DATASET_ITEM`) | Fixed $/result, compute included | Easy - use `pricePerUnitUsd` |
| **PPE** (`PAY_PER_EVENT`) | $/event (varies per event type) | Hard - check `pricingPerEvent` |
| **FREE** | No actor fee, pay compute (~$0.25-0.30/CU) | Estimate: ~$0.50-5.00/1K results |
| **Rental** (`FLAT_PRICE_PER_MONTH`) | Monthly subscription | Only for heavy recurring use |

**Always prefer PPR actors** for one-off tasks - cost is predictable.

## Quick Reference: Pre-Vetted Actors

Use these for common tasks to skip the search step. Still confirm price with user.

### Twitter/X

| Actor | Price/1K | Best For |
|-------|---------|---------|
| `scrape.badger/twitter-tweets-scraper` | $0.20 | Cheapest tweets (4.89 rating) |
| `kaitoeasyapi/...-cheapest` | $0.25 | Budget + popular (1.2K users/30d) |
| `danek/twitter-scraper-ppr` | $0.30 | Reliable, high rating (4.82, 15M runs) |

Input pattern:
```json
{"searchTerms": ["QUERY"], "maxItems": 100, "sort": "Latest"}
```
For threads: `{"conversationIds": ["TWEET_ID"], "maxItems": 500}`

### Instagram

| Actor | Price/1K | Best For |
|-------|---------|---------|
| `apify/instagram-scraper` | $0.50 | Posts, profiles, hashtags (176K users) |
| `sones/instagram-posts-scraper-lowcost` | $0.25 | Budget option |

Input: `{"directUrls": ["https://instagram.com/USERNAME"], "resultsLimit": 100}`

### TikTok

| Actor | Price/1K | Best For |
|-------|---------|---------|
| `clockworks/tiktok-scraper` | $0.30 | Videos, hashtags, profiles (123K users, dominant) |

Input: `{"profiles": ["USERNAME"], "resultsPerPage": 100}`

### YouTube

| Actor | Price/1K | Best For |
|-------|---------|---------|
| `apidojo/youtube-scraper` | $0.50 | Search results, channels (budget, 3.2M runs) |
| `scrape-creators/best-youtube-transcripts-scraper` | $1.00 | Transcripts (5.0 rating) |
| `streamers/youtube-scraper` | $5.00 | Most popular (3.7K users/30d, 11M runs, 4.72 rating) |

### LinkedIn

| Actor | Price/1K | Best For |
|-------|---------|---------|
| `curious_coder/linkedin-jobs-scraper` | $1.00 | Job postings |
| `supreme_coder/linkedin-profile-scraper` | $3.00 | Profiles (no login needed) |

### Reddit

| Actor | Price/1K | Best For |
|-------|---------|---------|
| `macrocosmos/reddit-scraper` | $0.50 | Cheapest, 1.9M runs, 5.0 rating |
| `benthepythondev/reddit-scraper` | $1.00 | Alternative |

### Facebook

| Actor | Price/1K | Best For |
|-------|---------|---------|
| `danek/facebook-comments-ppr` | $0.50 | Comments |
| `curious_coder/facebook-ads-library-scraper` | $0.75 | Ad library (1.7K users/30d, dominant) |
| `danek/facebook-search-ppr` | $2.59 | Search results (434 users/30d) |

### Telegram

| Actor | Price/1K | Best For |
|-------|---------|---------|
| `pamnard/telegram-channels-scraper` | FREE | Channel messages (5.0 rating, 51 users/30d) |
| `i-scraper/telegram-groups-scraper` | $0.30 | Group messages PPR |

### Discord

| Actor | Price/1K | Best For |
|-------|---------|---------|
| `louisdeconinck/discord-server-scraper` | $5.00 | Server discovery (PPR) |
| `curious_coder/discord-data-scraper` | $19/mo | Members + messages (rental, 605K runs) |

### Bluesky

| Actor | Price/1K | Best For |
|-------|---------|---------|
| `piotrv1001/bluesky-profile-posts-scraper` | $1.00 | Profile posts |
| `fatihtahta/All-In-One-Bluesky-Scraper` | ~$1.50 | All-in-one (PPE, 5.0 rating) |

### Google

| Actor | Price/1K | Best For |
|-------|---------|---------|
| `apify/google-search-scraper` | $0.50 | SERP results |
| `compass/crawler-google-places` | ~$4.00 | Google Maps (193K users, #1 on platform) |

### Web Crawling

| Actor | Price/1K | Best For |
|-------|---------|---------|
| `apify/website-content-crawler` | FREE (compute only) | Website → Markdown (LLM-ready) |
| `apify/web-scraper` | FREE (compute only) | General web scraping |

## Error Handling

- **"command not found: apify"**: Suggest `npm install -g apify-cli` or fall back to `curl`
- **401 Unauthorized**: Token missing/invalid. Check `APIFY_TOKEN` or run `apify login`
- **402 Payment Required**: Insufficient credits. Link: https://console.apify.com/billing
- **Actor run FAILED/TIMED-OUT**: Try with more memory (`-m 8192`) or longer timeout (`-t 600`)
- **Empty results**: Input may be wrong. Get actor's input schema: `apify actors info ACTOR_ID --input`
