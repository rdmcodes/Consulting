# Lead Gen Skills Package

Two Claude Code skills that work together to find businesses with bad websites, build them a free sample landing page, deploy it, and draft outreach messages.

## What's Inside

```
skills/
├── lead-scout/           # Finds businesses via Google Maps/Yelp/Facebook, audits them, ranks by need
│   ├── SKILL.md
│   └── config/
│       └── defaults.json  # Settings: how many to auto-build, filters, etc.
├── biz-site-scout/       # Builds landing pages, deploys to Netlify, runs QA, drafts outreach
│   ├── SKILL.md
│   └── references/
│       ├── scroll-transitions.md   # 12 unique scroll animation patterns + CSS + JS
│       ├── niche-patterns.md       # Design patterns per industry (colors, sections, trust signals)
│       └── vast-space-aesthetic.md # Premium dark-mode aerospace design system
```

## Quick Start

### 1. Install the skills

Copy the `skills/` folders into your Claude Code skills directory:

```bash
# Mac/Linux
cp -r skills/lead-scout ~/.claude/skills/
cp -r skills/biz-site-scout ~/.claude/skills/

# Windows
xcopy /E /I skills\lead-scout %USERPROFILE%\.claude\skills\lead-scout
xcopy /E /I skills\biz-site-scout %USERPROFILE%\.claude\skills\biz-site-scout
```

### 2. Set up MCP servers

These skills need external tools connected via MCP. Run these commands:

**Apify (REQUIRED — scraping engine):**
```bash
claude mcp add --transport http apify https://mcp.apify.com --header "Authorization: Bearer YOUR_APIFY_API_KEY" --scope user
```
Get your API key at: https://console.apify.com/account/integrations

**21st.dev Magic (design inspiration — optional but recommended):**
```bash
claude mcp add magic --scope user -- npx -y @21st-dev/magic@latest
```
Then edit `~/.claude.json` and add your API key to the magic server's `env` block:
```json
"env": { "API_KEY": "YOUR_21ST_DEV_KEY" }
```
Get key at: https://21st.dev/settings

**Nano Banana (AI image effects — optional):**
```bash
claude mcp add nanobanana --scope user -- npx -y gemini-nanobanana-mcp@latest
```
Then edit `~/.claude.json` and add your Google AI key:
```json
"env": { "GEMINI_API_KEY": "YOUR_GOOGLE_AI_KEY" }
```
Get key at: https://aistudio.google.com/apikey (free tier available)

### 3. Install CLI tools

```bash
npm install -g sharp-cli    # Image resizing (keeps images under 4MB for Gemini)
npx playwright install      # Browser screenshots for visual auditing + QA
```

### 4. Netlify account (for deploying sites)

Sign up at https://app.netlify.com. The skills will use `npx netlify-cli` to deploy.
Your auth token is stored at: `~/AppData/Roaming/netlify/Config/config.json` (Windows) or `~/.config/netlify/Config/config.json` (Mac/Linux).

### 5. Reload Claude Code

After adding MCP servers, reload your IDE window:
- VS Code: `Ctrl+Shift+P` → "Reload Window"
- Terminal: restart Claude Code

## How to Use

**Find leads in any niche + city:**
```
/lead-scout "pressure washing" "Waco, TX"
```
Or just say: "find me plumbers in Austin that need a website"

**Audit specific businesses:**
```
/biz-site-scout
```
Then paste a list of business names/URLs.

## What Happens

1. **Scrapes** Google Maps, Yelp, and Facebook for businesses in your niche + area
2. **Crawls** each business's full website (all pages, not just homepage)
3. **Screenshots** desktop + mobile views for visual audit
4. **Scores** each business on design, UX, content, and mobile experience
5. **Ranks** them by how badly they need help
6. **Builds** a polished landing page for the worst ones (unique design per site)
7. **Deploys** to Netlify with a live URL
8. **Verifies** the deployed site (HTML validation, responsive check, screenshot QA)
9. **Drafts** casual outreach messages (email + SMS) with the live link

## Key Features

- **Niche-agnostic** — works for any business type
- **Brand-faithful** — extracts the business's actual colors, logo, and photos
- **Unique designs** — each site gets different layout, transitions, typography (never cookie-cutter)
- **12 scroll transitions** — each section gets a different animation
- **Motion effects** — AI-enhanced photo animations via Gemini (optional)
- **QA verification** — catches bugs before outreach
- **Direct Gemini API fallback** — if the MCP server has issues, falls back to direct API calls

## Without MCP Servers

The skills degrade gracefully. Without Apify, lead-scout can't run at all. Without the others:
- No 21st.dev → uses built-in design patterns from niche-patterns.md
- No nanobanana → uses CSS-only ken-burns/crossfade effects
- No Playwright → skips visual screenshots, does code-only verification
- No Netlify → saves files locally for manual deploy
