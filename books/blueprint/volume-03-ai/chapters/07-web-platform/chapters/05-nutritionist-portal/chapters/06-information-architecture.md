---
document_id: BP-0003-V3-C07-04-06
chapter_id: CH-04-COACH-06
feature_pack: FP-COACH-0000
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

> *The Information Architecture defines how information is organized, structured and presented throughout the Coach Portal, enabling professionals to efficiently navigate, manage clients and access contextual information.*

---

# Executive Summary

The Coach Portal manages large volumes of operational, clinical and business information.

Its Information Architecture is designed around professional workflows rather than application modules, ensuring that users can quickly locate information, complete tasks and collaborate with EVO.

The architecture is modular, scalable and role-aware.

---

# Objectives

The Information Architecture shall:

- Organize information around business domains.
- Reduce cognitive load.
- Improve discoverability.
- Support role-based navigation.
- Enable contextual AI assistance.
- Scale with future capabilities.
- Maintain consistency across modules.

---

# Information Architecture Principles

The Coach Portal follows these principles:

- Task-Oriented Navigation.
- Client-Centered Information.
- Progressive Disclosure.
- Context Awareness.
- Consistency.
- Predictability.
- Scalability.

---

# Information Hierarchy

```text
Platform

↓

Portal

↓

Capability Domain

↓

Module

↓

Feature

↓

Screen

↓

Component
```

---

# Capability Domains

The Coach Portal is organized into strategic capability domains.

| Domain | Purpose |
|---------|----------|
| Dashboard | Operational overview |
| Client Care | Client lifecycle management |
| Programs | Training and nutrition |
| Communication | Messaging and video sessions |
| Scheduling | Calendar and appointments |
| AI Copilot | EVO recommendations |
| Analytics | Performance and KPIs |
| Business Operations | Billing and reports |
| Administration | Settings and configuration |

---

# Navigation Model

```text
Dashboard

├── Clients
│   ├── Profile
│   ├── Human Digital Twin
│   ├── Assessments
│   ├── Goals
│   ├── Programs
│   ├── Nutrition
│   ├── Progress
│   ├── Files
│   └── Timeline
│
├── Calendar
│
├── Messaging
│
├── AI Assistant
│
├── Analytics
│
├── Billing
│
├── Marketplace
│
└── Settings
```

---

# Client-Centered Architecture

Every client acts as a central information hub.

```text
Client

↓

Profile

↓

Digital Twin

↓

Assessments

↓

Programs

↓

Goals

↓

Messages

↓

Reports
```

The client becomes the primary navigation context for professionals.

---

# Information Ownership

| Information | Owner |
|-------------|-------|
| Client Profile | Client Domain |
| Human Digital Twin | Platform Core |
| Training Plans | Training Domain |
| Nutrition Plans | Nutrition Domain |
| Appointments | Calendar Domain |
| Messages | Communication Domain |
| Analytics | Analytics Platform |
| Billing | Billing Platform |

Ownership is explicit and aligned with Domain-Driven Design.

---

# Navigation Levels

## Level 1

Global Navigation

Examples:

- Dashboard
- Clients
- Calendar
- Analytics

---

## Level 2

Module Navigation

Examples:

- Assessments
- Training
- Nutrition
- Goals

---

## Level 3

Context Navigation

Examples:

- Selected Client
- Selected Program
- Selected Appointment

---

# Search Architecture

Global search supports:

- Clients.
- Programs.
- Exercises.
- Nutrition plans.
- Assessments.
- Reports.
- Messages.

Search results respect role-based permissions.

---

# Contextual Navigation

Navigation adapts according to:

- Active client.
- Current workflow.
- Professional role.
- Organization.
- Permissions.
- AI recommendations.

---

# AI Information Layer

EVO enriches information with:

- Context summaries.
- Recommendations.
- Risk indicators.
- Suggested actions.
- Related insights.

AI augments existing information rather than replacing it.

---

# Content Taxonomy

| Category | Examples |
|-----------|----------|
| Client Data | Profile, goals |
| Assessments | Measurements, evaluations |
| Programs | Workouts, nutrition |
| Communication | Messages, calls |
| Analytics | KPIs, reports |
| Administration | Settings, permissions |

---

# Metadata Strategy

Every entity includes:

- Identifier.
- Owner.
- Status.
- Version.
- Tags.
- Created Date.
- Updated Date.
- Visibility.
- Classification.

---

# Information Relationships

```text
Coach

↓

Clients

↓

Programs

↓

Sessions

↓

Results

↓

Analytics
```

---

# Information Lifecycle

```text
Created

↓

Validated

↓

Active

↓

Archived

↓

Deleted
```

---

# Discoverability

Information shall be accessible through:

- Navigation.
- Search.
- AI Assistant.
- Shortcuts.
- Dashboard widgets.
- Notifications.

---

# Personalization

The architecture adapts to:

- Professional role.
- Favorite modules.
- Recent activity.
- Active clients.
- AI recommendations.

---

# Scalability

Future modules integrate through the same information hierarchy.

Examples:

- Telemedicine.
- Laboratory Results.
- Wearable Devices.
- Research Studies.
- Insurance Integration.

No navigation redesign is required.

---

# Standard Visual Artifacts

## Information Hierarchy

```text
Platform

↓

Domain

↓

Module

↓

Screen
```

---

## Navigation Map

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

## Client Information Tree

```text
Client

↓

Digital Twin

↓

Programs

↓

Results
```

---

## Search Architecture

```text
Search

↓

Index

↓

Results

↓

Actions
```

---

# Visual Source Files

```text
artifacts/
└── information-architecture/
    ├── navigation-map.drawio
    ├── client-information-tree.drawio
    ├── capability-domains.drawio
    ├── information-flow.drawio
    ├── search-architecture.drawio
    ├── metadata-model.drawio
    ├── mermaid/
    │   ├── hierarchy.mmd
    │   ├── navigation.mmd
    │   ├── taxonomy.mmd
    │   ├── search.mmd
    │   └── ownership.mmd
    └── exports/
        ├── *.svg
        ├── *.png
        └── *.pdf
```

---

# Repository Structure

```text
catalog/
├── capability-domains/
├── navigation/
├── taxonomy/
├── search/
├── metadata/
├── ownership/
├── relationships/
└── diagrams/
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| User Journeys | Chapter 05 |
| UX Goals | Chapter 07 |
| Screen Catalog | Chapter 08 |
| Layout Architecture | Chapter 09 |
| Navigation Architecture | Chapter 10 |
| Workflow Architecture | Chapter 16 |
| Data Model | Chapter 22 |

---

# Acceptance Criteria

This chapter is complete when:

- Information hierarchy is defined.
- Capability domains are documented.
- Navigation levels are established.
- Search strategy is specified.
- Content taxonomy is documented.
- Metadata strategy is defined.
- Information ownership is assigned.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- The Information Architecture organizes the Coach Portal around professional workflows and client-centric contexts rather than isolated application modules.
- Capability domains, contextual navigation and metadata provide a scalable foundation for current and future platform features.
- AI enriches information with contextual insights while preserving a predictable and intuitive navigation experience.
- The architecture supports rapid information discovery, efficient task completion and long-term platform evolution.

---

# Next Chapter

**Chapter 07 — UX Goals**

This chapter defines the user experience objectives of the Coach Portal, including usability principles, interaction goals, productivity metrics, AI-assisted experiences and design success criteria.
