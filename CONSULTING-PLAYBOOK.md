# Consulting Business Playbook — Transfer Guide

> **Purpose:** This document is the complete brain dump for your consulting company. Drop this + the listed files into your shared repo so any Claude Code instance can operate with the same methodology, tools, and standards.

---

## Table of Contents

1. [What This Business Does](#1-what-this-business-does)
2. [The Full Process (How We Get Clients)](#2-the-full-process)
3. [Block & Drum — The Reference Example](#3-block--drum-reference-example)
4. [Tools & Infrastructure Setup](#4-tools--infrastructure-setup)
5. [Files to Copy Into the New Repo](#5-files-to-copy)
6. [The Research Methodology](#6-research-methodology)
7. [The Audit Framework](#7-audit-framework)
8. [Service Offering Design](#8-service-offering-design)
9. [Website Generation Standards](#9-website-generation-standards)
10. [Outreach & Sales Process](#10-outreach--sales)
11. [Deployed Portfolio](#11-deployed-portfolio)
12. [Key Rules & Lessons Learned](#12-key-rules)

---

## 1. What This Business Does

We're a **digital consulting company** that helps local businesses improve their online presence. Our model:

1. **Find businesses** with weak or no websites (automated via scraping)
2. **Research them deeply** — audit their web presence, Google profile, social media, reviews, competitors
3. **Build a free sample website** to demonstrate what they're missing
4. **Reach out** with the live demo link — casual, helpful, not salesy
5. **Convert to paying clients** — offer tiered services (website, Google optimization, social media, marketing automation)

**We are niche-agnostic.** We've done pressure washing, plumbing, roofing, landscaping, roadside assistance, nightlife venues, and staffing agencies. The process works for any local business.

---

## 2. The Full Process

### Pipeline Overview
```
Discovery → Research & Audit → Rank → Niche Research → Generate Website → Deploy → QA Verify → Outreach → Sales Call
```

### Phase-by-Phase Breakdown

#### Phase 1: Discovery (Finding Businesses)
**How we find leads:**
- **Automated (lead-scout skill):** Scrape Google Maps + Yelp + Facebook for businesses in a niche + city
  - Google Maps: `compass/crawler-google-places` (Apify) — up to 100 results
  - Yelp: `spiders/yelp-search-scraper` (Apify) — ~30 results per search
  - Facebook: `apify/facebook-pages-scraper`
  - Merge + deduplicate by phone number, then address
- **Manual:** User provides a list of businesses they've identified (CSV, pasted text, whatever)
- **In-person:** Met someone at a venue, got their info (Block & Drum example)

#### Phase 2: Research & Audit (Deep Business Intelligence)
For each business, we gather:
- **Website audit** — full site crawl (all pages, not just homepage), screenshots desktop + mobile
- **Google Business Profile** — rating, review count, categories, hours, photos, posts activity
- **Social media** — Instagram followers, Facebook page, TikTok presence, posting frequency
- **Reviews** — Google, Yelp, Facebook reviews (rating, count, recency, response rate)
- **Competitor benchmarks** — how do top 3 local competitors compare?
- **Asset extraction** — logo, brand colors, work photos, testimonials, services list

**Output:** A comprehensive business doc (like BLOCK-AND-DRUM.md) + audit screenshots + assets.json

#### Phase 3: Rank & Triage
Score each business 1-10 across:
- Visual Design (1-10)
- UX & Usability (1-10)
- Content Quality (1-10)
- Mobile Experience (1-10)

Group them:
- **No Website** — highest priority
- **Needs Full Rebuild** (1-3) — site is hurting them
- **Needs Improvement** (4-6) — functional but not competitive
- **Decent/Strong** (7+) — skip for now

#### Phase 4: Niche Research
Before building anything, research what "great" looks like in their industry:
- Top competitor websites (3-5 standout examples)
- Must-have sections for the niche
- Industry color schemes, trust signals, CTA strategy
- Local market context

#### Phase 5: Generate Website
Build a single-page landing page using:
- Business's own brand colors (NEVER generic)
- Their real logo, photos, testimonials
- Unique scroll transitions per section (12 patterns in our library)
- Mobile-first, conversion-optimized
- Single HTML file + Tailwind CDN (no build step needed)

#### Phase 6: Deploy
Push to Netlify via CLI → get a live URL (e.g., `business-name-demo.netlify.app`)

#### Phase 7: Visual QA (MANDATORY)
Before showing anyone:
- HTML validation (unclosed tags, broken images, broken links)
- Scroll transition audit (no bugs, proper timing)
- Responsive check (mobile/desktop)
- Screenshot verification (Playwright)
- Fix any critical issues, redeploy, re-verify

#### Phase 8: Outreach
Draft casual messages (email + SMS) with the live link:
- Compliment something specific about their business
- Mention you noticed [specific pain point]
- Share the sample site link
- Soft CTA: "let me know what you think"

#### Phase 9: Sales Call / Follow-Up
- Present the demo site
- Discuss pain points identified in research
- Propose tiered services
- Close the deal

---

## 3. Block & Drum — The Reference Example

This is our most thorough example of the full process. Use this as the template for how to approach ANY business.

### The Business
- **Name:** Block & Drum
- **Type:** Distillery, HiFi Lounge, Cafe, Live Music & Events Venue
- **Location:** 5105 Peachtree Blvd Building B, Chamblee GA 30341
- **Owner:** Justin Staples (ex-Bacardi executive)
- **Instagram:** @blockanddrum — 45.7K followers
- **Website:** blockanddrum.com (intermittently DOWN — critical problem)

### What We Found (Research Phase)

**10 Key Problems Identified:**
1. Website is intermittently down — every Google click bounces
2. Google Business Profile only lists "Distillery" (missing 6+ categories: Bar, Cafe, Music Venue, Event Space, etc.)
3. No Google Posts activity
4. Unknown review response rate
5. Events scattered across Eventbrite, Instagram, community groups — no central hub
6. No published food truck schedule (they rotate food trucks)
7. Zero TikTok presence (huge missed opportunity for a venue)
8. Facebook underutilized
9. No email list / newsletter
10. No SMS alerts for events

**Competitive Gaps:**
- 119 Google reviews vs. 561+ for top Atlanta competitors
- No online booking system
- No loyalty program
- No centralized event calendar

### What We Built
- Dark moody gold aesthetic website (matching their brand vibe)
- Playfair Display + Inter typography
- 12+ unique scroll transitions (blur-in, cascade, parallax, clip-reveal, etc.)
- Sections: Hero, Events, Distillery, Vinyl Room, Courtyard, Cafe, Food Trucks, Farm, Gallery, Contact
- Real photos from their Instagram + Atlanta Magazine articles
- Live at: https://block-and-drum-demo.netlify.app

### Service Tiers Proposed

**Tier 1 — Immediate Wins ($3-5K setup + $500-1K/mo retainer):**
- Website rebuild + hosting
- Google Business Profile optimization (add categories, posts, photos)
- Review generation system (QR codes, follow-up emails)

**Tier 2 — Growth Engine ($1-2K/mo):**
- SMS event alerts
- Email newsletter (weekly events digest)
- Weekly flyer/poster design
- Facebook + Instagram ad management

**Tier 3 — Full Partnership ($3-5K/mo):**
- TikTok content creation
- Full social media calendar management
- Performer booking portal
- Loyalty club program
- Analytics dashboard

### How We Got the Meeting
- Matthew went to Block & Drum in person
- Met Justin Staples
- Offered consulting services
- Follow-up: share the demo website link + propose services

### Key Takeaway
The research doc (BLOCK-AND-DRUM.md) is the template. For every business you approach, create a similar doc with: business intel, audit findings, problems identified, service opportunities, talking points with industry stats, and design direction.

---

## 4. Tools & Infrastructure Setup

### MCP Servers (MUST configure in ~/.claude.json)

These are the AI-powered tools that automate the pipeline. Without them, you can still work but lose capabilities.

#### 1. Apify MCP (Web Scraping — REQUIRED)
```bash
claude mcp add --transport http apify https://mcp.apify.com --header "Authorization: Bearer YOUR_APIFY_API_KEY" --scope user
```
- Get API key: https://console.apify.com/account/integrations
- **Used for:** Google Maps scraping, Yelp scraping, full website crawls, web search + page scraping
- **Key Actors:**
  - `apify/website-content-crawler` — crawl entire websites (all pages)
  - `apify/rag-web-browser` — Google search + scrape results
  - `compass/crawler-google-places` — Google Maps business data
  - `spiders/yelp-search-scraper` — Yelp business listings

#### 2. 21st.dev Magic MCP (Design Inspiration)
```bash
claude mcp add magic --scope user -- npx -y @21st-dev/magic@latest
```
Add env var in `~/.claude.json`:
```json
"magic": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@21st-dev/magic@latest"],
  "env": { "API_KEY": "YOUR_21ST_DEV_API_KEY" }
}
```
- Get API key: https://21st.dev/settings
- **Used for:** Design component inspiration, logo search. Returns React/Tailwind — extract design DNA, rebuild in vanilla HTML.

#### 3. Nano Banana MCP (Image Effects via Gemini)
```bash
claude mcp add nanobanana --scope user -- npx -y gemini-nanobanana-mcp@latest
```
Add env var in `~/.claude.json`:
```json
"nanobanana": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "gemini-nanobanana-mcp@latest"],
  "env": { "GEMINI_API_KEY": "YOUR_GOOGLE_AI_KEY" }
}
```
- Get API key: https://aistudio.google.com/apikey
- **Used for:** Motion effects from static photos (crossfade, parallax layers, morph)
- **IMPORTANT:** Resize images to <4MB with `sharp-cli` before sending

### Global NPM Packages
```bash
npm install -g sharp-cli        # Image resizing before nanobanana
npx playwright install           # Browser automation for screenshots
```

### Accounts Needed

| Service | Purpose | Setup |
|---------|---------|-------|
| **Netlify** | Deploy generated websites | https://app.netlify.com — free tier works |
| **FormSubmit.co** | Contact form handling | No account — forms POST to `https://formsubmit.co/YOUR_EMAIL` |
| **Apify** | Web scraping platform | https://console.apify.com — free tier has limits |
| **Google AI Studio** | Gemini image generation | https://aistudio.google.com/apikey — free tier available |
| **21st.dev** | Design component library | https://21st.dev/settings — API key needed |

### Deployment Details
- **Netlify CLI:** `npx netlify-cli deploy --prod --dir=FOLDER --site SITE_ID`
- **Create new site:** `npx netlify-cli sites:create --name business-name-demo`
- **Auth token location:** `~/AppData/Roaming/netlify/Config/config.json` (Windows)
- **Contact forms:** `<form action="https://formsubmit.co/YOUR_EMAIL" method="POST">`

---

## 5. Files to Copy Into the New Repo

### MUST COPY — Core Skills
These are the "brain" that powers the pipeline:

```
~/.claude/skills/biz-site-scout/
├── SKILL.md                              # Main pipeline (8 phases, ~45KB)
├── references/
│   ├── scroll-transitions.md             # 12 scroll transition patterns + CSS + JS
│   ├── niche-patterns.md                 # Design patterns for 7+ niches
│   └── vast-space-aesthetic.md           # Premium dark theme for high-end clients

~/.claude/skills/lead-scout/
├── SKILL.md                              # Lead discovery pipeline (6 phases, ~17KB)
└── config/
    └── defaults.json                     # Settings (top_n, auto_build, etc.)
```

### MUST COPY — This Playbook
```
CONSULTING-PLAYBOOK.md                    # This file — the master guide
```

### SHOULD COPY — Reference Examples
```
Block and drum/
├── BLOCK-AND-DRUM.md                     # Template for business research docs
└── site/index.html                       # Example of a generated website

career-staffing/
└── research/NICHE-RESEARCH.md            # Example of deep niche research
```

### SHOULD COPY — Memory Template
```
MEMORY-TEMPLATE.md                        # Template for project memory (create from MEMORY.md)
```

### Configuration
```
~/.claude.json                            # MCP server configs (copy structure, use your own API keys)
```

### How to Install Skills in the New Repo
Skills go in `~/.claude/skills/SKILL-NAME/SKILL.md`. Claude Code auto-discovers them. The new repo partner needs:

1. Copy the skill folders to their `~/.claude/skills/` directory
2. Configure MCP servers in their `~/.claude.json` with their own API keys
3. Install global npm packages (`sharp-cli`, `playwright`)
4. Set up Netlify account + auth token
5. Set up Apify account + API key

---

## 6. Research Methodology

### For Any Business — The Standard Research Process

**Step 1: Gather Business Intel**
- Name, location, phone, website URL, owner name
- Google Business Profile (rating, reviews, categories, hours)
- Social media accounts (Instagram, Facebook, TikTok, YouTube)
- Any press coverage or articles about them

**Step 2: Full Website Audit**
- Crawl ALL pages (not just homepage) using `apify/website-content-crawler`
  - `maxCrawlDepth: 3`, `maxCrawlPages: 25`, `crawlerType: "playwright:firefox"`
- Screenshot desktop (1440x900) and mobile (390x844) with Playwright
- Screenshot 1-2 key inner pages (services, about)

**Step 3: Extract Assets**
- Logo: Clearbit API → JSON-LD → og:image → DOM scraping → favicon
- Work photos: Gallery/portfolio pages, lazy-loaded images, CSS backgrounds
- Brand colors: CSS/inline styles, og:theme-color
- Testimonials: Schema.org Review data, review containers
- Services list: Service page headings and descriptions

**Step 4: Competitive Analysis**
- Find top 3 local competitors in the same niche + city
- Compare: review count, rating, website quality, social following
- Note what they do well that the target business doesn't

**Step 5: Identify Problems (Pain Points)**
Common pain points we look for:
1. No website or website is down
2. Website is outdated/ugly/not mobile-friendly
3. Google Business Profile incomplete (missing categories, no posts, few photos)
4. Low review count or old reviews
5. No review response strategy
6. No contact form or online booking
7. Social media inactive or underutilized
8. No email list or newsletter
9. Events/services scattered across platforms (no central hub)
10. Missing key pages (services, about, gallery)
11. No SSL certificate
12. Thin content (pages with <100 words)
13. Dead blog (last post months/years ago)

**Step 6: Design Service Offerings**
Map each problem to a service you can sell:
- Website down → Website rebuild ($3-5K + hosting)
- GBP incomplete → GBP optimization ($500-1K)
- Low reviews → Review generation system ($500/mo)
- No social → Social media management ($1-3K/mo)
- No events hub → Custom booking/events platform ($2-5K)

**Step 7: Compile Business Document**
Create a `BUSINESS-NAME.md` with everything:
- Business info
- Audit findings
- Problems identified (ranked by impact)
- Service opportunities (tiered)
- Talking points with industry stats
- Design direction for the website
- Photo inventory
- Next steps

### For a New Niche — Deep Niche Research
When entering a new industry you haven't worked in:
1. Top 10-15 competitor websites (national + local)
2. Industry market size and trends
3. Essential tools/software for that industry
4. Legal/compliance requirements
5. Revenue models and pricing benchmarks
6. Website best practices (must-have pages, features)
7. SEO strategy (target keywords, local SEO)
8. Social media strategy (which platforms, content types)

See `career-staffing/research/NICHE-RESEARCH.md` for a thorough example.

---

## 7. Audit Framework

### Scoring Rubric (1-10 per category)

**Visual Design (1-10)**
- 1-3: Outdated, clipart, no branding, looks like 2005
- 4-6: Generic template, decent but forgettable
- 7-10: Clean, modern, professional, unique brand identity

**UX & Usability (1-10)**
- 1-3: Confusing nav, broken elements, no clear CTAs
- 4-6: Functional but clunky, CTAs exist but weak
- 7-10: Intuitive, clear CTAs, smooth user journey

**Content Quality (1-10)**
- 1-3: Thin/missing content, no service descriptions, errors
- 4-6: Basic info but generic, template-fill energy
- 7-10: Compelling copy, clear services, unique voice

**Mobile Experience (1-10)**
- 1-3: Not responsive, impossible on phone
- 4-6: Partially responsive, awkward on mobile
- 7-10: Fully responsive, mobile-first feel

### Verdicts
- **No Website** — auto-priority
- **Needs Full Rebuild** (1-3) — site is hurting them
- **Needs Improvement** (4-6) — functional but not competitive
- **Decent** (7-8) — could still upsell services
- **Strong** (9-10) — skip

### Pain Point Scoring (Lead Scout)
More granular scoring for automated ranking:
- Website Quality (0-3): 0=no site, 1=broken/ancient, 2=functional/dated, 3=modern
- Review Health (0-3): rating + count + recency
- Lead Capture (0-2): 0=no forms, 1=basic contact, 2=forms+booking
- Mobile Experience (0-2): 0=not responsive, 1=partial, 2=fully responsive
- **Lower score = higher priority = needs us more**

---

## 8. Service Offering Design

### Standard Service Tiers (Adapt Per Business)

**Tier 1 — Foundation (One-Time + Small Retainer)**
- Website build/rebuild
- Google Business Profile optimization
- Basic SEO setup
- Contact form + booking integration
- Pricing: $2-5K setup + $300-500/mo maintenance

**Tier 2 — Growth (Monthly Retainer)**
- Review generation system
- Social media management (1-2 platforms)
- Email/SMS marketing setup
- Monthly content updates
- Pricing: $1-2K/mo

**Tier 3 — Full Partnership (Premium Retainer)**
- All Tier 1 + 2
- Paid advertising (Google/Facebook/Instagram ads)
- Content creation (video, photography, blog)
- Advanced analytics + reporting
- Custom automation (booking, CRM, etc.)
- Pricing: $3-5K+/mo

### Industry-Specific Talking Points
Always back up your pitch with data. Examples we've used:

- "88% of consumers trust online reviews as much as personal recommendations" (BrightLocal)
- "46% of all Google searches are looking for local information" (Google)
- "76% of people who search for something nearby visit a business within a day" (Google)
- "Businesses that respond to reviews earn 35% more revenue on average" (Harvard Business Review)
- "Adding secondary Google Business categories can increase visibility by 70%+" (Semrush)
- "Mobile traffic accounts for 60%+ of all web traffic" (StatCounter)
- "53% of mobile users abandon sites that take longer than 3 seconds to load" (Google)

---

## 9. Website Generation Standards

### Tech Stack
- **Single HTML file** — no framework, no build step
- **Tailwind CDN** — `<script src="https://cdn.tailwindcss.com"></script>`
- **Google Fonts** — `<link>` tags in `<head>`
- **Vanilla JS** — IntersectionObserver for scroll animations
- **FormSubmit.co** — contact form handling

### Design Rules

**Colors:**
- ALWAYS use the business's own brand colors — extract from their existing site/logo
- Never vary colors for "uniqueness" — colors ARE the brand
- Only use fallback niche colors if business has zero web presence

**Photos:**
- CRITICAL RULE: Never search stock photos by generic niche keyword ("plumbing", "restaurant")
- Instead: study the business's actual website screenshots, identify their VIBE, search for photos matching that vibe
- Use real business photos first (from their site, social media, Yelp)
- Only fall back to stock (Unsplash) if no real photos available
- Always visually verify every photo before using it
- Show RESULTS the customer gets, not workers doing the job

**Typography (8 pairings to rotate):**
1. Outfit + DM Sans
2. Playfair Display + Inter
3. Montserrat + Lora
4. Space Grotesk + Work Sans
5. Sora + Source Sans 3
6. Clash Display + General Sans
7. Cabinet Grotesk + Satoshi
8. Plus Jakarta Sans + Nunito Sans

**Scroll Transitions (12 patterns — never repeat adjacent):**
1. Cascade Stagger — children animate in sequence
2. Horizontal Slide-In — alternating left/right
3. Scale Pop — bounce from 80%
4. Clip Reveal — curtain effect
5. Blur Unblur — starts blurred, sharpens
6. Rotate-In Tilt — 3D rotation
7. Counter Slide — left from left, right from right
8. Rise & Spread — container rises, children spread
9. Typewriter Reveal — character by character
10. Parallax Float — background slower than foreground
11. Morph Border — border draws itself
12. Glowing Entrance — fade in with glow

**Section Dividers (5 shapes — rotate, never repeat):**
wave → diagonal → curve → mountain → arrow

**Critical Technical Rule:**
- Use `opacity: 0.05` (NOT `0`) for initial animation states — Playwright screenshots show blank sections with opacity:0
- Animation durations: 1.0-1.4s (not 0.5-0.8s — too jarring)
- IntersectionObserver threshold: 0.1 with rootMargin '0px 0px -20px 0px'

### Hero Layouts (Rotate — Never Repeat in Same Batch)
1. Centered dramatic — huge headline, gradient text, glow CTA
2. Split asymmetric — text 55%, image 45% with clip-path
3. Overlay dark — full-bleed image, dark gradient overlay
4. Bento hero — headline + bento grid cards
5. Video-style — image with play button overlay
6. Aerospace Premium — dark mode, massive geometric typography

### Required Sections (Minimum)
1. Hero — headline, CTA, phone
2. Trust bar — ratings, certifications
3. Services — grid/cards
4. Social proof — testimonials
5. About — company story
6. Gallery — work photos
7. Service area — cities served
8. Contact — form + phone
9. Footer — full business info

### QA Verification Checklist (Phase 7 — MANDATORY)
Before sharing any site:
- [ ] All tags closed properly
- [ ] All images load (no broken icons)
- [ ] All links work (no 404s)
- [ ] CSS has no syntax errors
- [ ] JS has no errors
- [ ] Fonts load correctly
- [ ] Contact form posts to correct endpoint
- [ ] Schema.org JSON-LD is valid
- [ ] Meta tags present (title, description, viewport)
- [ ] No opacity:0 (use 0.05)
- [ ] IntersectionObserver script present
- [ ] No duplicate transitions on adjacent sections
- [ ] Mobile layout is single-column
- [ ] Touch targets ≥ 44px
- [ ] No horizontal overflow
- [ ] Desktop + mobile screenshots look correct

---

## 10. Outreach & Sales

### Outreach Tone
- Sound like a real person, not a marketing agency
- No corporate buzzwords ("leverage", "synergy", "optimize")
- Be specific (mention their business name, city, services)
- Friendly and helpful, never pushy

### Email Template
```
Subject: Quick idea for [Business Name]

Hey [Name],

I was checking out [niche] spots in [city] and came across [Business Name].
Love what you're doing with [specific compliment — their food, their vibe, their reviews].

I noticed [specific pain point — site is down, no Google posts, etc.] and actually
put together a sample page to show what's possible: [LIVE LINK]

No strings attached — just thought it could help. Let me know what you think!

[Your Name]
```

### SMS/DM Template
```
Hey [Name]! Came across [Business Name] and loved [specific thing].
Put together a quick sample site for you: [LINK]
Totally free, just wanted to show what's possible. Lmk what you think!
```

### In-Person Follow-Up (Block & Drum Scenario)
When you've already met the owner:
```
Hey Justin, great meeting you last night at Block & Drum.
I went ahead and put together a sample site like I mentioned: [LINK]
Would love to walk you through it + chat about a few things I noticed
that could help drive more traffic. Coffee this week?
```

### Sales Call Framework
1. **Open casual** — reference the meeting/outreach, ask about their business
2. **Show the demo** — walk through the sample site on screen share
3. **Discuss pain points** — "I noticed [problem] during my research..."
4. **Present stats** — industry data that shows why it matters
5. **Propose services** — start with Tier 1, let them upgrade
6. **Soft close** — "Want me to make this site live for you?"

---

## 11. Deployed Portfolio

Sites we've built and deployed (use as references/proof of work):

| Business | Niche | Location | URL | Status |
|----------|-------|----------|-----|--------|
| ABC Home & Commercial | Plumbing | Waco, TX | https://abc-home-commercial-demo.netlify.app | Live |
| HURT Roadside | Roadside Assistance | Atlanta, GA | https://hurt-roadside-demo.netlify.app | Live |
| Hamby Roof Repair | Roofing | Douglasville, GA | https://hamby-roof-repair-demo.netlify.app | Live |
| Carter Landscaping | Landscaping | Douglasville, GA | https://carter-landscaping-demo.netlify.app | Live |
| Block & Drum | Distillery/Venue | Chamblee, GA | https://block-and-drum-demo.netlify.app | Live |

---

## 12. Key Rules & Lessons Learned

### Things We Learned the Hard Way

1. **Never use opacity:0 for animations** — Playwright screenshots show blank sections. Use 0.05.

2. **Never search stock photos by niche keyword** — "plumber" returns a guy with a wrench. Instead, study the business's actual vibe and search for THAT. A roadside company that shows dark SUVs at night → search "dark car city night".

3. **Always visually verify photos** — Unsplash descriptions lie. "Man cleaning driveway" might show a gas station. Screenshot the actual image before using it.

4. **Crawl ALL pages, not just the homepage** — Services, about, gallery pages contain the assets and info you need. Homepage-only audits miss 80% of the picture.

5. **Brand colors are sacred** — Never change a business's colors for "uniqueness". Colors ARE their brand. Vary layout, transitions, typography instead.

6. **Business owners don't care about Lighthouse scores** — They care about "does this look good?" Focus on visual impact, not technical metrics.

7. **QA before outreach, always** — Sending a business owner a broken demo site kills credibility instantly. Phase 7 exists for a reason.

8. **Casual outreach wins** — "Hey, I made this for you" beats "Dear Sir/Madam, we at Digital Solutions LLC would like to leverage synergies..." every time.

9. **Show results, not process** — Stock photos should show the RESULT the customer gets (clean driveway, beautiful home) not the worker doing the job (guy with a pressure washer).

10. **Real photos > stock photos, always** — Even a mediocre real photo of their actual business beats a perfect stock image. Authenticity matters.

11. **Animation durations matter** — 0.5-0.8s feels jarring and cheap. 1.0-1.4s feels smooth and premium.

12. **FormSubmit.co works great** — No backend needed, no account needed, just POST to the endpoint. Add a `_next` hidden input to redirect back to the site after submission.

13. **Nanobanana has rate limits** — Google AI Studio free tier: ~10 requests/minute. Queue sequentially, 10s delays, max 5 per business. Fall back to CSS ken-burns if rate-limited.

14. **Netlify CLI from VS Code works** — Set `NETLIFY_AUTH_TOKEN` env var, use `npx netlify-cli deploy --prod --dir=FOLDER --site=SITE_ID`.

15. **The business doc is the secret weapon** — A thorough BUSINESS-NAME.md with problems, services, talking points, and stats makes the sales call easy. You know more about their online presence than they do.

---

## Appendix: Quick Start Checklist

For a new team member to get up and running:

- [ ] Install Claude Code (VS Code extension)
- [ ] Copy skill files to `~/.claude/skills/` (biz-site-scout + lead-scout)
- [ ] Set up Apify account + API key
- [ ] Configure MCP servers in `~/.claude.json`
- [ ] Install npm packages: `sharp-cli`, `playwright`
- [ ] Set up Netlify account
- [ ] Set up FormSubmit email endpoint
- [ ] Read this playbook
- [ ] Read BLOCK-AND-DRUM.md as the reference example
- [ ] Pick a business and run the pipeline

---

*Last updated: 2026-03-15*
*Created by: Matthew (mmmoorer@yahoo.com)*
