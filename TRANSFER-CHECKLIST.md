# Transfer Checklist — Files to Copy to New Repo

## Folder Structure for New Repo
```
your-new-repo/
├── CONSULTING-PLAYBOOK.md                    # Master guide (this is the "brain")
├── CLAUDE.md                                 # Claude Code instructions (create from template below)
├── skills/                                   # Copy these to ~/.claude/skills/ on each machine
│   ├── biz-site-scout/
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── scroll-transitions.md
│   │       ├── niche-patterns.md
│   │       └── vast-space-aesthetic.md
│   └── lead-scout/
│       ├── SKILL.md
│       └── config/
│           └── defaults.json
├── examples/                                 # Reference examples of completed work
│   ├── block-and-drum/
│   │   ├── BLOCK-AND-DRUM.md                 # Business research doc template
│   │   └── site/index.html                   # Example generated website
│   └── career-staffing/
│       └── research/NICHE-RESEARCH.md        # Example deep niche research
├── templates/
│   ├── BUSINESS-RESEARCH-TEMPLATE.md         # Blank template for new business docs
│   └── MEMORY-TEMPLATE.md                    # Template for project memory
└── clients/                                  # Active client work goes here
    └── [business-slug]/
        ├── BUSINESS-NAME.md
        ├── audit/
        │   ├── assets.json
        │   ├── desktop.png
        │   └── mobile.png
        └── site/
            ├── index.html
            └── [images]
```

---

## Exact Copy Commands (Run from Current Machine)

### 1. Core Skills (MUST COPY)
```bash
# These go to ~/.claude/skills/ on the partner's machine
cp -r ~/.claude/skills/biz-site-scout/ [NEW-REPO]/skills/biz-site-scout/
cp -r ~/.claude/skills/lead-scout/ [NEW-REPO]/skills/lead-scout/
```

**Source paths (Windows):**
- `C:\Users\smoor\.claude\skills\biz-site-scout\SKILL.md`
- `C:\Users\smoor\.claude\skills\biz-site-scout\references\scroll-transitions.md`
- `C:\Users\smoor\.claude\skills\biz-site-scout\references\niche-patterns.md`
- `C:\Users\smoor\.claude\skills\biz-site-scout\references\vast-space-aesthetic.md`
- `C:\Users\smoor\.claude\skills\lead-scout\SKILL.md`
- `C:\Users\smoor\.claude\skills\lead-scout\config\defaults.json`

### 2. Playbook (MUST COPY)
```bash
cp CONSULTING-PLAYBOOK.md [NEW-REPO]/
```

### 3. Reference Examples (SHOULD COPY)
```bash
cp "Block and drum/BLOCK-AND-DRUM.md" [NEW-REPO]/examples/block-and-drum/
cp "Block and drum/site/index.html" [NEW-REPO]/examples/block-and-drum/site/
cp "career-staffing/research/NICHE-RESEARCH.md" [NEW-REPO]/examples/career-staffing/research/
```

### 4. MCP Config (COPY STRUCTURE — partner uses their own API keys)
The partner needs to add these to their `~/.claude.json`:

```json
{
  "mcpServers": {
    "apify": {
      "type": "http",
      "url": "https://mcp.apify.com",
      "headers": {
        "Authorization": "Bearer THEIR_APIFY_API_KEY"
      }
    },
    "magic": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@21st-dev/magic@latest"],
      "env": {
        "API_KEY": "THEIR_21ST_DEV_API_KEY"
      }
    },
    "nanobanana": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "gemini-nanobanana-mcp@latest"],
      "env": {
        "GEMINI_API_KEY": "THEIR_GEMINI_API_KEY"
      }
    }
  }
}
```

### 5. NPM Packages (Partner Must Install)
```bash
npm install -g sharp-cli
npx playwright install
```

### 6. Accounts Partner Needs
- [ ] Apify account (https://console.apify.com)
- [ ] Netlify account (https://app.netlify.com)
- [ ] 21st.dev account (https://21st.dev/settings)
- [ ] Google AI Studio API key (https://aistudio.google.com/apikey)

---

## CLAUDE.md Template for New Repo

Create this file at the root of your new repo:

```markdown
# Consulting Company — Claude Code Instructions

## What We Do
Digital consulting for local businesses. We audit web presence, build demo websites, and offer marketing/automation services.

## Skills Available
- **biz-site-scout**: Audit business websites, generate replacements, deploy, verify, draft outreach
- **lead-scout**: Find businesses via scraping, rank by need, auto-build for top prospects

## Pipeline
Intake → Research & Audit → Rank → Niche Research → Generate Website → Deploy → QA Verify → Outreach

## Key Rules
- Read CONSULTING-PLAYBOOK.md for the full methodology
- NEVER search stock photos by generic niche keyword — match the business's actual vibe
- ALWAYS use the business's own brand colors (never generic)
- Use opacity: 0.05 (not 0) for scroll animation initial states
- QA verify EVERY site before sharing (Phase 7 is mandatory)
- Outreach tone: casual, human, specific — not corporate or salesy
- Don't ask unnecessary questions — work autonomously

## Deployment
- Netlify CLI for hosting
- FormSubmit.co for contact forms (no backend needed)

## Working Directory Structure
- `clients/[business-slug]/` — active client work
- `examples/` — reference examples (Block & Drum, Career Staffing)
- `skills/` — skill files (also install to ~/.claude/skills/)
```

---

## Verification — After Transfer

Run this checklist to confirm the new repo is set up correctly:

- [ ] `~/.claude/skills/biz-site-scout/SKILL.md` exists
- [ ] `~/.claude/skills/lead-scout/SKILL.md` exists
- [ ] `~/.claude/skills/biz-site-scout/references/` has 3 files
- [ ] MCP servers configured in `~/.claude.json` (Apify, Magic, Nanobanana)
- [ ] `npm list -g sharp-cli` returns a version
- [ ] `npx playwright --version` returns a version
- [ ] Netlify CLI works: `npx netlify-cli status`
- [ ] CONSULTING-PLAYBOOK.md is in repo root
- [ ] CLAUDE.md is in repo root
- [ ] Examples folder has Block & Drum reference docs
- [ ] Test: Ask Claude "Run biz-site-scout for [any business]" — it should recognize the skill
