---
document_id: BP-0003-V3-C07-04-13
chapter_id: CH-04-COACH-13
feature_pack: FP-COACH-0001
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

> *The Widget Catalog defines every reusable dashboard widget available within the Coach Portal, providing configurable, AI-assisted and data-driven building blocks that enable professionals to monitor operations, prioritize work and make informed decisions.*

---

# Executive Summary

Widgets are the operational intelligence units of the Coach Portal.

Unlike standard UI components, widgets combine visualization, business logic, user interactions and AI-powered insights into configurable modules that can be arranged to create personalized professional workspaces.

Widgets transform dashboards into dynamic operational environments.

---

# Objectives

The Widget Catalog shall:

- Standardize dashboard widgets.
- Enable workspace personalization.
- Surface operational insights.
- Support AI recommendations.
- Provide reusable analytical modules.
- Improve productivity.
- Scale across future workspaces.

---

# Widget Philosophy

Widgets are autonomous operational modules.

Each widget:

- Displays meaningful information.
- Supports user interaction.
- Can be configured.
- Updates in real time.
- Integrates AI where applicable.
- Can be reused across workspaces.

---

# Widget Architecture

```text
Dashboard

↓

Widget

↓

Data Source

↓

Business Logic

↓

Visualization

↓

Actions
```

---

# Widget Categories

| Category | Purpose |
|----------|----------|
| Operational | Daily work |
| Client | Client management |
| AI | AI insights |
| Analytics | KPIs |
| Communication | Messages |
| Calendar | Scheduling |
| Business | Revenue |
| Monitoring | Platform status |

---

# Operational Widgets

| Widget ID | Widget |
|------------|---------|
| WDG-001 | Today's Agenda |
| WDG-002 | Pending Tasks |
| WDG-003 | Priority Clients |
| WDG-004 | Recent Activity |
| WDG-005 | Quick Actions |

---

# Client Widgets

| Widget ID | Widget |
|------------|---------|
| WDG-100 | Client Summary |
| WDG-101 | Active Programs |
| WDG-102 | Progress Overview |
| WDG-103 | Goal Tracking |
| WDG-104 | Assessment Status |
| WDG-105 | Human Digital Twin Snapshot |

---

# Training Widgets

Examples:

- Weekly Sessions
- Workout Compliance
- Exercise Progress
- Program Status
- Recovery Indicators

---

# Nutrition Widgets

Examples:

- Nutrition Adherence
- Meal Compliance
- Macro Distribution
- Water Intake
- Supplement Tracking

---

# AI Widgets

Examples:

- EVO Recommendations
- Risk Alerts
- Behavioral Insights
- Smart Priorities
- AI Daily Briefing
- Opportunity Detection
- Explainability Panel

---

# Analytics Widgets

Examples:

- Active Clients
- Monthly Revenue
- Coach Productivity
- Client Retention
- Goal Completion
- AI Adoption
- Engagement Trends

---

# Calendar Widgets

Examples:

- Today's Schedule
- Upcoming Sessions
- Availability
- Appointment Requests

---

# Communication Widgets

Examples:

- Inbox Summary
- Unread Messages
- Client Notifications
- Broadcast Status

---

# Business Widgets

Examples:

- Revenue Summary
- Subscription Status
- Outstanding Payments
- Monthly Growth

---

# Widget Metadata

Each widget includes:

- Widget ID
- Name
- Category
- Owner
- Data Source
- Refresh Strategy
- Permissions
- AI Integration
- Responsive Behavior
- Configuration Options

---

# Widget States

Every widget supports:

- Loading
- Ready
- Empty
- Error
- Offline
- Refreshing

---

# Widget Configuration

Professionals may configure:

- Position
- Size
- Refresh interval
- Filters
- Time period
- Display mode
- Theme

Configuration is synchronized across devices.

---

# Widget Personalization

Widgets can be:

- Added
- Removed
- Resized
- Pinned
- Collapsed
- Grouped

Personalization does not affect other users.

---

# AI Integration

AI-enabled widgets provide:

- Recommendations
- Priorities
- Predictions
- Explanations
- Suggested actions
- Confidence indicators

Professionals may provide feedback to improve recommendations.

---

# Data Sources

Widgets consume data from:

- Client Domain
- Training Domain
- Nutrition Domain
- Calendar
- Messaging
- Analytics Platform
- Human Digital Twin
- AI Platform

---

# Refresh Strategy

| Widget Type | Refresh |
|-------------|----------|
| Dashboard KPIs | 5 min |
| AI Alerts | Real-time |
| Messages | Real-time |
| Calendar | 1 min |
| Analytics | On demand |
| Reports | Manual |

---

# Widget Lifecycle

```text
Created

↓

Configured

↓

Active

↓

Updated

↓

Archived
```

---

# Accessibility

Widgets support:

- Keyboard navigation.
- Screen readers.
- High contrast.
- Responsive layouts.
- ARIA landmarks.

---

# Responsive Behavior

## Desktop

Multi-column grid.

---

## Tablet

Adaptive grid.

---

## Mobile

Single-column stack with collapsible cards.

---

# Widget Governance

Every widget requires:

- UX approval.
- Design review.
- Accessibility validation.
- Performance validation.
- AI review (if applicable).

Widgets are versioned independently.

---

# Standard Visual Artifacts

## Widget Architecture

```text
Dashboard

↓

Widget

↓

Logic

↓

Visualization
```

---

## Widget Lifecycle

```text
Create

↓

Configure

↓

Use

↓

Update
```

---

## Dashboard Composition

```text
Workspace

↓

Widgets

↓

Insights

↓

Actions
```

---

## AI Widget Flow

```text
Data

↓

AI

↓

Recommendation

↓

Coach
```

---

# Visual Source Files

```text
artifacts/
└── widget-catalog/
    ├── widget-architecture.drawio
    ├── dashboard-grid.drawio
    ├── ai-widget-flow.drawio
    ├── widget-lifecycle.drawio
    ├── responsive-widgets.drawio
    ├── widget-composition.drawio
    ├── mermaid/
    │   ├── widgets.mmd
    │   ├── dashboard.mmd
    │   ├── ai.mmd
    │   ├── lifecycle.mmd
    │   └── composition.mmd
    └── exports/
        ├── *.svg
        ├── *.png
        └── *.pdf
```

---

# Repository Structure

```text
widgets/
├── operational/
├── clients/
├── training/
├── nutrition/
├── analytics/
├── ai/
├── calendar/
├── communication/
├── business/
├── templates/
├── layouts/
└── metadata.yml
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

- Widget taxonomy is defined.
- Dashboard widgets are cataloged.
- Configuration model is documented.
- AI-enabled widgets are identified.
- Personalization rules are established.
- Responsive behavior is specified.
- Governance model is documented.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- Widgets are autonomous operational modules that combine data, business logic, visualization and AI-powered insights.
- Personalized dashboards allow professionals to organize information according to their workflows and priorities.
- AI-enhanced widgets transform operational data into actionable recommendations while preserving professional control.
- Independent governance, versioning and configuration ensure that widgets remain reusable, scalable and adaptable across the EVOXA platform.

---

# Next Chapter

**Chapter 14 — Design Tokens**

This chapter defines the foundational design tokens of the Coach Portal, including color palettes, typography, spacing, sizing, elevation, motion, semantic tokens and theming rules that ensure visual consistency across the entire Design System.
