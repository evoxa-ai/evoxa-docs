---
document_id: BP-0017-C08
chapter_id: CH-17-08
volume: Volume 17 — Identity Platform
title: Screen Catalog
version: 1.0.0
status: Draft
owner: Enterprise UX Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-07
---

# 08 — Screen Catalog

> *The Screen Catalog defines every user interface available within the EVOXA Identity Platform. It establishes the complete navigation structure, screen hierarchy, functional responsibilities and relationships between authentication, administration, governance, security, AI identity and identity lifecycle management.*

---

# Executive Summary

The Identity Platform contains a complete suite of enterprise interfaces supporting every identity operation.

Unlike traditional IAM solutions focused only on login screens, EVOXA provides dedicated workspaces for:

- Identity Administration
- Authentication
- Authorization
- Governance
- Federation
- AI Identity
- Machine Identity
- Security Operations
- Compliance
- Analytics

Every screen follows the EVOXA Enterprise Design System.

---

# UX Vision

The platform follows one principle:

> **Every Screen Has One Clear Purpose and One Primary User.**

Interfaces minimize cognitive load while exposing advanced functionality when necessary.

---

# Screen Categories

```text
Authentication

↓

Administration

↓

Identity Management

↓

Security

↓

Governance

↓

AI Identity

↓

Machine Identity

↓

Analytics

↓

Settings
```

---

# Navigation Structure

```text
Dashboard

├── Organizations
├── Users
├── Groups
├── Roles
├── Permissions
├── Applications
├── Devices
├── AI Agents
├── Service Accounts
├── Federation
├── Sessions
├── Audit
├── Analytics
├── Security
└── Settings
```

---

# Authentication Screens

---

## ID-001 — Login

Purpose

Authenticate enterprise users.

Functions

- Username
- Password
- Passkey
- SSO
- Remember Device
- Forgot Password

Primary Personas

- Employee
- Administrator
- Customer

---

## ID-002 — Passwordless Login

Supports

- Passkeys
- FIDO2
- WebAuthn
- Biometrics

---

## ID-003 — MFA Verification

Authentication methods

- TOTP
- Push Notification
- SMS
- Email
- Hardware Key

---

## ID-004 — Password Recovery

Functions

- Email verification
- Security questions (optional)
- Reset password
- Session revocation

---

## ID-005 — Account Verification

Functions

- Email confirmation
- Phone verification
- Identity verification

---

# Organization Management

---

## ID-100 — Organization Dashboard

Displays

- Organization profile
- Users
- Security score
- Active sessions
- Federation status
- Compliance overview

---

## ID-101 — Organization Details

Includes

- General information
- Subscription
- Identity providers
- Domains
- Policies

---

## ID-102 — Organization Settings

Configure

- Branding
- Authentication
- Password policy
- Session policy
- MFA defaults

---

# User Management

---

## ID-200 — User Directory

Features

- Global search
- Filters
- Bulk operations
- Import
- Export

Columns

- Name
- Email
- Status
- Department
- Risk
- Last Login

---

## ID-201 — User Profile

Sections

- Personal Information
- Credentials
- Roles
- Groups
- Devices
- Sessions
- Audit Timeline

---

## ID-202 — Create User

Supports

- Manual creation
- SCIM
- CSV import
- HR integration

---

## ID-203 — User Lifecycle

Displays

- Provisioning
- Activation
- Suspension
- Deactivation
- Archive

---

# Group Management

---

## ID-300 — Groups

Functions

- Create groups
- Nested groups
- Membership
- Dynamic groups

---

## ID-301 — Group Details

Displays

- Members
- Assigned roles
- Policies
- Audit history

---

# Role Management

---

## ID-400 — Roles

Supports

- RBAC
- Custom roles
- Built-in roles

---

## ID-401 — Role Designer

Capabilities

- Permission editor
- Scope assignment
- Inheritance
- Validation

---

# Permission Management

---

## ID-500 — Permission Catalog

Displays

- Resources
- Actions
- Conditions
- Scope

---

## ID-501 — Permission Matrix

Interactive matrix

Users

↓

Roles

↓

Permissions

↓

Resources

---

# Federation

---

## ID-600 — Identity Providers

Supported

- Entra ID
- Okta
- Google
- Auth0
- Keycloak
- LDAP

---

## ID-601 — Federation Configuration

Configure

- OIDC
- OAuth
- SAML
- SCIM

---

## ID-602 — Domain Management

Manage

- Verified domains
- Domain routing
- Federation mapping

---

# Applications

---

## ID-700 — Applications

Displays

- OAuth Clients
- Enterprise Apps
- API Consumers

---

## ID-701 — Application Details

Includes

- Secrets
- Redirect URIs
- Scopes
- Tokens
- Certificates

---

# AI Identity

---

## ID-800 — AI Agents

Displays

- AI identities
- Models
- Permissions
- Risk

---

## ID-801 — AI Agent Details

Sections

- Identity
- Policies
- Prompt Access
- Tool Access
- MCP Connections
- Audit

---

## ID-802 — AI Policy Center

Configure

- AI permissions
- AI scopes
- Token limits
- Cost controls

---

# MCP

---

## ID-850 — MCP Servers

Displays

- Registered servers
- Health
- Version
- Authentication

---

## ID-851 — MCP Resources

Displays

- Resources
- Tools
- Permissions
- Sessions

---

# Machine Identity

---

## ID-900 — Service Accounts

Manage

- Service identities
- Secrets
- Certificates
- Rotation

---

## ID-901 — Devices

Displays

- Managed devices
- Trust level
- Compliance
- Last activity

---

## ID-902 — Certificates

Manage

- Certificates
- Expiration
- Renewal
- Revocation

---

# Sessions

---

## ID-1000 — Active Sessions

Displays

- User
- Device
- IP
- Browser
- Risk
- Duration

Actions

- Revoke
- Terminate
- Investigate

---

# Audit

---

## ID-1100 — Audit Explorer

Supports

- Full search
- Timeline
- Correlation
- Export

---

## ID-1101 — Activity Timeline

Displays

- Authentication
- Authorization
- Changes
- Policies
- Federation
- AI

---

# Security

---

## ID-1200 — Security Dashboard

Displays

- Threats
- Risk Score
- Failed Logins
- MFA Adoption
- Passwordless Adoption

---

## ID-1201 — Conditional Access

Configure

- Risk policies
- Device policies
- Location rules
- Session controls

---

## ID-1202 — Risk Center

Displays

- Suspicious users
- Impossible travel
- Credential leaks
- High-risk sessions

---

# Governance

---

## ID-1300 — Access Reviews

Supports

- Certification campaigns
- Managers
- Review status

---

## ID-1301 — Identity Governance

Displays

- Role mining
- Segregation of Duties
- Policy violations
- Compliance

---

# Analytics

---

## ID-1400 — Identity Analytics

Displays

- Authentication trends
- User growth
- Federation usage
- AI identities

---

## ID-1401 — Security Analytics

Metrics

- Login success
- Threats
- Risk
- MFA usage
- Passwordless adoption

---

# Notifications

---

## ID-1500 — Notification Center

Displays

- Security alerts
- Approvals
- AI recommendations
- Compliance notices

---

# Administration

---

## ID-1600 — Platform Settings

Configure

- Identity platform
- Global policies
- Branding
- Integrations

---

## ID-1601 — System Health

Displays

- Services
- Federation
- OAuth
- Databases
- APIs

---

# Developer Portal

---

## ID-1700 — OAuth Applications

Manage

- OAuth clients
- Secrets
- Redirects
- PKCE

---

## ID-1701 — API Explorer

Supports

- OpenAPI
- Token testing
- OAuth playground
- SDK download

---

# Global Components

Available on every screen

- Global Search
- Notifications
- Help
- User Profile
- Tenant Selector
- Language Selector
- Theme Switcher

---

# Responsive Behavior

| Device | Experience |
|----------|------------|
| Desktop | Full |
| Laptop | Full |
| Tablet | Optimized |
| Mobile | Essential Operations |

---

# Screen Relationships

```text
Login

↓

Dashboard

↓

Users

↓

Profile

↓

Permissions

↓

Audit
```

---

# Estimated Screen Inventory

| Module | Screens |
|----------|---------:|
| Authentication | 12 |
| Organizations | 8 |
| Users | 18 |
| Groups | 8 |
| Roles | 10 |
| Permissions | 10 |
| Federation | 12 |
| Applications | 10 |
| AI Identity | 12 |
| MCP | 8 |
| Machine Identity | 10 |
| Sessions | 6 |
| Audit | 10 |
| Security | 12 |
| Governance | 8 |
| Analytics | 10 |
| Administration | 12 |
| Developer Portal | 8 |
| Notifications | 4 |
| **Total Screens** | **198** |

---

# Repository Structure

```text
08-screen-catalog/

├── authentication/
├── dashboard/
├── organizations/
├── users/
├── groups/
├── roles/
├── permissions/
├── federation/
├── applications/
├── ai-identities/
├── mcp/
├── machine-identities/
├── sessions/
├── audit/
├── security/
├── governance/
├── analytics/
├── administration/
├── developer/
├── navigation-map.md
├── screen-inventory.md
├── screen-matrix.md
├── glossary.md
├── diagrams/
│   ├── navigation-tree.drawio
│   ├── screen-map.drawio
│   ├── admin-console.drawio
│   ├── authentication-flow.drawio
│   ├── governance-navigation.drawio
│   ├── ai-console.drawio
│   ├── developer-portal.drawio
│   └── responsive-layouts.drawio
└── metadata.yml
```

---

# Screen Design Standards

Every screen must provide

- Consistent navigation
- Breadcrumbs
- Context actions
- Search
- Filters
- Inline help
- Keyboard navigation
- Accessibility
- Audit visibility
- Responsive layout

---

# Architecture Principles

The screen architecture follows

- Dashboard First
- Search First
- Progressive Disclosure
- Security by Design
- Mobile Awareness
- Accessibility First
- Enterprise Consistency
- AI Assistance
- Contextual Navigation
- Minimal Cognitive Load

---

# Traceability

| Related Volume | Relationship |
|---------------|--------------|
| Enterprise Platform | Shared Navigation |
| Security Platform | Security Dashboards |
| AI Platform | AI Agent Management |
| Integration Platform | OAuth & API Clients |
| Administration Platform | Platform Administration |
| Design System | UI Components |

---

# Acceptance Criteria

This chapter is complete when:

- All Identity Platform screens are cataloged and uniquely identified.
- Functional responsibilities, navigation hierarchy and supported personas are documented.
- Administrative, authentication, governance, AI, machine identity and analytics interfaces are specified.
- Repository organization, screen inventory, design standards and traceability are complete.
- The Screen Catalog provides a complete reference for designing and implementing every user interface of the EVOXA Identity Platform.

---

# Key Takeaways

- The EVOXA Identity Platform contains nearly 200 specialized screens supporting the complete identity lifecycle, governance and enterprise security operations.
- Authentication, administration, AI identity management, federation and compliance are organized into a consistent and scalable navigation model.
- Every screen adheres to common UX, accessibility, security and responsive design standards defined by the EVOXA Design System.
- This screen catalog serves as the authoritative UI reference for implementing a unified, enterprise-grade Identity Platform experience.

---

# Next Section

**09 — Layout Architecture**

The next chapter defines the structural layout of the Identity Platform, including global page templates, workspace organization, navigation regions, responsive layouts, dashboard composition and reusable interface patterns.
