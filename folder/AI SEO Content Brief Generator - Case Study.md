# Case Study: AI SEO Content Brief Generator

**Built by:** Christopher Christian Cosingan  
**Tools Used:** n8n, Claude Haiku (Anthropic), Google Sheets  
**Type:** AI Process Automation for SEO  

---

## The Problem

SEO teams spend a significant amount of time manually researching and writing content briefs before a single word is written. A typical brief includes keyword research, competitor analysis, content structure, meta copy, and CTA planning — tasks that are repetitive and time-consuming when done for every article.

For an agency managing dozens of content pieces per month, this becomes a bottleneck.

---

## The Solution

An automated n8n workflow that accepts a keyword, target audience, and content type, then uses Claude AI to instantly generate a full SEO content brief — and saves it directly to Google Sheets for the team to access.

**One API call. Under 10 seconds. Zero manual work.**

---

## How It Works

```
POST /webhook/seo-brief
        ↓
Claude Haiku (AI SEO Strategist)
        ↓
Parse & Format JSON Output
        ↓
Save Row to Google Sheets
        ↓
Return JSON Response
```

### Input (sent via Postman or any HTTP client)

```json
{
  "keyword": "local SEO for restaurants",
  "target_audience": "restaurant owners in the US",
  "content_type": "blog post"
}
```

### Output (returned in under 10 seconds)

```json
{
  "success": true,
  "generated_at": "2026-05-12T08:30:00.000Z",
  "brief": {
    "keyword": "local SEO for restaurants",
    "seo_title_suggestions": [
      "Local SEO for Restaurants: The Complete 2026 Guide",
      "How to Get Your Restaurant Found on Google (Local SEO Tips)",
      "Restaurant Local SEO: 10 Tactics That Actually Work"
    ],
    "meta_description": "Learn how to dominate local search results with proven SEO strategies built specifically for restaurant owners. More reservations, less guesswork.",
    "target_word_count": 1800,
    "search_intent": "informational",
    "secondary_keywords": [
      "restaurant Google My Business",
      "how to rank on Google Maps",
      "local search optimization",
      "restaurant marketing online",
      "Google reviews for restaurants"
    ],
    "lsi_keywords": [
      "NAP consistency",
      "local citations",
      "near me searches"
    ],
    "content_outline": [
      { "section": "Introduction", "notes": "Hook with stat on how many people search for restaurants near them before visiting. Introduce local SEO as the solution." },
      { "section": "Why Local SEO Matters for Restaurants", "notes": "Cover foot traffic impact, near-me search growth, and competitive advantage." },
      { "section": "Setting Up and Optimizing Google My Business", "notes": "Step-by-step: categories, photos, hours, posts, Q&A." },
      { "section": "Building Local Citations and NAP Consistency", "notes": "Explain NAP, list top directories (Yelp, TripAdvisor, etc.), and how inconsistency hurts rankings." },
      { "section": "FAQs", "notes": "Q: How long does local SEO take? Q: Do reviews affect ranking? Q: Is Google My Business free?" },
      { "section": "Conclusion", "notes": "Summarize key actions, reinforce urgency, lead into CTA." }
    ],
    "internal_link_opportunities": [
      "Google My Business optimization guide",
      "How to get more 5-star reviews"
    ],
    "cta_suggestion": "Get a free local SEO audit for your restaurant"
  }
}
```

---

## What Gets Saved to Google Sheets

Every run automatically appends a new row:

| Date | Keyword | Search Intent | Word Count | Title 1 | Meta Description | Secondary KWs | CTA |
|------|---------|--------------|-----------|---------|-----------------|--------------|-----|
| 2026-05-12 | local SEO for restaurants | informational | 1800 | Local SEO for Restaurants: The Complete 2026 Guide | Learn how to dominate local search... | restaurant Google My Business, how to rank on Google Maps... | Get a free local SEO audit |

The full content outline and all keywords are stored in their respective columns, giving the content team a ready-to-use reference without leaving their spreadsheet.

---

## Results & Benefits

| Before | After |
|--------|-------|
| 30–45 min per brief (manual) | Under 10 seconds (automated) |
| Inconsistent structure across writers | Standardized format every time |
| Briefs scattered across docs/emails | Centralized in one Google Sheet |
| Dependent on one person's availability | Available 24/7 via webhook |

---

## Why This Fits the Role

This workflow demonstrates exactly what the job requires:

- **n8n** — multi-node workflow with webhook, AI agent, code, and Google Sheets nodes
- **Claude AI** — used as the core intelligence for SEO strategy generation
- **Real business process** — solves an actual SEO team bottleneck
- **Scalable** — can be triggered by Zapier, Make, a form, or another n8n workflow
- **Extendable** — next steps could include Slack notifications, auto-draft in Google Docs, or SERP data enrichment via a search API
