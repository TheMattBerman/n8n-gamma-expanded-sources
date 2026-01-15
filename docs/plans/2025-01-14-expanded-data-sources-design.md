# Expanded Data Sources Design

**Date:** 2025-01-14
**Status:** Planned

## Overview

Expand the Competitor Playbook Generator workflow to include social media data, real ad creative data, and SEO intelligence. This enriches the competitive analysis with actual platform data instead of inferring from website content alone.

## New Data Sources

### ScrapeCreators API (3 calls)

| Call | Data Retrieved |
|------|----------------|
| LinkedIn Company | Follower count, about/tagline, recent 10-15 posts with engagement |
| Instagram Profile | Bio, follower count, recent 12-15 posts, engagement rates, content themes |
| Meta Ad Library | Active ads, ad count, creative types, copy/hooks, CTAs, run time |

### DataForSEO API (1 call)

| Call | Data Retrieved |
|------|----------------|
| Domain Overview | Traffic estimate, domain rank, top 5-10 organic keywords, backlink count |

### OpenRouter (replaces direct Anthropic)

| Change | Details |
|--------|---------|
| Endpoint | `https://openrouter.ai/api/v1/chat/completions` |
| Format | OpenAI-compatible (not Anthropic message format) |
| Benefits | Fallback models, unified billing, easy model swapping |

## Workflow Architecture

```
[Webhook Trigger]
       ↓ (parallel fan-out - 8 calls)
       ├─→ [Firecrawl - Homepage]
       ├─→ [Firecrawl - About Page]
       ├─→ [Firecrawl - Pricing Page]
       ├─→ [Firecrawl - Blog]
       ├─→ [ScrapeCreators - LinkedIn Company]    ← NEW
       ├─→ [ScrapeCreators - Instagram Profile]   ← NEW
       ├─→ [ScrapeCreators - Meta Ad Library]     ← NEW
       └─→ [DataForSEO - Domain Overview]         ← NEW
              ↓
       [Merge All Data] (8 inputs)
              ↓
       [Process All Data] (updated)
              ↓
       [Claude via OpenRouter] (updated)
              ↓
       ... rest unchanged ...
```

All new API calls use `continueOnFail: true` for graceful degradation.

## Deck Structure (10 Slides)

| # | Slide | Data Sources | Changes |
|---|-------|--------------|---------|
| 1 | Cover | - | No change |
| 2 | Executive Summary | All | No change |
| 3 | Positioning Analysis | Website + LinkedIn | LinkedIn adds social proof |
| 4 | Messaging Breakdown | Website + Social | Social hooks enrich analysis |
| 5 | Content Strategy | Website + LinkedIn + Instagram | **Expanded** with social metrics |
| 6 | Ad Psychology | Website + Meta Ad Library | **Enriched** with real ad data |
| 7 | SEO Intelligence | DataForSEO | **NEW SLIDE** |
| 8 | Gaps & Opportunities | All | No change |
| 9 | How to Beat Them | All | No change |
| 10 | CTA | - | No change |

## New Credentials

### ScrapeCreators API
- Type: Header Auth
- Name: `scrapeCreatorsApi`
- Header Name: `x-api-key`
- Header Value: `YOUR_SCRAPECREATORS_API_KEY`

### DataForSEO API
- Type: Header Auth
- Name: `dataForSeoApi`
- Header Name: `Authorization`
- Header Value: `Basic YOUR_BASE64_CREDENTIALS`

### OpenRouter API
- Type: Header Auth
- Name: `openRouterApi`
- Header Name: `Authorization`
- Header Value: `Bearer YOUR_OPENROUTER_API_KEY`

## HTTP Request Nodes to Add

| Node | Method | URL |
|------|--------|-----|
| ScrapeCreators - LinkedIn | POST | `https://api.scrapecreators.com/v1/linkedin/company` |
| ScrapeCreators - Instagram | POST | `https://api.scrapecreators.com/v1/instagram/profile` |
| ScrapeCreators - Meta Ads | POST | `https://api.scrapecreators.com/v1/meta/adlibrary` |
| DataForSEO - Domain Overview | POST | `https://api.dataforseo.com/v3/domain_analytics/overview/live` |

## Files to Modify

| File | Changes |
|------|---------|
| `workflow.json` | Add 4 HTTP nodes, update Merge (8 inputs), update Process Data code, update Claude node to OpenRouter |
| `README.md` | Add ScrapeCreators, DataForSEO, OpenRouter to requirements. Update credentials. Update architecture diagram. |
| `CLAUDE.md` | Update workflow architecture, add new credentials |
| `prompts/deck-structure.md` | Add SEO Intelligence slide (slide 7), update Content Strategy references |
| `prompts/analysis.md` | Document expected data formats for social/ads/seo fields |

## Future Enhancements

- **Founder/CEO LinkedIn profile** - Add personal brand analysis for founder-led companies

## Implementation Notes

- All 8 API calls run in parallel (no added latency vs current 4)
- New calls use `continueOnFail: true` matching existing pattern
- Claude prompt already has `social_data`, `ad_data`, `seo_data` placeholders
- SEO slide is inserted at position 7, pushing Gaps/Beat/CTA to 8/9/10
