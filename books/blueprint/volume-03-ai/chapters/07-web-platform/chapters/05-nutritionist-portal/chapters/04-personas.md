---
document_id: BP-0003-V3-C07-05-04
chapter_id: CH-05-NUTRITION-04
feature_pack: FP-NUTRITION-0000
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

> *The Personas chapter defines the primary and secondary user profiles of the Nutritionist Portal, including their goals, responsibilities, workflows, motivations and interaction patterns within the EVOXA ecosystem.*

---

# Executive Summary

The Nutritionist Portal supports a diverse ecosystem of nutrition professionals working across clinical, sports, preventive and corporate environments.

Each persona has unique objectives, responsibilities and decision-making processes.

Understanding these personas ensures that workflows, interfaces and AI capabilities remain aligned with real professional needs.

---

# Objectives

The Personas chapter shall:

- Define primary users.
- Identify secondary stakeholders.
- Describe professional goals.
- Understand workflow differences.
- Support UX decisions.
- Guide AI personalization.
- Improve product adoption.

---

# Persona Philosophy

Personas represent real professional behaviors rather than demographic profiles.

Each persona is described by:

- Responsibilities
- Goals
- Workflows
- Pain points
- Digital maturity
- AI expectations
- Collaboration patterns

---

# Persona Classification

The Nutritionist Portal supports:

- Primary Personas
- Secondary Personas
- Administrative Personas
- AI Personas

---

# Primary Persona 01

## Clinical Nutritionist

### Description

Provides nutritional care in hospitals, clinics and medical centers.

### Responsibilities

- Nutritional diagnosis
- Clinical assessments
- Therapeutic nutrition
- Medical collaboration
- Follow-up consultations

### Goals

- Improve patient outcomes
- Reduce treatment time
- Personalize interventions
- Increase adherence

### Pain Points

- Manual documentation
- Fragmented information
- Time-consuming meal planning
- Limited patient follow-up

### AI Support

- Clinical recommendations
- Nutrient deficiency alerts
- Meal optimization
- Risk detection

---

# Primary Persona 02

## Sports Nutritionist

### Description

Works with athletes and active individuals.

### Responsibilities

- Performance nutrition
- Competition planning
- Recovery nutrition
- Supplement guidance

### Goals

- Improve performance
- Optimize recovery
- Support body composition goals
- Maintain athlete compliance

### Pain Points

- Constant schedule changes
- High personalization requirements
- Training integration
- Competition preparation

### AI Support

- Performance meal planning
- Recovery optimization
- Energy expenditure prediction
- Supplement suggestions

---

# Primary Persona 03

## Wellness Nutritionist

### Description

Focuses on healthy lifestyle, prevention and weight management.

### Responsibilities

- Lifestyle coaching
- Weight management
- Habit formation
- Preventive nutrition

### Goals

- Long-term adherence
- Sustainable habits
- Client engagement
- Behavioral improvement

### Pain Points

- Client motivation
- Habit tracking
- Appointment follow-up
- Personalized education

### AI Support

- Habit coaching
- Behavioral analysis
- Personalized reminders
- Nutrition insights

---

# Primary Persona 04

## Pediatric Nutritionist

### Description

Provides nutritional care for infants, children and adolescents.

### Responsibilities

- Growth monitoring
- Nutritional education
- Family guidance
- Feeding behavior assessment

### Goals

- Healthy growth
- Family engagement
- Nutritional prevention
- Long-term development

### AI Support

- Growth monitoring
- Dietary recommendations
- Nutrient analysis
- Parent education

---

# Primary Persona 05

## Corporate Nutritionist

### Description

Supports employee wellness programs.

### Responsibilities

- Population nutrition
- Wellness campaigns
- Group education
- Preventive health initiatives

### Goals

- Improve workforce health
- Increase participation
- Reduce health risks
- Measure program impact

### AI Support

- Population analytics
- Wellness recommendations
- Engagement prediction
- Risk identification

---

# Secondary Personas

## Coach

Collaborates on:

- Training load
- Body composition
- Goal alignment
- Recovery planning

---

## Physician

Collaborates on:

- Clinical conditions
- Laboratory results
- Treatment plans
- Medical restrictions

---

## Physiotherapist

Collaborates on:

- Rehabilitation
- Recovery nutrition
- Injury management

---

## Organization Administrator

Responsible for:

- User management
- Organization settings
- Reporting
- Compliance

---

# AI Persona

## EVO

Acts as an intelligent assistant.

Responsibilities:

- Nutritional analysis
- Meal recommendations
- Progress summaries
- Risk alerts
- Predictive insights
- Professional assistance

EVO never replaces professional judgment.

---

# Shared Goals

All personas aim to:

- Improve nutritional outcomes.
- Increase client adherence.
- Reduce administrative effort.
- Deliver personalized care.
- Collaborate effectively.
- Leverage AI responsibly.

---

# Collaboration Matrix

| Persona | Coach | Physician | AI | Client |
|----------|--------|-----------|----|--------|
| Clinical Nutritionist | High | High | High | High |
| Sports Nutritionist | High | Medium | High | High |
| Wellness Nutritionist | Medium | Low | High | High |
| Pediatric Nutritionist | Medium | High | High | High |
| Corporate Nutritionist | Medium | Low | High | Medium |

---

# Permissions Overview

Examples:

| Persona | Clients | Meal Plans | Reports | AI |
|----------|---------|-----------|----------|----|
| Clinical Nutritionist | Full | Full | Full | Full |
| Sports Nutritionist | Full | Full | Full | Full |
| Wellness Nutritionist | Full | Full | Standard | Full |
| Pediatric Nutritionist | Full | Full | Full | Full |
| Corporate Nutritionist | Assigned Groups | Group Plans | Organization | Full |

Detailed permissions are defined in Chapter 23.

---

# User Characteristics

Professionals typically require:

- Fast workflows
- Evidence-based guidance
- Mobile accessibility
- AI transparency
- Secure collaboration
- Reliable analytics

---

# UX Implications

The portal should provide:

- Role-specific dashboards
- Personalized navigation
- Context-aware recommendations
- Minimal cognitive load
- Efficient documentation
- Intelligent automation

---

# Standard Visual Artifacts

## Persona Map

```text
Clinical

Sports

Wellness

Pediatric

Corporate
```

---

## Collaboration Network

```text
Nutritionist

↓

Coach

↓

Physician

↓

AI

↓

Client
```

---

## AI Interaction Model

```text
Nutritionist

↓

EVO

↓

Recommendation

↓

Decision
```

---

# Visual Source Files

```text
artifacts/
└── personas/
    ├── persona-map.drawio
    ├── collaboration-network.drawio
    ├── ai-interaction.drawio
    ├── permissions.drawio
    ├── workflows.drawio
    ├── mermaid/
    │   ├── personas.mmd
    │   ├── collaboration.mmd
    │   ├── ai.mmd
    │   ├── permissions.mmd
    │   └── workflow.mmd
    └── exports/
        ├── personas.svg
        ├── personas.png
        └── personas.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Overview | Chapter 01 |
| Business Overview | Chapter 02 |
| Product Scope | Chapter 03 |
| User Journeys | Chapter 05 |
| UX Goals | Chapter 07 |
| Workflow Architecture | Chapter 16 |
| Security & Permissions | Chapter 23 |

---

# Acceptance Criteria

This chapter is complete when:

- Primary personas are documented.
- Secondary personas are identified.
- Responsibilities and goals are defined.
- Pain points are analyzed.
- AI interaction is documented.
- Collaboration model is specified.
- UX implications are identified.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- The Nutritionist Portal serves multiple professional profiles across clinical, sports, wellness, pediatric and corporate nutrition, each with distinct workflows and objectives.
- AI capabilities provided by EVO are tailored to each persona, delivering contextual recommendations while preserving professional authority.
- Strong collaboration between nutritionists, coaches, physicians and other healthcare professionals is a core characteristic of the platform.
- Persona-driven design ensures that interfaces, workflows and intelligent services remain aligned with the real needs of nutrition professionals throughout the EVOXA ecosystem.

---

# Next Chapter

**Chapter 05 — User Journeys**

This chapter defines the end-to-end workflows of the Nutritionist Portal, including client onboarding, nutritional assessment, meal planning, AI-assisted decision making, progress monitoring and multidisciplinary collaboration.
