# Lead Scout — Design Document

**Date:** 2026-03-05
**Author:** Matthew + Claude
**Status:** Approved

## Purpose

A skill that automatically discovers businesses in a given niche + area, analyzes their web presence and pain points, ranks them by how much they need updated digital services, and auto-builds free sample websites for the top leads via biz-site-scout.

The strategy: lead with a free website to get in the door, then discover pain points in conversation and offer solutions.

## Invocation

```
/lead-scout "plumbers" "Austin, TX"
/lead-scout "landscapers" "Waco, TX" --top 10
```

## Pipeline

```
Scrape (Apify MCP) → Filter → Analyze (Playwright + Claude) → Rank → Report → Auto-Build Top N
```

---

## Phase 1: Scrape via Apify MCP

**Actor:** `compass/crawler-google-places` (Google Maps Scraper)

**Input:** Niche keyword + location (city/region)

**Data pulled per business:**
- Business name
- Address (full)
- Phone number
- Website URL (or null)
- Google rating (1-5 stars)
- Review count
- Most recent review date
- Business hours
- Category/tags
- Photos count
- Google Maps URL

**Config:** Scrape all results (no limit), let Apify paginate.

---

## Phase 2: Filter Dead Leads

Remove businesses that are:
- Permanently closed (Google Maps status)
- No reviews in 12+ months (likely inactive/abandoned)
- Duplicates (same phone or address)
- Chains/franchises (optional — controlled via `config/defaults.json`)

Flag businesses with no website as "No Website — High Priority" (don't filter them out).

---

## Phase 3: Analyze Web Presence + Pain Points

### 3a. Visual Audit (Playwright)

For businesses WITH a website:
1. Capture desktop screenshot: `npx playwright screenshot --full-page --viewport-size="1440,900" [URL] desktop.png`
2. Capture mobile screenshot: `npx playwright screenshot --full-page --viewport-size="390,844" [URL] mobile.png`
3. Feed screenshots to Claude vision for visual/UX analysis
4. Save screenshots to `runs/[session]/businesses/[name]/audit/`

For businesses with NO website:
- Skip screenshots
- Auto-score 1/10 across all visual categories
- Flag as highest priority

### 3b. Technical + Pain Point Analysis

For each business, assess:

**Website Quality Signals:**
- Does the site load? (HTTP status)
- Mobile responsive? (from mobile screenshot)
- Has contact form / clear CTA?
- Has online scheduling/booking?
- SSL certificate present?
- Last updated indicators (copyright year, blog dates)
- Overall visual quality (modern vs outdated)

**Pain Point Identification:**
- No contact form → "Losing leads — visitors can't easily reach you"
- No online booking → "Customers choosing competitors with easy scheduling"
- Low/no reviews → "Not leveraging social proof"
- Outdated design → "Site makes business look unprofessional"
- Not mobile-friendly → "Losing 60%+ of visitors on mobile"
- No Google Business Profile optimization → "Invisible in local search"
- Slow loading → "Visitors bouncing before page loads"

### 3c. Scoring Rubric

**Digital Presence Score (1-10):**

| Category | Points | Criteria |
|----------|--------|----------|
| Website Quality | 0-3 | 0 = no site, 1 = broken/ancient, 2 = functional but dated, 3 = modern + clean |
| Review Health | 0-3 | Rating + count + recency combined |
| Lead Capture | 0-2 | 0 = no forms/CTAs, 1 = basic contact info, 2 = forms + booking |
| Mobile Experience | 0-2 | 0 = not responsive, 1 = partially, 2 = fully responsive |

Lower score = needs you more.

Save analysis to `runs/[session]/businesses/[name]/audit/analysis.md`.

---

## Phase 4: Rank + Report

### In-Chat Output

Sort businesses by score (lowest first). Present summary table:

```
| # | Business | Score | Website | Rating | Reviews | Last Review | Top Pain Point | Verdict |
|---|----------|-------|---------|--------|---------|-------------|----------------|---------|
| 1 | Joe's Plumbing | 2/10 | None | 4.2 | 15 | 2 weeks ago | No website | Build |
| 2 | Austin Drains | 3/10 | broken-site.com | 3.8 | 8 | 1 month ago | Outdated, no mobile | Build |
| ... |
```

Verdicts:
- **Build** — No site or terrible site, active business, high priority
- **Strong Lead** — Has site but it's bad, would clearly benefit
- **Maybe** — Site is okay-ish, might still benefit
- **Skip** — Site is decent or business appears inactive

### CSV Export

Save to `runs/[session]/results.csv` with ALL fields:
- Name, address, phone, website, Google Maps URL, rating, review count, last review date, hours, category, score, website quality score, review health score, lead capture score, mobile score, pain points (semicolon-separated), verdict

### Summary

Save to `runs/[session]/summary.md` — human-readable version of the table.

---

## Phase 5: Auto-Build Top N

Take the top N businesses (default: 5, configurable) with verdict "Build" or "Strong Lead" and automatically invoke `biz-site-scout` skill:

1. Pass business name, phone, location, niche
2. biz-site-scout runs its full pipeline: niche research → generate landing page → deploy → draft outreach
3. Generated sites saved to `runs/[session]/businesses/[name]/generated-site/`
4. Outreach messages reference the specific pain points identified in Phase 3

### Outreach Enhancement

Because lead-scout identified specific pain points, the outreach messages are more targeted than generic biz-site-scout output:

> "Hey Joe, I was looking at plumbing companies in Austin and noticed you don't have a website yet — but you've got great reviews (4.2 stars!). I put together a sample site for you: [link]. No strings attached, just thought it could help you turn those Google searches into actual calls."

---

## Repo Organization

```
lead-scout/
├── runs/
│   └── YYYY-MM-DD-[niche]-[city]/
│       ├── results.csv
│       ├── summary.md
│       └── businesses/
│           ├── joes-plumbing/
│           │   ├── audit/
│           │   │   ├── desktop.png
│           │   │   ├── mobile.png
│           │   │   └── analysis.md
│           │   └── generated-site/
│           │       ├── index.html
│           │       └── outreach.md
│           └── austin-drain-pros/
│               ├── audit/
│               └── generated-site/
└── config/
    └── defaults.json
```

### defaults.json

```json
{
  "top_n": 5,
  "exclude_chains": true,
  "inactive_threshold_months": 12,
  "min_reviews_to_consider_active": 1,
  "auto_build": true
}
```

---

## MCP Requirements

### Apify MCP (Required)

Add to Claude settings (`~/.claude/settings.json` or project-level):

```json
{
  "mcpServers": {
    "apify": {
      "url": "https://mcp.apify.com",
      "headers": {
        "Authorization": "Bearer <APIFY_TOKEN>"
      }
    }
  }
}
```

### Playwright CLI (Required for visual audit)

```bash
npx playwright install chromium
```

Falls back to WebFetch text-based analysis if Playwright unavailable.

---

## Adaptive Behavior

| Tool | Available | Fallback |
|------|-----------|----------|
| Apify MCP | Google Maps scraping | Cannot run without this |
| Playwright CLI | Visual screenshot audit | WebFetch text-based audit |
| biz-site-scout skill | Auto-build + deploy + outreach | Just output the ranked list |
| Netlify/Vercel MCP | Live deploy of generated sites | Save files locally |
| 21st.dev Magic MCP | Polished component-based sites | Generate from scratch |

---

## Future Expansions (Not in v1)

- Additional data sources: Yelp API, Facebook pages, BBB listings
- Email finder integration (Hunter.io, Apollo)
- Tech stack analysis (BuiltWith)
- Social media presence scoring
- Automated follow-up scheduling
- CRM integration for lead tracking
