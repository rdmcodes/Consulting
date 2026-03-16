# Scroll Transition Patterns Reference

Every generated site MUST use unique, varied scroll transitions between sections. Never repeat the same transition twice on a page. Pick from this library and assign DIFFERENT ones per section.

All transitions use IntersectionObserver with `threshold: 0.15` and trigger once (`unobserve` after). Initial state is set in CSS (NOT inline styles) so Playwright screenshots still render content.

**CRITICAL:** Set initial states with `opacity: 0.05` (not `0`) so content is technically visible but appears hidden. This prevents blank sections in automated screenshots while still creating a dramatic reveal effect in browsers.

---

## Transition Library

### 1. Cascade Stagger
Each child element animates in sequence with a delay. Great for service cards, feature grids, team members.

```css
.cascade-item {
  opacity: 0.05;
  transform: translateY(40px);
  transition: opacity 0.6s ease, transform 0.6s ease;
}
.cascade-item.visible { opacity: 1; transform: translateY(0); }
/* JS adds 100ms delay per child: child.style.transitionDelay = `${i * 100}ms` */
```

### 2. Horizontal Slide-In (Alternating)
Odd items slide from left, even from right. Great for alternating content rows, about sections, feature highlights.

```css
.slide-left {
  opacity: 0.05;
  transform: translateX(-80px);
  transition: opacity 0.7s ease, transform 0.7s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}
.slide-right {
  opacity: 0.05;
  transform: translateX(80px);
  transition: opacity 0.7s ease, transform 0.7s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}
.slide-left.visible, .slide-right.visible { opacity: 1; transform: translateX(0); }
```

### 3. Scale Pop
Element scales up from 80% with a slight bounce. Great for hero stats, trust badges, single featured images.

```css
.scale-pop {
  opacity: 0.05;
  transform: scale(0.8);
  transition: opacity 0.5s ease, transform 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}
.scale-pop.visible { opacity: 1; transform: scale(1); }
```

### 4. Clip Reveal (Curtain)
Content reveals via clip-path like a curtain opening. Great for hero images, gallery photos, full-width sections.

```css
.clip-reveal {
  clip-path: inset(0 50% 0 50%);
  transition: clip-path 0.8s cubic-bezier(0.77, 0, 0.175, 1);
}
.clip-reveal.visible { clip-path: inset(0 0 0 0); }
```

### 5. Blur Unblur
Content starts blurred and sharpens into focus. Great for testimonials, about sections, mission statements.

```css
.blur-in {
  opacity: 0.05;
  filter: blur(12px);
  transform: translateY(20px);
  transition: opacity 0.7s ease, filter 0.7s ease, transform 0.7s ease;
}
.blur-in.visible { opacity: 1; filter: blur(0); transform: translateY(0); }
```

### 6. Rotate-In Tilt
Slight 3D rotation that flattens on scroll. Great for cards, pricing tables, portfolio items.

```css
.tilt-in {
  opacity: 0.05;
  transform: perspective(800px) rotateY(15deg) translateX(40px);
  transition: opacity 0.7s ease, transform 0.8s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}
.tilt-in.visible { opacity: 1; transform: perspective(800px) rotateY(0) translateX(0); }
```

### 7. Counter Slide (Split)
Left column slides in from left, right column from right simultaneously. Great for two-column layouts, about + image, stats + description.

```css
.split-left {
  opacity: 0.05;
  transform: translateX(-60px);
  transition: opacity 0.7s ease, transform 0.7s ease;
}
.split-right {
  opacity: 0.05;
  transform: translateX(60px);
  transition: opacity 0.7s ease, transform 0.7s ease;
}
.split-left.visible, .split-right.visible { opacity: 1; transform: translateX(0); }
```

### 8. Rise & Spread
Container rises while children spread outward from center. Great for service grids, icon features, trust badges row.

```css
.rise-container {
  opacity: 0.05;
  transform: translateY(60px);
  transition: opacity 0.6s ease, transform 0.6s ease;
}
.rise-container.visible { opacity: 1; transform: translateY(0); }
.rise-container .spread-child {
  opacity: 0.05;
  transform: scale(0.7);
  transition: opacity 0.4s ease, transform 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}
.rise-container.visible .spread-child { opacity: 1; transform: scale(1); }
/* JS adds stagger: child.style.transitionDelay = `${300 + i * 80}ms` */
```

### 9. Typewriter Reveal
Text appears character by character (via CSS animation). Great for headlines, taglines, key stats.

```css
.typewriter {
  overflow: hidden;
  white-space: nowrap;
  border-right: 3px solid currentColor;
  width: 0;
  animation: none;
}
.typewriter.visible {
  animation: typing 2s steps(30) forwards, blink 0.7s step-end 3;
}
@keyframes typing { from { width: 0; } to { width: 100%; } }
@keyframes blink { 50% { border-color: transparent; } }
```

### 10. Parallax Float
Background moves slower than foreground content creating depth. Great for hero sections, full-width image sections, CTA banners.

```css
.parallax-section {
  position: relative;
  overflow: hidden;
}
.parallax-bg {
  position: absolute;
  inset: -20% 0;
  background-size: cover;
  background-position: center;
  will-change: transform;
  /* JS: on scroll, translateY based on scroll position */
}
.parallax-content {
  position: relative;
  z-index: 2;
  opacity: 0.05;
  transform: translateY(30px);
  transition: opacity 0.6s ease, transform 0.6s ease;
}
.parallax-content.visible { opacity: 1; transform: translateY(0); }
```

### 11. Morph Border
Section border/outline draws itself on scroll. Great for contact forms, quote request sections, highlighted boxes.

```css
.morph-border {
  position: relative;
  opacity: 0.05;
  transition: opacity 0.5s ease;
}
.morph-border::before {
  content: '';
  position: absolute;
  inset: 0;
  border: 2px solid var(--accent-color);
  clip-path: polygon(0 0, 0 0, 0 0, 0 0);
  transition: clip-path 1s cubic-bezier(0.77, 0, 0.175, 1);
}
.morph-border.visible { opacity: 1; }
.morph-border.visible::before {
  clip-path: polygon(0 0, 100% 0, 100% 100%, 0 100%);
}
```

### 12. Glowing Entrance
Element fades in with a temporary glow/shadow that dissipates. Great for CTA buttons, key stats, pricing highlights.

```css
.glow-enter {
  opacity: 0.05;
  transform: translateY(20px);
  box-shadow: 0 0 0 rgba(var(--accent-rgb), 0);
  transition: opacity 0.5s ease, transform 0.5s ease, box-shadow 0.8s ease;
}
.glow-enter.visible {
  opacity: 1;
  transform: translateY(0);
  box-shadow: 0 0 40px rgba(var(--accent-rgb), 0.3);
}
/* After animation, remove glow */
.glow-enter.settled { box-shadow: 0 0 0 rgba(var(--accent-rgb), 0); }
```

---

## Section Divider Transitions

Use between sections for visual separation. Pick DIFFERENT dividers per section break.

### Wave
```css
.wave-divider { position: relative; }
.wave-divider::after {
  content: '';
  position: absolute;
  bottom: -1px;
  left: 0;
  width: 100%;
  height: 80px;
  background: var(--next-section-bg);
  clip-path: ellipse(55% 100% at 50% 100%);
}
```

### Diagonal Slash
```css
.diagonal-divider {
  clip-path: polygon(0 0, 100% 0, 100% 85%, 0 100%);
  padding-bottom: 80px;
}
```

### Jagged/Mountain
```css
.mountain-divider { position: relative; }
.mountain-divider::after {
  content: '';
  position: absolute;
  bottom: -1px;
  left: 0;
  width: 100%;
  height: 60px;
  background: var(--next-section-bg);
  clip-path: polygon(0% 100%, 5% 60%, 15% 80%, 25% 40%, 35% 70%, 45% 30%, 55% 60%, 65% 20%, 75% 50%, 85% 35%, 95% 65%, 100% 45%, 100% 100%);
}
```

### Curve Dip
```css
.curve-divider { position: relative; }
.curve-divider::after {
  content: '';
  position: absolute;
  bottom: -1px;
  left: 0;
  width: 100%;
  height: 100px;
  background: var(--next-section-bg);
  clip-path: ellipse(60% 100% at 50% 100%);
}
```

### Arrow Point
```css
.arrow-divider { position: relative; }
.arrow-divider::after {
  content: '';
  position: absolute;
  bottom: -1px;
  left: 0;
  width: 100%;
  height: 60px;
  background: var(--next-section-bg);
  clip-path: polygon(0 100%, 50% 0, 100% 100%);
}
```

---

## Assignment Rules

When generating a site, assign transitions to sections using this rotation strategy:

1. **Hero** → Parallax Float (10) or Clip Reveal (4)
2. **Trust Bar** → Rise & Spread (8)
3. **Services** → Cascade Stagger (1) or Tilt-In (6)
4. **About / Story** → Horizontal Slide-In Alternating (2) or Blur Unblur (5)
5. **Testimonials** → Blur Unblur (5) or Scale Pop (3)
6. **Gallery** → Cascade Stagger (1) with different timing than services
7. **Service Area** → Counter Slide Split (7)
8. **Contact/CTA** → Glowing Entrance (12) or Morph Border (11)
9. **Footer** → Simple fade (no fancy transition)

**Never repeat the same transition class on two adjacent sections.** If two sections would get the same one, swap one for an alternative from the library.

For section dividers, rotate through: wave → diagonal → curve → mountain → arrow. Never use the same divider twice in a row.

---

## Master IntersectionObserver Script

Include this once at the bottom of every generated page:

```javascript
document.addEventListener('DOMContentLoaded', () => {
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        entry.target.classList.add('visible');
        observer.unobserve(entry.target);
        // Handle stagger children
        const children = entry.target.querySelectorAll('[data-stagger]');
        children.forEach((child, i) => {
          child.style.transitionDelay = `${parseInt(child.dataset.stagger || 100) * i}ms`;
          setTimeout(() => child.classList.add('visible'), 10);
        });
        // Handle glow settle
        if (entry.target.classList.contains('glow-enter')) {
          setTimeout(() => entry.target.classList.add('settled'), 1200);
        }
      }
    });
  }, { threshold: 0.15, rootMargin: '0px 0px -50px 0px' });

  document.querySelectorAll(
    '.cascade-item, .slide-left, .slide-right, .scale-pop, .clip-reveal, ' +
    '.blur-in, .tilt-in, .split-left, .split-right, .rise-container, ' +
    '.parallax-content, .morph-border, .glow-enter'
  ).forEach(el => observer.observe(el));
});
```
