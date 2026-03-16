# Niche Design Patterns Reference

Baseline patterns for common local service niches. The skill should use these as starting points, then override with live research from Phase 4.

---

## Home Services (Pressure Washing, Plumbing, HVAC, Electrical, Roofing, Landscaping)

### Must-Have Sections
- Hero with action shot + phone number + "Get Free Quote" CTA
- Trust bar: Licensed & Insured, years in business, star rating, review count
- Services grid (4-6 cards with icons)
- Before/after gallery (especially for pressure washing, painting, landscaping)
- "How It Works" 3-step process (Request Quote → We Show Up → Job Done)
- Testimonials with star ratings and city names
- Service area list/map
- FAQ accordion
- Contact form (Name, Phone, Address, Service Needed)
- Sticky mobile CTA bar with click-to-call

### Color Palettes
- **Blue + White + Orange CTA** — trust, cleanliness, water (especially pressure washing, plumbing)
- **Green + White + Amber CTA** — eco-friendly, outdoors (landscaping, lawn care)
- **Red/Orange + Dark + White** — energy, urgency (HVAC, emergency services)
- **Navy + Gold + White** — premium feel (high-end remodeling, roofing)

### Trust Signals
- Google Reviews widget/rating
- BBB badge
- "Licensed & Insured" shield
- Angi / HomeAdvisor / Thumbtack badges
- Satisfaction guarantee badge
- "Locally Owned & Operated"
- Specific certifications (PWNA for pressure washing, NATE for HVAC, etc.)

### CTA Strategy
- Primary: "Get Your Free Quote" or "Call Now"
- Phone number in header, always visible
- Click-to-call on mobile
- Every section ends with a CTA
- Form: keep it short (4-5 fields max)

### Content Tone
- Friendly, confident, local
- "We" language, not corporate "the company"
- Mention the city/region naturally
- Address common concerns upfront (pricing, insurance, timing)

---

## Restaurants & Food Service

### Must-Have Sections
- Hero with food photography or interior ambiance shot
- Menu (or link to menu PDF)
- Hours & Location with embedded map
- Online ordering / reservation CTA
- Photo gallery (food, interior, events)
- Reviews/testimonials
- About / story section
- Contact info

### Color Palettes
- **Dark + Gold/Amber** — upscale, warm
- **Red + Cream + Dark** — appetite-stimulating, classic
- **White + Green + Natural** — healthy, fresh (salads, juice bars)
- **Black + White + Accent** — modern, trendy

### Trust Signals
- Yelp/Google rating
- "Locally sourced" / "Family recipes"
- Awards or press mentions
- Health inspection score (if good)

---

## Beauty & Personal Care (Salons, Barbers, Spas, Nail Salons)

### Must-Have Sections
- Hero with interior/service photography
- Services & pricing menu
- Online booking CTA (primary action)
- Before/after gallery (hair, nails, etc.)
- Team/stylist profiles
- Reviews
- Location & hours
- Instagram feed integration

### Color Palettes
- **Blush/Rose + White + Gold** — feminine, luxury (salons, spas)
- **Black + White + Gold** — modern, high-end (barbers, upscale salons)
- **Soft Purple + White + Silver** — relaxation (spas, wellness)
- **Bold colors** — trendy, youthful (nail art studios)

### Trust Signals
- Licensed cosmetologists/barbers
- Years of experience
- Instagram follower count
- "As seen in" press mentions

---

## Automotive (Mechanics, Detailing, Towing, Body Shops)

### Must-Have Sections
- Hero with shop/service imagery
- Services list with descriptions
- "Why Choose Us" differentiators
- Reviews/testimonials
- Certifications (ASE, etc.)
- Location & hours
- Contact/appointment form
- Fleet/commercial services (if applicable)

### Color Palettes
- **Red + Dark Gray + White** — energy, automotive classic
- **Blue + White + Dark** — trust, reliability
- **Black + Yellow + White** — industrial, bold (towing, body shops)

### Trust Signals
- ASE Certified
- Warranty on repairs
- "Family-owned since [year]"
- BBB rating
- Dealer-alternative messaging

---

## Professional Services (Law, Accounting, Real Estate, Insurance)

### Must-Have Sections
- Hero with professional headshot or office
- Practice areas / services
- Attorney/agent/advisor profiles
- Case results or testimonials
- Free consultation CTA
- Blog/resources section
- Contact form
- Credentials and affiliations

### Color Palettes
- **Navy + White + Gold** — authority, trust
- **Dark Green + Cream + Gold** — financial stability
- **Charcoal + White + Blue accent** — modern professional

### Trust Signals
- Bar association / professional licenses
- Years of practice
- Case results / client wins
- "Free Consultation" prominently displayed
- Avvo/Martindale rating (law)

---

## Fitness & Wellness (Gyms, Personal Trainers, Yoga Studios, Martial Arts)

### Must-Have Sections
- Hero with action/class photography
- Class schedule / programs
- Membership options with pricing
- Trainer/instructor profiles
- Transformation stories / testimonials
- Free trial or first-class-free CTA
- Location, hours, amenities
- FAQ (cancellation policy, what to bring, etc.)

### Color Palettes
- **Black + Neon accent (green, orange, red)** — high energy, gym culture
- **Earth tones + White** — wellness, yoga, mindfulness
- **Blue + White + Dark** — clean, health-focused

### Trust Signals
- Certified trainers (NASM, ACE, etc.)
- Transformation photos
- Member count
- Google/Yelp rating

---

## General Patterns (Apply to All Niches)

### Typography
- Headlines: Bold sans-serif (Inter, Montserrat, Poppins)
- Body: Clean sans-serif (Inter, Open Sans, Lato)
- Phone numbers: Extra bold, large
- CTAs: Bold, uppercase or semi-bold

### Mobile-First Requirements
- Sticky header with phone + CTA
- Touch-friendly buttons (min 44px tap targets)
- Single-column layout on mobile
- Compressed hero (no wasted space)
- Click-to-call phone numbers
- Collapsible navigation (hamburger menu)

### Page Speed
- Optimized images (WebP where possible)
- Minimal JavaScript
- System fonts or Google Fonts (max 2 families)
- No heavy animations on mobile
- Lazy-load below-fold images

### Local SEO Essentials
- Business name, address, phone (NAP) in footer
- Schema markup for LocalBusiness
- City/region mentioned naturally in headlines and copy
- Service area pages (for businesses serving multiple cities)
- Google Maps embed on contact section

---

## Technical Implementation

### Tailwind CDN Approach
For generated single-file HTML sites, use Tailwind CDN for utility-class styling:
```html
<script src="https://cdn.tailwindcss.com"></script>
<script>
tailwind.config = {
  theme: {
    extend: {
      colors: {
        primary: 'var(--primary)',
        secondary: 'var(--secondary)',
        accent: 'var(--accent)',
      }
    }
  }
}
</script>
```
This gives access to all Tailwind utilities without a build step. Combine with CSS custom properties for business-specific colors.

### Asset Integration Priority
When extracted assets are available from the audit phase, use them in this priority:
1. **Brand colors** → ALWAYS use the business's extracted colors as the palette. Their brand = their colors. Only fall back to niche defaults if no web presence exists at all.
2. **Logo** → header + footer (Clearbit URL or scraped)
3. **Work photos** → gallery section (real photos >> stock photos always)
4. **Hero photo** → use business's own if hero-quality, else Unsplash
5. **Testimonials** → real reviews with attribution over placeholders

### Scroll Transitions
Every generated site MUST use unique scroll transitions. See `references/scroll-transitions.md` for the full library. Key rules:
- Never repeat the same transition on adjacent sections
- Use `opacity: 0.05` (not `0`) for initial states (Playwright screenshot compatibility)
- Include the master IntersectionObserver script at page bottom
- Cap stagger delays at 1.5s total
- Reduce `translateX` values on mobile to prevent horizontal scroll

### Typography Pairings for Uniqueness
Rotate through these pairings across sites in the same batch to avoid sameness:
1. **Outfit + DM Sans** — modern, clean tech feel
2. **Playfair Display + Inter** — elegant serif + clean body
3. **Montserrat + Lora** — geometric headings + readable serif body
4. **Poppins + Source Sans 3** — friendly, approachable
5. **Raleway + Merriweather** — refined headings + classic body
6. **Space Grotesk + Work Sans** — techy, distinctive
7. **Fraunces + Nunito Sans** — artisanal headings + soft body
8. **Sora + Libre Franklin** — sharp modern + neutral body
