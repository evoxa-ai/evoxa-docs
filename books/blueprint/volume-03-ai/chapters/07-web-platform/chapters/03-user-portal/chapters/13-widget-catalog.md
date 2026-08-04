---
document_id: BP-0003-V3-C07-03-13
chapter_id: CH-03-USER-13
feature_pack: FP-DASH-0001
title: Widget Catalog
version: 1.0.0
status: Draft
owner: Widget Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-04
---

# Chapter 13 — Widget Catalog

> *Widgets are the primary business building blocks of the EVOXA User Portal. They encapsulate business capabilities, user interactions and AI-powered experiences in reusable, composable and independently deployable units.*

---

# Executive Summary

Widgets represent the functional layer between user interface components and business capabilities.

Unlike UI Components, which provide visual primitives, Widgets encapsulate complete user-facing functionality, integrating business rules, APIs, events, analytics and AI services.

Every widget is independently versioned, reusable and observable.

---

# Objectives

The Widget Catalog shall:

- Standardize business functionality.
- Enable modular dashboard composition.
- Support AI-driven personalization.
- Reduce implementation duplication.
- Allow independent evolution.
- Improve observability.
- Facilitate Feature Pack development.

---

# Widget Architecture

```text
Feature Pack

↓

Screen

↓

Widget

↓

Components

↓

Design Tokens
```

Widgets never contain business logic directly.

Business logic belongs to Application Services and Domain Services.

---

# Widget Philosophy

Every widget shall be:

- Business-oriented
- Self-contained
- Configurable
- Observable
- Responsive
- Accessible
- Versioned
- Independently testable

---

# Widget Categories

| Category | Prefix |
|-----------|---------|
| Dashboard | WGT-DASH |
| Nutrition | WGT-NUT |
| Training | WGT-TRN |
| Recovery | WGT-REC |
| Goals | WGT-GOAL |
| Progress | WGT-PROG |
| Analytics | WGT-ANL |
| AI | WGT-AI |
| Notifications | WGT-NOT |
| Profile | WGT-PROF |

---

# Dashboard Widgets

| Widget ID | Widget |
|------------|------------------------|
| WGT-DASH-001 | Daily Readiness |
| WGT-DASH-002 | AI Summary |
| WGT-DASH-003 | Daily Goals |
| WGT-DASH-004 | Quick Actions |
| WGT-DASH-005 | Health Snapshot |
| WGT-DASH-006 | Weekly Overview |

---

# Nutrition Widgets

| Widget ID | Widget |
|------------|--------------------|
| WGT-NUT-001 | Today's Meals |
| WGT-NUT-002 | Calories |
| WGT-NUT-003 | Macronutrients |
| WGT-NUT-004 | Water Intake |
| WGT-NUT-005 | Meal Suggestions |

---

# Training Widgets

| Widget ID | Widget |
|------------|---------------------|
| WGT-TRN-001 | Today's Workout |
| WGT-TRN-002 | Exercise Progress |
| WGT-TRN-003 | Training Calendar |
| WGT-TRN-004 | Personal Records |

---

# Recovery Widgets

| Widget ID | Widget |
|------------|--------------------|
| WGT-REC-001 | Sleep Score |
| WGT-REC-002 | Recovery Score |
| WGT-REC-003 | Stress Level |
| WGT-REC-004 | Readiness |

---

# Goal Widgets

| Widget ID | Widget |
|------------|--------------------|
| WGT-GOAL-001 | Active Goals |
| WGT-GOAL-002 | Goal Progress |
| WGT-GOAL-003 | Milestones |

---

# Progress Widgets

| Widget ID | Widget |
|------------|---------------------|
| WGT-PROG-001 | Weight Trend |
| WGT-PROG-002 | Body Composition |
| WGT-PROG-003 | Habit Tracker |
| WGT-PROG-004 | Performance Trend |

---

# AI Widgets

| Widget ID | Widget |
|------------|---------------------|
| WGT-AI-001 | AI Coach |
| WGT-AI-002 | Recommendations |
| WGT-AI-003 | Daily Insights |
| WGT-AI-004 | Explainability Card |
| WGT-AI-005 | AI Conversation |

---

# Notification Widgets

| Widget ID | Widget |
|------------|-----------------------|
| WGT-NOT-001 | Notification Center |
| WGT-NOT-002 | Reminder Timeline |
| WGT-NOT-003 | Achievement Feed |

---

# Widget Blueprint Object (WBO)

Every widget defines:

- Widget ID
- Name
- Category
- Business Purpose
- Owner
- Feature Pack
- Experience Slice
- Screen Mapping
- Components
- APIs
- Events
- Business Rules
- AI Capabilities
- Permissions
- Configuration
- Telemetry
- Accessibility
- Version

---

# Example

## WGT-DASH-001

### Name

Daily Readiness

### Business Purpose

Present the user's readiness for today's activities.

### Feature Pack

FP-DASH-0001

### Experience Slice

XS-DAILY-EXPERIENCE

### Components

- Card
- Progress Ring
- Status Badge
- Tooltip

### APIs

- API-DASH-001

### Events

- EVT-READINESS-VIEWED
- EVT-READINESS-CLICKED

### AI

- AIC-READINESS

### Permissions

- MEMBER

---

# Widget Composition

```text
Widget

↓

Components

↓

Business Data

↓

AI

↓

Actions
```

---

# Widget Lifecycle

```text
Designed

↓

Implemented

↓

Released

↓

Observed

↓

Optimized

↓

Deprecated
```

---

# Widget Personalization

Widgets may adapt:

- Visibility
- Order
- Size
- Priority
- Content
- Suggested actions

Core functionality remains unchanged.

---

# Widget Communication

Widgets communicate using:

- Events
- Shared State
- APIs
- Message Bus

Widgets shall never directly invoke each other.

---

# Widget Configuration

Each widget supports:

- Visibility
- Refresh Interval
- Theme
- Layout Preferences
- Notification Rules
- AI Behavior

Configuration is stored per user.

---

# Widget Performance

| Metric | Target |
|----------|---------|
| Initial Render | <300 ms |
| Refresh | <200 ms |
| Lazy Load | Yes |
| Offline Cache | Supported |

---

# Widget Analytics

Widgets publish:

- Viewed
- Loaded
- Refreshed
- Clicked
- Expanded
- Collapsed
- Completed
- Error

---

# Widget Accessibility

Every widget shall:

- Meet WCAG 2.2 AA
- Support keyboard navigation
- Provide semantic labels
- Announce dynamic updates
- Respect reduced motion settings

---

# Widget Governance

Every widget requires:

- Product approval
- UX validation
- Accessibility review
- Architecture review
- Security review
- Automated testing
- Performance validation

---

# Widget Dependency Model

```text
Widget

↓

Components

↓

Services

↓

APIs

↓

Events

↓

Analytics
```

---

# Standard Visual Artifacts

## Widget Hierarchy

```text
Screen

↓

Widgets

↓

Components

↓

Tokens
```

---

## Widget Lifecycle

```text
Design

↓

Build

↓

Deploy

↓

Observe

↓

Improve
```

---

## Widget Relationships

```text
Widget

↓

Business Rules

↓

API

↓

Events

↓

AI
```

---

## Widget Composition

```text
Widget

↓

Card

↓

Chart

↓

Actions

↓

Telemetry
```

---

# Visual Source Files

```text
artifacts/
└── widget-catalog/
    ├── widget-hierarchy.drawio
    ├── widget-composition.drawio
    ├── widget-lifecycle.drawio
    ├── widget-dependencies.drawio
    ├── dashboard-layout.drawio
    ├── mermaid/
    │   ├── hierarchy.mmd
    │   ├── composition.mmd
    │   ├── lifecycle.mmd
    │   └── dependencies.mmd
    └── exports/
        ├── *.svg
        ├── *.png
        └── *.pdf
```

---

# Widget Repository Structure

```text
catalog/
└── widgets/
    ├── WGT-DASH-001-daily-readiness/
    │   ├── README.md
    │   ├── metadata.yml
    │   ├── configuration.yml
    │   ├── layout.drawio
    │   ├── figma.fig
    │   ├── apis.yml
    │   ├── events.yml
    │   ├── ai.yml
    │   ├── permissions.yml
    │   ├── analytics.yml
    │   ├── tests.md
    │   └── changelog.md
    │
    ├── WGT-AI-001-ai-coach/
    ├── WGT-TRN-001-todays-workout/
    └── ...
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Component Catalog | Chapter 12 |
| Screen Catalog | Chapter 08 |
| Layout Architecture | Chapter 09 |
| Feature Packs | FP Series |
| Experience Slices | XS Series |
| APIs | API Series |
| Events | EVT Series |
| Business Rules | BR Series |
| AI Capabilities | AIC Series |

---

# Acceptance Criteria

This chapter is complete when:

- Every widget has a unique identifier.
- Widget categories are documented.
- Business purpose is defined.
- Screen mappings are established.
- APIs, Events and AI integrations are identified.
- Accessibility and performance requirements are documented.
- Governance and lifecycle are established.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- Widgets are the primary business units of the EVOXA user experience.
- They encapsulate functionality while remaining independent of presentation and domain logic.
- Every widget is reusable, observable and versioned.
- The Widget Catalog provides a scalable foundation for dashboards, Feature Packs and AI-powered personalization.

---

# Next Chapter

**Chapter 14 — Design Tokens**

This chapter defines the visual foundation of the EVOXA Design System, including color palettes, typography, spacing, sizing, elevation, motion, icons and semantic design tokens shared across every portal.
