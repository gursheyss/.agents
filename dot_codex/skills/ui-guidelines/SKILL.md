---
name: ui-guidelines
description: Opinionated constraints for building accessible, fast, delightful web UIs. Use when designing, implementing, or reviewing frontend components, interaction patterns, forms, navigation, animation, layout, typography, accessibility, performance, theming, or content handling. Includes stack preferences (Tailwind, motion/react, Base UI) and references to design system best practices.
---

# UI Guidelines

Apply these constraints when building or reviewing web interfaces. Use MUST/SHOULD/NEVER to guide decisions.

For detailed design specs (typography scales, spacing, colors, shadows, component patterns), see [references/design-systems.md](references/design-systems.md).

## Stack

- MUST use Tailwind CSS defaults unless custom values already exist or are explicitly requested
- MUST use `motion/react` (formerly `framer-motion`) when JavaScript animation is required
- SHOULD use `tw-animate-css` for entrance and micro-animations in Tailwind CSS
- MUST use `cn` utility (`clsx` + `tailwind-merge`) for class logic

## Components

- MUST use accessible component primitives for anything with keyboard or focus behavior (`Base UI`, `React Aria`, `Radix`)
- MUST use the project's existing component primitives first
- NEVER mix primitive systems within the same interaction surface
- SHOULD prefer [`Base UI`](https://base-ui.com/react/components) for new primitives if compatible with the stack
- MUST add an `aria-label` to icon-only buttons
- NEVER rebuild keyboard or focus behavior by hand unless explicitly requested

## Interaction & Input

- MUST provide full keyboard support per WAI-ARIA APG patterns
- MUST show visible focus rings with `:focus-visible` (group with `:focus-within`)
- MUST manage focus (trap, move, return) per APG patterns
- NEVER remove outlines without a visible replacement
- MUST use `AlertDialog` for destructive or irreversible actions
- MUST hit target >= 24px (mobile >= 44px); if visual <24px, expand hit area
- MUST add `touch-action: manipulation` to prevent double-tap zoom
- SHOULD set `-webkit-tap-highlight-color` to match the design
- MUST ensure if it looks clickable, it is clickable
- SHOULD use structural skeletons for loading states
- NEVER use `h-screen`, use `h-dvh`
- MUST respect `safe-area-inset` for fixed elements
- MUST show errors next to where the action happens
- NEVER block paste in `input` or `textarea` elements

## Forms

- MUST keep inputs hydration-safe (no lost focus/value)
- MUST allow Enter to submit focused input; in `textarea`, Cmd/Ctrl+Enter submits
- MUST keep submit enabled until request starts; then disable with spinner
- MUST show spinner and keep original label on loading buttons
- MUST accept free text first; validate after, do not block typing
- MUST allow incomplete form submission to surface validation
- MUST show errors inline near fields; on submit, focus first error
- MUST use `autocomplete` plus meaningful `name`; correct `type` and `inputmode`
- SHOULD disable spellcheck for emails/codes/usernames
- SHOULD end placeholders with ellipsis (U+2026) and include an example pattern
- MUST warn on unsaved changes before navigation
- MUST work with password managers and 2FA; allow pasting codes
- MUST trim values to handle trailing spaces from text expansion
- MUST avoid dead zones on checkboxes/radios; label + control share one hit target

## Navigation & State

- MUST reflect state in URL (deep-link filters, tabs, pagination, expanded panels)
- MUST restore scroll position on Back/Forward
- MUST use links (`a`/`Link`) for navigation to support Cmd/Ctrl/middle-click
- NEVER use `div` with `onClick` for navigation

## Feedback

- SHOULD use optimistic UI; reconcile on response; rollback or offer Undo on failure
- MUST confirm destructive actions or provide an Undo window
- MUST use polite `aria-live` for toasts and inline validation
- SHOULD use ellipsis (U+2026) for options opening follow-ups (e.g., "Rename...") and loading states

## Touch & Drag

- MUST use generous targets and clear affordances; avoid finicky interactions
- MUST delay first tooltip; subsequent peers can be instant
- MUST use `overscroll-behavior: contain` in modals/drawers
- MUST disable text selection during drag and set `inert` on dragged elements

## Animation

- NEVER add animation unless explicitly requested
- MUST honor `prefers-reduced-motion` with a reduced or disabled variant
- SHOULD prefer CSS > Web Animations API > JS libraries
- MUST animate only compositor props (`transform`, `opacity`)
- NEVER animate layout props (`width`, `height`, `top`, `left`, `margin`, `padding`)
- SHOULD avoid animating paint props (`background`, `color`) except for small, local UI
- NEVER use `transition: all`; list properties explicitly
- SHOULD use `ease-out` on entrance
- NEVER exceed `200ms` for interaction feedback
- MUST pause looping animations when off-screen
- NEVER introduce custom easing curves unless explicitly requested
- SHOULD avoid animating large images or full-screen surfaces
- MUST make animations interruptible and input-driven (no autoplay)
- MUST set correct `transform-origin` (motion starts where it physically should)
- MUST apply SVG transforms to a `g` wrapper with `transform-box: fill-box`

## Typography

- MUST use `text-balance` for headings and `text-pretty` for body/paragraphs
- MUST use `tabular-nums` for data and number comparisons
- SHOULD use `truncate` or `line-clamp` for dense UI
- NEVER modify `letter-spacing` (`tracking-*`) unless explicitly requested
- SHOULD avoid widows/orphans (`text-wrap: balance`)
- SHOULD prefer curly quotes (typographic double and single quotes) in copy
- MUST use ellipsis character (U+2026), not three dots

## Layout

- MUST use a fixed `z-index` scale (no arbitrary `z-*`)
- SHOULD use `size-*` for square elements instead of `w-*` + `h-*`
- SHOULD use optical alignment; adjust by +/-1px when perception beats geometry
- MUST align to grid/baseline/edges intentionally; avoid accidental placement
- SHOULD balance icon/text lockups (weight/size/spacing/color)
- MUST verify mobile, laptop, and ultra-wide (simulate ultra-wide at 50% zoom)
- MUST respect safe areas (`env(safe-area-inset-*)`)
- MUST avoid unwanted scrollbars; fix overflows
- SHOULD use flex/grid over JS measurement for layout

## Visual Design

- NEVER use gradients unless explicitly requested
- NEVER use purple or multicolor gradients
- NEVER use glow effects as primary affordances
- SHOULD use Tailwind CSS default shadow scale unless explicitly requested
- SHOULD use layered shadows (ambient + direct)
- SHOULD use crisp edges via semi-transparent borders + shadows
- SHOULD use nested radii; child <= parent and concentric
- SHOULD keep hue consistency (tint borders/shadows/text toward bg)
- MUST give empty states one clear next action
- SHOULD limit accent color usage to one per view
- SHOULD use existing theme or Tailwind CSS color tokens before introducing new ones
- MUST use accessible charts (color-blind-friendly palettes)
- MUST meet contrast; prefer APCA over WCAG 2
- MUST increase contrast on hover/active/focus
- SHOULD match browser UI to background
- SHOULD avoid dark gradient banding; use background images if needed

## Content & Accessibility

- MUST set `<title>` to match current context
- MUST set `scroll-margin-top` on headings; include "Skip to content"; maintain `h1`-`h6` hierarchy
- MUST provide redundant status cues (not color-only); icons have text labels
- MUST provide accessible names even when visuals omit labels
- MUST prefer native semantics (`button`, `a`, `label`, `table`) before ARIA
- MUST use non-breaking spaces for units, shortcuts, and brand names (e.g., "10 MB", "Cmd K")
- MUST mark decorative elements `aria-hidden`
- SHOULD prefer inline help; use tooltips as last resort
- MUST design empty, sparse, dense, and error states
- MUST be resilient to user-generated content (short/average/very long)
- MUST use locale-aware formatting (`Intl.DateTimeFormat`, `Intl.NumberFormat`)

## Content Handling

- MUST handle long content in text containers (`truncate`, `line-clamp-*`, `break-words`)
- MUST add `min-w-0` for flex children to allow truncation
- MUST handle empty states; no broken UI for empty strings/arrays
- MUST have skeletons mirror final content to avoid layout shift
- MUST avoid dead ends; always offer a next step or recovery

## Performance

- NEVER animate large `blur()` or `backdrop-filter` surfaces
- NEVER apply `will-change` outside an active animation
- NEVER use `useEffect` for anything that can be expressed as render logic
- SHOULD test iOS Low Power Mode and macOS Safari
- MUST measure reliably (disable extensions that skew runtime)
- MUST track and minimize re-renders (React DevTools/React Scan)
- MUST profile with CPU/network throttling
- MUST batch layout reads/writes; avoid reflows/repaints
- MUST target mutations (POST/PATCH/DELETE) <500ms
- SHOULD prefer uncontrolled inputs; controlled inputs must be cheap per keystroke
- MUST virtualize large lists (> 50 items)
- MUST preload above-the-fold images; lazy-load the rest
- MUST prevent CLS with explicit image dimensions
- SHOULD use `<link rel="preconnect">` for CDN domains
- SHOULD preload critical fonts with `<link rel="preload" as="font">` and `font-display: swap`

## Dark Mode & Theming

- MUST set `color-scheme: dark` on `<html>` for dark themes
- SHOULD set `meta name=theme-color` to match page background
- MUST set explicit `background-color` and `color` for native `select` (Windows fix)

## Hydration

- MUST ensure inputs with `value` have `onChange` (or use `defaultValue`)
- SHOULD guard date/time rendering against hydration mismatch
- MUST keep inputs hydration-safe (no lost focus/value)
