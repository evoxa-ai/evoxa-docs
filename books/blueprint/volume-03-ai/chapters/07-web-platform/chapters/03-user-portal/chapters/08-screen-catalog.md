---
document_id: BP-0003-V3-C07-03-08
chapter_id: CH-03-USER-08
feature_pack: FP-DASH-0001
title: Screen Catalog
version: 1.0.0
status: Draft
owner: UX Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-04
---

# Chapter 08 — Screen Catalog

> *The Screen Catalog defines every screen within the EVOXA User Portal, establishing its purpose, ownership, navigation, dependencies and relationship with the platform architecture.*

---

# Executive Summary

The Screen Catalog is the authoritative inventory of all screens belonging to the User Portal.

Each screen is uniquely identified, versioned and linked to its corresponding Feature Packs, Experience Slices, Business Rules, APIs, Events, Widgets and AI Capabilities.

Screens are considered architectural assets rather than isolated UI pages.

---

# Objectives

The Screen Catalog shall:

- Define every screen.
- Eliminate duplicate functionality.
- Standardize navigation.
- Support traceability.
- Enable automated documentation.
- Improve maintainability.
- Facilitate future expansion.

---

# Screen Architecture

```text
Portal

↓

Domain

↓

Screen

↓

Sections

↓

Widgets

↓

Components
```

---

# Screen Categories

| Category | Prefix |
|-----------|----------|
| Authentication | AUTH |
| Dashboard | DASH |
| Nutrition | NUT |
| Training | TRN |
| Recovery | REC |
| Goals | GOAL |
| Progress | PROG |
| AI Coach | AI |
| Notifications | NOT |
| Profile | PROF |
| Settings | SET |

---

# Screen Inventory

| Screen ID | Name | Domain |
|------------|---------------------|--------------|
| SCR-DASH-001 | Dashboard Home | Dashboard |
| SCR-NUT-001 | Nutrition Home | Nutrition |
| SCR-NUT-002 | Meal Detail | Nutrition |
| SCR-TRN-001 | Workout Home | Training |
| SCR-TRN-002 | Workout Execution | Training |
| SCR-REC-001 | Recovery Home | Recovery |
| SCR-GOAL-001 | Goals | Goals |
| SCR-PROG-001 | Progress | Progress |
| SCR-AI-001 | AI Coach | AI |
| SCR-NOT-001 | Notifications | Notifications |
| SCR-PROF-001 | User Profile | Profile |
| SCR-SET-001 | Settings | Settings |

---

# Screen Blueprint Object (SBO)

Every screen follows the same specification.

---

## SCR-DASH-001

### Name

Dashboard Home

---

### Purpose

Provide a personalized overview of the user's daily health status.

---

### Primary Users

- Members

---

### Entry Points

- Login
- Notifications
- Deep Links
- Daily Reminder

---

### Exit Points

- Nutrition
- Workout
- Recovery
- Progress
- AI Coach

---

### Feature Pack

FP-DASH-0001

---

### Experience Slice

XS-DAILY-EXPERIENCE

---

### Widgets

- WGT-READINESS
- WGT-AI-SUMMARY
- WGT-DAILY-GOALS
- WGT-NUTRITION
- WGT-WORKOUT
- WGT-RECOVERY

---

### APIs

- API-DASH-001
- API-AI-001
- API-NOT-001

---

### Events

- EVT-DASH-VIEWED
- EVT-DASH-REFRESHED

---

### AI Capabilities

- AIC-DAILY-SUMMARY
- AIC-RECOMMENDATIONS

---

### Business Rules

- BR-101
- BR-203
- BR-205

---

### Accessibility

WCAG 2.2 AA

---

### Performance Target

Dashboard ready in less than 2 seconds.

---

# Screen Lifecycle

```text
Designed

↓

Approved

↓

Implemented

↓

Released

↓

Observed

↓

Improved

↓

Deprecated
```

---

# Navigation Hierarchy

```text
Dashboard

├── Nutrition

├── Training

├── Recovery

├── Goals

├── Progress

├── AI Coach

├── Notifications

└── Profile
```

---

# Responsive Matrix

| Screen | Desktop | Tablet | Mobile |
|----------|---------|---------|---------|
| Dashboard | ✓ | ✓ | ✓ |
| Nutrition | ✓ | ✓ | ✓ |
| Training | ✓ | ✓ | ✓ |
| Recovery | ✓ | ✓ | ✓ |
| Goals | ✓ | ✓ | ✓ |
| Progress | ✓ | ✓ | ✓ |
| AI Coach | ✓ | ✓ | ✓ |

---

# Screen Dependencies

Every screen may depend on:

- Widgets
- Components
- APIs
- Business Rules
- Events
- AI Capabilities
- Permissions

---

# Screen Metadata

Every screen defines:

- Screen ID
- Version
- Owner
- Domain
- Status
- Category
- Route
- Feature Pack
- Experience Slice
- Tags

---

# Screen Performance

| Metric | Target |
|----------|---------|
| Initial Render | <1.5 s |
| Navigation | <300 ms |
| Widget Composition | <500 ms |
| Skeleton Display | <100 ms |

---

# Screen Analytics

Every screen publishes:

- Viewed
- Loaded
- Refreshed
- Left
- Error
- Interaction Count
- Time on Screen

---

# Accessibility

All screens shall:

- Support keyboard navigation.
- Provide semantic landmarks.
- Meet WCAG 2.2 AA.
- Support screen readers.
- Maintain focus order.
- Respect reduced motion preferences.

---

# Screen Governance

Every new screen requires:

- Product approval.
- UX review.
- Accessibility validation.
- Architecture review.
- Performance validation.
- AI review (when applicable).

---

# Standard Visual Artifacts

## Screen Hierarchy

```text
Portal

↓

Domains

↓

Screens

↓

Widgets

↓

Components
```

---

## Navigation Map

```text
Dashboard

↓

Nutrition

↓

Training

↓

Recovery

↓

Progress

↓

AI Coach
```

---

## Screen Lifecycle

```text
Design

↓

Development

↓

Release

↓

Observation

↓

Evolution
```

---

## Screen Relationships

```text
Screen

↓

Widgets

↓

Components

↓

APIs

↓

Events
```

---

# Visual Source Files

```text
artifacts/
└── screen-catalog/
    ├── screen-hierarchy.drawio
    ├── navigation-map.drawio
    ├── screen-lifecycle.drawio
    ├── screen-dependencies.drawio
    ├── responsive-matrix.drawio
    ├── mermaid/
    │   ├── hierarchy.mmd
    │   ├── navigation.mmd
    │   ├── lifecycle.mmd
    │   └── dependencies.mmd
    ├── catalog/
    │   ├── screen-catalog.yml
    │   ├── routes.yml
    │   └── screen-metadata.yml
    └── exports/
        ├── *.svg
        ├── *.png
        └── *.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Information Architecture | Chapter 06 |
| UX Goals | Chapter 07 |
| Layout Architecture | Chapter 09 |
| Navigation Architecture | Chapter 10 |
| Components | Chapter 12 |
| Widgets | Chapter 13 |
| APIs | API Series |
| Events | EVT Series |
| AI Capabilities | AIC Series |

---

# Acceptance Criteria

This chapter is complete when:

- Every screen has a unique identifier.
- Screen purposes are documented.
- Navigation relationships are defined.
- Dependencies are identified.
- Responsive behavior is documented.
- Analytics events are specified.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- Screens are architectural assets with standardized metadata and lifecycle management.
- Every screen is connected to Feature Packs, Experience Slices, APIs, Events and AI Capabilities.
- Consistent governance ensures maintainability and scalability across the User Portal.
- The Screen Catalog provides a single source of truth for UX, Engineering and Product teams.

---

# Next Chapter

**Chapter 09 — Layout Architecture**

This chapter defines the structural layout system of the User Portal, including page templates, responsive grids, regions, adaptive layouts, composition rules and integration with the Design System.
