---
document_id: BP-0003-V3-C07-05-12
chapter_id: CH-05-NUTRITION-12
feature_pack: FP-NUTRITION-0000
title: Component Catalog
version: 1.0.0
status: Draft
owner: Design System Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-04
---

# Chapter 12 — Component Catalog

> *The Component Catalog defines the reusable user interface components of the Nutritionist Portal, establishing consistent behaviors, accessibility, AI integration and implementation guidelines across the EVOXA ecosystem.*

---

# Executive Summary

The Nutritionist Portal is built using reusable UI components derived from the EVOXA Design System.

Each component follows standardized behavior, styling, accessibility and interaction rules to ensure a consistent user experience across all professional portals.

The catalog provides the single source of truth for designers, frontend engineers and QA teams.

---

# Objectives

The Component Catalog shall:

- Standardize reusable UI components.
- Reduce design duplication.
- Improve development consistency.
- Support accessibility.
- Enable AI-enhanced interfaces.
- Simplify maintenance.
- Ensure scalability.

---

# Component Philosophy

Every component shall be:

- Reusable.
- Configurable.
- Accessible.
- Responsive.
- Testable.
- Theme-aware.
- AI-ready.

---

# Component Classification

Components are organized into:

- Layout Components
- Navigation Components
- Form Components
- Data Components
- Visualization Components
- Feedback Components
- AI Components
- Human Digital Twin Components
- Collaboration Components

---

# Layout Components

## CMP-001 Page Container

Purpose

Defines the main content area.

Properties

- Full Width
- Fixed Width
- Scrollable
- Responsive

---

## CMP-002 Workspace

Supports:

- Client Workspace
- Assessment Workspace
- Meal Planning Workspace
- Analytics Workspace

---

## CMP-003 Section

Groups related information.

Supports:

- Header
- Body
- Footer

---

# Navigation Components

## CMP-010 Sidebar

Features

- Collapsible
- Favorites
- Search
- Icons
- Notifications

---

## CMP-011 Breadcrumb

Supports contextual navigation.

---

## CMP-012 Tabs

Variants

- Horizontal
- Vertical
- Scrollable

---

## CMP-013 Stepper

Used for:

- Assessments
- Meal Planning
- AI Review
- Client Onboarding

---

# Form Components

## CMP-020 Text Field

States

- Default
- Focus
- Error
- Disabled
- Success

---

## CMP-021 Number Field

Supports:

- Measurements
- Calories
- Portions
- Weight

---

## CMP-022 Date Picker

Used for:

- Appointments
- Assessments
- Meal schedules

---

## CMP-023 Select

Supports:

- Single selection
- Multiple selection
- Searchable

---

## CMP-024 Switch

Used for:

- Preferences
- Notifications
- AI options

---

## CMP-025 Checkbox

Supports grouped selections.

---

## CMP-026 Radio Group

Single-choice options.

---

# Data Components

## CMP-030 Table

Capabilities

- Sorting
- Filtering
- Pagination
- Export
- Selection

---

## CMP-031 Timeline

Displays:

- Assessments
- Meal Plans
- Progress
- AI Events

---

## CMP-032 Card

Used for:

- KPI
- Client Summary
- AI Recommendation
- Reports

---

## CMP-033 Statistic

Displays:

- Calories
- BMI
- Protein
- Water
- Body Fat

---

# Visualization Components

## CMP-040 Line Chart

Applications

- Weight evolution
- Calories
- Measurements

---

## CMP-041 Bar Chart

Applications

- Macronutrients
- Weekly intake
- Compliance

---

## CMP-042 Pie Chart

Applications

- Nutrient distribution
- Food groups

---

## CMP-043 Radar Chart

Applications

- Nutritional profile
- Dietary balance

---

## CMP-044 Progress Ring

Displays goal completion.

---

# Feedback Components

## CMP-050 Toast

Displays temporary notifications.

---

## CMP-051 Alert

Levels

- Success
- Warning
- Error
- Information

---

## CMP-052 Dialog

Supports:

- Confirmation
- Approval
- Delete
- AI Review

---

## CMP-053 Loading Skeleton

Preferred loading indicator.

---

# AI Components

## CMP-060 AI Recommendation Card

Displays:

- Recommendation
- Confidence
- Evidence
- Actions

---

## CMP-061 AI Chat

Supports:

- Conversations
- Suggestions
- References
- History

---

## CMP-062 AI Insight

Displays contextual nutritional insights.

---

## CMP-063 AI Confidence Indicator

Shows confidence percentage and explanation.

---

# Human Digital Twin Components

## CMP-070 Digital Twin Summary

Displays:

- Current status
- Trends
- Health indicators

---

## CMP-071 Prediction Timeline

Shows future nutritional projections.

---

## CMP-072 Nutrition Risk Indicator

Visualizes nutritional risks.

---

# Collaboration Components

## CMP-080 Comment Thread

Supports multidisciplinary discussions.

---

## CMP-081 Mention

Allows user mentions.

---

## CMP-082 Activity Feed

Displays:

- Assessments
- AI actions
- Reports
- Updates

---

# Component States

Every interactive component supports:

- Default
- Hover
- Focus
- Active
- Disabled
- Error
- Loading

---

# Accessibility

All components comply with:

- WCAG 2.2 AA
- Keyboard navigation
- Screen reader compatibility
- Focus visibility
- High contrast

---

# Responsive Behavior

Components adapt to:

- Desktop
- Tablet
- Mobile

Layouts remain consistent.

---

# AI Integration

AI-enabled components provide:

- Explainability
- Confidence
- Alternative recommendations
- Professional approval
- Feedback collection

---

# Design Tokens

Components consume centralized tokens for:

- Colors
- Typography
- Spacing
- Borders
- Shadows
- Motion

Defined in Chapter 14.

---

# Component Metadata

Each component includes:

- Component ID
- Version
- Owner
- Status
- Accessibility
- Documentation
- Figma Reference
- Code Reference

---

# Repository Structure

```text
components/
├── layout/
├── navigation/
├── forms/
├── data/
├── charts/
├── feedback/
├── ai/
├── digital-twin/
├── collaboration/
├── documentation/
└── metadata.yml
```

---

# Standard Visual Artifacts

## Component Hierarchy

```text
Layout

↓

Navigation

↓

Content

↓

Actions
```

---

## Component Relationships

```text
Page

↓

Workspace

↓

Section

↓

Component
```

---

## AI Components

```text
AI

↓

Recommendation

↓

Review

↓

Approval
```

---

## Human Digital Twin

```text
Profile

↓

Prediction

↓

Insight
```

---

# Visual Source Files

```text
artifacts/
└── component-catalog/
    ├── component-hierarchy.drawio
    ├── forms.drawio
    ├── navigation.drawio
    ├── ai-components.drawio
    ├── digital-twin.drawio
    ├── charts.drawio
    ├── mermaid/
    │   ├── hierarchy.mmd
    │   ├── components.mmd
    │   ├── ai.mmd
    │   ├── charts.mmd
    │   └── collaboration.mmd
    └── exports/
        ├── component-catalog.svg
        ├── component-catalog.png
        └── component-catalog.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Screen Catalog | Chapter 08 |
| Layout Architecture | Chapter 09 |
| Navigation Architecture | Chapter 10 |
| Design Principles | Chapter 11 |
| Widget Catalog | Chapter 13 |
| Design Tokens | Chapter 14 |
| Accessibility & Internationalization | Chapter 25 |

---

# Acceptance Criteria

This chapter is complete when:

- Component taxonomy is defined.
- Reusable components are documented.
- Accessibility requirements are specified.
- AI-enabled components are identified.
- Human Digital Twin components are documented.
- Metadata standards are established.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- The Component Catalog establishes the reusable building blocks that power every interface of the Nutritionist Portal.
- Standardized behaviors, accessibility rules and centralized design tokens ensure consistency, maintainability and scalability across the EVOXA ecosystem.
- Specialized AI and Human Digital Twin components enable intelligent, explainable and context-aware professional experiences.
- A shared component architecture allows designers, developers and QA teams to collaborate using a single, versioned source of truth.

---

# Next Chapter

**Chapter 13 — Widget Catalog**

This chapter defines the reusable dashboard widgets and information panels used throughout the Nutritionist Portal, including KPIs, analytics, AI insights, nutritional summaries and configurable workspace widgets.
