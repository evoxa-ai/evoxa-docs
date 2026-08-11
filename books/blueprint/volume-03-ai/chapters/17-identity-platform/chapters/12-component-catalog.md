---
document_id: BP-0017-C12
chapter_id: CH-17-12
volume: Volume 17 — Identity Platform
title: Component Catalog
version: 1.0.0
status: Draft
owner: Enterprise UX Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-07
---

# 12 — Component Catalog

> *The Component Catalog defines every reusable user interface component used throughout the EVOXA Identity Platform. Components follow the EVOXA Enterprise Design System and provide consistency, accessibility, responsiveness and enterprise-grade usability across authentication, identity governance, security, AI identity and administration.*

---

# Executive Summary

Enterprise applications cannot be built with isolated UI elements.

Instead, they are assembled from reusable, standardized and accessible components.

The Identity Platform component library enables:

- Faster development
- Consistent UX
- Reduced maintenance
- Better accessibility
- Design governance
- Cross-platform consistency

Every screen in the Identity Platform is composed using these components.

---

# Design Vision

The platform follows one principle:

> **Build Once. Reuse Everywhere.**

Every reusable component becomes part of the Enterprise Design System.

---

# Component Categories

```text
Foundation

↓

Navigation

↓

Inputs

↓

Data Display

↓

Identity

↓

Security

↓

AI

↓

Administration

↓

Feedback

↓

Visualization
```

---

# Component Inventory

| Category | Components |
|-----------|-----------:|
| Foundation | 18 |
| Navigation | 20 |
| Forms | 34 |
| Tables | 18 |
| Identity | 30 |
| Security | 22 |
| AI | 16 |
| Administration | 18 |
| Analytics | 15 |
| Feedback | 20 |
| **Total Components** | **211** |

---

# Foundation Components

---

## IdentityAvatar

Displays

- User avatar
- AI avatar
- Service avatar
- Initials
- Status badge

Supports

- Presence
- Risk indicator
- Identity type

---

## StatusBadge

Represents

- Active
- Disabled
- Suspended
- Locked
- Pending
- Deleted

---

## RiskBadge

Displays

- Low
- Medium
- High
- Critical

---

## Tag

Supports

- Department
- Labels
- Categories
- Metadata

---

## Chip

Used for

- Roles
- Permissions
- Groups
- Claims

---

# Navigation Components

---

## EnterpriseSidebar

Provides

- Module navigation
- Favorites
- Search
- Collapse

---

## Breadcrumb

Example

```text
Dashboard

>

Users

>

John Smith
```

---

## GlobalSearch

Supports

- Identity search
- Applications
- Devices
- AI Agents
- Sessions

Autocomplete enabled.

---

## QuickActions

Examples

- Create User
- Reset Password
- Create Group
- Add Role

---

## TenantSwitcher

Allows switching between organizations.

---

# Form Components

---

## IdentityInput

Supports

- Username
- Email
- Employee ID
- External ID

Validation included.

---

## PasswordField

Features

- Strength meter
- Visibility toggle
- Breach detection
- Policy validation

---

## MFASelector

Supports

- TOTP
- SMS
- Email
- Passkey
- Security Key

---

## PolicySelector

Selects

- Authentication policies
- Conditional access
- Session policies

---

## RoleSelector

Supports

- Multi-select
- Search
- Hierarchy
- Suggestions

---

## PermissionSelector

Supports

- Tree view
- Search
- Categories
- Bulk assignment

---

# Identity Components

---

## UserCard

Displays

- Avatar
- Name
- Email
- Status
- Department
- Last login

---

## OrganizationCard

Displays

- Organization
- Users
- Domains
- Security score

---

## GroupCard

Displays

- Members
- Roles
- Policies

---

## DeviceCard

Displays

- Device name
- Trust status
- Compliance
- Last activity

---

## ServiceAccountCard

Displays

- Identity
- Secret status
- Expiration
- Permissions

---

## AIAgentCard

Displays

- Model
- Status
- Permissions
- Cost
- Risk

---

## MCPServerCard

Displays

- Server
- Version
- Authentication
- Health
- Tools

---

# Authentication Components

---

## LoginForm

Supports

- Username
- Password
- Passkey
- SSO

---

## PasswordlessButton

Starts

- WebAuthn
- Passkey
- FIDO2

---

## MFAChallenge

Displays

- QR Code
- OTP
- Push notification
- Security key

---

## SessionCard

Displays

- Browser
- Device
- Location
- Duration
- Risk

---

# Authorization Components

---

## RoleMatrix

Visual RBAC matrix.

---

## PermissionMatrix

Shows

Users

↓

Roles

↓

Permissions

↓

Resources

---

## ScopeSelector

Supports OAuth scopes.

---

## ClaimsViewer

Displays

- JWT Claims
- Custom claims
- Identity attributes

---

# Security Components

---

## SecurityScoreCard

Displays

- Overall score
- Recommendations
- Compliance

---

## ThreatCard

Displays

- Threat level
- Incident
- Recommendation

---

## RiskTimeline

Shows

- Risk changes
- Authentication events
- Device trust

---

## ConditionalAccessEditor

Configure

- Device
- Location
- User
- Risk
- Time

---

## PolicyBuilder

Visual editor

```text
Condition

↓

Operator

↓

Action
```

---

# AI Components

---

## AIRecommendation

Displays

- Suggested action
- Explanation
- Confidence
- Accept
- Reject

---

## AIConversation

Enterprise AI assistant.

---

## PromptViewer

Displays

- Prompt
- Variables
- History
- Version

---

## AgentPermissionViewer

Displays

- Allowed APIs
- Tools
- Resources
- Limits

---

# Analytics Components

---

## KPIWidget

Displays

- Metric
- Trend
- Comparison

---

## IdentityChart

Supports

- Line
- Bar
- Pie
- Area
- Heatmap

---

## AuthenticationTimeline

Displays

Authentication events over time.

---

## GeographicMap

Displays

- Login locations
- Risk
- Device origin

---

# Table Components

---

## EnterpriseTable

Supports

- Pagination
- Filters
- Export
- Saved Views
- Bulk actions

---

## AuditTable

Displays

- Timestamp
- Actor
- Resource
- Action
- Result

---

## SessionTable

Displays

- Sessions
- Devices
- Duration
- Risk

---

# Administration Components

---

## SettingsPanel

Supports

- Categories
- Validation
- Audit

---

## ConfigurationWizard

Used for

- Federation
- MFA
- OAuth
- SCIM

---

## SystemHealthCard

Displays

- APIs
- Database
- Federation
- AI Services

---

# Feedback Components

---

## AlertBanner

Levels

- Info
- Success
- Warning
- Error

---

## ToastNotification

Displays

- Success
- Warning
- Failure

---

## EmptyState

Example

```text
No Users Found

Create your first identity.
```

---

## ProgressIndicator

Supports

- Linear
- Circular
- Multi-step

---

# Modal Components

---

## ConfirmationDialog

Used for

- Delete
- Disable
- Revoke
- Reset

---

## SideDrawer

Displays

- Details
- Audit
- AI Insights

---

## WizardDialog

Supports

- Multi-step configuration
- Validation
- Review

---

# Identity Lifecycle Components

Supports

```text
Provision

↓

Activate

↓

Authenticate

↓

Authorize

↓

Suspend

↓

Archive
```

---

# Accessibility Requirements

Every component supports

- Keyboard navigation
- Screen readers
- Focus states
- WCAG 2.2 AA
- High contrast
- Reduced motion

---

# Component States

Every component includes

- Default
- Hover
- Focus
- Active
- Disabled
- Loading
- Error
- Success

---

# Responsive Behavior

Desktop

```text
Full Layout
```

Tablet

```text
Adaptive Layout
```

Mobile

```text
Compact Layout
```

---

# Component Naming Convention

```text
IdentityAvatar

OrganizationCard

RoleMatrix

PermissionMatrix

RiskBadge

AIAgentCard

EnterpriseTable
```

PascalCase is mandatory.

---

# Component Composition

```text
Page

↓

Section

↓

Panel

↓

Component

↓

Subcomponent
```

---

# Component Dependencies

```text
Foundation

↓

Forms

↓

Identity

↓

Security

↓

Analytics
```

---

# Repository Structure

```text
12-component-catalog/

├── foundation/
├── navigation/
├── forms/
├── identity/
├── authentication/
├── authorization/
├── ai/
├── security/
├── analytics/
├── administration/
├── feedback/
├── tables/
├── dialogs/
├── lifecycle/
├── accessibility/
├── responsive/
├── naming-conventions.md
├── component-matrix.md
├── component-lifecycle.md
├── glossary.md
├── diagrams/
│   ├── component-library.drawio
│   ├── dependency-tree.drawio
│   ├── ui-composition.drawio
│   ├── authentication-components.drawio
│   ├── security-components.drawio
│   ├── ai-components.drawio
│   ├── responsive-components.drawio
│   └── accessibility-components.drawio
└── metadata.yml
```

---

# Component Asset Inventory

| Area | Assets |
|------|--------:|
| Foundation Components | 18 |
| Navigation Components | 20 |
| Form Controls | 34 |
| Identity Components | 30 |
| Security Components | 22 |
| AI Components | 16 |
| Analytics Components | 15 |
| Administration Components | 18 |
| Feedback Components | 20 |
| Table Components | 18 |
| Dialog Components | 12 |
| **Total Components** | **223** |

---

# Architecture Principles

The component architecture follows

- Atomic Design
- Design System First
- Accessibility First
- Component Reuse
- Stateless by Default
- API Driven
- Secure by Design
- Responsive by Default
- Theme Aware
- AI Ready

---

# Traceability

| Related Volume | Relationship |
|---------------|--------------|
| Enterprise Design System | Base Components |
| Enterprise Platform | Shared UI Library |
| Security Platform | Security Components |
| AI Platform | AI Components |
| Administration Platform | Admin Components |
| Developer Platform | Developer UI |

---

# Acceptance Criteria

This chapter is complete when:

- All reusable UI components are cataloged and categorized.
- Identity, authentication, authorization, AI, analytics and administration components are fully specified.
- Accessibility, responsiveness, naming conventions and component lifecycle standards are documented.
- Repository organization, inventories, architectural principles and traceability are complete.
- The Component Catalog serves as the authoritative reference for building every interface within the EVOXA Identity Platform.

---

# Key Takeaways

- The EVOXA Identity Platform is built from a comprehensive library of more than 220 reusable enterprise components.
- Components are organized by functional domains, ensuring consistency across authentication, identity governance, AI management, security operations and analytics.
- Accessibility, responsiveness, design system compliance and security are embedded into every component from the outset.
- This catalog provides the reusable UI foundation required to build a scalable, maintainable and enterprise-grade Identity Platform.

---

# Next Section

**13 — Widget Catalog**

The next chapter defines the complete catalog of reusable dashboard widgets, KPI cards, security visualizations, identity analytics, AI insights and operational monitoring widgets available throughout the EVOXA Identity Platform.
