# Competitive Intelligence Analysis Prompt

Use this prompt in the Claude/Anthropic HTTP Request node in n8n.

## System Prompt

```
You are an elite competitive intelligence analyst with expertise in:
- Brand positioning and messaging strategy
- Content marketing analysis
- Advertising psychology (Kallaway framework)
- Market gap identification
- Tactical competitive recommendations

Your analysis style is:
- Direct and actionable (no fluff)
- Data-driven with specific examples
- Focused on "stealable" tactics
- Written for operators who want to WIN

You will receive scraped data about a company and produce a structured competitive intelligence report.
```

## User Prompt Template

```
Analyze this competitor and generate a comprehensive playbook breakdown that I can use to beat them.

## COMPANY URL
{{$json.url}}

## WEBSITE DATA (from Firecrawl)
{{$json.website_data}}

## SOCIAL PRESENCE (LinkedIn/Twitter)
{{$json.social_data}}

## AD CREATIVE DATA (from Meta Ad Library)
{{$json.ad_data}}

## SEO DATA (from DataForSEO - if available)
{{$json.seo_data}}

---

Generate your analysis in the following JSON structure. Be specific, cite examples from the data, and focus on actionable insights:

{
  "company_name": "extracted company name",
  "company_url": "the URL analyzed",
  "analysis_date": "today's date",

  "executive_summary": {
    "one_liner": "One sentence summary of who they are and what they do",
    "threat_level": "LOW | MEDIUM | HIGH | CRITICAL",
    "key_insights": [
      "Insight 1 - most important thing to know",
      "Insight 2",
      "Insight 3",
      "Insight 4",
      "Insight 5"
    ]
  },

  "positioning": {
    "value_proposition": "Their core value prop in one sentence",
    "target_audience": "Who they're selling to",
    "key_differentiators": [
      "Differentiator 1",
      "Differentiator 2",
      "Differentiator 3"
    ],
    "positioning_statement": "For [audience] who [need], [Company] provides [solution] that [key benefit] unlike [alternatives]",
    "market_position": "Leader | Challenger | Niche | Emerging"
  },

  "messaging": {
    "brand_voice": "Description of their tone and style",
    "primary_hooks": [
      {
        "hook": "Exact hook text or pattern",
        "type": "Pain | Gain | Fear | Curiosity | Social Proof",
        "effectiveness": "Why it works"
      }
    ],
    "key_phrases": ["Phrases they repeat", "Taglines", "Power words"],
    "objection_handling": [
      {
        "objection": "Common objection",
        "how_they_handle": "Their approach"
      }
    ],
    "proof_elements": ["Case studies", "Testimonials", "Metrics they cite"]
  },

  "content_strategy": {
    "content_pillars": ["Topic 1", "Topic 2", "Topic 3"],
    "formats_used": ["Blog", "Video", "Podcast", "Newsletter", etc],
    "publishing_frequency": "How often they publish",
    "top_performing_content": [
      {
        "title": "Content title",
        "format": "Format type",
        "why_it_works": "Analysis"
      }
    ],
    "distribution_channels": ["Where they share content"],
    "engagement_tactics": ["How they drive engagement"]
  },

  "ad_psychology": {
    "ad_volume": "How many active ads / ad activity level",
    "primary_ad_types": ["Static", "Video", "Carousel", etc],
    "hook_patterns": [
      {
        "pattern": "Hook pattern name",
        "example": "Exact text example",
        "psychological_trigger": "What it triggers"
      }
    ],
    "visual_style": "Description of their ad visual approach",
    "cta_patterns": ["CTA examples"],
    "landing_page_strategy": "Where ads lead and why",
    "estimated_spend": "If discernible from ad volume"
  },

  "gaps_and_opportunities": {
    "content_gaps": [
      {
        "gap": "Topic/angle they're NOT covering",
        "opportunity": "How you could own this"
      }
    ],
    "messaging_gaps": [
      {
        "gap": "Messaging angle they're missing",
        "opportunity": "How to exploit"
      }
    ],
    "audience_gaps": [
      {
        "gap": "Audience segment they're ignoring",
        "opportunity": "How to capture them"
      }
    ],
    "format_gaps": [
      {
        "gap": "Content format they don't use",
        "opportunity": "Why you should"
      }
    ]
  },

  "how_to_beat_them": {
    "quick_wins": [
      {
        "tactic": "Specific action to take",
        "why": "Why it will work",
        "effort": "LOW | MEDIUM | HIGH"
      }
    ],
    "positioning_plays": [
      {
        "play": "Strategic positioning move",
        "execution": "How to execute"
      }
    ],
    "content_to_steal": [
      {
        "what": "Content idea to adapt",
        "your_angle": "How to make it better"
      }
    ],
    "ad_ideas": [
      {
        "concept": "Ad concept based on their gaps",
        "hook": "Suggested hook",
        "angle": "Positioning angle"
      }
    ]
  },

  "seo_intelligence": {
    "estimated_traffic": "If available from DataForSEO",
    "top_keywords": ["Keywords they rank for"],
    "keyword_gaps": ["Keywords you could target"],
    "backlink_profile": "Summary of their link building",
    "domain_authority": "If available"
  }
}

IMPORTANT:
- Be SPECIFIC - cite actual text, URLs, and examples from the scraped data
- Be ACTIONABLE - every insight should suggest what to do about it
- Be HONEST - if data is limited, say so rather than making things up
- Be CONCISE - no fluff, just signal
- Return ONLY the JSON, no other text
```

## Response Format

The prompt requests JSON output that maps directly to Gamma deck slides:

| JSON Section | Gamma Slide |
|--------------|-------------|
| executive_summary | Slide 2: Executive Summary |
| positioning | Slide 3: Positioning Analysis |
| messaging | Slide 4: Messaging Breakdown |
| content_strategy | Slide 5: Content Strategy |
| ad_psychology | Slide 6: Ad Psychology |
| gaps_and_opportunities | Slide 7: Gaps & Opportunities |
| how_to_beat_them | Slide 8: How to Beat Them |
| seo_intelligence | (Include in relevant slides) |

## n8n Configuration

In your HTTP Request node:
- Method: POST
- URL: `https://api.anthropic.com/v1/messages`
- Headers:
  - `x-api-key`: {{$credentials.anthropicApi.apiKey}}
  - `anthropic-version`: 2023-06-01
  - `content-type`: application/json
- Body:
```json
{
  "model": "claude-sonnet-4-20250514",
  "max_tokens": 8000,
  "system": "{{system_prompt}}",
  "messages": [
    {
      "role": "user",
      "content": "{{user_prompt_with_data}}"
    }
  ]
}
```

## Tips for Best Results

1. **More data = better analysis** - Include as much scraped data as possible
2. **Clean the data first** - Remove HTML, scripts, and noise before sending to Claude
3. **Handle missing data gracefully** - If ad data isn't available, the prompt still works
4. **Parse the JSON response** - Use a Code node to extract the JSON from Claude's response
5. **Validate structure** - Ensure all required fields exist before sending to Gamma
