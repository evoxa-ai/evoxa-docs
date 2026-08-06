---
document_id: BP-0003-V3-C07-09-10
chapter_id: CH-09-OPS-10
feature_pack: FP-OPS-0000
title: Navigation Architecture
version: 1.0.0
status: Draft
owner: Platform UX Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-05
---

# Chapter 10 — Navigation Architecture

> *The Navigation Architecture chapter defines the navigation model, routing hierarchy, contextual navigation, deep-linking strategy and interaction flows that connect every operational workspace within the EVOXA Operations Portal.*

---

# Executive Summary

The Operations Portal is designed for users who must navigate rapidly between dashboards, alerts, incidents, AI services, infrastructure and operational analytics.

The navigation architecture minimizes the number of interactions required to investigate, diagnose and resolve operational events while maintaining complete situational awareness.

Navigation is hierarchical, contextual and state-aware.

---

# Objectives

The Navigation Architecture shall:

- Minimize navigation depth.
- Preserve operational context.
- Enable rapid investigations.
- Support contextual exploration.
- Reduce cognitive load.
- Enable deep linking.
- Scale with future operational modules.

---

# Navigation Principles

The portal follows:

- Context First
- Three-Click Rule
- Persistent Navigation
- Progressive Disclosure
- State Preservation
- Search Everywhere
- Cross-Link Everything
- Predictable Navigation

---

# Navigation Hierarchy

```text
Operations Portal

↓

Operational Domain

↓

Module

↓

Workspace

↓

Entity

↓

Operational Detail
```

---

# Navigation Layers

The portal contains five navigation layers.

## Layer 1 — Global Navigation

Accessible from every screen.

Includes:

- Logo
- Global Search
- Environment Selector
- Notifications
- AI Assistant
- User Menu

---

## Layer 2 — Primary Navigation

Persistent left sidebar.

Domains include:

- Executive
- Platform
- AI Operations
- Monitoring
- Incidents
- Reliability
- Security
- Deployments
- Capacity
- FinOps
- Analytics
- Administration

---

## Layer 3 — Module Navigation

Displays module-specific options.

Example:

AI Operations

↓

- Models
- Prompts
- Agents
- Workflows
- Knowledge Bases
- RAG
- MCP

---

## Layer 4 — Context Navigation

Changes according to the current workspace.

Example:

Incident

↓

Timeline

↓

Metrics

↓

Logs

↓

Traces

↓

Runbooks

---

## Layer 5 — Entity Navigation

Entity-specific navigation.

Example:

Model

↓

Versions

↓

Metrics

↓

Deployments

↓

Audit

↓

History

---

# Global Navigation

Always visible.

Contains:

- EVOXA Logo
- Search
- Environment
- Region
- Notifications
- AI Assistant
- User Profile

The global header remains fixed.

---

# Sidebar Navigation

Supports:

- Expand
- Collapse
- Favorites
- Recently Used
- Search
- Quick Access

Users may customize shortcuts.

---

# Breadcrumb Navigation

Example:

```text
Operations

>

AI Operations

>

Models

>

GPT-4 Enterprise

>

Metrics
```

Breadcrumbs support direct navigation.

---

# Context Navigation

Every operational object exposes contextual navigation.

Example:

Incident

↓

Affected Service

↓

Deployment

↓

Logs

↓

Metrics

↓

Runbook

↓

Resolution

---

# Cross-Domain Navigation

Operational relationships enable navigation across modules.

Examples:

Alert

↓

Incident

↓

Deployment

↓

Service

↓

Logs

↓

Root Cause

---

Workflow

↓

Prompt

↓

Model

↓

Knowledge Base

↓

Evaluation

---

Cluster

↓

Nodes

↓

Pods

↓

Containers

↓

Metrics

---

# Deep Linking

Every operational object supports permanent URLs.

Examples:

```text
/incidents/INC-2026-00453

/models/model-001

/workflows/wf-023

/deployments/rel-150

/alerts/ALT-000342
```

Deep links preserve:

- Filters
- Time Range
- Environment
- Selected Tabs

---

# Navigation by Search

Global Search supports:

- Services
- Incidents
- Alerts
- Models
- Agents
- Workflows
- Deployments
- Users
- Runbooks
- Documentation

Capabilities:

- Full Text Search
- Semantic Search
- Saved Searches
- Recent Searches

---

# Quick Navigation

Keyboard shortcuts include:

| Shortcut | Action |
|----------|--------|
| Ctrl + K | Global Search |
| G + D | Executive Dashboard |
| G + I | Incident Center |
| G + A | AI Operations |
| G + M | Monitoring |
| G + R | Reports |
| ? | Keyboard Help |

---

# Favorites

Users may bookmark:

- Dashboards
- Incidents
- Services
- Models
- Workflows
- Queries
- Reports

Favorites synchronize across devices.

---

# Recent Activity

Displays:

- Recent dashboards
- Recent investigations
- Recent searches
- Recently viewed entities

---

# Workspace Switching

Users may switch between:

- Production
- Staging
- QA
- Integration
- Development

Context is preserved whenever possible.

---

# Multi-Environment Navigation

Navigation always displays:

- Environment
- Region
- Tenant
- Workspace

to avoid operational mistakes.

---

# Notification Navigation

Notifications provide direct access to:

- Incident
- Alert
- Deployment
- Service
- Security Event
- AI Recommendation

---

# Investigation Navigation

Typical investigation flow:

```text
Dashboard

↓

Alert

↓

Incident

↓

Service

↓

Metrics

↓

Logs

↓

Trace

↓

Runbook

↓

Resolution
```

---

# AI Operations Navigation

```text
AI Dashboard

↓

Model

↓

Prompt

↓

Workflow

↓

Agent

↓

Evaluation

↓

Health
```

---

# Executive Navigation

Executives primarily navigate through:

- Executive Dashboard
- SLA Reports
- Cost Reports
- Operational KPIs
- Monthly Reviews

---

# Mobile Navigation

Supports:

- Bottom Navigation
- Alerts
- Incidents
- Notifications
- Executive KPIs

Advanced operational workspaces remain desktop-first.

---

# Navigation State

The portal remembers:

- Sidebar state
- Dashboard layout
- Selected filters
- Time ranges
- Search history
- Open panels

---

# Error Navigation

Unavailable resources display:

- Error description
- Related resources
- Suggested navigation
- Search shortcuts
- AI recommendations

---

# Accessibility

Navigation supports:

- Keyboard navigation
- Screen readers
- Skip navigation links
- Focus management
- WCAG 2.2 AA

---

# Performance Goals

| Navigation Operation | Target |
|----------------------|--------|
| Route Change | <300 ms |
| Search Results | <500 ms |
| Breadcrumb Update | <100 ms |
| Sidebar Expand | <150 ms |
| Dashboard Switch | <1 s |

---

# Route Organization

```text
/

├── executive
├── platform
├── ai
├── monitoring
├── incidents
├── reliability
├── security
├── deployments
├── capacity
├── finops
├── analytics
├── administration
└── settings
```

---

# Repository Structure

```text
navigation-architecture/
├── global-navigation/
├── sidebar/
├── routing/
├── breadcrumbs/
├── search/
├── deep-links/
├── shortcuts/
├── accessibility/
├── mobile/
└── metadata.yml
```

---

# Standard Visual Artifacts

## Navigation Hierarchy

```text
Portal

↓

Domain

↓

Module

↓

Workspace

↓

Entity
```

---

## Investigation Navigation

```text
Dashboard

↓

Alert

↓

Incident

↓

Evidence

↓

Resolution
```

---

## Sidebar Structure

```text
Executive

↓

Operations

↓

Monitoring

↓

Security

↓

Administration
```

---

## Route Architecture

```text
Global

↓

Context

↓

Entity

↓

Detail
```

---

# Visual Source Files

```text
artifacts/
└── navigation-architecture/
    ├── navigation-hierarchy.drawio
    ├── routing-map.drawio
    ├── sidebar-navigation.drawio
    ├── investigation-flow.drawio
    ├── deep-link-model.drawio
    ├── search-navigation.drawio
    ├── mermaid/
    │   ├── navigation.mmd
    │   ├── routing.mmd
    │   ├── hierarchy.mmd
    │   ├── search.mmd
    │   └── investigations.mmd
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
| Component Catalog | Chapter 12 |
| Responsive Design | Chapter 15 |

---

# Acceptance Criteria

This chapter is complete when:

- Navigation hierarchy and routing model are fully documented.
- Global, contextual and entity navigation patterns are defined.
- Deep linking, search, shortcuts and multi-environment navigation are specified.
- Accessibility, performance targets, repository structure, visual artifacts and traceability are complete.

---

# Key Takeaways

- The EVOXA Operations Portal provides a layered navigation architecture that balances rapid access to operational information with consistent contextual awareness.
- Global navigation, contextual workspaces and cross-domain links enable operators to move efficiently between alerts, incidents, AI services and infrastructure.
- Deep linking, persistent state, semantic search and keyboard shortcuts reduce investigation time and improve productivity during operational events.
- A scalable routing model ensures that new operational capabilities can be incorporated without disrupting the overall navigation experience.

---

# Next Chapter

**Chapter 11 — Design Principles**

This chapter defines the visual language, interaction principles, operational design standards and user interface guidelines that establish a consistent and scalable design system for the EVOXA Operations Portal.
