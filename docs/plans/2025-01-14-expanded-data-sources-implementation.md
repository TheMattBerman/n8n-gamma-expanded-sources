# Expanded Data Sources Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add ScrapeCreators (LinkedIn, Instagram, Meta Ads), DataForSEO, and OpenRouter to the competitor playbook workflow.

**Architecture:** Four new HTTP Request nodes run in parallel with existing Firecrawl nodes. Data merges into a single processing node, feeds Claude via OpenRouter, generates a 10-slide deck with new SEO slide.

**Tech Stack:** n8n workflow, HTTP Request nodes, ScrapeCreators API, DataForSEO API, OpenRouter API, Gamma API

---

## Task 1: Add ScrapeCreators LinkedIn Company Node

**Files:**
- Modify: `workflow.json` (add node to `nodes` array, add connection from Webhook)

**Step 1: Add the LinkedIn Company node**

Add this node to the `nodes` array in `workflow.json` (after the Firecrawl nodes, around line 116):

```json
{
  "parameters": {
    "method": "GET",
    "url": "=https://api.scrapecreators.com/v1/linkedin/company?url=https://linkedin.com/company/{{ $json.body.linkedin_handle }}",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        {
          "name": "x-api-key",
          "value": "={{ $credentials.scrapeCreatorsApi.apiKey }}"
        }
      ]
    },
    "options": {
      "timeout": 30000
    }
  },
  "id": "scrapecreators-linkedin",
  "name": "ScrapeCreators - LinkedIn Company",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [250, 700],
  "continueOnFail": true
}
```

**Step 2: Add connection from Webhook Trigger**

In the `connections` object, update `"Webhook Trigger"` to include the new node:

```json
{
  "node": "ScrapeCreators - LinkedIn Company",
  "type": "main",
  "index": 0
}
```

**Step 3: Validate in n8n**

Run via n8n MCP:
```
mcp__n8n-mcp__n8n_get_workflow_structure
```
Expected: Node appears in workflow structure

**Step 4: Commit**

```bash
git add workflow.json
git commit -m "feat: add ScrapeCreators LinkedIn Company node"
```

---

## Task 2: Add ScrapeCreators Instagram Profile Node

**Files:**
- Modify: `workflow.json`

**Step 1: Add the Instagram Profile node**

Add this node to the `nodes` array:

```json
{
  "parameters": {
    "method": "GET",
    "url": "=https://api.scrapecreators.com/v1/instagram/profile?handle={{ $json.body.instagram_handle }}",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        {
          "name": "x-api-key",
          "value": "={{ $credentials.scrapeCreatorsApi.apiKey }}"
        }
      ]
    },
    "options": {
      "timeout": 30000
    }
  },
  "id": "scrapecreators-instagram",
  "name": "ScrapeCreators - Instagram Profile",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [250, 850],
  "continueOnFail": true
}
```

**Step 2: Add connection from Webhook Trigger**

Add to `"Webhook Trigger"` connections:

```json
{
  "node": "ScrapeCreators - Instagram Profile",
  "type": "main",
  "index": 0
}
```

**Step 3: Commit**

```bash
git add workflow.json
git commit -m "feat: add ScrapeCreators Instagram Profile node"
```

---

## Task 3: Add ScrapeCreators Meta Ad Library Node

**Files:**
- Modify: `workflow.json`

**Step 1: Add the Meta Ad Library node**

Add this node to the `nodes` array:

```json
{
  "parameters": {
    "method": "GET",
    "url": "=https://api.scrapecreators.com/v1/facebookadlibrary/profile?handle={{ $json.body.facebook_handle }}",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        {
          "name": "x-api-key",
          "value": "={{ $credentials.scrapeCreatorsApi.apiKey }}"
        }
      ]
    },
    "options": {
      "timeout": 30000
    }
  },
  "id": "scrapecreators-meta-ads",
  "name": "ScrapeCreators - Meta Ad Library",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [250, 1000],
  "continueOnFail": true
}
```

**Step 2: Add connection from Webhook Trigger**

Add to `"Webhook Trigger"` connections:

```json
{
  "node": "ScrapeCreators - Meta Ad Library",
  "type": "main",
  "index": 0
}
```

**Step 3: Commit**

```bash
git add workflow.json
git commit -m "feat: add ScrapeCreators Meta Ad Library node"
```

---

## Task 4: Add DataForSEO Domain Overview Node

**Files:**
- Modify: `workflow.json`

**Step 1: Add the DataForSEO node**

Add this node to the `nodes` array:

```json
{
  "parameters": {
    "method": "POST",
    "url": "https://api.dataforseo.com/v3/dataforseo_labs/google/domain_rank_overview/live",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        {
          "name": "Authorization",
          "value": "={{ $credentials.dataForSeoApi.apiKey }}"
        },
        {
          "name": "Content-Type",
          "value": "application/json"
        }
      ]
    },
    "sendBody": true,
    "specifyBody": "json",
    "jsonBody": "=[\n  {\n    \"target\": \"{{ $json.body.url.replace('https://', '').replace('http://', '').replace('www.', '').split('/')[0] }}\"\n  }\n]",
    "options": {
      "timeout": 30000
    }
  },
  "id": "dataforseo-overview",
  "name": "DataForSEO - Domain Overview",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [250, 1150],
  "continueOnFail": true
}
```

**Step 2: Add connection from Webhook Trigger**

Add to `"Webhook Trigger"` connections:

```json
{
  "node": "DataForSEO - Domain Overview",
  "type": "main",
  "index": 0
}
```

**Step 3: Commit**

```bash
git add workflow.json
git commit -m "feat: add DataForSEO Domain Overview node"
```

---

## Task 5: Update Merge Node for 8 Inputs

**Files:**
- Modify: `workflow.json`

**Step 1: Add connections from new nodes to Merge**

Add these entries to the `connections` object:

```json
"ScrapeCreators - LinkedIn Company": {
  "main": [
    [
      {
        "node": "Merge Website Data",
        "type": "main",
        "index": 4
      }
    ]
  ]
},
"ScrapeCreators - Instagram Profile": {
  "main": [
    [
      {
        "node": "Merge Website Data",
        "type": "main",
        "index": 5
      }
    ]
  ]
},
"ScrapeCreators - Meta Ad Library": {
  "main": [
    [
      {
        "node": "Merge Website Data",
        "type": "main",
        "index": 6
      }
    ]
  ]
},
"DataForSEO - Domain Overview": {
  "main": [
    [
      {
        "node": "Merge Website Data",
        "type": "main",
        "index": 7
      }
    ]
  ]
}
```

**Step 2: Commit**

```bash
git add workflow.json
git commit -m "feat: connect new data sources to Merge node"
```

---

## Task 6: Update Process Website Data Code Node

**Files:**
- Modify: `workflow.json` (update `jsCode` in `process-website-data` node)

**Step 1: Replace the jsCode**

Find the `process-website-data` node and replace its `jsCode` parameter:

```javascript
// Combine all data sources
const items = $input.all();

let data = {
  website: {
    homepage: '',
    about: '',
    pricing: '',
    blog: ''
  },
  social: {
    linkedin: null,
    instagram: null
  },
  ads: {
    meta: null
  },
  seo: null
};

// Process each input
for (const item of items) {
  const json = item.json;

  // Firecrawl website data
  if (json.data && json.data.markdown) {
    const url = json.data.metadata?.url || '';
    const content = json.data.markdown.substring(0, 15000);

    if (url.includes('/about')) {
      data.website.about = content;
    } else if (url.includes('/pricing')) {
      data.website.pricing = content;
    } else if (url.includes('/blog')) {
      data.website.blog = content;
    } else if (json.data.metadata?.url) {
      data.website.homepage = content;
    }
  }

  // ScrapeCreators LinkedIn
  if (json.name && json.employeeCount !== undefined) {
    data.social.linkedin = {
      name: json.name,
      description: json.description,
      slogan: json.slogan,
      employeeCount: json.employeeCount,
      website: json.website,
      location: json.location,
      posts: json.posts || []
    };
  }

  // ScrapeCreators Instagram
  if (json.username && json.followerCount !== undefined) {
    data.social.instagram = {
      username: json.username,
      fullName: json.fullName,
      bio: json.bio,
      followerCount: json.followerCount,
      followingCount: json.followingCount,
      postCount: json.postCount,
      posts: (json.posts || []).slice(0, 12)
    };
  }

  // ScrapeCreators Meta Ads
  if (json.ads || json.adCount !== undefined) {
    data.ads.meta = {
      adCount: json.adCount || (json.ads ? json.ads.length : 0),
      ads: (json.ads || []).slice(0, 10),
      pageInfo: json.pageInfo
    };
  }

  // DataForSEO
  if (json.tasks && json.tasks[0]?.result) {
    const seoResult = json.tasks[0].result[0];
    if (seoResult?.metrics) {
      data.seo = {
        organicTraffic: seoResult.metrics.organic?.etv,
        organicKeywords: seoResult.metrics.organic?.count,
        paidTraffic: seoResult.metrics.paid?.etv,
        estimatedValue: seoResult.metrics.organic?.estimated_paid_traffic_cost
      };
    }
  }
}

// Get original URL from webhook
const originalUrl = items[0]?.json?.body?.url || 'Unknown';

return {
  json: {
    original_url: originalUrl,
    website_data: data.website,
    social_data: data.social,
    ad_data: data.ads,
    seo_data: data.seo,
    scraped_at: new Date().toISOString()
  }
};
```

**Step 2: Commit**

```bash
git add workflow.json
git commit -m "feat: update Process Data node to handle all data sources"
```

---

## Task 7: Update Claude Node to Use OpenRouter

**Files:**
- Modify: `workflow.json` (update `claude-analysis` node)

**Step 1: Update the HTTP Request URL and headers**

Find the `claude-analysis` node and update:

```json
{
  "parameters": {
    "url": "https://openrouter.ai/api/v1/chat/completions",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        {
          "name": "Authorization",
          "value": "=Bearer {{ $credentials.openRouterApi.apiKey }}"
        },
        {
          "name": "Content-Type",
          "value": "application/json"
        },
        {
          "name": "HTTP-Referer",
          "value": "https://github.com/themattberman/n8n-gamma"
        }
      ]
    },
    "sendBody": true,
    "specifyBody": "json",
    "jsonBody": "={\n  \"model\": \"anthropic/claude-sonnet-4\",\n  \"max_tokens\": 8000,\n  \"messages\": [\n    {\n      \"role\": \"system\",\n      \"content\": \"You are an elite competitive intelligence analyst. Your analysis is direct, actionable, and data-driven. You focus on 'stealable' tactics and specific recommendations. Always output valid JSON.\"\n    },\n    {\n      \"role\": \"user\",\n      \"content\": \"Analyze this competitor and generate a comprehensive playbook breakdown.\\n\\n## COMPANY URL\\n{{ $json.original_url }}\\n\\n## HOMEPAGE CONTENT\\n{{ $json.website_data.homepage }}\\n\\n## ABOUT PAGE\\n{{ $json.website_data.about }}\\n\\n## PRICING PAGE\\n{{ $json.website_data.pricing }}\\n\\n## BLOG CONTENT\\n{{ $json.website_data.blog }}\\n\\n## LINKEDIN DATA\\n{{ JSON.stringify($json.social_data.linkedin) }}\\n\\n## INSTAGRAM DATA\\n{{ JSON.stringify($json.social_data.instagram) }}\\n\\n## META ADS DATA\\n{{ JSON.stringify($json.ad_data.meta) }}\\n\\n## SEO DATA\\n{{ JSON.stringify($json.seo_data) }}\\n\\n---\\n\\nGenerate your analysis as a JSON object with this exact structure:\\n\\n{\\n  \\\"company_name\\\": \\\"extracted company name\\\",\\n  \\\"company_url\\\": \\\"the URL analyzed\\\",\\n  \\\"analysis_date\\\": \\\"today's date\\\",\\n  \\\"executive_summary\\\": {\\n    \\\"one_liner\\\": \\\"One sentence summary\\\",\\n    \\\"threat_level\\\": \\\"LOW | MEDIUM | HIGH | CRITICAL\\\",\\n    \\\"key_insights\\\": [\\\"insight 1\\\", \\\"insight 2\\\", \\\"insight 3\\\", \\\"insight 4\\\", \\\"insight 5\\\"]\\n  },\\n  \\\"positioning\\\": {\\n    \\\"value_proposition\\\": \\\"Their core value prop\\\",\\n    \\\"target_audience\\\": \\\"Who they sell to\\\",\\n    \\\"key_differentiators\\\": [\\\"diff 1\\\", \\\"diff 2\\\", \\\"diff 3\\\"],\\n    \\\"market_position\\\": \\\"Leader | Challenger | Niche | Emerging\\\"\\n  },\\n  \\\"messaging\\\": {\\n    \\\"brand_voice\\\": \\\"Their tone and style\\\",\\n    \\\"primary_hooks\\\": [{\\\"hook\\\": \\\"exact text\\\", \\\"type\\\": \\\"Pain | Gain | Fear | Curiosity\\\", \\\"effectiveness\\\": \\\"why it works\\\"}],\\n    \\\"key_phrases\\\": [\\\"phrase 1\\\", \\\"phrase 2\\\"],\\n    \\\"proof_elements\\\": [\\\"proof 1\\\", \\\"proof 2\\\"]\\n  },\\n  \\\"content_strategy\\\": {\\n    \\\"content_pillars\\\": [\\\"topic 1\\\", \\\"topic 2\\\"],\\n    \\\"formats_used\\\": [\\\"blog\\\", \\\"video\\\"],\\n    \\\"publishing_frequency\\\": \\\"how often\\\",\\n    \\\"top_performing_content\\\": [{\\\"title\\\": \\\"content title\\\", \\\"format\\\": \\\"type\\\", \\\"why_it_works\\\": \\\"analysis\\\"}],\\n    \\\"distribution_channels\\\": [\\\"channel 1\\\", \\\"channel 2\\\"],\\n    \\\"social_metrics\\\": {\\n      \\\"linkedin_followers\\\": \\\"count or N/A\\\",\\n      \\\"instagram_followers\\\": \\\"count or N/A\\\",\\n      \\\"posting_frequency\\\": \\\"how often\\\",\\n      \\\"engagement_rate\\\": \\\"estimate or N/A\\\"\\n    }\\n  },\\n  \\\"ad_psychology\\\": {\\n    \\\"ad_volume\\\": \\\"number of active ads or N/A\\\",\\n    \\\"primary_ad_types\\\": [\\\"type 1\\\"],\\n    \\\"hook_patterns\\\": [{\\\"pattern\\\": \\\"pattern name\\\", \\\"example\\\": \\\"text\\\", \\\"psychological_trigger\\\": \\\"trigger\\\"}],\\n    \\\"visual_style\\\": \\\"description\\\",\\n    \\\"cta_patterns\\\": [\\\"cta 1\\\"]\\n  },\\n  \\\"seo_intelligence\\\": {\\n    \\\"estimated_traffic\\\": \\\"monthly organic traffic or N/A\\\",\\n    \\\"organic_keywords\\\": \\\"number of ranking keywords or N/A\\\",\\n    \\\"estimated_traffic_value\\\": \\\"dollar value or N/A\\\",\\n    \\\"top_opportunities\\\": [\\\"keyword gap 1\\\", \\\"keyword gap 2\\\"]\\n  },\\n  \\\"gaps_and_opportunities\\\": {\\n    \\\"content_gaps\\\": [{\\\"gap\\\": \\\"what's missing\\\", \\\"opportunity\\\": \\\"how to exploit\\\"}],\\n    \\\"messaging_gaps\\\": [{\\\"gap\\\": \\\"angle missing\\\", \\\"opportunity\\\": \\\"how to exploit\\\"}],\\n    \\\"audience_gaps\\\": [{\\\"gap\\\": \\\"segment ignored\\\", \\\"opportunity\\\": \\\"how to capture\\\"}]\\n  },\\n  \\\"how_to_beat_them\\\": {\\n    \\\"quick_wins\\\": [{\\\"tactic\\\": \\\"action\\\", \\\"why\\\": \\\"reason\\\", \\\"effort\\\": \\\"LOW | MEDIUM | HIGH\\\"}],\\n    \\\"positioning_plays\\\": [{\\\"play\\\": \\\"strategy\\\", \\\"execution\\\": \\\"how to do it\\\"}],\\n    \\\"content_to_steal\\\": [{\\\"what\\\": \\\"content idea\\\", \\\"your_angle\\\": \\\"improvement\\\"}],\\n    \\\"ad_ideas\\\": [{\\\"concept\\\": \\\"ad concept\\\", \\\"hook\\\": \\\"suggested hook\\\"}]\\n  }\\n}\\n\\nReturn ONLY the JSON, no other text.\"\n    }\n  ]\n}",
    "options": {
      "timeout": 120000
    }
  }
}
```

**Step 2: Update Parse Analysis node for OpenRouter response format**

Find the `parse-analysis` node and update its `jsCode` to handle OpenRouter's response:

```javascript
// Parse Claude's response from OpenRouter
const response = $input.first().json;

// OpenRouter returns in OpenAI format: choices[0].message.content
const content = response.choices?.[0]?.message?.content || response.content?.[0]?.text;

if (!content) {
  throw new Error('No content in response: ' + JSON.stringify(response).substring(0, 500));
}

let analysis;
try {
  analysis = JSON.parse(content);
} catch (e) {
  const jsonMatch = content.match(/```json\n?([\s\S]*?)\n?```/) || content.match(/\{[\s\S]*\}/);
  if (jsonMatch) {
    const jsonStr = jsonMatch[1] || jsonMatch[0];
    analysis = JSON.parse(jsonStr);
  } else {
    throw new Error('Could not parse response as JSON: ' + content.substring(0, 500));
  }
}

// Validate required fields
const required = ['company_name', 'executive_summary', 'positioning', 'messaging', 'gaps_and_opportunities', 'how_to_beat_them'];
for (const field of required) {
  if (!analysis[field]) {
    throw new Error(`Missing required field: ${field}`);
  }
}

return { json: analysis };
```

**Step 3: Commit**

```bash
git add workflow.json
git commit -m "feat: switch Claude to OpenRouter API"
```

---

## Task 8: Update Format Gamma Prompt for 10 Slides

**Files:**
- Modify: `workflow.json` (update `format-gamma-prompt` node)

**Step 1: Replace the jsCode with updated 10-slide format**

Find the `format-gamma-prompt` node and replace its `jsCode`:

```javascript
// Format the analysis into a Gamma-friendly prompt (10 slides)
const a = $input.first().json;

const formatList = (arr) => {
  if (!arr || !Array.isArray(arr)) return '- N/A';
  return arr.map(item => {
    if (typeof item === 'string') return `- ${item}`;
    if (typeof item === 'object') {
      return Object.entries(item).map(([k, v]) => `  - ${k}: ${v}`).join('\n');
    }
    return `- ${item}`;
  }).join('\n');
};

const gammaPrompt = `
Create a professional competitive intelligence deck analyzing ${a.company_name}.

## SLIDE 1: COVER
Title: "${a.company_name} Playbook"
Subtitle: "Competitive Intelligence Report"
Date: ${a.analysis_date || new Date().toISOString().split('T')[0]}
Style: Bold, professional, dark theme

## SLIDE 2: EXECUTIVE SUMMARY
Title: "Key Intelligence"

Threat Level: ${a.executive_summary?.threat_level || 'MEDIUM'}

${a.executive_summary?.one_liner || ''}

Key Insights:
${formatList(a.executive_summary?.key_insights)}

## SLIDE 3: POSITIONING ANALYSIS
Title: "How They Position Themselves"

Value Proposition: ${a.positioning?.value_proposition || 'N/A'}

Target Audience: ${a.positioning?.target_audience || 'N/A'}

Market Position: ${a.positioning?.market_position || 'N/A'}

Key Differentiators:
${formatList(a.positioning?.key_differentiators)}

## SLIDE 4: MESSAGING BREAKDOWN
Title: "Their Messaging Playbook"

Brand Voice: ${a.messaging?.brand_voice || 'N/A'}

Primary Hooks:
${a.messaging?.primary_hooks?.map(h => `- "${h.hook}" (${h.type}) - ${h.effectiveness}`).join('\n') || '- N/A'}

Key Phrases: ${a.messaging?.key_phrases?.join(', ') || 'N/A'}

Proof Elements:
${formatList(a.messaging?.proof_elements)}

## SLIDE 5: CONTENT & SOCIAL STRATEGY
Title: "Content Machine Analysis"

Content Pillars: ${a.content_strategy?.content_pillars?.join(', ') || 'N/A'}

Formats Used: ${a.content_strategy?.formats_used?.join(', ') || 'N/A'}

Publishing Frequency: ${a.content_strategy?.publishing_frequency || 'N/A'}

Social Metrics:
- LinkedIn Followers: ${a.content_strategy?.social_metrics?.linkedin_followers || 'N/A'}
- Instagram Followers: ${a.content_strategy?.social_metrics?.instagram_followers || 'N/A'}
- Posting Frequency: ${a.content_strategy?.social_metrics?.posting_frequency || 'N/A'}

Top Content:
${a.content_strategy?.top_performing_content?.map(c => `- ${c.title} (${c.format})`).join('\n') || '- N/A'}

Distribution: ${a.content_strategy?.distribution_channels?.join(', ') || 'N/A'}

## SLIDE 6: AD PSYCHOLOGY
Title: "Advertising Tactics"

Ad Activity: ${a.ad_psychology?.ad_volume || 'N/A'}

Ad Types: ${a.ad_psychology?.primary_ad_types?.join(', ') || 'N/A'}

Visual Style: ${a.ad_psychology?.visual_style || 'N/A'}

Hook Patterns:
${a.ad_psychology?.hook_patterns?.map(p => `- ${p.pattern}: "${p.example}"`).join('\n') || '- N/A'}

CTA Patterns: ${a.ad_psychology?.cta_patterns?.join(', ') || 'N/A'}

## SLIDE 7: SEO INTELLIGENCE
Title: "Search Performance"

Estimated Monthly Traffic: ${a.seo_intelligence?.estimated_traffic || 'N/A'}

Organic Keywords Ranking: ${a.seo_intelligence?.organic_keywords || 'N/A'}

Estimated Traffic Value: ${a.seo_intelligence?.estimated_traffic_value || 'N/A'}

Top Keyword Opportunities:
${formatList(a.seo_intelligence?.top_opportunities)}

## SLIDE 8: GAPS & OPPORTUNITIES
Title: "Where They're Vulnerable"

Content Gaps:
${a.gaps_and_opportunities?.content_gaps?.map(g => `- GAP: ${g.gap}\n  OPPORTUNITY: ${g.opportunity}`).join('\n') || '- N/A'}

Messaging Gaps:
${a.gaps_and_opportunities?.messaging_gaps?.map(g => `- GAP: ${g.gap}\n  OPPORTUNITY: ${g.opportunity}`).join('\n') || '- N/A'}

Audience Gaps:
${a.gaps_and_opportunities?.audience_gaps?.map(g => `- GAP: ${g.gap}\n  OPPORTUNITY: ${g.opportunity}`).join('\n') || '- N/A'}

## SLIDE 9: HOW TO BEAT THEM
Title: "Your Attack Plan"

Quick Wins (This Week):
${a.how_to_beat_them?.quick_wins?.map(w => `- ${w.tactic} [${w.effort}] - ${w.why}`).join('\n') || '- N/A'}

Positioning Plays:
${a.how_to_beat_them?.positioning_plays?.map(p => `- ${p.play}: ${p.execution}`).join('\n') || '- N/A'}

Content to Steal:
${a.how_to_beat_them?.content_to_steal?.map(c => `- ${c.what} → Your angle: ${c.your_angle}`).join('\n') || '- N/A'}

Ad Ideas:
${a.how_to_beat_them?.ad_ideas?.map(ad => `- ${ad.concept}: "${ad.hook}"`).join('\n') || '- N/A'}

## SLIDE 10: NEXT STEPS
Title: "Ready to Win?"

This analysis was generated by AI in under 60 seconds.

Want us to run this on YOUR competitors?

Book a strategy call →
`.trim();

return {
  json: {
    prompt: gammaPrompt,
    company_name: a.company_name,
    company_url: a.company_url,
    analysis: a
  }
};
```

**Step 2: Commit**

```bash
git add workflow.json
git commit -m "feat: update Gamma prompt for 10-slide deck with SEO"
```

---

## Task 9: Update Webhook Input Schema

**Files:**
- Modify: `workflow.json` (document expected input format)

**Step 1: Document new webhook input format**

The webhook now expects additional optional fields. Update the README with the new input format:

```json
{
  "url": "https://competitor-website.com",
  "linkedin_handle": "shopify",
  "instagram_handle": "shopify",
  "facebook_handle": "shopify"
}
```

Note: The `url` is required. Social handles are optional - if not provided, those data sources will return empty (graceful degradation via `continueOnFail`).

**Step 2: Commit**

```bash
git add workflow.json
git commit -m "docs: document expanded webhook input schema"
```

---

## Task 10: Update README.md

**Files:**
- Modify: `README.md`

**Step 1: Update API requirements table**

Replace the existing requirements table:

```markdown
### APIs Needed

| Service | Purpose | Get API Key |
|---------|---------|-------------|
| **Firecrawl** | Website scraping | [firecrawl.dev](https://firecrawl.dev) |
| **ScrapeCreators** | Social media & ads data | [scrapecreators.com](https://scrapecreators.com) |
| **DataForSEO** | SEO intelligence | [dataforseo.com](https://dataforseo.com) |
| **OpenRouter** | AI analysis (Claude) | [openrouter.ai](https://openrouter.ai) |
| **Gamma** | Deck generation | [gamma.app](https://gamma.app) (Pro required) |
```

**Step 2: Add new credentials section**

Add after existing credentials:

```markdown
#### ScrapeCreators API
- Type: Header Auth
- Name: `scrapeCreatorsApi`
- Header Name: `x-api-key`
- Header Value: `YOUR_SCRAPECREATORS_API_KEY`

#### DataForSEO API
- Type: Header Auth
- Name: `dataForSeoApi`
- Header Name: `Authorization`
- Header Value: `Basic YOUR_BASE64_CREDENTIALS`

#### OpenRouter API
- Type: Header Auth
- Name: `openRouterApi`
- Header Name: `Authorization`
- Header Value: `Bearer YOUR_OPENROUTER_API_KEY`
```

**Step 3: Update usage example**

```markdown
### Via Webhook (API)

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

Note: Only `url` is required. Social handles are optional for enriched analysis.
```

**Step 4: Update architecture diagram**

```markdown
## Workflow Architecture

```
[Webhook] → [Firecrawl x4 + ScrapeCreators x3 + DataForSEO] → [Merge] → [Claude via OpenRouter] → [Gamma] → [Response]
     │                    │                                      │              │                   │
     │                    ├─ Homepage                            │              │                   │
     │                    ├─ About                               │              │                   │
     │                    ├─ Pricing                             │              │                   │
     │                    ├─ Blog                                │              │                   │
     │                    ├─ LinkedIn Company ←NEW               │              │                   │
     │                    ├─ Instagram Profile ←NEW              │              │                   │
     │                    ├─ Meta Ad Library ←NEW                │              │                   │
     │                    └─ SEO Overview ←NEW                   │              │                   │
     │                                                      Combines        Analyzes          Generates
     └─ Receives URL + handles                              all data        via OpenRouter    10-slide deck
```
```

**Step 5: Commit**

```bash
git add README.md
git commit -m "docs: update README with new APIs and credentials"
```

---

## Task 11: Update CLAUDE.md

**Files:**
- Modify: `CLAUDE.md`

**Step 1: Update workflow architecture section**

Update the workflow architecture to reflect 8 parallel calls and OpenRouter:

```markdown
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
       [Gamma - Generate Deck] → [Wait] → [Poll Status] → [Response]
```

**21 nodes total:** 1 webhook trigger, 8 data source HTTP requests, 1 merge, 3 code nodes, 2 Gamma HTTP requests, 2 wait nodes, 1 if node, 1 webhook response
```

**Step 2: Update API dependencies table**

```markdown
## API Dependencies

| API | Purpose | Auth | Rate Limits |
|-----|---------|------|-------------|
| Firecrawl | Website scraping | Bearer token | Check dashboard |
| ScrapeCreators | LinkedIn, Instagram, Meta Ads | x-api-key header | 100 free calls |
| DataForSEO | SEO metrics | Basic auth | Pay per call |
| OpenRouter | Claude AI analysis | Bearer token | Per-model limits |
| Gamma | Deck generation (Pro required) | X-API-KEY header | Async with polling |
```

**Step 3: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: update CLAUDE.md with expanded architecture"
```

---

## Task 12: Update prompts/deck-structure.md

**Files:**
- Modify: `prompts/deck-structure.md`

**Step 1: Update slide count and add SEO slide**

Update the document to reflect 10 slides and add the SEO Intelligence slide specification between Ad Psychology and Gaps & Opportunities.

Add after SLIDE 6 (Ad Psychology):

```markdown
SLIDE 7: SEO INTELLIGENCE
Title: "Search Performance"
Content sections:
ESTIMATED TRAFFIC: {{estimated_traffic}}
ORGANIC KEYWORDS: {{organic_keywords}}
TRAFFIC VALUE: {{estimated_traffic_value}}
TOP OPPORTUNITIES:
{{#each top_opportunities}}
- {{this}}
{{/each}}
Style: Data-focused with key metrics prominent
```

Update subsequent slide numbers (Gaps → 8, Beat → 9, CTA → 10).

**Step 2: Commit**

```bash
git add prompts/deck-structure.md
git commit -m "docs: add SEO Intelligence slide to deck structure"
```

---

## Task 13: Final Validation

**Step 1: Validate workflow structure via n8n MCP**

```
mcp__n8n-mcp__validate_workflow with the full workflow.json
```

Expected: No errors, all nodes connected properly

**Step 2: Test with sample data**

Trigger webhook with:
```json
{
  "url": "https://example.com",
  "linkedin_handle": "example",
  "instagram_handle": "example",
  "facebook_handle": "example"
}
```

Expected: 10-slide deck generated with social/ads/SEO data included

**Step 3: Final commit**

```bash
git add -A
git commit -m "feat: complete expanded data sources implementation"
```

---

## Summary

| Task | Description | Files |
|------|-------------|-------|
| 1 | LinkedIn Company node | workflow.json |
| 2 | Instagram Profile node | workflow.json |
| 3 | Meta Ad Library node | workflow.json |
| 4 | DataForSEO node | workflow.json |
| 5 | Update Merge connections | workflow.json |
| 6 | Update Process Data code | workflow.json |
| 7 | Switch to OpenRouter | workflow.json |
| 8 | Update Gamma prompt (10 slides) | workflow.json |
| 9 | Document webhook input | workflow.json |
| 10 | Update README | README.md |
| 11 | Update CLAUDE.md | CLAUDE.md |
| 12 | Update deck-structure.md | prompts/deck-structure.md |
| 13 | Final validation | - |

**New credentials required:**
- `scrapeCreatorsApi` (Header Auth, x-api-key)
- `dataForSeoApi` (Header Auth, Basic auth)
- `openRouterApi` (Header Auth, Bearer token)
