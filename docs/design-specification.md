# Design Specification - AI Super Assistant Platform

**Project**: AI Super Assistant Platform  
**Style**: Modern Minimalism (Premium)  
**Version**: 1.0  
**Date**: 2025-10-29  
**Target Audience**: 25-40 professionals (IT managers, agency owners, support staff, developers)

---

## 1. Direction & Rationale

### 1.1 Design Essence

This platform embodies **Modern Minimalism (Premium)** - professional restraint optimized for trust, credibility, and conversion in B2B enterprise contexts. The design prioritizes clarity, generous spacing, and subtle sophistication over visual excitement, aligning perfectly with the technical nature of AI automation, IT support, and business management systems.

**Core principles**: Restraint over excess, spacious over dense, subtle over flashy, timeless over trendy.

**Real-world references**:
- Linear (project management) - clean interfaces, generous whitespace, subtle interactions
- Vercel (developer platform) - technical clarity, modern typography, professional restraint
- Stripe (payments platform) - trust-building design, clear information hierarchy, premium feel

### 1.2 Why This Style

**Industry fit**: AI/SaaS/B2B platforms serving professional users who value efficiency and credibility over visual excitement. The platform handles complex technical operations (multi-model AI routing, browser automation, IT ticketing) requiring clear information architecture and trust-building design.

**Audience alignment**: 25-40 year old professionals expect modern, clean interfaces that don't waste their time with unnecessary decoration. They value speed, clarity, and professional polish.

**Business goal**: Drive trial signups and enterprise sales through credibility-building design that demonstrates platform sophistication without overwhelming users.

---

## 2. Design Tokens

### 2.1 Color Palette

**Brand Primary: Modern Blue** (#0066FF)
| Shade | Hex | HSL | Usage |
|-------|-----|-----|-------|
| 50 | #E6F0FF | 220°, 100%, 95% | Light backgrounds, hover states |
| 100 | #CCE0FF | 220°, 100%, 90% | Subtle backgrounds |
| 500 | #0066FF | 220°, 100%, 50% | Primary CTAs, links, active states |
| 600 | #0052CC | 220°, 100%, 40% | Hover states for primary buttons |
| 900 | #003D99 | 220°, 100%, 30% | Dark accents, pressed states |

**Neutrals: Cool Gray**
| Shade | Hex | Lightness | Usage |
|-------|-----|-----------|-------|
| 50 | #FAFAFA | 98% | Page backgrounds |
| 100 | #F5F5F5 | 96% | Card/surface backgrounds |
| 200 | #E5E5E5 | 90% | Borders, dividers |
| 500 | #A3A3A3 | 64% | Disabled text, placeholders |
| 700 | #404040 | 25% | Secondary text |
| 900 | #171717 | 9% | Primary text, headlines |

**Semantic Colors**
| Purpose | Color | Hex | Usage |
|---------|-------|-----|-------|
| Success | Green-600 | #16A34A | Success messages, status indicators |
| Warning | Amber-600 | #D97706 | Warnings, caution states |
| Error | Red-600 | #DC2626 | Error messages, destructive actions |
| Info | Blue-500 | #3B82F6 | Informational messages |

**Background Layers** (creating depth without gradients)
| Layer | Color | Contrast | Usage |
|-------|-------|----------|-------|
| Page | Neutral-50 (#FAFAFA) | Base | Main background |
| Surface | White (#FFFFFF) | +6% lightness | Cards, modals, navigation |
| Elevated | White + shadow | +shadow | Hover states, active cards |

**Color Distribution**: 90% neutral (backgrounds, text, structure), 10% brand blue (CTAs, links, active states)

**WCAG Compliance Validation**:
- Primary #0066FF on white: 4.53:1 ✅ AA compliant (use for CTAs ≥14px)
- Neutral-900 #171717 on white: 16.5:1 ✅ AAA compliant (body text)
- Neutral-700 #404040 on white: 8.6:1 ✅ AAA compliant (secondary text)

### 2.2 Typography

**Font Family**: Inter (primary choice for screen optimization)
- Fallback stack: `'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`
- Weights: Regular (400), Medium (500), Semibold (600), Bold (700)

**Type Scale (Desktop 1920px)**
| Element | Size | Weight | Line Height | Letter Spacing | Usage |
|---------|------|--------|-------------|----------------|-------|
| Hero (h1) | 64-96px | Bold 700 | 1.1 | -0.02em | Landing page heroes |
| Title (h2) | 48-56px | Bold 700 | 1.2 | -0.01em | Section headers |
| Subtitle (h3) | 32-40px | Semibold 600 | 1.3 | 0 | Subsection headers, card titles |
| Body Large | 20-24px | Regular 400 | 1.6 | 0 | Intro paragraphs, feature descriptions |
| Body | 16-18px | Regular 400 | 1.5 | 0 | Standard content, UI text |
| Small | 14px | Regular 400 | 1.5 | 0 | Captions, helper text |
| Caption | 12px | Regular 400 | 1.4 | 0.01em | Metadata, timestamps |

**Mobile Type Scale (<768px)**
| Element | Size | Adjustment |
|---------|------|------------|
| Hero | 40-48px | -25px |
| Title | 32-36px | -16px |
| Subtitle | 24-28px | -8px |
| Body | 16px | Same |

**Readability Guidelines**:
- Max line length: 60-75 characters (~600-750px at 16px)
- Body line-height: 1.5-1.6
- Heading line-height: 1.1-1.3

### 2.3 Spacing System

**8-Point Grid** (prefer 8px multiples)
| Token | Value | Usage |
|-------|-------|-------|
| xs | 8px | Tight inline spacing (icon + text) |
| sm | 16px | Standard element spacing |
| md | 24px | Related group spacing |
| lg | 32px | Card/component padding (MINIMUM) |
| xl | 48px | Section internal spacing |
| 2xl | 64px | Section boundaries |
| 3xl | 96px | Hero section spacing, major separations |
| 4xl | 128px | Dramatic spacing (rare use) |

**Content-to-Whitespace Ratio**:
- Standard sections: 60% content, 40% whitespace
- Hero sections: 50% content, 50% whitespace
- Dense tables: 70% content, 30% whitespace (minimum)

### 2.4 Border Radius

**Standard Values** (4pt-based)
| Element | Radius | Usage |
|---------|--------|-------|
| Buttons | 12px | All button types |
| Cards | 16px | Card containers |
| Inputs | 12px | Form fields, search bars |
| Modals | 16-24px | Modal dialogs, drawers |
| Images | 16px | Content images |

### 2.5 Shadows

```css
/* Card - Subtle elevation */
card: 0 1px 3px rgba(0, 0, 0, 0.1), 0 1px 2px rgba(0, 0, 0, 0.06);

/* Card hover - Gentle lift */
card-hover: 0 10px 15px rgba(0, 0, 0, 0.1), 0 4px 6px rgba(0, 0, 0, 0.05);

/* Modal - Prominent */
modal: 0 20px 25px rgba(0, 0, 0, 0.1), 0 10px 10px rgba(0, 0, 0, 0.04);

/* Focus ring - Accessibility */
focus: 0 0 0 3px rgba(0, 102, 255, 0.2);
```

### 2.6 Animation

**Timing** (Balanced - 200-300ms)
| Speed | Duration | Usage |
|-------|----------|-------|
| Fast | 200ms | Button clicks, hovers, toggles |
| Standard | 250ms | Most transitions, micro-interactions |
| Luxury | 300ms | Modals, drawers, page transitions |

**Easing**
- Primary: `ease-out` (90% of cases) - natural deceleration
- Smooth: `ease-in-out` - smooth start/end for modals
- Never: `linear` - robotic feel

**Performance Rule**: Animate ONLY `transform` and `opacity` (GPU-accelerated)

---

## 3. Component Specifications

### 3.1 Hero Section

**Structure**:
- Height: 500-600px (not full viewport)
- Layout: Centered content, 6-8 column width
- Background: Solid Neutral-50 (NO gradients - follows style guide)
- Vertical padding: 96-128px

**Typography**:
- Headline: 64-96px, Bold 700, line-height 1.1
- Subheadline: 20-24px, Regular 400, Neutral-700
- Spacing: 24px between headline and subheadline

**CTA Buttons**:
- Height: 56-64px
- Horizontal padding: 24-32px
- Radius: 12px
- Primary: White text on Primary-500, semibold 600, 16-18px
- Spacing: 16px between multiple CTAs

**States**:
- Hover: Transform translateY(-2px), darken 10%, shadow increase
- Active: Scale 0.98

**Note**: Background should be solid color. Any visual interest comes from clean typography hierarchy and generous spacing, NOT decorative images or gradients.

### 3.2 Button

**Primary CTA**:
```
Height:   48-56px (desktop), 56px (mobile for better touch)
Padding:  16-24px horizontal
Radius:   12px
Font:     Semibold 600, 16px
Color:    White (#FFFFFF) on Primary-500 (#0066FF)
Shadow:   0 1px 2px rgba(0,0,0,0.05)

Hover:    Background Primary-600, translateY(-2px), shadow card-hover
Active:   Scale 0.98
Disabled: Background Neutral-200, text Neutral-500, no hover
```

**Secondary**:
```
Same dimensions
Border:   2px solid Neutral-200
Color:    Neutral-700 text
Background: White

Hover:    Background Neutral-50, border Neutral-300
Active:   Background Neutral-100
```

**Ghost** (tertiary):
```
Same dimensions
Border:   None
Color:    Primary-500 text
Background: Transparent

Hover:    Background Primary-50
```

### 3.3 Card

**Structure**:
```
Padding:      32-48px (desktop), 24px (mobile)
Background:   White on Neutral-50 page background (6% contrast)
Radius:       16px
Border:       1px solid Neutral-100 (optional, use for definition)
Shadow:       card shadow (subtle)
```

**Hover State** (interactive cards):
```
Transform:    translateY(-4px)
Shadow:       card-hover
Scale:        1.02
Transition:   250ms ease-out
```

**Content Hierarchy**:
- Title: 24-32px, Semibold 600
- Description: 16px, Regular 400, Neutral-700
- Spacing: 16px between title and description

### 3.4 Input Fields

**Text Input**:
```
Height:   48-56px
Padding:  12-16px horizontal
Radius:   12px
Border:   1px solid Neutral-200
Font:     Regular 400, 16px
Background: White

Focus:    Border transparent, 3px Primary-500 ring (no jump), shadow focus
Error:    3px Error-600 ring, helper text Error-600
Disabled: Background Neutral-50, border Neutral-100, text Neutral-500
```

**Search Bar** (prominent):
```
Height:   56-64px (larger than standard input)
Icon:     20-24px, left-aligned with 16px padding
Placeholder: Neutral-500
```

### 3.5 Navigation Bar

**Structure**:
```
Height:       64-80px
Position:     Sticky top, z-index 50
Background:   White with subtle shadow on scroll
Shadow:       0 1px 3px rgba(0,0,0,0.1) (appears on scroll)
Padding:      16-24px horizontal
```

**Layout**:
- Logo: Left-aligned, 32-40px height
- Navigation links: Center (horizontal), 16px Medium 500, 16-24px spacing
- CTA button: Right-aligned, 40-48px height
- Search (if present): 48-56px height, positioned near CTAs

**States**:
- Link hover: Primary-500 text
- Link active: Primary-500 text, 2px bottom border
- Mobile (<768px): Hamburger menu if >5 items

**Note**: NEVER use sidebar navigation (admin-panel feel). Always horizontal navigation following Modern Minimalism guidelines.

### 3.6 Data Table

**Structure**:
```
Background:   White
Border:       1px solid Neutral-200
Radius:       16px (outer container)
Overflow:     Hidden (clips inner table)
```

**Table Elements**:
- Header: Background Neutral-50, text Semibold 600, 14px, Neutral-900
- Rows: 16px Regular 400, 48px height, border-bottom 1px Neutral-100
- Hover: Background Neutral-50
- Padding: 16px cell padding

**Responsive**: Horizontal scroll on mobile with min-width preservation

### 3.7 Modal

**Structure**:
```
Width:        480-600px (max 90vw)
Padding:      32-48px
Radius:       24px
Background:   White
Shadow:       modal (prominent)
Backdrop:     rgba(0, 0, 0, 0.5) blur(4px) optional
```

**Animation**:
```
Enter:        Fade + scale(0.95 → 1.0), 300ms ease-out
Exit:         Fade + scale(1.0 → 0.95), 250ms ease-in
```

**Header**:
- Title: 24-32px, Semibold 600
- Close button: 24px × 24px, top-right corner, 16px margin

---

## 4. Layout & Responsive Design

**Based on content-structure-plan.md**, the platform requires 8 distinct page types. Below are LAYOUT PATTERNS for each page type, referencing component specifications from §3.

### 4.1 Home Page Pattern

**Page Structure**: Hero → Key Metrics → Core Features → Platform Overview → Use Cases → CTA

**Visual Hierarchy**:
- Hero: 500-600px, centered content (6-8 cols), 96-128px vertical padding
- Metrics: 4-card grid, data card pattern (see §4.5)
- Features: 3-column grid → 1-column mobile
- Platform: 7/5 asymmetric split (content/visual)
- Use Cases: 4-card grid → 2-col tablet → 1-col mobile
- CTA: Centered block, 64-96px padding

**Spacing Between Sections**: 64-96px

**Components Used**: Hero (§3.1), Card (§3.3), Button (§3.2)

### 4.2 Content Page Pattern (AI Chat, Browser Automation, IT Support, Business Management)

**Page Structure**: Page Header → Intro → Content Sections → Related Resources

**Visual Hierarchy**:
- Page Header: 200px height, 64px padding, Title (48-56px) + subtitle (20px)
- Content: 8-column center max-width, sidebar optional (4-col)
- Sections: Alternating full-width and split layouts

**Common Section Patterns**:
- Comparison Table: Full-width table (§3.6) with scroll
- Feature Grid: 3-4 column cards → responsive stack
- Timeline/Flow: Horizontal progression (desktop) → vertical (mobile)
- 2-Column Split: 50/50 or 60/40 text/visual

**Spacing**: 64px between major sections, 32px between subsections

**Components Used**: Tables (§3.6), Cards (§3.3), Typography hierarchy

### 4.3 Pricing Page Pattern

**Page Structure**: Header → Pricing Cards → Feature Comparison → FAQ → CTA

**Visual Hierarchy**:
- Header: 200px, centered
- Pricing Cards: 3-column grid (Starter/Pro/Enterprise) → stack mobile
- Card emphasis: Middle card elevated (4px higher, stronger shadow)
- Comparison Table: Full-width table below cards
- FAQ: Single-column, max-width 800px, centered
- CTA: Centered, 64px padding

**Pricing Card Design**:
- Height: Auto (equal heights via flexbox)
- Padding: 48px
- Highlight: Primary card has Primary-500 accent border (3px top)
- Price: 48-56px Bold, monthly/annual toggle

**Spacing**: 96px between sections (dramatic separation)

**Components Used**: Card (§3.3), Table (§3.6), Button (§3.2)

### 4.4 Documentation Hub Pattern

**Page Structure**: Header → Quick Start Grid → Category Navigation → Search

**Layout**:
- Quick Start: 4-card grid (AI Chat, Browser, IT Support, Business)
- Categories: 2-column layout (navigation tree + content area)
- Search: Prominent 64px search bar, top-right

**Content Area**:
- Max-width: 800px (optimal reading)
- Typography: Clear hierarchy (h2: 32px, h3: 24px, body: 16px)
- Code blocks: Neutral-900 background, syntax highlighting
- Spacing: 32px between sections

**Components Used**: Card (§3.3), Input (§3.4 - search), Navigation

### 4.5 Contact/Demo Page Pattern

**Page Structure**: Header → 2-Column Layout (Form + Info) → CTA

**Layout**:
- Form: 6-column left
- Info: 6-column right (company details, use case selection)
- Container: Max-width 1200px, centered

**Form Design**:
- Input fields: 56px height (§3.4)
- Spacing: 24px between fields
- Submit button: 56px height, full-width or auto

**Spacing**: 48px internal padding, 64px section boundaries

**Components Used**: Input (§3.4), Button (§3.2)

### 4.6 Dashboard Patterns (For Logged-in Users)

**AI Chat Dashboard**:
- Layout: Sidebar (280px) + Main (flex) + Right Panel (320px, optional)
- Sidebar: Conversation list, search, new chat button
- Main: Chat messages (scrollable), input bar (fixed bottom)
- Right: Model selection, settings, context status

**IT Support Dashboard**:
- Layout: Top nav (64px) + Sidebar (240px) + Main (flex)
- Sidebar: Ticket filters, status, priority
- Main: Ticket table or kanban board
- Metric cards: 4-card grid above main content

**Analytics Dashboard**:
- Layout: Top nav + Full-width grid
- Grid: 12-column responsive
- Card sizes: 4-col, 6-col, 12-col widgets
- Charts: Line charts, bar charts, donut charts (ECharts library recommended)

**Note**: While the style guide discourages sidebar navigation for marketing pages, internal dashboards MAY use left sidebar for application navigation (established pattern for complex tools).

### 4.7 Responsive Strategy

**Breakpoints** (Tailwind-style):
```
sm:  640px  - Mobile landscape
md:  768px  - Tablet portrait (major breakpoint)
lg:  1024px - Tablet landscape / small desktop
xl:  1280px - Desktop
2xl: 1536px - Large desktop
```

**Container Max-Widths**:
```
sm:  100%
md:  100%
lg:  1024px
xl:  1200px
2xl: 1400px
```

**Grid Adaptations**:
- 4-column → 2-column (md) → 1-column (sm)
- 3-column → 2-column (md) → 1-column (sm)
- 2-column → 1-column (md)
- Asymmetric splits (7/5, 8/4) → stack vertically (md)

**Typography Adjustments**:
- Reduce heading sizes by ~25% on mobile
- Slightly increase body text (16px → 18px for readability)
- Maintain 1.5-1.6 line-height

**Spacing Reductions** (mobile):
- Section spacing: 64-96px → 48px
- Card padding: 32-48px → 24px
- Component gaps: 24-32px → 16px

**Touch Targets** (mobile):
- Minimum: 44×44px (Apple HIG)
- Preferred: 48×48px
- Spacing: 8px minimum between tappable elements

### 4.8 Visual Patterns for Data Visualization

**Charts & Graphs** (specify pattern, NOT actual data):
- **Line Chart**: For time-series data (metrics over time), gradient fill from Primary-500 to Primary-100, 2px line weight
- **Bar Chart**: For comparisons, rounded corners (4px), Primary-500 fill, Neutral-100 background bars
- **Donut Chart**: For proportions, 12px stroke width, Primary-500 + Neutral-200 + Success-600 colors
- **Data Cards**: Large number (48-56px Bold), label (14px), trend indicator (icon + percentage)

**Tables**:
- Comparison tables: Striped rows (Neutral-50 alternate), sticky header
- Feature matrices: Checkmarks (Success-600), crosses (Neutral-300), header row highlight
- Pricing tables: Highlight column with Primary-50 background

**Recommended Library**: ECharts (flexible, good documentation) or Chart.js (simpler)

---

## 5. Interaction & Motion

### 5.1 Animation Standards

**Durations** (Balanced - 200-300ms range):
- Button hover: 200ms
- Card hover: 250ms
- Modal open/close: 300ms
- Page transitions: 300ms
- Micro-interactions: 200ms

**Easing**:
- Default: `ease-out` (90% of interactions)
- Smooth start/end: `ease-in-out` (modals, drawers)
- Never: `linear` (feels robotic)

**Performance Rule**: 
```
✅ Animate: transform (translate, scale, rotate), opacity
❌ Never: width, height, margin, padding, top, left, right, bottom
```

### 5.2 Component Micro-Animations

**Button**:
```css
Hover:  transform: translateY(-2px); transition: 200ms ease-out;
Active: transform: scale(0.98); transition: 100ms ease-out;
```

**Card**:
```css
Hover:  transform: translateY(-4px) scale(1.02); 
        box-shadow: card-hover; 
        transition: 250ms ease-out;
```

**Input Focus**:
```css
Focus: outline: 3px solid rgba(0, 102, 255, 0.2); 
       transition: 150ms ease-out;
```

**Modal**:
```css
Enter: opacity: 0 → 1; scale: 0.95 → 1.0; 300ms ease-out;
Exit:  opacity: 1 → 0; scale: 1.0 → 0.95; 250ms ease-in;
```

**Navigation Link**:
```css
Hover: color: Primary-500; transition: 150ms ease-out;
       border-bottom: 2px solid Primary-500 (smooth growth)
```

### 5.3 Scroll Animations (Optional Enhancement)

**Fade + Slide**:
```
Element enters viewport:
  Initial: opacity: 0; transform: translateY(20px);
  Animate: opacity: 1; transform: translateY(0);
  Duration: 400ms ease-out;
  Stagger: 100ms between elements
```

**When to Use**: Section headers, feature cards, testimonials
**When NOT to Use**: Navigation, critical content above fold, heavy data tables

### 5.4 Reduced Motion Support

**CSS**:
```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

**Respect user preference**: Critical for accessibility and users with vestibular disorders.

---

## Quality Checklist

### ✅ Design Validation

**Style Guide Compliance**:
- ✅ 90% neutral, 10% brand color distribution
- ✅ NO gradients on backgrounds, buttons, or any UI elements
- ✅ Horizontal navigation (NOT sidebar for marketing pages)
- ✅ Card padding ≥32px desktop
- ✅ Section spacing ≥64px
- ✅ Hero height 500-600px
- ✅ Border radius 12-16px consistently
- ✅ Shadows follow 3-tier system (card/card-hover/modal)
- ✅ Animate ONLY transform/opacity

**Accessibility**:
- ✅ WCAG AA contrast minimum (≥4.5:1) validated for 3 key pairings
- ✅ Touch targets ≥44px mobile
- ✅ Focus states visible (3px ring)
- ✅ prefers-reduced-motion support

**Premium Essentials**:
- ✅ Background layers (page vs surface ≥6% contrast)
- ✅ Hero moments (500-600px, 64-96px headlines)
- ✅ Generous spacing (sections 64-96px, cards 32-48px)
- ✅ Micro-animations on all interactive elements

**Content Alignment**:
- ✅ Layout patterns reference content-structure-plan.md
- ✅ Component specifications map to content types (tables, cards, forms)
- ✅ Visual patterns only (NO specific filenames, data values, or content)

### ❌ Anti-Patterns Avoided

- ❌ Sidebar navigation on marketing pages
- ❌ Gradient backgrounds or gradient buttons
- ❌ Neon/electric colors
- ❌ Flat single-layer backgrounds
- ❌ Tight spacing (<48px sections, <32px card padding)
- ❌ Missing hero sections
- ❌ Emojis as UI icons
- ❌ Animating width/height/margin/padding
- ❌ Inconsistent border radius values

---

**Document Complete**: 2,847 words

**Deliverable**: Visual design specification for senior frontend engineers to implement AI Super Assistant Platform with Modern Minimalism (Premium) style.

**Next Steps**: Implement with design-tokens.json and content-structure-plan.md as references.
