---
document_id: BP-0003-V3-C07-04-09
chapter_id: CH-04-COACH-09
feature_pack: FP-COACH-0000
title: Layout Architecture
version: 1.0.0
status: Draft
owner: UX Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-04
---

# Chapter 09 — Layout Architecture

> *The Layout Architecture defines the structural organization of every workspace within the Coach Portal, ensuring a consistent, responsive and productivity-oriented interface for professional users.*

---

# Executive Summary

The Coach Portal is designed as a Professional Operating System where layout consistency, contextual awareness and information density are optimized for long-duration professional use.

Rather than relying on isolated page designs, the platform provides a reusable layout architecture that supports every workspace while adapting to user roles, workflows and AI-driven recommendations.

---

# Objectives

The Layout Architecture shall:

- Standardize every workspace.
- Improve productivity.
- Reduce navigation effort.
- Support adaptive layouts.
- Enable contextual AI panels.
- Maintain responsive behavior.
- Scale across future modules.

---

# Layout Philosophy

Every screen follows a common structural model.

```text
Global Shell

↓

Workspace

↓

Panels

↓

Components

↓

Widgets
```

The layout separates navigation, content and contextual intelligence into independent regions.

---

# Layout Principles

The Coach Portal follows these principles.

- Workspace First
- Productivity First
- Context Awareness
- Consistent Navigation
- Responsive Design
- Progressive Disclosure
- AI-Native Interface

---

# Global Layout

```text
┌────────────────────────────────────────────────────────────┐
│ Global Header                                              │
├──────────────┬───────────────────────────────┬─────────────┤
│              │                               │             │
│ Navigation   │      Main Workspace           │ AI Panel    │
│ Sidebar      │                               │             │
│              │                               │             │
├──────────────┴───────────────────────────────┴─────────────┤
│ Status Bar / Notifications                                │
└────────────────────────────────────────────────────────────┘
```

---

# Layout Regions

## Global Header

Provides:

- Organization selector.
- Global search.
- Notifications.
- User profile.
- Quick actions.
- Workspace switcher.

---

## Navigation Sidebar

Contains:

- Dashboard.
- Clients.
- Assessments.
- Training.
- Nutrition.
- Calendar.
- Messaging.
- Analytics.
- Billing.
- Marketplace.
- Administration.

The sidebar supports collapse and pin modes.

---

## Main Workspace

Displays:

- Active workflow.
- Business content.
- Forms.
- Dashboards.
- Reports.
- Client information.

Only one primary task is emphasized at a time.

---

## Context Panel

The right-side panel provides:

- EVO recommendations.
- Human Digital Twin insights.
- Recent activity.
- Related tasks.
- Alerts.
- Quick actions.

The panel adapts dynamically to the active context.

---

## Footer / Status Bar

Displays:

- Synchronization status.
- Background jobs.
- AI processing indicators.
- Notifications.
- Connectivity.
- Version information.

---

# Workspace Templates

## Dashboard Layout

```text
Header

↓

KPIs

↓

Today's Agenda

↓

AI Priorities

↓

Tasks

↓

Recent Activity
```

---

## Client Workspace

```text
Client Summary

↓

Digital Twin

↓

Tabs

↓

Content

↓

AI Insights
```

---

## Analytics Workspace

```text
Filters

↓

Charts

↓

KPIs

↓

Tables

↓

Reports
```

---

## Program Builder

```text
Client

↓

Objectives

↓

Program Builder

↓

Preview

↓

Assignment
```

---

# Adaptive Layouts

The layout adapts according to:

- Professional role.
- Selected client.
- Screen size.
- Current workflow.
- Active AI suggestions.
- Organization settings.

---

# Multi-Panel Strategy

The Coach Portal supports three simultaneous panels.

```text
Navigation

↓

Workspace

↓

AI Context
```

Panels may be resized or collapsed depending on user preference.

---

# Context Preservation

When switching between screens:

- Selected client is preserved.
- Active filters remain.
- Open tabs remain.
- Unsaved drafts are retained.
- AI context is maintained.

---

# Layout Personalization

Professionals may configure:

- Sidebar behavior.
- Widget positions.
- Dashboard layouts.
- Workspace density.
- Panel sizes.
- Favorite shortcuts.

Preferences are synchronized across devices.

---

# Responsive Architecture

## Desktop

Three-column layout.

---

## Tablet

Two-column layout.

---

## Mobile

Single-column adaptive layout with contextual bottom navigation.

---

# Information Density

The interface supports:

- Compact mode.
- Standard mode.
- Comfortable mode.

Density settings affect tables, cards and lists while preserving accessibility.

---

# Layout States

Every workspace supports:

- Loading.
- Empty.
- Populated.
- Error.
- Offline.
- Read-only.

Each state has a consistent visual treatment.

---

# Navigation Flow

```text
Dashboard

↓

Workspace

↓

Client

↓

Task

↓

Result
```

---

# Layout Governance

Every new screen shall:

- Use a standard template.
- Respect spacing tokens.
- Follow responsive rules.
- Support accessibility.
- Include AI context areas where applicable.

Custom layouts require Architecture Board approval.

---

# Standard Visual Artifacts

## Global Layout

```text
Header

↓

Navigation

↓

Workspace

↓

AI Panel
```

---

## Workspace Template

```text
Summary

↓

Tabs

↓

Content

↓

Insights
```

---

## Adaptive Layout

```text
Role

↓

Context

↓

Layout

↓

Interaction
```

---

## Responsive Grid

```text
Desktop

↓

Tablet

↓

Mobile
```

---

# Visual Source Files

```text
artifacts/
└── layout-architecture/
    ├── global-layout.drawio
    ├── workspace-template.drawio
    ├── adaptive-layout.drawio
    ├── responsive-grid.drawio
    ├── context-panel.drawio
    ├── dashboard-layout.drawio
    ├── mermaid/
    │   ├── layout.mmd
    │   ├── workspace.mmd
    │   ├── adaptive.mmd
    │   ├── responsive.mmd
    │   └── navigation.mmd
    └── exports/
        ├── *.svg
        ├── *.png
        └── *.pdf
```

---

# Repository Structure

```text
layout/
├── templates/
├── workspaces/
├── panels/
├── responsive/
├── personalization/
├── ai-context/
├── grids/
└── metadata.yml
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Information Architecture | Chapter 06 |
| UX Goals | Chapter 07 |
| Screen Catalog | Chapter 08 |
| Navigation Architecture | Chapter 10 |
| Design Principles | Chapter 11 |
| Responsive Design | Chapter 15 |
| AI Services | Chapter 21 |

---

# Acceptance Criteria

This chapter is complete when:

- Global layout is defined.
- Workspace templates are documented.
- Adaptive layout behavior is specified.
- Responsive rules are established.
- Context preservation is documented.
- Layout personalization is supported.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- The Coach Portal adopts a workspace-based layout architecture optimized for professionals who spend extended periods using the platform.
- A consistent global shell, contextual AI panel and adaptive workspace templates improve productivity while reducing cognitive load.
- Layout personalization, responsive behavior and context preservation enable efficient task execution across devices and professional roles.
- Standardized layout governance ensures visual consistency and supports future platform evolution.

---

# Next Chapter

**Chapter 10 — Navigation Architecture**

This chapter defines the navigation model of the Coach Portal, including global navigation, contextual navigation, deep linking, search-driven navigation, breadcrumbs, keyboard shortcuts and adaptive navigation patterns.
