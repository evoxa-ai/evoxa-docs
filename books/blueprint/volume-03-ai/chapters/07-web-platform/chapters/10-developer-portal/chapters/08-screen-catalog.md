---
document_id: BP-0003-V3-C10-08
chapter_id: CH-10-DEV-08
feature_pack: FP-DEV-0000
title: Screen Catalog
version: 1.0.0
status: Draft
owner: UX Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-05
---

# Chapter 08 — Screen Catalog

> *The Screen Catalog defines every page, screen, workspace and interactive view that composes the EVOXA Developer Portal, establishing the visual foundation for frontend implementation, navigation, usability and developer workflows.*

---

# Executive Summary

The EVOXA Developer Portal is much more than API documentation.

It is a complete Developer Experience (DX) platform where developers build applications, configure AI, manage organizations, publish marketplace assets and monitor production systems.

The portal contains approximately **120–150 primary screens** organized into logical workspaces that support the complete software development lifecycle.

---

# Objectives

The Screen Catalog shall:

- Standardize every interface.
- Define navigation consistency.
- Support scalable frontend development.
- Simplify design collaboration.
- Improve discoverability.
- Accelerate implementation.
- Enable future expansion.

---

# Screen Organization

The platform is divided into the following workspaces:

```text
Developer Portal

├── Dashboard
├── Documentation
├── APIs
├── SDKs
├── AI Studio
├── Workflow Studio
├── Agent Studio
├── Marketplace
├── Billing
├── Administration
└── Settings
```

---

# Workspace 01 — Dashboard

Purpose

Provide a personalized overview of developer activity.

---

## Screens

### DEV-001

Developer Dashboard

Displays:

- Recent Activity
- API Usage
- Active Projects
- AI Consumption
- Notifications
- Billing Summary

---

### DEV-002

Developer Home

Quick links

Recent projects

Learning resources

Latest releases

---

### DEV-003

Activity Timeline

Developer events

Deployments

API calls

Marketplace publications

---

# Workspace 02 — Documentation

---

### DEV-100

Documentation Home

---

### DEV-101

Getting Started

---

### DEV-102

Tutorial Catalog

---

### DEV-103

Concept Guides

---

### DEV-104

Architecture Guides

---

### DEV-105

Best Practices

---

### DEV-106

Release Notes

---

### DEV-107

Search Results

---

### DEV-108

Documentation Viewer

Interactive documentation with:

- TOC
- Code examples
- AI Assistant
- Related APIs

---

# Workspace 03 — API Platform

---

### DEV-200

API Catalog

---

### DEV-201

API Explorer

---

### DEV-202

REST API Reference

---

### DEV-203

GraphQL Explorer

---

### DEV-204

OpenAPI Viewer

---

### DEV-205

Authentication Guide

---

### DEV-206

Rate Limits

---

### DEV-207

API Changelog

---

### DEV-208

API Playground

Supports:

- Live requests
- Authentication
- SDK snippets
- Response inspection

---

# Workspace 04 — SDK Center

---

### DEV-300

SDK Catalog

---

### DEV-301

SDK Downloads

---

### DEV-302

SDK Documentation

---

### DEV-303

SDK Examples

---

### DEV-304

Package Registry

---

### DEV-305

CLI Documentation

---

### DEV-306

CLI Playground

---

# Workspace 05 — AI Studio

---

### DEV-400

AI Dashboard

---

### DEV-401

Prompt Studio

---

### DEV-402

Prompt Playground

---

### DEV-403

Model Comparison

---

### DEV-404

AI Evaluations

---

### DEV-405

Token Usage

---

### DEV-406

Cost Analysis

---

### DEV-407

AI Logs

---

### DEV-408

Model Registry

---

# Workspace 06 — Agent Studio

---

### DEV-500

Agent Catalog

---

### DEV-501

Agent Builder

---

### DEV-502

Conversation Tester

---

### DEV-503

Memory Configuration

---

### DEV-504

Tools Manager

---

### DEV-505

MCP Server Connections

---

### DEV-506

Knowledge Sources

---

### DEV-507

Agent Analytics

---

# Workspace 07 — Workflow Studio

---

### DEV-600

Workflow Dashboard

---

### DEV-601

Visual Workflow Builder

---

### DEV-602

Node Library

---

### DEV-603

Trigger Configuration

---

### DEV-604

Execution History

---

### DEV-605

Workflow Logs

---

### DEV-606

Workflow Analytics

---

### DEV-607

Workflow Templates

---

# Workspace 08 — Events

---

### DEV-700

Event Catalog

---

### DEV-701

Schema Viewer

---

### DEV-702

Webhook Manager

---

### DEV-703

Webhook Deliveries

---

### DEV-704

Event Replay

---

### DEV-705

Subscription Manager

---

# Workspace 09 — Marketplace

---

### DEV-800

Marketplace Home

---

### DEV-801

Extensions

---

### DEV-802

AI Agents

---

### DEV-803

Prompt Library

---

### DEV-804

Workflow Templates

---

### DEV-805

Connector Catalog

---

### DEV-806

Publish Extension

---

### DEV-807

Marketplace Analytics

---

# Workspace 10 — Organizations

---

### DEV-900

Organization Dashboard

---

### DEV-901

Members

---

### DEV-902

Roles

---

### DEV-903

Projects

---

### DEV-904

Applications

---

### DEV-905

API Keys

---

### DEV-906

OAuth Clients

---

### DEV-907

Audit Logs

---

# Workspace 11 — Billing

---

### DEV-1000

Billing Dashboard

---

### DEV-1001

Usage

---

### DEV-1002

Invoices

---

### DEV-1003

Plans

---

### DEV-1004

Credits

---

### DEV-1005

Cost Explorer

---

# Workspace 12 — Analytics

---

### DEV-1100

Developer Analytics

---

### DEV-1101

API Analytics

---

### DEV-1102

AI Analytics

---

### DEV-1103

Marketplace Analytics

---

### DEV-1104

SDK Analytics

---

### DEV-1105

Organization Analytics

---

# Workspace 13 — Administration

---

### DEV-1200

Administration Dashboard

---

### DEV-1201

Users

---

### DEV-1202

Organizations

---

### DEV-1203

Permissions

---

### DEV-1204

Feature Flags

---

### DEV-1205

Platform Configuration

---

### DEV-1206

System Health

---

# Workspace 14 — Settings

---

### DEV-1300

Profile

---

### DEV-1301

Preferences

---

### DEV-1302

Notifications

---

### DEV-1303

Security

---

### DEV-1304

API Tokens

---

### DEV-1305

Connected Accounts

---

### DEV-1306

Developer Preferences

---

# Global Components

Every screen includes:

- Global Header
- Workspace Navigation
- Breadcrumbs
- Global Search
- AI Copilot
- Notifications
- User Menu
- Theme Selector

---

# Responsive Behavior

Layouts support:

Desktop

- Full Workspace

Tablet

- Adaptive Navigation

Mobile

- Documentation
- Monitoring
- Notifications
- Billing
- Profile

---

# Navigation Relationships

```text
Dashboard

↓

Projects

↓

Applications

↓

APIs

↓

AI

↓

Deployments
```

---

# Screen Inventory

| Workspace | Approx. Screens |
|------------|----------------:|
| Dashboard | 3 |
| Documentation | 9 |
| APIs | 9 |
| SDK Center | 7 |
| AI Studio | 9 |
| Agent Studio | 8 |
| Workflow Studio | 8 |
| Events | 6 |
| Marketplace | 8 |
| Organizations | 8 |
| Billing | 6 |
| Analytics | 6 |
| Administration | 7 |
| Settings | 7 |

**Estimated Total:** **101+ primary screens**

---

# Naming Convention

Every screen follows:

```text
DEV-XXXX

Workspace

↓

Screen
```

Example:

```text
DEV-501

Agent Builder
```

---

# Repository Structure

```text
screen-catalog/
├── dashboard/
├── documentation/
├── apis/
├── sdk/
├── ai/
├── agents/
├── workflows/
├── marketplace/
├── organizations/
├── billing/
├── analytics/
├── administration/
├── settings/
└── metadata.yml
```

---

# Standard Visual Artifacts

## Screen Hierarchy

```text
Portal

↓

Workspace

↓

Screen

↓

Component
```

---

## Navigation Map

```text
Dashboard

↓

Applications

↓

API Explorer

↓

Deploy
```

---

## Workspace Relationships

```text
Documentation

↓

API

↓

SDK

↓

AI

↓

Marketplace
```

---

## Developer Workspace

```text
Projects

↓

Applications

↓

Credentials

↓

Production
```

---

# Visual Source Files

```text
artifacts/
└── screen-catalog/
    ├── screen-map.drawio
    ├── workspace-navigation.drawio
    ├── developer-dashboard.drawio
    ├── ai-studio.drawio
    ├── marketplace.drawio
    ├── application-flow.drawio
    ├── mermaid/
    │   ├── screens.mmd
    │   ├── navigation.mmd
    │   ├── workspaces.mmd
    │   ├── dashboard.mmd
    │   └── ai-studio.mmd
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
| Responsive Design | Chapter 15 |

---

# Acceptance Criteria

This chapter is complete when:

- Every major workspace is identified.
- All primary screens are cataloged and uniquely identified.
- Navigation relationships between workspaces are documented.
- Responsive behavior and global UI components are specified.
- Repository structure, visual artifacts and traceability are complete.

---

# Key Takeaways

- The EVOXA Developer Portal is organized into fourteen functional workspaces that collectively support the complete software development lifecycle.
- More than one hundred primary screens provide dedicated interfaces for APIs, AI, SDKs, workflows, organizations, billing, analytics and marketplace operations.
- A consistent naming convention, navigation hierarchy and responsive design ensure scalability as new capabilities are introduced.
- The Screen Catalog serves as the master reference for UX design, frontend implementation and future platform expansion.

---

# Next Chapter

**Chapter 09 — Layout Architecture**

This chapter defines the structural layout of every workspace, including page composition, responsive grids, navigation containers, content regions and reusable layout patterns that establish a consistent visual framework across the EVOXA Developer Portal.
