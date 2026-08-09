---
document_id: BP-0003-V3-C15-08
chapter_id: CH-15-MCP-08
feature_pack: FP-MCP-0001
title: Screen Catalog
version: 1.0.0
status: Draft
owner: Enterprise UX Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-07
---

# Chapter 08 — Screen Catalog

> *The Screen Catalog defines every user interface of the EVOXA MCP Platform. It establishes the structure, purpose, navigation hierarchy and interaction model of each screen required to administer, monitor, govern and consume the Model Context Protocol ecosystem.*

---

# Executive Summary

The MCP Platform contains multiple specialized workspaces designed for different personas.

Rather than exposing hundreds of disconnected pages, the platform groups functionality into coherent management consoles.

Each screen follows the EVOXA Design System and shares:

- Navigation
- Components
- AI Assistance
- Security
- Accessibility
- Observability

---

# UX Architecture

The platform is organized into six major workspaces.

```text
Home

↓

Management

↓

Development

↓

Operations

↓

Governance

↓

Marketplace
```

---

# Navigation Hierarchy

```text
Dashboard

├── MCP Servers
├── Tools
├── Resources
├── Prompts
├── Context
├── Models
├── Connectors
├── Marketplace
├── Analytics
├── Security
└── Administration
```

---

# Screen Categories

| Category | Screens |
|-----------|---------:|
| Dashboard | 8 |
| Registry | 12 |
| Development | 15 |
| Operations | 14 |
| Administration | 12 |
| Marketplace | 10 |
| Analytics | 8 |
| Security | 10 |
| AI | 9 |
| **Total** | **98+ Screens** |

---

# Dashboard Screens

---

## MCP Home Dashboard

Purpose

Platform overview.

Widgets

- Platform Health
- Active Servers
- Registered Tools
- AI Usage
- Recent Activity
- Alerts
- Marketplace Updates
- Recommendations

Primary Users

- Administrators
- Architects
- AI Engineers

---

## Executive Dashboard

Displays

- Adoption
- Business KPIs
- AI Consumption
- Platform Growth
- Enterprise Health

---

## Operations Dashboard

Displays

- CPU
- Memory
- Requests
- Error Rates
- Active Sessions
- Gateway Health

---

## AI Dashboard

Displays

- Models
- Tokens
- Cost
- AI Requests
- Tool Calls
- Context Usage

---

# MCP Server Management

---

## Server Catalog

Functions

- Search Servers
- Filter
- Health
- Versions
- Status

Actions

- Create
- Edit
- Publish
- Archive

---

## Server Details

Displays

- Metadata
- Endpoints
- Tools
- Resources
- Prompts
- Policies
- Logs
- Analytics

---

## Create Server Wizard

Steps

```text
General

↓

Metadata

↓

Capabilities

↓

Authentication

↓

Review

↓

Publish
```

---

## Server Monitoring

Displays

- Availability
- Latency
- Active Sessions
- Errors
- Requests
- Health

---

# Tool Registry

---

## Tool Catalog

Displays

- Tool Name
- Version
- Server
- Owner
- Category
- Status

Supports

- Search
- AI Search
- Filters
- Bulk Actions

---

## Tool Details

Displays

- Schema
- Inputs
- Outputs
- Permissions
- Documentation
- Usage
- Metrics

---

## Tool Editor

Features

- Schema Editor
- JSON Validation
- Parameter Builder
- AI Generator
- Documentation Generator

---

## Tool Testing Console

Functions

- Execute
- Debug
- View Response
- Trace
- Logs

---

# Resource Registry

---

## Resource Catalog

Displays

- APIs
- Documents
- Databases
- Images
- Reports

---

## Resource Details

Displays

- Metadata
- Permissions
- Version
- Usage
- Relationships

---

## Resource Explorer

Supports

- Preview
- Search
- Semantic Search
- AI Summary

---

# Prompt Registry

---

## Prompt Catalog

Displays

- Prompt
- Version
- Owner
- Category
- Rating

---

## Prompt Editor

Supports

- Variables
- Templates
- AI Suggestions
- Validation
- Preview

---

## Prompt Playground

Features

- Test Prompt
- Compare Models
- Streaming
- Token Analysis
- Cost Estimation

---

# Context Platform

---

## Context Explorer

Displays

- Session Context
- User Context
- Organization Context
- Memory
- Runtime Variables

---

## Context Inspector

Displays

- Injected Variables
- Sources
- Permissions
- Lifecycle

---

# AI Models

---

## Model Catalog

Displays

- Provider
- Version
- Latency
- Cost
- Availability

---

## Model Router

Functions

- Routing Rules
- Priorities
- Failover
- Policies

---

# Enterprise Connectors

---

## Connector Catalog

Displays

- Microsoft 365
- SAP
- Salesforce
- GitHub
- PostgreSQL
- ServiceNow
- Jira

---

## Connector Configuration

Functions

- Authentication
- Secrets
- Permissions
- Connectivity Test

---

# Marketplace

---

## Marketplace Home

Displays

- Featured Assets
- Categories
- Recommendations
- Recently Updated

---

## MCP Asset Details

Displays

- Documentation
- Reviews
- Versions
- Downloads
- Compatibility

---

## Publisher Dashboard

Displays

- Downloads
- Ratings
- Revenue
- Installations

---

# Analytics

---

## Usage Analytics

Displays

- Tool Usage
- AI Requests
- Active Users
- Sessions

---

## Cost Analytics

Displays

- Token Usage
- AI Costs
- Infrastructure Cost
- Trends

---

## Search Analytics

Displays

- Queries
- Popular Tools
- AI Discovery
- Success Rate

---

# Security

---

## Security Dashboard

Displays

- Login Attempts
- Threats
- Policy Violations
- Risk Score

---

## Permissions Console

Functions

- Roles
- Policies
- RBAC
- ABAC
- API Keys

---

## Audit Explorer

Displays

- User Actions
- AI Actions
- Tool Calls
- Configuration Changes

---

# Administration

---

## Platform Configuration

Sections

- General
- Branding
- Domains
- Integrations
- AI Providers

---

## Organization Management

Displays

- Organizations
- Tenants
- Workspaces
- Licenses

---

## User Administration

Functions

- Users
- Groups
- Roles
- Invitations

---

# Monitoring

---

## Live Monitoring

Displays

- Active Requests
- Tool Calls
- Sessions
- Queue
- AI Activity

---

## Event Explorer

Displays

- Event Stream
- Event Details
- Replay
- Filters

---

# Developer Workspace

---

## SDK Center

Downloads

- Python
- TypeScript
- Java
- .NET
- Go
- Rust

---

## API Explorer

Supports

- REST
- GraphQL
- MCP
- OpenAPI

---

## Documentation Portal

Includes

- Guides
- Tutorials
- API Docs
- SDK Docs
- Examples

---

# AI Workspace

---

## AI Assistant

Supports

- Natural Language
- Platform Search
- Documentation
- Recommendations

---

## AI Copilot

Capabilities

- Generate Tool
- Generate Prompt
- Generate Schema
- Explain Errors
- Optimize Configuration

---

# Universal Screens

Every module supports

- Search
- Filters
- Export
- Favorites
- Tags
- Comments
- Activity Timeline
- Notifications

---

# Screen Layout Standard

```text
Header

↓

Navigation

↓

Toolbar

↓

Content

↓

Inspector

↓

Footer
```

---

# Responsive Behavior

Desktop

- Full Workspace

Tablet

- Simplified Panels

Mobile

- Monitoring
- Notifications
- AI Assistant

---

# Screen Navigation Matrix

| Area | Primary Screen |
|-------|----------------|
| Servers | Server Catalog |
| Tools | Tool Registry |
| Resources | Resource Catalog |
| Prompts | Prompt Registry |
| Context | Context Explorer |
| Models | Model Catalog |
| Marketplace | Marketplace Home |
| Analytics | Usage Dashboard |
| Security | Security Dashboard |
| Administration | Platform Configuration |

---

# Repository Structure

```text
08-screen-catalog/
├── dashboard/
├── servers/
├── tools/
├── resources/
├── prompts/
├── context/
├── models/
├── connectors/
├── marketplace/
├── analytics/
├── security/
├── administration/
├── monitoring/
├── developer/
├── ai/
├── responsive/
├── diagrams/
└── metadata.yml
```

---

# Screen Inventory

| Area | Screens |
|------|---------:|
| Dashboards | 8 |
| Server Management | 12 |
| Tool Registry | 14 |
| Resource Registry | 10 |
| Prompt Registry | 9 |
| Context Platform | 8 |
| AI Models | 6 |
| Connectors | 8 |
| Marketplace | 10 |
| Analytics | 8 |
| Security | 10 |
| Administration | 12 |
| Monitoring | 7 |
| Developer Tools | 8 |
| AI Workspace | 6 |
| **Total Screens** | **126+** |

---

# Architecture Decision Records (ADRs)

| ADR | Decision |
|------|----------|
| ADR-0801 | Workspace-Based Navigation |
| ADR-0802 | Registry-Centric UX |
| ADR-0803 | AI-First Management Console |
| ADR-0804 | Unified Screen Framework |
| ADR-0805 | Responsive Enterprise Console |
| ADR-0806 | Dashboard-Centric Operations |
| ADR-0807 | Embedded AI Copilot |
| ADR-0808 | Shared Design System |

---

# Standard Visual Artifacts

## Platform Navigation

```text
Dashboard

↓

Workspace

↓

Screen

↓

Details
```

---

## Screen Hierarchy

```text
Workspace

↓

Catalog

↓

Details

↓

Editor
```

---

## Dashboard Layout

```text
Header

↓

Widgets

↓

Insights

↓

Actions
```

---

## Developer Workspace

```text
Servers

↓

Tools

↓

Resources

↓

Deploy
```

---

## Administration Console

```text
Configuration

↓

Users

↓

Security

↓

Monitoring
```

---

# Visual Source Files

```text
artifacts/
└── screen-catalog/
    ├── navigation-map.drawio
    ├── workspace-layout.drawio
    ├── dashboard-layout.drawio
    ├── developer-workspace.drawio
    ├── administration-console.drawio
    ├── ai-workspace.drawio
    ├── responsive-layout.drawio
    ├── mermaid/
    │   ├── navigation.mmd
    │   ├── workspaces.mmd
    │   ├── dashboard.mmd
    │   ├── registry.mmd
    │   ├── administration.mmd
    │   ├── ai.mmd
    │   └── responsive.mmd
    └── exports/
        ├── screen-catalog.svg
        ├── screen-catalog.png
        └── screen-catalog.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Chapter 06 — Information Architecture | Domain Organization |
| Chapter 07 — UX Goals | UX Principles |
| Chapter 09 — Layout Architecture | Screen Layouts |
| Chapter 10 — Navigation Architecture | Navigation Flows |
| Chapter 12 — Component Catalog | Shared Components |
| Chapter 13 — Widget Catalog | Dashboard Widgets |
| Chapter 16 — Workflow Architecture | Screen Workflows |
| Chapter 24 — Observability & Analytics | Operational Dashboards |

---

# Acceptance Criteria

This chapter is complete when:

- Every major screen and workspace of the MCP Platform is cataloged and categorized.
- Dashboards, registries, editors, monitoring consoles, administration interfaces and AI workspaces are fully documented.
- Standard layouts, responsive behavior, navigation hierarchy and reusable interaction patterns are defined.
- Repository structure, screen inventory, ADRs, visual artifacts and traceability are complete.
- The Screen Catalog provides a comprehensive blueprint for implementing the complete user interface of the EVOXA MCP Platform.

---

# Key Takeaways

- The EVOXA MCP Platform organizes more than 120 enterprise screens into role-based workspaces that support administration, development, governance, analytics and AI operations.
- Every interface follows a unified design system, ensuring consistency, discoverability and efficient navigation across the entire platform.
- AI copilots, semantic search and operational dashboards are embedded throughout the experience to maximize developer productivity and platform visibility.
- This screen catalog serves as the implementation reference for the visual and interaction architecture defined in the following chapters.

---

# Next Chapter

**Chapter 09 — Layout Architecture**

The next chapter defines the structural layout of every MCP Platform interface, including page composition, responsive grid system, workspace organization, panel architecture, docking behavior and enterprise workspace patterns.
