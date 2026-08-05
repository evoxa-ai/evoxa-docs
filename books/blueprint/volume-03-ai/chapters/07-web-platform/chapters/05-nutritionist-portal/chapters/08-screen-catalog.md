---
document_id: BP-0003-V3-C07-05-08
chapter_id: CH-05-NUTRITION-08
feature_pack: FP-NUTRITION-0000
title: Screen Catalog
version: 1.0.0
status: Draft
owner: UX Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-04
---

# Chapter 08 — Screen Catalog

> *The Screen Catalog defines the complete inventory of user interface screens within the Nutritionist Portal, including their purpose, navigation, permissions, business responsibilities, AI capabilities and integration points.*

---

# Executive Summary

The Nutritionist Portal consists of a collection of specialized workspaces supporting every stage of the nutritional care lifecycle.

Each screen is designed around professional workflows, minimizing cognitive load while exposing contextual information and AI-powered assistance.

The catalog serves as the authoritative reference for UX, Frontend Engineering, QA and Product teams.

---

# Objectives

The Screen Catalog shall:

- Standardize screen definitions.
- Provide complete UI inventory.
- Improve traceability.
- Support navigation architecture.
- Enable consistent UX.
- Document permissions.
- Facilitate future expansion.

---

# Screen Philosophy

Every screen shall have:

- Unique identifier.
- Business purpose.
- Navigation entry.
- Required permissions.
- Primary workflows.
- AI capabilities.
- Related APIs.
- Associated events.

---

# Screen Classification

Screens are grouped into:

- Global Screens
- Dashboard
- Client Management
- Assessments
- Nutrition Planning
- Food Management
- Progress
- Collaboration
- Reporting
- Administration

---

# Global Screens

| ID | Screen | Purpose |
|----|---------|----------|
| SCR-0001 | Login | User authentication |
| SCR-0002 | MFA Verification | Multi-factor authentication |
| SCR-0003 | Forgot Password | Password recovery |
| SCR-0004 | Notifications Center | Global notifications |
| SCR-0005 | User Profile | Personal settings |

---

# Dashboard

## SCR-0100 — Nutrition Dashboard

Purpose

Central workspace for nutrition professionals.

Displays:

- Daily appointments
- Pending assessments
- AI insights
- Client alerts
- Recent activity
- Performance indicators

Permissions

Nutritionist

---

# Client Management

## SCR-0200 — Client Directory

Displays:

- Client list
- Filters
- Search
- Status
- Assigned nutrition plans

---

## SCR-0201 — Client Profile

Contains:

- Personal information
- Medical history
- Goals
- Dietary restrictions
- Human Digital Twin
- Timeline

---

## SCR-0202 — Client Timeline

Displays:

- Assessments
- Meal plans
- Progress
- Messages
- Appointments
- AI recommendations

---

# Nutritional Assessments

## SCR-0300 — Assessment List

Features:

- Search
- Filters
- Status
- Scheduled assessments

---

## SCR-0301 — New Assessment

Supports:

- Anthropometric measurements
- Dietary assessment
- Clinical indicators
- Lifestyle evaluation

---

## SCR-0302 — Assessment Detail

Displays:

- Results
- AI analysis
- Nutritional diagnosis
- Recommendations
- Attachments

---

# Meal Planning

## SCR-0400 — Meal Plan Library

Displays:

- Existing plans
- Templates
- Favorites
- Organization library

---

## SCR-0401 — Meal Plan Editor

Capabilities:

- Daily meals
- Weekly schedule
- Macro distribution
- Portion control
- Alternatives

AI assists throughout planning.

---

## SCR-0402 — Meal Plan Preview

Displays:

- Nutritional summary
- Macronutrients
- Micronutrients
- Calories
- Export options

---

# Recipes

## SCR-0500 — Recipe Library

Features:

- Categories
- Search
- Nutritional values
- Favorites

---

## SCR-0501 — Recipe Editor

Supports:

- Ingredients
- Preparation
- Nutritional calculation
- Dietary tags

---

# Food Database

## SCR-0600 — Food Catalog

Displays:

- Foods
- Categories
- Nutritional values
- Serving sizes

---

## SCR-0601 — Food Detail

Contains:

- Macronutrients
- Micronutrients
- Glycemic index
- Allergens

---

# Supplements

## SCR-0700 — Supplement Catalog

Displays:

- Products
- Categories
- Contraindications
- Recommendations

---

## SCR-0701 — Supplement Planner

Supports:

- Scheduling
- Dosage
- Duration
- AI recommendations

---

# Shopping Lists

## SCR-0800 — Shopping List

Displays:

- Ingredients
- Quantities
- Categories
- Export
- Share

---

# Progress

## SCR-0900 — Progress Dashboard

Displays:

- Weight trends
- Body composition
- Goal achievement
- Nutritional adherence

---

## SCR-0901 — Measurements History

Contains:

- Historical measurements
- Charts
- Comparative analysis

---

# Reports

## SCR-1000 — Report Center

Supports:

- Progress reports
- Assessment reports
- Nutrition summaries
- Organization reports

---

## SCR-1001 — Report Viewer

Displays:

- Interactive reports
- Export
- Print
- Share

---

# Calendar

## SCR-1100 — Calendar

Supports:

- Daily schedule
- Weekly agenda
- Consultations
- Follow-ups

---

# Messaging

## SCR-1200 — Messages

Features:

- Conversations
- Attachments
- AI summaries
- Notifications

---

# AI Workspace

## SCR-1300 — EVO Assistant

Capabilities:

- Meal recommendations
- Recipe suggestions
- Nutritional analysis
- AI Chat
- Explainability

---

## SCR-1301 — AI Recommendation Review

Displays:

- Recommendation
- Confidence
- Evidence
- Alternatives
- Approval workflow

---

# Administration

## SCR-1400 — Organization Settings

Supports:

- Preferences
- Branding
- AI configuration
- Integrations

---

## SCR-1401 — User Management

Features:

- Users
- Roles
- Permissions
- Activity

---

# Human Digital Twin

## SCR-1500 — Digital Twin Dashboard

Displays:

- Nutritional profile
- Behavior
- Body composition
- Predictions
- AI insights

---

# Screen Relationships

```text
Dashboard

↓

Clients

↓

Assessment

↓

Meal Plan

↓

Progress

↓

Reports
```

---

# Navigation Levels

The portal follows four navigation levels:

1. Global
2. Domain
3. Context
4. Task

---

# Permission Model

Every screen defines:

- View
- Create
- Update
- Delete
- Export
- AI Actions

Permissions are governed by Chapter 23.

---

# Screen Metadata

Every screen includes:

- Screen ID
- Name
- Owner
- Version
- Navigation Path
- Required Role
- APIs
- Events
- Components

---

# AI Touchpoints

AI is integrated into:

- Dashboard
- Assessments
- Meal Planning
- Recipes
- Supplements
- Progress
- Reports
- Human Digital Twin

---

# Screen Metrics

Measured indicators:

- Screen usage
- Task completion
- Navigation efficiency
- AI utilization
- Time on screen
- Error rate

---

# Repository Structure

```text
screens/
├── authentication/
├── dashboard/
├── clients/
├── assessments/
├── meal-plans/
├── recipes/
├── foods/
├── supplements/
├── shopping/
├── progress/
├── reports/
├── calendar/
├── messaging/
├── ai/
├── administration/
├── digital-twin/
└── metadata.yml
```

---

# Standard Visual Artifacts

## Screen Map

```text
Dashboard

↓

Clients

↓

Assessment

↓

Meal Plans

↓

Progress
```

---

## Navigation Tree

```text
Portal

↓

Domain

↓

Screen

↓

Action
```

---

## Client Workspace

```text
Client

↓

Assessment

↓

Plan

↓

Monitoring
```

---

## AI Integration

```text
Screen

↓

AI

↓

Recommendation
```

---

# Visual Source Files

```text
artifacts/
└── screen-catalog/
    ├── screen-map.drawio
    ├── navigation-tree.drawio
    ├── workspace.drawio
    ├── ai-integration.drawio
    ├── permissions.drawio
    ├── screen-matrix.drawio
    ├── mermaid/
    │   ├── screens.mmd
    │   ├── navigation.mmd
    │   ├── workspace.mmd
    │   ├── ai.mmd
    │   └── permissions.mmd
    └── exports/
        ├── screen-catalog.svg
        ├── screen-catalog.png
        └── screen-catalog.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Information Architecture | Chapter 06 |
| UX Goals | Chapter 07 |
| Layout Architecture | Chapter 09 |
| Navigation Architecture | Chapter 10 |
| Component Catalog | Chapter 12 |
| Workflow Architecture | Chapter 16 |
| Security & Permissions | Chapter 23 |

---

# Acceptance Criteria

This chapter is complete when:

- All portal screens are cataloged.
- Screen identifiers are unique.
- Navigation hierarchy is documented.
- Permissions are defined.
- AI touchpoints are identified.
- Metadata standards are established.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- The Screen Catalog provides the authoritative inventory of every user interface within the Nutritionist Portal.
- Each screen is uniquely identified and linked to business capabilities, navigation, permissions, APIs, events and AI services.
- A structured screen taxonomy improves consistency, maintainability and cross-team collaboration throughout the EVOXA platform.
- Comprehensive traceability ensures that every screen can be connected to user journeys, workflows, security policies and future product evolution.

---

# Next Chapter

**Chapter 09 — Layout Architecture**

This chapter defines the structural layout of the Nutritionist Portal, including page templates, workspace organization, responsive behavior, layout regions, adaptive interfaces and interaction patterns that support professional nutritional workflows.
