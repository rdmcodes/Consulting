# Lead Scout Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a `lead-scout` skill that scrapes businesses via Apify MCP, audits their web presence with Playwright + Claude vision, ranks them by need, and auto-builds top leads via biz-site-scout.

**Architecture:** Skill-based pipeline. User invokes `/lead-scout "niche" "city"`. Skill orchestrates: Apify MCP call → filter → parallel Playwright audits → scoring → ranked report + CSV → auto-handoff to biz-site-scout for top N.

**Tech Stack:** Apify MCP (Google Maps Scraper), Playwright CLI (screenshots), Claude vision (visual analysis), biz-site-scout skill (site generation), CSV/MD output.

**Design Doc:** `docs/plans/2026-03-05-lead-scout-design.md`

---

## Task 1: Configure Apify MCP

**Files:**
- Modify: `~/.claude/settings.json` (add Apify MCP server)

**Step 1: Add Apify MCP to Claude settings**

Add the `apify` entry to the `mcpServers` object in `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "apify": {
      "url": "https://mcp.apify.com",
      "headers": {
        "Authorization": "Bearer YOUR_APIFY_API_KEY"
      }
    }
  }
}
```

**Step 2: Verify MCP is connected**

Restart Claude Code or reload the window. Confirm Apify tools appear in the available MCP tools list. Test with a simple actor call if possible.

**Step 3: Commit**

No code to commit — this is a settings change.

---

## Task 2: Create lead-scout directory structure

**Files:**
- Create: `lead-scout/config/defaults.json`

**Step 1: Create directories and defaults config**

```bash
mkdir -p lead-scout/config lead-scout/runs
```

**Step 2: Write defaults.json**

Create `lead-scout/config/defaults.json`:

```json
{
  "top_n": 5,
  "exclude_chains": true,
  "inactive_threshold_months": 12,
  "min_reviews_to_consider_active": 1,
  "auto_build": true
}
```

**Step 3: Commit**

```bash
git add lead-scout/
git commit -m "feat: add lead-scout directory structure and defaults config"
```

---

## Task 3: Write the lead-scout SKILL.md

**Files:**
- Create: `~/.claude/skills/lead-scout/SKILL.md`

**Step 1: Create the skill directory**

```bash
mkdir -p ~/.claude/skills/lead-scout
```

**Step 2: Write SKILL.md**

Create `~/.claude/skills/lead-scout/SKILL.md`:

```markdown
---
name: lead-scout
description: Find and qualify business leads using Apify scraping. Use when the user wants to find businesses in a niche + area, prospect for leads, discover businesses that need better websites, or says "lead-scout". Triggers on prospecting, lead finding, business discovery, niche scouting, or market research for potential clients.
---

# Lead Scout

You are running the Lead Scout pipeline — a workflow that discovers businesses in a niche + area via Apify, audits their web presence, ranks them by how much they need digital services, and auto-builds sample sites for top leads.

## Invocation

The user provides a niche and location:
- `/lead-scout "plumbers" "Austin, TX"`
- `/lead-scout "landscapers" "Waco, TX"`
- Or naturally: "find me plumbers in Austin that need a website"

Parse the niche and location from whatever format the user provides. If either is missing, ask.

## Pipeline

```
Scrape (Apify MCP) → Filter → Analyze (Playwright + Vision) → Rank → Report → Auto-Build Top N
```

Run each phase in order. Present the ranked report to the user before auto-building.

---

## Phase 1: Scrape via Apify MCP

Use the Apify MCP to run the Google Maps Scraper actor.

**Actor:** `compass/crawler-google-places`

**Input to the actor:**
- `searchStringsArray`: ["[niche] in [city]"]
- `maxCrawledPlacesPerSearch`: 200 (or higher if user wants more)
- `language`: "en"
- `includeWebResults`: false

**Data to extract per business:**
- Business name
- Full address
- Phone number
- Website URL (may be null)
- Google rating (stars)
- Total review count
- Category/tags
- Google Maps URL
- Business status (open/closed)

If the Apify MCP is not available, stop and tell the user: "Apify MCP is required for lead-scout. Add it to your Claude settings."

---

## Phase 2: Filter Dead Leads

Remove businesses from the list that are:
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

Use parallel agents (subagent_type: "general-purpose") for businesses with websites — batch 5 at a time to avoid overwhelming the system.

### For businesses WITH a website:

**3a. Screenshot Capture (Playwright CLI):**

```bash
npx playwright screenshot --full-page --viewport-size="1440,900" "[URL]" "[output-path]/desktop.png"
npx playwright screenshot --full-page --viewport-size="390,844" "[URL]" "[output-path]/mobile.png"
```

Save screenshots to: `lead-scout/runs/[session-name]/businesses/[slug]/audit/`

Where `[session-name]` = `YYYY-MM-DD-[niche]-[city]` (e.g., `2026-03-05-plumbers-austin`)
Where `[slug]` = business name slugified (e.g., `joes-plumbing`)

**3b. Visual Analysis (Claude Vision):**

Read the desktop and mobile screenshot files and analyze:
- Is the design modern or outdated?
- Is it mobile responsive?
- Are there clear CTAs?
- Does it look professional?
- Any broken elements visible?

**3c. Technical Check (WebFetch):**

Use WebFetch on the URL to check:
- Does the site load? (or error/redirect)
- Has a contact form?
- Has online booking/scheduling?
- SSL present? (https vs http)
- Copyright year / last updated signs
- Meta tags present?

**Fallback:** If Playwright is not available, skip screenshots and use WebFetch-only analysis. Flag to the user that visual scoring will be less accurate.

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

For each business, identify applicable pain points from this list:
- No website → "No online presence — invisible to customers searching Google"
- No contact form → "Losing leads — visitors can't easily request service"
- No online booking → "Customers choosing competitors with easy scheduling"
- Low/old reviews → "Not leveraging social proof to build trust"
- Outdated design → "Website makes business look unprofessional or abandoned"
- Not mobile-friendly → "Losing 60%+ of visitors who browse on mobile"
- No SSL → "Browser shows 'Not Secure' warning — scares customers away"

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

Show the ranked table to the user:

```
## Lead Scout Results: [Niche] in [City]
### Scraped: X → Filtered: Y → Analyzed: Z

| # | Business | Score | Website | Rating | Reviews | Top Pain Point | Verdict |
|---|----------|-------|---------|--------|---------|----------------|---------|
| 1 | Joe's Plumbing | 1/10 | None | 4.2⭐ | 15 | No website | Build |
| 2 | Austin Drains | 3/10 | brokenstuff.com | 3.8⭐ | 8 | Outdated, no mobile | Build |
```

### Save CSV

Write `results.csv` to `lead-scout/runs/[session-name]/results.csv` with columns:
`name,address,phone,website,google_maps_url,rating,review_count,category,score,website_quality,review_health,lead_capture,mobile_score,pain_points,verdict`

Pain points column: semicolon-separated list.

### Save Summary

Write `summary.md` to `lead-scout/runs/[session-name]/summary.md` — the same table shown in chat plus full details per business.

---

## Phase 5: Auto-Build Top N

Read `top_n` from `lead-scout/config/defaults.json` (default: 5).

Take the top N businesses with verdict "Build" or "Strong Lead".

For each one, invoke the `biz-site-scout` skill with:
- The business name, phone, location
- The niche (already known)
- Skip biz-site-scout's Phase 1 (Intake) and Phase 2 (Audit) — we already have this data
- Start biz-site-scout at Phase 4 (Niche Research) — or skip if we already have niche research from a previous run
- Run Phase 5 (Generate) and Phase 6 (Deploy + Outreach)

**Outreach Enhancement:** When biz-site-scout drafts outreach messages, include the specific pain points from our analysis. Example:

> "Hey [Name], I was checking out [niche] companies in [city] and came across [Business]. You've got solid reviews (4.2 stars!) but I noticed you don't have a website — which means people searching Google for [niche] in [city] can't find you. I put together a sample site: [link]. No cost, no strings — just thought it could help turn those searches into calls."

Save generated sites to `lead-scout/runs/[session-name]/businesses/[slug]/generated-site/`.

---

## Presenting Final Results

After all phases, present:

```
## Lead Scout — Complete

### [Niche] in [City]
- Scraped: X businesses
- After filtering: Y
- Analyzed: Z
- Sites built: N

| Business | Score | Live URL | Outreach Ready? |
|----------|-------|----------|-----------------|
| Joe's Plumbing | 1/10 | https://joes-plumbing.netlify.app | ✓ Email + SMS |
| Austin Drains | 3/10 | https://austin-drains.netlify.app | ✓ Email + SMS |

### Outreach Messages
[Show email + SMS drafts for each built site]

### Full report saved to:
- CSV: lead-scout/runs/[session]/results.csv
- Summary: lead-scout/runs/[session]/summary.md
```

---

## Adaptive Behavior

| Tool | Available | Fallback |
|------|-----------|----------|
| Apify MCP | Google Maps scraping | **REQUIRED — cannot run without** |
| Playwright CLI | Visual screenshot audit | WebFetch text-based audit |
| biz-site-scout skill | Auto-build + deploy + outreach | Just output the ranked list |
| Netlify/Vercel MCP | Live deploy of generated sites | Save files locally |
| 21st.dev Magic MCP | Polished component-based sites | Generate from scratch |

If a tool is missing, the skill still works (except Apify) — it just loses that capability and tells the user.
```

---

## Implementation Tasks

### Task 1: Configure Apify MCP ✅
(Settings change — no code)

### Task 2: Create directory structure + defaults.json ✅
(Already covered above)

### Task 3: Write SKILL.md ✅
(The full skill content is above — that IS the implementation)

### Task 4: Test the pipeline

**Step 1:** Run `/lead-scout "pressure washing" "Waco, TX"` as a test

**Step 2:** Verify Apify MCP returns Google Maps data

**Step 3:** Verify filtering removes closed/inactive businesses

**Step 4:** Verify Playwright captures screenshots

**Step 5:** Verify scoring and ranking output

**Step 6:** Verify CSV export

**Step 7:** Verify auto-build triggers biz-site-scout

---

## Notes

- The skill is entirely prompt-based — no external code files needed. The SKILL.md IS the implementation.
- Apify costs apply per scrape. Google Maps Scraper charges per result.
- Playwright must be installed: `npx playwright install chromium`
- The skill reuses biz-site-scout for generation — no duplication.
