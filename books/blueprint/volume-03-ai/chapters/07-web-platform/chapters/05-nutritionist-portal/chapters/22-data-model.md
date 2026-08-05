---
document_id: BP-0003-V3-C07-04-22
chapter_id: CH-04-COACH-22
feature_pack: FP-COACH-0000
title: Data Model
version: 1.0.0
status: Draft
owner: Data Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-04
---

# Chapter 22 — Data Model

> *The Data Model defines the logical, conceptual and physical data architecture of the Coach Portal, including business entities, aggregates, domain ownership, Human Digital Twin structures, AI data requirements and persistence strategies across the EVOXA platform.*

---

# Executive Summary

The Coach Portal manages highly interconnected operational, analytical and AI-driven information.

Rather than relying on a single relational model, EVOXA adopts a polyglot persistence architecture in which different storage technologies are selected according to workload characteristics.

The data model follows Domain-Driven Design (DDD), ensuring clear ownership, scalability and long-term maintainability.

---

# Objectives

The Data Model shall:

- Standardize business entities.
- Support Human Digital Twins.
- Enable AI services.
- Preserve data integrity.
- Support analytics.
- Scale horizontally.
- Enable event sourcing.

---

# Data Philosophy

Data is organized around business domains.

Every entity has:

- Single ownership.
- Version history.
- Auditability.
- Traceability.
- Lifecycle.
- Relationships.
- Metadata.

---

# Data Architecture

```text
Applications

↓

Domain Services

↓

Aggregates

↓

Repositories

↓

Persistence Layer

↓

Storage Engines
```

---

# Storage Architecture

```text
Transactional Database

↓

Event Store

↓

Analytics Warehouse

↓

Vector Database

↓

Knowledge Graph

↓

Object Storage

↓

Cache
```

---

# Storage Technologies

| Storage | Purpose |
|----------|----------|
| PostgreSQL | Transactional data |
| Event Store | Domain events |
| Redis | Cache |
| Object Storage | Files and media |
| Vector Database | Semantic search |
| Knowledge Graph | AI relationships |
| Data Warehouse | Analytics |

Technology selection may evolve without changing the domain model.

---

# Domain Catalog

| Domain | Owner |
|----------|-------|
| Identity | IAM |
| Organizations | Core Platform |
| Clients | Client Domain |
| Assessments | Assessment Domain |
| Training | Training Domain |
| Nutrition | Nutrition Domain |
| Calendar | Scheduling Domain |
| Messaging | Communication Domain |
| AI | AI Platform |
| Billing | Finance Domain |
| Analytics | Data Platform |

---

# Aggregate Catalog

Core aggregates include:

- Organization
- Coach
- Client
- Human Digital Twin
- Assessment
- Training Program
- Nutrition Plan
- Appointment
- Conversation
- Invoice

Aggregates enforce transactional consistency.

---

# Core Entities

## Organization

Stores organizational information.

Relationships:

- Coaches
- Clients
- Billing
- Settings

---

## Coach

Represents professional users.

Includes:

- Profile
- Credentials
- Permissions
- Specialties
- Availability

---

## Client

Represents end users receiving services.

Includes:

- Personal profile
- Goals
- Assessments
- Programs
- History
- Consent

---

## Human Digital Twin

Maintains:

- Physiological profile.
- Behavioral profile.
- Training profile.
- Nutrition profile.
- Recovery profile.
- Predictive indicators.

Exactly one Digital Twin exists per active client.

---

## Assessment

Stores:

- Measurements.
- Evaluations.
- Health indicators.
- Functional tests.
- Attachments.

Assessments are immutable after approval.

---

## Training Program

Contains:

- Objectives.
- Sessions.
- Exercises.
- Progressions.
- Versions.

Programs support version history.

---

## Nutrition Plan

Stores:

- Meals.
- Macronutrients.
- Supplements.
- Restrictions.
- Adherence.

---

## Appointment

Includes:

- Date.
- Duration.
- Participants.
- Notes.
- Status.

---

## Conversation

Contains:

- Messages.
- Attachments.
- Participants.
- AI summaries.

---

## Billing

Includes:

- Subscriptions.
- Invoices.
- Payments.
- Transactions.

---

# Relationships

```text
Organization

↓

Coach

↓

Clients

↓

Programs

↓

Assessments

↓

Digital Twin
```

---

# Entity Lifecycle

Every entity supports:

```text
Created

↓

Active

↓

Updated

↓

Archived

↓

Deleted
```

Soft delete is preferred for business entities.

---

# Metadata Model

Every entity contains:

- ID
- Version
- Created At
- Updated At
- Created By
- Updated By
- Tenant ID
- Status
- Audit Reference

---

# Data Integrity

Integrity is maintained through:

- Foreign keys.
- Aggregate boundaries.
- Business Rules.
- Domain validation.
- Optimistic locking.

---

# Human Digital Twin Model

```text
Identity

↓

Health

↓

Behavior

↓

Nutrition

↓

Training

↓

Recovery

↓

Predictions
```

The Digital Twin is continuously updated through events.

---

# AI Data Layer

AI consumes:

- Assessments.
- Progress.
- Programs.
- Nutrition.
- Behaviors.
- Events.
- Knowledge Graph.
- Vector embeddings.

---

# Event Sourcing

Selected aggregates support event sourcing.

Examples:

- Human Digital Twin
- Assessments
- Programs

Events remain immutable.

---

# Data Retention

Retention policies vary by entity.

Examples:

| Entity | Retention |
|----------|-----------|
| Audit Logs | 7 years |
| Messages | Configurable |
| Assessments | Organization Policy |
| AI Logs | Configurable |

---

# Data Security

Every entity supports:

- Tenant isolation.
- Encryption at rest.
- Encryption in transit.
- RBAC.
- Audit logging.
- Data masking.

---

# Data Governance

Each domain defines:

- Data owner.
- Steward.
- Classification.
- Retention.
- Quality rules.
- Lineage.
- Versioning.

---

# Repository Structure

```text
data-model/
├── domains/
├── aggregates/
├── entities/
├── relationships/
├── schemas/
├── metadata/
├── ai/
├── events/
├── warehouse/
├── diagrams/
└── metadata.yml
```

---

# Standard Visual Artifacts

## Domain Model

```text
Organization

↓

Coach

↓

Client

↓

Programs
```

---

## Aggregate Model

```text
Aggregate

↓

Entities

↓

Value Objects
```

---

## Human Digital Twin

```text
Data

↓

Twin

↓

Prediction

↓

Recommendation
```

---

## Storage Architecture

```text
PostgreSQL

↓

Events

↓

Warehouse

↓

AI
```

---

# Visual Source Files

```text
artifacts/
└── data-model/
    ├── domain-model.drawio
    ├── er-diagram.drawio
    ├── aggregates.drawio
    ├── digital-twin.drawio
    ├── storage-architecture.drawio
    ├── ai-data-layer.drawio
    ├── mermaid/
    │   ├── domains.mmd
    │   ├── entities.mmd
    │   ├── er.mmd
    │   ├── storage.mmd
    │   └── twin.mmd
    └── exports/
        ├── *.svg
        ├── *.png
        └── *.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Workflow Architecture | Chapter 16 |
| Business Rules | Chapter 17 |
| User Stories | Chapter 18 |
| API Contracts | Chapter 19 |
| Event Architecture | Chapter 20 |
| AI Services | Chapter 21 |
| Security & Permissions | Chapter 23 |
| Observability & Analytics | Chapter 24 |

---

# Acceptance Criteria

This chapter is complete when:

- Domain model is documented.
- Aggregate boundaries are defined.
- Entity catalog is complete.
- Relationships are specified.
- Human Digital Twin model is documented.
- Polyglot persistence architecture is defined.
- Data governance is established.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- The Coach Portal adopts a domain-driven, polyglot persistence strategy that aligns storage technologies with business and technical requirements.
- Business entities, aggregates and metadata are governed through clear ownership, lifecycle management and versioning.
- The Human Digital Twin acts as a central domain object, continuously enriched by events, assessments and AI services.
- Strong governance, security and traceability provide a scalable data foundation for analytics, automation and intelligent decision support across the EVOXA ecosystem.

---

# Next Chapter

**Chapter 23 — Security & Permissions**

This chapter defines the security architecture of the Coach Portal, including authentication, authorization, tenant isolation, data protection, audit logging, AI security, compliance controls and permission governance.
