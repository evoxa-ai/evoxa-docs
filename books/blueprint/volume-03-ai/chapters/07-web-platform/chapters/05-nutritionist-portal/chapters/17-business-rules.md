---
document_id: BP-0003-V3-C07-05-17
chapter_id: CH-05-NUTRITION-17
feature_pack: FP-NUTRITION-0000
title: Business Rules
version: 1.0.0
status: Draft
owner: Business Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-04
---

# Chapter 17 — Business Rules

> *The Business Rules chapter defines the policies, constraints, validations and decision logic that govern the Nutritionist Portal, ensuring consistent, compliant and evidence-based nutritional care across the EVOXA ecosystem.*

---

# Executive Summary

Business Rules represent the operational knowledge of the Nutritionist Portal.

They define how the platform validates information, authorizes actions, manages workflows and enforces professional standards.

Rules are centralized, versioned and auditable, enabling consistent behavior across user interfaces, APIs, AI services and workflow engines.

---

# Objectives

The Business Rules shall:

- Standardize business decisions.
- Protect data integrity.
- Support regulatory compliance.
- Enable AI governance.
- Simplify maintenance.
- Improve auditability.
- Facilitate future evolution.

---

# Rule Philosophy

Every business rule shall be:

- Explicit
- Versioned
- Testable
- Traceable
- Auditable
- Independent from UI
- Independent from APIs

---

# Rule Categories

Business Rules are organized into:

- Client Rules
- Assessment Rules
- Meal Planning Rules
- Nutrition Rules
- Supplement Rules
- AI Rules
- Workflow Rules
- Reporting Rules
- Security Rules
- Organization Rules

---

# Client Rules

## BR-001 — Unique Client

Each client shall have a unique identifier within an organization.

Duplicate active client records are not permitted.

---

## BR-002 — Consent Required

A client must provide valid consent before:

- Assessments
- Meal plans
- AI recommendations
- Data sharing
- Human Digital Twin activation

---

## BR-003 — Active Status

Only active clients may receive new nutritional interventions.

---

# Assessment Rules

## BR-010 — Required Measurements

An assessment cannot be completed without:

- Height
- Weight
- Assessment date

Additional measurements may be optional according to organization policies.

---

## BR-011 — Historical Integrity

Approved assessments become immutable.

Corrections require a new version.

---

## BR-012 — Assessment Ownership

Only authorized nutrition professionals may approve assessments.

---

# Meal Planning Rules

## BR-020 — Assessment Dependency

A meal plan requires at least one approved assessment.

---

## BR-021 — Nutritional Validation

Meal plans shall include:

- Energy target
- Macronutrient distribution
- Meal schedule

---

## BR-022 — Publication Approval

Only approved meal plans may be published.

---

## BR-023 — Version Control

Publishing a new meal plan automatically archives the previous active version.

---

# Nutrition Rules

## BR-030 — Dietary Restrictions

Meal recommendations shall respect:

- Allergies
- Intolerances
- Medical restrictions
- Religious restrictions
- Lifestyle preferences

---

## BR-031 — Nutritional Targets

Every plan shall define measurable nutritional goals.

---

## BR-032 — Food Database Integrity

Only validated food entries may be used in published meal plans.

---

# Supplement Rules

## BR-040 — Professional Approval

Supplements recommended by AI require professional approval.

---

## BR-041 — Contraindication Validation

Potential contraindications must be evaluated before publication.

---

# Progress Rules

## BR-050 — Progress Recording

Measurements shall be timestamped and linked to the responsible professional.

---

## BR-051 — Goal Evaluation

Progress shall be calculated using the latest approved measurements.

---

# AI Rules

## BR-060 — Explainability

Every AI recommendation shall include:

- Confidence score
- Explanation
- Evidence references
- Generation timestamp

---

## BR-061 — Human Approval

AI recommendations shall never be applied automatically to client care.

Professional approval is mandatory.

---

## BR-062 — Traceability

Every AI interaction shall be recorded for audit purposes.

---

# Human Digital Twin Rules

## BR-070 — Automatic Synchronization

Approved assessments automatically update the Human Digital Twin.

---

## BR-071 — Version History

Historical Digital Twin states shall remain accessible.

---

## BR-072 — Predictive Models

Predictions shall be regenerated after significant clinical or nutritional changes.

---

# Workflow Rules

## BR-080 — State Validation

Workflow transitions must follow approved state diagrams.

Invalid transitions shall be rejected.

---

## BR-081 — Mandatory Review

Critical workflows require professional review before publication.

---

# Reporting Rules

## BR-090 — Data Consistency

Reports shall use only approved data.

Draft information shall never appear in official reports.

---

## BR-091 — Export Security

Report exports must respect user permissions and organization policies.

---

# Security Rules

## BR-100 — Authorization

Every operation shall validate:

- Identity
- Role
- Organization
- Permissions

---

## BR-101 — Audit Logging

Business-critical actions shall generate immutable audit records.

---

## BR-102 — Data Isolation

Organizations cannot access data belonging to other organizations.

---

# Organization Rules

## BR-110 — Organization Configuration

Organizations may customize:

- Meal templates
- Nutritional protocols
- Branding
- Notification policies

Core business rules remain centrally governed.

---

# Rule Evaluation

Rules are evaluated by the Business Rules Engine before:

- Workflow execution
- API processing
- AI recommendation publication
- Report generation

---

# Rule Priorities

| Priority | Description |
|----------|-------------|
| Critical | Must always be enforced |
| High | Required for workflow completion |
| Medium | Operational recommendation |
| Low | Informational validation |

---

# Rule Versioning

Each rule includes:

- Rule ID
- Name
- Version
- Effective date
- Owner
- Status
- Change history

Deprecated rules remain available for audit purposes.

---

# Exception Handling

Exceptions require:

- Justification
- Professional approval
- Audit logging
- Organization policy validation

---

# Compliance

Business rules support compliance with:

- Healthcare regulations
- Data privacy requirements
- Organizational policies
- Internal governance standards

---

# Repository Structure

```text
business-rules/
├── client/
├── assessments/
├── meal-plans/
├── nutrition/
├── supplements/
├── ai/
├── workflows/
├── reporting/
├── security/
├── governance/
└── metadata.yml
```

---

# Standard Visual Artifacts

## Rule Evaluation Flow

```text
Request

↓

Business Rules Engine

↓

Validation

↓

Decision

↓

Workflow
```

---

## Rule Hierarchy

```text
Policy

↓

Business Rule

↓

Validation

↓

Action
```

---

## AI Governance

```text
AI Recommendation

↓

Business Rules

↓

Professional Approval

↓

Publication
```

---

## Workflow Validation

```text
Workflow

↓

Rule Check

↓

Transition

↓

Audit
```

---

# Visual Source Files

```text
artifacts/
└── business-rules/
    ├── rule-engine.drawio
    ├── validation-flow.drawio
    ├── ai-governance.drawio
    ├── workflow-rules.drawio
    ├── compliance.drawio
    ├── rule-hierarchy.drawio
    ├── mermaid/
    │   ├── rules.mmd
    │   ├── validation.mmd
    │   ├── ai.mmd
    │   ├── workflow.mmd
    │   └── governance.mmd
    └── exports/
        ├── business-rules.svg
        ├── business-rules.png
        └── business-rules.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Workflow Architecture | Chapter 16 |
| User Stories | Chapter 18 |
| API Contracts | Chapter 19 |
| Event Architecture | Chapter 20 |
| AI Services | Chapter 21 |
| Data Model | Chapter 22 |
| Security & Permissions | Chapter 23 |

---

# Acceptance Criteria

This chapter is complete when:

- Business rule categories are documented.
- Client, assessment, meal planning and AI rules are defined.
- Workflow validations are specified.
- Security and compliance rules are documented.
- Rule versioning and exception handling are established.
- Governance and traceability are complete.
- Visual artifacts are available.

---

# Key Takeaways

- Business Rules encapsulate the operational policies that govern every professional activity within the Nutritionist Portal.
- Centralized, versioned and auditable rules ensure consistent behavior across workflows, APIs, AI services and user interfaces.
- Human oversight remains mandatory for all critical nutritional decisions, particularly those assisted by AI.
- A dedicated Business Rules Engine enables maintainable, compliant and scalable governance throughout the EVOXA ecosystem.

---

# Next Chapter

**Chapter 18 — User Stories**

This chapter defines the functional user stories, acceptance criteria and business scenarios that drive the implementation of the Nutritionist Portal, ensuring alignment between product requirements, workflows and development activities.
