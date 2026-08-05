---
document_id: BP-0003-V3-C07-05-10
chapter_id: CH-05-NUTRITION-10
feature_pack: FP-NUTRITION-0000
title: Navigation Architecture
version: 1.0.0
status: Draft
owner: UX Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-04
---

# Chapter 10 — Navigation Architecture

> *The Navigation Architecture chapter defines how users move throughout the Nutritionist Portal, ensuring intuitive, contextual and efficient navigation across professional workflows while maintaining consistency with the EVOXA ecosystem.*

---

# Executive Summary

The Navigation Architecture provides a structured navigation model that supports complex nutritional workflows while minimizing cognitive load.

Navigation is organized around professional tasks rather than technical modules, allowing nutritionists to move naturally between assessments, meal planning, client monitoring and AI-assisted recommendations.

The architecture combines persistent global navigation, contextual navigation and workflow-driven transitions.

---

# Objectives

The Navigation Architecture shall:

- Reduce navigation complexity.
- Minimize clicks.
- Preserve user context.
- Support professional workflows.
- Enable contextual navigation.
- Integrate AI naturally.
- Scale across future modules.

---

# Navigation Philosophy

Navigation follows four principles:

- Context
- Continuity
- Predictability
- Efficiency

Users should always know:

- Where they are.
- What they are doing.
- What they can do next.
- How to return.

---

# Navigation Layers

The portal implements five navigation layers.

```text
Global Navigation

↓

Domain Navigation

↓

Context Navigation

↓

Task Navigation

↓

Action Navigation
```

Each layer addresses a specific navigation need.

---

# Global Navigation

Global Navigation is always available.

Primary sections:

- Dashboard
- Clients
- Assessments
- Meal Plans
- Recipes
- Food Database
- Supplements
- Shopping Lists
- Reports
- Calendar
- Messages
- AI Workspace
- Settings

---

# Domain Navigation

Each domain exposes its own navigation.

Example:

Client

↓

Overview

Assessments

Meal Plans

Progress

Reports

Messages

Digital Twin

---

# Context Navigation

Context changes according to:

- Current client.
- Current assessment.
- Meal plan.
- Active consultation.
- AI conversation.

The current context is always visible.

---

# Task Navigation

Task navigation guides users through workflows.

Example:

```text
Assessment

↓

Diagnosis

↓

Meal Plan

↓

Review

↓

Publish
```

Progress indicators show the current step.

---

# Action Navigation

Every screen exposes primary actions.

Examples:

- Save
- Publish
- Generate
- Export
- Share
- Approve
- AI Suggest

Primary actions remain visually prominent.

---

# Navigation Components

The portal uses:

- Sidebar
- Top Navigation
- Breadcrumbs
- Tabs
- Stepper
- Context Panel
- Quick Actions
- Floating Action Button (mobile)

---

# Sidebar Navigation

The sidebar supports:

- Expand/Collapse
- Favorites
- Recently Used
- Search
- Organization switching

Navigation state persists across sessions.

---

# Breadcrumb Navigation

Example:

```text
Dashboard

>

Clients

>

John Smith

>

Assessment

>

Follow-up
```

Breadcrumbs support rapid navigation without losing context.

---

# Tabs

Tabs organize related information.

Example:

Client Profile

- Overview
- History
- Assessments
- Meal Plans
- Progress
- Messages
- Files
- Digital Twin

Tabs preserve state during navigation.

---

# Search Navigation

Global search provides direct access to:

- Clients
- Foods
- Recipes
- Meal Plans
- Reports
- Messages
- AI Conversations

Search supports:

- Full-text search
- Semantic search
- Filters
- Saved searches
- Recent searches

---

# Quick Navigation

Frequently used shortcuts include:

- New Client
- New Assessment
- New Meal Plan
- AI Assistant
- Today's Schedule
- Reports

Shortcuts are configurable.

---

# Workflow Navigation

Typical workflow:

```text
Client

↓

Assessment

↓

Meal Plan

↓

Shopping List

↓

Monitoring

↓

Reports
```

Users complete workflows without unnecessary navigation.

---

# AI Navigation

The AI Workspace integrates into every workflow.

Examples:

Assessment

↓

AI Analysis

↓

Recommendations

↓

Approval

↓

Continue Workflow

AI assistance never forces users to leave the current screen.

---

# Human Digital Twin Navigation

Every client workspace includes quick access to:

- Digital Twin Summary
- Nutrition History
- Body Composition
- Predictions
- Behavioral Trends

The Digital Twin remains available across all client-related workflows.

---

# Cross-Portal Navigation

Authorized users may navigate to:

- User Portal
- Coach Portal
- Enterprise Portal
- Admin Portal

Context is preserved when supported.

---

# Deep Linking

Every screen supports unique URLs.

Examples:

```text
/client/{id}

/assessment/{id}

/meal-plan/{id}

/report/{id}
```

Links can be shared securely according to permissions.

---

# Navigation State

The portal preserves:

- Current client
- Filters
- Sorting
- Scroll position
- Selected tabs
- Search queries

This reduces unnecessary repetition.

---

# Notifications Navigation

Notifications provide contextual links to:

- Assessments
- Reports
- Messages
- Meal Plans
- AI Recommendations

Each notification opens the relevant workspace.

---

# Mobile Navigation

Mobile uses:

- Bottom Navigation
- Slide-out Sidebar
- Floating Action Button
- Gesture support

Navigation remains consistent with desktop workflows.

---

# Accessibility

Navigation supports:

- Keyboard shortcuts
- Logical tab order
- Screen readers
- Skip navigation links
- Focus indicators

---

# Navigation Metrics

Measured indicators:

| Metric | Target |
|----------|---------|
| Average Clicks per Workflow | <6 |
| Navigation Success Rate | >95% |
| Client Search Time | <5 sec |
| Screen Transition | <300 ms |
| Breadcrumb Usage | Tracked |
| AI Shortcut Usage | Tracked |

---

# Navigation Governance

Every navigation element shall:

- Follow Design System standards.
- Preserve context.
- Support responsive behavior.
- Respect permissions.
- Maintain accessibility compliance.

---

# Repository Structure

```text
navigation/
├── global/
├── domains/
├── workflows/
├── breadcrumbs/
├── search/
├── shortcuts/
├── mobile/
├── accessibility/
├── governance/
└── metadata.yml
```

---

# Standard Visual Artifacts

## Navigation Layers

```text
Global

↓

Domain

↓

Context

↓

Task

↓

Action
```

---

## Client Navigation

```text
Client

↓

Assessment

↓

Meal Plan

↓

Monitoring
```

---

## Workflow Navigation

```text
Start

↓

Task

↓

Review

↓

Complete
```

---

## Cross-Portal Navigation

```text
Nutritionist Portal

↓

Coach Portal

↓

User Portal

↓

Enterprise Portal
```

---

# Visual Source Files

```text
artifacts/
└── navigation-architecture/
    ├── navigation-layers.drawio
    ├── sidebar.drawio
    ├── breadcrumbs.drawio
    ├── workflow-navigation.drawio
    ├── cross-portal.drawio
    ├── mobile-navigation.drawio
    ├── mermaid/
    │   ├── navigation.mmd
    │   ├── workflows.mmd
    │   ├── breadcrumbs.mmd
    │   ├── mobile.mmd
    │   └── cross-portal.mmd
    └── exports/
        ├── navigation-architecture.svg
        ├── navigation-architecture.png
        └── navigation-architecture.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Information Architecture | Chapter 06 |
| UX Goals | Chapter 07 |
| Screen Catalog | Chapter 08 |
| Layout Architecture | Chapter 09 |
| Design Principles | Chapter 11 |
| Responsive Design | Chapter 15 |
| Workflow Architecture | Chapter 16 |

---

# Acceptance Criteria

This chapter is complete when:

- Navigation layers are defined.
- Global and contextual navigation models are documented.
- Workflow navigation is specified.
- AI and Human Digital Twin navigation are integrated.
- Deep linking and state preservation are documented.
- Mobile and accessibility navigation are defined.
- Navigation metrics are established.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- The Navigation Architecture is centered on professional nutritional workflows rather than isolated application modules.
- Multi-layer navigation preserves user context while enabling efficient movement between clients, assessments, meal plans, progress monitoring and AI-assisted tasks.
- Shared navigation patterns, deep linking and state preservation improve usability, productivity and collaboration across the EVOXA ecosystem.
- Consistent governance, accessibility and responsive navigation ensure a scalable experience across desktop, tablet and mobile devices.

---

# Next Chapter

**Chapter 11 — Design Principles**

This chapter defines the visual, interaction and experience principles that guide every interface of the Nutritionist Portal, ensuring consistency, usability, accessibility and alignment with the EVOXA Design System.
