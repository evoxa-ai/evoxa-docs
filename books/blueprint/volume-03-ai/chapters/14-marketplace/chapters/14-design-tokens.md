---
document_id: BP-0003-V3-C14-14
chapter_id: CH-14-MP-14
feature_pack: FP-MARKETPLACE-0001
title: Design Tokens
version: 1.0.0
status: Draft
owner: Enterprise Design System Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-07
---

# Chapter 14 — Design Tokens

> *The Design Tokens chapter defines the visual foundation of the EVOXA Marketplace. It establishes a unified system of semantic tokens governing color, typography, spacing, elevation, borders, motion, icons and responsive behavior. These tokens ensure visual consistency across every Marketplace screen, widget and component while supporting theming, accessibility and enterprise branding.*

---

# Executive Summary

The Marketplace contains hundreds of interfaces, thousands of components and millions of visual states.

Without a centralized token system, maintaining visual consistency becomes impossible.

The EVOXA Marketplace adopts a **Design Token Architecture** where every visual property is represented as a semantic token instead of hard-coded values.

This enables:

- Consistency
- Themeability
- White-label branding
- Accessibility
- Responsive design
- Future scalability

---

# Objectives

The Design Token System shall:

- Standardize visual properties.
- Eliminate hard-coded styles.
- Support Light/Dark themes.
- Enable Enterprise Branding.
- Improve accessibility.
- Simplify maintenance.
- Integrate with the EVOXA Design System.

---

# Token Hierarchy

```text
Core Tokens

↓

Semantic Tokens

↓

Component Tokens

↓

Widget Tokens

↓

Application Theme
```

---

# Token Categories

The Marketplace defines:

- Color Tokens
- Typography Tokens
- Spacing Tokens
- Radius Tokens
- Shadow Tokens
- Border Tokens
- Motion Tokens
- Icon Tokens
- Elevation Tokens
- Layout Tokens
- Opacity Tokens
- Z-Index Tokens

---

# Color Tokens

## Brand Colors

| Token | Value |
|--------|--------|
| color.brand.primary | #3B82F6 |
| color.brand.secondary | #8B5CF6 |
| color.brand.success | #22C55E |
| color.brand.warning | #F59E0B |
| color.brand.error | #EF4444 |
| color.brand.info | #06B6D4 |

---

## Neutral Palette

| Token | Value |
|--------|--------|
| color.gray.50 | #F9FAFB |
| color.gray.100 | #F3F4F6 |
| color.gray.200 | #E5E7EB |
| color.gray.300 | #D1D5DB |
| color.gray.400 | #9CA3AF |
| color.gray.500 | #6B7280 |
| color.gray.600 | #4B5563 |
| color.gray.700 | #374151 |
| color.gray.800 | #1F2937 |
| color.gray.900 | #111827 |

---

# Semantic Colors

```yaml
color.background.primary
color.background.secondary
color.background.surface
color.background.overlay

color.text.primary
color.text.secondary
color.text.tertiary
color.text.inverse

color.border.default
color.border.focus
color.border.error
color.border.success

color.action.primary
color.action.secondary

color.status.success
color.status.warning
color.status.error
color.status.info

color.ai.primary
color.ai.secondary
```

---

# AI Colors

Artificial Intelligence elements use dedicated colors.

| Token | Value |
|--------|--------|
| color.ai.primary | Purple |
| color.ai.secondary | Indigo |
| color.ai.background | Lavender |
| color.ai.highlight | Violet |

AI interactions remain visually distinguishable.

---

# Typography Tokens

Typeface

```text
Primary

Inter

Secondary

IBM Plex Sans

Monospace

JetBrains Mono
```

---

# Font Sizes

| Token | Size |
|--------|------|
| font.xs | 12 px |
| font.sm | 14 px |
| font.md | 16 px |
| font.lg | 18 px |
| font.xl | 20 px |
| font.2xl | 24 px |
| font.3xl | 30 px |
| font.4xl | 36 px |
| font.display | 48 px |

---

# Font Weights

```yaml
font.weight.light
font.weight.regular
font.weight.medium
font.weight.semibold
font.weight.bold
```

---

# Line Heights

| Token | Value |
|--------|------:|
| line.tight | 1.2 |
| line.normal | 1.5 |
| line.relaxed | 1.75 |

---

# Spacing Tokens

| Token | Value |
|--------|------:|
| space.0 | 0 px |
| space.1 | 4 px |
| space.2 | 8 px |
| space.3 | 12 px |
| space.4 | 16 px |
| space.5 | 20 px |
| space.6 | 24 px |
| space.8 | 32 px |
| space.10 | 40 px |
| space.12 | 48 px |
| space.16 | 64 px |

All spacing follows an 8-point grid.

---

# Border Radius Tokens

| Token | Value |
|--------|------:|
| radius.none | 0 px |
| radius.sm | 4 px |
| radius.md | 8 px |
| radius.lg | 12 px |
| radius.xl | 16 px |
| radius.full | 9999 px |

---

# Shadow Tokens

| Token | Usage |
|--------|-------|
| shadow.sm | Inputs |
| shadow.md | Cards |
| shadow.lg | Dialogs |
| shadow.xl | Drawers |
| shadow.overlay | Modal Windows |

---

# Border Tokens

```yaml
border.width.thin
border.width.default
border.width.thick

border.style.solid
border.style.dashed
border.style.dotted
```

---

# Elevation Tokens

```text
Level 0

Flat

↓

Level 1

Cards

↓

Level 2

Dropdowns

↓

Level 3

Dialogs

↓

Level 4

Drawers

↓

Level 5

Overlays
```

---

# Icon Tokens

Icon sizes

| Token | Size |
|--------|------:|
| icon.xs | 12 px |
| icon.sm | 16 px |
| icon.md | 20 px |
| icon.lg | 24 px |
| icon.xl | 32 px |

Supported icon families

- Material Symbols
- Heroicons
- Lucide
- Custom EVOXA Icons

---

# Motion Tokens

Animation durations

| Token | Duration |
|--------|----------|
| motion.instant | 0 ms |
| motion.fast | 150 ms |
| motion.normal | 250 ms |
| motion.slow | 400 ms |

---

# Easing Tokens

```yaml
ease.linear
ease.standard
ease.accelerate
ease.decelerate
ease.emphasized
```

---

# Opacity Tokens

| Token | Value |
|--------|------:|
| opacity.0 | 0% |
| opacity.25 | 25% |
| opacity.50 | 50% |
| opacity.75 | 75% |
| opacity.100 | 100% |

---

# Layout Tokens

Grid

```text
Desktop

12 Columns

Tablet

8 Columns

Mobile

4 Columns
```

Container widths

| Token | Width |
|--------|-------|
| container.sm | 640 px |
| container.md | 768 px |
| container.lg | 1024 px |
| container.xl | 1280 px |
| container.2xl | 1440 px |

---

# Z-Index Tokens

```yaml
z.base
z.dropdown
z.sticky
z.drawer
z.modal
z.toast
z.tooltip
```

---

# Component Tokens

Example

```yaml
button.primary.background

button.primary.text

button.primary.hover

button.primary.disabled

button.border.radius

button.padding
```

Every Marketplace component references semantic tokens instead of raw values.

---

# Widget Tokens

Widgets inherit from component tokens while exposing additional settings.

Examples

```yaml
widget.card.padding

widget.card.radius

widget.header.height

widget.chart.colors

widget.background
```

---

# Theme Support

Supported themes

- Light
- Dark
- High Contrast
- Enterprise Branding
- Custom Themes

Theme switching occurs without recompilation.

---

# Accessibility Tokens

Supports

- High Contrast Colors
- Reduced Motion
- Focus Rings
- Minimum Font Sizes
- Color Blind Safe Palette

Accessibility tokens override visual tokens when necessary.

---

# Marketplace Branding

Organizations may override

- Primary Brand Color
- Logo
- Accent Color
- Typography
- Border Radius
- Dashboard Style

Without modifying Marketplace functionality.

---

# Token Naming Convention

```text
category.property.variant.state
```

Examples

```yaml
color.text.primary

button.primary.background

card.surface.border

navigation.sidebar.background

widget.chart.axis

marketplace.header.height
```

---

# Design Token Example

```yaml
color:
  text:
    primary: "#111827"

space:
  md: 16

radius:
  lg: 12

shadow:
  md: 0 4px 8px rgba(0,0,0,.08)
```

---

# Performance Goals

| Metric | Target |
|---------|--------|
| Token Resolution | <1 ms |
| Theme Switch | <100 ms |
| CSS Variables | 100% |
| Hard-coded Styles | 0% |
| Accessibility Compliance | WCAG 2.2 AA |

---

# Repository Structure

```text
14-design-tokens/
├── colors/
├── typography/
├── spacing/
├── shadows/
├── borders/
├── radius/
├── motion/
├── icons/
├── layouts/
├── themes/
├── accessibility/
├── branding/
├── assets/
└── metadata.yml
```

---

# Standard Visual Artifacts

## Token Hierarchy

```text
Core

↓

Semantic

↓

Component

↓

Widget

↓

Application
```

---

## Theme Architecture

```text
Light

↓

Dark

↓

Enterprise

↓

Custom
```

---

## Color System

```text
Brand

↓

Semantic

↓

Component

↓

State
```

---

## Typography Scale

```text
Display

↓

Heading

↓

Body

↓

Caption
```

---

## Layout Grid

```text
Desktop

↓

Tablet

↓

Mobile
```

---

# Design Token Inventory

| Category | Estimated Tokens |
|-----------|-----------------:|
| Colors | 120 |
| Typography | 40 |
| Spacing | 25 |
| Borders | 20 |
| Radius | 10 |
| Shadows | 15 |
| Motion | 30 |
| Icons | 25 |
| Layout | 40 |
| Accessibility | 30 |
| **Total Design Tokens** | **355+** |

---

# Visual Source Files

```text
artifacts/
└── design-tokens/
    ├── token-hierarchy.drawio
    ├── color-system.drawio
    ├── typography-scale.drawio
    ├── spacing-system.drawio
    ├── theme-architecture.drawio
    ├── accessibility.drawio
    ├── layout-grid.drawio
    ├── mermaid/
    │   ├── hierarchy.mmd
    │   ├── colors.mmd
    │   ├── typography.mmd
    │   ├── themes.mmd
    │   ├── spacing.mmd
    │   ├── layout.mmd
    │   └── accessibility.mmd
    └── exports/
        ├── design-tokens.svg
        ├── design-tokens.png
        └── design-tokens.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Chapter 09 — Layout Architecture | Layout Tokens |
| Chapter 11 — Design Principles | Visual Language |
| Chapter 12 — Component Catalog | Component Tokens |
| Chapter 13 — Widget Catalog | Widget Tokens |
| Chapter 15 — Responsive Design | Responsive Tokens |
| Chapter 25 — Accessibility & Internationalization | Accessibility Overrides |

---

# Acceptance Criteria

This chapter is complete when:

- Core, semantic, component and widget design tokens are fully documented.
- Color, typography, spacing, motion, elevation, layout and accessibility token systems are defined.
- Theme architecture, enterprise branding and naming conventions are specified.
- Repository structure, token inventory, visual artifacts and traceability are complete.
- The Design Tokens chapter provides a scalable visual foundation that guarantees consistency, maintainability and themeability across every Marketplace interface.

---

# Key Takeaways

- The EVOXA Marketplace uses a semantic Design Token Architecture to eliminate hard-coded visual values and ensure consistency across all interfaces.
- Every component and widget inherits its visual properties from reusable tokens, enabling rapid evolution, theming and enterprise branding.
- Accessibility, responsiveness and AI-specific visual cues are integrated directly into the token system rather than treated as exceptions.
- This token architecture establishes the visual foundation that supports long-term scalability of the Marketplace and the broader EVOXA Design System.

---

# Next Chapter

**Chapter 15 — Responsive Design**

The next chapter defines how the EVOXA Marketplace adapts across desktop, tablet, mobile and large-screen environments, including responsive layouts, adaptive navigation, breakpoint behavior and device-specific interaction patterns.
