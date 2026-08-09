---
document_id: BP-0003-V3-C14-08
chapter_id: CH-14-MP-08
feature_pack: FP-MARKETPLACE-0001
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

> *The Screen Catalog defines every user interface of the EVOXA Marketplace. It provides the complete inventory of screens, workspaces, dialogs, dashboards and administrative consoles required to support asset discovery, publishing, governance, monetization and lifecycle management.*

---

# Executive Summary

The Marketplace is one of the largest user-facing modules of the EVOXA Platform.

It supports multiple personas including:

- Enterprise Consumers
- Publishers
- AI Creators
- Partners
- Enterprise Administrators
- Marketplace Administrators
- Reviewers
- Platform Operators

Each persona interacts with specialized interfaces while sharing a unified design system.

The Marketplace UI is designed as a modular experience where every screen is composed of reusable components and widgets.

---

# Screen Design Principles

Every Marketplace screen follows:

- Enterprise-first
- AI-assisted
- Responsive
- Accessible
- Context-aware
- Component-based
- Minimal cognitive load
- Consistent navigation

---

# Screen Architecture

```text
Workspace

↓

Module

↓

Screen

↓

Section

↓

Component

↓

Widget
```

---

# Marketplace Navigation Map

```text
Marketplace

├── Home
├── Browse
├── Categories
├── Collections
├── Search
├── Asset Details
├── Install Wizard
├── My Assets
├── Publisher Center
├── Partner Center
├── Analytics
├── Administration
└── Settings
```

---

# Screen Classification

The Marketplace contains seven major screen groups:

- Discovery
- Asset Management
- Publishing
- Commerce
- Administration
- Analytics
- Platform Settings

---

# Discovery Screens

---

## MP-001 — Marketplace Home

Purpose

Landing page providing personalized discovery.

Primary Components

- Hero Banner
- AI Search
- Featured Collections
- Trending Assets
- Personalized Recommendations
- Categories
- Recently Updated
- Continue Exploring

Primary Users

- All users

---

## MP-002 — Browse Marketplace

Purpose

Browse assets by category.

Features

- Filters
- Sort
- Pagination
- Category Tree
- AI Suggestions

---

## MP-003 — Category Explorer

Purpose

Browse hierarchical Marketplace taxonomy.

Supports

- Categories
- Subcategories
- Collections
- Industry Packs

---

## MP-004 — Collections

Purpose

Display curated Marketplace collections.

Examples

- AI Essentials
- HR Automation
- Finance
- Retail
- Healthcare

---

## MP-005 — Search Results

Purpose

Display Marketplace search results.

Supports

- Semantic Search
- AI Search
- Faceted Filters
- Saved Searches

---

# Asset Screens

---

## MP-010 — Asset Detail

Purpose

Complete information about an asset.

Sections

- Overview
- Gallery
- Features
- Documentation
- Versions
- Reviews
- Dependencies
- Pricing
- Publisher
- Installation

---

## MP-011 — Documentation

Displays

- Guides
- API Reference
- Tutorials
- Examples
- FAQs

---

## MP-012 — Version History

Displays

- Releases
- Changelog
- Compatibility
- Downloads

---

## MP-013 — Reviews

Displays

- Ratings
- Reviews
- Publisher Responses
- AI Summary

---

## MP-014 — Compatibility

Displays

- EVOXA Version
- Required Modules
- Dependencies
- Supported Platforms

---

# Installation Screens

---

## MP-020 — Installation Wizard

Steps

```text
License

↓

Compatibility

↓

Dependencies

↓

Configuration

↓

Installation

↓

Success
```

---

## MP-021 — Dependency Review

Displays

- Required Assets
- Optional Assets
- Missing Components

---

## MP-022 — Configuration Wizard

Allows

- Environment Selection
- Variables
- Credentials
- Policies

---

## MP-023 — Installation Progress

Displays

- Progress
- Logs
- Validation
- Completion

---

## MP-024 — Installation Summary

Displays

- Installed Components
- Version
- Configuration
- Next Steps

---

# Publisher Screens

---

## MP-100 — Publisher Dashboard

Displays

- Downloads
- Revenue
- Reviews
- Active Installations
- Published Assets
- Notifications

---

## MP-101 — Asset Manager

Functions

- Create
- Edit
- Delete
- Version
- Archive

---

## MP-102 — Create Asset

Wizard

```text
Metadata

↓

Upload

↓

Documentation

↓

Pricing

↓

Review

↓

Submit
```

---

## MP-103 — Package Upload

Supports

- Package Validation
- Digital Signature
- Manifest
- Dependency Validation

---

## MP-104 — Documentation Editor

Supports

- Markdown
- Images
- Videos
- API Docs
- AI-generated content

---

## MP-105 — Pricing Configuration

Options

- Free
- Commercial
- Subscription
- Usage-based
- Enterprise

---

## MP-106 — Release Management

Functions

- New Version
- Release Notes
- Rollback
- Compatibility Matrix

---

# AI Marketplace Screens

---

## MP-200 — AI Asset Studio

Supports

- AI Agents
- Skills
- Prompt Packs
- Personas

---

## MP-201 — Prompt Editor

Functions

- Prompt Editing
- Testing
- AI Evaluation
- Versioning

---

## MP-202 — Agent Builder

Displays

- Instructions
- Memory
- Tools
- Policies
- Models

---

## MP-203 — AI Validation

Displays

- Safety
- Quality
- Performance
- Hallucination Score

---

# Commerce Screens

---

## MP-300 — Pricing

Displays

- Plans
- Enterprise License
- Marketplace Credits
- Discounts

---

## MP-301 — Checkout

Supports

- Payment
- Subscription
- Invoice
- License Agreement

---

## MP-302 — License Manager

Displays

- Active Licenses
- Renewals
- Expiration
- Assignments

---

## MP-303 — Purchase History

Displays

- Orders
- Invoices
- Renewals
- Refunds

---

# Administration Screens

---

## MP-400 — Marketplace Dashboard

Displays

- Marketplace Health
- Revenue
- Active Publishers
- Active Customers

---

## MP-401 — Asset Review Queue

Functions

- Review
- Approve
- Reject
- Request Changes

---

## MP-402 — Publisher Management

Displays

- Publishers
- Certifications
- Violations
- Revenue

---

## MP-403 — Category Management

Allows

- Categories
- Collections
- Taxonomy
- Tags

---

## MP-404 — Policy Center

Configure

- Publishing Policies
- Security Rules
- AI Policies
- Marketplace Rules

---

## MP-405 — Certification Center

Displays

- Enterprise Certified
- AI Certified
- Security Certified

---

# Analytics Screens

---

## MP-500 — Marketplace Analytics

Displays

- Downloads
- Revenue
- Growth
- Search Trends
- Conversion

---

## MP-501 — Publisher Analytics

Displays

- Revenue
- Adoption
- Ratings
- Customer Segments

---

## MP-502 — Customer Analytics

Displays

- Installed Assets
- Adoption
- Departments
- Usage

---

## MP-503 — AI Analytics

Displays

- AI Downloads
- Agent Usage
- Prompt Usage
- Token Consumption

---

# Settings Screens

---

## MP-600 — Marketplace Settings

Configure

- Preferences
- Notifications
- Privacy
- Language

---

## MP-601 — Organization Settings

Configure

- Internal Marketplace
- Policies
- Approvals
- Branding

---

## MP-602 — API Tokens

Manage

- Keys
- OAuth
- Webhooks
- Access Tokens

---

# Dialog Catalog

Dialogs include

- Install Asset
- Delete Asset
- Publish Confirmation
- Version Upgrade
- License Acceptance
- Payment Confirmation
- Security Warning
- Dependency Conflict

---

# Notification Center

Displays

- Updates
- Reviews
- Revenue
- Purchases
- Security Alerts
- AI Recommendations

---

# Screen Permissions

| Persona | Accessible Screens |
|-----------|-------------------|
| Consumer | Discovery, Installation |
| Publisher | Publishing, Analytics |
| AI Creator | AI Studio |
| Partner | Partner Center |
| Enterprise Admin | Organization Settings |
| Marketplace Admin | Administration |
| Reviewer | Review Queue |
| Platform Operator | Operations Dashboard |

---

# Repository Structure

```text
08-screen-catalog/
├── discovery/
├── assets/
├── installation/
├── publisher/
├── ai-marketplace/
├── commerce/
├── administration/
├── analytics/
├── settings/
├── dialogs/
├── notifications/
├── assets/
└── metadata.yml
```

---

# Screen Inventory

| Module | Screens |
|----------|---------:|
| Discovery | 10 |
| Asset Management | 12 |
| Installation | 8 |
| Publishing | 15 |
| AI Marketplace | 10 |
| Commerce | 8 |
| Administration | 15 |
| Analytics | 12 |
| Settings | 8 |
| Dialogs | 20 |
| **Total UI Screens** | **118+** |

---

# Standard Visual Artifacts

## Marketplace Navigation

```text
Home

↓

Browse

↓

Asset

↓

Install
```

---

## Publishing Flow

```text
Create

↓

Validate

↓

Publish
```

---

## Administration

```text
Review

↓

Approve

↓

Marketplace
```

---

## Analytics

```text
Events

↓

KPIs

↓

Dashboards
```

---

## Screen Hierarchy

```text
Workspace

↓

Module

↓

Screen

↓

Components
```

---

# Visual Source Files

```text
artifacts/
└── screen-catalog/
    ├── navigation-map.drawio
    ├── discovery.drawio
    ├── publisher-center.drawio
    ├── installation.drawio
    ├── administration.drawio
    ├── analytics.drawio
    ├── screen-hierarchy.drawio
    ├── mermaid/
    │   ├── navigation.mmd
    │   ├── discovery.mmd
    │   ├── publishing.mmd
    │   ├── installation.mmd
    │   ├── administration.mmd
    │   ├── analytics.mmd
    │   └── hierarchy.mmd
    └── exports/
        ├── screen-catalog.svg
        ├── screen-catalog.png
        └── screen-catalog.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Chapter 05 — User Journeys | User Flows |
| Chapter 06 — Information Architecture | Navigation Structure |
| Chapter 07 — UX Goals | UX Principles |
| Chapter 09 — Layout Architecture | Page Layouts |
| Chapter 10 — Navigation Architecture | Navigation Components |
| Chapter 12 — Component Catalog | UI Components |
| Chapter 13 — Widget Catalog | Screen Widgets |
| Chapter 18 — User Stories | Functional Requirements |
| Chapter 19 — API Contracts | Backend Services |

---

# Acceptance Criteria

This chapter is complete when:

- All Marketplace screens are cataloged and uniquely identified.
- Discovery, publishing, installation, AI, commerce, administration, analytics and settings interfaces are documented.
- Screen purposes, primary components, user permissions and interaction flows are defined.
- Repository structure, screen inventory, visual artifacts and traceability are complete.
- The Screen Catalog provides a complete visual inventory that serves as the foundation for detailed UI design, implementation and quality assurance across the EVOXA Marketplace.

---

# Key Takeaways

- The EVOXA Marketplace includes more than 100 specialized screens organized into modular workspaces for consumers, publishers, administrators, partners and AI creators.
- Every interface follows a consistent enterprise design system while providing role-specific functionality and AI-assisted workflows.
- The screen architecture emphasizes modularity, discoverability and governance, enabling the Marketplace to scale as new asset categories and platform capabilities are introduced.
- This catalog serves as the master UI inventory for all Marketplace interfaces and guides the detailed layout, navigation and component specifications in the following chapters.

---

# Next Chapter

**Chapter 09 — Layout Architecture**

The next chapter defines the structural layout of every Marketplace screen, including page composition, responsive grids, workspace organization, content regions and reusable layout patterns used throughout the EVOXA Marketplace.
