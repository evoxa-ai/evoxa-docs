---
document_id: BP-0003-V3-C08-08
chapter_id: CH-08-MOB-08
feature_pack: FP-MOBILE-0000
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

> *The Screen Catalog defines every screen of the EVOXA Mobile Platform, its purpose, navigation relationships, user interactions and responsibilities. It serves as the master inventory for UI design, development, QA and documentation.*

---

# Executive Summary

The Mobile Platform is composed of modular screens designed for speed, simplicity and AI-assisted productivity.

Unlike traditional mobile applications, every screen is designed as an intelligent workspace capable of adapting to:

- User Role
- Organization
- Device
- Permissions
- AI Context
- Connectivity
- Personal Preferences

Every screen follows the EVOXA Mobile Design System.

---

# Objectives

The Screen Catalog shall:

- Document every screen.
- Standardize layouts.
- Define navigation.
- Reduce UI inconsistencies.
- Improve developer productivity.
- Support scalability.
- Simplify QA validation.

---

# Screen Design Principles

Every screen follows:

- Mobile First
- AI Native
- Touch Optimized
- Responsive
- Accessible
- Offline Ready
- Performance Focused

---

# Screen Hierarchy

```text
Authentication

↓

Home

↓

Business Modules

↓

Detail Screens

↓

Dialogs

↓

Bottom Sheets
```

---

# Screen Categories

The platform organizes screens into:

- Authentication
- Home
- AI
- Productivity
- Business
- Collaboration
- Administration
- Settings
- System

---

# Authentication Module

---

## Splash Screen

Purpose

- Application startup

Features

- Branding
- Version Check
- Token Validation
- Auto Login

---

## Welcome Screen

Purpose

Introduce the application.

Features

- Sign In
- Register
- Language Selection
- Theme Selection

---

## Login Screen

Purpose

Authenticate users.

Features

- Email
- Password
- MFA
- Biometric Login
- Forgot Password

---

## MFA Screen

Purpose

Validate second factor.

Methods

- Authenticator
- SMS
- Email
- Passkeys

---

## Organization Selection

Purpose

Allow users to choose an organization.

Features

- Organization List
- Search
- Favorites

---

# Home Module

---

## Home Dashboard

Purpose

Main application entry point.

Widgets

- KPI Cards
- AI Recommendations
- Recent Activity
- Tasks
- Calendar
- Notifications

---

## AI Assistant

Purpose

Provide conversational AI.

Capabilities

- Chat
- Voice
- File Upload
- Prompt Templates
- Suggested Actions

---

## Notifications

Purpose

Display alerts.

Types

- AI
- Security
- Workflow
- System
- Billing

---

## Search

Purpose

Global search.

Supports

- Semantic Search
- Voice Search
- AI Search
- Recent Searches

---

# Productivity Module

---

## Task List

Purpose

Display assigned work.

Features

- Filters
- Priorities
- Status
- Due Dates

---

## Task Detail

Features

- Description
- Attachments
- Comments
- AI Summary
- Approval

---

## Calendar

Displays

- Meetings
- Tasks
- AI Events
- Workflow Deadlines

---

## Notes

Features

- Rich Text
- Voice Notes
- Images
- AI Summaries

---

# Workflow Module

---

## Workflow List

Displays

- Running
- Pending
- Completed

---

## Workflow Detail

Features

- Timeline
- Participants
- Documents
- Approvals
- AI Suggestions

---

## Approval Screen

Actions

- Approve
- Reject
- Request Changes

---

# Document Module

---

## Documents

Purpose

Manage files.

Features

- Upload
- Download
- Search
- Tags
- AI Classification

---

## Document Viewer

Supports

- PDF
- Images
- Office Files
- Markdown
- AI Summary

---

## Camera Capture

Supports

- Photos
- Documents
- QR
- OCR

---

# AI Module

---

## AI Chat

Capabilities

- Streaming
- Voice
- Attachments
- Context
- Suggestions

---

## Prompt Library

Displays

- Favorites
- Categories
- Recent
- Shared

---

## Prompt Detail

Shows

- Description
- Variables
- History
- Versions

---

# Profile Module

---

## User Profile

Displays

- Avatar
- Organization
- Activity
- Statistics

---

## Settings

Includes

- Notifications
- Theme
- Language
- Privacy
- Accessibility

---

## Security

Features

- MFA
- Sessions
- Devices
- Password
- Biometrics

---

# Administration Module

Available according to permissions.

Includes

- Users
- Roles
- Organizations
- Audit Logs
- Billing

---

# System Screens

Includes

- Offline
- Maintenance
- Update Available
- Error
- Session Expired

---

# Dialog Catalog

Dialogs include:

- Confirm Delete
- Logout
- Approve
- Share
- Save Changes
- Sync Conflict
- AI Feedback

---

# Bottom Sheets

Examples

- Quick Actions
- File Picker
- AI Actions
- Filters
- Date Picker

---

# Navigation Structure

```text
Home

├── Dashboard
├── AI
├── Tasks
├── Documents
├── Calendar
├── Notifications
├── Profile
└── Settings
```

---

# Screen Relationships

```text
Login

↓

Dashboard

↓

Business Module

↓

Detail

↓

Action

↓

Confirmation
```

---

# Floating Action Buttons

Primary actions:

- Create Task
- New Chat
- Upload
- Scan QR
- New Note

---

# Contextual Actions

Context menus include:

- Share
- Favorite
- Archive
- Delete
- Duplicate
- AI Analyze

---

# AI-Enhanced Screens

AI appears on:

- Dashboard
- Search
- Documents
- Notes
- Tasks
- Calendar
- Reports

---

# Offline Screens

Offline support includes:

- Dashboard
- Tasks
- Documents
- Forms
- Notes
- AI History

---

# Accessibility

Every screen supports:

- Dynamic Fonts
- VoiceOver
- TalkBack
- Keyboard Navigation
- High Contrast

---

# Responsive Layouts

Optimized for:

- Phones
- Tablets
- Foldables

---

# Screen Lifecycle

```text
Open

↓

Load

↓

Interact

↓

Save

↓

Close
```

---

# Naming Convention

```
mobile-[module]-[screen]

Examples

mobile-auth-login

mobile-home-dashboard

mobile-ai-chat

mobile-task-detail
```

---

# Screen Metrics

Measured

- Open Frequency
- Time on Screen
- Navigation Success
- Errors
- AI Usage
- Conversion

---

# Screen KPIs

| KPI | Target |
|------|--------|
| Average Screen Load | <500 ms |
| Navigation Success | >98% |
| Crash Rate | <0.1% |
| Screen Completion | >95% |
| AI Engagement | >75% |

---

# Repository Structure

```text
screen-catalog/
├── authentication/
├── dashboard/
├── ai/
├── tasks/
├── workflows/
├── documents/
├── profile/
├── settings/
├── administration/
├── dialogs/
├── bottom-sheets/
├── system/
└── metadata.yml
```

---

# Standard Visual Artifacts

## Screen Map

```text
Authentication

↓

Dashboard

↓

Modules

↓

Details
```

---

## Navigation Tree

```text
Home

↓

Feature

↓

Detail
```

---

## Mobile Flow

```text
Open

↓

Navigate

↓

Interact

↓

Complete
```

---

## Screen Lifecycle

```text
Launch

↓

Display

↓

Action

↓

Exit
```

---

## AI Navigation

```text
Prompt

↓

AI

↓

Screen
```

---

# Visual Source Files

```text
artifacts/
└── screen-catalog/
    ├── screen-map.drawio
    ├── navigation-tree.drawio
    ├── mobile-flow.drawio
    ├── dashboard-layout.drawio
    ├── screen-lifecycle.drawio
    ├── ai-screen-map.drawio
    ├── dialog-catalog.drawio
    ├── mermaid/
    │   ├── screens.mmd
    │   ├── navigation.mmd
    │   ├── dialogs.mmd
    │   ├── ai.mmd
    │   ├── lifecycle.mmd
    │   └── modules.mmd
    └── exports/
        ├── screen-catalog.svg
        ├── screen-catalog.png
        └── screen-catalog.pdf
```

---

# Traceability

| Artifact | Reference |
|----------|-----------|
| User Journeys | Chapter 05 |
| Information Architecture | Chapter 06 |
| UX Goals | Chapter 07 |
| Navigation Architecture | Chapter 09 |
| Design System | Chapter 11 |
| Component Catalog | Chapter 12 |
| Widget Catalog | Chapter 13 |

---

# Acceptance Criteria

This chapter is complete when:

- All mobile application screens are identified and categorized.
- Navigation relationships and screen responsibilities are defined.
- Dialogs, bottom sheets, system screens and contextual actions are documented.
- Responsive behavior, accessibility and AI-enhanced interactions are specified.
- KPIs, repository structure, visual artifacts and traceability are complete.

---

# Key Takeaways

- The EVOXA Mobile Platform organizes its interface into a modular catalog of reusable screens that support consistent development and maintenance.
- Every screen is optimized for mobile-first interactions, AI-assisted workflows and enterprise productivity while remaining responsive across phones, tablets and foldable devices.
- Native mobile patterns such as bottom navigation, floating action buttons, bottom sheets and contextual actions provide a familiar and efficient user experience.
- The Screen Catalog serves as the authoritative reference for designers, developers and QA teams, ensuring consistency across the entire mobile application lifecycle.

---

# Next Chapter

**Chapter 09 — Navigation Architecture**

This chapter defines the navigation system, routing model, deep-linking strategy, transition patterns and interaction architecture that connect every screen and workflow within the EVOXA Mobile Platform.
