---
document_id: BP-0003-V3-C07-04-10
chapter_id: CH-04-COACH-10
feature_pack: FP-COACH-0000
title: Navigation Architecture
version: 1.0.0
status: Draft
owner: UX Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-04
---

# Chapter 10 — Navigation Architecture

> *The Navigation Architecture defines how professionals move through the Coach Portal, ensuring fast, predictable and context-aware access to every capability, workflow and AI-powered experience.*

---

# Executive Summary

The Coach Portal supports complex professional workflows involving multiple clients, programs, appointments and business activities.

The navigation architecture is designed to minimize context switching while allowing professionals to access information and actions with minimal effort.

Navigation adapts to the user's role, current workflow, permissions and AI recommendations.

---

# Objectives

The Navigation Architecture shall:

- Reduce navigation effort.
- Support complex workflows.
- Preserve user context.
- Improve discoverability.
- Enable AI-assisted navigation.
- Scale across future modules.
- Maintain consistency.

---

# Navigation Philosophy

Navigation is task-oriented rather than menu-oriented.

Every navigation action should answer one of the following questions:

- Where am I?
- What can I do?
- What should I do next?
- How do I return?

---

# Navigation Layers

```text
Global Navigation

↓

Workspace Navigation

↓

Context Navigation

↓

Action Navigation
```

Each layer has a specific responsibility.

---

# Global Navigation

Global navigation provides access to major capability domains.

```text
Dashboard

Clients

Programs

Calendar

Messaging

Analytics

Billing

Marketplace

Administration
```

Characteristics:

- Always visible.
- Role-aware.
- Permission-aware.
- Search-integrated.
- Responsive.

---

# Workspace Navigation

Each workspace defines its own navigation.

Example:

```text
Clients

↓

Profile

Assessments

Training

Nutrition

Timeline

Messages

Reports
```

Workspace navigation remains consistent across all clients.

---

# Context Navigation

The Coach Portal preserves business context.

Examples:

- Current client.
- Selected program.
- Active appointment.
- Selected report.
- Current organization.

Changing screens does not reset the active context unless explicitly requested.

---

# Action Navigation

Users access contextual actions directly from the current screen.

Examples:

- Create Program
- Schedule Appointment
- Send Message
- Generate Report
- Request AI Recommendation

Actions are available where they are needed.

---

# Navigation Regions

The interface includes:

- Global Header.
- Navigation Sidebar.
- Workspace Menu.
- Breadcrumbs.
- Context Panel.
- Quick Actions.
- Footer Status.

---

# Sidebar Navigation

The sidebar supports:

- Collapse.
- Expand.
- Pin.
- Favorites.
- Recently Used.
- Workspace grouping.

---

# Breadcrumb Navigation

Example:

```text
Dashboard

>

Clients

>

John Smith

>

Training

>

Week 03
```

Breadcrumbs always represent the current navigation path.

---

# Search-Driven Navigation

Global search supports:

- Clients.
- Programs.
- Exercises.
- Nutrition plans.
- Reports.
- Messages.
- AI conversations.

Search results include direct navigation actions.

---

# Quick Navigation

Professionals may access:

- Recent clients.
- Favorite clients.
- Active sessions.
- Recent reports.
- AI suggestions.

This minimizes repetitive navigation.

---

# Keyboard Navigation

Supported shortcuts include:

| Shortcut | Action |
|-----------|--------|
| Ctrl + K | Global Search |
| Ctrl + D | Dashboard |
| Ctrl + C | Clients |
| Ctrl + Shift + A | AI Assistant |
| Ctrl + / | Keyboard Shortcuts |
| Esc | Close dialog |

Additional shortcuts may be configured by users.

---

# Deep Linking

Every major entity has a unique URL.

Examples:

```text
/clients/1024

/clients/1024/training

/programs/540

/calendar/appointments/884
```

Deep links support:

- Browser history.
- Sharing.
- Notifications.
- AI-generated links.

---

# AI-Assisted Navigation

EVO recommends navigation based on context.

Examples:

- Client requires attention.
- Appointment begins soon.
- Nutrition review pending.
- High-risk alert generated.
- Program needs adjustment.

Recommendations include direct navigation.

---

# Notification Navigation

Notifications link directly to:

- Client.
- Program.
- Appointment.
- Report.
- AI recommendation.
- Task.

Navigation preserves workflow state.

---

# Multi-Task Navigation

Professionals may work across multiple contexts.

Supported features:

- Multiple browser tabs.
- Pinned workspaces.
- Draft preservation.
- Navigation history.
- Workspace restoration.

---

# Navigation States

Every navigation flow supports:

- Loading.
- Empty.
- Active.
- Error.
- Offline.
- Unauthorized.

Transitions remain consistent across the platform.

---

# Navigation Personalization

Professionals can customize:

- Favorite modules.
- Sidebar order.
- Default landing page.
- Quick actions.
- Keyboard shortcuts.
- Recently used items.

Preferences synchronize across devices.

---

# Responsive Navigation

## Desktop

Persistent sidebar.

---

## Tablet

Collapsible sidebar.

---

## Mobile

Bottom navigation with contextual menus.

---

# Navigation Analytics

The platform measures:

- Navigation paths.
- Search usage.
- Frequently accessed screens.
- Drop-off points.
- Shortcut usage.
- AI navigation acceptance.

Insights support continuous UX improvements.

---

# Navigation Governance

Every new module shall:

- Integrate into the navigation hierarchy.
- Preserve context.
- Support breadcrumbs.
- Support deep links.
- Respect permissions.
- Follow naming standards.

Navigation consistency is mandatory.

---

# Standard Visual Artifacts

## Navigation Hierarchy

```text
Global

↓

Workspace

↓

Context

↓

Action
```

---

## Navigation Flow

```text
Dashboard

↓

Clients

↓

Profile

↓

Training

↓

Session
```

---

## Search Navigation

```text
Search

↓

Results

↓

Entity

↓

Action
```

---

## AI Navigation

```text
Alert

↓

Recommendation

↓

Navigation

↓

Workflow
```

---

# Visual Source Files

```text
artifacts/
└── navigation-architecture/
    ├── navigation-hierarchy.drawio
    ├── workspace-navigation.drawio
    ├── breadcrumbs.drawio
    ├── ai-navigation.drawio
    ├── search-navigation.drawio
    ├── responsive-navigation.drawio
    ├── mermaid/
    │   ├── navigation.mmd
    │   ├── breadcrumbs.mmd
    │   ├── search.mmd
    │   ├── ai-navigation.mmd
    │   └── responsive.mmd
    └── exports/
        ├── *.svg
        ├── *.png
        └── *.pdf
```

---

# Repository Structure

```text
navigation/
├── global/
├── workspaces/
├── breadcrumbs/
├── search/
├── deep-links/
├── shortcuts/
├── ai-navigation/
├── responsive/
└── metadata.yml
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| Information Architecture | Chapter 06 |
| UX Goals | Chapter 07 |
| Screen Catalog | Chapter 08 |
| Layout Architecture | Chapter 09 |
| Workflow Architecture | Chapter 16 |
| Accessibility & Internationalization | Chapter 25 |

---

# Acceptance Criteria

This chapter is complete when:

- Navigation hierarchy is defined.
- Global and workspace navigation are documented.
- Context preservation is specified.
- Search and deep linking are supported.
- AI-assisted navigation is defined.
- Responsive behavior is documented.
- Visual artifacts are available.
- Traceability is complete.

---

# Key Takeaways

- The Coach Portal provides a multi-layer navigation architecture optimized for professional workflows and long-duration use.
- Navigation is context-aware, preserving the active client and workflow while reducing unnecessary context switching.
- AI enhances discoverability by surfacing relevant actions, alerts and navigation shortcuts without disrupting the user experience.
- Standardized navigation patterns ensure scalability, consistency and efficient task execution across the entire EVOXA platform.

---

# Next Chapter

**Chapter 11 — Design Principles**

This chapter defines the visual, interaction and experience principles that guide every screen, component and workflow of the Coach Portal, ensuring a cohesive and professional design system aligned with the EVOXA platform.
