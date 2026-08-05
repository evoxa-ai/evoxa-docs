---
document_id: BP-0003-V3-C07-04-05
chapter_id: CH-04-COACH-05
feature_pack: FP-COACH-0000
title: User Journeys
version: 1.0.0
status: Draft
owner: UX Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-04
---

# Chapter 05 — User Journeys

> *The User Journeys chapter defines the end-to-end workflows performed by professional users within the Coach Portal, ensuring a consistent, AI-assisted and user-centered experience across all operational scenarios.*

---

# Executive Summary

The Coach Portal supports professionals throughout the complete coaching lifecycle.

Each journey represents a business process rather than a sequence of screens.

Journeys integrate Human Digital Twins, Artificial Intelligence, business rules, analytics and collaboration to improve operational efficiency and client outcomes.

---

# Journey Philosophy

A journey represents a complete professional objective.

Each journey defines:

- Persona
- Goal
- Trigger
- Preconditions
- Workflow
- AI Participation
- Success Criteria
- Related APIs
- Events
- Business Rules

---

# Journey Lifecycle

```text
Goal

↓

Planning

↓

Execution

↓

Monitoring

↓

Decision

↓

Completion
```

---

# Journey Catalog

| Journey ID | Journey | Primary Persona |
|------------|-----------------------------|----------------|
| JRN-COACH-001 | Coach Onboarding | Coach |
| JRN-COACH-002 | Daily Dashboard Review | Coach |
| JRN-COACH-003 | Client Registration | Coach |
| JRN-COACH-004 | Initial Assessment | Coach |
| JRN-COACH-005 | Training Program Creation | Coach |
| JRN-COACH-006 | Nutrition Plan Creation | Nutritionist |
| JRN-COACH-007 | Client Follow-up | Coach |
| JRN-COACH-008 | Appointment Management | Coach |
| JRN-COACH-009 | AI Recommendation Review | Coach |
| JRN-COACH-010 | Progress Evaluation | Coach |
| JRN-COACH-011 | Team Collaboration | Team Manager |
| JRN-COACH-012 | Business Analytics Review | Enterprise Coach |

---

# Journey 01 — Coach Onboarding

## Goal

Configure the professional workspace and prepare the platform for daily operations.

---

### Steps

```text
Login

↓

Profile Setup

↓

Organization Selection

↓

Permissions

↓

AI Configuration

↓

Dashboard Ready
```

---

### AI Assistance

EVO:

- Configures preferences.
- Explains platform capabilities.
- Suggests initial setup.
- Recommends dashboard widgets.

---

### Success Criteria

- Profile completed.
- Organization assigned.
- Dashboard configured.
- Notifications enabled.

---

# Journey 02 — Daily Dashboard Review

## Goal

Understand the operational status before beginning work.

---

### Activities

- Review today's appointments.
- Check AI alerts.
- Review client messages.
- Identify overdue tasks.
- Analyze high-risk clients.

---

### AI Assistance

EVO prioritizes:

- Urgent interventions.
- High-risk clients.
- Missed workouts.
- Nutrition adherence.
- Recovery warnings.

---

# Journey 03 — Client Registration

```text
New Client

↓

Identity Verification

↓

Profile

↓

Assessment

↓

Human Digital Twin

↓

AI Initialization

↓

Ready
```

---

### Output

A complete client profile with an initialized Human Digital Twin.

---

# Journey 04 — Initial Assessment

Professional performs:

- Medical questionnaire.
- Physical measurements.
- Goals definition.
- Lifestyle assessment.
- Mobility evaluation.

---

### AI Participation

EVO generates:

- Risk analysis.
- Initial recommendations.
- Suggested goals.
- Baseline summary.

---

# Journey 05 — Training Program Creation

```text
Assessment

↓

Goals

↓

AI Suggestions

↓

Coach Review

↓

Adjustments

↓

Approval

↓

Assignment
```

---

### AI Support

- Exercise recommendations.
- Weekly planning.
- Volume optimization.
- Progression suggestions.

Final approval belongs to the coach.

---

# Journey 06 — Nutrition Plan Creation

Nutritionist:

- Reviews assessments.
- Defines nutritional objectives.
- Creates meal plans.
- Schedules reviews.

---

### EVO

Suggests:

- Calorie targets.
- Meal timing.
- Macronutrient distribution.
- Alternative foods.

---

# Journey 07 — Client Follow-up

Professional reviews:

- Progress.
- Adherence.
- Messages.
- Measurements.
- Recovery.

---

### AI Analysis

EVO identifies:

- Positive trends.
- Risk patterns.
- Low adherence.
- Recommended interventions.

---

# Journey 08 — Appointment Management

```text
Schedule

↓

Confirmation

↓

Reminder

↓

Session

↓

Summary

↓

Next Appointment
```

---

### AI Support

- Detect scheduling conflicts.
- Recommend available slots.
- Optimize daily agenda.

---

# Journey 09 — AI Recommendation Review

Coach receives:

- AI insights.
- Risk alerts.
- Suggested adjustments.
- Behavioral analysis.

Professional chooses:

- Accept.
- Modify.
- Reject.

Every decision is recorded.

---

# Journey 10 — Progress Evaluation

Coach compares:

- Initial assessment.
- Current status.
- Goals.
- Adherence.
- Health indicators.

---

### AI

Produces:

- Progress summary.
- Predictive analysis.
- Suggested next objectives.

---

# Journey 11 — Team Collaboration

Team Manager:

- Assigns clients.
- Reviews workload.
- Shares observations.
- Approves interventions.

---

### AI

Recommends:

- Workload balancing.
- Specialist referrals.
- Team performance insights.

---

# Journey 12 — Business Analytics Review

Enterprise users review:

- Active clients.
- Revenue.
- Coach productivity.
- Program effectiveness.
- AI adoption.
- Customer retention.

---

# Cross-Journey Capabilities

All journeys support:

- Notifications.
- Messaging.
- Attachments.
- Audit logging.
- AI assistance.
- Accessibility.
- Offline recovery.

---

# Journey States

```text
Created

↓

In Progress

↓

Completed

↓

Archived
```

---

# Journey Metrics

| Metric | Description |
|----------|-------------|
| Completion Rate | Journey success |
| Average Duration | Time to completion |
| AI Usage | AI interaction frequency |
| User Satisfaction | Journey quality |
| Drop-off Rate | Abandoned journeys |

---

# Journey Analytics

Analytics evaluate:

- Journey efficiency.
- Bottlenecks.
- AI contribution.
- User behavior.
- Productivity.
- Clinical outcomes.

---

# Standard Journey Structure

```text
Trigger

↓

Workflow

↓

AI

↓

Decision

↓

Completion
```

---

# Standard Visual Artifacts

## Journey Map

```text
Persona

↓

Actions

↓

AI

↓

Outcome
```

---

## Client Lifecycle

```text
Registration

↓

Assessment

↓

Program

↓

Follow-up

↓

Results
```

---

## Coaching Cycle

```text
Assess

↓

Plan

↓

Execute

↓

Monitor

↓

Improve
```

---

## AI Decision Flow

```text
Context

↓

AI

↓

Recommendation

↓

Coach Decision
```

---

# Visual Source Files

```text
artifacts/
└── user-journeys/
    ├── coach-onboarding.drawio
    ├── client-lifecycle.drawio
    ├── coaching-cycle.drawio
    ├── ai-decision.drawio
    ├── business-journeys.drawio
    ├── journey-map.drawio
    ├── mermaid/
    │   ├── onboarding.mmd
    │   ├── assessment.mmd
    │   ├── training.mmd
    │   ├── followup.mmd
    │   └── analytics.mmd
    └── exports/
        ├── *.svg
        ├── *.png
        └── *.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Personas | Chapter 04 |
| Information Architecture | Chapter 06 |
| Workflow Architecture | Chapter 16 |
| Business Rules | Chapter 17 |
| User Stories | Chapter 18 |
| AI Services | Chapter 21 |
| Event Architecture | Chapter 20 |

---

# Acceptance Criteria

This chapter is complete when:

- Primary user journeys are documented.
- Journey objectives are defined.
- AI participation is specified.
- Success criteria are identified.
- Journey metrics are established.
- Cross-journey capabilities are documented.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- User Journeys describe complete professional workflows rather than isolated interface interactions.
- EVO supports every journey by providing contextual recommendations while preserving the professional's authority over decisions.
- Human Digital Twins, analytics and business rules enrich each journey with personalized insights and operational intelligence.
- Standardized journeys ensure consistency across UX, APIs, workflows, testing and analytics throughout the Coach Portal.

---

# Next Chapter

**Chapter 06 — Information Architecture**

This chapter defines the structural organization of the Coach Portal, including navigation hierarchy, content taxonomy, module relationships, information ownership and discoverability principles.
