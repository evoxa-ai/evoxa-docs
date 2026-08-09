---
document_id: BP-0003-V3-C15-06
chapter_id: CH-15-MCP-06
feature_pack: FP-MCP-0001
title: Information Architecture
version: 1.0.0
status: Draft
owner: Enterprise UX Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-07
---

# Chapter 06 — Information Architecture

> *The Information Architecture chapter defines the logical organization of the EVOXA MCP Platform. It establishes how information, MCP assets, enterprise services, AI resources and operational capabilities are structured, categorized, navigated and governed to provide an intuitive, scalable and enterprise-ready user experience.*

---

# Executive Summary

The MCP Platform manages thousands of enterprise assets including:

- MCP Servers
- Tools
- Resources
- Prompt Templates
- Context Providers
- Connectors
- Sessions
- Policies
- Logs
- Analytics

A consistent Information Architecture ensures these assets remain:

- Discoverable
- Reusable
- Governed
- Secure
- Extensible
- AI-Friendly

The architecture is designed to support both human users and autonomous AI agents.

---

# Information Architecture Principles

The platform follows:

- Domain-Driven Organization
- Progressive Disclosure
- Context Awareness
- Consistent Navigation
- AI-First Discovery
- Multi-Tenant Isolation
- Search-Centric Access
- Scalable Taxonomy

---

# Information Hierarchy

```text
Platform

↓

Domain

↓

Module

↓

Feature

↓

Entity

↓

Object
```

---

# Primary Navigation Domains

The MCP Platform is organized into:

```text
Dashboard

Servers

Tools

Resources

Prompts

Context

Models

Connectors

Marketplace

Analytics

Security

Administration
```

---

# Domain Overview

| Domain | Purpose |
|----------|----------|
| Dashboard | Operational overview |
| MCP Servers | Server management |
| Tools | Tool registry |
| Resources | Enterprise resources |
| Prompts | Prompt library |
| Context | Context providers |
| Models | AI providers |
| Connectors | Enterprise integrations |
| Marketplace | Shared assets |
| Analytics | Usage & telemetry |
| Security | Governance |
| Administration | Platform configuration |

---

# Information Layers

```text
Presentation

↓

Navigation

↓

Application

↓

Domain

↓

Metadata

↓

Persistence
```

---

# MCP Server Architecture

```text
MCP Server

↓

Capabilities

↓

Tools

↓

Resources

↓

Prompts

↓

Policies
```

---

# Tool Organization

Each Tool contains:

```text
Category

↓

Metadata

↓

Schema

↓

Permissions

↓

Version

↓

Documentation
```

---

# Resource Organization

Resources are grouped by:

- Knowledge
- APIs
- Files
- Databases
- Documents
- Images
- Reports
- External Systems

---

# Prompt Organization

Prompt taxonomy

```text
Business

↓

Technical

↓

AI

↓

Operations

↓

Security
```

Examples

- Customer Support
- Finance
- HR
- DevOps
- Security Review
- Architecture Review

---

# Context Organization

Context is categorized into:

```text
Identity

↓

Organization

↓

Tenant

↓

Conversation

↓

Session

↓

Memory

↓

Runtime
```

---

# Connector Organization

Enterprise connectors grouped by

- Productivity
- CRM
- ERP
- HR
- Collaboration
- Databases
- Cloud
- AI Services

---

# Marketplace Organization

Marketplace sections

```text
Featured

↓

Categories

↓

Publishers

↓

Collections

↓

My Assets
```

---

# Security Organization

Security information includes:

- Policies
- Roles
- Permissions
- API Keys
- OAuth Clients
- Audit Logs
- Compliance

---

# Analytics Organization

Analytics grouped into

- Usage
- Performance
- Costs
- AI
- Security
- Marketplace
- Connectors
- Organizations

---

# Metadata Model

Every managed object contains

```text
Identifier

Name

Description

Category

Owner

Version

Tags

Status

Created

Updated
```

---

# Taxonomy

Primary taxonomy

```text
Platform

↓

Domain

↓

Category

↓

Subcategory

↓

Entity
```

---

# Search Architecture

Unified enterprise search indexes:

- MCP Servers
- Tools
- Resources
- Prompts
- Connectors
- Documentation
- Marketplace Assets

Search supports

- Full-text
- Semantic
- Filters
- Tags
- AI Recommendations

---

# Tagging Strategy

Supported tags

- Business
- AI
- Security
- Finance
- HR
- Sales
- Marketing
- Production
- Internal
- Public

Objects may have unlimited tags.

---

# Classification Model

Information classification

| Level | Description |
|---------|-------------|
| Public | Publicly available |
| Internal | Enterprise users |
| Confidential | Restricted |
| Secret | Security controlled |

---

# Naming Standards

Naming conventions

- Human readable
- Globally unique
- Version aware
- Namespace based

Example

```text
finance.invoice.create
```

---

# Navigation Model

```text
Dashboard

↓

Domain

↓

Entity List

↓

Entity Details

↓

Configuration

↓

Analytics
```

---

# Breadcrumb Structure

Example

```text
Home

>

Servers

>

Finance Server

>

Invoice Tool
```

---

# Content Relationships

```text
Server

↓

Tool

↓

Resource

↓

Prompt

↓

Context

↓

Execution
```

---

# Cross-Linking

Every entity links to

- Owner
- Versions
- Permissions
- Usage
- Analytics
- Dependencies
- Documentation

---

# AI Navigation

AI Agents navigate through

```text
Task

↓

Discovery

↓

Registry

↓

Selection

↓

Execution
```

---

# Multi-Tenant Organization

```text
Tenant

↓

Organization

↓

Workspace

↓

Projects

↓

Assets
```

Every navigation path respects tenant isolation.

---

# Information Lifecycle

```text
Draft

↓

Review

↓

Published

↓

Deprecated

↓

Archived
```

---

# User Navigation

Business Users

```text
Dashboard

↓

Marketplace

↓

Assistant

↓

Execution
```

---

Developers

```text
Servers

↓

Tools

↓

Resources

↓

Publish
```

---

Administrators

```text
Platform

↓

Security

↓

Monitoring

↓

Configuration
```

---

# Information Governance

Every object supports

- Ownership
- Versioning
- Audit
- Classification
- Policies
- Approval
- Lifecycle

---

# Repository Structure

```text
06-information-architecture/
├── navigation/
├── taxonomy/
├── metadata/
├── search/
├── domains/
├── hierarchy/
├── classification/
├── lifecycle/
├── governance/
├── diagrams/
└── metadata.yml
```

---

# Information Inventory

| Domain | Objects |
|----------|--------:|
| MCP Servers | 5,000+ |
| Tools | 100,000+ |
| Resources | Millions |
| Prompt Templates | 25,000+ |
| Connectors | 500+ |
| AI Models | 100+ |
| Policies | 2,000+ |
| Marketplace Assets | Millions |

---

# Architecture Decision Records (ADRs)

| ADR | Decision |
|------|----------|
| ADR-0601 | Domain-Driven Information Architecture |
| ADR-0602 | Search-First Navigation |
| ADR-0603 | Unified Metadata Model |
| ADR-0604 | Multi-Tenant Taxonomy |
| ADR-0605 | Semantic Search Integration |
| ADR-0606 | Standardized Naming Conventions |
| ADR-0607 | AI-Native Information Discovery |
| ADR-0608 | Lifecycle-Based Information Governance |

---

# Standard Visual Artifacts

## Information Hierarchy

```text
Platform

↓

Domain

↓

Module

↓

Entity
```

---

## Navigation Model

```text
Dashboard

↓

Category

↓

Object
```

---

## Search Architecture

```text
Query

↓

Search

↓

Results

↓

Execution
```

---

## Taxonomy

```text
Domain

↓

Category

↓

Object
```

---

## Information Lifecycle

```text
Draft

↓

Published

↓

Archived
```

---

# Visual Source Files

```text
artifacts/
└── information-architecture/
    ├── navigation.drawio
    ├── taxonomy.drawio
    ├── metadata-model.drawio
    ├── search-architecture.drawio
    ├── hierarchy.drawio
    ├── governance.drawio
    ├── lifecycle.drawio
    ├── mermaid/
    │   ├── hierarchy.mmd
    │   ├── navigation.mmd
    │   ├── taxonomy.mmd
    │   ├── search.mmd
    │   ├── lifecycle.mmd
    │   ├── governance.mmd
    │   └── metadata.mmd
    └── exports/
        ├── information-architecture.svg
        ├── information-architecture.png
        └── information-architecture.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Chapter 04 — Personas | Role-Based Navigation |
| Chapter 05 — User Journeys | Navigation Flows |
| Chapter 08 — Screen Catalog | Screen Organization |
| Chapter 10 — Navigation Architecture | Navigation System |
| Chapter 12 — Component Catalog | UI Components |
| Chapter 13 — Widget Catalog | Information Widgets |
| Chapter 22 — Data Model | Metadata Structure |
| Chapter 23 — Security & Permissions | Information Classification |

---

# Acceptance Criteria

This chapter is complete when:

- The logical organization of MCP Servers, Tools, Resources, Prompts, Context Providers and Connectors is fully defined.
- Navigation hierarchy, metadata model, taxonomy, classification and search architecture are documented.
- Multi-tenant organization, information governance, lifecycle management and AI navigation patterns are specified.
- Repository structure, inventories, ADRs, visual artifacts and traceability are complete.
- The Information Architecture provides a scalable, discoverable and enterprise-ready organizational model that supports both human users and autonomous AI agents across the EVOXA MCP Platform.

---

# Key Takeaways

- The EVOXA MCP Platform organizes all information through a domain-driven architecture centered on discoverability, governance and AI-native interaction.
- Unified metadata, semantic search, standardized taxonomy and lifecycle management ensure that enterprise assets remain reusable and easy to manage at scale.
- Human users and AI agents share the same logical information model, enabling consistent discovery, execution and governance across every platform capability.
- This information architecture establishes the structural foundation for the user interface, navigation system, registries and management consoles defined in the following chapters.

---

# Next Chapter

**Chapter 07 — UX Goals**

The next chapter defines the user experience objectives of the EVOXA MCP Platform, including usability principles, interaction patterns, AI-assisted workflows, consistency guidelines and experience metrics for developers, administrators, business users and autonomous AI agents.
