---
document_id: BP-0003-V3-C07-05-05
chapter_id: CH-05-NUTRITION-05
feature_pack: FP-NUTRITION-0000
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

> *The User Journeys chapter defines the end-to-end professional workflows performed within the Nutritionist Portal, covering nutritional care, AI-assisted planning, monitoring, collaboration and continuous client engagement.*

---

# Executive Summary

The Nutritionist Portal supports the complete nutritional care lifecycle.

Every journey has been designed to minimize administrative work while maximizing clinical effectiveness, personalization and collaboration.

Artificial Intelligence assists professionals throughout each workflow without replacing professional judgment.

---

# Objectives

The User Journeys shall:

- Standardize nutritional workflows.
- Improve professional productivity.
- Reduce repetitive work.
- Increase client adherence.
- Enable multidisciplinary collaboration.
- Integrate AI naturally.
- Continuously update the Human Digital Twin.

---

# Journey Philosophy

Every journey follows:

Assessment

↓

Analysis

↓

Planning

↓

Execution

↓

Monitoring

↓

Optimization

↓

Continuous Improvement

---

# Journey Catalog

The Nutritionist Portal includes:

- Client Onboarding
- Initial Nutritional Assessment
- Follow-up Consultation
- Meal Plan Creation
- Recipe Management
- Supplement Planning
- Shopping List Generation
- Progress Monitoring
- AI Consultation
- Multidisciplinary Collaboration
- Reporting
- Client Communication

---

# Journey 01

## Client Onboarding

### Trigger

A new client is registered.

### Workflow

```text
Create Client

↓

Verify Identity

↓

Collect Medical History

↓

Define Goals

↓

Consent Management

↓

Create Human Digital Twin

↓

Schedule Assessment
```

### AI Assistance

- Risk identification
- Missing information detection
- Suggested assessment forms

### Outcome

Client profile ready for nutritional assessment.

---

# Journey 02

## Initial Nutritional Assessment

### Workflow

```text
Client

↓

Anthropometric Assessment

↓

Dietary Assessment

↓

Lifestyle Evaluation

↓

Clinical Indicators

↓

AI Analysis

↓

Nutrition Diagnosis
```

Collected information includes:

- Height
- Weight
- Body composition
- Waist circumference
- Dietary habits
- Allergies
- Health conditions
- Medications
- Activity level
- Goals

### AI Assistance

- Nutritional risk scoring
- Deficiency detection
- Recommended additional assessments

### Outcome

Comprehensive nutritional profile.

---

# Journey 03

## Meal Plan Creation

### Workflow

```text
Assessment

↓

Energy Calculation

↓

Macro Distribution

↓

Meal Design

↓

Recipe Selection

↓

AI Optimization

↓

Professional Review

↓

Publish
```

### AI Assistance

Supports:

- Calorie estimation
- Macronutrient optimization
- Meal timing
- Recipe suggestions
- Portion calculations
- Alternative meals

### Outcome

Personalized nutrition plan.

---

# Journey 04

## Recipe Management

Workflow:

```text
Recipe Library

↓

Search

↓

Customize

↓

Nutrition Analysis

↓

Assign

↓

Meal Plan
```

AI recommends:

- Ingredient substitutions
- Portion adjustments
- Dietary adaptations
- Preparation alternatives

---

# Journey 05

## Supplement Planning

Workflow:

```text
Assessment

↓

Identify Needs

↓

Review Contraindications

↓

AI Suggestions

↓

Professional Approval

↓

Publish
```

AI never prescribes supplements independently.

Professional approval is mandatory.

---

# Journey 06

## Shopping List Generation

Workflow:

```text
Meal Plan

↓

Aggregate Ingredients

↓

Calculate Quantities

↓

Group Categories

↓

Generate Shopping List
```

Output formats:

- Mobile
- PDF
- Printable
- Shared List

---

# Journey 07

## Progress Monitoring

Workflow

```text
Measurements

↓

Diet Adherence

↓

Behavior Analysis

↓

AI Prediction

↓

Professional Review

↓

Plan Adjustment
```

Indicators include:

- Weight
- Body composition
- Compliance
- Energy intake
- Goal progress
- Behavioral consistency

---

# Journey 08

## AI Nutrition Consultation

Workflow

```text
Professional Question

↓

Context Collection

↓

Digital Twin

↓

Knowledge Retrieval

↓

AI Recommendation

↓

Explanation

↓

Professional Decision
```

AI provides:

- Nutritional guidance
- Meal alternatives
- Evidence summaries
- Risk indicators

Every recommendation includes confidence and explanation.

---

# Journey 09

## Multidisciplinary Collaboration

Workflow

```text
Nutritionist

↓

Coach

↓

Physician

↓

Physiotherapist

↓

Shared Client Timeline
```

Shared information respects:

- Permissions
- Consent
- Privacy
- Audit requirements

---

# Journey 10

## Client Communication

Supports:

- Messages
- Meal reminders
- Progress updates
- Educational material
- Appointment reminders
- AI summaries

---

# Journey 11

## Reporting

Workflow

```text
Assessment

↓

Progress

↓

AI Insights

↓

Charts

↓

Professional Review

↓

Export
```

Available formats:

- PDF
- Excel
- Interactive Dashboard

---

# Human Digital Twin Updates

Every journey contributes to:

- Nutritional profile
- Eating behavior
- Body composition
- Goal evolution
- Recovery indicators
- AI predictions

The Digital Twin evolves continuously.

---

# AI Interaction Points

AI assists during:

- Assessments
- Meal planning
- Recipe recommendations
- Deficiency detection
- Progress analysis
- Report generation
- Client communication

Human professionals always approve critical recommendations.

---

# Notifications

Typical notifications include:

- Assessment due
- Meal plan published
- Follow-up reminder
- Low adherence alert
- Nutritional risk detected
- Client message received

---

# Business Events

Journeys generate events such as:

- NutritionalAssessmentCompleted
- MealPlanPublished
- SupplementPlanApproved
- ProgressUpdated
- AIRecommendationAccepted
- ShoppingListGenerated

---

# Success Metrics

Measured indicators:

| KPI | Target |
|------|---------|
| Assessment Completion | >95% |
| Meal Plan Publication | <30 minutes |
| Client Adherence | >80% |
| AI Acceptance | >75% |
| Follow-up Completion | >90% |

---

# Standard Visual Artifacts

## Complete Nutrition Journey

```text
Client

↓

Assessment

↓

Meal Plan

↓

Monitoring

↓

Optimization
```

---

## AI Assisted Journey

```text
Nutritionist

↓

AI

↓

Recommendation

↓

Approval

↓

Client
```

---

## Human Digital Twin Evolution

```text
Assessment

↓

Digital Twin

↓

Prediction

↓

Recommendation
```

---

## Collaboration Journey

```text
Nutritionist

↓

Coach

↓

Physician

↓

Client
```

---

# Visual Source Files

```text
artifacts/
└── user-journeys/
    ├── onboarding.drawio
    ├── assessment.drawio
    ├── meal-planning.drawio
    ├── progress.drawio
    ├── ai-consultation.drawio
    ├── collaboration.drawio
    ├── digital-twin.drawio
    ├── mermaid/
    │   ├── onboarding.mmd
    │   ├── assessment.mmd
    │   ├── meal-plan.mmd
    │   ├── progress.mmd
    │   ├── ai.mmd
    │   └── collaboration.mmd
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
| Data Model | Chapter 22 |

---

# Acceptance Criteria

This chapter is complete when:

- All core nutritional workflows are documented.
- AI interaction points are defined.
- Human Digital Twin updates are identified.
- Collaboration journeys are specified.
- Business events are mapped.
- Success metrics are established.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- The Nutritionist Portal supports the complete nutritional care lifecycle, from client onboarding through continuous monitoring and optimization.
- AI is embedded across every major workflow, providing contextual recommendations, predictive insights and evidence-based guidance while preserving professional oversight.
- Every interaction enriches the Human Digital Twin, enabling increasingly personalized nutritional interventions over time.
- Standardized journeys, multidisciplinary collaboration and event-driven integration ensure consistent, scalable and measurable nutritional care throughout the EVOXA ecosystem.

---

# Next Chapter

**Chapter 06 — Information Architecture**

This chapter defines the information architecture of the Nutritionist Portal, including navigation hierarchies, content organization, domain boundaries, information flows and relationships between modules to support efficient, intuitive and scalable professional workflows.
