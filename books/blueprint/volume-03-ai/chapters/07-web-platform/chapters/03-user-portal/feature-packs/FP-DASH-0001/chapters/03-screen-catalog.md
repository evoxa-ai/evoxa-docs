---
document_id: BP-0003-V3-C07-03-FP-DASH-03
feature_pack_id: FP-DASH-0001
domain_feature_pack: DFP-DASH
experience_slice: XS-DAILY-EXPERIENCE
chapter: 03
title: Screen Catalog
version: 1.0.0
status: Draft
owner: Product Architecture Board
classification: Internal
---

# Chapter 03 — Screen Catalog

> *Every screen represents a business capability and shall be uniquely identifiable, traceable and reusable.*

---

# Executive Summary

The Dashboard Feature Pack is composed of multiple specialized screens.

Although users perceive the Dashboard as a single experience, internally it is implemented as a collection of reusable screens that collaborate to present personalized information.

Every screen is identified by a unique Screen ID (SCR) and is registered in the EVOXA Catalog.

---

# Screen Architecture

```text
Dashboard Experience

│

├── SCR-1000 Dashboard Home

├── SCR-1001 Daily Plan

├── SCR-1002 Health Summary

├── SCR-1003 AI Recommendations

├── SCR-1004 Quick Actions

├── SCR-1005 Daily Progress

├── SCR-1006 Notifications

└── SCR-1007 Dashboard Settings
```

---

# Screen Catalog

| Screen ID | Name | Purpose | Layout |
|------------|------|---------|--------|
| SCR-1000 | Dashboard Home | Main dashboard | LAY-0002 |
| SCR-1001 | Daily Plan | Today's activities | LAY-0002 |
| SCR-1002 | Health Summary | Current health indicators | LAY-0002 |
| SCR-1003 | AI Recommendations | Personalized recommendations | LAY-0002 |
| SCR-1004 | Quick Actions | Frequently used actions | LAY-0002 |
| SCR-1005 | Daily Progress | Daily completion status | LAY-0002 |
| SCR-1006 | Notification Center | Alerts and reminders | LAY-0002 |
| SCR-1007 | Dashboard Preferences | User customization | LAY-0002 |

---

# Screen Responsibilities

## SCR-1000 — Dashboard Home

Purpose

Provide a complete overview of the user's current situation.

Responsibilities

- Present today's priorities.
- Display Daily Readiness Score.
- Surface AI summary.
- Provide navigation.
- Aggregate widgets.

---

## SCR-1001 — Daily Plan

Responsibilities

- Display scheduled meals.
- Display scheduled workouts.
- Display recovery tasks.
- Display reminders.
- Present daily timeline.

---

## SCR-1002 — Health Summary

Responsibilities

- Weight trend.
- Recovery score.
- Nutrition adherence.
- Hydration.
- Sleep quality.
- Readiness Score.

---

## SCR-1003 — AI Recommendations

Responsibilities

- Explain recommendations.
- Prioritize actions.
- Present AI confidence.
- Display expected outcomes.
- Provide contextual guidance.

---

## SCR-1004 — Quick Actions

Responsibilities

Provide immediate access to:

- Log Meal
- Start Workout
- Log Water
- Record Weight
- Ask EVO
- View Calendar

---

## SCR-1005 — Daily Progress

Responsibilities

Display:

- Completed meals
- Completed workouts
- Daily calories
- Protein target
- Hydration
- Steps
- Recovery

---

## SCR-1006 — Notification Center

Responsibilities

- Critical alerts
- Scheduled reminders
- AI notifications
- Coach messages
- System announcements

---

## SCR-1007 — Dashboard Preferences

Responsibilities

Allow users to:

- Reorder cards
- Pin widgets
- Configure notifications
- Configure dashboard density
- Configure accessibility

---

# Screen Relationships

```text
SCR-1000 Dashboard

│

├── opens SCR-1001

├── opens SCR-1002

├── opens SCR-1003

├── opens SCR-1004

├── opens SCR-1005

├── opens SCR-1006

└── opens SCR-1007
```

---

# Navigation Model

```text
Dashboard Home

↓

Today's Plan

↓

Workout

↓

Nutrition

↓

Recovery

↓

Progress
```

Users can always return to Dashboard Home.

---

# Layout Assignment

All Dashboard screens inherit:

```text
LAY-0002

Authenticated Layout
```

Shared regions

```text
Global Header

↓

Navigation Sidebar

↓

Main Workspace

↓

Context Panel

↓

Footer
```

---

# Screen Lifecycle

```text
Draft

↓

Design

↓

Prototype

↓

Approved

↓

Implemented

↓

Validated

↓

Released

↓

Deprecated
```

---

# Screen Metadata

Every screen shall contain:

- Screen ID
- Name
- Description
- Owner
- Version
- Status
- Layout
- Components
- Widgets
- APIs
- Events
- Permissions
- Analytics
- Accessibility Profile

---

# Screen Dependencies

Every screen depends on:

- Identity Platform
- Adaptive Experience Engine
- Recommendation Engine
- Human Digital Twin
- Analytics Platform

Some screens additionally depend on:

- Nutrition Domain
- Training Domain
- Recovery Domain

---

# Screen Inventory Matrix

| Screen | Components | Widgets | APIs | Events |
|----------|-----------:|--------:|------:|--------:|
| SCR-1000 | 12 | 8 | 6 | 7 |
| SCR-1001 | 8 | 4 | 5 | 4 |
| SCR-1002 | 7 | 5 | 4 | 3 |
| SCR-1003 | 5 | 3 | 4 | 4 |
| SCR-1004 | 4 | 2 | 2 | 2 |
| SCR-1005 | 6 | 4 | 4 | 3 |
| SCR-1006 | 5 | 2 | 3 | 3 |
| SCR-1007 | 5 | 1 | 2 | 2 |

---

# Screen Traceability

Every screen shall reference:

Portal

PRT-0002

Experience Slice

XS-DAILY-EXPERIENCE

Domain Feature Pack

DFP-DASH

Feature Pack

FP-DASH-0001

Engineering Specification

ESP-WEB-DASH

Implementation Story

IS-WEB-DASH

Catalog Entry

evoxa-catalog/catalog/screens/

---

# Standard Visual Artifacts

## Screen Hierarchy

```text
Dashboard

├── Home

├── Daily Plan

├── Health Summary

├── AI

├── Quick Actions

├── Progress

├── Notifications

└── Preferences
```

---

## Navigation Graph

```text
Dashboard Home

├── Daily Plan

├── AI

├── Progress

├── Notifications

└── Settings
```

---

## Screen Dependency Diagram

```text
Dashboard Home

↓

Widgets

↓

Components

↓

APIs

↓

Events

↓

Analytics
```

---

# Visual Source Files

```text
artifacts/
└── screen-catalog/
    ├── screen-map.drawio
    ├── navigation.drawio
    ├── dependency-map.drawio
    ├── lifecycle.drawio
    ├── inventory.drawio
    ├── mermaid/
    │   ├── screen-map.mmd
    │   ├── navigation.mmd
    │   ├── dependencies.mmd
    │   └── lifecycle.mmd
    ├── figma/
    │   ├── dashboard-home.fig
    │   ├── health-summary.fig
    │   ├── ai-recommendations.fig
    │   └── quick-actions.fig
    └── exports/
        ├── *.svg
        ├── *.png
        └── *.pdf
```

---

# Screen Acceptance Criteria

Every Dashboard screen shall:

- Have a unique identifier.
- Be linked to one Feature Pack.
- Use one approved layout.
- Publish analytics events.
- Declare consumed APIs.
- Declare produced events.
- Meet accessibility standards.
- Be registered in the EVOXA Catalog.
- Be covered by automated UI tests.
- Be linked to an Engineering Specification and an Implementation Story.

---

# Key Takeaways

- Screens are architectural objects, not merely pages.
- Each screen has clear responsibilities and explicit dependencies.
- All Dashboard screens inherit a shared layout while exposing specialized capabilities.
- Every screen is fully traceable across the Blueprint, the EVOXA Catalog and engineering artifacts.

---

# Next Chapter

## Chapter 04 — Layout Architecture

The next chapter defines the internal layout structure of the Dashboard, including responsive regions, adaptive panels, widget containers, visual hierarchy and interaction zones.
