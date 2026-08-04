---
document_id: BP-0003-V3-C07-04-08
chapter_id: CH-04-COACH-08
feature_pack: FP-COACH-0000
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

> *The Screen Catalog defines every screen, workspace and operational interface available within the Coach Portal, providing a complete inventory of professional user experiences, navigation paths and AI-assisted interactions.*

---

# Executive Summary

The Coach Portal is composed of multiple interconnected workspaces designed around professional workflows.

Rather than organizing screens by technical modules, the platform groups interfaces into business-oriented workspaces that enable professionals to complete end-to-end operational tasks efficiently.

Each screen has a clearly defined responsibility, ownership, permission model and relationship with business workflows, AI services and data domains.

---

# Objectives

The Screen Catalog shall:

- Define every screen.
- Standardize naming.
- Support modular navigation.
- Improve discoverability.
- Enable UX consistency.
- Define ownership.
- Support future expansion.

---

# Screen Architecture

```text
Portal

↓

Workspace

↓

Screen

↓

Section

↓

Component

↓

Widget
```

---

# Workspace Catalog

| Workspace | Purpose |
|------------|----------|
| Dashboard | Operational overview |
| Clients | Client lifecycle |
| Assessments | Health evaluations |
| Training | Workout management |
| Nutrition | Nutrition management |
| Calendar | Scheduling |
| Messaging | Communication |
| Video Sessions | Telecoaching |
| AI Copilot | AI assistance |
| Analytics | Insights |
| Billing | Financial operations |
| Marketplace | Resources |
| Administration | Configuration |

---

# Screen Classification

| Type | Description |
|--------|-------------|
| Landing | Entry screen |
| List | Collection of entities |
| Detail | Individual entity |
| Wizard | Guided workflow |
| Dashboard | Operational summary |
| Workspace | Business environment |
| Configuration | Settings |
| Analytics | KPI visualization |

---

# Dashboard Workspace

| Screen ID | Screen |
|------------|---------|
| SCR-COACH-001 | Main Dashboard |
| SCR-COACH-002 | Today's Agenda |
| SCR-COACH-003 | AI Priorities |
| SCR-COACH-004 | Pending Tasks |
| SCR-COACH-005 | Business KPIs |

---

# Client Workspace

| Screen ID | Screen |
|------------|---------|
| SCR-COACH-100 | Client Directory |
| SCR-COACH-101 | Client Profile |
| SCR-COACH-102 | Human Digital Twin |
| SCR-COACH-103 | Goals |
| SCR-COACH-104 | Timeline |
| SCR-COACH-105 | Progress |
| SCR-COACH-106 | Documents |
| SCR-COACH-107 | Notes |
| SCR-COACH-108 | Activity History |

---

# Assessment Workspace

| Screen ID | Screen |
|------------|---------|
| SCR-COACH-200 | Assessment Dashboard |
| SCR-COACH-201 | Initial Assessment |
| SCR-COACH-202 | Body Composition |
| SCR-COACH-203 | Physical Evaluation |
| SCR-COACH-204 | Lifestyle Assessment |
| SCR-COACH-205 | Medical History |

---

# Training Workspace

| Screen ID | Screen |
|------------|---------|
| SCR-COACH-300 | Training Library |
| SCR-COACH-301 | Program Builder |
| SCR-COACH-302 | Exercise Library |
| SCR-COACH-303 | Weekly Planner |
| SCR-COACH-304 | Session Detail |
| SCR-COACH-305 | Progression Planner |

---

# Nutrition Workspace

| Screen ID | Screen |
|------------|---------|
| SCR-COACH-400 | Nutrition Dashboard |
| SCR-COACH-401 | Meal Planner |
| SCR-COACH-402 | Food Database |
| SCR-COACH-403 | Supplement Plans |
| SCR-COACH-404 | Nutrition Adherence |

---

# Calendar Workspace

| Screen ID | Screen |
|------------|---------|
| SCR-COACH-500 | Calendar |
| SCR-COACH-501 | Daily Agenda |
| SCR-COACH-502 | Appointment Detail |
| SCR-COACH-503 | Availability |
| SCR-COACH-504 | Recurring Sessions |

---

# Messaging Workspace

| Screen ID | Screen |
|------------|---------|
| SCR-COACH-600 | Inbox |
| SCR-COACH-601 | Conversation |
| SCR-COACH-602 | Broadcast Messages |
| SCR-COACH-603 | Attachments |

---

# Video Workspace

| Screen ID | Screen |
|------------|---------|
| SCR-COACH-700 | Session Lobby |
| SCR-COACH-701 | Live Session |
| SCR-COACH-702 | Session Summary |

---

# AI Workspace

| Screen ID | Screen |
|------------|---------|
| SCR-COACH-800 | EVO Copilot |
| SCR-COACH-801 | AI Recommendations |
| SCR-COACH-802 | Risk Center |
| SCR-COACH-803 | Client Insights |
| SCR-COACH-804 | AI History |

---

# Analytics Workspace

| Screen ID | Screen |
|------------|---------|
| SCR-COACH-900 | Analytics Dashboard |
| SCR-COACH-901 | Client KPIs |
| SCR-COACH-902 | Business KPIs |
| SCR-COACH-903 | Retention Analysis |
| SCR-COACH-904 | AI Analytics |

---

# Billing Workspace

| Screen ID | Screen |
|------------|---------|
| SCR-COACH-1000 | Billing Dashboard |
| SCR-COACH-1001 | Invoices |
| SCR-COACH-1002 | Subscriptions |
| SCR-COACH-1003 | Payments |

---

# Marketplace Workspace

| Screen ID | Screen |
|------------|---------|
| SCR-COACH-1100 | Marketplace |
| SCR-COACH-1101 | Templates |
| SCR-COACH-1102 | Integrations |

---

# Administration Workspace

| Screen ID | Screen |
|------------|---------|
| SCR-COACH-1200 | Settings |
| SCR-COACH-1201 | Users |
| SCR-COACH-1202 | Permissions |
| SCR-COACH-1203 | Organization |
| SCR-COACH-1204 | Audit Logs |

---

# Screen Metadata

Each screen defines:

- Screen ID
- Workspace
- Owner
- Description
- Primary Persona
- Permissions
- Related Workflows
- Related APIs
- Events
- Components
- AI Capabilities
- Responsive Behavior

---

# Navigation Relationships

```text
Dashboard

↓

Workspace

↓

Screen

↓

Detail

↓

Actions
```

---

# Screen Lifecycle

```text
Draft

↓

Designed

↓

Implemented

↓

Validated

↓

Released
```

---

# Future Screens

Future workspaces include:

- Wearables
- Laboratory Results
- Clinical Decision Support
- Research
- Insurance
- AI Agents
- Population Health

---

# Standard Visual Artifacts

## Workspace Map

```text
Dashboard

↓

Clients

↓

Programs

↓

Analytics
```

---

## Screen Hierarchy

```text
Workspace

↓

Screen

↓

Section

↓

Component
```

---

## Navigation Graph

```text
Dashboard

↓

Workspace

↓

Screen

↓

Action
```

---

## Professional Workspace

```text
Coach

↓

Clients

↓

AI

↓

Programs

↓

Results
```

---

# Visual Source Files

```text
artifacts/
└── screen-catalog/
    ├── workspace-map.drawio
    ├── navigation-graph.drawio
    ├── screen-hierarchy.drawio
    ├── professional-workspace.drawio
    ├── dashboard-layout.drawio
    ├── catalog-overview.drawio
    ├── mermaid/
    │   ├── workspaces.mmd
    │   ├── navigation.mmd
    │   ├── hierarchy.mmd
    │   ├── catalog.mmd
    │   └── relationships.mmd
    └── exports/
        ├── *.svg
        ├── *.png
        └── *.pdf
```

---

# Repository Structure

```text
catalog/
├── workspaces/
├── screens/
├── navigation/
├── metadata/
├── permissions/
├── layouts/
├── ai/
└── diagrams/
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Information Architecture | Chapter 06 |
| UX Goals | Chapter 07 |
| Layout Architecture | Chapter 09 |
| Navigation Architecture | Chapter 10 |
| Component Catalog | Chapter 12 |
| Widget Catalog | Chapter 13 |
| Workflow Architecture | Chapter 16 |

---

# Acceptance Criteria

This chapter is complete when:

- All workspaces are identified.
- Screens are cataloged.
- Screen ownership is defined.
- Navigation relationships are documented.
- Metadata standards are established.
- Future expansion is supported.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- The Coach Portal organizes its interface into business-oriented workspaces rather than isolated application modules.
- Every screen has a defined purpose, ownership, navigation context and relationship with workflows, AI services and business domains.
- A standardized catalog simplifies UX governance, implementation, testing and future platform expansion.
- The workspace model enables professionals to perform complex operational tasks efficiently while maintaining a consistent user experience.

---

# Next Chapter

**Chapter 09 — Layout Architecture**

This chapter defines the structural layout of every workspace, including page templates, responsive grids, navigation regions, panel composition, adaptive workspaces and layout governance.
