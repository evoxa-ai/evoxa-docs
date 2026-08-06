---
document_id: BP-0003-V3-C07-09-08
chapter_id: CH-09-OPS-08
feature_pack: FP-OPS-0000
title: Screen Catalog
version: 1.0.0
status: Draft
owner: Platform UX Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-05
---

# Chapter 08 — Screen Catalog

> *The Screen Catalog defines every workspace, dashboard and operational interface available in the EVOXA Operations Portal.*

---

# Executive Summary

The Operations Portal is composed of operational workspaces designed around real-time monitoring, incident response, AI operations and service reliability.

Unlike traditional administrative portals, every screen is built around operational decision-making, investigation and execution.

Each workspace follows a consistent layout, standardized navigation and reusable design system.

---

# Objectives

The Screen Catalog shall:

- Standardize all operational screens.
- Ensure navigation consistency.
- Define reusable layouts.
- Minimize operator learning curve.
- Support rapid investigations.
- Improve operational efficiency.
- Enable future expansion.

---

# Screen Classification

The platform organizes screens into:

```text
Dashboards

↓

Operations

↓

Investigations

↓

Management

↓

Reports

↓

Settings
```

---

# Executive Dashboards

---

## OPS-001 Executive Dashboard

Purpose

Provide executive operational visibility.

Widgets

- Platform Availability
- SLA Compliance
- Active Incidents
- AI Health
- Operational KPIs
- Cost Summary
- Capacity Overview

Primary Users

- COO
- Operations Director
- CTO

---

## OPS-002 Operations Dashboard

Purpose

Central operational overview.

Widgets

- Service Status
- Alerts
- Incidents
- Active Deployments
- AI Services
- Queue Health

---

## OPS-003 Platform Health Dashboard

Purpose

Display infrastructure health.

Widgets

- API Health
- Services
- Databases
- Clusters
- Network
- Storage

---

## OPS-004 AI Operations Dashboard

Purpose

Monitor AI platform.

Widgets

- Models
- Prompts
- Agents
- Workflows
- RAG
- MCP
- AI Health Score

---

# Incident Management

---

## OPS-101 Incident Center

Purpose

Manage operational incidents.

Functions

- Create
- Assign
- Escalate
- Resolve
- Timeline
- Comments
- Postmortem

---

## OPS-102 Incident Details

Contains

- Timeline
- Root Cause
- Metrics
- Logs
- Traces
- Related Alerts
- Related Deployments
- Runbooks

---

## OPS-103 Alert Center

Purpose

Manage operational alerts.

Capabilities

- Correlation
- Prioritization
- Acknowledgement
- Escalation
- Automation

---

# Monitoring

---

## OPS-201 Service Monitoring

Displays

- Health
- Availability
- Latency
- Errors
- Dependencies

---

## OPS-202 Infrastructure Monitoring

Displays

- Compute
- Storage
- Network
- Containers
- Kubernetes
- Databases

---

## OPS-203 Workflow Monitoring

Displays

- Active Executions
- Failed Nodes
- Retries
- Queue Status
- Execution History

---

## OPS-204 Deployment Monitoring

Displays

- Releases
- Deployments
- Rollbacks
- Environment Health

---

# AI Operations

---

## OPS-301 Model Operations

Displays

- Model Health
- Latency
- Token Usage
- Costs
- Availability

---

## OPS-302 Prompt Operations

Displays

- Prompt Versions
- Executions
- Failures
- Token Consumption
- Quality

---

## OPS-303 Agent Operations

Displays

- Sessions
- Memory
- Tool Usage
- Reasoning Timeline
- Health

---

## OPS-304 Knowledge Operations

Displays

- Knowledge Bases
- Documents
- Chunks
- Embeddings
- Retrieval

---

## OPS-305 RAG Operations

Displays

- Retrieval
- Precision
- Recall
- Citations
- Similarity

---

## OPS-306 MCP Operations

Displays

- Servers
- Connections
- Resources
- Tools
- Sessions

---

# Reliability

---

## OPS-401 SLA Dashboard

Displays

- Availability
- SLA
- SLO
- Error Budgets

---

## OPS-402 Capacity Center

Displays

- Resource Usage
- Forecast
- Scaling
- AI Capacity

---

## OPS-403 Reliability Center

Displays

- Error Trends
- Reliability Score
- Incidents
- Improvements

---

# Security Operations

---

## OPS-501 Security Dashboard

Displays

- Threats
- Authentication
- Authorization
- Compliance
- Audit

---

## OPS-502 Audit Explorer

Displays

- Audit Timeline
- User Activity
- Changes
- Security Events

---

## OPS-503 Guardrail Monitoring

Displays

- Violations
- Prompt Injection
- Sensitive Content
- Blocked Requests

---

# Cost Operations

---

## OPS-601 FinOps Dashboard

Displays

- Token Costs
- Provider Costs
- Cloud Costs
- Budget Usage
- Forecast

---

## OPS-602 Provider Analytics

Displays

- Provider Comparison
- Availability
- Cost
- Latency

---

# Analytics

---

## OPS-701 Operational Analytics

Displays

- KPIs
- Trends
- AI Metrics
- Forecasting

---

## OPS-702 Executive Reports

Displays

- Monthly Reports
- SLA Reports
- Availability
- Cost
- Productivity

---

# Operations

---

## OPS-801 Runbook Center

Displays

- Operational Procedures
- Recovery Guides
- Maintenance Guides

---

## OPS-802 Maintenance Center

Displays

- Maintenance Windows
- Planned Tasks
- Change Calendar

---

## OPS-803 Disaster Recovery

Displays

- Recovery Plans
- Backups
- Replication
- Recovery Status

---

# Administration

---

## OPS-901 Notification Center

Displays

- Email
- Push
- SMS
- Webhooks
- Routing Rules

---

## OPS-902 Personal Workspace

Displays

- Assigned Incidents
- Favorite Dashboards
- Saved Filters
- Recent Activity

---

## OPS-903 Preferences

Displays

- Theme
- Layout
- Time Zone
- Notifications
- Accessibility

---

# Common Screen Layout

Every screen follows:

```text
Global Header

↓

Left Navigation

↓

Page Header

↓

Toolbar

↓

Workspace

↓

Details Panel

↓

Footer
```

---

# Standard Screen Components

Every workspace contains:

- Breadcrumb
- Search
- Filters
- Actions
- Export
- Refresh
- Help
- AI Assistant

---

# Screen Navigation Rules

Every screen supports:

- Global Search
- Keyboard Navigation
- Breadcrumb Navigation
- Related Resources
- Deep Linking

---

# Responsive Behavior

Desktop

Full operational workspace.

Tablet

Simplified panels.

Mobile

Alerts, incidents and executive dashboards.

---

# Screen Naming Convention

```text
OPS-001

OPS-002

OPS-101

OPS-201

OPS-301

OPS-401

OPS-501

OPS-601

OPS-701

OPS-801

OPS-901
```

---

# Repository Structure

```text
screen-catalog/
├── dashboards/
├── operations/
├── incidents/
├── monitoring/
├── ai/
├── security/
├── reliability/
├── analytics/
├── administration/
└── metadata.yml
```

---

# Standard Visual Artifacts

## Screen Map

```text
Dashboard

↓

Operations

↓

Investigation

↓

Resolution
```

---

## Screen Hierarchy

```text
Portal

↓

Domain

↓

Screen

↓

Workspace
```

---

## Navigation Flow

```text
Dashboard

↓

Alert

↓

Incident

↓

Resolution
```

---

## Operational Workspace

```text
Header

↓

Toolbar

↓

Content

↓

Details
```

---

# Visual Source Files

```text
artifacts/
└── screen-catalog/
    ├── screen-map.drawio
    ├── navigation-flow.drawio
    ├── workspace-layout.drawio
    ├── dashboard-landscape.drawio
    ├── operations-center.drawio
    ├── screen-hierarchy.drawio
    ├── mermaid/
    │   ├── screens.mmd
    │   ├── navigation.mmd
    │   ├── hierarchy.mmd
    │   ├── dashboards.mmd
    │   └── workspaces.mmd
    └── exports/
        ├── screen-catalog.svg
        ├── screen-catalog.png
        └── screen-catalog.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| UX Goals | Chapter 07 |
| Layout Architecture | Chapter 09 |
| Navigation Architecture | Chapter 10 |
| Component Catalog | Chapter 12 |
| Widget Catalog | Chapter 13 |

---

# Acceptance Criteria

This chapter is complete when:

- All operational screens are cataloged.
- Dashboards, monitoring, AI operations, incident management and administration workspaces are defined.
- Screen layouts, navigation standards and responsive behavior are documented.
- Naming conventions, repository structure, visual artifacts and traceability are complete.

---

# Key Takeaways

- The Operations Portal provides a comprehensive catalog of operational workspaces covering monitoring, AI operations, incident management, reliability, security, analytics and administration.
- Every screen follows a standardized layout and interaction model, enabling operators to move efficiently between dashboards, investigations and corrective actions.
- Reusable components, consistent navigation and responsive behavior ensure a unified experience across desktop, tablet and mobile devices.
- The screen catalog establishes the foundation for the detailed layout, navigation and component specifications defined in the following chapters.

---

# Next Chapter

**Chapter 09 — Layout Architecture**

This chapter defines the structural layout of every workspace, including page templates, panel organization, responsive grids, workspace regions and layout patterns used throughout the EVOXA Operations Portal.
