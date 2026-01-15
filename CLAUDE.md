# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **n8n workflow configuration project**, not a traditional code project. It contains a single workflow (`workflow.json`) that generates competitive intelligence presentation decks using AI.

**Pipeline:** Webhook → Firecrawl (scrape competitor site) → Claude (analyze) → Gamma (generate deck) → Response

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
   - `anthropicApi` - Header Auth with `x-api-key: <key>`
   - `gammaApi` - Header Auth with `X-API-KEY: <key>`
3. Activate the workflow

### Testing

Trigger via webhook:
```bash
curl -X POST "YOUR_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://competitor-website.com"}'
```

Or test in n8n UI: Click Webhook Trigger node → Test workflow panel → Enter `{"body": {"url": "https://example.com"}}`

## Workflow Architecture

```
[Webhook Trigger]
       ↓ (parallel fan-out)
       ├─→ [Firecrawl - Homepage]
       ├─→ [Firecrawl - About Page] (continueOnFail)
       ├─→ [Firecrawl - Pricing Page] (continueOnFail)
       └─→ [Firecrawl - Blog] (continueOnFail)
              ↓ (merge)
       [Merge Website Data]
              ↓
       [Process Website Data] (Code node - combines scraped content)
              ↓
       [Claude - Competitive Analysis] (HTTP Request to Anthropic API)
              ↓
       [Parse Analysis JSON] (Code node - extracts JSON from Claude response)
              ↓
       [Format Gamma Prompt] (Code node - builds slide-by-slide markdown)
              ↓
       [Gamma - Generate Deck] (HTTP Request to Gamma API)
              ↓
       [Wait 10s]
              ↓
       [Gamma - Check Status] ←──┐
              ↓                   │
       [Is Complete?] ──No──→ [Wait & Retry]
              ↓ Yes
       [Format Success Response]
              ↓
       [Respond to Webhook]
```

**17 nodes total:** 1 webhook trigger, 4 Firecrawl HTTP requests, 1 merge, 3 code nodes, 2 Gamma HTTP requests, 2 wait nodes, 1 if node, 1 webhook response

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

| API | Purpose | Rate Limits |
|-----|---------|-------------|
| Firecrawl | Website scraping | Check dashboard |
| Anthropic (claude-sonnet-4-20250514) | Competitive analysis | 120s timeout configured |
| Gamma | Deck generation (Pro required) | Async with polling |

## Error Handling

- Firecrawl: `retryOnFail: true` on homepage, `continueOnFail: true` on subpages (workflow continues even if /about, /pricing, /blog fail)
- Claude: 120s timeout for complex analyses
- Gamma: Polling loop checks status every 5-10s until `status === "completed"`
