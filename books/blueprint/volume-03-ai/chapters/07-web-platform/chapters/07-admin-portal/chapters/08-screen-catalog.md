---
document_id: BP-0003-V3-C07-07-08
chapter_id: CH-07-ADMIN-08
feature_pack: FP-ADMIN-0000
title: Screen Catalog
version: 1.0.0
status: Draft
owner: UX Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-05
---

# Chapter 08 — Screen Catalog

> *The Screen Catalog chapter defines the complete inventory of workspaces, screens, views, dialogs and administrative interfaces that compose the EVOXA Admin Portal.*

---

# Executive Summary

The Admin Portal consists of a comprehensive collection of administrative workspaces that provide centralized management of the EVOXA ecosystem.

Rather than isolated pages, screens are organized into domain-specific workspaces that support operational efficiency, governance and scalability.

Each workspace contains dashboards, data grids, detail views, configuration pages and administrative tools.

---

# Objectives

The Screen Catalog shall:

- Standardize administrative interfaces.
- Organize screens by domain.
- Support modular evolution.
- Reduce navigation complexity.
- Improve discoverability.
- Maintain UX consistency.
- Enable future expansion.

---

# Screen Organization

Screens are organized into:

- Dashboard
- Platform
- Tenants
- Organizations
- Users
- Roles & Permissions
- Security Center
- AI Center
- Billing
- Marketplace
- Integrations
- Monitoring
- Operations
- Audit Center
- Global Configuration

---

# Dashboard Workspace

## ADM-001 — Global Dashboard

Purpose

Platform overview.

Components

- KPI Cards
- Active Alerts
- AI Insights
- Platform Status
- Recent Activity
- Notifications

---

## ADM-002 — Personalized Dashboard

Purpose

Role-specific dashboard.

Widgets

- Favorites
- Tasks
- Metrics
- Recommendations

---

# Platform Workspace

## ADM-010 — Platform Overview

## ADM-011 — Global Settings

## ADM-012 — Environment Management

## ADM-013 — Regions

## ADM-014 — Maintenance Windows

## ADM-015 — Feature Flags

## ADM-016 — Platform Branding

---

# Tenant Workspace

## ADM-100 — Tenant List

## ADM-101 — Tenant Details

## ADM-102 — Create Tenant

## ADM-103 — Edit Tenant

## ADM-104 — Tenant Policies

## ADM-105 — Tenant Analytics

## ADM-106 — Tenant Activity

## ADM-107 — Tenant Subscriptions

---

# Organization Workspace

## ADM-200 — Organization List

## ADM-201 — Organization Details

## ADM-202 — Create Organization

## ADM-203 — Domains

## ADM-204 — Business Units

## ADM-205 — Licenses

## ADM-206 — Organization Settings

---

# User Workspace

## ADM-300 — User Directory

## ADM-301 — User Details

## ADM-302 — Create User

## ADM-303 — Invitations

## ADM-304 — Sessions

## ADM-305 — User Activity

## ADM-306 — User Devices

---

# Roles & Permissions

## ADM-400 — Roles

## ADM-401 — Role Details

## ADM-402 — Permissions

## ADM-403 — Policy Editor

## ADM-404 — Access Reviews

## ADM-405 — Permission Matrix

---

# Security Center

## ADM-500 — Security Dashboard

## ADM-501 — Identity Providers

## ADM-502 — Authentication Policies

## ADM-503 — MFA Configuration

## ADM-504 — Threat Center

## ADM-505 — Audit Logs

## ADM-506 — Compliance

## ADM-507 — Security Incidents

---

# AI Center

## ADM-600 — AI Dashboard

## ADM-601 — AI Models

## ADM-602 — AI Agents

## ADM-603 — Prompt Library

## ADM-604 — AI Analytics

## ADM-605 — AI Costs

## ADM-606 — AI Governance

## ADM-607 — AI Evaluations

---

# Billing Workspace

## ADM-700 — Billing Dashboard

## ADM-701 — Plans

## ADM-702 — Licenses

## ADM-703 — Usage

## ADM-704 — Invoices

## ADM-705 — Payments

---

# Marketplace Workspace

## ADM-800 — Marketplace

## ADM-801 — Applications

## ADM-802 — Plugins

## ADM-803 — Connectors

## ADM-804 — Publish Application

---

# Integration Workspace

## ADM-900 — Integrations

## ADM-901 — API Keys

## ADM-902 — OAuth Clients

## ADM-903 — Webhooks

## ADM-904 — API Usage

---

# Monitoring Workspace

## ADM-1000 — Platform Health

## ADM-1001 — Metrics

## ADM-1002 — Logs

## ADM-1003 — Traces

## ADM-1004 — Alerts

## ADM-1005 — Dashboards

---

# Operations Workspace

## ADM-1100 — Operations Center

## ADM-1101 — Jobs

## ADM-1102 — Queues

## ADM-1103 — Deployments

## ADM-1104 — Runbooks

## ADM-1105 — Maintenance

---

# Audit Workspace

## ADM-1200 — Audit Dashboard

## ADM-1201 — Audit Search

## ADM-1202 — Compliance Reports

## ADM-1203 — Export Center

---

# Global Configuration

## ADM-1300 — General Settings

## ADM-1301 — Localization

## ADM-1302 — Notification Templates

## ADM-1303 — Email Templates

## ADM-1304 — System Defaults

---

# Shared Screens

Available in every module:

- Search
- Filters
- Bulk Actions
- Import
- Export
- Activity Timeline
- Audit History
- Comments
- Attachments
- Notifications

---

# Modal Catalog

Administrative dialogs include:

- Confirmation
- Delete
- Approval
- Risk Warning
- Bulk Operation
- Import Wizard
- Export Wizard
- AI Assistant
- Permission Review

---

# Wizard Catalog

Multi-step administrative assistants:

- Create Tenant
- Create Organization
- Configure SSO
- Configure Identity Provider
- AI Model Deployment
- Platform Initialization
- Marketplace Publication

---

# Workspace Layout

```text
Header

↓

Sidebar

↓

Workspace

├── Dashboard
├── Data Grid
├── Detail Panel
├── Inspector
└── Activity Feed
```

---

# Navigation Relationships

```text
Dashboard

↓

Workspace

↓

List

↓

Detail

↓

Configuration
```

---

# Repository Structure

```text
screen-catalog/
├── dashboard/
├── platform/
├── tenants/
├── organizations/
├── users/
├── security/
├── ai/
├── billing/
├── marketplace/
├── integrations/
├── monitoring/
├── operations/
├── audit/
├── configuration/
└── metadata.yml
```

---

# Standard Visual Artifacts

## Workspace Catalog

```text
Dashboard

↓

Workspace

↓

Resources
```

---

## Screen Hierarchy

```text
Workspace

↓

List

↓

Detail

↓

Configuration
```

---

## Navigation Flow

```text
Dashboard

↓

Workspace

↓

Action
```

---

## Administrative Views

```text
Grid

↓

Detail

↓

History

↓

Audit
```

---

# Visual Source Files

```text
artifacts/
└── screen-catalog/
    ├── workspace-map.drawio
    ├── navigation-flow.drawio
    ├── screen-hierarchy.drawio
    ├── admin-layout.drawio
    ├── module-map.drawio
    ├── dialogs.drawio
    ├── mermaid/
    │   ├── catalog.mmd
    │   ├── navigation.mmd
    │   ├── hierarchy.mmd
    │   ├── workspaces.mmd
    │   └── dialogs.mmd
    └── exports/
        ├── screen-catalog.svg
        ├── screen-catalog.png
        └── screen-catalog.pdf
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

---

# Acceptance Criteria

This chapter is complete when:

- Administrative workspaces are cataloged.
- Screens are organized by functional domain.
- Shared dialogs, wizards and reusable views are identified.
- Navigation relationships between screens are documented.
- Visual artifacts and repository structure are complete.
- Traceability with adjacent chapters is established.

---

# Key Takeaways

- The Admin Portal is organized into domain-based workspaces that group related administrative capabilities into cohesive operational environments.
- Each workspace combines dashboards, lists, detail views, configuration pages and administrative tools to maximize productivity.
- Standardized layouts, dialogs and multi-step wizards ensure a consistent administrative experience across all platform domains.
- The Screen Catalog provides the structural foundation for navigation, layouts, workflows and reusable UI components throughout the EVOXA ecosystem.

---

# Next Chapter

**Chapter 09 — Layout Architecture**

This chapter defines the structural layout of the Admin Portal, including global page templates, workspace composition, responsive behavior, navigation containers, panels and layout patterns that support efficient enterprise administration.
