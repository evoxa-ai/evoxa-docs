---
document_id: BP-0003-V3-C11-06
chapter_id: CH-11-ANL-06
feature_pack: FP-ANL-0001
title: Information Architecture
version: 1.0.0
status: Draft
owner: Enterprise Information Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-07
---

# Chapter 06 — Information Architecture

> *The Information Architecture chapter defines how analytical information is organized, classified, related, discovered and consumed within the EVOXA Enterprise Analytics Platform. It establishes the taxonomy, semantic hierarchy, navigation model, metadata structure and governance framework that enable users and AI services to efficiently access trusted business intelligence.*

---

# Executive Summary

Enterprise Analytics is successful only when users can easily find, understand and trust information.

The Enterprise Analytics Platform organizes every dashboard, KPI, report, metric, dataset and AI insight into a unified information architecture built around business domains rather than technical systems.

This architecture enables:

- Fast information discovery
- Consistent business terminology
- Semantic navigation
- AI-powered search
- Data governance
- Self-service analytics

---

# Objectives

The Information Architecture shall:

- Organize enterprise knowledge.
- Standardize business terminology.
- Simplify navigation.
- Improve discoverability.
- Enable semantic analytics.
- Support AI reasoning.
- Maintain governance.

---

# Information Architecture Principles

The platform follows:

- Business First
- Semantic by Design
- Metadata Driven
- AI Ready
- Search First
- Consistent Navigation
- Enterprise Taxonomy
- Governed Information

---

# Information Hierarchy

```text
Organization

↓

Business Domain

↓

Analytical Area

↓

Dashboard

↓

Page

↓

Widget

↓

Metric
```

---

# Enterprise Information Model

```text
Enterprise

↓

Business Domains

↓

Datasets

↓

Semantic Models

↓

Analytics Assets

↓

Insights
```

---

# Core Information Components

The platform organizes:

- Dashboards
- Reports
- KPIs
- Metrics
- Datasets
- Dimensions
- Measures
- Alerts
- AI Insights
- Forecasts
- Scorecards
- Data Catalog
- Semantic Models

---

# Business Domains

Analytics are organized by business domains.

Primary domains include:

- Executive
- Finance
- Sales
- Marketing
- Operations
- Customer Success
- Human Resources
- AI Operations
- Security
- Infrastructure
- Product
- Integration

---

# Information Taxonomy

```text
Domain

↓

Category

↓

Analytics Asset

↓

Visualization

↓

Metric
```

---

# Navigation Hierarchy

```text
Workspace

↓

Analytics Hub

↓

Business Domain

↓

Dashboard

↓

Detail View

↓

Report

↓

Insight
```

---

# Analytics Asset Types

The platform manages:

## Dashboards

Interactive visual workspaces.

---

## Reports

Structured business documents.

---

## KPIs

Certified business indicators.

---

## Metrics

Reusable numerical measurements.

---

## Datasets

Governed analytical data.

---

## Semantic Models

Business-friendly analytical abstractions.

---

## AI Insights

Automatically generated recommendations.

---

## Forecasts

Predictive analytical models.

---

# Content Organization

Content is grouped by:

- Business Domain
- Department
- Team
- Workspace
- Project
- Product
- Region
- Time Period

---

# Metadata Architecture

Every analytical asset contains metadata.

Standard metadata includes:

- Name
- Description
- Owner
- Business Domain
- Tags
- Classification
- Version
- Created Date
- Modified Date
- Certification Status
- Sensitivity Level

---

# Business Glossary

The platform maintains a centralized glossary.

Examples:

| Business Term | Definition |
|---------------|------------|
| Revenue | Total recognized income |
| Active Customer | Customer with activity during the selected period |
| Monthly Recurring Revenue | Predictable subscription revenue |
| Customer Churn | Percentage of customers lost during a period |

---

# Semantic Layer

The Semantic Layer defines:

- Business Dimensions
- Business Measures
- Standard KPIs
- Derived Metrics
- Business Relationships

It ensures analytical consistency across every dashboard.

---

# Knowledge Relationships

```text
Dataset

↓

Semantic Model

↓

Dashboard

↓

KPI

↓

Business Insight
```

---

# Search Architecture

Users can search by:

- Dashboard
- KPI
- Report
- Dataset
- Metric
- Business Term
- AI Insight
- Tags
- Owner

---

# AI Semantic Search

The AI Search Engine supports:

- Natural language search
- Intent detection
- Similar dashboards
- Related metrics
- Suggested reports
- Context-aware recommendations

Example:

> "Show revenue trends for LATAM during the last 12 months."

---

# Information Discovery

Discovery methods include:

- Global Search
- Navigation Tree
- Favorites
- Recently Viewed
- Recommended Assets
- AI Recommendations
- Trending Dashboards

---

# Dashboard Classification

Dashboards are categorized as:

- Executive
- Operational
- Financial
- Product
- Customer
- Marketing
- AI
- Security
- Infrastructure

---

# Dataset Organization

Datasets are classified by:

- Domain
- Business Owner
- Data Steward
- Sensitivity
- Refresh Frequency
- Certification Status

---

# KPI Catalog

Every KPI contains:

- Business Definition
- Formula
- Unit
- Target
- Thresholds
- Owner
- Refresh Frequency
- Data Source

---

# Report Catalog

Every report contains:

- Purpose
- Audience
- Schedule
- Export Formats
- Data Sources
- Business Owner

---

# Information Lifecycle

```text
Create

↓

Review

↓

Approve

↓

Publish

↓

Monitor

↓

Archive
```

---

# Workspace Organization

```text
Enterprise

├── Executive
├── Finance
├── Sales
├── Marketing
├── Product
├── Operations
├── AI
├── Security
└── Infrastructure
```

---

# Content Governance

Governed assets include:

- Dashboards
- Reports
- Metrics
- KPIs
- Datasets
- AI Insights
- Semantic Models

---

# Versioning

Every analytical asset supports:

- Version History
- Change Log
- Rollback
- Approval Workflow
- Certification

---

# Information Security Classification

Assets are classified as:

- Public
- Internal
- Confidential
- Restricted

Classification determines visibility and permissions.

---

# Localization

Information architecture supports:

- Multi-language labels
- Regional terminology
- Localized business glossary
- Regional date formats
- Currency formatting

---

# Mobile Information Architecture

Mobile prioritizes:

- Executive KPIs
- Alerts
- AI Summaries
- Favorite Dashboards
- Recently Viewed Reports

---

# Information Analytics

The platform measures:

- Search Success
- Dashboard Discovery
- Report Usage
- KPI Consumption
- AI Recommendation Acceptance
- Navigation Efficiency

---

# Performance Targets

| Metric | Target |
|---------|---------|
| Search Response | <1 sec |
| Dashboard Discovery | <3 clicks |
| Metadata Retrieval | <500 ms |
| Semantic Query | <2 sec |
| Navigation Response | <200 ms |

---

# Information KPIs

| KPI | Target |
|------|--------|
| Metadata Coverage | 100% |
| Certified KPIs | 100% |
| Search Success Rate | >95% |
| Dashboard Discoverability | >90% |
| Business Glossary Coverage | 100% |
| Semantic Consistency | 100% |

---

# Repository Structure

```text
06-information-architecture/
├── taxonomy/
├── semantic-layer/
├── metadata/
├── dashboards/
├── reports/
├── datasets/
├── kpis/
├── glossary/
├── search/
├── governance/
├── navigation/
├── localization/
├── versioning/
├── assets/
└── metadata.yml
```

---

# Standard Visual Artifacts

## Enterprise Information Hierarchy

```text
Enterprise

↓

Business Domain

↓

Dashboard

↓

Widget

↓

Metric
```

---

## Semantic Relationships

```text
Dataset

↓

Semantic Model

↓

Dashboard

↓

Insight
```

---

## Navigation Tree

```text
Workspace

↓

Domain

↓

Dashboard

↓

Report
```

---

## Information Lifecycle

```text
Create

↓

Review

↓

Publish

↓

Archive
```

---

## AI Search Flow

```text
Question

↓

Semantic Search

↓

Knowledge Graph

↓

Analytics Assets

↓

Results
```

---

# Visual Source Files

```text
artifacts/
└── information-architecture/
    ├── enterprise-taxonomy.drawio
    ├── semantic-layer.drawio
    ├── navigation-tree.drawio
    ├── information-lifecycle.drawio
    ├── ai-search.drawio
    ├── metadata-model.drawio
    ├── workspace-structure.drawio
    ├── mermaid/
    │   ├── taxonomy.mmd
    │   ├── semantic-layer.mmd
    │   ├── metadata.mmd
    │   ├── navigation.mmd
    │   ├── lifecycle.mmd
    │   ├── ai-search.mmd
    │   └── governance.mmd
    └── exports/
        ├── information-architecture.svg
        ├── information-architecture.png
        └── information-architecture.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Chapter 03 — Product Scope | Analytics Assets |
| Chapter 04 — Personas | User Navigation |
| Chapter 05 — User Journeys | Information Flows |
| Chapter 08 — Screen Catalog | Screen Organization |
| Chapter 10 — Navigation Architecture | Navigation Model |
| Chapter 12 — Component Catalog | Information Components |
| Chapter 17 — Business Rules | Information Governance |
| Chapter 21 — AI Services | Semantic Search & AI Discovery |
| Chapter 22 — Data Model | Metadata & Semantic Model |

---

# Acceptance Criteria

This chapter is complete when:

- The enterprise information architecture, taxonomy and semantic organization are fully documented.
- Dashboards, reports, datasets, KPIs, metadata, glossary and search architecture are defined.
- Information governance, lifecycle management, localization and versioning strategies are specified.
- Repository structure, visual artifacts, KPIs and traceability are complete.
- The Information Architecture provides a scalable, discoverable and governed foundation for every analytical asset within the EVOXA Enterprise Analytics Platform.

---

# Key Takeaways

- The Enterprise Analytics Platform organizes information around business concepts rather than technical systems, improving usability and decision-making.
- A centralized semantic layer, business glossary and metadata model ensure that every analytical asset is consistent, searchable and trustworthy.
- AI-powered semantic search and intelligent discovery dramatically reduce the effort required to locate dashboards, reports and business insights.
- This information architecture establishes the structural foundation for scalable enterprise analytics, self-service BI and AI-assisted knowledge discovery.

---

# Next Chapter

**Chapter 07 — UX Goals**

The next chapter defines the user experience strategy for the Enterprise Analytics Platform, including usability objectives, interaction principles, AI-assisted experiences, accessibility goals and design metrics that guide every analytical interface.
