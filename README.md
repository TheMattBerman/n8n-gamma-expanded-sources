# Competitor Playbook Generator

**Steal any competitor's playbook in 60 seconds.**

An n8n workflow that takes any competitor URL and generates a comprehensive Gamma presentation deck with their full playbook - positioning, messaging, content strategy, ad psychology, and gaps you can exploit.

## What You Get

Input a competitor URL → Get a professional Gamma deck containing:

1. **Executive Summary** - Threat level and top 5 insights
2. **Positioning Analysis** - Value prop, audience, differentiators
3. **Messaging Breakdown** - Brand voice, hooks, proof elements
4. **Content Strategy** - Pillars, formats, top content
5. **Ad Psychology** - Hook patterns, visual style, CTAs
6. **Gaps & Opportunities** - Where they're vulnerable
7. **How to Beat Them** - Quick wins, positioning plays, ad ideas
8. **Next Steps** - CTA to book a call

## Requirements

### APIs Needed

| Service | Purpose | Get API Key |
|---------|---------|-------------|
| **Firecrawl** | Website scraping | [firecrawl.dev](https://firecrawl.dev) |
| **Anthropic (Claude)** | AI analysis | [console.anthropic.com](https://console.anthropic.com) |
| **Gamma** | Deck generation | [gamma.app](https://gamma.app) (Pro required) |

### n8n Setup

- n8n Cloud or self-hosted n8n instance
- Version 1.0+ recommended

## Installation

### Step 1: Import the Workflow

1. Open your n8n instance
2. Go to **Workflows** → **Import from File**
3. Select `workflow.json` from this folder
4. Click **Import**

### Step 2: Set Up Credentials

Create these credentials in n8n (**Settings** → **Credentials**):

#### Firecrawl API
- Type: Header Auth
- Name: `firecrawlApi`
- Header Name: `Authorization`
- Header Value: `Bearer YOUR_FIRECRAWL_API_KEY`

#### Anthropic API
- Type: Header Auth
- Name: `anthropicApi`
- Header Name: `x-api-key`
- Header Value: `YOUR_ANTHROPIC_API_KEY`

#### Gamma API
- Type: Header Auth
- Name: `gammaApi`
- Header Name: `X-API-KEY`
- Header Value: `YOUR_GAMMA_API_KEY`

### Step 3: Update Credential References

In each HTTP Request node, update the credential expressions:
- Replace `$credentials.firecrawlApi.apiKey` with your actual credential name
- Replace `$credentials.anthropicApi.apiKey` with your actual credential name
- Replace `$credentials.gammaApi.apiKey` with your actual credential name

### Step 4: Activate the Workflow

1. Click **Activate** to enable the webhook
2. Copy the webhook URL (shown in the Webhook Trigger node)

## Usage

### Via Webhook (API)

```bash
curl -X POST "YOUR_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://competitor-website.com"}'
```

### Response

```json
{
  "success": true,
  "deck_url": "https://gamma.app/docs/competitor-playbook-abc123",
  "deck_id": "abc123",
  "company_analyzed": "Competitor Name",
  "generated_at": "2025-01-14T12:00:00.000Z",
  "message": "Competitive playbook generated successfully!"
}
```

### Via n8n UI

1. Open the workflow
2. Click on the **Webhook Trigger** node
3. In the test panel, enter:
   ```json
   {
     "body": {
       "url": "https://competitor-website.com"
     }
   }
   ```
4. Click **Test workflow**

## Customization

### Change the Analysis Focus

Edit the Claude prompt in the `Claude - Competitive Analysis` node to:
- Add industry-specific analysis
- Focus on specific competitor aspects
- Include additional data sources

### Modify Deck Structure

Edit the `Format Gamma Prompt` node to:
- Change slide order
- Add/remove slides
- Customize messaging

### Add More Data Sources

The workflow is designed to be extended with:
- **Social scraping** - Add ScrapeCreators or Apify nodes
- **Ad library data** - Add Meta Ad Library scraping
- **SEO data** - Add DataForSEO or similar

## Workflow Architecture

```
[Webhook] → [Firecrawl x4] → [Merge] → [Claude] → [Gamma] → [Response]
     │           │              │          │          │          │
     │           ├─ Homepage    │          │          │          │
     │           ├─ About       │          │          │          │
     │           ├─ Pricing     │          │          │          │
     │           └─ Blog        │          │          │          │
     │                          │          │          │          │
     │                     Combines    Analyzes   Generates   Returns
     │                     scraped     data &     Gamma       deck URL
     └─ Receives URL       data        outputs    deck        to caller
                                       JSON
```

## Troubleshooting

### Firecrawl Errors
- Some pages may fail to scrape (404, blocked, etc.)
- The workflow continues even if individual pages fail
- Check Firecrawl dashboard for usage limits

### Claude Errors
- Ensure your Anthropic API key is valid
- Check for rate limits if running many analyses
- Timeout is set to 120s for complex analyses

### Gamma Errors
- Verify you have a Gamma Pro subscription
- Check API key is correct
- Generation may take 30-60 seconds

### Polling Loop
- If deck never completes, check Gamma dashboard
- Maximum wait time before timeout: ~2 minutes
- Failed generations return error status

## Files Included

| File | Description |
|------|-------------|
| `workflow.json` | Main n8n workflow (import this) |
| `README.md` | This setup guide |
| `prompts/analysis.md` | Claude prompt documentation |
| `prompts/deck-structure.md` | Gamma deck template spec |
| `content/announcement-post.md` | Social media announcement draft |

## Support

Built by [@themattberman](https://x.com/themattberman) in partnership with [Gamma](https://gamma.app).

Questions? DM me on X or comment on the giveaway post.

## License

Free to use and modify. Attribution appreciated but not required.

#gammapartner
