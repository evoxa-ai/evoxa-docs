---
document_id: BP-0003-V3-C07-08-08
chapter_id: CH-08-AI-08
feature_pack: FP-AI-0000
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

> *The Screen Catalog chapter defines every workspace, screen and interface available within the EVOXA AI Console, including navigation relationships, responsibilities, supported interactions and primary UI components.*

---

# Executive Summary

The AI Console is organized into specialized workspaces that manage the complete lifecycle of enterprise Artificial Intelligence.

Each workspace groups related capabilities while maintaining consistent navigation, shared interaction patterns and a unified design language.

The catalog serves as the authoritative inventory of all user-facing interfaces within the AI Console.

---

# Objectives

The Screen Catalog shall:

- Standardize screen organization.
- Define workspace responsibilities.
- Improve discoverability.
- Support consistent navigation.
- Enable modular expansion.
- Simplify UX governance.
- Provide implementation guidance.

---

# Screen Organization

The AI Console is divided into:

```text
Dashboard

Foundation

Engineering

Knowledge

Operations

Governance

Administration
```

---

# Dashboard Workspace

---

## AI Executive Dashboard

Purpose

Provide an overview of the AI ecosystem.

Displays

- Platform Health
- Active Models
- Running Agents
- AI Requests
- Costs
- Alerts
- Deployments
- Usage Trends

Components

- KPI Cards
- Charts
- Timeline
- Activity Feed
- Alert Panel

---

## AI Operations Dashboard

Displays

- Live Metrics
- Active Workflows
- Agent Status
- AI Queue
- Error Rate
- Provider Health

---

## AI Cost Dashboard

Displays

- Token Usage
- Cost by Provider
- Cost by Tenant
- Daily Consumption
- Monthly Forecast

---

# Foundation Workspace

---

## AI Providers

Purpose

Manage AI providers.

Functions

- Add Provider
- Configure Credentials
- Health Checks
- Billing
- Regions

---

## Model Registry

Displays

- Models
- Versions
- Providers
- Capabilities
- Status

---

## AI Gateway

Displays

- Routes
- Policies
- Quotas
- Requests
- Errors

---

## Model Router

Displays

- Routing Rules
- Fallback Models
- Cost Optimization
- Latency

---

# Prompt Workspace

---

## Prompt Studio

Functions

- Create Prompt
- Edit Prompt
- Variables
- Templates

---

## Prompt Registry

Displays

- Prompt Library
- Owners
- Tags
- Versions

---

## Prompt Playground

Functions

- Live Testing
- Compare Outputs
- Multiple Models
- Streaming

---

## Prompt Evaluation

Displays

- Scores
- Hallucinations
- Quality Metrics
- Benchmarks

---

## Prompt Versions

Displays

- Version History
- Differences
- Rollback

---

# Agent Workspace

---

## Agent Catalog

Displays

- Available Agents
- Status
- Owners
- Usage

---

## Agent Builder

Functions

- Instructions
- Tools
- Memory
- Models
- Goals

---

## Agent Playground

Functions

- Test Agent
- Debug
- Inspect Memory
- Inspect Tools

---

## Multi-Agent Studio

Displays

- Agent Collaboration
- Communication
- Task Routing

---

## Agent Memory

Displays

- Memory Types
- Context
- Sessions
- History

---

# Knowledge Workspace

---

## Knowledge Bases

Displays

- Available Bases
- Owners
- Documents

---

## Documents

Functions

- Upload
- Index
- Organize
- Version

---

## Chunk Manager

Displays

- Chunks
- Quality
- Metadata

---

## Embeddings

Displays

- Embedding Models
- Dimensions
- Statistics

---

## Vector Database

Displays

- Collections
- Indexes
- Search Performance

---

## Retrieval Studio

Functions

- Test Retrieval
- Ranking
- Hybrid Search
- Semantic Search

---

# MCP Workspace

---

## MCP Servers

Displays

- Registered Servers
- Status
- Transport

---

## MCP Clients

Displays

- Active Clients
- Connections

---

## Tool Registry

Displays

- Tools
- Permissions
- Categories

---

## Resource Registry

Displays

- Resources
- Metadata
- Ownership

---

# Workflow Workspace

---

## Workflow Designer

Functions

- Build Workflow
- Connect Nodes
- Validate

---

## Workflow Executions

Displays

- Running Jobs
- History
- Errors

---

## Scheduler

Displays

- Scheduled Tasks
- Triggers

---

# Operations Workspace

---

## AI Monitoring

Displays

- Availability
- Requests
- Latency
- Errors

---

## AI Observability

Displays

- Traces
- Metrics
- Logs
- Events

---

## AI Analytics

Displays

- Adoption
- Quality
- Usage
- Trends

---

## Incident Center

Displays

- Active Incidents
- Alerts
- Timeline

---

# Governance Workspace

---

## Policies

Displays

- AI Policies
- Status
- Scope

---

## Guardrails

Displays

- Prompt Filters
- Safety Rules
- Output Policies

---

## Compliance Center

Displays

- Regulations
- Compliance Reports
- Reviews

---

## Audit Center

Displays

- AI Audit Logs
- Events
- Changes

---

# Marketplace Workspace

---

## Marketplace Home

Displays

- Featured Assets
- New Assets
- Popular Assets

---

## Agent Marketplace

Displays

- Shared Agents
- Ratings
- Versions

---

## Prompt Marketplace

Displays

- Templates
- Packs
- Downloads

---

## Connector Marketplace

Displays

- MCP
- APIs
- Integrations

---

# Administration Workspace

---

## Users

## Roles

## Permissions

## API Keys

## Secrets

## Tenant Settings

## Environment Settings

---

# Global Screens

Available everywhere

- Notifications
- Command Palette
- Global Search
- Activity Center
- Help Center
- Profile
- Settings

---

# Navigation Relationships

```text
Dashboard

↓

Models

↓

Prompts

↓

Agents

↓

Knowledge

↓

Workflows

↓

Monitoring

↓

Governance
```

---

# Screen Metadata

Every screen contains

- Screen ID
- Route
- Workspace
- Required Permission
- Owner
- Version
- Status

---

# Repository Structure

```text
screen-catalog/
├── dashboard/
├── foundation/
├── prompts/
├── agents/
├── knowledge/
├── workflows/
├── operations/
├── governance/
├── marketplace/
├── administration/
└── metadata.yml
```

---

# Standard Visual Artifacts

## Screen Hierarchy

```text
Dashboard

↓

Workspace

↓

Screen

↓

Components
```

---

## Navigation Map

```text
Foundation

↓

Engineering

↓

Operations
```

---

## Workspace Model

```text
Workspace

↓

Screen

↓

Panels

↓

Widgets
```

---

## Screen Relationships

```text
Models

↓

Prompts

↓

Agents

↓

Knowledge

↓

Deployment
```

---

# Visual Source Files

```text
artifacts/
└── screen-catalog/
    ├── workspace-map.drawio
    ├── navigation-map.drawio
    ├── screen-hierarchy.drawio
    ├── ai-workspaces.drawio
    ├── dashboard-layout.drawio
    ├── screen-relationships.drawio
    ├── mermaid/
    │   ├── navigation.mmd
    │   ├── workspaces.mmd
    │   ├── hierarchy.mmd
    │   ├── screens.mmd
    │   └── relationships.mmd
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

---

# Acceptance Criteria

This chapter is complete when:

- Every workspace and screen is cataloged.
- Screen purposes, responsibilities and primary components are documented.
- Navigation relationships between workspaces are defined.
- Global screens and shared interactions are standardized.
- Repository structure, visual artifacts and traceability are complete.

---

# Key Takeaways

- The EVOXA AI Console organizes its user interface into specialized workspaces that cover the entire enterprise AI lifecycle.
- Each screen has a clearly defined purpose, standardized metadata and consistent interaction patterns.
- Shared navigation, reusable layouts and common UI components create a coherent experience across all AI domains.
- The Screen Catalog provides the foundation for interface implementation, UX governance and future expansion of the AI Console.

---

# Next Chapter

**Chapter 09 — Layout Architecture**

This chapter defines the structural layout of every AI Console workspace, including page templates, panel organization, responsive regions, navigation containers and reusable layout patterns that ensure consistency across all interfaces.
