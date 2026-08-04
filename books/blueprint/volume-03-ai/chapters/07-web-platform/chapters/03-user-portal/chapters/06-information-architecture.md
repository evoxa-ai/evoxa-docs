---
document_id: BP-0003-V3-C07-03-06
chapter_id: CH-03-USER-06
feature_pack: FP-DASH-0001
title: Information Architecture
version: 1.0.0
status: Draft
owner: UX Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-04
---

# Chapter 06 — Information Architecture

> *Information Architecture defines how information is organized, connected and presented so users can efficiently achieve their health goals.*

---

# Executive Summary

The Information Architecture (IA) of the EVOXA User Portal organizes content around user goals instead of technical modules.

Information is structured into logical domains that enable intuitive navigation, contextual discovery and AI-driven personalization.

The architecture ensures that users always have access to the right information at the right moment.

---

# Objectives

The Information Architecture shall:

- Reduce cognitive load.
- Improve discoverability.
- Support AI personalization.
- Enable contextual navigation.
- Scale as new capabilities are introduced.
- Maintain consistency across the platform.
- Separate information from presentation.

---

# Information Architecture Principles

The User Portal follows these principles:

- Goal-oriented navigation.
- Progressive disclosure.
- Consistent taxonomy.
- Minimal navigation depth.
- Contextual relevance.
- AI-assisted discovery.
- Accessibility-first organization.

---

# Information Domains

The User Portal is organized into the following domains.

| Domain | Purpose |
|---------|---------|
| Dashboard | Daily overview |
| Nutrition | Meals and nutrition |
| Training | Exercise and workouts |
| Recovery | Sleep and recovery |
| Goals | Objectives and milestones |
| Progress | Historical insights |
| AI Coach | Personalized guidance |
| Notifications | Alerts and reminders |
| Profile | Personal configuration |

---

# Information Hierarchy

```text
User Portal

├── Dashboard
│     ├── Daily Summary
│     ├── Readiness
│     ├── AI Summary
│     └── Quick Actions
│
├── Nutrition
│     ├── Meal Plan
│     ├── Food Log
│     ├── Hydration
│     └── Recommendations
│
├── Training
│     ├── Today's Workout
│     ├── Exercise Library
│     ├── History
│     └── Performance
│
├── Recovery
│     ├── Sleep
│     ├── Recovery Score
│     ├── Stress
│     └── Readiness
│
├── Goals
│
├── Progress
│
├── AI Coach
│
├── Notifications
│
└── Profile
```

---

# Navigation Depth

The platform shall maintain:

| Level | Description |
|---------|-------------|
| Level 1 | Main Navigation |
| Level 2 | Functional Area |
| Level 3 | Detail View |
| Level 4 | Contextual Information |

No critical user flow should require more than four navigation levels.

---

# Content Taxonomy

All information belongs to one of the following categories.

- Health
- Nutrition
- Training
- Recovery
- Goals
- Progress
- AI
- Notifications
- Account

Each category is uniquely identified and versioned.

---

# Information Objects

The User Portal manages several reusable information objects.

| Object | Identifier |
|----------|------------|
| Screen | SCR-* |
| Component | CMP-* |
| Widget | WGT-* |
| Workflow | FLOW-* |
| Business Rule | BR-* |
| API | API-* |
| Event | EVT-* |
| AI Capability | AIC-* |

---

# Information Relationships

```text
Information Domain

↓

Screen

↓

Widget

↓

Component

↓

Data Object

↓

Business Rule

↓

Event
```

---

# Search & Discoverability

Users can access information through:

- Navigation menus.
- Dashboard widgets.
- Global search.
- AI conversations.
- Notifications.
- Recommendations.
- Contextual links.

Search results are ranked using relevance, recency and user context.

---

# Contextual Navigation

The platform adapts navigation according to:

- User goals.
- Current journey.
- Human Digital Twin.
- Time of day.
- Recovery status.
- Active workout.
- Nutrition schedule.

---

# Personalization

Information order is dynamically adjusted based on:

- Frequently accessed content.
- Current objectives.
- Behavioral patterns.
- AI recommendations.
- User preferences.

The underlying taxonomy remains consistent.

---

# Content Ownership

| Domain | Owner |
|---------|--------|
| Dashboard | Product Team |
| Nutrition | Nutrition Domain |
| Training | Training Domain |
| Recovery | Recovery Domain |
| Goals | Goal Domain |
| AI Coach | AI Platform |
| Notifications | Notification Platform |
| Profile | Identity Platform |

---

# Metadata Strategy

Every information object defines:

- Identifier.
- Title.
- Description.
- Owner.
- Category.
- Version.
- Status.
- Tags.
- Dependencies.

---

# Scalability

The architecture supports:

- New domains.
- New widgets.
- New Feature Packs.
- Additional portals.
- Future AI capabilities.

No redesign should be required when adding new modules.

---

# Accessibility

Information Architecture supports:

- Screen readers.
- Keyboard navigation.
- Clear headings.
- Landmark regions.
- Predictable navigation.
- Consistent terminology.

---

# Analytics

The platform measures:

- Navigation paths.
- Search usage.
- Most visited areas.
- Abandonment points.
- Content discoverability.
- AI-assisted navigation.

---

# Standard Visual Artifacts

## Domain Map

```text
Dashboard

↓

Nutrition

↓

Training

↓

Recovery

↓

Goals

↓

Progress

↓

AI Coach
```

---

## Information Hierarchy

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

## Navigation Tree

```text
Home

├── Dashboard
├── Nutrition
├── Training
├── Recovery
├── Goals
├── Progress
├── AI Coach
└── Profile
```

---

## Information Relationships

```text
Domain

↓

Screen

↓

Widget

↓

Data

↓

AI
```

---

# Visual Source Files

```text
artifacts/
└── information-architecture/
    ├── domain-map.drawio
    ├── navigation-tree.drawio
    ├── hierarchy.drawio
    ├── relationships.drawio
    ├── taxonomy.drawio
    ├── mermaid/
    │   ├── domains.mmd
    │   ├── hierarchy.mmd
    │   ├── navigation.mmd
    │   └── taxonomy.mmd
    └── exports/
        ├── *.svg
        ├── *.png
        └── *.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Product Scope | Chapter 03 |
| Personas | Chapter 04 |
| User Journeys | Chapter 05 |
| UX Goals | Chapter 07 |
| Screen Catalog | Chapter 08 |
| Navigation Architecture | Chapter 10 |
| Widgets | WGT Series |
| APIs | API Series |
| Events | EVT Series |

---

# Acceptance Criteria

This chapter is complete when:

- Information domains are defined.
- Navigation hierarchy is documented.
- Taxonomy is established.
- Content ownership is assigned.
- Personalization rules are described.
- Accessibility considerations are included.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- Information is organized around user goals rather than technical implementation.
- The Information Architecture provides a scalable foundation for future Feature Packs.
- AI enhances discoverability without changing the underlying taxonomy.
- A consistent hierarchy and metadata strategy ensure maintainability, usability and long-term evolution.

---

# Next Chapter

**Chapter 07 — UX Goals**

This chapter defines the user experience objectives, usability principles, interaction quality standards, emotional design goals and measurable UX success criteria for the EVOXA User Portal.
