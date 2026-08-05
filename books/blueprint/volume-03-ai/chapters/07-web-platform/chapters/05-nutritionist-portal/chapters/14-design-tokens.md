---
document_id: BP-0003-V3-C07-05-14
chapter_id: CH-05-NUTRITION-14
feature_pack: FP-NUTRITION-0000
title: Design Tokens
version: 1.0.0
status: Draft
owner: Design System Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-04
---

# Chapter 14 — Design Tokens

> *The Design Tokens chapter defines the atomic visual properties that ensure consistency across every interface, component and widget of the Nutritionist Portal and the broader EVOXA Design System.*

---

# Executive Summary

Design Tokens represent the smallest reusable visual decisions of the EVOXA Design System.

Rather than hardcoding visual values into components, tokens provide centralized definitions for colors, typography, spacing, sizing, borders, shadows, motion and semantic states.

Every interface within the Nutritionist Portal consumes these tokens to ensure consistency, maintainability and scalability.

---

# Objectives

The Design Token System shall:

- Centralize visual properties.
- Enable theme support.
- Improve consistency.
- Simplify maintenance.
- Support accessibility.
- Scale across portals.
- Enable automated design synchronization.

---

# Token Philosophy

Every visual property shall be represented as a token.

Components never define colors, spacing or typography directly.

Instead they consume standardized tokens.

---

# Token Hierarchy

```text
Primitive Tokens

↓

Semantic Tokens

↓

Component Tokens

↓

Widget Tokens

↓

Screen Themes
```

---

# Token Categories

The EVOXA Design System defines:

- Color Tokens
- Typography Tokens
- Spacing Tokens
- Size Tokens
- Border Tokens
- Radius Tokens
- Shadow Tokens
- Motion Tokens
- Elevation Tokens
- Opacity Tokens
- Icon Tokens
- Z-Index Tokens

---

# Color Tokens

## Brand Colors

Examples:

```text
color.brand.primary

color.brand.secondary

color.brand.accent
```

---

## Neutral Colors

```text
color.gray.50

color.gray.100

...

color.gray.900
```

---

## Semantic Colors

```text
color.success

color.warning

color.error

color.info
```

---

## Nutrition Colors

Used for nutritional indicators.

Examples:

```text
color.nutrition.protein

color.nutrition.carbohydrate

color.nutrition.fat

color.nutrition.water

color.nutrition.fiber

color.nutrition.vitamins
```

---

## Human Digital Twin Colors

```text
color.digitalTwin.health

color.digitalTwin.behavior

color.digitalTwin.prediction

color.digitalTwin.alert
```

---

## AI Colors

```text
color.ai.primary

color.ai.background

color.ai.recommendation

color.ai.confidence
```

---

# Typography Tokens

Typography includes:

```text
font.family.primary

font.family.monospace

font.size.xs

font.size.sm

font.size.md

font.size.lg

font.size.xl

font.weight.light

font.weight.regular

font.weight.medium

font.weight.bold

line.height.default

letter.spacing.default
```

---

# Spacing Tokens

Standard spacing scale:

```text
space.0

space.2

space.4

space.8

space.12

space.16

space.24

space.32

space.48

space.64
```

Spacing follows an 8-point grid.

---

# Size Tokens

Examples:

```text
size.icon.sm

size.icon.md

size.icon.lg

size.avatar.sm

size.avatar.md

size.avatar.lg

size.sidebar.width

size.header.height
```

---

# Border Tokens

```text
border.width.none

border.width.sm

border.width.md

border.width.lg
```

---

# Radius Tokens

```text
radius.none

radius.sm

radius.md

radius.lg

radius.xl

radius.full
```

---

# Shadow Tokens

```text
shadow.sm

shadow.md

shadow.lg

shadow.xl
```

Used to indicate elevation rather than decoration.

---

# Elevation Tokens

```text
elevation.0

elevation.1

elevation.2

elevation.3

elevation.4
```

---

# Motion Tokens

Animations are standardized.

Examples:

```text
motion.duration.fast

motion.duration.normal

motion.duration.slow

motion.easing.standard

motion.easing.emphasized
```

---

# Opacity Tokens

```text
opacity.disabled

opacity.overlay

opacity.hover

opacity.focus
```

---

# Icon Tokens

```text
icon.size.sm

icon.size.md

icon.size.lg

icon.stroke.default
```

---

# Z-Index Tokens

```text
z.modal

z.tooltip

z.dropdown

z.toast

z.overlay
```

---

# Semantic Tokens

Semantic tokens map business meaning.

Examples:

```text
surface.background

surface.card

surface.sidebar

surface.workspace

text.primary

text.secondary

text.disabled

border.default
```

---

# Component Tokens

Examples:

```text
button.primary.background

button.primary.color

input.border

input.focus

table.header.background

card.shadow
```

Component tokens reference semantic tokens.

---

# Widget Tokens

Widget-specific examples:

```text
widget.kpi.background

widget.ai.border

widget.timeline.color

widget.analytics.chart
```

---

# Theme Support

Supported themes:

- Light
- Dark
- High Contrast
- Organization Themes

Themes override semantic tokens without modifying components.

---

# Accessibility

Tokens support:

- WCAG 2.2 AA
- Contrast validation
- Color blindness compatibility
- Reduced motion
- Large text scaling

---

# Design Tool Integration

Tokens synchronize with:

- Figma
- Storybook
- Angular Library
- React Library
- Documentation

Synchronization is automated whenever possible.

---

# Versioning

Every token includes:

- Token ID
- Category
- Version
- Status
- Owner
- Deprecation status

Breaking changes require governance approval.

---

# Token Naming Convention

Examples:

```text
color.brand.primary

space.16

font.size.md

radius.lg

button.primary.background
```

Names are:

- Human-readable
- Consistent
- Hierarchical
- Stable

---

# Repository Structure

```text
design-tokens/
├── colors/
├── typography/
├── spacing/
├── sizing/
├── borders/
├── radius/
├── shadows/
├── motion/
├── icons/
├── semantic/
├── themes/
├── documentation/
└── metadata.yml
```

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
```

---

## Color System

```text
Brand

↓

Semantic

↓

Component
```

---

## Theme Resolution

```text
Theme

↓

Semantic Tokens

↓

Components
```

---

## Component Consumption

```text
Component

↓

Semantic Token

↓

Primitive Token
```

---

# Visual Source Files

```text
artifacts/
└── design-tokens/
    ├── token-hierarchy.drawio
    ├── color-system.drawio
    ├── typography.drawio
    ├── themes.drawio
    ├── token-resolution.drawio
    ├── component-consumption.drawio
    ├── mermaid/
    │   ├── hierarchy.mmd
    │   ├── colors.mmd
    │   ├── themes.mmd
    │   ├── typography.mmd
    │   └── tokens.mmd
    └── exports/
        ├── design-tokens.svg
        ├── design-tokens.png
        └── design-tokens.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Design Principles | Chapter 11 |
| Component Catalog | Chapter 12 |
| Widget Catalog | Chapter 13 |
| Responsive Design | Chapter 15 |
| Accessibility & Internationalization | Chapter 25 |

---

# Acceptance Criteria

This chapter is complete when:

- Token hierarchy is documented.
- Primitive and semantic tokens are defined.
- Component and widget tokens are specified.
- Theme support is documented.
- Accessibility requirements are incorporated.
- Naming conventions and versioning rules are established.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- Design Tokens provide the atomic visual foundation for every interface within the Nutritionist Portal and the broader EVOXA ecosystem.
- A layered token hierarchy enables centralized control over colors, typography, spacing, motion and semantic meaning while preserving flexibility for themes and organizational branding.
- Components and widgets consume tokens rather than hardcoded values, ensuring consistency, maintainability and scalability.
- Standardized naming, governance and automated synchronization with design and development tools create a robust foundation for the long-term evolution of the EVOXA Design System.

---

# Next Chapter

**Chapter 15 — Responsive Design**

This chapter defines the responsive design strategy of the Nutritionist Portal, including adaptive layouts, breakpoints, device-specific behaviors, touch interactions, responsive components and cross-device user experience principles.
