---
name: biz-site-scout
description: Audit business websites and generate replacements. Use this skill whenever the user provides a list of businesses to research, wants to evaluate websites, score web presence, audit local business sites, generate landing pages for businesses with bad or no websites, or mentions outreach to businesses. Also use when the user mentions scraping results, lead lists, prospecting, or building sites for clients. Triggers on any business audit, website review, competitor analysis, or cold outreach workflow — even if the user doesn't say "biz-site-scout" by name.
---

# Biz Site Scout

You are running the Biz Site Scout pipeline — a workflow that takes a list of businesses, audits their web presence, and for the weakest ones, generates a polished landing page, deploys it, and drafts outreach messages.

This skill is niche-agnostic. It works for any business type (pressure washing, plumbing, restaurants, salons, landscaping, etc.). The niche is determined from the business list the user provides.

## Setup & Prerequisites

**Anyone running this skill needs the following configured. Without these, the pipeline still works but loses capabilities (see Adaptive Behavior table at bottom).**

### Required MCP Servers

Add these to `~/.claude.json` under `"mcpServers"`, or run the CLI commands:

**1. Apify MCP (web scraping, site crawling, RAG browser):**
```bash
claude mcp add --transport http apify https://mcp.apify.com --header "Authorization: Bearer YOUR_APIFY_API_KEY" --scope user
```
- Get API key at: https://console.apify.com/account/integrations
- Used for: `apify/website-content-crawler` (full site crawl), `apify/rag-web-browser` (Unsplash image search), `compass/crawler-google-places`, `spiders/yelp-search-scraper`

**2. 21st.dev Magic MCP (UI component inspiration):**
```bash
claude mcp add magic --scope user -- npx -y @21st-dev/magic@latest
```
Then add env var in `~/.claude.json`:
```json
"magic": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@21st-dev/magic@latest"],
  "env": { "API_KEY": "YOUR_21ST_DEV_API_KEY" }
}
```
- Get API key at: https://21st.dev/settings
- Used for: design inspiration, component patterns, logo search. Returns React/Tailwind code — extract design DNA and rebuild in vanilla HTML + Tailwind CDN.

**3. Nano Banana MCP (motion effects from static photos):**
```bash
claude mcp add nanobanana --scope user -- npx -y gemini-nanobanana-mcp@latest
```
Then add env var in `~/.claude.json`:
```json
"nanobanana": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "gemini-nanobanana-mcp@latest"],
  "env": { "GEMINI_API_KEY": "YOUR_GOOGLE_AI_KEY" }
}
```
- Get API key at: https://aistudio.google.com/apikey (Google AI Studio, free tier available)
- **IMPORTANT:** The correct npm package is `gemini-nanobanana-mcp` (uses Gemini API directly). Do NOT use `@aeven/nanobanana-mcp` (requires OpenRouter, not Gemini) or `nanobanana-mcp-server` (does not exist on npm).
- Uses Gemini 2.5 Flash Image model (`gemini-2.5-flash-image`) — for generating images and creating motion/animation effects from existing static business photos
- **Image size limit:** ~4MB per image. Always resize before sending (see sharp-cli below)
- **Free tier rate limits:** Google AI Studio free tier has aggressive rate limits (~10 requests/minute). See Rate Limit Handling section below.

### Required NPM Packages (Global)

```bash
npm install -g sharp-cli    # Image resizing before nanobanana (keeps under 4MB limit)
npx playwright install      # Browser automation for screenshots + visual QA
```

### Accounts & Credentials

| Service | What For | How to Get |
|---------|----------|------------|
| **Netlify** | Deploy generated sites | https://app.netlify.com — auth token in `~/AppData/Roaming/netlify/Config/config.json` |
| **FormSubmit.co** | Contact form submissions | No account needed — forms POST to `https://formsubmit.co/YOUR_EMAIL` |
| **Clearbit Logo API** | Business logos | No key needed — `https://logo.clearbit.com/{domain}` is free |
| **Google Fonts** | Typography | No key needed — `<link>` tags in HTML |

### File Structure

```
~/.claude/skills/biz-site-scout/
├── SKILL.md                          # This file — the full pipeline
├── references/
│   ├── scroll-transitions.md         # 12 scroll transition patterns + CSS + JS observer
│   └── niche-patterns.md             # Design patterns per niche (colors, sections, trust signals)
```

Generated output goes to the working directory:
```
[working-dir]/generated-sites/[business-slug]/
├── audit/
│   ├── assets.json                   # Extracted logo, photos, colors, testimonials
│   ├── desktop.png                   # Audit screenshot
│   └── mobile.png                    # Audit screenshot
├── generated-site/
│   ├── index.html                    # The generated landing page
│   └── assets/motion/                # Nano Banana motion effect images
├── verify-desktop.png                # QA verification screenshot
└── verify-mobile.png                 # QA verification screenshot
```

### Outreach Config

- Default outreach email: set by the user running the pipeline
- FormSubmit endpoint: `https://formsubmit.co/USER_EMAIL` — replace with actual email in generated forms
- Tone: casual, human, not salesy (see Phase 8 for examples)

---

## Pipeline Overview

```
Intake → Audit + Asset Extract → Rank → Niche Research → Generate → Deploy → Verify → Outreach
```

Run each phase in order. Present results to the user between phases for approval before continuing.

---

## Phase 1: Intake & Parse

Accept the business list in whatever format the user provides (CSV, JSON, pasted text, spreadsheet link, etc.).

For each business, extract:
- **Name** (required)
- **Location** (city, state — required)
- **Phone** (if available)
- **Website URL** (if available — "none" is a valid answer)
- **Niche/Industry** (infer from context if not stated)

Present the parsed list back to the user in a clean table and ask them to confirm before proceeding. If anything looks wrong, fix it.

If the user hasn't specified a niche, ask: "What industry are these businesses in?"

---

## Phase 2: Research & Visual Audit

For each business, conduct a full web presence audit. Use parallel agents when there are 3+ businesses to speed things up.

### Step 1: Full Site Crawl (all pages, not just homepage)

**If Apify MCP is available**, crawl the ENTIRE website using `apify/website-content-crawler`:

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

- `crawlerType: "playwright:firefox"` — handles Wix/Squarespace/WordPress JS-rendered sites
- `maxCrawlDepth: 3` — covers most small business sites (typically 5-20 pages)
- `maxCrawlPages: 25` — safety cap to prevent runaway costs

**Output per page:** `url`, `title`, `text`, `markdown`, `metadata` — every page's content in one pass.

**If Apify MCP is NOT available (fallback):**
1. WebFetch the homepage
2. Parse nav/footer links to discover internal pages
3. WebFetch up to 10 additional internal pages
4. Flag to user: "Full crawl unavailable — using nav-link discovery (less complete)"

### Step 2: Screenshot Capture (run IN PARALLEL with Step 1)

Capture screenshots via Playwright CLI while the crawl runs:

1. **Desktop homepage:**
   ```bash
   npx playwright screenshot --full-page --viewport-size="1440,900" [URL] "[output-path]/desktop.png"
   ```
2. **Mobile homepage:**
   ```bash
   npx playwright screenshot --full-page --viewport-size="390,844" [URL] "[output-path]/mobile.png"
   ```
3. **1-2 key inner pages** (services page, about page — pick from nav):
   ```bash
   npx playwright screenshot --full-page --viewport-size="1440,900" [inner-URL] "[output-path]/inner-[slug].png"
   ```
4. Save screenshots under `generated-sites/[business-name]/audit/`

**Fallback (no Playwright):** Skip screenshots, rely on crawl content + WebFetch. Flag visual scoring as less accurate.

### Step 3: Analyze ALL Pages

With full crawl data + screenshots, produce a comprehensive audit:

**Content Analysis (from crawl markdown — covers every page):**
- Total page count and site structure
- Services listed (extract from services/menu pages)
- About page presence and owner info
- Contact page — has form? Has booking/scheduling?
- Testimonials/reviews found on any page
- Content depth per page (thin < 100 words vs substantial)
- Gallery/portfolio pages with work photos
- Blog presence and last update date
- Service area listed?

**Visual Analysis (from screenshots):**
- Modern or outdated design?
- Mobile responsive?
- Clear CTAs?
- Professional appearance?
- Broken elements visible?

**Page-by-Page Summary:**
```
Pages found: X total
Homepage: [brief assessment]
Services: [brief assessment or "Missing"]
About: [brief assessment or "Missing"]
Contact: [brief assessment or "Missing"]
Gallery: [brief assessment or "Missing"]
Blog: [brief assessment, last post date, or "Missing"]
Other: [list any other pages found]
```

### If the business has NO website:
- Search the web for their business name + city to check for:
  - Google Business Profile
  - Facebook page
  - Yelp listing
  - Any other web presence
- Score them as 1/10 across all visual/UX categories
- Flag as "No Website — High Priority" for generation

### Asset Extraction (from crawl data — scans ALL pages, not just homepage)

For every business WITH a website, extract reusable assets:

**Logo Extraction (priority order):**
1. **Clearbit API** (always works, no scraping): `https://logo.clearbit.com/{domain}` → save URL
2. **JSON-LD structured data**: Parse `<script type="application/ld+json">` for `"logo"` field
3. **og:image meta tag**: `<meta property="og:image">` — almost always branded
4. **DOM header parsing**: `<img>` inside `<header>` or `<nav>` where alt/class/id/src contains "logo"
5. **Favicon fallback**: `<link rel="apple-touch-icon">` (clean 180x180 logo) or Google favicon service: `https://www.google.com/s2/favicons?domain={domain}&sz=128`

**Work Photos Extraction (scan ALL crawled pages):**
- Scan for images in `.gallery`, `.portfolio`, `.before-after`, `.our-work`, `.projects` containers
- Check gallery/portfolio pages specifically (common on contractor sites)
- Check `data-src`, `data-lazy-src`, `data-original` attributes (lazy-loaded images)
- Extract CSS `background-image` URLs from hero/gallery sections
- Filter out icons (<30px), stock badges, and social media icons
- Keep up to 8 best work photos per business

**Brand Color Extraction:**
- Parse inline styles and CSS for dominant colors
- Check og:theme-color meta tag
- Note primary, secondary, and accent colors

**Testimonial Extraction (scan ALL pages):**
- Look for Schema.org Review structured data on any page
- Parse `.testimonial`, `.review`, `.quote` containers
- Check dedicated reviews/testimonials page if it exists
- Extract reviewer name, text, rating if available

**Service List Extraction (from services pages):**
- Parse service page headings and descriptions
- Extract pricing if listed
- Note which services are prominently featured vs buried

### Customer Persona Analysis (MANDATORY — do this for EVERY business)

**Why this exists:** A pressure washing company that also does window cleaning, gutter cleaning, and garbage can cleaning serves HOMEOWNERS who want a clean, beautiful home — not just "pressure washing." The generated site must reflect ALL their services and show results from the CUSTOMER'S perspective.

**Step 1 — Identify ALL services (not just the primary one):**
From the full site crawl, extract every service the business offers. Don't stop at the business name or niche label. A "pressure washing" company might also do:
- Window cleaning, gutter cleaning, roof washing, garbage can cleaning, deck staining, etc.

A "plumber" might also do water heater installation, bathroom remodeling, drain cleaning, etc.

**Step 2 — Identify the customer:**
Ask: "Who is paying this business money?" The answer determines imagery and messaging.

| Business Type | Primary Customer | What They Care About |
|--------------|-----------------|---------------------|
| Exterior cleaning | Homeowners | Beautiful home, curb appeal, clean surfaces, pride of ownership |
| Plumbing | Homeowners / property managers | Working fixtures, no leaks, peace of mind |
| Landscaping | Homeowners / commercial property | Beautiful yard, neat appearance, property value |
| Restaurant | Diners / families | Food quality, ambiance, convenience |
| Auto repair | Car owners | Reliability, fair pricing, trust |
| Salon/spa | Individuals | Looking/feeling good, relaxation |

**Step 3 — Map services to imagery:**
For EACH service the business offers, identify what photo would represent it from the CUSTOMER'S perspective (showing the RESULT, not the worker doing the job):

| Service | Bad Photo | Good Photo |
|---------|-----------|------------|
| Pressure washing | Worker spraying a driveway | Clean, pristine driveway or walkway |
| Window cleaning | Worker on a ladder with squeegee | Sparkling clean windows on a beautiful home |
| Gutter cleaning | Worker scooping gunk from gutters | Clean gutters with water flowing properly |
| Garbage can cleaning | Dirty trash cans | Clean, sanitized trash cans |
| General exterior | Random stock photo | Beautiful, well-maintained home exterior |

**Step 4 — Save persona data in assets.json:**
Add to the `assets.json` output:
```json
{
  "customer_persona": {
    "primary_customer": "homeowners",
    "customer_cares_about": ["curb appeal", "clean home", "property value", "pride of ownership"],
    "all_services": ["Pressure Washing", "Window Cleaning", "Gutter Cleaning", "Garbage Can Cleaning"],
    "service_photo_needs": {
      "Pressure Washing": "clean driveway or patio",
      "Window Cleaning": "sparkling windows on home exterior",
      "Gutter Cleaning": "clean gutters, well-maintained roofline",
      "Garbage Can Cleaning": "clean sanitized trash cans"
    },
    "imagery_perspective": "Show RESULTS the customer gets, not workers doing the job. The customer wants to see their home looking beautiful."
  }
}
```

**CRITICAL RULES for imagery selection:**
1. **MATCH THE BUSINESS'S OWN VISUAL VIBE** — This is the #1 rule. NEVER search for stock photos by generic niche keyword (e.g. "plumbing", "roadside assistance", "mechanic"). Instead, study the business's actual website screenshots from the audit and identify what they PORTRAY — their imagery style, the types of visuals they use, their aesthetic vibe. Then search for stock photos matching THAT specific vibe. Examples:
   - HURT Roadside's site shows dark SUVs, sports cars, night city scenes → search "dark car city night tail lights", "SUV headlights urban" — NOT "roadside assistance mechanic"
   - ABC Plumbing's site shows clean bathrooms, chrome fixtures → search "modern bathroom shower", "chrome faucet closeup" — NOT "plumber working"
   - A pressure washing site shows before/after driveways → search "clean concrete driveway", "suburban home exterior" — NOT "man with pressure washer"
2. **Photos must represent ALL services** — not just the one in the business name. If they do 4 services, show photos for all 4.
3. **Show the customer's perspective** — what the customer GETS (clean home, sparkling windows), not what the worker DOES (spraying, scrubbing).
4. **Use real business photos first** — from their website, Yelp, Google Business Profile. Only fall back to stock photos if the business has no usable photos.
5. **Stock photos must be accurate** — if you use a stock photo labeled "driveway cleaning," visually verify it actually shows a driveway, not a gas station or random location. Read/view the image before using it.
6. **Match the customer demographic** — if the customers are homeowners, show residential homes. If commercial, show commercial properties.
7. **Stock photos must look REAL and RELATABLE** — no luxury mansions, villas, or aspirational dream homes. Show NORMAL houses that look like actual customer homes in the business's service area. A pressure washing company in Waco, TX serves regular middle-class homeowners — show ranch-style houses, typical suburban homes, normal driveways. The photos should look like actual before/after work the company did, not a magazine photoshoot. Search for terms like "suburban house driveway" or "normal residential home exterior" not "luxury villa" or "beautiful mansion."
8. **Only include social links that actually exist** — check the business's website footer/contact page for their actual social media profiles. Don't add YouTube, Instagram, Twitter, etc. links unless the business actually has those accounts. When in doubt, only include links you can verify from the crawl data.
9. **Verify every photo visually** — before using ANY stock photo, screenshot or view the actual image to confirm it matches what you think it shows. Unsplash descriptions can be misleading (e.g., "man cleaning driveway" might actually be a gas station). When in doubt, find a different photo.

Save extracted assets to `[business-slug]/audit/assets.json`:
```json
{
  "logo_url": "https://logo.clearbit.com/example.com",
  "logo_source": "clearbit",
  "og_image": "https://example.com/hero.jpg",
  "work_photos": ["url1", "url2"],
  "brand_colors": { "primary": "#2d6a1e", "secondary": "#7bc142", "accent": "#f5a623" },
  "testimonials": [{ "name": "John D.", "text": "Great work!", "rating": 5 }],
  "services": ["Pressure Washing", "Soft Washing", "Window Cleaning"],
  "pages_found": 12,
  "pages_crawled": ["url1", "url2"],
  "has_blog": false,
  "has_gallery": true,
  "has_contact_form": true,
  "last_blog_post_date": null
}
```

### Scoring Rubric

Score each business 1-10 in four categories:

**Visual Design (1-10)**
- 1-3: Outdated look, clipart/stock imagery, no consistent branding, feels like 2005
- 4-6: Generic template, decent but forgettable, stock photos everywhere
- 7-10: Clean, modern, professional, unique brand identity, high-quality imagery

**UX & Usability (1-10)**
- 1-3: Confusing navigation, broken elements, no clear CTAs, frustrating to use
- 4-6: Functional but clunky, CTAs exist but aren't compelling, average flow
- 7-10: Intuitive navigation, clear CTAs, smooth user journey, everything works

**Content Quality (1-10)**
- 1-3: Thin or missing content, no service descriptions, spelling errors
- 4-6: Basic info present but generic, reads like a template was filled in
- 7-10: Compelling copy, clear service descriptions, unique voice, strong value props

**Mobile Experience (1-10)**
- 1-3: Not responsive, impossible to use on phone, text too small
- 4-6: Partially responsive, usable but awkward on mobile
- 7-10: Fully responsive, mobile-first feel, touch-friendly, fast loading

### Audit Output Format

For each business, produce:

```
## [Business Name] — Overall: X/10
- Visual: X/10
- UX: X/10
- Content: X/10
- Mobile: X/10

**Website:** [URL or "None"]
**Key Issues:** [2-3 bullet points on biggest problems]
**Verdict:** [One of: "No Website", "Needs Full Rebuild", "Needs Improvement", "Decent", "Strong"]
```

---

## Phase 3: Rank & Triage

1. Sort all businesses by overall score (lowest first)
2. Group them:
   - **No Website** — no site at all
   - **Needs Full Rebuild** (score 1-3) — site exists but is actively hurting them
   - **Needs Improvement** (score 4-6) — functional but not competitive
   - **Decent/Strong** (score 7+) — skip these for generation
3. Present the ranked report to the user
4. Ask: "Which businesses do you want me to build landing pages for?" — let the user choose

---

## Phase 4: Niche Research

Before generating any pages, research what "great" looks like in the user's niche.

### Research Tasks (use parallel agents):

**Agent 1 — Top Competitor Sites:**
- Web search: "best [niche] websites" and "top [niche] company website design"
- Identify 3-5 standout examples
- Note what makes them effective: layout, colors, CTAs, trust signals

**Agent 2 — Industry Design Patterns:**
- Web search: "best [niche] landing page design" and "[niche] website must-have features"
- Identify the must-have sections for this niche
- Note common color schemes, imagery styles, and conversion patterns

**Agent 3 — Local Market Context:**
- Web search: "[niche] in [city/region]" to understand local competition
- Check what the top local competitors look like
- Note any regional preferences or expectations

### Compile Design Brief

Synthesize the research into a design brief that includes:
- **Must-have sections** for this niche (e.g., for pressure washing: hero with before/after, services grid, trust badges, quote form)
- **Fallback color palette** based on industry norms — ONLY used when a business has no existing brand colors to extract. The business's own colors always take priority.
- **Trust signals** specific to the niche (certifications, badges, guarantees)
- **CTA strategy** (what the primary action should be — call, quote form, booking)
- **Content tone** (professional, casual, urgent, etc.)
- **Premium Tier Check**: Determine if this niche warrants the "Vast Space Aesthetic" (e.g., high-end tech, luxury contracting, modern architectural). If so, explicitly state in the brief to use the premium aerospace dark-mode design.

Read `references/niche-patterns.md` for baseline patterns across common niches.
Also read `references/vast-space-aesthetic.md` if the niche requires a hyper-premium, next-gen aerospace feel.

---

## Phase 5: Generate Landing Page + Site Plan

For each selected business, invoke the `frontend-design` skill.

### Pre-Generation: Visual Component Scouting (21st.dev Magic MCP)

If 21st.dev Magic MCP is available, do a **visual** component hunt BEFORE generating:

**Step 1 — Search for components by section type:**
Run multiple searches in parallel for each section needed:
```
mcp__magic__21st_magic_component_inspiration("hero section landing")
mcp__magic__21st_magic_component_inspiration("testimonial cards")
mcp__magic__21st_magic_component_inspiration("service cards grid")
mcp__magic__21st_magic_component_inspiration("contact form")
mcp__magic__21st_magic_component_inspiration("pricing section")
mcp__magic__21st_magic_component_inspiration("feature section bento")
```

**Step 2 — Visually preview the top results:**
Each component returned has a preview URL on 21st.dev. Screenshot the top 2-3 candidates to actually SEE them:
```bash
npx playwright screenshot --viewport-size="1440,900" "https://21st.dev/[component-url]" "[output-path]/inspiration-[name].png"
```
Read the screenshots with Claude Vision to evaluate:
- Does it look modern and premium?
- Would it work for this niche (e.g., a pressure washing company)?
- Does the layout structure match our design brief?
- Are there cool effects (gradients, glow, animations) worth adapting?

**Step 3 — Extract design DNA from the winner:**
From the best component's code, extract:
- **Layout pattern** (grid structure, flex alignment, spacing ratios)
- **Visual effects** (gradients, glow effects, backdrop blur, clip-paths, shadows)
- **Animation techniques** (hover states, entrance animations, parallax)
- **Typography scale** (heading sizes, weights, letter-spacing)
- **Color application** (how they use primary/secondary/accent)
- **Micro-interactions** (button hover effects, card transforms)

**Step 4 — Also search for logos:**
Use `mcp__magic__logo_search` if the business is a known brand or franchise.

**IMPORTANT: Do NOT copy React components directly** — they require JSX/build tools. Instead, extract the design DNA and rebuild in single-file HTML. Use Tailwind CDN for utility classes:
```html
<script src="https://cdn.tailwindcss.com"></script>
```

### Hero Section Design Standard

The hero is the first thing the business owner sees — it MUST be impressive. Apply these techniques from 21st.dev patterns:

**Visual techniques to use (pick 2-3 per hero):**
- **Gradient text** — `background: linear-gradient(...); -webkit-background-clip: text;` on the headline
- **Glow effect** — radial gradient blob behind the CTA or hero image
- **Dot grid / noise background** — subtle pattern behind the hero content
- **Animated badge/pill** — "Waco's #1 Rated" with subtle shimmer animation
- **Split layout** — text left, angled/clipped hero image right with overlay
- **Floating elements** — trust badges or stats that float with subtle parallax
- **Glass morphism** — frosted glass cards over a vivid background image
- **Gradient border buttons** — CTAs with animated gradient borders

**Hero layouts to rotate between (never use the same one twice in a batch):**
1. **Centered dramatic** — huge headline, gradient text, glow CTA, full-width background
2. **Split asymmetric** — text left 55%, hero image right 45% with clip-path diagonal
3. **Overlay dark** — full-bleed hero image with dark gradient overlay, white text
4. **Bento hero** — headline left, right side has 2-3 bento grid cards with stats/photos
5. **Video-style** — hero image with play button overlay, stats bar underneath
6. **Aerospace Premium (Vast Space style)** — dark mode, massive geometric typography (Inter or space sans), high-contrast white text, smooth slow blur-reveals, and glassmorphism. Use for high-end or tech-adjacent niches.

### Pre-Generation: Asset Integration

Pull in extracted assets from Phase 2 (`audit/assets.json`):

- **Logo**: Use extracted logo (Clearbit or scraped). Place in header and footer.
- **Work Photos**: Use real business photos in gallery/portfolio section. If they have good hero-quality photos, use those for the hero background instead of Unsplash.
- **Brand Colors**: **ALWAYS use the business's own brand colors.** Extract primary, secondary, and accent from their existing site/logo. These are THEIR colors — the generated site must feel like THEIR brand, not a generic template. Only fall back to niche-appropriate colors from the design brief if the business has zero web presence and no extractable colors.
- **Testimonials**: If extracted, use real testimonials with attribution. If not, use generic placeholders marked `<!-- PLACEHOLDER: Replace with real testimonials -->`.

**Unsplash hero images**: Only use Unsplash for the hero if the business has no high-quality photos of their own. Search via Apify RAG browser: `"unsplash [niche] [specific scene]"` to get CDN URLs. Never hotlink non-Unsplash stock sites.

### Pre-Generation: Motion Effects with Nano Banana Pro (nanobanana MCP)

**Purpose:** Take static business photos and create motion/animation assets — NOT generate new images. Make static photos feel alive.

**When to use:** Business has 2+ good photos from asset extraction.

**IMPORTANT — Image Size Prep:** Gemini API has a ~4MB base image limit. Before sending ANY image to nanobanana, resize it:
```bash
# Resize to max 1920px wide, compress to <3MB
npx sharp-cli --input photo.jpg --output photo-ready.jpg resize 1920 --withoutEnlargement
# OR with curl + tinypng if sharp unavailable
```
Always use the resized version as input, never raw full-res.

**Motion techniques:**
1. **Dual-Image Crossfade Hero** — 2 work photos → generate blended intermediate → CSS crossfade (8s cycle)
2. **Parallax Layer Separation** — Prompt: "foreground subject on transparent background" → original bg + foreground parallax overlay
3. **Before/After Morph** — Generate 2-3 intermediate blend frames → CSS morph on scroll/hover
4. **Subtle Motion Loops** — Prompt: "slightly different angle/lighting" → slow 8s crossfade loop
5. **Color-Shifted Variants** — Time-of-day variants (dawn/midday/dusk) → slow ambient cycle

**Rules:**
- Only on business's OWN photos — never stock/Unsplash
- ALWAYS resize to <3MB before sending to nanobanana
- Subtle animations (8+ second cycles)
- Reduced motion fallback: `@media (prefers-reduced-motion: reduce) { .hero-frame { animation: none; } }`
- Save to `[business-slug]/generated-site/assets/motion/`
- No nanobanana? CSS-only ken-burns fallback: `@keyframes kenBurns { 0% { transform: scale(1); } 50% { transform: scale(1.08) translate(-1%,-1%); } 100% { transform: scale(1); } }`

**Nanobanana Rate Limit Handling (IMPORTANT):**

Google AI Studio free tier has aggressive rate limits (~10 requests/minute, ~50/hour). Multiple failed MCP connection attempts also burn quota. Follow these rules:

1. **Queue requests sequentially** — never fire multiple nanobanana calls in parallel. Process one image at a time.
2. **Wait between requests** — add a 10-second pause between nanobanana API calls: `sleep 10` between each call.
3. **Limit total calls per session** — max 5 nanobanana image operations per business. Prioritize the hero image and 1-2 key gallery images.
4. **Graceful degradation on 429** — if you get a rate limit error (HTTP 429 or "quota exceeded"):
   - Do NOT retry immediately
   - Log which images still need processing
   - Fall back to CSS-only effects (ken-burns, crossfade between static images)
   - Tell the user: "Nanobanana rate-limited — using CSS-only motion effects. You can re-run the motion enhancement later."
5. **Cache results** — save all nanobanana outputs to `assets/motion/`. Never re-process an image that already has a motion version saved.
6. **Pre-check before calling** — before making any nanobanana call, check if the output file already exists in `assets/motion/`. Skip if it does.

**Direct Gemini API Fallback (if MCP server is down or rate-limited):**

If the nanobanana MCP server fails to connect or is rate-limited, use the Gemini API directly via curl/Python. The `gemini-2.0-flash-exp-image-generation` model has **separate quota** from other Gemini models and is proven to work:

```bash
curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-exp-image-generation:generateContent?key=YOUR_GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{"parts": [{"text": "YOUR_PROMPT_HERE"}]}],
    "generationConfig": {"responseModalities": ["TEXT", "IMAGE"]}
  }' | python3 -c "
import sys, json, base64
data = json.load(sys.stdin)
parts = data.get('candidates', [{}])[0].get('content', {}).get('parts', [])
for p in parts:
    if 'inlineData' in p:
        img = base64.b64decode(p['inlineData']['data'])
        with open('OUTPUT_PATH.png', 'wb') as f:
            f.write(img)
        print(f'Saved ({len(img)} bytes)')
"
```

- Use 12-second delays between requests to avoid 429s
- On 429, wait 30s then retry once
- This approach bypasses the MCP server entirely — useful when the MCP has connection issues

**Niche-Agnostic Stock Photo + Nanobanana Workflow:**
This workflow applies to ANY niche, not just pressure washing:
1. Identify what photos are needed based on customer persona analysis (Step 3 above)
2. Search for high-quality stock photos that match each service: `"unsplash [service] [result] residential"` — prioritize RESULTS over action shots, and NORMAL homes over luxury
3. Download the best matches locally to `assets/`
4. Resize each to <3MB with sharp-cli
5. Feed to nanobanana one at a time (with 10s delays) for motion effects
6. If rate-limited, fall back to CSS ken-burns on the static photos
7. Use the motion-enhanced versions in the hero crossfade and gallery

### Inputs to frontend-design:
- Business name, phone, location
- Services offered (from audit or user input)
- The niche design brief from Phase 4
- Extracted assets (logo, photos, colors, testimonials) from Phase 2
- 21st.dev design patterns to reference (if available)
- Scroll transition assignments from `references/scroll-transitions.md`
- Instruction: "Create a single-page landing page that is mobile-first, conversion-optimized, uses unique scroll transitions per section, and follows the design brief"

### Required Landing Page Sections:
These come from the design brief but at minimum every generated page needs:
1. **Hero** — headline, subheadline, primary CTA, phone number. Use Parallax Float or Clip Reveal transition.
2. **Trust bar** — ratings, years in business, certifications. Use Rise & Spread transition.
3. **Services** — grid or cards showing what they do. Use Cascade Stagger or Tilt-In transition.
4. **Social proof** — testimonials or review excerpts. Use Blur Unblur or Scale Pop transition.
5. **About** — brief company/owner story. Use Horizontal Slide-In (alternating) transition.
6. **Gallery** — work photos (real if available). Use Cascade Stagger with different timing.
7. **Service area** — cities/neighborhoods served. Use Counter Slide Split transition.
8. **Contact/CTA** — form + phone number + clear call to action. Use Glowing Entrance or Morph Border transition.
9. **Footer** — full business info. Simple fade, no fancy transition.

**Section Dividers:** Use DIFFERENT dividers between each section. Rotate through: wave → diagonal → curve → mountain → arrow. Never repeat the same divider consecutively.

Read `references/scroll-transitions.md` for the full transition library, CSS patterns, and the master IntersectionObserver script to include at the bottom of every page.

### Uniqueness Rule

**Every generated site MUST be structurally distinct from other sites in the same batch.** Vary the LAYOUT and STRUCTURE — NOT the brand colors. Colors come from the business's brand, period.

**What to vary (structure/layout/interaction):**
- Hero layout (centered dramatic vs split asymmetric vs overlay dark vs bento vs video-style)
- Typography pairing (rotate through the 8 combos in niche-patterns.md)
- Button style (pill vs rounded-rect vs sharp corners vs ghost outline)
- Section divider shapes (wave vs diagonal vs curve vs mountain vs arrow)
- Card styles (shadow cards vs bordered vs left-accent vs gradient overlay)
- Transition assignments (rotate which scroll transitions go with which sections)
- Section ordering (vary which sections come in what order after hero)
- Visual effects (gradient text vs glow vs glass morphism vs floating elements)

**What NEVER changes based on "uniqueness" — always driven by the business's brand:**
- Color palette — comes from extracted brand colors or their existing site
- Logo — always their actual logo
- Business photos — always their real work photos when available
- Testimonials — always their real reviews when available
- Service descriptions — always their actual services

### Full Site Plan:
In addition to the landing page, produce a site plan document:
```
## Site Plan: [Business Name]

### Recommended Pages
1. Home (the landing page we just built)
2. [Service Page 1]
3. [Service Page 2]
4. ...
5. About Us
6. Service Areas (with sub-pages per city)
7. Gallery / Portfolio
8. Reviews
9. Contact
10. Blog (for SEO)

### Content Outline
[Brief description of what each page should contain]

### SEO Strategy
[Target keywords, local SEO approach]
```

---

## Phase 6: Deploy

### Deployment

**Netlify CLI (preferred — proven to work in VS Code IDE):**
1. Set the auth token: `NETLIFY_AUTH_TOKEN` env var (from `~/AppData/Roaming/netlify/Config/config.json`)
2. Deploy:
   ```bash
   NETLIFY_AUTH_TOKEN="[token]" npx --yes netlify-cli deploy --dir="[path-to-site-folder]" --prod --site="[site-id]" --message="[Business Name] landing page"
   ```
3. If no site exists yet, create one first via API:
   ```bash
   curl -s -X POST "https://api.netlify.com/api/v1/sites" -H "Authorization: Bearer [token]" -H "Content-Type: application/json" -d '{"name": "[business-slug]-demo"}'
   ```
4. Capture the live URL (e.g., `https://business-name-demo.netlify.app`)

**Vercel (alternative — if Vercel MCP is available):**
1. Deploy to Vercel instead
2. Same flow as Netlify

**No deploy tool available (fallback):**
1. Save the HTML/CSS/JS files locally under `generated-sites/[business-name]/`
2. Tell the user: "Files are saved locally. Deploy by dragging to netlify.com/drop or vercel.com/new"

---

## Phase 7: Visual Verification (QA Pass)

**This phase is MANDATORY.** Every generated site must pass verification before outreach. Run this for EACH deployed site.

### 7a. HTML Validation

Read the generated `index.html` and check for:

| Check | What to Look For | Severity |
|-------|-----------------|----------|
| **Unclosed tags** | Every `<div>`, `<section>`, `<a>`, `<span>` has a matching close tag | CRITICAL |
| **Broken images** | Every `<img src="">` and `background-image: url()` points to a valid URL or local file | CRITICAL |
| **Missing alt text** | Every `<img>` has an `alt` attribute | WARN |
| **Broken links** | Every `<a href="">` points to a valid anchor (`#section`), `tel:`, `mailto:`, or URL | CRITICAL |
| **CSS syntax errors** | No unclosed brackets `{}`, missing semicolons, or invalid property values | CRITICAL |
| **JS syntax errors** | No unclosed parentheses, missing semicolons, undefined variables | CRITICAL |
| **Font loading** | Google Fonts `<link>` tags use valid family names and are in `<head>` | HIGH |
| **Form action** | Contact form `action` attribute points to a working endpoint (FormSubmit, Netlify Forms, etc.) | HIGH |
| **Schema markup** | JSON-LD structured data is valid JSON and uses correct Schema.org types | WARN |
| **Meta tags** | `<title>`, `<meta name="description">`, `<meta name="viewport">` all present | HIGH |

### 7b. Scroll Transition Audit

Verify scroll transitions won't cause visual bugs:

| Check | What to Look For | Fix |
|-------|-----------------|-----|
| **No opacity:0** | Initial states must use `opacity: 0.05` not `opacity: 0` | Change to 0.05 |
| **IntersectionObserver exists** | Master observer script is at bottom of page | Add from scroll-transitions.md |
| **Observer unobserves** | Each element is unobserved after triggering to prevent re-fire | Add `observer.unobserve(entry.target)` |
| **No duplicate transitions** | Adjacent sections don't use the same transition class | Swap one for an alternative |
| **Mobile safe** | No `translateX` values > 50px on mobile (causes horizontal scroll) | Add media query to reduce values |
| **No transform conflicts** | Elements with `position: fixed/sticky` don't have scroll transforms | Remove transform from fixed elements |
| **Stagger delay cap** | Stagger delays don't exceed 1.5s total (feels broken if too slow) | Cap at max 8 items × 100ms |
| **Animation speed** | All scroll animation durations must be 1.0s-1.4s (NOT 0.5-0.8s — those feel too fast/jarring) | Increase to 1.0-1.4s range |
| **Observer trigger point** | IntersectionObserver `threshold` should be 0.1 with `rootMargin: '0px 0px -20px 0px'` — triggers early so animations have time to complete smoothly | Adjust threshold and rootMargin |

**Animation Duration Defaults (MANDATORY):**
- fadeUp: `1.2s ease`
- blurReveal: `1.4s ease`
- scalePop: `1.0s cubic-bezier(0.34, 1.56, 0.64, 1)`
- slideLeft / slideRight: `1.2s ease`
- counterSlide: `1.3s ease`
- cascadeStagger: `1.0s ease` per item, 100ms delay between items
- Hero crossfade: `2s ease-in-out` opacity transition, 8s per frame
- Ken Burns: `20s ease-in-out infinite`

### 7c. Responsive Layout Check

Scan CSS for common responsive bugs:

| Check | What to Look For | Fix |
|-------|-----------------|-----|
| **Viewport meta** | `<meta name="viewport" content="width=device-width, initial-scale=1">` present | Add if missing |
| **Horizontal overflow** | No elements wider than viewport (100vw without accounting for scrollbar) | Use `max-width: 100%` or `overflow-x: hidden` on body |
| **Mobile nav** | `.mobile-menu` or `.mobile-nav` has `display: none` on desktop BEFORE media query | Add explicit desktop hide |
| **Touch targets** | CTA buttons are at least 44px tall on mobile | Increase padding |
| **Font sizes** | No text smaller than 14px on mobile | Increase in media query |
| **Image sizing** | Large images have `max-width: 100%; height: auto;` | Add if missing |
| **Grid/flex wrapping** | Grid/flex containers use `flex-wrap: wrap` or responsive `grid-template-columns` | Add wrapping |

### 7d. Visual Screenshot Verification (if Playwright available)

```bash
# Desktop
npx playwright screenshot --full-page --viewport-size="1440,900" "[LIVE_URL]" "[output-path]/verify-desktop.png"
# Mobile
npx playwright screenshot --full-page --viewport-size="390,844" "[LIVE_URL]" "[output-path]/verify-mobile.png"
```

Read the screenshots and check:
- Header renders correctly (logo visible, nav links present, no overlap)
- Hero section has visible content (not blank from opacity:0 bug)
- All sections have content (no empty/collapsed sections)
- Images are loading (no broken image icons)
- Mobile layout is single-column (no side-scroll)
- Footer has full business info visible
- No text overlapping other text
- No elements cut off at edges

### 7e. Verification Report

Produce a verification report for each site:

```
## Verification: [Business Name]
**URL:** [live URL]
**Status:** PASS / FAIL (with issue count)

### Results
- HTML Validation: X issues (Y critical)
- Scroll Transitions: X issues
- Responsive: X issues
- Visual (screenshot): PASS / FAIL / SKIPPED

### Issues Found
1. [CRITICAL] [description] → FIXED
2. [HIGH] [description] → FIXED
3. [WARN] [description] → noted, non-blocking

### Fixes Applied
- [what was fixed and how]
```

**If any CRITICAL issues are found:** Fix them immediately, redeploy, and re-verify. Do NOT proceed to outreach until all CRITICAL issues are resolved.

**If only WARN issues remain:** Proceed to outreach. Note the warnings in the report for future improvement.

---

## Phase 8: Outreach

### Outreach Drafts

For each deployed site, draft **two casual outreach messages** with the live link:

**Email Draft:**
- Subject line that's curiosity-driven, not salesy
- 3-4 sentences max
- Mention you noticed their business, compliment something specific
- Casually mention you put together a sample page
- Include the live link
- Soft CTA — "let me know what you think" energy, not "BUY NOW"

**SMS/DM Draft:**
- 2-3 sentences max
- Even more casual than email
- Same structure: noticed you → made this → check it out → lmk

**Tone guidelines:**
- Sound like a real person, not a marketing agency
- No corporate buzzwords ("leverage," "synergy," "optimize")
- Specific > generic (mention their actual business name, city, services)
- Friendly and helpful, not pushy
- Example energy: "Hey [Name], I was looking at pressure washing companies in Waco and came across [Business]. I actually put together a sample website for you — check it out: [link]. No strings attached, just thought it could help. Let me know what you think!"

---

## Presenting Final Results

After all phases complete, present a summary:

```
## Biz Site Scout — Results

### Audited: X businesses
### Generated: X landing pages
### Deployed: X live sites
### Verified: X passed / Y warnings

| Business | Score | Status | Verified | Live URL | Outreach |
|----------|-------|--------|----------|----------|----------|
| [Name]   | 2/10  | Deployed | PASS | [link] | Ready |
| [Name]   | N/A   | Deployed | PASS (2 warns) | [link] | Ready |

### Verification Summary
[Pass/fail status for each site with issue counts]

### Outreach Messages
[Collapsible sections with email + SMS drafts for each business]
```

---

## Adaptive Behavior

This skill adapts to available tools:

| Tool | Available | Fallback |
|------|-----------|----------|
| Apify MCP (`apify/website-content-crawler`) | Full site crawl + asset extraction | WebFetch homepage only |
| Playwright CLI | Visual audit + QA verification screenshots | Code-only verification |
| 21st.dev Magic MCP | Design inspiration, hero patterns, transitions | Generate from niche-patterns.md |
| Clearbit Logo API | Instant logo: `logo.clearbit.com/{domain}` | Scrape DOM / favicon |
| Netlify CLI (`npx netlify-cli deploy`) | Auto-deploy + live URL | Save files locally |
| Vercel MCP (alternative) | Auto-deploy + live URL | Save files locally |
| Memory MCP | Persist niche research across sessions | Re-research each time |
| Nano Banana MCP (`nanobanana`) | Motion effects from static photos (crossfade, parallax layers, morph) | CSS-only ken-burns / crossfade |
| sharp-cli (`npx sharp-cli`) | Resize images to <3MB before nanobanana | Skip motion effects |
| frontend-design skill | Full design generation with transitions | Build page directly |
| Web search | Competitor/niche research | Use knowledge + user input |

If a tool is missing, the skill still works — it just loses that capability and tells the user what they're missing.
