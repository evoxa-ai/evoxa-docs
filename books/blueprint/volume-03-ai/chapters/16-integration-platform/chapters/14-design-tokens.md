---
document_id: BP-0003-V3-C16-14
chapter_id: CH-16-14
feature_pack: FP-INTEGRATION-0001
title: Design Tokens
version: 1.0.0
status: Draft
owner: Enterprise Design System Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-07
---

# 14 — Design Tokens

> *The Design Tokens chapter defines the foundational visual language of the EVOXA Integration Platform. Tokens establish a single source of truth for colors, typography, spacing, sizing, elevation, motion, icons, borders, shadows and responsive behavior, ensuring visual consistency across all applications, widgets and enterprise modules.*

---

# Executive Summary

Design Tokens are the atomic elements of the EVOXA Design System.

Every interface, component and widget derives its appearance from standardized tokens instead of hardcoded values.

This approach enables:

- Consistency
- Scalability
- Themeability
- Accessibility
- Maintainability
- Multi-brand Support

---

# Design Philosophy

The token system follows one principle:

> **Design Once. Apply Everywhere.**

Tokens represent design decisions—not implementation details.

---

# Token Architecture

```text
Primitive Tokens

↓

Semantic Tokens

↓

Component Tokens

↓

Widget Tokens

↓

Application Themes
```

---

# Token Categories

| Category | Tokens |
|-----------|--------:|
| Colors | 120 |
| Typography | 60 |
| Spacing | 32 |
| Borders | 24 |
| Radius | 16 |
| Shadows | 24 |
| Elevation | 12 |
| Motion | 30 |
| Icons | 350 |
| Breakpoints | 8 |
| Z-Index | 18 |
| Layout | 40 |
| **Total Tokens** | **734+** |

---

# Color System

## Brand Colors

```yaml
Primary:
  50:  #F3F8FF
 100:  #D9E9FF
 200:  #B4D2FF
 300:  #8AB8FF
 400:  #5C98FF
 500:  #2563EB
 600:  #1D4ED8
 700:  #1E40AF
 800:  #1E3A8A
 900:  #172554
```

---

## Secondary

```yaml
500: #14B8A6
```

---

## Accent

```yaml
500: #7C3AED
```

---

# Semantic Colors

| Token | Usage |
|---------|--------|
| Success | Operations completed |
| Warning | User attention |
| Error | Failures |
| Information | Informational content |
| AI | AI-generated content |
| Disabled | Inactive controls |

---

## Status Palette

```yaml
Success:
 500: #16A34A

Warning:
 500: #EA580C

Danger:
 500: #DC2626

Info:
 500: #2563EB

AI:
 500: #7C3AED
```

---

# Neutral Palette

```yaml
Gray-50

↓

Gray-100

↓

Gray-200

↓

Gray-300

↓

Gray-400

↓

Gray-500

↓

Gray-600

↓

Gray-700

↓

Gray-800

↓

Gray-900
```

---

# Background Tokens

```yaml
surface.primary

surface.secondary

surface.tertiary

surface.overlay

surface.sidebar

surface.card

surface.modal

surface.dashboard
```

---

# Text Tokens

```yaml
text.primary

text.secondary

text.tertiary

text.disabled

text.inverse

text.link
```

---

# Border Tokens

```yaml
border.default

border.focus

border.active

border.error

border.success
```

---

# Typography

Primary Font

```text
Inter
```

Fallback

```text
Segoe UI

Roboto

Helvetica

Arial
```

---

# Typography Scale

| Token | Size |
|---------|-----:|
| Display XL | 64px |
| Display L | 56px |
| H1 | 40px |
| H2 | 32px |
| H3 | 28px |
| H4 | 24px |
| H5 | 20px |
| H6 | 18px |
| Body L | 18px |
| Body | 16px |
| Small | 14px |
| Caption | 12px |

---

# Font Weights

```yaml
Light: 300

Regular: 400

Medium: 500

SemiBold: 600

Bold: 700
```

---

# Line Heights

```yaml
Compact: 1.2

Default: 1.5

Comfortable: 1.75
```

---

# Letter Spacing

```yaml
Tight

Normal

Wide
```

---

# Spacing Scale

```yaml
0: 0px
1: 4px
2: 8px
3: 12px
4: 16px
5: 20px
6: 24px
8: 32px
10: 40px
12: 48px
16: 64px
20: 80px
24: 96px
32: 128px
```

Base spacing unit

```text
4 px
```

---

# Grid System

Desktop

```text
12 Columns
```

Tablet

```text
8 Columns
```

Mobile

```text
4 Columns
```

---

# Border Radius

```yaml
xs: 2px

sm: 4px

md: 8px

lg: 12px

xl: 16px

2xl: 24px

full: 9999px
```

---

# Border Width

```yaml
hairline

1px

2px

4px
```

---

# Shadows

```yaml
xs

sm

md

lg

xl

2xl
```

Example

```css
shadow-md

0 4px 12px rgba(0,0,0,.12)
```

---

# Elevation Levels

| Level | Usage |
|---------|-------|
| 0 | Background |
| 1 | Cards |
| 2 | Menus |
| 3 | Drawers |
| 4 | Modals |
| 5 | Dialogs |
| 6 | Notifications |

---

# Icon System

Library

```text
Material Symbols Rounded
```

Categories

- Navigation
- APIs
- Connectors
- AI
- MCP
- Workflow
- Monitoring
- Security
- Administration
- Marketplace

---

# Icon Sizes

```yaml
Small: 16px

Default: 20px

Medium: 24px

Large: 32px

XL: 48px
```

---

# Motion Tokens

Animation Duration

```yaml
Fast: 100ms

Normal: 200ms

Slow: 350ms
```

---

# Easing

```yaml
ease-in

ease-out

ease-in-out

standard
```

---

# Motion Principles

Animations must be

- Functional
- Fast
- Subtle
- Accessible

---

# Z-Index Scale

```yaml
Base: 0

Dropdown: 100

Sticky: 200

Drawer: 300

Modal: 400

Popover: 500

Tooltip: 600

Toast: 700

Emergency Overlay: 1000
```

---

# Breakpoints

```yaml
xs: 0

sm: 576px

md: 768px

lg: 992px

xl: 1200px

2xl: 1440px

3xl: 1600px
```

---

# Layout Tokens

```yaml
header.height

sidebar.width

sidebar.collapsed

footer.height

content.maxWidth

drawer.width
```

---

# Form Tokens

Defines

- Input Height
- Label Gap
- Validation Colors
- Placeholder Colors
- Focus Ring

---

# Table Tokens

Defines

- Row Height
- Header Height
- Border Color
- Hover Color
- Selected Row
- Density

---

# Chart Tokens

Includes

- Series Colors
- Grid
- Axis
- Tooltip
- Legend

---

# Dashboard Tokens

Defines

- Card Padding
- Widget Gap
- KPI Size
- Chart Height
- Alert Colors

---

# AI Tokens

Unique tokens for AI features

```yaml
ai.background

ai.border

ai.icon

ai.message

ai.recommendation

ai.confidence
```

---

# Notification Tokens

Defines

- Toast Colors
- Duration
- Icons
- Priority
- Position

---

# Theme Support

Supported themes

```text
Light

Dark

High Contrast

Enterprise Branding

Partner Branding
```

---

# CSS Variables

Example

```css
:root {

--ev-primary-500:#2563EB;

--ev-success:#16A34A;

--ev-spacing-4:16px;

--ev-radius-md:8px;

--ev-font-body:16px;

}
```

---

# Angular Token Package

```text
@evoxa/design-tokens
```

Exports

- SCSS Variables
- CSS Variables
- TypeScript Tokens
- JSON Tokens

---

# Tailwind Mapping

```js
theme: {

colors: tokens.colors,

spacing: tokens.spacing,

fontSize: tokens.typography,

borderRadius: tokens.radius

}
```

---

# Token Governance

Every token change requires

- Design Review
- Accessibility Validation
- UX Approval
- Regression Testing

---

# Token Versioning

Semantic Versioning

```text
MAJOR.MINOR.PATCH
```

Example

```text
2.4.1
```

---

# Accessibility Compliance

Every token satisfies

- WCAG 2.2 AA
- Color contrast
- Keyboard visibility
- Focus indicators
- Reduced motion

---

# Performance Goals

| Metric | Target |
|----------|--------|
| Theme Switch | <100 ms |
| Token Resolution | <5 ms |
| CSS Variables | 100% |
| Render Cost | Minimal |

---

# Repository Structure

```text
14-design-tokens/
├── colors/
├── typography/
├── spacing/
├── radius/
├── borders/
├── shadows/
├── elevation/
├── motion/
├── icons/
├── layouts/
├── themes/
├── css/
├── scss/
├── json/
├── angular/
├── tailwind/
├── diagrams/
└── metadata.yml
```

---

# Token Inventory

| Area | Assets |
|------|--------:|
| Color Tokens | 120 |
| Typography Tokens | 60 |
| Layout Tokens | 40 |
| Motion Tokens | 30 |
| Radius Tokens | 16 |
| Border Tokens | 24 |
| Shadow Tokens | 24 |
| Icon Tokens | 350 |
| Theme Definitions | 12 |
| CSS Variables | 500+ |
| **Total Design Assets** | **1,176+** |

---

# Architecture Decision Records (ADRs)

| ADR | Decision |
|------|----------|
| ADR-161401 | Token-Driven Design System |
| ADR-161402 | CSS Variable Strategy |
| ADR-161403 | Semantic Color Architecture |
| ADR-161404 | Multi-Theme Support |
| ADR-161405 | Enterprise Typography Standard |
| ADR-161406 | Responsive Token Framework |
| ADR-161407 | AI Visual Identity |
| ADR-161408 | Design Token Governance |

---

# Standard Visual Artifacts

## Token Hierarchy

```text
Primitive

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

## Color System

```text
Brand

↓

Semantic

↓

Components
```

---

## Typography Scale

```text
Display

↓

Headings

↓

Body

↓

Caption
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

Partner
```

---

## Token Distribution

```text
JSON

↓

CSS

↓

SCSS

↓

Angular

↓

Tailwind
```

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
    ├── css-variable-map.drawio
    ├── token-lifecycle.drawio
    ├── mermaid/
    │   ├── hierarchy.mmd
    │   ├── colors.mmd
    │   ├── typography.mmd
    │   ├── themes.mmd
    │   ├── spacing.mmd
    │   ├── lifecycle.mmd
    │   └── distribution.mmd
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
| Chapter 11 — Design Principles | Visual Standards |
| Chapter 12 — Component Catalog | Component Styling |
| Chapter 13 — Widget Catalog | Widget Themes |
| Chapter 15 — Responsive Design | Responsive Breakpoints |
| EVOXA Design System | Global Tokens |
| Angular UI Kit | CSS Variables |
| Tailwind Configuration | Utility Mapping |

---

# Acceptance Criteria

This chapter is complete when:

- Primitive, semantic and component-level design tokens are fully documented.
- Color palettes, typography, spacing, motion, elevation, layout and theme tokens are standardized.
- CSS variables, Angular package exports and Tailwind mappings are defined.
- Accessibility, governance, versioning and performance objectives are specified.
- Repository structure, inventories, ADRs, visual artifacts and traceability are complete.
- The Design Tokens chapter provides the single source of truth for all visual styling across the EVOXA Integration Platform.

---

# Key Takeaways

- Design Tokens provide the foundational visual language that guarantees consistency across every screen, component and widget.
- A layered token architecture enables scalable theming, enterprise branding and long-term maintainability without duplicating design decisions.
- Standardized exports for CSS, SCSS, Angular and Tailwind simplify implementation while preserving alignment with the EVOXA Design System.
- This token framework ensures that future UI evolution can occur rapidly while maintaining accessibility, visual coherence and enterprise-grade quality.

---

# Next Section

**15 — Responsive Design**

The next chapter defines the responsive behavior of the Integration Platform, including adaptive layouts, breakpoint strategies, mobile experiences, responsive components and cross-device usability standards.
