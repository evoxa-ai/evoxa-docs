---
document_id: BP-0018-C06
chapter_id: CH-18-06
volume: Volume 18 — Analytics Platform
title: Information Architecture
version: 1.0.0
status: Approved
owner: Enterprise Information Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-07
---

# 06 — Information Architecture

> *The Information Architecture chapter defines how analytical information is organized, structured, classified, discovered and consumed throughout the EVOXA Analytics Platform. It establishes the enterprise taxonomy, semantic organization, metadata model, navigation hierarchy and information relationships that enable users and AI services to efficiently locate and understand analytical assets.*

---

# Executive Summary

Enterprise Analytics is successful only when users can quickly locate trusted information.

A modern analytics platform must organize:

- Dashboards
- Reports
- KPIs
- Metrics
- Datasets
- AI Insights
- Alerts
- Data Models
- Business Glossaries
- Semantic Objects

into a coherent information architecture.

The EVOXA Analytics Platform implements a semantic, metadata-driven and AI-assisted information architecture that allows information to be discovered naturally by both humans and AI.

---

# Information Vision

The platform follows one guiding principle:

> **Every Analytical Asset Should Be Easy to Discover, Understand and Trust.**

---

# Objectives

The Information Architecture provides

- Enterprise Taxonomy
- Semantic Organization
- Metadata Management
- Content Discoverability
- AI Search
- Information Governance
- Data Lineage
- Consistent Navigation
- Business Glossary
- Enterprise Knowledge Organization

---

# Information Architecture Layers

```text
Presentation Layer

↓

Navigation Layer

↓

Business Objects

↓

Semantic Layer

↓

Datasets

↓

Metadata

↓

Data Sources
```

---

# Information Hierarchy

```text
Platform

↓

Workspace

↓

Analytics Domain

↓

Dashboard

↓

Page

↓

Widget

↓

Visualization

↓

Metric
```

---

# Primary Information Domains

| Domain | Description |
|----------|-------------|
| Dashboards | Analytical workspaces |
| Reports | Structured reporting |
| KPIs | Business indicators |
| Metrics | Calculated values |
| Datasets | Analytical datasets |
| AI Insights | AI-generated intelligence |
| Alerts | Business notifications |
| Models | Analytical models |
| Data Sources | Enterprise systems |
| Catalog | Discoverable assets |

---

# Enterprise Taxonomy

```text
Analytics

├── Executive
├── Financial
├── Sales
├── Marketing
├── Customer
├── HR
├── Operations
├── Security
├── Identity
├── Product
├── AI
└── Infrastructure
```

---

# Navigation Hierarchy

```text
Home

↓

Workspaces

↓

Dashboards

↓

Reports

↓

Datasets

↓

AI Analytics

↓

Administration
```

---

# Workspace Organization

Each workspace contains

- Dashboards
- Reports
- KPIs
- Metrics
- Alerts
- Saved Queries
- AI Insights
- Shared Assets

---

# Dashboard Hierarchy

```text
Workspace

↓

Dashboard

↓

Sections

↓

Widgets

↓

Charts

↓

Metrics
```

---

# Report Hierarchy

```text
Category

↓

Report

↓

Sections

↓

Charts

↓

Tables

↓

Narratives
```

---

# KPI Organization

Each KPI contains

- Name
- Description
- Formula
- Owner
- Data Source
- Business Domain
- Thresholds
- History
- Status

---

# Metric Hierarchy

```text
Business Domain

↓

Category

↓

Metric

↓

Calculation

↓

Visualization
```

---

# Semantic Layer

The Semantic Layer abstracts technical complexity.

Business users interact with

- Customers
- Orders
- Revenue
- Products
- Employees
- Campaigns

instead of raw database tables.

---

# Metadata Model

Each analytical asset stores

- Identifier
- Name
- Description
- Owner
- Classification
- Tags
- Business Domain
- Version
- Lineage
- Security Level
- Last Updated

---

# Metadata Categories

| Category | Examples |
|----------|----------|
| Business | KPI, Department |
| Technical | Dataset, Schema |
| Governance | Steward, Owner |
| Security | Classification |
| Operational | Refresh Schedule |
| AI | Confidence Score |

---

# Business Glossary

Defines

- Business Terms
- KPI Definitions
- Formulas
- Acronyms
- Data Owners
- Usage Guidelines

---

# Data Catalog

The catalog indexes

- Dashboards
- Reports
- Datasets
- Metrics
- AI Models
- Data Sources
- APIs
- Events
- Pipelines

---

# Search Architecture

Search supports

- Full Text
- Metadata
- Business Terms
- AI Search
- Semantic Search
- Natural Language Queries

---

# AI Knowledge Graph

```text
Business Domains

↓

Metrics

↓

KPIs

↓

Reports

↓

Dashboards

↓

Insights
```

Relationships enable AI reasoning.

---

# Content Classification

Information is classified as

| Level | Description |
|--------|-------------|
| Public | Organization-wide |
| Internal | Internal analytics |
| Confidential | Restricted business data |
| Sensitive | Regulated information |

---

# Data Lineage

Every analytical object traces

```text
Source System

↓

Pipeline

↓

Transformation

↓

Dataset

↓

Metric

↓

Dashboard
```

---

# Information Lifecycle

```text
Create

↓

Review

↓

Publish

↓

Consume

↓

Update

↓

Archive
```

---

# Content Ownership

Every asset has

- Business Owner
- Technical Owner
- Data Steward
- Reviewer
- Approver

---

# Discoverability

Users locate assets through

- Search
- Categories
- Tags
- Favorites
- Recently Viewed
- Recommendations
- AI Copilot

---

# Personalization

The platform personalizes

- Home page
- Favorite dashboards
- Recommended reports
- Frequently used KPIs
- AI suggestions

---

# Cross-Platform Navigation

Links are available to

- Identity Platform
- Workflow Platform
- AI Platform
- Integration Platform
- Data Platform
- Security Platform

---

# Information Governance

Governance controls

- Naming standards
- Versioning
- Ownership
- Metadata quality
- Lifecycle management
- Approval workflows

---

# Accessibility

Information architecture supports

- Screen readers
- Keyboard navigation
- WCAG 2.2 AA
- Responsive layouts
- Internationalization

---

# Information Quality Principles

Every asset must be

- Accurate
- Complete
- Timely
- Discoverable
- Consistent
- Governed
- Explainable
- Auditable

---

# Information Flow

```text
Enterprise Data

↓

Metadata

↓

Semantic Layer

↓

Analytics Assets

↓

User Experience

↓

Business Decisions
```

---

# Repository Structure

```text
06-information-architecture/

├── overview.md
├── taxonomy.md
├── navigation.md
├── workspaces.md
├── dashboards.md
├── reports.md
├── semantic-layer.md
├── metadata-model.md
├── business-glossary.md
├── data-catalog.md
├── search.md
├── personalization.md
├── governance.md
├── information-lifecycle.md
├── glossary.md
├── diagrams/
│   ├── information-architecture.drawio
│   ├── navigation-hierarchy.drawio
│   ├── semantic-layer.drawio
│   ├── metadata-model.drawio
│   ├── information-flow.drawio
│   ├── data-lineage.drawio
│   ├── taxonomy.drawio
│   ├── search-architecture.drawio
│   └── workspace-organization.drawio
└── metadata.yml
```

---

# Information Asset Inventory

| Area | Assets |
|------|--------:|
| Information Domains | 12 |
| Navigation Structures | 18 |
| Taxonomy Definitions | 42 |
| Metadata Attributes | 55 |
| Business Glossary Terms | 300+ |
| Semantic Objects | 180 |
| Dashboard Categories | 20 |
| Search Indexes | 25 |
| Governance Rules | 32 |
| Data Lineage Models | 18 |
| **Total Information Assets** | **702** |

---

# Architecture Principles

The Information Architecture follows

- Information First
- Semantic by Design
- Metadata Driven
- Discoverability by Default
- AI-Ready Knowledge
- Business-Oriented Navigation
- Governance by Design
- Consistent Taxonomy
- Accessibility by Default
- Enterprise Scalability

---

# Traceability

| Related Volume | Relationship |
|---------------|--------------|
| Data Platform | Data Catalog & Metadata |
| AI Platform | Semantic Knowledge Graph |
| Enterprise Platform | Enterprise Taxonomy |
| Identity Platform | Information Security |
| Integration Platform | Source Systems |
| UX Platform | Navigation & User Experience |

---

# Acceptance Criteria

This chapter is complete when:

- The information hierarchy, taxonomy and navigation model are fully documented.
- Semantic layer, metadata model, data catalog and business glossary are defined.
- Information governance, discoverability, personalization and lifecycle management are established.
- Repository organization, information assets, architectural principles and traceability are complete.
- The Analytics Platform provides a consistent, searchable and governed information architecture that enables users and AI services to efficiently discover and consume trusted analytical assets.

---

# Key Takeaways

- The EVOXA Analytics Platform organizes all analytical assets through a semantic, metadata-driven information architecture.
- Enterprise taxonomy, business glossary and data catalog ensure that dashboards, reports, KPIs and datasets are consistently classified and easily discoverable.
- AI-enhanced search, personalization and semantic relationships significantly reduce the time required to locate and understand analytical information.
- This information architecture establishes the structural foundation for scalable, governed and AI-powered enterprise analytics.

---

# Next Section

**07 — UX Goals**

The next chapter defines the user experience goals, usability principles, interaction patterns, personalization strategy and AI-assisted experience standards that guide every interface of the EVOXA Analytics Platform.
