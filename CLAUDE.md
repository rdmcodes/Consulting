# CLAUDE.md — Consulting Business Agent Instructions

This is the operating manual for any Claude agent working in this repo. Read this before doing anything else.

---

## What This Business Does

We are a two-person digital consulting company that helps local businesses improve their online presence. Our clients are niche-agnostic — we've served pressure washing, landscaping, locksmith, roofing, roadside assistance, plumbing, and nightlife venue businesses.

**Core services we sell:**
- Website development & maintenance
- SEO (Google Business Profile optimization, local keyword strategy)
- Social media management (content creation, visual design, posting)
- Workflow automation (booking systems, email/SMS marketing, CRM)
- Monthly audit reports delivered in-person or via Zoom

**Business philosophy:**
- Lead with value first — build a free demo site, THEN pitch
- Build relationships, not transactions
- Gradually take more off the business owner's plate
- Request access to client Google accounts for real analytics
- Deliver a 30-day audit at end of each month showing exactly what was done

---

## The Two Founders

- **Dorien Mayfield** — works from this machine (`/Users/dorienmayfield/Projects/Consulting-main`)
- **Matthew (rdmcodes)** — co-founder, pushes to `https://github.com/rdmcodes/Consulting`

Both push directly to `main`. See `GIT-COLLABORATION-GUIDE.md` for workflow.

---

## Repo Structure — Know This Before Touching Anything

```
Consulting-main/
├── CLAUDE.md                          ← You are here
├── CONSULTING-PLAYBOOK.md             ← Master guide — read if you need deep context
├── BUSINESS-RESEARCH-TEMPLATE.md      ← Template for new business research docs
├── GIT-COLLABORATION-GUIDE.md         ← Two-person git workflow
│
├── businesses/                        ← ONE FOLDER PER CLIENT
│   └── Block and drum/                ← Reference example (read this)
│       ├── BLOCK-AND-DRUM.md          ← Full research doc (template for all clients)
│       ├── site/index.html            ← Generated website
│       ├── *.png / *.jpg              ← Business photos used in site
│       └── verify-*.png              ← QA screenshots
│
└── skills/
    └── lead-gen-skills-package/
        ├── README.md
        └── skills/
            ├── biz-site-scout/        ← Audits businesses + builds sites
            │   ├── SKILL.md
            │   └── references/
            │       ├── niche-patterns.md
            │       ├── scroll-transitions.md
            │       └── vast-space-aesthetic.md
            └── lead-scout/            ← Scrapes leads from Google Maps/Yelp/Facebook
                ├── SKILL.md
                └── config/defaults.json
```

### Business Folder Rule
**Every new client gets their own folder under `businesses/`.**

```
businesses/
└── Business Name/
    ├── BUSINESS-NAME.md        ← Research doc (use BUSINESS-RESEARCH-TEMPLATE.md)
    ├── site/
    │   ├── index.html          ← Generated website
    │   └── [photos/]           ← Any images referenced in index.html
    ├── audit/
    │   ├── desktop.png         ← Screenshot of their existing site (desktop)
    │   └── mobile.png          ← Screenshot of their existing site (mobile)
    └── verify-desktop.png      ← QA screenshot of OUR generated site
```

---

## Skills — How to Use Them

Skills are stored in `skills/lead-gen-skills-package/skills/`. They must also be installed at `~/.claude/skills/` to be invocable. The two skills are:

### 1. `lead-scout` — Find Leads
**Use when:** User wants to find businesses in a niche + city, prospect for clients, or do market research.

**Trigger phrases:** "find businesses", "prospect", "lead scout", "find me [niche] in [city]", "scout for leads", "which businesses need a website"

**What it does:**
1. Scrapes Google Maps, Yelp, and Facebook using Apify MCP
2. Crawls each business website (all pages, not just homepage)
3. Screenshots desktop + mobile views
4. Scores each business (design, UX, content, mobile) — lower score = higher priority
5. Ranks leads by how badly they need help
6. Auto-calls `biz-site-scout` to build sites for the worst ones

**Output goes to:** `skills/lead-scout/runs/YYYY-MM-DD-[niche]-[city]/`
- `results.csv` — ranked lead list
- `summary.md` — narrative summary
- `niche-research.md` — industry/market research
- `businesses/[business-name]/` — per-business audit + generated site

**Requires:** Apify MCP (non-optional). See `skills/lead-gen-skills-package/README.md` for setup.

---

### 2. `biz-site-scout` — Audit + Build Sites
**Use when:** User provides a list of businesses to audit, wants a website built for a specific business, or wants to evaluate web presence.

**Trigger phrases:** "audit this business", "build a site for", "biz-site-scout", "evaluate their website", "make a demo site"

**What it does (8 phases):**
1. Research & Audit — crawl the business's full site, screenshot it, extract assets (logo, colors, photos)
2. Niche Research — what do great sites in this industry look like?
3. Build Website — single HTML file, their real brand colors, real photos, unique scroll transitions
4. Deploy — push to Netlify, get a live URL
5. QA Verification — validate HTML, check images, verify mobile, take screenshots
6. Outreach Draft — casual email + SMS messages with the live link

**Website tech stack:**
- Single `index.html` file — no framework, no build step
- Tailwind CDN
- Vanilla JS (IntersectionObserver for scroll animations)
- Google Fonts
- FormSubmit.co for contact forms
- Deploy via `npx netlify-cli`

---

## The Full Pipeline (How We Go From Zero to Client)

```
Discovery → Research & Audit → Rank → Niche Research → Build Site → Deploy → QA → Outreach → Sales Call → Client Onboarding → Monthly Audit
```

1. **Discovery** — `lead-scout` scrapes niche + city or user provides a list
2. **Research & Audit** — `biz-site-scout` crawls every page, screenshots desktop + mobile, extracts brand assets
3. **Rank** — score 1-10 on design, UX, content, mobile. No site = auto-priority.
4. **Niche Research** — research what "great" looks like in their industry (top competitors, design patterns, must-have sections)
5. **Build Site** — generate a polished landing page using their actual brand identity
6. **Deploy** — Netlify, get a live URL (format: `business-name-demo.netlify.app`)
7. **QA** — mandatory before any outreach. Fix bugs, redeploy, re-verify.
8. **Outreach** — casual email + SMS. Sound like a real person, never a marketing agency.
9. **Sales Call** — demo the site, present pain points with industry stats, propose tiered services
10. **Client Onboarding** — request Google account access, establish monthly cadence
11. **Monthly Audit** — document everything done in the past 30 days, deliver in-person or Zoom

---

## Research Methodology — Do This for Every Business

When researching a business (for the research doc or for scraping), always gather:

- Business name, location, phone, owner name (if known)
- Website URL — crawl ALL pages (not just homepage)
- Google Business Profile — rating, review count, categories, hours, photos, posts
- Social media — Instagram followers, Facebook, TikTok (follower count, posting frequency)
- Reviews — Google, Yelp, Facebook (count, rating, recency, owner response rate)
- Competitors — top 3 local competitors, how they compare
- Pain points — list every problem you find (see common pain points below)
- Assets — logo, brand colors (from CSS/inline styles), work photos, testimonials

**Common pain points to look for:**
1. No website, or website is down
2. Outdated/non-mobile-responsive site
3. Google Business Profile incomplete (missing categories, no posts, few photos)
4. Low review count or old reviews
5. No review response strategy
6. No contact form or online booking
7. Social media inactive or absent
8. No email list or newsletter
9. No TikTok presence (major opportunity for venues/visual businesses)
10. Events/services scattered across platforms with no central hub
11. Thin content (pages with <100 words)
12. No SSL certificate

**Output:** Save everything to `businesses/[Business Name]/BUSINESS-NAME.md` using `BUSINESS-RESEARCH-TEMPLATE.md` as the structure.

---

## Website Generation — Quality Standards (Non-Negotiable)

### Tech
- Single `index.html` + Tailwind CDN + vanilla JS + Google Fonts
- `opacity: 0.05` for initial animation states — NEVER `opacity: 0` (Playwright screenshots go blank)
- Animation duration: 1.0–1.4s (not 0.5–0.8s — too jarring)
- IntersectionObserver threshold: 0.1, rootMargin: `'0px 0px -20px 0px'`

### Design Rules
- **Use their actual brand colors** — extract from their existing site/logo. Never substitute generic colors.
- **Use their real photos** — from their website, Instagram, Yelp, press articles. Only fall back to stock if nothing exists.
- **Never search stock photos by generic niche keyword** — "plumber" returns a wrench. Study the business's vibe, search for THAT.
- **Always visually verify every photo** before using it — Unsplash descriptions lie.
- **Show results customers get**, not workers doing the job.

### Scroll Transitions (12 patterns — never repeat adjacent sections)
Cascade Stagger, Horizontal Slide-In, Scale Pop, Clip Reveal, Blur Unblur, Rotate-In Tilt, Counter Slide, Rise & Spread, Typewriter Reveal, Parallax Float, Morph Border, Glowing Entrance

### Typography (8 pairings — rotate, never reuse across a batch)
Outfit+DM Sans, Playfair Display+Inter, Montserrat+Lora, Space Grotesk+Work Sans, Sora+Source Sans 3, Clash Display+General Sans, Cabinet Grotesk+Satoshi, Plus Jakarta Sans+Nunito Sans

### Required Sections (minimum for every site)
Hero → Trust Bar → Services → Testimonials → About → Gallery → Service Area → Contact → Footer

### QA Checklist (mandatory before outreach)
- [ ] All tags closed properly
- [ ] All images load (no broken icons)
- [ ] All links work
- [ ] No `opacity: 0` (use `0.05`)
- [ ] IntersectionObserver script present
- [ ] No duplicate transitions on adjacent sections
- [ ] Mobile layout is single-column
- [ ] Touch targets ≥ 44px
- [ ] No horizontal overflow
- [ ] Contact form posts to correct endpoint
- [ ] Desktop + mobile screenshots taken and look correct

---

## Niche Research — Do This Before Building for a New Industry

Before building a site for any niche you haven't worked in yet:
1. Find 3–5 top competitor websites (national + local)
2. Identify must-have sections for the niche
3. Note industry color schemes and trust signals
4. Identify key CTA strategy (call now, book online, get a quote, etc.)
5. Research local market context (city-specific competition)

See `skills/lead-gen-skills-package/skills/biz-site-scout/references/niche-patterns.md` for patterns already documented.

---

## Scraping Setup (Required for Lead Scout)

Scraping is powered by **Apify MCP**. Must be configured before running lead-scout.

```bash
claude mcp add --transport http apify https://mcp.apify.com --header "Authorization: Bearer YOUR_APIFY_API_KEY" --scope user
```

**Key Apify actors:**
- `compass/crawler-google-places` — Google Maps business data (up to 100 results)
- `spiders/yelp-search-scraper` — Yelp listings (~30 results per search)
- `apify/facebook-pages-scraper` — Facebook pages
- `apify/website-content-crawler` — Full site crawl (all pages, not just homepage)
- `apify/rag-web-browser` — Google search + page scraping (used for niche research + stock photos)

Crawl settings: `maxCrawlDepth: 3`, `maxCrawlPages: 25`, `crawlerType: "playwright:firefox"`

---

## Service Tiers (What We Sell)

| Tier | Services | Pricing |
|------|---------|---------|
| **Tier 1 — Foundation** | Website build, GBP optimization, basic SEO, contact form | $2–5K setup + $300–500/mo |
| **Tier 2 — Growth** | Review generation, social media (1–2 platforms), email/SMS marketing | $1–2K/mo |
| **Tier 3 — Full Partnership** | Everything above + ads, content creation, automation, analytics dashboard | $3–5K+/mo |

---

## Reference Example — Block & Drum

The gold standard for how this process should work. Read before starting any new business.

**Location:** `businesses/Block and drum/`
- `BLOCK-AND-DRUM.md` — research doc template (business intel, audit, pain points, services, talking points)
- `site/index.html` — example of a high-quality generated website (dark moody gold aesthetic)
- `verify-*.png` — what passing QA screenshots look like

**Live demo:** https://block-and-drum-demo.netlify.app

**What made it good:**
- Crawled ALL pages (not just homepage)
- Extracted 10 specific pain points (site down, GBP missing categories, no TikTok, etc.)
- Used real photos from their Instagram + Atlanta Magazine press coverage
- Dark moody gold aesthetic matched their brand exactly
- 12 unique scroll transitions, Playfair Display + Inter typography

---

## Deployed Portfolio

| Business | Niche | URL |
|----------|-------|-----|
| ABC Home & Commercial | Plumbing, Waco TX | https://abc-home-commercial-demo.netlify.app |
| HURT Roadside | Roadside Assistance, Atlanta GA | https://hurt-roadside-demo.netlify.app |
| Hamby Roof Repair | Roofing, Douglasville GA | https://hamby-roof-repair-demo.netlify.app |
| Carter Landscaping | Landscaping, Douglasville GA | https://carter-landscaping-demo.netlify.app |
| Block & Drum | Distillery/Venue, Chamblee GA | https://block-and-drum-demo.netlify.app |

Use these as proof of work and design references.

---

## Outreach Tone (Always)

Sound like a real person. Never corporate.

**Email:**
> Hey [Name], I was checking out [niche] spots in [city] and came across [Business Name]. Love what you're doing with [specific compliment]. I noticed [specific pain point] and put together a sample page to show what's possible: [LINK]. No strings attached — just thought it could help. Let me know what you think!

**SMS/DM:**
> Hey [Name]! Came across [Business Name] and loved [specific thing]. Put together a quick sample site for you: [LINK]. Totally free, just wanted to show what's possible. Lmk what you think!

---

## Key Rules — Never Break These

1. **Pull before pushing** — always `git pull origin main` first (two people on this repo)
2. **Business folder required** — every client gets `businesses/[Name]/` with research doc + site
3. **QA before outreach** — never share a demo site that hasn't been verified
4. **Brand colors are sacred** — never change a business's colors for "variety"
5. **Crawl all pages** — homepage-only audits miss 80% of what matters
6. **Opacity 0.05, not 0** — Playwright screenshots go blank with opacity:0
7. **Real photos beat stock** — always prefer actual business photos
8. **Never search stock by niche** — "landscaping" returns generic results; match the business's vibe instead
9. **Animation 1.0–1.4s** — faster feels cheap, slower feels sluggish
10. **Casual outreach wins** — no buzzwords, no corporate language, ever
