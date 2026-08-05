---
document_id: BP-0003-V3-C07-05-06
chapter_id: CH-05-NUTRITION-06
feature_pack: FP-NUTRITION-0000
title: Information Architecture
version: 1.0.0
status: Draft
owner: UX Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-04
---

# Chapter 06 — Information Architecture

> *The Information Architecture chapter defines how information is organized, classified, connected and presented throughout the Nutritionist Portal, enabling intuitive navigation, efficient workflows and AI-assisted decision making.*

---

# Executive Summary

The Nutritionist Portal organizes information according to professional nutritional workflows rather than technical system modules.

Information is grouped into business domains, allowing nutrition professionals to navigate naturally between clients, assessments, meal plans, progress and collaboration.

The architecture supports scalability, discoverability and contextual AI assistance.

---

# Objectives

The Information Architecture shall:

- Organize information logically.
- Reduce navigation effort.
- Improve information discoverability.
- Support multidisciplinary workflows.
- Enable AI contextualization.
- Scale with future modules.
- Maintain consistency across EVOXA.

---

# Information Philosophy

Information is organized around the client lifecycle.

Every piece of information belongs to a business domain.

Every domain has:

- Ownership
- Navigation
- Permissions
- Metadata
- Relationships
- Searchability

---

# Information Domains

The Nutritionist Portal is composed of the following domains:

- Dashboard
- Clients
- Assessments
- Meal Plans
- Recipes
- Food Database
- Supplements
- Shopping Lists
- Progress
- Reports
- Calendar
- Messaging
- Notifications
- AI Workspace
- Administration

---

# Domain Hierarchy

```text
Nutritionist Portal

├── Dashboard
├── Clients
│   ├── Profile
│   ├── Medical History
│   ├── Goals
│   ├── Assessments
│   ├── Meal Plans
│   ├── Progress
│   ├── Messages
│   └── Human Digital Twin
├── Recipes
├── Food Database
├── Supplements
├── Reports
├── Calendar
├── AI Workspace
└── Settings
```

---

# Navigation Model

The portal follows a hybrid navigation model:

- Global Navigation
- Context Navigation
- Client Navigation
- Workflow Navigation
- Search Navigation

Navigation adapts according to user context.

---

# Dashboard

Provides a consolidated overview including:

- Daily appointments
- Pending assessments
- Client alerts
- AI recommendations
- Recent activity
- Nutritional statistics

---

# Client Workspace

Each client acts as an information hub.

Client workspace includes:

- Personal profile
- Nutritional history
- Body composition
- Assessments
- Meal plans
- Supplements
- Reports
- Communication
- Digital Twin
- Timeline

---

# Assessment Domain

Contains:

- Anthropometric data
- Clinical indicators
- Dietary history
- Laboratory values
- Lifestyle information
- Nutritional diagnosis

Historical assessments remain immutable after approval.

---

# Meal Planning Domain

Stores:

- Meal plans
- Daily menus
- Weekly schedules
- Macronutrient targets
- Micronutrient analysis
- Alternatives

Plans maintain complete version history.

---

# Recipe Domain

Organizes recipes by:

- Meal type
- Dietary pattern
- Calories
- Macronutrients
- Allergens
- Cuisine
- Preparation time

Recipes are reusable across meal plans.

---

# Food Database

Contains:

- Foods
- Nutritional values
- Serving sizes
- Food groups
- Micronutrients
- Glycemic index
- Dietary tags

---

# Progress Domain

Tracks:

- Weight history
- Body composition
- Measurements
- Goal achievement
- Adherence
- Behavioral trends
- AI predictions

---

# AI Workspace

Provides:

- AI chat
- Meal recommendations
- Recipe generation
- Nutritional analysis
- Deficiency detection
- Report summaries
- Decision support

---

# Search Architecture

Global search indexes:

- Clients
- Meal plans
- Assessments
- Recipes
- Foods
- Reports
- Messages

Supports:

- Full-text search
- Semantic search
- AI search
- Filters
- Saved searches

---

# Metadata Model

Every object includes:

- Identifier
- Owner
- Creation date
- Last update
- Version
- Tags
- Status
- Permissions

---

# Taxonomy

Primary taxonomy includes:

- Nutrition
- Assessment
- Meals
- Foods
- Supplements
- Progress
- Communication
- AI

Secondary classifications:

- Clinical
- Sports
- Wellness
- Pediatric
- Corporate

---

# Human Digital Twin Information Model

```text
Identity

↓

Nutrition

↓

Body Composition

↓

Behavior

↓

Recovery

↓

Predictions
```

The Digital Twin aggregates information from every domain.

---

# Relationships

```text
Client

↓

Assessment

↓

Meal Plan

↓

Progress

↓

Reports

↓

AI
```

Every relationship is bidirectional where appropriate.

---

# Cross-Portal Information

Information shared with:

- User Portal
- Coach Portal
- Enterprise Portal
- AI Platform
- Analytics Platform

Data sharing follows permission and consent policies.

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

# Scalability

The architecture supports future domains including:

- Wearables
- Genomics
- Continuous glucose monitoring
- Population health
- Marketplace
- Clinical protocols

---

# Standard Visual Artifacts

## Domain Map

```text
Clients

↓

Assessments

↓

Meal Plans

↓

Progress

↓

Reports
```

---

## Information Hierarchy

```text
Portal

↓

Domain

↓

Module

↓

Entity
```

---

## Client Information Hub

```text
Client

↓

History

↓

Plans

↓

Progress

↓

Digital Twin
```

---

## Search Architecture

```text
Search

↓

Index

↓

Results

↓

AI Ranking
```

---

# Visual Source Files

```text
artifacts/
└── information-architecture/
    ├── domain-map.drawio
    ├── hierarchy.drawio
    ├── client-workspace.drawio
    ├── search.drawio
    ├── taxonomy.drawio
    ├── relationships.drawio
    ├── mermaid/
    │   ├── hierarchy.mmd
    │   ├── domains.mmd
    │   ├── relationships.mmd
    │   ├── search.mmd
    │   └── digital-twin.mmd
    └── exports/
        ├── information-architecture.svg
        ├── information-architecture.png
        └── information-architecture.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Personas | Chapter 04 |
| User Journeys | Chapter 05 |
| UX Goals | Chapter 07 |
| Screen Catalog | Chapter 08 |
| Navigation Architecture | Chapter 10 |
| Workflow Architecture | Chapter 16 |
| Data Model | Chapter 22 |

---

# Acceptance Criteria

This chapter is complete when:

- Information domains are defined.
- Navigation hierarchy is documented.
- Client workspace organization is specified.
- Metadata and taxonomy models are established.
- Search architecture is documented.
- Human Digital Twin information model is defined.
- Cross-portal information relationships are specified.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- The Nutritionist Portal organizes information around professional nutritional workflows rather than technical modules, enabling faster and more intuitive navigation.
- A domain-driven information architecture, combined with standardized metadata and taxonomy, improves discoverability, consistency and long-term scalability.
- The Client Workspace acts as the central information hub, integrating assessments, meal plans, progress, communication and the Human Digital Twin into a unified professional view.
- Shared information models and AI-aware search capabilities ensure seamless collaboration across the EVOXA ecosystem while preserving governance, permissions and data integrity.

---

# Next Chapter

**Chapter 07 — UX Goals**

This chapter defines the user experience objectives of the Nutritionist Portal, including usability principles, interaction goals, cognitive load reduction, AI-assisted experiences and measurable UX success criteria.
