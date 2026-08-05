---
document_id: BP-0003-V3-C07-05-13
chapter_id: CH-05-NUTRITION-13
feature_pack: FP-NUTRITION-0000
title: Widget Catalog
version: 1.0.0
status: Draft
owner: Design System Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-04
---

# Chapter 13 — Widget Catalog

> *The Widget Catalog defines the reusable dashboard widgets, analytical panels and intelligent workspace modules used throughout the Nutritionist Portal.*

---

# Executive Summary

Widgets provide high-level business functionality by combining multiple UI components into reusable modules.

Unlike individual components, widgets present complete business information, AI insights and actionable recommendations.

Widgets are configurable, responsive and reusable across dashboards, client workspaces and analytics screens.

---

# Objectives

The Widget Catalog shall:

- Standardize dashboard modules.
- Enable workspace personalization.
- Support AI-assisted experiences.
- Improve information discoverability.
- Encourage component reuse.
- Simplify future customization.
- Maintain consistency across EVOXA.

---

# Widget Philosophy

Every widget shall be:

- Reusable.
- Configurable.
- Responsive.
- Accessible.
- Context-aware.
- AI-ready.
- Data-driven.

---

# Widget Classification

Widgets are grouped into:

- Dashboard Widgets
- Client Widgets
- Assessment Widgets
- Meal Planning Widgets
- Analytics Widgets
- AI Widgets
- Human Digital Twin Widgets
- Collaboration Widgets
- Productivity Widgets

---

# Dashboard Widgets

## WDG-001 Today's Agenda

Displays:

- Scheduled consultations
- Follow-ups
- Reminders
- Calendar integration

---

## WDG-002 Pending Tasks

Displays:

- Pending assessments
- Meal plans awaiting approval
- Messages
- Reports

---

## WDG-003 Client Alerts

Displays:

- Missed appointments
- Low adherence
- Nutritional risks
- AI alerts

---

## WDG-004 KPI Summary

Displays:

- Active clients
- Meal plans created
- Follow-up rate
- Average adherence
- AI utilization

---

# Client Widgets

## WDG-010 Client Summary

Displays:

- Personal profile
- Current goals
- Latest measurements
- Current nutrition plan

---

## WDG-011 Progress Overview

Displays:

- Weight trend
- Body fat
- Muscle mass
- Goal completion

---

## WDG-012 Nutrition Timeline

Displays:

- Assessments
- Meal plans
- Consultations
- AI recommendations

---

## WDG-013 Dietary Restrictions

Displays:

- Allergies
- Intolerances
- Food preferences
- Religious restrictions

---

# Assessment Widgets

## WDG-020 Anthropometric Summary

Displays:

- BMI
- Body composition
- Circumferences
- Weight evolution

---

## WDG-021 Clinical Indicators

Displays:

- Blood pressure
- Laboratory markers
- Clinical observations
- Risk factors

---

## WDG-022 Dietary Analysis

Displays:

- Daily intake
- Nutritional balance
- Eating habits
- Meal frequency

---

# Meal Planning Widgets

## WDG-030 Meal Planner

Displays:

- Daily meals
- Weekly schedule
- Macro distribution
- Calories

---

## WDG-031 Recipe Suggestions

Displays:

- AI recommendations
- Favorite recipes
- Seasonal recipes
- Organization templates

---

## WDG-032 Shopping Summary

Displays:

- Ingredients
- Quantities
- Categories
- Estimated cost

---

# Analytics Widgets

## WDG-040 Nutrition KPIs

Displays:

- Client adherence
- Weight evolution
- Goal achievement
- Consultation completion

---

## WDG-041 Trend Analysis

Displays:

- Historical charts
- Nutritional trends
- Behavioral evolution

---

## WDG-042 Population Analytics

Enterprise widget displaying:

- Population nutrition
- Organization trends
- Program effectiveness

---

# AI Widgets

## WDG-050 EVO Assistant

Displays:

- AI conversation
- Suggested actions
- Evidence
- References

---

## WDG-051 AI Recommendations

Displays:

- Meal suggestions
- Supplement proposals
- Nutritional risks
- Alternative options

---

## WDG-052 AI Confidence

Displays:

- Confidence score
- Explanation
- Evidence level
- Recommendation status

---

# Human Digital Twin Widgets

## WDG-060 Digital Twin Summary

Displays:

- Nutritional profile
- Current health indicators
- Lifestyle summary

---

## WDG-061 Prediction Panel

Displays:

- Weight projection
- Adherence prediction
- Nutritional risks
- Goal forecast

---

## WDG-062 Behavioral Insights

Displays:

- Eating habits
- Consistency
- Behavioral trends
- Motivation indicators

---

# Collaboration Widgets

## WDG-070 Shared Notes

Displays:

- Coach comments
- Physician observations
- Team discussions

---

## WDG-071 Activity Feed

Displays:

- Client updates
- Reports
- Assessments
- AI events

---

# Productivity Widgets

## WDG-080 Quick Actions

Provides:

- New Client
- New Assessment
- New Meal Plan
- Generate Report
- Launch EVO

---

## WDG-081 Recent Clients

Displays recently accessed clients.

---

## WDG-082 Favorites

Displays:

- Favorite reports
- Favorite recipes
- Saved searches
- Templates

---

# Widget States

Every widget supports:

- Default
- Loading
- Empty
- Error
- Refreshing
- Expanded
- Collapsed

---

# Personalization

Users may configure:

- Widget visibility
- Widget size
- Position
- Refresh interval
- Preferred metrics

Layouts are persisted per user.

---

# Responsive Behavior

Widgets adapt to:

- Desktop
- Tablet
- Mobile

Content density changes according to available space.

---

# Accessibility

Widgets comply with:

- WCAG 2.2 AA
- Keyboard navigation
- Screen readers
- High contrast
- Reduced motion

---

# AI Integration

AI-enabled widgets provide:

- Explainability
- Confidence indicators
- Professional approval
- Context awareness
- Feedback collection

---

# Widget Metadata

Every widget defines:

- Widget ID
- Name
- Owner
- Category
- Inputs
- Outputs
- Permissions
- Events
- APIs
- Version

---

# Repository Structure

```text
widgets/
├── dashboard/
├── clients/
├── assessments/
├── meal-planning/
├── analytics/
├── ai/
├── digital-twin/
├── collaboration/
├── productivity/
├── documentation/
└── metadata.yml
```

---

# Standard Visual Artifacts

## Widget Hierarchy

```text
Dashboard

↓

Workspace

↓

Widget

↓

Component
```

---

## Dashboard Composition

```text
Dashboard

├── KPIs
├── Tasks
├── Alerts
├── Calendar
└── AI
```

---

## AI Widget Flow

```text
Context

↓

AI Analysis

↓

Recommendation

↓

Professional Review
```

---

## Human Digital Twin Widgets

```text
Measurements

↓

Digital Twin

↓

Predictions

↓

Insights
```

---

# Visual Source Files

```text
artifacts/
└── widget-catalog/
    ├── dashboard.drawio
    ├── widgets.drawio
    ├── ai-widgets.drawio
    ├── digital-twin.drawio
    ├── analytics.drawio
    ├── workspace.drawio
    ├── mermaid/
    │   ├── widgets.mmd
    │   ├── dashboard.mmd
    │   ├── ai.mmd
    │   ├── analytics.mmd
    │   └── workspace.mmd
    └── exports/
        ├── widget-catalog.svg
        ├── widget-catalog.png
        └── widget-catalog.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Screen Catalog | Chapter 08 |
| Layout Architecture | Chapter 09 |
| Component Catalog | Chapter 12 |
| Design Tokens | Chapter 14 |
| Workflow Architecture | Chapter 16 |
| AI Services | Chapter 21 |

---

# Acceptance Criteria

This chapter is complete when:

- Widget taxonomy is documented.
- Dashboard widgets are defined.
- AI widgets are specified.
- Human Digital Twin widgets are documented.
- Personalization capabilities are established.
- Accessibility and responsiveness are verified.
- Widget metadata standards are defined.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- Widgets encapsulate complete business capabilities by combining reusable UI components into configurable workspace modules.
- The Nutritionist Portal uses specialized widgets for client management, assessments, meal planning, analytics, AI assistance and Human Digital Twin visualization.
- Personalizable, responsive and accessible widgets allow professionals to adapt their workspaces without compromising consistency across the EVOXA platform.
- A centralized widget catalog simplifies maintenance, promotes reuse and supports the future evolution of intelligent dashboards and configurable workspaces.

---

# Next Chapter

**Chapter 14 — Design Tokens**

This chapter defines the design tokens used throughout the Nutritionist Portal, including colors, typography, spacing, sizing, borders, shadows, motion and semantic tokens that ensure visual consistency across the EVOXA Design System.
