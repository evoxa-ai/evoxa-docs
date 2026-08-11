---
document_id: BP-0003-V3-C16-08
chapter_id: CH-16-08
feature_pack: FP-INTEGRATION-0001
title: Screen Catalog
version: 1.0.0
status: Draft
owner: Enterprise UX Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-07
---

# 08 — Screen Catalog

> *The Screen Catalog defines every user interface of the EVOXA Integration Platform. It describes the purpose, functionality, navigation, permissions and relationships between dashboards, configuration screens, operational consoles and administration modules that collectively deliver the enterprise integration experience.*

---

# Executive Summary

The Integration Platform contains a large number of enterprise interfaces designed for different personas including Integration Architects, API Developers, Platform Administrators, AI Engineers and Business Analysts.

Every screen follows the EVOXA Design System and shares a consistent navigation model, interaction pattern and responsive layout.

The catalog serves as the authoritative reference for UX, development and quality assurance teams.

---

# UX Principles

Every screen follows:

- Minimal clicks
- Progressive disclosure
- AI assistance
- Consistent navigation
- Enterprise responsiveness
- Accessibility by default
- Context awareness
- Role-based visibility

---

# Screen Architecture

```text
Home

↓

Workspace

↓

Module

↓

Detail

↓

Configuration
```

---

# Screen Categories

| Category | Screens |
|-----------|---------:|
| Dashboards | 12 |
| API Management | 16 |
| Connector Management | 18 |
| Workflow Designer | 14 |
| Event Platform | 10 |
| AI Integration | 12 |
| MCP Integration | 10 |
| Marketplace | 8 |
| Monitoring | 16 |
| Governance | 12 |
| Administration | 20 |
| Settings | 12 |
| **Total Screens** | **160+** |

---

# Home Dashboard

**Purpose**

Enterprise operational overview.

### Features

- Platform health
- Active APIs
- Connector status
- Running workflows
- Event throughput
- AI activity
- Recent deployments
- Notifications

---

# API Management

## API Catalog

Displays

- API inventory
- Version
- Status
- Consumers
- Providers
- SLA
- Owner

Actions

- Create
- Import
- Publish
- Version
- Archive

---

## API Detail

Displays

- OpenAPI
- Endpoints
- Security
- Analytics
- Consumers
- Changelog

---

## API Designer

Supports

- OpenAPI editor
- AI generation
- Live validation
- Mock server
- Documentation preview

---

## API Analytics

Displays

- Requests
- Latency
- Errors
- Consumers
- Rate limits
- Geographic usage

---

# Connector Management

## Connector Catalog

Displays

- Installed connectors
- Marketplace connectors
- Versions
- Health
- Categories

---

## Connector Builder

Features

- Template selection
- Authentication
- Mapping
- Validation
- Testing

---

## Connector Detail

Displays

- Configuration
- Endpoints
- Credentials
- Logs
- Metrics
- Dependencies

---

## Connector Marketplace

Displays

- Certified connectors
- Ratings
- Publisher
- Downloads
- Compatibility

---

# Workflow Management

## Workflow Catalog

Displays

- Workflow library
- Categories
- Status
- Owner
- Version

---

## Workflow Designer

Supports

- Drag & Drop
- BPMN
- AI-generated flows
- Validation
- Simulation

---

## Workflow Execution

Displays

- Active instances
- History
- Errors
- Duration
- Inputs
- Outputs

---

## Workflow Templates

Provides

- Business templates
- AI templates
- Industry templates
- Integration templates

---

# Event Platform

## Event Catalog

Displays

- Topics
- Producers
- Consumers
- Throughput
- Schema

---

## Event Detail

Displays

- Payload
- Subscribers
- History
- DLQ
- Metrics

---

## Event Monitoring

Displays

- Event flow
- Delays
- Queue length
- Failures

---

# AI Integration

## AI Dashboard

Displays

- Models
- Providers
- Token usage
- Cost
- Performance

---

## Prompt Studio

Supports

- Prompt editor
- Testing
- Versioning
- AI recommendations

---

## Agent Console

Displays

- AI agents
- Memory
- Tools
- Tasks
- Conversations

---

# MCP Platform

## MCP Registry

Displays

- Servers
- Status
- Tools
- Resources
- Health

---

## MCP Server Detail

Displays

- Capabilities
- Tools
- Prompts
- Security
- Logs

---

## Tool Explorer

Displays

- Available tools
- Parameters
- Documentation
- Examples

---

# Marketplace

## Marketplace Home

Displays

- Featured connectors
- Workflows
- AI templates
- APIs

---

## Asset Detail

Displays

- Description
- Publisher
- Reviews
- Version
- Installation

---

# Monitoring

## Operations Dashboard

Displays

- Platform health
- APIs
- Connectors
- Events
- AI
- Infrastructure

---

## Alert Center

Displays

- Critical alerts
- Warnings
- Notifications
- Incidents

---

## Logs

Displays

- API logs
- Connector logs
- Workflow logs
- Security logs

---

## Metrics

Displays

- Latency
- Availability
- Throughput
- Error rate
- AI metrics

---

# Governance

## Governance Dashboard

Displays

- Policies
- Standards
- Reviews
- Compliance

---

## API Governance

Displays

- Version lifecycle
- Policy compliance
- Reviews

---

## Connector Governance

Displays

- Certification
- Ownership
- Lifecycle

---

# Administration

## Organization Management

Supports

- Organizations
- Tenants
- Business Units

---

## User Management

Supports

- Users
- Roles
- Permissions
- Groups

---

## Role Management

Supports

- RBAC
- Policies
- Privileges

---

## Audit Center

Displays

- Activity logs
- Changes
- Security events

---

## Configuration Center

Supports

- Platform configuration
- Integrations
- Security
- Notifications

---

# Settings

Includes

- Profile
- Preferences
- Themes
- Language
- Notifications
- API Keys
- Integrations

---

# Universal Components

Available on every screen

- Navigation menu
- Global search
- Notifications
- User profile
- Help
- AI Copilot
- Breadcrumbs

---

# Screen States

Each screen supports

- Empty
- Loading
- Error
- Read-only
- Editing
- Success

---

# Navigation Model

```text
Dashboard

↓

Module

↓

Catalog

↓

Detail

↓

Configuration
```

---

# AI Assistance

Every major screen includes AI capabilities

- Recommendations
- Contextual help
- Documentation
- Error explanation
- Workflow suggestions
- Connector recommendations

---

# Responsive Behavior

| Device | Support |
|----------|----------|
| Desktop | Full |
| Laptop | Full |
| Tablet | Optimized |
| Mobile | Monitoring & Approvals |

---

# Permission Matrix

| Screen | Admin | Architect | Engineer | Viewer |
|----------|:----:|:---------:|:--------:|:------:|
| Dashboard | ✓ | ✓ | ✓ | ✓ |
| API Designer | ✓ | ✓ | ✓ | No |
| Workflow Designer | ✓ | ✓ | ✓ | No |
| Connector Builder | ✓ | ✓ | ✓ | No |
| Governance | ✓ | ✓ | No | Read |
| Administration | ✓ | No | No | No |
| Monitoring | ✓ | ✓ | ✓ | ✓ |

---

# Screen Inventory

| Module | Screens |
|----------|---------:|
| Dashboard | 12 |
| APIs | 16 |
| Connectors | 18 |
| Workflows | 14 |
| Events | 10 |
| AI | 12 |
| MCP | 10 |
| Marketplace | 8 |
| Monitoring | 16 |
| Governance | 12 |
| Administration | 20 |
| Settings | 12 |
| **Total** | **160+** |

---

# Repository Structure

```text
08-screen-catalog/
├── dashboard/
├── api-management/
├── connectors/
├── workflows/
├── events/
├── ai/
├── mcp/
├── marketplace/
├── monitoring/
├── governance/
├── administration/
├── settings/
├── shared/
├── mockups/
├── wireframes/
├── diagrams/
└── metadata.yml
```

---

# Architecture Decision Records (ADRs)

| ADR | Decision |
|------|----------|
| ADR-160801 | Unified Enterprise Navigation |
| ADR-160802 | Catalog-Based User Experience |
| ADR-160803 | AI Copilot Embedded in Every Module |
| ADR-160804 | Role-Based Screen Visibility |
| ADR-160805 | Dashboard-Centric Operations |
| ADR-160806 | Shared Screen Components |
| ADR-160807 | Responsive Enterprise UI |
| ADR-160808 | Screen Standardization Framework |

---

# Standard Visual Artifacts

## Navigation Tree

```text
Dashboard

↓

Modules

↓

Catalog

↓

Detail
```

---

## API Management

```text
Catalog

↓

API

↓

Version

↓

Analytics
```

---

## Workflow Navigation

```text
Catalog

↓

Designer

↓

Execution

↓

History
```

---

## Operations Dashboard

```text
Health

↓

Alerts

↓

Metrics

↓

Logs
```

---

## Marketplace Navigation

```text
Marketplace

↓

Category

↓

Asset

↓

Installation
```

---

# Visual Source Files

```text
artifacts/
└── screen-catalog/
    ├── navigation-map.drawio
    ├── dashboard-layout.drawio
    ├── api-management.drawio
    ├── workflow-designer.drawio
    ├── connector-builder.drawio
    ├── monitoring-dashboard.drawio
    ├── marketplace.drawio
    ├── mermaid/
    │   ├── navigation.mmd
    │   ├── api-flow.mmd
    │   ├── workflow.mmd
    │   ├── monitoring.mmd
    │   ├── governance.mmd
    │   ├── marketplace.mmd
    │   └── administration.mmd
    └── exports/
        ├── screen-catalog.svg
        ├── screen-catalog.png
        └── screen-catalog.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Chapter 04 — Personas | Role-Based Screens |
| Chapter 05 — User Journeys | Navigation Flows |
| Chapter 06 — Information Architecture | Information Hierarchy |
| Chapter 07 — UX Goals | UX Standards |
| Chapter 09 — Layout Architecture | Screen Layouts |
| Chapter 10 — Navigation Architecture | Navigation Model |
| Chapter 12 — Component Catalog | UI Components |
| EVOXA Design System | Shared Design Standards |

---

# Acceptance Criteria

This chapter is complete when:

- Every major user interface of the Integration Platform is cataloged and described.
- Dashboards, API management, connector management, workflow, AI, MCP, monitoring, governance and administration screens are documented.
- Navigation patterns, permission models, responsive behaviors and AI-assisted interactions are specified.
- Repository structure, inventories, ADRs, visual artifacts and traceability are complete.
- The Screen Catalog provides the definitive reference for designing, implementing and validating the user interface of the EVOXA Integration Platform.

---

# Key Takeaways

- The Screen Catalog defines a comprehensive enterprise user interface covering the complete lifecycle of integrations, APIs, workflows, AI services and operational management.
- Every screen follows a unified navigation model, role-based access control and consistent design language aligned with the EVOXA Design System.
- AI-assisted interactions, contextual guidance and responsive layouts improve usability while reducing operational complexity.
- This catalog serves as the foundation for UX design, frontend implementation, testing and long-term platform evolution.

---

# Next Section

**09 — Layout Architecture**

The next section defines the structural composition of every screen, including workspace layouts, panel organization, responsive behavior, dashboard composition and reusable layout templates for the EVOXA Integration Platform.
