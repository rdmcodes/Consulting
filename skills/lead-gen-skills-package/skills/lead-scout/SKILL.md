---
name: lead-scout
description: Find and qualify business leads using Apify scraping. Use when the user wants to find businesses in a niche + area, prospect for leads, discover businesses that need better websites, or says "lead-scout". Triggers on prospecting, lead finding, business discovery, niche scouting, or market research for potential clients.
---

# Lead Scout

You are running the Lead Scout pipeline — a workflow that discovers businesses in a niche + area via Apify MCP, audits their web presence, ranks them by how much they need digital services, and auto-builds sample sites for top leads via biz-site-scout.

Strategy: lead with a free website to get in the door, then discover pain points in conversation and offer solutions.

## Setup & Prerequisites

**This skill requires Apify MCP (non-optional) and benefits from several other tools. See `biz-site-scout/SKILL.md` Setup section for full MCP server install commands.**

### Required
- **Apify MCP** — scraping Google Maps, Yelp, Facebook, and full site crawls. Without this, lead-scout cannot run at all.
  ```bash
  claude mcp add --transport http apify https://mcp.apify.com --header "Authorization: Bearer YOUR_APIFY_API_KEY" --scope user
  ```
  Get key: https://console.apify.com/account/integrations

### Recommended (enhances quality)
- **Playwright CLI** — screenshots for visual auditing + QA verification
  ```bash
  npx playwright install
  ```
- **biz-site-scout skill** — auto-builds landing pages for top leads. Must be installed at `~/.claude/skills/biz-site-scout/`
- **21st.dev Magic MCP** — design inspiration for generated sites (see biz-site-scout setup)
- **Nano Banana MCP** — motion effects on static business photos (see biz-site-scout setup)
- **sharp-cli** — image resizing before nanobanana: `npm install -g sharp-cli`
- **Netlify account** — deploy generated sites to live URLs

### File Structure
```
~/.claude/skills/lead-scout/
├── SKILL.md                          # This file
├── config/
│   └── defaults.json                 # Settings: top_n, auto_build, exclude_chains, etc.
```

Generated output:
```
[working-dir]/lead-scout/runs/[YYYY-MM-DD-niche-city]/
├── results.csv                       # Full ranked lead list
├── summary.md                        # Report with scores + verdicts
├── businesses/[slug]/
│   ├── audit/
│   │   ├── analysis.md               # Score breakdown + pain points
│   │   ├── assets.json               # Extracted logo, photos, colors
│   │   ├── desktop.png               # Audit screenshot
│   │   └── mobile.png
│   └── generated-site/              # (if auto-build triggered)
│       ├── index.html
│       └── assets/motion/
```

---

## Invocation

The user provides a niche and location:
- `/lead-scout "plumbers" "Austin, TX"`
- Or naturally: "find me plumbers in Austin that need a website"

Parse the niche and location from whatever format the user provides. If either is missing, ask.

## Pipeline

```
Scrape (Apify MCP) → Filter → Analyze (Playwright + Vision) → Rank → Report → Auto-Build Top N
```

Run each phase in order. Present the ranked report to the user before auto-building.

---

## Phase 1: Multi-Source Scrape via Apify MCP

Scrape from **three sources** in parallel to maximize coverage. Different business types live on different platforms — Google Maps catches most, Yelp catches service businesses, Facebook catches those running entirely off social media.

### Source 1: Google Maps
**Actor:** `compass/crawler-google-places`
**Input:**
- `searchStringsArray`: ["[niche] in [city]"]
- `maxCrawledPlacesPerSearch`: 100
- `language`: "en"
- `includeWebResults`: false

**Data:** name, address, phone, website, rating, review count, recent review date, hours, category, Google Maps URL, open/closed status

### Source 2: Yelp
**Actor:** `spiders/yelp-search-scraper`
**Input:**
- `search_urls`: ["[niche]"]
- `search_location`: "[city], [state]"
- `search_limit`: 3 (pages to scrape, ~10 results per page)
- `search_sort`: "recommended"

Note: The `search_urls` field accepts search terms directly (not URLs). The `search_limit` is pages, not results.

**Data:** name, phone, website, rating, review count, categories, Yelp URL

### Source 3: Facebook Pages
**Actor:** `apify/facebook-pages-scraper`
**Input:**
- Search Facebook for "[niche] [city]" pages
- Or use Facebook search URL: `https://www.facebook.com/search/pages/?q=[niche]%20[city]`

**Data:** page name, phone, website, likes, followers, last post date, about info

### Merge + Deduplicate

After all three scrapes complete:
1. Normalize business names (lowercase, trim, remove "LLC"/"Inc" etc.)
2. Match duplicates by **phone number** first (most reliable), then by **address** if no phone match
3. When merging duplicates, keep the richest data from each source (e.g., Google rating + Yelp reviews + Facebook followers)
4. Tag each business with which platforms they were found on: `[Google, Yelp, Facebook]`
5. Businesses found on fewer platforms = weaker digital presence = higher priority lead

### Budget Control

To stay within budget, use these limits per source:
- If user specifies a dollar budget, split roughly: 50% Google Maps, 30% Yelp, 20% Facebook
- Default: 100 results per source (well under $1 total for all three)
- The skill reports estimated Apify cost after scraping completes

If the Apify MCP is not available or not connected, stop and tell the user: "Apify MCP is required for lead-scout. Run: `claude mcp add --transport http apify https://mcp.apify.com --header 'Authorization: Bearer YOUR_KEY' --scope user` then reload."

---

## Phase 2: Filter Dead Leads

Remove businesses that are:
- **Permanently closed** (Google Maps status)
- **No reviews at all** and no website (likely not a real/active business)
- **Duplicates** (same phone number or exact same address)

Read `lead-scout/config/defaults.json` for settings:
- `exclude_chains`: if true, remove known franchise/chain businesses
- `inactive_threshold_months`: remove if last review is older than this
- `min_reviews_to_consider_active`: minimum review count to keep

Report how many were scraped vs how many passed filtering.

---

## Phase 3: Analyze Web Presence

Use parallel agents (subagent_type: "general-purpose") for businesses with websites — batch 5 at a time.

### For businesses WITH a website:

**3a. Full Site Crawl (Apify — all pages, not just homepage):**

Since Apify MCP is already available (required for Phase 1), crawl each business's full site:

**Actor:** `apify/website-content-crawler`
```json
{
  "startUrls": [{ "url": "[business-website-url]" }],
  "crawlerType": "playwright:firefox",
  "maxCrawlDepth": 3,
  "maxCrawlPages": 25,
  "excludeUrlGlobs": ["/**/*.pdf", "/**/*.zip"],
  "outputFormats": ["markdown"]
}
```

This returns every page's content as Markdown — services, about, gallery, contact, blog, etc. Way more data than just the homepage.

**Efficiency tip:** Launch crawls for multiple businesses in parallel (up to 5 concurrent). Each crawl takes ~10-30 seconds for a small site.

**3b. Screenshot Capture (Playwright CLI — run in parallel with crawl):**

```bash
npx playwright screenshot --full-page --viewport-size="1440,900" "[URL]" "[output-path]/desktop.png"
npx playwright screenshot --full-page --viewport-size="390,844" "[URL]" "[output-path]/mobile.png"
```

Save screenshots to: `lead-scout/runs/[session-name]/businesses/[slug]/audit/`

Where `[session-name]` = `YYYY-MM-DD-[niche]-[city]` (e.g., `2026-03-05-plumbers-austin`)
Where `[slug]` = business name slugified (lowercase, hyphens, no special chars)

**3c. Analyze All Pages (Claude Vision + Full Content):**

With the full crawl data, analyze the ENTIRE site:

**Content analysis (from crawl):**
- Total page count and site structure
- Services listed across all pages
- Contact form / booking system present?
- Testimonials found on any page?
- Content depth per page (thin < 100 words?)
- Gallery/portfolio pages?
- Blog — present? Last update date?
- Service area listed?

**Visual analysis (from screenshots):**
- Is the design modern or outdated?
- Is it mobile responsive?
- Are there clear CTAs?
- Does it look professional?
- Any broken elements visible?

**Page-by-page summary:**
```
Pages found: X | Homepage: [assessment] | Services: [assessment or "Missing"]
About: [assessment or "Missing"] | Contact: [assessment or "Missing"]
```

**Fallback (no Playwright):** Use crawl content + WebFetch for analysis. Flag visual scoring as less accurate.

### For businesses WITHOUT a website:

- Skip all analysis
- Auto-assign score: 1/10
- Pain points: ["No website — invisible online", "Losing all web traffic to competitors"]
- Verdict: "Build"

### Scoring Rubric

Score each business 1-10:

| Category | Points | Criteria |
|----------|--------|----------|
| Website Quality | 0-3 | 0 = no site, 1 = broken/ancient, 2 = functional but dated, 3 = modern + clean |
| Review Health | 0-3 | Combines: rating (0-1), count (0-1), recency (0-1) |
| Lead Capture | 0-2 | 0 = no forms/CTAs, 1 = basic contact info only, 2 = forms + booking |
| Mobile Experience | 0-2 | 0 = not responsive, 1 = partially, 2 = fully responsive |

**Lower score = needs you more.**

### Pain Point Identification

For each business, identify applicable pain points:
- No website → "No online presence — invisible to customers searching Google"
- No contact form → "Losing leads — visitors can't easily request service"
- No online booking → "Customers choosing competitors with easy scheduling"
- Low/old reviews → "Not leveraging social proof to build trust"
- Outdated design → "Website makes business look unprofessional or abandoned"
- Not mobile-friendly → "Losing 60%+ of visitors who browse on mobile"
- No SSL → "Browser shows 'Not Secure' warning — scares customers away"
- Thin content → "Only X pages — not enough info for customers to choose you"
- No services page → "Customers can't see what you offer without calling"
- Dead blog → "Blog hasn't been updated since [date] — signals neglect"
- No gallery → "No photos of your work — customers want proof before they call"

### Save Analysis

Write `analysis.md` for each business to their audit folder:

```markdown
## [Business Name] — Score: X/10

**Website:** [URL or "None"]
**Rating:** X stars (Y reviews, last review: [date])
**Phone:** [phone]
**Address:** [address]

### Scores
- Website Quality: X/3
- Review Health: X/3
- Lead Capture: X/2
- Mobile Experience: X/2

### Pain Points
- [pain point 1]
- [pain point 2]

### Verdict: [Build / Strong Lead / Maybe / Skip]
```

---

## Phase 4: Rank + Report

### Sort and Categorize

Sort all businesses by score (lowest = highest priority). Assign verdicts:
- **Build** (score 0-3) — No site or terrible site, active business
- **Strong Lead** (score 4-5) — Has site but it's clearly hurting them
- **Maybe** (score 6-7) — Site is mediocre, might benefit
- **Skip** (score 8-10) — Site is decent, not worth pursuing

### Present in Chat

Show the ranked table:

```
## Lead Scout Results: [Niche] in [City]
### Scraped: X → Filtered: Y → Analyzed: Z

| # | Business | Score | Website | Rating | Reviews | Last Review | Top Pain Point | Verdict |
|---|----------|-------|---------|--------|---------|-------------|----------------|---------|
| 1 | Joe's Plumbing | 1/10 | None | 4.2 | 15 | 2 weeks ago | No website | Build |
| 2 | Austin Drains | 3/10 | brokenstuff.com | 3.8 | 8 | 1 month ago | Outdated, no mobile | Build |
```

### Save CSV

Write `results.csv` to `lead-scout/runs/[session-name]/results.csv` with columns:
`name,address,phone,website,google_maps_url,yelp_url,facebook_url,found_on,rating,review_count,last_review_date,category,score,website_quality,review_health,lead_capture,mobile_score,pain_points,verdict`

Pain points column: semicolon-separated list.

### Save Summary

Write `summary.md` to `lead-scout/runs/[session-name]/summary.md` — the same table shown in chat plus full details per business.

---

## Phase 5: Auto-Build Top N

Read `top_n` and `auto_build` from `lead-scout/config/defaults.json`.

If `auto_build` is false, stop after the report and ask the user which businesses to build for.

If `auto_build` is true, take the top N businesses with verdict "Build" or "Strong Lead" and invoke the `biz-site-scout` skill:

- Pass business name, phone, location, niche, and **extracted assets** (logo, photos, colors, testimonials)
- Skip biz-site-scout's Phase 1 (Intake) and Phase 2 (Audit) — we already have this data
- Start biz-site-scout at Phase 4 (Niche Research) — or skip if we already have niche research from a previous run in the same niche
- Run Phase 5 (Generate), Phase 6 (Deploy), **Phase 7 (Verify)**, and Phase 8 (Outreach)
- biz-site-scout will use unique scroll transitions per section and run a full QA pass before outreach

**Asset Extraction:** During Phase 3 analysis, also extract reusable assets from each business website:
- Logo via Clearbit API: `https://logo.clearbit.com/{domain}`
- og:image and JSON-LD logo from HTML
- Work photos from gallery/portfolio sections
- Brand colors from CSS/inline styles
- Testimonials from structured data or review containers
- Save to `businesses/[slug]/audit/assets.json`

**Outreach Enhancement:** Include the specific pain points from our analysis:

> "Hey [Name], I was checking out [niche] companies in [city] and came across [Business]. You've got solid reviews ([rating] stars!) but I noticed [specific pain point]. I put together a sample site: [link]. No cost, no strings — just thought it could help turn those searches into calls."

**Premium Upsell Enhancement (if the Vault Space aesthetic was used):**
> "Hey [Name], I love the work you do at [Business]. I noticed [specific pain point] on your site, which doesn't reflect the high quality of your services. We specialize in building next-generation, premium digital experiences—I put together a high-end sample site to show you what's possible: [link]. Let me know what you think!"

Save generated sites to `lead-scout/runs/[session-name]/businesses/[slug]/generated-site/`.

---

## Final Summary

After all phases, present:

```
## Lead Scout — Complete

### [Niche] in [City]
- Scraped: X businesses
- After filtering: Y
- Analyzed: Z
- Sites built: N
- Verified: N passed (X warnings)

| Business | Score | Verified | Live URL | Outreach Ready? |
|----------|-------|----------|----------|-----------------|
| Joe's Plumbing | 1/10 | PASS | https://joes-plumbing.netlify.app | Email + SMS |
| Austin Drains | 3/10 | PASS (1 warn) | https://austin-drains.netlify.app | Email + SMS |

### Full report saved to:
- CSV: lead-scout/runs/[session]/results.csv
- Summary: lead-scout/runs/[session]/summary.md
```

Then show all outreach message drafts (email + SMS) for each built site.

---

## Adaptive Behavior

| Tool | Available | Fallback |
|------|-----------|----------|
| Apify MCP | Google Maps + Yelp + Facebook scraping + full site crawl | **REQUIRED — cannot run without** |
| Apify `website-content-crawler` | Full site crawl + asset extraction | WebFetch homepage only |
| Playwright CLI | Visual audit + QA verification screenshots | Code-only verification |
| 21st.dev Magic MCP | Design inspiration, hero patterns, transitions | Generate from niche-patterns.md |
| Clearbit Logo API | Instant logo: `logo.clearbit.com/{domain}` | Scrape DOM / favicon |
| biz-site-scout skill | Auto-build + verify + deploy + outreach | Just output the ranked list |
| Netlify CLI (`npx netlify-cli deploy`) | Live deploy + URL | Save files locally |
| Nano Banana MCP (`nanobanana`) | Motion effects from static photos (crossfade, parallax, morph) | CSS-only ken-burns / crossfade |
| sharp-cli (`npx sharp-cli`) | Resize images to <3MB before nanobanana | Skip motion effects |

If a tool is missing, the skill still works (except Apify) — it just loses that capability and tells the user.
