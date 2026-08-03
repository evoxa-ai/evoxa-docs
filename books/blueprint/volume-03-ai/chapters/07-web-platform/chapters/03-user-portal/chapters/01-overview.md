---
document_id: BP-0003-V3-C07-03-01
product_id: PRT-0002
portal_id: PRT-0002
chapter_id: PRT-0002-C01
book: EVOXA Blueprint
volume: Volume III
chapter: 7
section: 3
subsection: 1
title: User Portal Overview
version: 1.0.0
status: Draft
classification: Internal
owner: Product Architecture Board
author: EVOXA Architecture Office
reviewers:
  - Product Architecture Board
  - UX Advisory Board
  - Platform Engineering Board
language: en
created: 2026-08-03
last_updated: 2026-08-03
---

# Chapter 7.3.1 — User Portal Overview

> *The User Portal is the primary digital workspace where individuals interact daily with the EVOXA ecosystem through personalized experiences powered by Artificial Intelligence.*

---

# Executive Summary

The User Portal provides a unified, personalized and AI-assisted environment that supports users throughout their health journey.

Every experience is centered on improving long-term health outcomes through intelligent recommendations, scientific evidence and continuous feedback.

The portal integrates nutrition, training, recovery, goals, progress tracking, community interaction and AI coaching into a single experience.

---

# Purpose

The User Portal exists to:

- Centralize all user activities.
- Provide personalized health guidance.
- Track long-term progress.
- Deliver AI-assisted recommendations.
- Encourage sustainable healthy habits.
- Improve engagement through continuous feedback.

---

# Product Vision

The User Portal shall become the user's daily health companion.

Every interaction should provide value, reduce friction and help users make better decisions with confidence.

Rather than presenting isolated tools, the portal delivers a cohesive experience that adapts to the user's goals, preferences and progress.

---

# Design Principles

The User Portal follows the Web Platform principles and extends them with user-focused objectives.

| Principle | Description |
|-----------|-------------|
| Personalized | Every experience adapts to the individual. |
| Action-Oriented | Interfaces guide users toward meaningful actions. |
| Explainable | Recommendations always include context. |
| Predictive | The platform anticipates future needs. |
| Continuous | Progress is visible over time. |
| Motivational | Positive reinforcement encourages adherence. |
| Accessible | Every feature supports inclusive interaction. |

---

# Supported Personas

| Persona | Description |
|----------|-------------|
| Member | Uses EVOXA to improve health. |
| Athlete | Tracks performance and training. |
| Weight Management User | Focuses on nutrition and body composition. |
| Chronic Care User | Monitors long-term health indicators. |
| Beginner | Requires guided onboarding and education. |

---

# Experience Goals

The portal shall enable users to:

- Understand today's priorities.
- Follow personalized nutrition plans.
- Complete scheduled workouts.
- Monitor recovery.
- Receive proactive AI guidance.
- Track progress.
- Celebrate achievements.
- Stay motivated over time.

---

# Domain Feature Packs

The User Portal is organized into Domain Feature Packs.

| Domain Feature Pack | Prefix |
|---------------------|--------|
| Dashboard | DFP-DASH |
| Nutrition | DFP-NUT |
| Training | DFP-TRN |
| Recovery | DFP-REC |
| AI Coach | DFP-AI |
| Goals | DFP-GOAL |
| Progress | DFP-PROG |
| Calendar | DFP-CAL |
| Community | DFP-COM |
| Profile | DFP-PROFILE |
| Settings | DFP-SET |

Each Domain Feature Pack contains one or more Feature Packs describing complete user capabilities.

---

# Portal Architecture

```text
User Portal
│
├── Dashboard
├── Nutrition
├── Training
├── Recovery
├── AI Coach
├── Goals
├── Progress
├── Calendar
├── Community
├── Achievements
├── Notifications
├── Profile
└── Settings
```

---

# Navigation Model

```text
Dashboard
│
├── Nutrition
├── Training
├── Recovery
├── AI Coach
├── Goals
├── Progress
├── Calendar
├── Community
├── Notifications
├── Profile
└── Settings
```

Navigation shall remain consistent across desktop and tablet layouts.

---

# Shared Layout

```text
LAY-0002

Authenticated Layout

┌──────────────────────────────────────────┐
│ Global Header                            │
├───────────────┬──────────────────────────┤
│ Sidebar       │ Main Workspace           │
│               │                          │
│               │                          │
├───────────────┴──────────────────────────┤
│ Status Bar / Notifications               │
└──────────────────────────────────────────┘
```

---

# Master Screen Inventory

| Range | Domain |
|--------|--------|
| SCR-1000–1099 | Dashboard |
| SCR-1100–1199 | Nutrition |
| SCR-1200–1299 | Training |
| SCR-1300–1399 | Recovery |
| SCR-1400–1499 | AI Coach |
| SCR-1500–1599 | Goals |
| SCR-1600–1699 | Progress |
| SCR-1700–1799 | Calendar |
| SCR-1800–1899 | Community |
| SCR-1900–1999 | Profile & Settings |

The ranges reserve identifiers for future growth while preserving catalog stability.

---

# Shared Components

The following components are reused across the portal:

- Global Header
- Navigation Sidebar
- AI Assistant Panel
- Notification Center
- Search Bar
- Breadcrumb
- Quick Actions
- Status Banner
- User Avatar
- Footer

These components are defined centrally and reused by all Feature Packs.

---

# AI Integration

The User Portal integrates AI across all experiences.

Capabilities include:

- Daily summaries
- Nutrition recommendations
- Training adjustments
- Recovery insights
- Habit coaching
- Goal optimization
- Contextual explanations
- Natural language search

AI shall always provide explainable recommendations.

---

# Data Flow

```text
User Actions

↓

Business Domains

↓

Human Digital Twin

↓

AI Orchestrator

↓

Personalized Recommendations

↓

User Interface
```

---

# Security Model

Authentication is provided by the Identity Platform.

Authorization is role-based and permission-driven.

Sensitive health information shall always be protected according to platform security policies.

---

# Analytics

The User Portal publishes analytics for:

- Screen views
- Workflow completion
- Feature usage
- AI interactions
- Goal progress
- Session duration
- User engagement

Analytics are consumed by the Analytics Platform.

---

# Standard Visual Artifacts

## Portal Map

```text
Dashboard
│
├── Nutrition
├── Training
├── Recovery
├── AI Coach
├── Goals
├── Progress
├── Calendar
├── Community
└── Profile
```

---

## Experience Flow

```text
Login

↓

Dashboard

↓

Daily Activities

↓

Recommendations

↓

Progress Review

↓

Continuous Improvement
```

---

## Domain Relationship Diagram

```text
Dashboard

↓

Nutrition

↓

Training

↓

Recovery

↓

AI Coach

↓

Progress
```

---

# Visual Source Files

```text
artifacts/
└── chapter-01/
    ├── portal-map.drawio
    ├── navigation.drawio
    ├── experience-flow.drawio
    ├── domain-map.drawio
    ├── layout.drawio
    ├── mermaid/
    │   ├── portal-map.mmd
    │   ├── navigation.mmd
    │   ├── experience-flow.mmd
    │   └── domains.mmd
    ├── wireframes/
    │   └── authenticated-layout.fig
    └── exports/
        ├── *.svg
        ├── *.png
        └── *.pdf
```

---

# Product Design Package

This overview defines the parent specification for:

- Dashboard
- Nutrition
- Training
- Recovery
- AI Coach
- Goals
- Progress
- Calendar
- Community
- Profile
- Settings

Each domain expands into dedicated Feature Packs and Engineering Specifications.

---

# Implementation Readiness

## Parent Feature Packs

- DFP-DASH
- DFP-NUT
- DFP-TRN
- DFP-REC
- DFP-AI
- DFP-GOAL
- DFP-PROG
- DFP-CAL
- DFP-COM
- DFP-PROFILE
- DFP-SET

## Related Specifications

- ESP-WEB-1000 — User Portal
- DSB-0001 — Design System
- EES-UI-0001 — UI Standards

---

# Key Takeaways

- The User Portal is the primary daily workspace for EVOXA users.
- Domain Feature Packs organize functionality by business capability.
- Every screen, component, workflow and API belongs to a traceable structure.
- AI is integrated across the entire experience while remaining explainable.
- This overview serves as the parent specification for every User Portal module.

---

# Next Chapter

## Chapter 7.3.2 — Dashboard

The next chapter defines the Dashboard Feature Pack, including its screen inventory, widgets, AI summaries, daily plan, navigation, APIs, events, analytics, permissions and engineering specifications.
