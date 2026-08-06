---
document_id: BP-0003-V3-C07-07-03
chapter_id: CH-07-ADMIN-03
feature_pack: FP-ADMIN-0000
title: Product Scope
version: 1.0.0
status: Draft
owner: Product Strategy Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-05
---

# Chapter 03 — Product Scope

> *The Product Scope chapter defines the functional boundaries, capabilities, responsibilities and strategic positioning of the EVOXA Admin Portal within the overall platform ecosystem.*

---

# Executive Summary

The Admin Portal is the centralized administration platform responsible for governing, configuring and operating the EVOXA ecosystem.

Its scope focuses on platform administration rather than business operations. Customer-facing activities remain within their respective portals, while the Admin Portal provides shared services, governance and operational control.

---

# Objectives

The Product Scope shall:

- Define functional boundaries.
- Clarify platform responsibilities.
- Prevent capability overlap.
- Support modular evolution.
- Enable platform governance.
- Standardize administrative operations.
- Facilitate ecosystem scalability.

---

# Scope Definition

The Admin Portal provides centralized administration for:

- Platform Configuration
- Tenant Management
- Organization Management
- User Administration
- Identity & Access Management
- Security Governance
- AI Administration
- Marketplace Administration
- Subscription & Billing
- Integrations
- Feature Flags
- Platform Monitoring
- Operational Management
- Global Notifications
- Audit & Compliance

---

# In Scope

## Platform Administration

- Global platform settings
- Environment management
- Maintenance mode
- System configuration

---

## Tenant Management

- Create tenants
- Update tenant settings
- Suspend tenants
- Archive tenants
- Regional configuration

---

## Organization Management

- Organization lifecycle
- Business unit configuration
- Domain management
- Organizational policies

---

## Identity Management

- User provisioning
- Global administrators
- Authentication providers
- MFA configuration
- SSO configuration
- Session management

---

## Roles & Permissions

- Global roles
- Tenant roles
- Permission templates
- Policy assignment
- Role inheritance

---

## Security Center

- Security policies
- Threat monitoring
- Audit logs
- Access reviews
- Compliance controls

---

## AI Administration

- AI Providers
- AI Models
- AI Agents
- Prompt Library
- AI Governance
- AI Usage Analytics
- AI Cost Monitoring

---

## Marketplace

- Extensions
- Plugins
- Integrations
- Connector management
- Application publishing

---

## Billing & Licensing

- Plans
- Licenses
- Consumption
- Invoices
- Payments
- Quotas

---

## Platform Monitoring

- Infrastructure health
- API monitoring
- Event monitoring
- AI monitoring
- Workflow monitoring
- Database monitoring

---

## Operations

- Scheduled jobs
- Background services
- Maintenance windows
- Incident management
- Platform diagnostics

---

## Global Configuration

- Branding
- Regional settings
- Email templates
- Notification templates
- Platform defaults

---

# Out of Scope

The Admin Portal does **not** manage business operations performed by customer portals.

Examples include:

### User Portal

- Personal wellness
- Daily activities
- Health tracking

---

### Coach Portal

- Coaching sessions
- Client follow-up
- Training plans

---

### Medical Portal

- Medical consultations
- Clinical records
- Patient monitoring

---

### Nutritionist Portal

- Nutrition plans
- Meal tracking
- Dietary assessments

---

### Enterprise Portal

- Employee management
- Wellness campaigns
- Executive dashboards
- HR operations

---

# Functional Domains

```text
Platform

├── Administration
├── Security
├── AI
├── Marketplace
├── Billing
├── Monitoring
├── Operations
├── Configuration
└── Governance
```

---

# Product Boundaries

The Admin Portal owns:

- Platform governance
- Platform configuration
- Shared platform services
- Administrative operations

Customer portals own:

- Business workflows
- End-user interactions
- Operational data
- Domain-specific functionality

---

# External Dependencies

The Admin Portal integrates with:

- Identity Platform
- Security Platform
- AI Platform
- Event Platform
- Data Platform
- Billing Platform
- Notification Platform
- Operations Platform
- Developer Platform

---

# Shared Platform Services

Managed shared services include:

- Authentication
- Authorization
- Audit
- Notifications
- AI Gateway
- Event Bus
- API Gateway
- Feature Flags
- Secrets Management
- Configuration Service

---

# Platform Capabilities

| Capability | Responsibility |
|------------|----------------|
| Tenant Management | Full |
| Organization Management | Full |
| Identity | Full |
| Security | Full |
| Billing | Full |
| AI Governance | Full |
| Marketplace | Full |
| Platform Monitoring | Full |
| Operations | Full |
| Business Operations | External |
| Customer Workflows | External |

---

# Product Position

```text
Customer Portals

↓

Shared Services

↓

Admin Portal

↓

Cloud Infrastructure
```

---

# Repository Structure

```text
product-scope/
├── capabilities/
├── domains/
├── boundaries/
├── integrations/
├── dependencies/
├── governance/
├── platform-services/
└── metadata.yml
```

---

# Standard Visual Artifacts

## Functional Scope

```text
Administration

↓

Governance

↓

Operations

↓

Platform
```

---

## Scope Boundaries

```text
Admin Portal

↓

Shared Services

↓

Customer Portals
```

---

## Capability Map

```text
Platform

↓

Security

↓

AI

↓

Operations
```

---

## Ecosystem Position

```text
Products

↓

Platform

↓

Administration
```

---

# Visual Source Files

```text
artifacts/
└── product-scope/
    ├── scope-overview.drawio
    ├── capability-map.drawio
    ├── platform-boundaries.drawio
    ├── ecosystem.drawio
    ├── shared-services.drawio
    ├── governance.drawio
    ├── mermaid/
    │   ├── scope.mmd
    │   ├── boundaries.mmd
    │   ├── capabilities.mmd
    │   ├── platform.mmd
    │   └── ecosystem.mmd
    └── exports/
        ├── product-scope.svg
        ├── product-scope.png
        └── product-scope.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Overview | Chapter 01 |
| Business Overview | Chapter 02 |
| Information Architecture | Chapter 06 |
| Workflow Architecture | Chapter 16 |
| API Contracts | Chapter 19 |
| Security & Permissions | Chapter 23 |

---

# Acceptance Criteria

This chapter is complete when:

- Functional scope is clearly defined.
- In-scope and out-of-scope capabilities are documented.
- Platform responsibilities are distinguished from customer-facing portals.
- Shared services and external dependencies are identified.
- Capability maps and visual artifacts are complete.
- Traceability with related chapters is established.

---

# Key Takeaways

- The Admin Portal is responsible for platform-wide governance, configuration and operational management across the EVOXA ecosystem.
- Its scope includes shared administrative capabilities such as identity, security, AI governance, billing, monitoring and platform operations.
- Business-specific workflows remain within their respective portals, ensuring a clear separation of responsibilities.
- Well-defined product boundaries promote scalability, maintainability and consistent governance as the EVOXA platform evolves.

---

# Next Chapter

**Chapter 04 — Personas**

This chapter defines the primary administrative personas, their responsibilities, goals, permissions, operational contexts and interaction patterns within the EVOXA Admin Portal.
