# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **n8n workflow configuration project**, not a traditional code project. It contains a single workflow (`workflow.json`) that generates competitive intelligence presentation decks using AI.

**Pipeline:** Webhook → [Firecrawl + ScrapeCreators + DataForSEO] (8 parallel calls) → Claude via OpenRouter (analyze) → Gamma Remix API (fill template with data) → Response

**Note:** This workflow uses Gamma's **Remix API** (`/from-template`) which requires a pre-designed template deck. See "Setting Up Gamma Template" below.

## Setting Up Gamma Template

Before running this workflow, you must create a template deck in Gamma:

1. Go to [gamma.app](https://gamma.app) and create a new presentation
2. Design a 10-slide competitive playbook template with your branding
3. Use placeholder content that matches the slide structure in `prompts/deck-structure.md`
4. Copy the deck ID from the URL: `gamma.app/docs/YOUR_TEMPLATE_ID`
5. Replace `YOUR_TEMPLATE_ID_HERE` in the workflow's "Gamma - Remix Template" node

The template ID goes in the `gammaId` field of the API request. The workflow will fill your template with competitor analysis data while preserving your design.

## Working With This Project

### Via n8n MCP (Preferred)

This project is configured to use the n8n MCP server for workflow management. The MCP connection is configured in `.mcp.json`.

**Common operations:**
```
# Check n8n connection
mcp__n8n-mcp__n8n_health_check

# List workflows
mcp__n8n-mcp__n8n_list_workflows

# Get workflow details
mcp__n8n-mcp__n8n_get_workflow with id

# Update workflow
mcp__n8n-mcp__n8n_update_full_workflow with id and full workflow JSON
mcp__n8n-mcp__n8n_update_partial_workflow with id and operations array
```

### Via n8n UI

1. Import `workflow.json` in n8n (Workflows → Import from File)
2. Configure credentials in Settings → Credentials:
   - `firecrawlApi` - Header Auth with `Authorization: Bearer <key>`
   - `scrapeCreatorsApi` - Header Auth with `x-api-key: <key>`
   - `dataForSeoApi` - Header Auth with `Authorization: Basic <base64>`
   - `openRouterApi` - Header Auth with `Authorization: Bearer <key>`
   - `gammaApi` - Header Auth with `X-API-KEY: <key>`
3. Activate the workflow

### Testing

Trigger via webhook:
```bash
curl -X POST "YOUR_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://competitor-website.com",
    "linkedin_handle": "competitor",
    "instagram_handle": "competitor",
    "facebook_handle": "competitor"
  }'
```

Or test in n8n UI: Click Webhook Trigger node → Test workflow panel → Enter `{"body": {"url": "https://example.com", "linkedin_handle": "example"}}`

Note: Only `url` is required. Social handles are optional for enriched analysis.

## Workflow Architecture

```
[Webhook Trigger]
       ↓ (parallel fan-out - 8 calls)
       ├─→ [Firecrawl - Homepage]
       ├─→ [Firecrawl - About Page] (continueOnFail)
       ├─→ [Firecrawl - Pricing Page] (continueOnFail)
       ├─→ [Firecrawl - Blog] (continueOnFail)
       ├─→ [ScrapeCreators - LinkedIn Company] (continueOnFail)
       ├─→ [ScrapeCreators - Instagram Profile] (continueOnFail)
       ├─→ [ScrapeCreators - Meta Ad Library] (continueOnFail)
       └─→ [DataForSEO - Domain Overview] (continueOnFail)
              ↓ (merge)
       [Merge Website Data]
              ↓
       [Process All Data] (Code node - structures all data sources)
              ↓
       [Claude via OpenRouter] (HTTP Request to OpenRouter API)
              ↓
       [Parse Analysis JSON] (Code node)
              ↓
       [Format Gamma Prompt] (Code node - 10 slides)
              ↓
       [Gamma - Remix Template] → [Wait] → [Poll Status] → [Response]
```

**21 nodes total:** 1 webhook trigger, 8 data source HTTP requests, 1 merge, 3 code nodes, 2 Gamma HTTP requests, 2 wait nodes, 1 if node, 1 webhook response

## Key Files

| File | Purpose |
|------|---------|
| `workflow.json` | Main n8n workflow definition (import this into n8n) |
| `prompts/analysis.md` | Claude analysis prompt template and expected JSON schema |
| `prompts/deck-structure.md` | Gamma API spec and slide-by-slide template |

## Modifying the Workflow

### Changing Analysis Focus
Edit the Claude prompt in the `Claude - Competitive Analysis` node's `jsonBody` parameter. The prompt is embedded directly in the HTTP request body.

### Changing Deck Structure
Edit the `Format Gamma Prompt` code node. The `gammaPrompt` template variable defines each slide's content.

### Adding Data Sources
Add new HTTP Request nodes after the Webhook Trigger and connect them to the Merge node. Update the `Process Website Data` code node to incorporate the new data.

## API Dependencies

| API | Purpose | Auth | Rate Limits |
|-----|---------|------|-------------|
| Firecrawl | Website scraping | Bearer token | Check dashboard |
| ScrapeCreators | LinkedIn, Instagram, Meta Ads | x-api-key header | 100 free calls |
| DataForSEO | SEO metrics | Basic auth | Pay per call |
| OpenRouter | Claude AI analysis | Bearer token | Per-model limits |
| Gamma | Deck generation via Remix API (Pro required) | X-API-KEY header | Async with polling |

## Error Handling

- Firecrawl: `retryOnFail: true` on homepage, `continueOnFail: true` on subpages (workflow continues even if /about, /pricing, /blog fail)
- ScrapeCreators: `continueOnFail: true` on all nodes (missing social handles return empty data)
- DataForSEO: `continueOnFail: true` (invalid domains return empty data)
- OpenRouter/Claude: 120s timeout for complex analyses
- Gamma: Polling loop checks status every 5-10s until `status === "completed"`
