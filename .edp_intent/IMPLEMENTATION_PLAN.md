# GSAP Animation Implementation Plan

## Selected Animation Options

| Category | Option | Key Config |
|----------|--------|------------|
| Hero | C: Playful Bounce | `scale: 0, rotation: -15, ease: 'back.out(1.7)'` |
| Page Transition | B: Expanding Circle | Pink circle (#FFD4E6) expands from center |
| Scroll | B: Scale Pop | `scale: 0, ease: 'back.out(1.7)'` |
| Hover | C: Lift + Micro Rotate | `y: -8, rotation: 2` |

---

## Phase 1: Simplicity Assessment

### Complexity Score: 6/10 (Acceptable)

**Justification:**
- Using inline scripts (simplest approach for 3-page static site)
- Reusing same animation code pattern across all pages
- No build tools, no external dependencies beyond GSAP CDN
- Each animation function is self-contained and readable

### Scope Limitations (Will NOT Implement):
- External JS files
- Custom GSAP plugins beyond ScrollTrigger
- Complex stagger configurations
- Scrub/pin scroll effects

---

## Phase 2: Foundation - DRY Analysis

### Reusable Patterns Identified:

1. **GSAP CDN Includes** (copy to all 3 pages):
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/ScrollTrigger.min.js"></script>
```

2. **Page Transition Overlay HTML** (copy to all 3 pages):
```html
<div class="page-transition-circle"></div>
```

3. **Page Transition CSS** (copy to all 3 pages):
```css
.page-transition-circle {
    position: fixed;
    top: 50%;
    left: 50%;
    width: 0;
    height: 0;
    background: #FFD4E6;
    border-radius: 50%;
    transform: translate(-50%, -50%);
    z-index: 9999;
    pointer-events: none;
}
```

4. **Shared Animation Functions** (copy to all 3 pages):
   - `initHoverEffects()` - Same for all pages
   - `navigateWithTransition(url)` - Same for all pages
   - `contractCircleOnLoad()` - Same for all pages

### Page-Specific Animations:

| Page | Unique Animations |
|------|-------------------|
| thingyportfolio.html | Hero title, subtitle, buttons, mini-cards, highlight cards scroll |
| about.html | About hero, about-card, interests, fun-facts scroll |
| projects.html | Projects hero, project-cards scroll |

---

## Phase 3: Implementation Checklist

### Step 1: Add GSAP CDN to all pages
**Files:** thingyportfolio.html, about.html, projects.html
**Location:** Before closing `</head>` tag
**Code:**
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/ScrollTrigger.min.js"></script>
```
**Success Criteria:** `typeof gsap !== 'undefined'` in console
**Validation:** Open each page, check console for gsap object

---

### Step 2: Add page transition overlay HTML
**Files:** All 3 pages
**Location:** Immediately after opening `<body>` tag
**Code:**
```html
<div class="page-transition-circle"></div>
```
**Success Criteria:** Element exists in DOM
**Validation:** Inspect element in dev tools

---

### Step 3: Add page transition CSS
**Files:** All 3 pages
**Location:** Add to existing `<style>` block
**Code:**
```css
.page-transition-circle {
    position: fixed;
    top: 50%;
    left: 50%;
    width: 0;
    height: 0;
    background: #FFD4E6;
    border-radius: 50%;
    transform: translate(-50%, -50%);
    z-index: 9999;
    pointer-events: none;
}
```
**Success Criteria:** Circle is invisible (0x0) on page load
**Validation:** Inspect computed styles

---

### Step 4: Implement thingyportfolio.html animations
**Location:** Replace existing `<script>` block (keep theme toggle, add animations)

#### 4a. Page Load Timeline (Hero)
**Selector Targets:**
- `.logo` - slide down with bounce
- `nav a` - slide down with stagger
- `.hero-text h1 span` - scale + rotate (3 spans)
- `.hero-text p` - fade in
- `.btn` - scale pop
- `.mini-card` - scale pop from center stagger

**Code Pattern:**
```javascript
gsap.registerPlugin(ScrollTrigger);

document.addEventListener('DOMContentLoaded', function() {
    // Existing theme code...

    // Contract circle if coming from another page
    contractCircleOnLoad();

    // Initialize all animations
    initPageLoadAnimations();
    initScrollAnimations();
    initHoverEffects();
});

function initPageLoadAnimations() {
    const tl = gsap.timeline();

    // Nav animations
    tl.from('.logo', {
        y: -50,
        opacity: 0,
        duration: 0.6,
        ease: 'back.out(1.7)'
    })
    .from('nav a', {
        y: -30,
        opacity: 0,
        stagger: 0.1,
        duration: 0.5,
        ease: 'back.out(1.4)'
    }, '-=0.3')

    // Hero title - Playful Bounce
    .from('.hero-text h1 span', {
        scale: 0,
        rotation: -15,
        opacity: 0,
        stagger: 0.2,
        duration: 0.8,
        ease: 'back.out(1.7)'
    }, '-=0.2')

    // Subtitle
    .from('.hero-text p', {
        y: 30,
        opacity: 0,
        duration: 0.5,
        ease: 'power2.out'
    }, '-=0.3')

    // Buttons
    .from('.btn', {
        scale: 0,
        stagger: 0.15,
        duration: 0.5,
        ease: 'back.out(2)'
    }, '-=0.2')

    // Mini-cards
    .from('.mini-card', {
        scale: 0,
        opacity: 0,
        stagger: {
            each: 0.1,
            from: 'center'
        },
        duration: 0.6,
        ease: 'back.out(1.4)'
    }, '-=0.3');
}
```
**Success Criteria:** 6+ animations in timeline sequence
**Validation:** Visual inspection, timeline contains nav + hero + cards

---

#### 4b. Scroll Animations (Scale Pop)
**Selector Targets:**
- `.section-header` - triggers at 80%
- `.highlight-card` - triggers at 70%, stagger

**Code Pattern:**
```javascript
function initScrollAnimations() {
    // Section header
    gsap.from('.section-header', {
        scrollTrigger: {
            trigger: '.highlights-section',
            start: 'top 80%'
        },
        scale: 0.8,
        opacity: 0,
        duration: 0.5,
        ease: 'back.out(1.7)'
    });

    // Highlight cards
    gsap.from('.highlight-card', {
        scrollTrigger: {
            trigger: '.highlights-section',
            start: 'top 70%'
        },
        scale: 0,
        stagger: 0.12,
        duration: 0.6,
        ease: 'back.out(1.7)'
    });
}
```
**Success Criteria:** Cards animate when scrolling to section
**Validation:** Scroll to highlights section, cards should pop in

---

#### 4c. Hover Effects (Lift + Micro Rotate)
**Selector Targets:**
- `.btn` - buttons
- `.mini-card` - hero cards
- `.highlight-card` - section cards

**Code Pattern:**
```javascript
function initHoverEffects() {
    const hoverElements = document.querySelectorAll('.btn, .mini-card, .highlight-card');

    hoverElements.forEach(el => {
        el.addEventListener('mouseenter', () => {
            gsap.to(el, {
                y: -8,
                rotation: 2,
                boxShadow: '0 15px 35px rgba(0,0,0,0.12)',
                duration: 0.35,
                ease: 'power2.out'
            });
        });

        el.addEventListener('mouseleave', () => {
            gsap.to(el, {
                y: 0,
                rotation: 0,
                boxShadow: '0 4px 15px rgba(0,0,0,0.05)',
                duration: 0.35,
                ease: 'power2.out'
            });
        });
    });
}
```
**Success Criteria:** Elements lift and rotate on hover
**Validation:** Hover over buttons and cards

---

#### 4d. Page Transition (Expanding Circle)
**Code Pattern:**
```javascript
function navigateWithTransition(url) {
    const circle = document.querySelector('.page-transition-circle');
    const maxSize = Math.max(window.innerWidth, window.innerHeight) * 2.5;

    gsap.to(circle, {
        width: maxSize,
        height: maxSize,
        duration: 0.5,
        ease: 'power2.in',
        onComplete: () => {
            window.location.href = url;
        }
    });
}

function contractCircleOnLoad() {
    const circle = document.querySelector('.page-transition-circle');
    const maxSize = Math.max(window.innerWidth, window.innerHeight) * 2.5;

    // Check if coming from transition (circle should be expanded)
    if (sessionStorage.getItem('pageTransition') === 'true') {
        sessionStorage.removeItem('pageTransition');
        gsap.set(circle, { width: maxSize, height: maxSize });
        gsap.to(circle, {
            width: 0,
            height: 0,
            duration: 0.5,
            ease: 'power2.out',
            delay: 0.1
        });
    }
}

// Update nav links to use transition
document.querySelectorAll('nav a:not(.active)').forEach(link => {
    link.addEventListener('click', function(e) {
        e.preventDefault();
        sessionStorage.setItem('pageTransition', 'true');
        navigateWithTransition(this.getAttribute('href'));
    });
});
```
**Success Criteria:** Circle expands on nav click, contracts on new page
**Validation:** Click nav links, observe transition

---

### Step 5: Implement about.html animations
**Location:** Add to existing `<script>` block

**Page-Specific Elements:**
- `.about-hero h1 span` - title animation
- `.about-hero p` - subtitle
- `.about-card` - main card (scroll trigger)
- `.interest-item` - interests grid (scroll trigger)
- `.fact-item` - fun facts (scroll trigger)

**Unique initPageLoadAnimations():**
```javascript
function initPageLoadAnimations() {
    const tl = gsap.timeline();

    tl.from('.logo', {
        y: -50, opacity: 0, duration: 0.6, ease: 'back.out(1.7)'
    })
    .from('nav a', {
        y: -30, opacity: 0, stagger: 0.1, duration: 0.5, ease: 'back.out(1.4)'
    }, '-=0.3')
    .from('.about-hero h1 span', {
        scale: 0, rotation: -15, opacity: 0, stagger: 0.2, duration: 0.8, ease: 'back.out(1.7)'
    }, '-=0.2')
    .from('.about-hero p', {
        y: 30, opacity: 0, duration: 0.5, ease: 'power2.out'
    }, '-=0.3');
}
```

**Unique initScrollAnimations():**
```javascript
function initScrollAnimations() {
    gsap.from('.about-card', {
        scrollTrigger: { trigger: '.about-card', start: 'top 80%' },
        scale: 0.9, opacity: 0, duration: 0.6, ease: 'back.out(1.4)'
    });

    gsap.from('.interest-item', {
        scrollTrigger: { trigger: '.interests-section', start: 'top 70%' },
        scale: 0, stagger: 0.1, duration: 0.5, ease: 'back.out(1.7)'
    });

    gsap.from('.fact-item', {
        scrollTrigger: { trigger: '.fun-facts', start: 'top 70%' },
        x: -50, opacity: 0, stagger: 0.1, duration: 0.5, ease: 'power2.out'
    });
}
```
**Success Criteria:** About page animations match style
**Validation:** All elements animate correctly

---

### Step 6: Implement projects.html animations
**Location:** Add to existing `<script>` block

**Page-Specific Elements:**
- `.projects-hero h1 span` - title animation
- `.projects-hero p` - subtitle
- `.project-card` - project cards (scroll trigger)
- `.coming-soon` - coming soon section (scroll trigger)

**Unique initPageLoadAnimations():**
```javascript
function initPageLoadAnimations() {
    const tl = gsap.timeline();

    tl.from('.logo', {
        y: -50, opacity: 0, duration: 0.6, ease: 'back.out(1.7)'
    })
    .from('nav a', {
        y: -30, opacity: 0, stagger: 0.1, duration: 0.5, ease: 'back.out(1.4)'
    }, '-=0.3')
    .from('.projects-hero h1 span', {
        scale: 0, rotation: -15, opacity: 0, stagger: 0.2, duration: 0.8, ease: 'back.out(1.7)'
    }, '-=0.2')
    .from('.projects-hero p', {
        y: 30, opacity: 0, duration: 0.5, ease: 'power2.out'
    }, '-=0.3');
}
```

**Unique initScrollAnimations():**
```javascript
function initScrollAnimations() {
    gsap.from('.project-card', {
        scrollTrigger: { trigger: '.projects-grid', start: 'top 75%' },
        scale: 0, stagger: 0.12, duration: 0.6, ease: 'back.out(1.7)'
    });

    gsap.from('.coming-soon', {
        scrollTrigger: { trigger: '.coming-soon', start: 'top 80%' },
        scale: 0.9, opacity: 0, duration: 0.6, ease: 'back.out(1.4)'
    });
}
```
**Success Criteria:** Projects page animations match style
**Validation:** All elements animate correctly

---

## Phase 4: Validation Checklist

| AC ID | Criteria | Test Method |
|-------|----------|-------------|
| AC-001 | GSAP loads | Console: `typeof gsap` |
| AC-002 | Hero bounces | Visual: title scales + rotates |
| AC-003 | Timeline 3+ | Count animations in sequence |
| AC-004 | ScrollTrigger | Scroll to highlights |
| AC-005 | Page transition | Click nav links |
| AC-006 | Hover effects | Hover buttons/cards |
| AC-007 | gsap.to() used | Code review |
| AC-008 | gsap.from() used | Code review |
| AC-009 | Non-linear easing | Code review |

---

## File Modification Summary

| File | Changes |
|------|---------|
| thingyportfolio.html | +GSAP CDN, +transition HTML/CSS, +animation script (~120 lines) |
| about.html | +GSAP CDN, +transition HTML/CSS, +animation script (~100 lines) |
| projects.html | +GSAP CDN, +transition HTML/CSS, +animation script (~90 lines) |

**Total New Lines:** ~310 lines across 3 files
**Complexity Budget:** 4 functions, 0 new files, ~310 LOC, 2 dependencies

---

## Execution Order

1. thingyportfolio.html (main page, most animations)
2. about.html (simpler, reuse patterns)
3. projects.html (simplest, reuse patterns)
4. Cross-page testing (transitions work between all pages)
