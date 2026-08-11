---
document_id: BP-0017-C04
chapter_id: CH-17-04
volume: Volume 17 — Identity Platform
title: Personas
version: 1.0.0
status: Draft
owner: Enterprise Identity Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-07
---

# 04 — Personas

> *The Personas chapter defines the primary users, systems and digital identities that interact with the EVOXA Identity Platform. It describes their objectives, responsibilities, permissions, behaviors and security requirements to ensure the platform delivers a secure, intuitive and enterprise-grade identity experience.*

---

# Executive Summary

The Identity Platform serves far more than traditional end users.

Modern Identity and Access Management (IAM) platforms manage identities belonging to:

- Employees
- Customers
- Contractors
- Administrators
- Developers
- Security teams
- AI Agents
- Service Accounts
- APIs
- Machines
- Devices

Each identity type has unique goals, responsibilities and security requirements.

---

# Persona Vision

The platform follows one principle:

> **Every Identity Has a Purpose. Every Purpose Defines Trust.**

---

# Persona Classification

The Identity Platform recognizes four major categories.

```text
Human Identities

↓

Machine Identities

↓

AI Identities

↓

Organizational Identities
```

---

# Persona Map

```text
Enterprise

├── Executive Users

├── Business Users

├── Technical Users

├── Security Teams

├── AI Systems

├── Services

└── External Organizations
```

---

# Primary Personas

| Persona | Category |
|----------|----------|
| Organization Owner | Human |
| Identity Administrator | Human |
| Security Administrator | Human |
| IAM Engineer | Human |
| Platform Administrator | Human |
| Developer | Human |
| AI Engineer | Human |
| Auditor | Human |
| Business User | Human |
| External Partner | Human |
| Customer | Human |
| AI Agent | AI |
| MCP Server | AI |
| Service Account | Machine |
| API Client | Machine |
| Trusted Device | Machine |

---

# Persona 1 — Organization Owner

## Description

Responsible for the organization's identity governance.

---

## Responsibilities

- Manage organization
- Configure security policies
- Approve administrators
- Review compliance
- Manage subscriptions

---

## Goals

- Maintain security
- Ensure compliance
- Delegate responsibilities
- Monitor identity health

---

## Permissions

- Full tenant administration
- User lifecycle
- Billing visibility
- Security policies

---

## Authentication

Mandatory

- MFA
- Passwordless
- Device trust

---

# Persona 2 — Identity Administrator

## Description

Manages enterprise identities.

---

## Responsibilities

- Provision users
- Manage groups
- Reset credentials
- Configure federation
- Assign roles

---

## Goals

- Automate identity lifecycle
- Reduce manual work
- Maintain consistency

---

## Typical Activities

- Create users
- Disable accounts
- Configure SCIM
- Manage SSO
- Review permissions

---

# Persona 3 — Security Administrator

## Description

Responsible for enterprise security.

---

## Responsibilities

- MFA policies
- Conditional access
- Risk monitoring
- Threat response
- Incident investigation

---

## Goals

- Prevent identity attacks
- Reduce risk
- Enforce Zero Trust

---

# Persona 4 — IAM Engineer

## Description

Designs and operates the Identity Platform.

---

## Responsibilities

- Identity architecture
- Federation
- OAuth
- OpenID Connect
- Directory synchronization

---

## Goals

- High availability
- Scalability
- Security
- Standards compliance

---

# Persona 5 — Platform Administrator

## Description

Operates the complete EVOXA platform.

---

## Responsibilities

- Platform configuration
- Service monitoring
- Identity integration
- Tenant management

---

# Persona 6 — Developer

## Description

Consumes Identity APIs.

---

## Responsibilities

- OAuth integration
- API development
- Authentication flows
- SDK implementation

---

## Needs

- Documentation
- SDKs
- API Keys
- Test environments

---

# Persona 7 — AI Engineer

## Description

Builds AI systems using enterprise identities.

---

## Responsibilities

- AI Agents
- MCP Servers
- AI Permissions
- Prompt security

---

## Goals

- Secure AI
- Govern AI identities
- AI authorization

---

# Persona 8 — Auditor

## Description

Reviews compliance.

---

## Responsibilities

- Audit logs
- Identity reviews
- Compliance reports
- Access certification

---

## Requirements

- Read-only access
- Immutable logs
- Reporting

---

# Persona 9 — Business User

## Description

Standard enterprise employee.

---

## Responsibilities

- Authenticate
- Use applications
- Manage profile
- Complete MFA

---

## Goals

- Fast login
- Secure access
- Simple experience

---

# Persona 10 — External Partner

## Description

Business partner accessing enterprise resources.

---

## Responsibilities

- Authenticate
- Collaborate
- Access shared applications

---

## Authentication

Typically

- Federation
- SAML
- OIDC

---

# Persona 11 — Customer

## Description

External consumer.

---

## Capabilities

- Self-registration
- MFA
- Password reset
- Consent management

---

# Persona 12 — AI Agent

## Description

Autonomous enterprise identity.

---

## Responsibilities

- Execute workflows
- Invoke APIs
- Use MCP tools
- Generate recommendations

---

## Identity

Owns

- Credentials
- Roles
- Policies
- Audit history
- Risk score

---

# Persona 13 — MCP Server

## Description

Enterprise Model Context Protocol server.

---

## Responsibilities

- Publish tools
- Serve resources
- Authenticate requests
- Maintain sessions

---

## Identity

Every MCP Server receives

- Service identity
- Certificate
- Token
- Permissions

---

# Persona 14 — Service Account

## Description

Machine identity used by applications.

---

## Responsibilities

- Background processing
- API authentication
- Workflow execution

---

## Authentication

- Certificates
- OAuth Client Credentials
- JWT

---

# Persona 15 — API Client

## Description

External application.

---

## Responsibilities

- Access protected APIs
- Refresh tokens
- Authenticate securely

---

# Persona 16 — Trusted Device

## Description

Enterprise-managed endpoint.

---

## Capabilities

- Device authentication
- Compliance verification
- Risk evaluation

---

# Persona Relationships

```text
Organization

├── Owner

├── Administrators

├── Users

├── Developers

├── AI Engineers

├── Auditors

├── Customers

├── AI Agents

├── Services

└── Devices
```

---

# Permission Matrix

| Persona | Read | Write | Admin | Audit |
|----------|:---:|:-----:|:-----:|:-----:|
| Organization Owner | ✓ | ✓ | ✓ | ✓ |
| Identity Administrator | ✓ | ✓ | ✓ | ✓ |
| Security Administrator | ✓ | ✓ | ✓ | ✓ |
| IAM Engineer | ✓ | ✓ | ✓ | ✓ |
| Platform Administrator | ✓ | ✓ | ✓ | ✓ |
| Developer | ✓ | ✓ | ✗ | ✗ |
| AI Engineer | ✓ | ✓ | Limited | ✗ |
| Auditor | ✓ | ✗ | ✗ | ✓ |
| Business User | Limited | Limited | ✗ | ✗ |
| Customer | Limited | Limited | ✗ | ✗ |
| AI Agent | Scoped | Scoped | ✗ | Logged |
| Service Account | Scoped | Scoped | ✗ | Logged |

---

# Authentication Requirements

| Persona | MFA | Passwordless | Federation |
|----------|:--:|:------------:|:----------:|
| Organization Owner | ✓ | ✓ | ✓ |
| Identity Administrator | ✓ | ✓ | ✓ |
| Security Administrator | ✓ | ✓ | ✓ |
| IAM Engineer | ✓ | ✓ | ✓ |
| Developer | Optional | ✓ | ✓ |
| Business User | Optional | ✓ | ✓ |
| Customer | Optional | ✓ | Optional |
| AI Agent | N/A | Token | N/A |
| Service Account | N/A | Certificate | N/A |

---

# Security Levels

```text
Level 1

Customer

↓

Business User

↓

Developer

↓

Administrator

↓

Security Administrator

↓

Organization Owner
```

---

# AI Personas

AI identities include

- Enterprise Copilot
- Workflow Agent
- Security Agent
- Identity Agent
- Governance Agent
- Documentation Agent
- Integration Agent

Each receives independent authorization.

---

# Machine Personas

Supported machines

- Containers
- Kubernetes Pods
- APIs
- Workers
- Connectors
- Databases
- Event Brokers
- MCP Servers

---

# User Journey Summary

```text
Provision

↓

Authenticate

↓

Authorize

↓

Execute

↓

Audit

↓

Review
```

---

# UX Considerations

The platform should provide

- Simple login
- Fast authentication
- Accessible interfaces
- Clear error messages
- Minimal authentication friction
- Secure defaults

---

# Repository Structure

```text
04-personas/

├── executive-personas.md
├── administrator-personas.md
├── developer-personas.md
├── ai-personas.md
├── machine-personas.md
├── customer-personas.md
├── permission-matrix.md
├── authentication-models.md
├── persona-journeys.md
├── glossary.md
├── diagrams/
│   ├── persona-map.drawio
│   ├── identity-types.drawio
│   ├── organization-hierarchy.drawio
│   ├── trust-levels.drawio
│   └── authentication-matrix.drawio
└── metadata.yml
```

---

# Persona Statistics

| Category | Count |
|-----------|------:|
| Human Personas | 11 |
| AI Personas | 6 |
| Machine Personas | 8 |
| Organization Roles | 12 |
| Permission Models | 30+ |
| Authentication Flows | 20+ |

---

# Design Principles

All personas follow

- Least Privilege
- Zero Trust
- Secure by Default
- Federation First
- Continuous Verification
- Adaptive Authentication
- Auditability
- Privacy by Design

---

# Traceability

| Related Volume | Relationship |
|---------------|--------------|
| Enterprise Platform | Employee & Organization Roles |
| Security Platform | Security Roles |
| AI Platform | AI Agents |
| Integration Platform | Service Accounts & API Clients |
| Administration Platform | Administrative Users |
| Developer Platform | Developer Personas |

---

# Acceptance Criteria

This chapter is complete when:

- All human, machine, AI and organizational personas are identified and documented.
- Responsibilities, goals, permissions, authentication methods and trust levels are clearly defined.
- Persona relationships, authentication requirements and authorization models are specified.
- UX considerations and repository artifacts support persona-driven design decisions.
- Readers understand who interacts with the Identity Platform, why they do so and what level of trust and access each persona requires.

---

# Key Takeaways

- The EVOXA Identity Platform manages a diverse ecosystem of human, machine and AI identities through a unified trust model.
- Each persona has clearly defined responsibilities, authentication requirements and authorization boundaries aligned with Zero Trust principles.
- AI agents, MCP servers and service accounts are treated as first-class enterprise identities with independent governance and auditing.
- A persona-driven architecture ensures that identity services remain secure, scalable and optimized for every type of enterprise actor.

---

# Next Section

**05 — User Journeys**

The next chapter documents the end-to-end journeys for each persona, including onboarding, authentication, federation, passwordless login, MFA enrollment, AI agent registration, service account provisioning, identity governance and access lifecycle management.
