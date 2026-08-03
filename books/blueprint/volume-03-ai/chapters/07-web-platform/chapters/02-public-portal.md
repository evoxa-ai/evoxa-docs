---
document_id: BP-0003-V3-C07-02
product_id: PRT-0001
portal_id: PRT-0001
chapter_id: PRT-0001-C02
book: EVOXA Blueprint
volume: Volume III
chapter: 7
section: 2
title: Public Portal
version: 1.0.0
status: Draft
classification: Internal
owner: Product Architecture Board
language: en
created: 2026-08-03
last_updated: 2026-08-03
---

# Chapter 7.2 — Public Portal

> *The Public Portal is the primary gateway to the EVOXA ecosystem. It informs, inspires, educates and converts visitors into active users.*

---

# Executive Summary

The Public Portal is the marketing, communication and onboarding experience of EVOXA.

It is optimized for:

- Discoverability
- Trust
- Education
- Conversion
- Accessibility
- Performance
- Search Engine Optimization (SEO)

Unlike authenticated portals, this portal is publicly accessible and designed to communicate EVOXA's mission while guiding visitors toward registration.

---

# Objectives

The Public Portal shall:

- Present EVOXA to the public.
- Explain platform capabilities.
- Showcase scientific credibility.
- Convert visitors into registered users.
- Support multilingual content.
- Provide educational resources.
- Offer AI-assisted product guidance.

---

# Primary Personas

| Persona | Goal |
|----------|------|
| Visitor | Learn about EVOXA |
| Potential Customer | Register |
| Enterprise Customer | Request a demo |
| Healthcare Professional | Understand scientific foundation |
| Researcher | Access publications |
| Developer | Explore APIs |
| Media | Access press resources |

---

# Portal Goals

- Increase registrations
- Increase product awareness
- Generate enterprise leads
- Build trust
- Reduce onboarding friction
- Improve SEO ranking

---

# Information Architecture

```text
Home
│
├── Platform
│   ├── Features
│   ├── AI
│   ├── Digital Twin
│   └── Technology
│
├── Solutions
│   ├── Individuals
│   ├── Coaches
│   ├── Clinics
│   ├── Enterprises
│   └── Researchers
│
├── Resources
│   ├── Blog
│   ├── Documentation
│   ├── Research
│   ├── Success Stories
│   └── FAQ
│
├── Pricing
│
├── Contact
│
└── Authentication
```

---

# Screen Catalog

| Screen ID | Screen |
|------------|----------------|
| SCR-0001 | Home |
| SCR-0002 | Features |
| SCR-0003 | Solutions |
| SCR-0004 | AI Platform |
| SCR-0005 | Digital Twin |
| SCR-0006 | Pricing |
| SCR-0007 | Blog |
| SCR-0008 | Research |
| SCR-0009 | Documentation |
| SCR-0010 | FAQ |
| SCR-0011 | Contact |
| SCR-0012 | About |
| SCR-0013 | Privacy Policy |
| SCR-0014 | Terms of Service |
| SCR-0015 | Login |
| SCR-0016 | Register |

---

# Layout

Portal Layout:

```
LAY-0001 Public Layout
```

Structure

```text
Header

↓

Hero

↓

Content Sections

↓

Testimonials

↓

Call To Action

↓

Footer
```

---

# Navigation

```text
Home

↓

Platform

↓

Solutions

↓

Resources

↓

Pricing

↓

Login

↓

Register
```

---

# Component Catalog

| Component ID | Component |
|--------------|--------------------------|
| CMP-0001 | Global Header |
| CMP-0002 | Navigation Menu |
| CMP-0003 | Hero Banner |
| CMP-0004 | Feature Card |
| CMP-0005 | CTA Banner |
| CMP-0006 | Pricing Table |
| CMP-0007 | Testimonial Carousel |
| CMP-0008 | FAQ Accordion |
| CMP-0009 | Blog Preview |
| CMP-0010 | Footer |

---

# Widget Catalog

| Widget ID | Widget |
|------------|----------------|
| WGT-0001 | Hero CTA |
| WGT-0002 | Platform Statistics |
| WGT-0003 | Success Stories |
| WGT-0004 | Latest Articles |
| WGT-0005 | Scientific Publications |
| WGT-0006 | Newsletter |
| WGT-0007 | AI Product Assistant |

---

# Forms

| Form ID | Form |
|----------|----------------|
| FORM-0001 | Contact Form |
| FORM-0002 | Newsletter |
| FORM-0003 | Demo Request |
| FORM-0004 | Login |
| FORM-0005 | Registration |

---

# Workflows

| Flow ID | Workflow |
|----------|------------------------|
| FLOW-0001 | Browse Website |
| FLOW-0002 | Register |
| FLOW-0003 | Login |
| FLOW-0004 | Subscribe Newsletter |
| FLOW-0005 | Request Demo |
| FLOW-0006 | Contact Support |

---

# Use Cases

| UC | Description |
|----|-------------|
| UC-0001 | Explore Platform |
| UC-0002 | Create Account |
| UC-0003 | Compare Plans |
| UC-0004 | Read Documentation |
| UC-0005 | Contact Sales |
| UC-0006 | Subscribe Newsletter |

---

# APIs

| API | Purpose |
|------|----------|
| API-0001 | Register |
| API-0002 | Login |
| API-0003 | Contact |
| API-0004 | Blog |
| API-0005 | Pricing |
| API-0006 | Newsletter |

---

# Published Events

| Event |
|--------|
| EVT-0001 LandingViewed |
| EVT-0002 CTASelected |
| EVT-0003 RegistrationStarted |
| EVT-0004 RegistrationCompleted |
| EVT-0005 ContactSubmitted |

---

# Permissions

Public screens require no authentication.

Authenticated actions:

- Register
- Login
- Password Reset

---

# AI Features

The Public Portal integrates:

- AI Product Guide
- Intelligent Search
- FAQ Assistant
- Product Recommendation Wizard

AI interactions are informational only and do not expose personal health recommendations.

---

# Analytics

Tracked events include:

- Landing page views
- CTA clicks
- Scroll depth
- Registration funnel
- Demo requests
- Contact submissions
- Newsletter subscriptions
- Search queries

---

# SEO Strategy

The Public Portal shall implement:

- Semantic HTML
- Structured data (Schema.org)
- XML sitemap
- Open Graph metadata
- Twitter Cards
- Canonical URLs
- Breadcrumbs
- Optimized Core Web Vitals

---

# Accessibility

Minimum compliance:

- WCAG 2.2 AA
- Keyboard navigation
- Screen reader compatibility
- High contrast support
- Responsive layouts

---

# Responsive Breakpoints

| Device | Width |
|----------|-------|
| Mobile | <768 px |
| Tablet | 768–1023 px |
| Desktop | ≥1024 px |
| Wide | ≥1440 px |

---

# Standard Visual Artifacts

## Sitemap

```text
Home
├── Platform
├── Solutions
├── Resources
├── Pricing
├── Contact
├── Login
└── Register
```

---

## Navigation Flow

```text
Visitor

↓

Landing

↓

Explore

↓

Register

↓

Dashboard
```

---

## Conversion Funnel

```text
Visit

↓

Read

↓

Trust

↓

Register

↓

Activate Account

↓

Onboarding
```

---

## Component Hierarchy

```text
Public Layout
│
├── Header
├── Hero
├── Sections
├── CTA
├── Footer
```

---

# Visual Source Files

```text
artifacts/
└── chapter-02/
    ├── sitemap.drawio
    ├── conversion-funnel.drawio
    ├── navigation.drawio
    ├── component-tree.drawio
    ├── user-journey.bpmn
    ├── sequence-register.puml
    ├── wireframes/
    │   ├── home.fig
    │   ├── pricing.fig
    │   ├── register.fig
    │   └── contact.fig
    ├── mermaid/
    │   ├── sitemap.mmd
    │   ├── funnel.mmd
    │   └── navigation.mmd
    └── exports/
        ├── *.svg
        ├── *.png
        └── *.pdf
```

---

# Product Design Package (PDP)

## Portal Inventory

- 1 Public Portal
- 16 Screens
- 10 Components
- 7 Widgets
- 5 Forms
- 6 Workflows
- 6 Use Cases
- 6 APIs
- 5 Events

---

# Implementation Readiness

## Engineering Specifications

- ESP-WEB-0100 — Public Portal
- ESP-WEB-0101 — Landing Page
- ESP-WEB-0102 — Registration
- ESP-WEB-0103 — Marketing Content

## Implementation Stories

- IS-WEB-0101 — Build Landing Page
- IS-WEB-0102 — Build Registration Flow
- IS-WEB-0103 — Build Contact Experience
- IS-WEB-0104 — Build Pricing Experience

---

# Key Takeaways

- The Public Portal is the primary acquisition channel for EVOXA.
- Every public interaction is measurable and traceable.
- Marketing, documentation and onboarding share a unified experience.
- AI enhances discovery without exposing protected user information.
- Every screen, component and workflow is registered in the EVOXA Catalog.

---

# Next Chapter

## Chapter 7.3 — User Portal

The next chapter specifies the authenticated experience for end users, including dashboards, nutrition, training, recovery, AI coaching, goals, analytics, community, profile management and personalized health journeys.
