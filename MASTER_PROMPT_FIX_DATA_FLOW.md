# Master Prompt: Fix n8n Workflow Data Flow Issues

## Project Context

This is an n8n workflow that generates competitive intelligence presentation decks. The workflow:
1. Takes a competitor URL as input
2. Scrapes website content (homepage, about, pricing, blog) via Firecrawl
3. Discovers social handles via ScrapeCreators company search
4. Scrapes LinkedIn, Instagram, Meta Ads data via ScrapeCreators
5. Gets SEO metrics via DataForSEO
6. Analyzes all data with Claude via OpenRouter
7. Generates a Gamma presentation deck

**Repository:** `/Users/matthewberman/Documents/Github/n8n-gamma`
**Main file:** `workflow.json`
**n8n instance:** `https://n8n.emeralddigital.synology.me`
**Workflow ID:** `HtkJ4jhyXoCPiqjF`

## What Was Already Fixed

In the previous session, we fixed:
1. **OpenRouter API call** - Changed from GET to POST method, confirmed model is `anthropic/claude-sonnet-4.5`
2. **Merge node configuration** - Added `"numberInputs": 8` and assigned each scraper a unique input index (0-7)
3. **Gamma API call** - Changed to POST method, fixed gammaId format to `g_8s1srwjr2mn34s9`
4. **Parallel execution** - All 8 data sources now run in parallel and merge correctly

## Current Issues (Your Task)

After running execution 5389 with ClickUp as the target, the generated report (https://gamma.app/docs/LEAKED-INTEL-x6h4s3nz2k48j6v) shows:
- **Instagram Followers: N/A** (Instagram data not scraped)
- **Active Creatives: N/A** with note "Meta ads data not provided"
- **SEO Dominance: all N/A** with note "SEO data not provided"

Yet all nodes ran successfully (green checkmarks). The data is getting lost somewhere in the pipeline.

---

## Issue 1: Social Handle Discovery Fails

### Root Cause
The `ScrapeCreators - Company Search` node uses Facebook Ad Library's company search:
```
https://api.scrapecreators.com/v1/facebook/adLibrary/search/companies?query={{ encodeURIComponent($json.body.company_name) }}
```

For ClickUp, this returned empty results:
- `instagram_handle: ""`
- `facebook_page_id: ""`
- `facebook_page_alias: ""`

When these are empty, the downstream nodes scrape nothing:
- `ScrapeCreators - Instagram Profile` calls: `?handle=` (empty)
- `ScrapeCreators - Meta Ad Library` calls: `?handle=` (empty)

### Current Code: Extract Social Handles (lines 519-530)
This node only extracts LinkedIn from homepage links. It does NOT extract Instagram or Facebook.

```javascript
// Extract LinkedIn company handle from website links
let linkedin_handle = '';
const linkedinMatch = content.match(/linkedin\.com\/company\/([a-zA-Z0-9_-]+)/i);
if (linkedinMatch) {
  linkedin_handle = linkedinMatch[1];
}
```

### Fix Required
1. **Add Instagram extraction from homepage** - Most company websites link to their Instagram in header/footer
2. **Add Facebook extraction from homepage** - Same pattern for Facebook page links
3. **Use homepage-extracted handles as fallback** when company search returns empty

Add to `Extract Social Handles` node:
```javascript
// Extract Instagram handle from website links
let instagram_handle = '';
const instagramMatch = content.match(/instagram\.com\/([a-zA-Z0-9._]+)/i);
if (instagramMatch) {
  instagram_handle = instagramMatch[1];
}

// Extract Facebook page from website links
let facebook_page_alias = '';
const facebookMatch = content.match(/facebook\.com\/([a-zA-Z0-9._-]+)/i);
if (facebookMatch) {
  facebook_page_alias = facebookMatch[1];
}
```

Then update `Process Search Results` to use homepage-extracted handles as fallback:
```javascript
const instagram_handle = bestMatch.ig_username || prevData.body.instagram_handle || '';
const facebook_page_alias = bestMatch.page_alias || prevData.body.facebook_page_alias || '';
```

---

## Issue 2: DataForSEO Data Lost in Processing

### Root Cause
The `Process Website Data` node (lines 274-286) expects this structure:
```javascript
if (json.tasks && json.tasks[0]?.result) {
  const seoResult = json.tasks[0].result[0];
  if (seoResult?.metrics) {  // <-- This might be wrong
    data.seo = {
      organicTraffic: seoResult.metrics.organic?.etv,
      organicKeywords: seoResult.metrics.organic?.count,
      ...
    };
  }
}
```

But the actual DataForSEO response structure might be different. The node executed successfully and returned data, but the parsing failed silently because `seoResult?.metrics` doesn't exist.

### Fix Required
1. **Inspect actual DataForSEO response** - Check execution 5389's DataForSEO node output to see the real structure
2. **Update the parsing logic** to match the actual API response format

To debug, you should:
1. Go to https://n8n.emeralddigital.synology.me/workflow/HtkJ4jhyXoCPiqjF/executions/5389
2. Click on `DataForSEO - Domain Overview` node
3. Examine the OUTPUT JSON structure
4. Update `Process Website Data` to correctly parse that structure

The DataForSEO API for `domain_rank_overview/live` returns data like:
```json
{
  "tasks": [{
    "result": [{
      "target": "clickup.com",
      "metrics": {
        "organic": {
          "etv": 12345,
          "count": 5000,
          "estimated_paid_traffic_cost": 50000
        },
        "paid": {
          "etv": 1000
        }
      }
    }]
  }]
}
```

If the structure differs, the parsing code needs adjustment.

---

## Issue 3: Empty Handle Validation

### Root Cause
The scraper nodes don't validate handles before making API calls. They send requests with empty parameters, wasting API calls and returning useless data.

### Fix Required
Add validation before API calls. Options:

**Option A: Add IF nodes before each social scraper**
Only call Instagram scraper if `instagram_handle` is not empty.

**Option B: Update the HTTP Request node expressions**
Change the URL expressions to include a check:
```
={{ $json.body.instagram_handle ? 'https://api.scrapecreators.com/v1/instagram/profile?handle=' + $json.body.instagram_handle : '' }}
```
(Though this might cause issues with empty URLs)

**Option C: Add validation in Process Website Data**
Check for valid response data before trying to parse:
```javascript
// Only parse Instagram if we have valid data
if (json.username && json.followerCount !== undefined && json.username !== '') {
  // ... parse Instagram data
}
```

---

## Files to Modify

| File | Changes Needed |
|------|----------------|
| `workflow.json` | Update Extract Social Handles, Process Search Results, and Process Website Data nodes |

## Test Plan

1. After making fixes, run the workflow with ClickUp URL again
2. Verify in execution that:
   - `Extract Social Handles` extracts Instagram/Facebook from homepage
   - `Process Search Results` shows populated handles (either from search or homepage fallback)
   - `DataForSEO` data appears in `Process Website Data` output
3. Check final Gamma report shows populated values for Instagram, Meta Ads, and SEO sections

## n8n MCP Commands

Use these MCP commands to interact with the workflow:
```
mcp__n8n-mcp__n8n_health_check
mcp__n8n-mcp__n8n_get_workflow with id: "HtkJ4jhyXoCPiqjF"
mcp__n8n-mcp__n8n_update_full_workflow with id and full workflow JSON
```

Or edit `workflow.json` directly and reimport into n8n.

---

## Priority Order

1. **High: Fix DataForSEO parsing** - Data exists but isn't being parsed. Inspect actual response structure.
2. **High: Add homepage social extraction** - Simple regex addition to Extract Social Handles
3. **Medium: Add fallback logic** - Use homepage handles when company search returns empty
4. **Low: Add empty handle validation** - Prevent wasted API calls
