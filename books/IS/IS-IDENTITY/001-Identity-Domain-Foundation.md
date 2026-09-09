IS-IDENTITY-001 — Identity Domain Foundation

Document ID: IS-IDENTITY-001
Domain: Identity
ESP: ESP-0001 — Identity
Status: Refined
Priority: Critical
Version: 1.1.0
Target Repository: evoxa-platform
Documentation Repository: evoxa-docs

1. Purpose

Establish the implementation foundation of the EVOXA Identity Domain according to the lifecycle:
Blueprint → Reference Architecture → Engineering Standards → ESP → ADR → Implementation Story → Technical Tasks → Source Code → Testing → Deployment → Monitoring → Continuous Evolution.

2. Business Capability

Identity provides the foundation for Users, Organizations, Memberships, Roles, Permissions, Authentication, Sessions, Refresh Tokens, MFA, Authorization, Security Policies, Delegation, Audit/Security Events, Recovery, Federation, Provisioning and Domain/Integration Events.

3. User Story

As the EVOXA Platform, I need a secure and modular Identity Domain foundation so that authentication, authorization and tenant isolation can evolve consistently without bypassing the platform security model.

4. Scope

Identity bounded-domain structure and modules

Domain/Application/Infrastructure/API layers

Shared Kernel

Aggregates, entities, value objects and identifiers

Repository and Unit of Work contracts

Domain/integration event contracts and transactional outbox boundary

Tenant and actor contexts

Security/authentication abstractions

Authorization Runtime boundary

Policy, criticality, freshness and cache boundaries

Exceptions, observability, migrations, testing, CI and public contracts

Architecture documentation and traceability

Out of scope

Complete login/JWT workflow, complete refresh-token implementation, MFA providers, complete policy catalog, complete Identity PostgreSQL schema, production Redis authorization cache, complete API implementation, frontend/mobile, production Kubernetes and Event Platform provisioning.

5. Architecture

Presentation
    ↓
API
    ↓
Application
    ↓
Domain
    ↑
Infrastructure

Domain never depends on API or Infrastructure. Cross-module direct persistence access is forbidden. Authorization is centralized. Tenant isolation is explicit. Security failures fail closed.

6. Identity Modules

identity/
├── account/
├── organization/
├── membership/
├── authentication/
├── authorization/
├── role_permission/
├── policy/
├── session/
├── token/
├── mfa/
├── recovery/
├── federation/
├── provisioning/
├── audit/
├── security_events/
├── integration/
└── shared/

Each module follows domain/, application/, infrastructure/, api/.

7. Domain Foundation

Primary aggregate boundaries include User, Organization, Membership, Role, Permission, Delegation, Session, RefreshTokenFamily, MFAFactor, MFAChallenge and SecurityPolicy. AuditEvent, SecurityEvent and OutboxEvent are platform integration/persistence concepts.

8. Tenant Model

Global User
    ↓
Organization-scoped Membership
    ↓
Organization

Membership is the access boundary. Users may belong to multiple organizations. Cross-tenant access is denied by default. Platform cross-tenant operations require explicit authorization and audit.

9. Actor Model

Supported actors: USER, SERVICE, SYSTEM, AI_AGENT. AI agents use the same Identity authorization model and cannot weaken security.

10. Repository and Transaction Boundaries

Repositories are aggregate-oriented ports. Infrastructure implements them. Unit of Work owns transaction boundaries. Critical state changes and their transactional outbox records commit atomically.

11. Persistence

PostgreSQL is the transactional source of truth. Redis is cache/temporary state/rate-limiting/coordination infrastructure and is never authoritative for critical Identity state. Event Platform distributes versioned integration events.

12. Events

Domain Change
→ Transactional Outbox
→ Event Publisher
→ Event Platform
→ Consumers

Delivery is At-Least-Once; consumers are idempotent; events are versioned; correlation/causation/trace context is propagated; secrets are forbidden; global ordering is not assumed.

13. Security

The foundation supports Argon2id, asymmetric JWT signing/JWKS, stateful rotated refresh tokens, MFA, recovery, RBAC, ABAC, ownership, delegation, policy evaluation, tenant isolation and audit/security events. Exact cryptographic parameters remain governed implementation details.

14. Authentication and Tokens

Access tokens are short-lived, stateless and asymmetrically signed. Refresh tokens are opaque, stateful, random, protected, rotated, family-based and reuse-detected. Web refresh tokens use HttpOnly/Secure cookies; mobile uses platform secure storage; no localStorage or URL tokens.

15. MFA

MFA separates MFAFactor from MFAChallenge. Policy determines when MFA is required; MFA verifies the factor. Recovery cannot become an ungoverned bypass. Secrets and recovery material are never logged, emitted or exposed to AI.

16. Authorization Runtime

Request Validation
→ Authentication Validation
→ Tenant Guard
→ Context Construction
→ Policy Selection
→ Policy Evaluation
→ Mandatory Security Guards
→ RBAC
→ Ownership
→ ABAC
→ Delegation
→ Policy Composition
→ Conflict Resolution
→ Criticality
→ Freshness
→ Cache Eligibility
→ Decision
→ Audit/Telemetry

Decisions: ALLOW, DENY, CHALLENGE, REAUTHENTICATE, REQUIRE_MFA, REVOKE. Unknown/error cannot safely become ALLOW.

17. Criticality and Freshness

Authorization combines RBAC + ABAC + ownership + delegation + policy + tenant isolation.

Effective criticality:

MAX(Resource, Action, Context, Risk, SecurityState,
    Tenant, Delegation, MFA, PlatformMinimum)

Levels: C0 Critical Security, C1 High Security, C2 Standard, C3 Low Risk.

Freshness: F0 Authoritative, F1 Security Validated, F2 Controlled Eventual, F3 Restricted Degraded. Baseline: C0→F0, C1→F1, C2→F2-SHORT, C3→F2-MEDIUM.

18. Cache

Cache is an optimization, never the source of truth. Security epoch, entity version, policy version, context fingerprint and cache generation protect against stale authorization. Critical revocations require fresh/authoritative behavior.

19. Exceptions and Context

Exception hierarchy:
EvoxaError → DomainError/ApplicationError/SecurityError/InfrastructureError/SystemError.
Public errors use the standard EVOXA envelope with code, message and correlation_id.

Tenant, actor, security, trace, request and idempotency contexts remain explicit and distinct. Trace data never authorizes.

20. Observability

Identity follows Logs, Metrics, Traces, Audit and Security Events. Tokens, passwords, MFA secrets, recovery codes and private keys must never enter telemetry.

21. Testing and CI

Foundation testing includes unit, integration, API, architecture, contract, security, performance, concurrency, chaos/recovery and AI-security tests. Baselines: ≥85% overall, ≥95% security-critical, 100% critical tenant-isolation and critical security workflows. CI must block security regressions and architecture violations.

22. Public Contracts

Identity APIs are versioned under /api/v1 with OpenAPI 3.1. Initial families:
/auth/*, /users/*, /organizations/*, /memberships/*, /roles/*, /permissions/*, /sessions/*, /mfa/*, /authorization/*, /policies/*, /audit/*.

Canonical auth endpoints:
POST /api/v1/auth/login, /refresh, /logout, /logout-all.

23. Technical Tasks

TASK-001-01 through TASK-001-24:

API Runtime Foundation

Identity Domain Package

Layer Architecture

Identity Shared Kernel

Domain Events Foundation

Repository Contracts

Unit of Work / Transaction Boundary

PostgreSQL Infrastructure

Redis Infrastructure

Configuration & Secrets

Security Foundation

Tenant Context Foundation

Actor Context

Correlation & Trace Context

Exception Model

Dependency Injection

Architecture Boundary Tests

Initial Observability

Initial Migration Framework

Test Foundation

CI Foundation

Identity Public Contracts

Architecture Documentation

IS-001 Validation

24. Acceptance Criteria

AC-001: approved Identity module/layer structure exists.

AC-002: forbidden dependencies fail architecture tests.

AC-003: Shared Kernel has no API/Infrastructure dependency.

AC-004: invalid/missing tenant context cannot produce unsafe ALLOW.

AC-005: USER/SERVICE/SYSTEM/AI_AGENT are explicit actor types.

AC-006: persistence uses repository contracts.

AC-007: critical state and outbox write atomically.

AC-008: events contain no secrets.

AC-009: access and refresh-token responsibilities remain separated.

AC-010: MFA factor/challenge lifecycles remain separate.

AC-011: authorization uses the central runtime.

AC-012: authorization errors/unknowns fail closed.

AC-013: criticality selects the strongest applicable requirement.

AC-014: C0 cannot use stale cache-only ALLOW.

AC-015: Redis failure cannot create an authorization bypass.

AC-016: internal errors are sanitized.

AC-017: telemetry contains correlation/trace context without secrets.

AC-018: migrations establish schema without create_all().

AC-019: deterministic architecture/unit quality gates execute in CI.

AC-020: APIs are versioned under /api/v1.

AC-021: implementation is traceable to ESP/ADR/IS.

AC-022: after all tasks and validation, the foundation is ready for later Identity stories.

25. Security Invariants

No cross-tenant authorization by default; unknown/error never becomes ALLOW; criticality is monotonic toward stronger protection; revocation has priority; platform minimums cannot be weakened; delegation cannot amplify privilege; AI cannot weaken security; critical ALLOW cannot rely on stale cache; cache is never authority; no secrets in telemetry; refresh reuse triggers security handling; MFA recovery cannot silently bypass policy; authorization remains centralized.

26. Traceability

Blueprint
↓
ESP-0001
↓
ADR-IDENTITY-001..069
↓
IS-IDENTITY-001
↓
TASK-001-01..024
↓
Source Code
↓
Tests
↓
CI/CD

27. Definition of Done

All applicable tasks complete or formally deferred; architecture/security/tenant/contract tests pass; quality gates pass; traceability is complete; documentation matches implementation. Do not mark Implemented or Validated without evidence from evoxa-platform.

28. Lifecycle

Draft → Refined → Approved → Implemented → Validated → Released → Archived

Current status: Refined
