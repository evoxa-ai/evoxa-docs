---
document_id: BP-0003-V3-C13-14
chapter_id: CH-13-WF-14
feature_pack: FP-WORKFLOW-0001
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

> *The Design Tokens chapter defines the visual foundation of the EVOXA Workflow Platform. Design Tokens are the atomic values that drive colors, typography, spacing, sizing, elevation, borders, animations and semantic states, ensuring every interface remains visually consistent, scalable and maintainable across the entire ecosystem.*

---

# Executive Summary

The Workflow Platform shares the EVOXA Enterprise Design System.

Instead of hardcoding visual properties, every UI element consumes standardized Design Tokens.

This approach enables:

- Consistent visual identity
- Theme switching
- Dark Mode
- Enterprise Branding
- Accessibility
- Cross-platform consistency
- Simplified maintenance

Design Tokens are the lowest abstraction layer of the design system.

---

# Objectives

The Design Tokens shall:

- Standardize all visual values.
- Enable theme customization.
- Improve maintainability.
- Support accessibility.
- Reduce UI inconsistencies.
- Enable multi-brand deployments.
- Support responsive interfaces.

---

# Token Architecture

```text
Foundation Tokens

↓

Semantic Tokens

↓

Component Tokens

↓

Widget Tokens

↓

Application
```

---

# Token Categories

The Workflow Platform defines:

- Color Tokens
- Typography Tokens
- Spacing Tokens
- Sizing Tokens
- Radius Tokens
- Border Tokens
- Elevation Tokens
- Shadow Tokens
- Motion Tokens
- Opacity Tokens
- Z-Index Tokens
- Icon Tokens
- Workflow Tokens
- AI Tokens

---

# Naming Convention

Pattern

```text
<Category>.<Group>.<Variant>
```

Examples

```text
color.primary.500

spacing.md

radius.lg

shadow.level2

workflow.node.success

ai.assistant.primary
```

---

# Color System

## Primary Colors

```yaml
Primary 50
Primary 100
Primary 200
Primary 300
Primary 400
Primary 500
Primary 600
Primary 700
Primary 800
Primary 900
```

Primary 500 represents the EVOXA brand color.

---

# Semantic Colors

| Token | Purpose |
|---------|----------|
| Success | Completed workflows |
| Warning | Pending approvals |
| Error | Failed execution |
| Info | Notifications |
| AI | AI interactions |
| Neutral | General UI |

---

# Workflow Colors

Workflow-specific colors

| Workflow State | Token |
|---------------|-------|
| Draft | Gray |
| Published | Blue |
| Running | Green |
| Waiting | Orange |
| Completed | Emerald |
| Failed | Red |
| Cancelled | Dark Gray |
| Archived | Slate |

---

# AI Colors

Dedicated colors for AI interfaces

```text
AI Purple

↓

Reasoning

↓

Suggestions

↓

Generation

↓

Analysis
```

---

# Typography Tokens

Hierarchy

```text
Display

Heading

Title

Subtitle

Body

Caption

Code
```

---

# Font Families

```yaml
Primary:
  Inter

Monospace:
  JetBrains Mono

Code:
  Fira Code
```

---

# Font Sizes

| Token | Size |
|---------|------|
| xs | 12 px |
| sm | 14 px |
| md | 16 px |
| lg | 18 px |
| xl | 20 px |
| 2xl | 24 px |
| 3xl | 30 px |
| 4xl | 36 px |

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
Tight

Normal

Relaxed

Loose
```

---

# Letter Spacing

```yaml
Tighter

Normal

Wide
```

---

# Spacing Scale

Based on an 8-point system.

```text
4

8

12

16

24

32

40

48

64

96
```

---

# Border Radius

| Token | Radius |
|---------|---------|
| xs | 2 px |
| sm | 4 px |
| md | 8 px |
| lg | 12 px |
| xl | 16 px |
| full | 9999 px |

---

# Border Width

```yaml
Thin

Normal

Medium

Thick
```

---

# Elevation Tokens

```text
Level 0

↓

Level 1

↓

Level 2

↓

Level 3

↓

Modal

↓

Overlay
```

---

# Shadow Tokens

Examples

```yaml
shadow.sm

shadow.md

shadow.lg

shadow.xl
```

---

# Opacity Tokens

```yaml
100%

90%

75%

50%

25%

10%
```

---

# Motion Tokens

Animation durations

| Token | Duration |
|---------|----------|
| Fast | 100 ms |
| Normal | 200 ms |
| Medium | 300 ms |
| Slow | 500 ms |

---

# Easing Tokens

```yaml
Ease In

Ease Out

Ease In Out

Linear
```

---

# Z-Index Tokens

```yaml
Background

Base

Dropdown

Sticky

Modal

Popover

Tooltip

Notification
```

---

# Icon Tokens

Sizes

```yaml
16

20

24

32

40

48
```

Supported families

- Material Symbols
- Heroicons
- Lucide
- EVOXA Workflow Icons

---

# Grid Tokens

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

# Breakpoints

| Device | Width |
|---------|------:|
| xs | <576 px |
| sm | ≥576 px |
| md | ≥768 px |
| lg | ≥992 px |
| xl | ≥1200 px |
| 2xl | ≥1440 px |

---

# Workflow Node Tokens

Node dimensions

```yaml
Width:
  220 px

Height:
  96 px
```

Spacing

```yaml
Horizontal:
  64 px

Vertical:
  48 px
```

---

# Connection Tokens

```yaml
Stroke Width:
  2 px

Selected:
  4 px

Animated:
  Yes
```

---

# AI Tokens

Dedicated tokens

```yaml
AI Message

AI Suggestion

AI Prompt

AI Confidence

AI Badge

AI Glow
```

---

# Status Tokens

```yaml
Draft

Running

Paused

Completed

Cancelled

Failed

Retrying
```

Each status defines:

- Color
- Icon
- Border
- Background
- Badge

---

# Accessibility Tokens

Supports

- High Contrast
- Reduced Motion
- Focus Ring
- Accessible Colors
- Minimum Contrast Ratio 4.5:1

---

# Theme Tokens

Themes

```text
Light

↓

Dark

↓

High Contrast

↓

Enterprise Custom
```

---

# Enterprise Branding

Organizations may customize

- Primary Color
- Logo
- Typography
- Corner Radius
- Icons
- Dashboard Colors

without affecting platform functionality.

---

# Token Storage

Example

```json
{
  "color.primary.500": "#4F46E5",
  "spacing.md": "16px",
  "radius.lg": "12px"
}
```

---

# Token Versioning

Pattern

```text
Design Tokens

↓

v1

↓

v2

↓

v3
```

Backwards compatibility is maintained whenever possible.

---

# Token Validation

Every release validates

- Naming
- Accessibility
- Duplicates
- Unused Tokens
- Contrast
- Theme Support

---

# Performance

The design system shall

- Load tokens once
- Support runtime theme switching
- Cache tokens
- Avoid duplicated values

---

# Repository Structure

```text
14-design-tokens/
├── colors/
├── typography/
├── spacing/
├── sizing/
├── borders/
├── shadows/
├── motion/
├── icons/
├── workflow/
├── ai/
├── themes/
├── accessibility/
├── assets/
└── metadata.yml
```

---

# Standard Visual Artifacts

## Token Hierarchy

```text
Foundation

↓

Semantic

↓

Component

↓

Application
```

---

## Color System

```text
Primary

↓

Semantic

↓

Workflow

↓

AI
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

## Theme Architecture

```text
Light

↓

Dark

↓

Enterprise
```

---

## Workflow Tokens

```text
Workflow

↓

Nodes

↓

Connections

↓

Execution
```

---

# Token Inventory

| Category | Estimated Tokens |
|-----------|-----------------:|
| Colors | 180 |
| Typography | 60 |
| Spacing | 40 |
| Radius | 20 |
| Borders | 20 |
| Shadows | 25 |
| Motion | 30 |
| Icons | 80 |
| Workflow | 70 |
| AI | 40 |
| Themes | 60 |
| **Total** | **625+** |

---

# Visual Source Files

```text
artifacts/
└── design-tokens/
    ├── color-system.drawio
    ├── typography.drawio
    ├── spacing.drawio
    ├── workflow-tokens.drawio
    ├── ai-tokens.drawio
    ├── themes.drawio
    ├── token-hierarchy.drawio
    ├── mermaid/
    │   ├── tokens.mmd
    │   ├── colors.mmd
    │   ├── typography.mmd
    │   ├── workflow.mmd
    │   ├── ai.mmd
    │   ├── themes.mmd
    │   └── hierarchy.mmd
    └── exports/
        ├── design-tokens.svg
        ├── design-tokens.png
        └── design-tokens.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Chapter 09 — Layout Architecture | Layout Foundation |
| Chapter 11 — Design Principles | Visual Language |
| Chapter 12 — Component Catalog | Component Styling |
| Chapter 13 — Widget Catalog | Widget Themes |
| Chapter 15 — Responsive Design | Responsive Tokens |
| Chapter 25 — Accessibility & Internationalization | Accessible Colors |
| EVOXA Enterprise Design System | Global Token Library |

---

# Acceptance Criteria

This chapter is complete when:

- The complete design token architecture is defined.
- Foundation, semantic and component tokens are documented.
- Colors, typography, spacing, motion, workflow and AI tokens are specified.
- Theme management, branding, accessibility and versioning strategies are documented.
- Repository structure, visual artifacts, token inventory and traceability are complete.
- The Design Tokens provide the single source of truth for all visual values used throughout the EVOXA Workflow Platform.

---

# Key Takeaways

- Design Tokens are the foundational layer of the EVOXA Design System, ensuring consistency across every screen, component and workflow experience.
- Workflow-specific and AI-specific tokens extend traditional design systems with semantics tailored to intelligent automation platforms.
- Token-based theming enables enterprise branding, accessibility and runtime customization without changing application code.
- This chapter establishes a scalable visual foundation that supports long-term evolution, multi-tenant customization and cross-platform consistency for the Workflow Platform.

---

# Next Chapter

**Chapter 15 — Responsive Design**

The next chapter defines how the Workflow Platform adapts to desktop, laptop, tablet and mobile devices, including responsive layouts, adaptive components, workflow editing constraints, touch interactions and cross-device user experiences.
