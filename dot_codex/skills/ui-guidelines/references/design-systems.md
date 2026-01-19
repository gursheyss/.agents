# Design Systems Reference

Detailed specs from world-class design systems. Reference when generating design variations or needing specific values.

## Table of Contents

- [UX Foundations](#ux-foundations)
- [Typography](#typography)
- [Spacing](#spacing)
- [Color](#color)
- [Border Radius](#border-radius)
- [Shadows](#shadows)
- [Component Patterns](#component-patterns)
- [Motion & Animation](#motion--animation)
- [Accessibility](#accessibility)
- [Design System Inspirations](#design-system-inspirations)

---

## UX Foundations

### Nielsen's 10 Usability Heuristics

1. **Visibility of system status** - Keep users informed through appropriate feedback
2. **Match system and real world** - Use familiar language and conventions
3. **User control and freedom** - Provide clear exits (undo, cancel, back)
4. **Consistency and standards** - Follow platform conventions
5. **Error prevention** - Eliminate error-prone conditions or confirm
6. **Recognition over recall** - Make options visible; minimize memory load
7. **Flexibility and efficiency** - Provide accelerators for experts
8. **Aesthetic and minimalist design** - Remove irrelevant information
9. **Help users recover from errors** - Plain language with solutions
10. **Help and documentation** - Concise, task-focused help

### Norman's Design Principles

- **Affordances** - Elements suggest their usage
- **Signifiers** - Visual cues indicate where actions happen
- **Mapping** - Controls relate spatially to effects
- **Feedback** - Every action gets a perceivable response
- **Conceptual model** - Users understand how the system works

### Cognitive Load

- **Limit choices** - 5-7 items in navigation; 3-4 options in decisions
- **Progressive disclosure** - Show only what's needed at each step
- **Chunking** - Group related items; break long forms into steps
- **Visual hierarchy** - Guide attention with size, color, contrast, position

---

## Typography

### Hierarchy (from iA, Stripe, Linear)

```
Display:    32-48px, -0.02em tracking, 700 weight
Heading 1:  24-32px, -0.02em tracking, 600 weight
Heading 2:  20-24px, -0.01em tracking, 600 weight
Heading 3:  16-18px, normal tracking, 600 weight
Body:       14-16px, normal tracking, 400 weight
Caption:    12-13px, +0.01em tracking, 400-500 weight
```

### Best Practices

- Max 60-75 characters per line
- Line height: 1.4-1.6 for body, 1.2-1.3 for headings
- Use weight contrast (400 vs 600) more than size contrast
- Limit to 2 font families
- System fonts: `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif`

---

## Spacing

### 8px Grid System

```
4px   - Tight: icon padding, inline spacing
8px   - Base: related elements, form field padding
12px  - Comfortable: between form fields
16px  - Standard: section padding, card padding
24px  - Relaxed: between sections
32px  - Spacious: major section breaks
48px  - Generous: page section separation
64px+ - Hero: landing page sections
```

### Principles

- Related items closer together (Gestalt proximity)
- Consistent internal padding (all sides equal, or vertical > horizontal)
- White space creates focus
- Touch targets minimum 44x44px

---

## Color

### Neutral Foundation (from Stripe, Linear, Vercel)

```
Background:     #FFFFFF / #000000 (dark)
Surface:        #FAFAFA / #111111 (dark)
Border:         #E5E5E5 / #333333 (dark)
Text primary:   #171717 / #EDEDED (dark)
Text secondary: #737373 / #A3A3A3 (dark)
Text tertiary:  #A3A3A3 / #737373 (dark)
```

### Accent Usage

- Primary action: single brand color, used sparingly
- Interactive elements: consistent color for all clickable items
- Semantic: red (error), green (success), yellow (warning), blue (info)
- Hover: 10% darker or add subtle background
- Focus: 2px ring with offset, high contrast

### Principles

- WCAG AA: 4.5:1 for text, 3:1 for UI elements
- One primary accent; avoid rainbow interfaces
- Use opacity for secondary states
- Dark mode: reduce contrast, use darker surfaces (don't just invert)

---

## Border Radius

```
None (0px):     Tables, dividers, full-bleed images
Small (4px):    Buttons, inputs, tags, badges
Medium (8px):   Cards, modals, dropdowns
Large (12px):   Feature cards, hero elements
Full (9999px):  Avatars, pills, toggle tracks
```

### Principles

- Pick 2-3 values and stick to them
- Nested: inner radius = outer radius - padding
- Sharp = technical/precise; round = friendly/approachable

---

## Shadows

### Elevation Scale (from Material, Linear)

```
Level 0: none (flat)
Level 1: 0 1px 2px rgba(0,0,0,0.05)      - Subtle lift (cards)
Level 2: 0 4px 6px rgba(0,0,0,0.07)      - Raised (dropdowns)
Level 3: 0 10px 15px rgba(0,0,0,0.1)     - Floating (modals)
Level 4: 0 20px 25px rgba(0,0,0,0.15)    - High (popovers)
```

### Principles

- Natural light feel (top-down, slight offset)
- Dark mode: use lighter surfaces instead of shadows
- Combine with subtle border for definition
- Interactive elements can elevate on hover

---

## Component Patterns

### Buttons

**Hierarchy:**
1. **Primary** - One per view, main action, filled
2. **Secondary** - Supporting actions, outlined or ghost
3. **Tertiary** - Low-emphasis, text-only with hover
4. **Destructive** - Red with confirmation

**States:** Default -> Hover (+shadow or darken) -> Active (scale 0.98) -> Disabled (50% opacity)

**Best practices:**
- Verb + noun labels ("Create project" not "Create")
- Sentence case, not ALL CAPS
- Icon left of text; icon-only needs tooltip
- Primary button right-aligned in forms/dialogs
- Loading: replace text with spinner, maintain width
- Min: 80px width, 36px height (touch: 44px)

### Forms

**Input anatomy:**
```
Label
[Input field with placeholder/value]
Helper text or error message
```

**Best practices:**
- Labels above inputs (not inside)
- Placeholder for format hints only, not labels
- Inline validation on blur, not every keystroke
- Error messages: specific, actionable
- Mark optional fields instead of required
- Single column outperforms multi-column

### Cards

**Anatomy:**
```
[Media/Image]        <- Optional
Eyebrow / Metadata   <- Optional
Title                <- Required
Description          <- Optional
[Actions]            <- Optional
```

**Best practices:**
- Entire card clickable for primary action
- Consistent padding (16-24px)
- Image aspect ratios: 16:9, 4:3, 1:1 (consistent)
- Max 2 actions; overflow to menu
- Hover: translateY -2px + shadow increase

### Tables

- Left-align text, right-align numbers
- Zebra striping OR row hover, not both
- Sticky header on scroll
- Sortable columns show sort indicator
- Row hover reveals action buttons (or kebab menu)
- Empty state: message + action
- Min row height: 48px (touch), 40px (dense)

### Navigation

**By scale:**
- 2-5 items: Tab bar / horizontal tabs
- 5-10 items: Side navigation (collapsible)
- 10+ items: Side nav with sections/groups

**Best practices:**
- Current location always visible
- Breadcrumbs for deep hierarchy only
- Mobile: bottom nav for primary actions
- Icons + labels together; icon-only needs tooltip

---

## Motion & Animation

### Timing Guidelines

```
Micro-interactions:     100-150ms (buttons, toggles, hover)
Small transitions:      150-200ms (dropdowns, tooltips)
Medium transitions:     200-300ms (modals, panels)
Large transitions:      300-500ms (page transitions)
Staggered lists:        50-100ms between items
```

### Easing Functions

```css
--ease-out: cubic-bezier(0.16, 1, 0.3, 1);      /* Entrances */
--ease-in: cubic-bezier(0.7, 0, 0.84, 0);       /* Exits */
--ease-in-out: cubic-bezier(0.65, 0, 0.35, 1);  /* Move/resize */
--ease-spring: cubic-bezier(0.34, 1.56, 0.64, 1);  /* Playful */
--ease-smooth: cubic-bezier(0.4, 0, 0.2, 1);       /* Material */
```

### Patterns

**Entrances:** Fade in + slide up (8-16px), scale 0.95->1 + fade, stagger children 50ms

**Exits:** Fade out (faster than entrance), scale to 0.95, slide in dismissal direction

**Hover/Focus:** TranslateY -2px, scale 1.02-1.05, shadow increase, background shift

**Loading:** Skeleton shimmer, pulse opacity 0.5-1, spinner rotate

### Reduced Motion

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

Replace motion with instant state changes, opacity only, no parallax/autoplay.

---

## Accessibility

### WCAG Quick Reference

**Perceivable:**
- Contrast: 4.5:1 text, 3:1 UI components
- Don't rely on color alone (add icons, patterns)
- Text resizable to 200%
- Captions for video; transcripts for audio

**Operable:**
- All functionality via keyboard
- No keyboard traps
- Skip links for repeated content
- Touch targets: 44x44px minimum

**Understandable:**
- Consistent navigation
- Identify input errors clearly
- Labels and instructions for forms

**Robust:**
- Semantic HTML
- ARIA only when HTML isn't enough
- Test with screen readers

### Focus Management

```css
:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

:focus:not(:focus-visible) {
  outline: none;
}
```

### ARIA Patterns

```html
<!-- Modal -->
<div role="dialog" aria-modal="true" aria-labelledby="title">

<!-- Tab panel -->
<div role="tablist">
  <button role="tab" aria-selected="true" aria-controls="panel1">
</div>
<div role="tabpanel" id="panel1">

<!-- Live region -->
<div aria-live="polite" aria-atomic="true">

<!-- Loading -->
<button aria-busy="true" aria-describedby="loading-text">
```

---

## Design System Inspirations

### Clarity & Precision
- [Linear](https://linear.app) - Information density
- [Stripe](https://stripe.com) - Trust through craft
- [Vercel](https://vercel.com) - Developer simplicity

### Warmth & Approachability
- [Airbnb](https://airbnb.com) - Friendly, image-forward
- [Notion](https://notion.so) - Flexible, playful
- [Slack](https://slack.com) - Conversational, colorful

### Data & Density
- [Bloomberg Terminal](https://bloomberg.com) - Maximum information
- [Figma](https://figma.com) - Tool-like precision
- [GitHub](https://github.com) - Code-centric clarity

### Motion & Delight
- [Apple](https://apple.com) - Cinematic quality
- [Framer](https://framer.com) - Motion-first
- [Lottie](https://lottiefiles.com) - Micro-animation inspiration

Reference specific aspects: "Linear's density", "Stripe's button hierarchy", "Airbnb's card layout", "Vercel's dark mode palette".

---

## Quick Decision Framework

When unsure, ask:

1. **Is it clear?** - User knows what to do and what happened
2. **Is it fast?** - Minimum steps, appropriate feedback
3. **Is it consistent?** - Matches patterns elsewhere
4. **Is it accessible?** - Keyboard, screen reader, contrast
5. **Is it calm?** - No unnecessary motion, color, or elements
