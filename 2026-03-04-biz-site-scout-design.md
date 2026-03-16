# Biz Site Scout — Design Document

**Date:** 2026-03-04
**Status:** Approved
**Skill Location:** `~/.claude/skills/biz-site-scout/`

## Purpose

A Claude Code skill that takes a list of businesses (any niche, any format), audits their web presence, ranks them, and for the weakest ones: generates a polished landing page, deploys it to Vercel, and drafts casual outreach messages with the live link.

## Pipeline

### Phase 1 — Intake & Parse
- Accept business list in any format (CSV, JSON, pasted text, spreadsheet)
- Extract: business name, address/city, phone, website URL (if any), niche/industry
- Confirm parsed list with user before proceeding

### Phase 2 — Research & Visual Audit
- For each business:
  - Web search to find their site (or confirm no site exists)
  - Use Playwright MCP to capture screenshots (desktop + mobile viewport)
  - Feed screenshots to Claude vision for design quality scoring
  - Evaluate: visual design, UX/usability, content quality, mobile-friendliness
- Score each business on a 1-10 rubric across 4 categories
- Use parallel agents when processing multiple businesses

### Phase 3 — Rank & Triage
- Sort businesses by overall score
- Flag: no website, score below threshold (4/10), or major gaps
- Present ranked report to user
- User picks which businesses to build for

### Phase 4 — Niche Research
- Web search for top-performing websites in the specific niche + region
- Analyze design patterns: layouts, colors, trust signals, CTAs
- Compile a "design brief" that feeds into generation phase

### Phase 5 — Generate Landing Page + Site Plan
- Invoke `frontend-design` skill for each selected business
- Inputs: business info, niche research design brief, audit gaps to fix
- Outputs:
  - One polished landing page (HTML/CSS/JS)
  - Full site plan (sitemap, page descriptions, content outline)

### Phase 6 — Deploy & Outreach
- Deploy each landing page to Vercel (via MCP or CLI)
- Capture live URL
- Draft casual outreach messages for 2 channels (email + SMS/DM)
- Present: deployed URLs + draft messages, ready to send

## Scoring Rubric

| Category | What it evaluates | 1-3 (Bad) | 4-6 (OK) | 7-10 (Good) |
|----------|------------------|-----------|-----------|-------------|
| Visual | Modern design, branding, imagery | Outdated, clipart, no branding | Decent but generic template | Clean, professional, unique |
| UX | Navigation, mobile, CTAs | Broken/confusing, not mobile-friendly | Functional but clunky | Smooth, intuitive, mobile-first |
| Content | Copy, service descriptions | Thin/missing content | Basic info present | Compelling copy, clear services |
| Mobile | Responsive, touch-friendly | Not responsive at all | Partially responsive | Fully responsive, mobile-first |

## Tool Dependencies

### Required
- Web search (built-in)
- WebFetch (built-in)
- `frontend-design` skill (for page generation)

### Recommended MCPs (skill adapts to what's available)
- **Playwright MCP** — screenshot capture for visual audit
- **Vercel MCP** — auto-deploy generated pages
- **Memory MCP** — persist client/niche data across sessions

### Fallback Behavior
- No Playwright → use WebFetch for text-based analysis, skip visual screenshots
- No Vercel → save HTML files locally, user deploys manually
- No Memory → works fine, just doesn't persist across sessions

## Design Decisions

1. **Niche-agnostic** — works for any business type, not just pressure washing
2. **Format-agnostic input** — accepts whatever the user gives
3. **Visual-first audit** — Claude vision on screenshots is the core scorer, not Lighthouse/axe-core technical scores (business owners don't care about those)
4. **Modular deployment** — uses `frontend-design` skill for generation, Vercel MCP for deploy
5. **Casual outreach tone** — messages sound human, not corporate
