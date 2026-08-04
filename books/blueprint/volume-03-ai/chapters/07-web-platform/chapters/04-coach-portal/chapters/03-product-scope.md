---
document_id: BP-0003-V3-C07-04-03
chapter_id: CH-04-COACH-03
feature_pack: FP-COACH-0000
title: Product Scope
version: 1.0.0
status: Draft
owner: Product Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-04
---

# Chapter 03 — Product Scope

> *The Product Scope defines the functional boundaries, business capabilities, feature packs and integration responsibilities of the EVOXA Coach Portal.*

---

# Executive Summary

The Coach Portal is the professional workspace of the EVOXA Platform.

Its primary purpose is to enable coaches and healthcare professionals to efficiently manage clients, deliver personalized services and leverage Artificial Intelligence throughout the coaching lifecycle.

This chapter establishes what is included within the Coach Portal, what belongs to other platform modules and how responsibilities are distributed across the EVOXA ecosystem.

---

# Product Purpose

The Coach Portal exists to help professionals:

- Manage clients.
- Deliver coaching services.
- Monitor progress.
- Collaborate with AI.
- Scale their business.
- Improve client outcomes.

---

# Product Boundaries

The Coach Portal owns professional workflows.

It does **not** replace:

- Public Portal
- User Portal
- Administration Portal
- Enterprise Portal
- AI Runtime

Instead, it orchestrates these platform capabilities.

---

# Functional Scope

The Coach Portal includes:

- Dashboard
- Client Management
- Human Digital Twin
- Assessments
- Training Programs
- Nutrition Programs
- Calendar
- Messaging
- Video Sessions
- Tasks
- Goals
- Progress Monitoring
- AI Assistant
- Analytics
- Reports
- Billing
- Marketplace
- Notifications
- Settings

---

# Out of Scope

The following capabilities belong to other modules:

| Capability | Owner |
|------------|-------|
| User Registration | User Portal |
| Marketing Website | Public Portal |
| Authentication | Identity Platform |
| AI Runtime | EVO Platform |
| Platform Administration | Admin Portal |
| Enterprise Management | Enterprise Portal |

---

# Feature Pack Catalog

## FP-COACH-0001

Dashboard

Professional operational dashboard.

---

## FP-COACH-0002

Client Management

Complete client lifecycle.

---

## FP-COACH-0003

Assessments

Physical and health evaluations.

---

## FP-COACH-0004

Training

Workout planning.

---

## FP-COACH-0005

Nutrition

Meal plans and nutrition tracking.

---

## FP-COACH-0006

Calendar

Appointments and scheduling.

---

## FP-COACH-0007

Messaging

Secure communication.

---

## FP-COACH-0008

Video Sessions

Remote coaching.

---

## FP-COACH-0009

AI Assistant

EVO Professional Copilot.

---

## FP-COACH-0010

Analytics

Professional insights.

---

## FP-COACH-0011

Billing

Subscriptions and payments.

---

## FP-COACH-0012

Marketplace

Templates and third-party integrations.

---

# Capability Map

```text
Coach Portal

├── Dashboard
├── Clients
├── Digital Twins
├── Assessments
├── Training
├── Nutrition
├── Calendar
├── Messaging
├── Video Sessions
├── Goals
├── Tasks
├── Reports
├── Analytics
├── Billing
├── Marketplace
├── Settings
└── AI Assistant
```

---

# User Responsibilities

The platform supports different professional roles.

| Role | Responsibility |
|------|----------------|
| Coach | Training management |
| Nutritionist | Nutrition |
| Physiotherapist | Recovery |
| Doctor | Clinical supervision |
| Administrator | Organization |
| Enterprise Manager | Corporate wellness |

Permissions are defined in Chapter 23.

---

# AI Scope

Artificial Intelligence supports:

- Client summaries.
- Risk detection.
- Program generation.
- Nutrition suggestions.
- Goal optimization.
- Progress prediction.
- Report generation.
- Professional recommendations.

AI never replaces the professional.

---

# Human Digital Twin Scope

The Coach Portal consumes Human Digital Twins to:

- Understand client history.
- Personalize interventions.
- Predict outcomes.
- Support AI reasoning.
- Improve decision making.

The Human Digital Twin remains owned by the Platform Core.

---

# Integration Scope

The Coach Portal integrates with:

| Platform | Purpose |
|-----------|----------|
| Identity | Authentication |
| User Portal | Shared client information |
| AI Platform | Recommendations |
| Notification Platform | Alerts |
| Analytics Platform | KPIs |
| Billing Platform | Payments |
| Marketplace | Extensions |
| Event Platform | Event processing |

---

# Business Scope

Professional operations include:

- Client onboarding.
- Assessments.
- Program delivery.
- Progress monitoring.
- Communication.
- Scheduling.
- Reporting.
- Billing.

---

# Non-Functional Scope

The Coach Portal supports:

- Multi-tenant architecture.
- Responsive interface.
- High availability.
- Accessibility.
- Explainable AI.
- Audit logging.
- Enterprise security.

---

# Future Scope

Future capabilities include:

- Wearable integrations.
- Clinical workflows.
- AI Agents.
- Smart scheduling.
- Automated follow-up.
- Predictive health monitoring.
- Team collaboration.
- Research tools.

---

# Product Context

```text
EVOXA Platform

↓

Coach Portal

↓

Professional Services

↓

Human Digital Twin

↓

AI Assistant

↓

Clients
```

---

# Product Dependencies

The Coach Portal depends on:

- Identity Platform.
- AI Platform.
- Human Digital Twin.
- Event Platform.
- Analytics Platform.
- Billing Platform.
- Marketplace.

---

# Success Criteria

The Product Scope is considered complete when:

- Functional boundaries are defined.
- Feature Packs are identified.
- Ownership is documented.
- Integration scope is established.
- Future evolution is considered.

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Overview | Chapter 01 |
| Business Overview | Chapter 02 |
| Personas | Chapter 04 |
| Workflow Architecture | Chapter 16 |
| AI Services | Chapter 21 |
| Product Evolution | Chapter 30 |

---

# Acceptance Criteria

This chapter is complete when:

- Product boundaries are defined.
- Functional scope is documented.
- Out-of-scope capabilities are identified.
- Feature Packs are cataloged.
- AI responsibilities are defined.
- Integration boundaries are documented.
- Dependencies are identified.
- Traceability is complete.

---

# Key Takeaways

- The Coach Portal is responsible for the complete professional coaching workflow within the EVOXA ecosystem.
- Functional ownership is clearly separated from other platform modules, ensuring modularity and scalability.
- AI and the Human Digital Twin are core capabilities that enhance professional decision-making without replacing human expertise.
- The Feature Pack architecture provides a modular foundation for incremental development and future expansion.

---

# Next Chapter

**Chapter 04 — Personas**

This chapter defines the primary and secondary user personas of the Coach Portal, including goals, responsibilities, workflows, pain points, permissions and interactions with EVO, the AI-native assistant.
