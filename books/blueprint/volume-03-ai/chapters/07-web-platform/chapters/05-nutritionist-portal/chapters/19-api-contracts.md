---
document_id: BP-0003-V3-C07-04-19
chapter_id: CH-04-COACH-19
feature_pack: FP-COACH-0000
title: API Contracts
version: 1.0.0
status: Draft
owner: API Architecture Board
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-04
---

# Chapter 19 — API Contracts

> *The API Contracts chapter defines the complete service interface of the Coach Portal, establishing REST conventions, security standards, resource models, AI integration contracts and interoperability guidelines across the EVOXA Platform.*

---

# Executive Summary

The Coach Portal exposes a comprehensive set of REST APIs that enable interaction between professional applications, AI services, mobile clients and external systems.

Every API follows a standardized contract that guarantees consistency, security, discoverability and long-term compatibility.

API contracts are versioned independently from implementation.

---

# Objectives

The API Architecture shall:

- Standardize REST APIs.
- Ensure interoperability.
- Support AI integration.
- Enable platform scalability.
- Protect client data.
- Simplify SDK generation.
- Maintain backward compatibility.

---

# API Philosophy

Every API follows:

- Resource-oriented design.
- REST principles.
- OpenAPI specification.
- Stateless communication.
- Consistent naming.
- Versioned contracts.
- Predictable behavior.

---

# API Layers

```text
Client

↓

Gateway

↓

REST API

↓

Application Services

↓

Domain Services

↓

Repositories
```

---

# Base URL

```text
/api/v1
```

Future versions:

```text
/api/v2
```

Versioning is URI-based.

---

# Authentication

Supported methods:

- OAuth2
- JWT Bearer
- Refresh Tokens
- API Keys (Integrations)
- Service Accounts

Every request requires authentication unless explicitly documented.

---

# Authorization

Authorization uses Role-Based Access Control (RBAC).

Examples:

- Coach
- Nutritionist
- Physiotherapist
- Team Manager
- Administrator

Permissions are evaluated before every protected operation.

---

# API Naming Standards

Resources use plural nouns.

Examples:

```text
GET /clients

POST /clients

GET /programs

GET /appointments
```

Avoid verbs in resource paths.

---

# HTTP Methods

| Method | Purpose |
|----------|----------|
| GET | Read |
| POST | Create |
| PUT | Replace |
| PATCH | Partial Update |
| DELETE | Remove |

---

# Standard Response

```json
{
  "success": true,
  "data": {},
  "meta": {},
  "links": {}
}
```

---

# Standard Error

```json
{
  "success": false,
  "error": {
    "code": "CLIENT_NOT_FOUND",
    "message": "Client not found."
  }
}
```

---

# Pagination

Supported parameters:

```text
?page=1

?page_size=25
```

Response:

```json
{
  "data": [],
  "pagination": {
    "page": 1,
    "page_size": 25,
    "total": 340
  }
}
```

---

# Filtering

Example:

```text
GET /clients?status=active

GET /appointments?date=today

GET /programs?coach=12
```

---

# Sorting

```text
?sort=name

?sort=-created_at
```

---

# Search

```text
GET /clients/search?q=john
```

---

# Resource Catalog

## Clients

```text
GET /clients

POST /clients

GET /clients/{id}

PATCH /clients/{id}

DELETE /clients/{id}
```

---

## Assessments

```text
GET /assessments

POST /assessments

GET /assessments/{id}
```

---

## Training

```text
GET /training-programs

POST /training-programs

PATCH /training-programs/{id}
```

---

## Nutrition

```text
GET /nutrition-plans

POST /nutrition-plans

PATCH /nutrition-plans/{id}
```

---

## Calendar

```text
GET /appointments

POST /appointments

PATCH /appointments/{id}
```

---

## Messaging

```text
GET /messages

POST /messages
```

---

## AI

```text
POST /ai/recommendations

POST /ai/chat

POST /ai/summaries

GET /ai/history
```

---

## Analytics

```text
GET /analytics/dashboard

GET /analytics/clients

GET /analytics/business
```

---

## Billing

```text
GET /subscriptions

GET /payments

POST /payments
```

---

# AI Contracts

Every AI endpoint returns:

```json
{
  "recommendation": "...",
  "confidence": 0.94,
  "explanation": "...",
  "sources": []
}
```

AI contracts remain explainable.

---

# Validation

Validation errors return:

```http
422 Unprocessable Entity
```

---

# Status Codes

| Code | Meaning |
|--------|----------|
| 200 | Success |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict |
| 422 | Validation Error |
| 429 | Too Many Requests |
| 500 | Internal Error |

---

# Idempotency

POST endpoints supporting retries accept:

```text
Idempotency-Key
```

header.

---

# Rate Limiting

Example:

```text
100 requests/minute
```

Headers:

```text
X-RateLimit-Limit

X-RateLimit-Remaining
```

---

# Event Integration

Every write operation publishes domain events.

Example:

```text
ClientCreated

ProgramAssigned

AssessmentCompleted
```

---

# Webhooks

Supported events:

- client.created
- assessment.completed
- program.assigned
- payment.completed

Webhook deliveries are signed and retryable.

---

# API Versioning

Major versions:

```
v1

v2
```

Breaking changes require a new major version.

---

# OpenAPI

Every endpoint is documented through:

- OpenAPI 3.1
- Swagger UI
- Redoc

SDKs are generated automatically.

---

# Security

APIs support:

- HTTPS
- JWT
- RBAC
- Audit Logging
- Rate Limiting
- Input Validation
- CORS
- CSRF protection (where applicable)

---

# Repository Structure

```text
api/
├── openapi/
├── clients/
├── assessments/
├── training/
├── nutrition/
├── calendar/
├── messaging/
├── analytics/
├── ai/
├── billing/
├── webhooks/
├── schemas/
└── metadata.yml
```

---

# Standard Visual Artifacts

## API Flow

```text
Client

↓

Gateway

↓

API

↓

Service

↓

Database
```

---

## Authentication Flow

```text
Login

↓

JWT

↓

API

↓

Authorization
```

---

## Request Lifecycle

```text
Request

↓

Validation

↓

Business Rules

↓

Response
```

---

## AI API

```text
Request

↓

AI

↓

Recommendation

↓

Response
```

---

# Visual Source Files

```text
artifacts/
└── api-contracts/
    ├── api-overview.drawio
    ├── authentication.drawio
    ├── request-lifecycle.drawio
    ├── ai-api.drawio
    ├── versioning.drawio
    ├── webhooks.drawio
    ├── mermaid/
    │   ├── api.mmd
    │   ├── auth.mmd
    │   ├── lifecycle.mmd
    │   ├── ai.mmd
    │   └── webhooks.mmd
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
| Event Architecture | Chapter 20 |
| AI Services | Chapter 21 |
| Data Model | Chapter 22 |
| Security & Permissions | Chapter 23 |

---

# Acceptance Criteria

This chapter is complete when:

- API conventions are documented.
- Resource catalog is defined.
- Authentication and authorization are specified.
- Error handling is standardized.
- AI contracts are documented.
- Versioning strategy is established.
- Webhooks and events are defined.
- OpenAPI documentation is available.
- Traceability is complete.

---

# Key Takeaways

- The Coach Portal exposes a standardized, versioned and secure REST API aligned with modern API-first principles.
- Consistent contracts, OpenAPI documentation and independent versioning simplify integration, SDK generation and long-term maintenance.
- AI endpoints provide explainable recommendations with confidence scores, preserving transparency and professional oversight.
- API governance ensures interoperability across the EVOXA ecosystem while supporting scalability, security and future platform evolution.

---

# Next Chapter

**Chapter 20 — Event Architecture**

This chapter defines the event-driven architecture of the Coach Portal, including domain events, event contracts, message flows, asynchronous processing, integration patterns and event governance.
