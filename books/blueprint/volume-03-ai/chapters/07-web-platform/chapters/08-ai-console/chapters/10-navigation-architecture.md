---
document_id: BP-0003-V3-C07-08-10
chapter_id: CH-08-AI-10
feature_pack: FP-AI-0000
title: Navigation Architecture
version: 1.0.0
status: Draft
owner: UX Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-05
---

# Chapter 10 — Navigation Architecture

> *The Navigation Architecture chapter defines the navigation model, routing strategy, workspace transitions, contextual navigation and interaction patterns used throughout the EVOXA AI Console.*

---

# Executive Summary

The EVOXA AI Console uses a multi-layer navigation architecture designed to support complex AI workflows while minimizing cognitive load and context switching.

Navigation combines global access, workspace navigation, contextual actions, AI asset relationships and search-driven discovery.

Rather than navigating isolated screens, users move through interconnected AI assets while preserving context.

---

# Objectives

The Navigation Architecture shall:

- Simplify navigation.
- Reduce context switching.
- Improve discoverability.
- Preserve workflow continuity.
- Support scalable expansion.
- Enable fast asset access.
- Provide predictable interactions.

---

# Navigation Principles

The platform follows:

- Workspace First
- Asset-Centric Navigation
- Context Preservation
- Progressive Disclosure
- Search First
- Keyboard Accessible
- Predictable Navigation
- AI-Assisted Navigation

---

# Navigation Layers

The navigation architecture consists of:

- Global Navigation
- Workspace Navigation
- Asset Navigation
- Context Navigation
- Search Navigation
- Command Navigation
- Relationship Navigation

---

# Global Navigation

Accessible from every screen.

Provides access to:

- Dashboard
- Foundation
- Prompt Engineering
- Agents
- Knowledge
- MCP
- Workflows
- Operations
- Governance
- Marketplace
- Administration

The Global Navigation remains persistent.

---

# Workspace Navigation

Each workspace contains dedicated navigation.

Example:

```text
Prompt Workspace

↓

Prompt Studio

↓

Registry

↓

Playground

↓

Evaluations

↓

Versions
```

---

# Asset Navigation

Every AI asset exposes related resources.

Example:

```text
Model

↓

Prompts

↓

Agents

↓

Deployments

↓

Monitoring
```

Users navigate relationships directly.

---

# Context Navigation

Context includes:

- Active Tenant
- Environment
- Workspace
- Asset
- Version
- Lifecycle Stage

Navigation preserves context across transitions.

---

# Breadcrumb Navigation

Example:

```text
Dashboard

>

Models

>

GPT-4 Enterprise

>

Prompt Library

>

Prompt v3
```

Breadcrumbs provide hierarchical navigation and quick return paths.

---

# Search Navigation

Global Search supports:

- Models
- Prompts
- Agents
- Knowledge Bases
- Documents
- Tools
- Workflows
- Policies
- Deployments
- Users

Search supports:

- Full-text
- Semantic search
- Tags
- Metadata
- Filters
- Saved searches

---

# Command Palette

Keyboard shortcut:

```text
Ctrl + K

or

⌘ + K
```

Supports:

- Open screens
- Search assets
- Execute commands
- Create resources
- Run workflows
- Switch workspaces
- Navigate history

---

# Relationship Navigation

Assets expose graph-based navigation.

Example:

```text
Provider

↓

Model

↓

Prompt

↓

Agent

↓

Workflow

↓

Deployment

↓

Application
```

Relationships are navigable in both directions.

---

# Tab Navigation

Supports:

- Multiple open assets
- Pin tabs
- Reorder tabs
- Close inactive tabs
- Restore previous session

---

# Secondary Navigation

Within workspaces, secondary navigation provides:

- Categories
- Filters
- Collections
- Favorites
- Recent Items

---

# Favorites

Users can bookmark:

- Models
- Prompts
- Agents
- Workflows
- Knowledge Bases
- Dashboards

Favorites synchronize across sessions.

---

# Recent Items

Automatically displays:

- Recently opened assets
- Recent deployments
- Recent prompts
- Recent agents
- Recent searches

---

# Navigation History

Supports:

- Back
- Forward
- Recently visited assets
- Recently executed actions

History is workspace-aware.

---

# Deep Linking

Every resource has a unique URL.

Example:

```text
/workspaces/prompts/{promptId}

/models/{modelId}

/agents/{agentId}

/knowledge/{knowledgeBaseId}
```

Deep links preserve navigation context and permissions.

---

# Notifications Navigation

Notifications include contextual links to:

- Assets
- Incidents
- Evaluations
- Policies
- Deployments

Selecting a notification opens the related resource directly.

---

# AI-Assisted Navigation

Embedded AI assists users by:

- Recommending destinations
- Suggesting related assets
- Highlighting dependencies
- Explaining navigation paths
- Predicting next actions

---

# Navigation States

Supported states include:

- Default
- Expanded
- Collapsed
- Focus Mode
- Full Screen
- Split View

Navigation adapts without losing user context.

---

# Routing Strategy

Routing follows:

```text
Workspace

↓

Category

↓

Asset

↓

Action
```

Example:

```text
/agents/catalog

/prompts/playground

/models/router

/workflows/designer
```

---

# Navigation Permissions

Navigation visibility depends on:

- User Role
- Tenant
- Environment
- Feature Flags
- AI Policies
- Licensing

Unauthorized destinations are hidden or disabled.

---

# Navigation Analytics

Metrics include:

- Screen visits
- Navigation paths
- Search usage
- Task completion
- Abandonment rate
- Most accessed assets
- Command palette usage

---

# Performance Targets

| Navigation Action | Target |
|-------------------|--------|
| Route Change | <100 ms |
| Workspace Switch | <200 ms |
| Asset Load | <500 ms |
| Search Result | <500 ms |
| Command Palette | <100 ms |

---

# Repository Structure

```text
navigation-architecture/
├── global-navigation/
├── workspace-navigation/
├── asset-navigation/
├── routing/
├── search/
├── command-palette/
├── favorites/
├── history/
├── permissions/
└── metadata.yml
```

---

# Standard Visual Artifacts

## Global Navigation

```text
Header

↓

Sidebar

↓

Workspace

↓

Asset
```

---

## Navigation Hierarchy

```text
Workspace

↓

Category

↓

Asset

↓

Action
```

---

## Relationship Navigation

```text
Model

↓

Prompt

↓

Agent

↓

Workflow
```

---

## Routing Strategy

```text
URL

↓

Workspace

↓

Screen

↓

Context
```

---

# Visual Source Files

```text
artifacts/
└── navigation-architecture/
    ├── global-navigation.drawio
    ├── workspace-navigation.drawio
    ├── routing-model.drawio
    ├── asset-graph.drawio
    ├── command-palette.drawio
    ├── search-flow.drawio
    ├── mermaid/
    │   ├── navigation.mmd
    │   ├── routing.mmd
    │   ├── hierarchy.mmd
    │   ├── search.mmd
    │   └── graph.mmd
    └── exports/
        ├── navigation-architecture.svg
        ├── navigation-architecture.png
        └── navigation-architecture.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Information Architecture | Chapter 06 |
| UX Goals | Chapter 07 |
| Screen Catalog | Chapter 08 |
| Layout Architecture | Chapter 09 |
| Component Catalog | Chapter 12 |
| Workflow Architecture | Chapter 16 |

---

# Acceptance Criteria

This chapter is complete when:

- Navigation layers and routing strategy are documented.
- Global, workspace and asset navigation models are defined.
- Search, command palette and relationship navigation are specified.
- Permission-aware navigation and deep linking are documented.
- Performance targets, repository structure, visual artifacts and traceability are complete.

---

# Key Takeaways

- The EVOXA AI Console uses a multi-layer navigation model that combines global navigation, workspaces, AI asset relationships and intelligent search.
- Context-aware routing, persistent workspace state and graph-based navigation reduce cognitive load and improve productivity during complex AI workflows.
- Deep linking, keyboard-first navigation and AI-assisted recommendations enable fast access to resources while preserving operational context.
- A scalable navigation architecture allows the AI Console to evolve with new AI capabilities without disrupting the user experience.

---

# Next Chapter

**Chapter 11 — Design Principles**

This chapter defines the visual language, interaction principles, usability standards and design philosophy that guide the implementation of every interface, component and workflow within the EVOXA AI Console.
