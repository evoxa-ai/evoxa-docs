---
document_id: BP-0003-V3-C07-04-04
chapter_id: CH-04-COACH-04
feature_pack: FP-COACH-0000
title: Personas
version: 1.0.0
status: Draft
owner: UX Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-04
---

# Chapter 04 — Personas

> *The Personas chapter defines the professional users of the Coach Portal, describing their goals, responsibilities, workflows, permissions and interactions with the EVOXA Platform.*

---

# Executive Summary

The Coach Portal is designed for professionals responsible for delivering health, fitness and wellness services.

Unlike the User Portal, which serves individual members, the Coach Portal supports multiple professional roles with different responsibilities, permissions and operational objectives.

Each persona represents a primary user profile that guides product design, workflows, AI capabilities and security policies.

---

# Persona Philosophy

Personas represent real professional workflows rather than job titles.

Every feature within the Coach Portal shall identify:

- Primary Persona
- Secondary Persona
- Required Permissions
- Business Goals
- AI Interaction

---

# Persona Catalog

| Persona | Primary Role |
|----------|--------------|
| Personal Trainer | Physical training |
| Nutritionist | Nutrition planning |
| Physiotherapist | Recovery management |
| Medical Specialist | Clinical supervision |
| Wellness Coach | Lifestyle improvement |
| Enterprise Coach | Corporate wellness |
| Team Manager | Team supervision |
| Platform Administrator | System administration |

---

# Persona 01 — Personal Trainer

## Overview

The Personal Trainer manages the physical conditioning and long-term progress of individual clients.

---

### Responsibilities

- Create workout plans.
- Monitor adherence.
- Review assessments.
- Track goals.
- Communicate with clients.
- Adjust training plans.

---

### Primary Objectives

- Improve client performance.
- Increase adherence.
- Prevent injuries.
- Deliver measurable results.

---

### KPIs

- Workout completion rate.
- Goal achievement.
- Client retention.
- Client satisfaction.
- Active programs.

---

### AI Assistance

EVO helps by:

- Generating workouts.
- Suggesting progressions.
- Detecting plateaus.
- Predicting adherence.
- Producing summaries.

---

### Required Permissions

- Manage Clients
- Create Workouts
- View Analytics
- Messaging
- Calendar
- Reports

---

# Persona 02 — Nutritionist

## Overview

Responsible for personalized nutrition programs.

---

### Responsibilities

- Nutrition assessments.
- Meal planning.
- Supplement recommendations.
- Dietary follow-up.
- Progress reviews.

---

### KPIs

- Nutrition adherence.
- Weight evolution.
- Body composition changes.
- Goal completion.

---

### AI Assistance

- Meal suggestions.
- Nutrition optimization.
- Risk detection.
- Diet analysis.
- Automatic reports.

---

# Persona 03 — Physiotherapist

## Overview

Manages rehabilitation and recovery programs.

---

### Responsibilities

- Functional assessments.
- Recovery planning.
- Mobility programs.
- Injury monitoring.
- Progress evaluation.

---

### AI Assistance

- Recovery recommendations.
- Mobility analysis.
- Risk alerts.
- Recovery monitoring.

---

# Persona 04 — Medical Specialist

## Overview

Provides clinical oversight for users with medical conditions or specialized care requirements.

---

### Responsibilities

- Review health indicators.
- Validate care plans.
- Monitor medical risks.
- Coordinate multidisciplinary care.

---

### AI Assistance

- Trend summaries.
- Clinical alerts.
- Longitudinal analytics.

AI supports decision-making and never replaces clinical judgment.

---

# Persona 05 — Wellness Coach

## Overview

Focuses on behavior change, habits and long-term lifestyle improvement.

---

### Responsibilities

- Habit coaching.
- Goal tracking.
- Motivation.
- Client engagement.

---

### AI Assistance

- Habit analysis.
- Motivation recommendations.
- Behavioral insights.
- Progress summaries.

---

# Persona 06 — Enterprise Coach

## Overview

Supports wellness initiatives for organizations.

---

### Responsibilities

- Manage employee groups.
- Review organizational KPIs.
- Coordinate campaigns.
- Produce executive reports.

---

### KPIs

- Participation rate.
- Engagement.
- Program completion.
- Organizational health indicators.

---

# Persona 07 — Team Manager

## Overview

Leads teams of coaches and supervises service quality.

---

### Responsibilities

- Assign clients.
- Review workload.
- Monitor quality.
- Approve programs.
- Review KPIs.

---

### AI Assistance

- Capacity planning.
- Resource recommendations.
- Team analytics.

---

# Persona 08 — Platform Administrator

## Overview

Responsible for operational configuration of the Coach Portal.

---

### Responsibilities

- User management.
- Permissions.
- Integrations.
- Configuration.
- Auditing.

---

### AI Assistance

- Operational insights.
- Usage analytics.
- Security alerts.

---

# Persona Relationships

```text
Administrator

↓

Team Manager

↓

Coach

↓

Client

↓

Human Digital Twin

↓

AI (EVO)
```

---

# AI Interaction Model

Every professional collaborates with EVO.

```text
Professional

↓

Context

↓

Human Digital Twin

↓

EVO

↓

Recommendation

↓

Professional Decision
```

AI provides recommendations.

The professional always approves final actions.

---

# Permission Matrix

| Persona | Clients | Programs | Analytics | Billing | AI |
|-----------|---------|----------|-----------|----------|-----|
| Personal Trainer | ✓ | ✓ | ✓ | — | ✓ |
| Nutritionist | ✓ | ✓ | ✓ | — | ✓ |
| Physiotherapist | ✓ | ✓ | ✓ | — | ✓ |
| Medical Specialist | ✓ | View | ✓ | — | ✓ |
| Wellness Coach | ✓ | ✓ | ✓ | — | ✓ |
| Enterprise Coach | ✓ | ✓ | ✓ | ✓ | ✓ |
| Team Manager | ✓ | ✓ | ✓ | ✓ | ✓ |
| Administrator | ✓ | ✓ | ✓ | ✓ | ✓ |

---

# UX Considerations

The interface shall adapt according to:

- Professional role.
- Permissions.
- Organization.
- Subscription.
- Active clients.
- Preferred workflows.

Personalization improves efficiency while preserving consistency.

---

# Future Personas

The architecture supports future roles such as:

- Sports Scientist.
- Psychologist.
- Occupational Therapist.
- Corporate Health Manager.
- Insurance Advisor.
- Research Coordinator.

New personas integrate through the platform's RBAC model.

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Business Overview | Chapter 02 |
| Product Scope | Chapter 03 |
| User Journeys | Chapter 05 |
| Workflow Architecture | Chapter 16 |
| Security & Permissions | Chapter 23 |

---

# Acceptance Criteria

This chapter is complete when:

- Primary personas are defined.
- Responsibilities are documented.
- KPIs are identified.
- AI interactions are specified.
- Permission requirements are established.
- Persona relationships are documented.
- UX implications are identified.
- Traceability is complete.

---

# Key Takeaways

- The Coach Portal supports multiple professional personas with distinct goals, responsibilities and permissions.
- EVO acts as an AI copilot, augmenting professional expertise while preserving human oversight.
- Role-based experiences ensure that each persona accesses only the capabilities required for their responsibilities.
- The persona model provides the foundation for UX design, workflows, security policies and AI interactions throughout the EVOXA platform.

---

# Next Chapter

**Chapter 05 — User Journeys**

This chapter defines the end-to-end journeys of each professional persona, including onboarding, client management, program delivery, AI-assisted decision-making, communication and long-term follow-up.
