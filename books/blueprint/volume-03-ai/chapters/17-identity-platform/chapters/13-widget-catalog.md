---
document_id: BP-0017-C13
chapter_id: CH-17-13
volume: Volume 17 — Identity Platform
title: Widget Catalog
version: 1.0.0
status: Draft
owner: Enterprise UX Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-07
---

# 13 — Widget Catalog

> *The Widget Catalog defines every reusable dashboard widget available throughout the EVOXA Identity Platform. Widgets provide real-time visibility into identity operations, authentication, authorization, security posture, governance, AI identities, federation and platform health.*

---

# Executive Summary

Widgets are reusable visualization modules that transform raw identity data into actionable operational insights.

Unlike UI components, which provide interaction capabilities, widgets aggregate data, metrics and events into meaningful dashboards for administrators, security analysts, auditors, developers and executives.

Every widget follows the EVOXA Enterprise Dashboard Framework.

---

# Widget Vision

The platform follows one guiding principle:

> **Every Widget Answers a Business or Security Question.**

Widgets exist to provide immediate visibility and support informed decision-making.

---

# Widget Objectives

The Widget Catalog provides

- Real-time monitoring
- Identity visibility
- Security awareness
- Governance insights
- AI recommendations
- Operational analytics
- Compliance indicators
- Executive dashboards

---

# Widget Categories

```text
Identity

↓

Authentication

↓

Authorization

↓

Security

↓

Governance

↓

AI

↓

Operations

↓

Analytics

↓

Compliance

↓

Administration
```

---

# Widget Inventory

| Category | Widgets |
|-----------|---------:|
| Identity | 22 |
| Authentication | 18 |
| Authorization | 16 |
| Security | 26 |
| Governance | 18 |
| AI | 18 |
| Analytics | 20 |
| Compliance | 12 |
| Operations | 14 |
| Administration | 10 |
| **Total Widgets** | **174** |

---

# Identity Widgets

---

## WDG-001 — Total Identities

Displays

- Total users
- Customers
- AI agents
- Service accounts
- Devices

Visualization

- KPI Card

Refresh

- Real-time

---

## WDG-002 — Identity Growth

Displays

- Daily
- Weekly
- Monthly growth

Visualization

- Line Chart

---

## WDG-003 — User Distribution

Shows

- Departments
- Regions
- Organizations

Visualization

- Donut Chart

---

## WDG-004 — Active Identities

Displays

- Online users
- Active sessions
- Active AI agents

Visualization

- KPI + Trend

---

# Authentication Widgets

---

## WDG-010 — Authentication Success Rate

Displays

- Successful logins
- Failed logins
- Success percentage

Visualization

- KPI

---

## WDG-011 — Login Timeline

Shows

- Login volume
- Hourly activity
- Daily trends

Visualization

- Timeline

---

## WDG-012 — MFA Adoption

Displays

- Enabled users
- Pending enrollment
- Compliance percentage

Visualization

- Gauge

---

## WDG-013 — Passwordless Adoption

Shows

- Passkeys
- WebAuthn
- FIDO2 usage

Visualization

- KPI + Trend

---

## WDG-014 — Authentication Methods

Displays

- Password
- Passkey
- SAML
- OIDC
- OAuth

Visualization

- Pie Chart

---

# Authorization Widgets

---

## WDG-020 — Role Distribution

Displays

- Roles
- Assignments
- Growth

Visualization

- Bar Chart

---

## WDG-021 — Permission Utilization

Shows

- Active permissions
- Unused permissions

Visualization

- Heatmap

---

## WDG-022 — Privileged Accounts

Displays

- Administrators
- Service accounts
- AI identities

Visualization

- KPI Card

---

# Security Widgets

---

## WDG-030 — Security Score

Displays

- Global score
- Trend
- Recommendations

Visualization

- Gauge

---

## WDG-031 — High Risk Logins

Shows

- Suspicious activity
- Impossible travel
- Credential stuffing

Visualization

- Alert Panel

---

## WDG-032 — Threat Timeline

Displays

- Identity attacks
- Risk events
- Security alerts

Visualization

- Timeline

---

## WDG-033 — Locked Accounts

Displays

- Locked users
- Reasons
- Trends

Visualization

- KPI

---

## WDG-034 — Trusted Devices

Shows

- Managed devices
- Compliance
- Trust level

Visualization

- Donut Chart

---

## WDG-035 — Session Risk

Displays

- Active risky sessions
- Severity
- Location

Visualization

- Geographic Map

---

# Governance Widgets

---

## WDG-040 — Access Reviews

Displays

- Active reviews
- Completed
- Pending

Visualization

- Progress Card

---

## WDG-041 — Segregation of Duties

Displays

- Violations
- Severity
- Resolution status

Visualization

- Table

---

## WDG-042 — Identity Lifecycle

Shows

- Provisioned
- Active
- Suspended
- Archived

Visualization

- Funnel

---

# AI Widgets

---

## WDG-050 — AI Agent Activity

Displays

- Running agents
- Executions
- Success rate

Visualization

- KPI + Chart

---

## WDG-051 — AI Recommendations

Shows

- Suggested actions
- Confidence
- Acceptance rate

Visualization

- AI Insight Card

---

## WDG-052 — AI Resource Usage

Displays

- Token consumption
- Cost
- Models
- API usage

Visualization

- Area Chart

---

## WDG-053 — MCP Server Health

Displays

- Online servers
- Response time
- Errors

Visualization

- Status Grid

---

# Analytics Widgets

---

## WDG-060 — Identity Analytics

Displays

- User trends
- Growth
- Activity

Visualization

- Dashboard

---

## WDG-061 — Geographic Authentication

Displays

- Login locations
- Country distribution

Visualization

- Interactive Map

---

## WDG-062 — Peak Login Hours

Displays

- Hourly authentication

Visualization

- Heatmap

---

## WDG-063 — Authentication Latency

Displays

- Login response time

Visualization

- Line Chart

---

# Compliance Widgets

---

## WDG-070 — Compliance Score

Displays

- GDPR
- ISO 27001
- SOC2
- Internal controls

Visualization

- Score Card

---

## WDG-071 — Audit Completeness

Shows

- Logged events
- Missing events

Visualization

- KPI

---

## WDG-072 — Policy Compliance

Displays

- Compliant users
- Exceptions

Visualization

- Donut Chart

---

# Operations Widgets

---

## WDG-080 — Platform Health

Displays

- API availability
- Identity service
- Database
- Federation

Visualization

- Health Dashboard

---

## WDG-081 — Service Status

Displays

- Online services
- Maintenance
- Failures

Visualization

- Status Matrix

---

## WDG-082 — Queue Monitoring

Displays

- Event queues
- Processing rate
- Delays

Visualization

- Timeline

---

# Administration Widgets

---

## WDG-090 — Tenant Overview

Displays

- Organizations
- Licenses
- Domains
- Active users

Visualization

- KPI Grid

---

## WDG-091 — Subscription Usage

Displays

- Licenses used
- Remaining
- Expiration

Visualization

- Progress Card

---

# Universal Widget Features

Every widget supports

- Refresh
- Full Screen
- Export
- Drill-down
- Filters
- Personalization
- Resize
- Move
- Pin
- Share

---

# Widget Sizes

| Size | Grid |
|------|------|
| Small | 1×1 |
| Medium | 2×1 |
| Large | 2×2 |
| Extra Large | 4×2 |

---

# Refresh Policies

| Type | Refresh |
|------|----------|
| Security | Real-time |
| Authentication | 5 sec |
| Sessions | 5 sec |
| AI | 15 sec |
| Analytics | 1 min |
| Compliance | 5 min |

---

# Dashboard Templates

Supported dashboards

- Executive Dashboard
- Security Operations
- Identity Administration
- Governance
- AI Operations
- Compliance
- Federation
- Platform Health
- Developer Portal
- Personal Workspace

---

# Widget Lifecycle

```text
Create

↓

Configure

↓

Display

↓

Refresh

↓

Interact

↓

Archive
```

---

# Widget Permissions

Widgets inherit RBAC permissions.

Access depends on

- Role
- Organization
- Tenant
- Security Policy
- Feature Flags

---

# Widget Personalization

Users may customize

- Layout
- Position
- Theme
- Refresh interval
- Default filters
- Widget size
- Saved dashboards

---

# Widget States

Every widget supports

- Loading
- Ready
- Empty
- Error
- Refreshing
- Offline

---

# Accessibility

Widgets comply with

- WCAG 2.2 AA
- Keyboard navigation
- Screen readers
- High contrast
- Reduced motion

---

# Performance Targets

| Metric | Target |
|---------|--------|
| Widget Load | <300 ms |
| Dashboard Load | <2 sec |
| Refresh Time | <500 ms |
| Interaction Delay | <100 ms |

---

# Repository Structure

```text
13-widget-catalog/

├── identity/
├── authentication/
├── authorization/
├── security/
├── governance/
├── ai/
├── analytics/
├── compliance/
├── operations/
├── administration/
├── dashboard-templates.md
├── personalization.md
├── widget-lifecycle.md
├── widget-permissions.md
├── widget-performance.md
├── glossary.md
├── diagrams/
│   ├── dashboard-layout.drawio
│   ├── widget-library.drawio
│   ├── widget-composition.drawio
│   ├── security-dashboard.drawio
│   ├── identity-dashboard.drawio
│   ├── ai-dashboard.drawio
│   ├── governance-dashboard.drawio
│   ├── compliance-dashboard.drawio
│   └── widget-lifecycle.drawio
└── metadata.yml
```

---

# Widget Asset Inventory

| Area | Assets |
|------|--------:|
| Identity Widgets | 22 |
| Authentication Widgets | 18 |
| Authorization Widgets | 16 |
| Security Widgets | 26 |
| Governance Widgets | 18 |
| AI Widgets | 18 |
| Analytics Widgets | 20 |
| Compliance Widgets | 12 |
| Operations Widgets | 14 |
| Administration Widgets | 10 |
| Dashboard Templates | 10 |
| **Total Widget Assets** | **184** |

---

# Architecture Principles

The widget architecture follows

- Dashboard First
- Real-Time by Default
- Modular Composition
- Configurable Widgets
- Secure Data Access
- Accessibility First
- Responsive Design
- AI-Augmented Insights
- Event-Driven Updates
- Enterprise Scalability

---

# Traceability

| Related Volume | Relationship |
|---------------|--------------|
| Enterprise Platform | Executive Dashboards |
| Security Platform | Security Widgets |
| AI Platform | AI Monitoring Widgets |
| Integration Platform | API Metrics |
| Administration Platform | Tenant Dashboards |
| Enterprise Design System | Widget Framework |

---

# Acceptance Criteria

This chapter is complete when:

- All reusable dashboard widgets are cataloged and categorized by business domain.
- Widget capabilities, visualization types, refresh policies, personalization options and lifecycle states are fully documented.
- Dashboard templates, accessibility requirements, performance targets and security inheritance are defined.
- Repository organization, inventories, architectural principles and traceability are complete.
- The Widget Catalog serves as the authoritative reference for implementing dashboards across the EVOXA Identity Platform.

---

# Key Takeaways

- The EVOXA Identity Platform provides a standardized library of more than 180 reusable widgets for identity management, security operations, governance, AI monitoring and compliance.
- Widgets deliver real-time operational intelligence through configurable, responsive and accessible visualizations.
- Personalization, RBAC-aware visibility and event-driven updates ensure every user receives relevant information without compromising security.
- This widget catalog establishes the visualization foundation for all dashboards within the Identity Platform.

---

# Next Section

**14 — Design Tokens**

The next chapter defines the complete design token system, including colors, typography, spacing, icons, elevation, motion, responsive breakpoints and semantic tokens used throughout the EVOXA Identity Platform.
