# IS-IDENTITY-002 — User & Account Management

**Document ID:** IS-IDENTITY-002  
**Domain:** Identity  
**ESP:** ESP-0001 — Identity  
**Status:** Refined  
**Priority:** Critical  
**Version:** 1.1.0  
**Target Repository:** evoxa-platform  
**Documentation Repository:** evoxa-docs

---

# 1. Purpose

Establish the complete domain foundation and lifecycle management model for EVOXA Users and Accounts.

This Implementation Story defines:

- User identity
- Account lifecycle
- Account security state
- User attributes
- Credential boundary
- Account activation
- Account suspension
- Account disabling
- Account reactivation
- Account deletion/deactivation boundaries
- User security state
- User versioning
- Tenant-independent identity
- Organization membership relationship
- Audit and security event integration
- Domain event integration
- Repository contracts
- Concurrency and consistency requirements
- Authorization integration boundaries
- Authentication integration boundaries
- Testing and quality requirements

This story does not replace Authentication, Authorization, Membership, MFA, Recovery or Session Management. Those capabilities remain independently governed by their corresponding Implementation Stories.

---

# 2. Business Capability

Identity → User & Account Management

The User & Account Management capability provides the canonical identity representation for EVOXA.

It establishes the difference between:

```text
User Identity
        ↓
Account State
        ↓
Membership
        ↓
Authentication
        ↓
Authorization

The existence of a User does not automatically grant access to an Organization.

3. User Story

As the EVOXA Identity Domain, I need to manage users and their account security lifecycle so that every authentication, authorization and platform interaction operates against a consistent, secure and auditable user identity.

4. Feature Pack

Identity → User & Account Management

Includes:

User creation
User identity management
Account lifecycle management
Account state management
Account activation
Account suspension
Account disabling
Account reactivation
User attribute management
Credential boundary management
Account security state
User versioning
Audit integration
Security event integration
Domain event integration
Authentication integration
Membership integration
Authorization integration boundary
Concurrency protection
User/account testing
5. Scope
5.1 Included

This story covers:

User aggregate
Account lifecycle
User identity attributes
User status
Account security state
User repository contract
User creation
User update
User activation
User suspension
User disabling
User reactivation
Account lifecycle validation
Credential abstraction boundary
Authentication integration boundary
Membership relationship boundary
Tenant-independent User identity
Audit events
Security events
Domain events
Concurrency/versioning
Testing
Observability
Traceability
6. Out of Scope

The following capabilities are not fully implemented by this story:

Authentication workflow
Login
JWT issuance
Refresh token lifecycle
Session lifecycle
MFA factor management
MFA challenge verification
Password reset workflow
Authorization Policy Engine
RBAC evaluation
ABAC evaluation
Permission evaluation
Organization lifecycle
Membership lifecycle
Federation implementation
OAuth provider implementation
SCIM implementation
Frontend
Mobile application
Kubernetes deployment

These capabilities are handled by their respective Identity Implementation Stories and ADRs.

7. Architectural Context

Identity follows:

Presentation
      ↓
API
      ↓
Application
      ↓
Domain
      ↑
Infrastructure

The User and Account module must respect these boundaries.

The Domain layer must not depend directly on:

FastAPI
HTTP
PostgreSQL
Redis
JWT libraries
password hashing libraries
external identity providers
messaging infrastructure

Infrastructure implements technical concerns behind explicit contracts.

8. User Identity Model

The EVOXA User represents a global identity.

Conceptually:

User
├── Identity
├── Account State
├── Credential Boundary
├── Security State
├── Memberships
└── Audit History

A User is not equivalent to a tenant.

A User may have relationships with multiple Organizations through Memberships.

                 USER
                  │
        ┌─────────┼─────────┐
        │         │         │
   Membership  Membership  Membership
        │         │         │
        ▼         ▼         ▼
      Org A     Org B     Org C
9. User Aggregate

The User is the primary aggregate for account identity and lifecycle state.

Conceptual responsibilities:

Identity invariants
Account state transitions
Security state transitions
User attribute consistency
Version management
Domain event generation

The User aggregate must not directly perform:

Authentication
Authorization
Password hashing
JWT generation
MFA verification
Session creation
External provider communication

Those responsibilities belong to their respective modules.

10. User Identity

The User identity contains attributes required to represent the person/account within EVOXA.

Conceptually:

User
├── id
├── email
├── status
├── created_at
├── updated_at
└── version

Additional identity attributes may be introduced through governed evolution of the domain model.

The exact final persistence schema belongs to:

ESP-0001/11 — Identity Persistence & Data Model

and its associated implementation tasks.

11. User Identifier

User identifiers must be globally unique within EVOXA.

Conceptually:

UserId

The identifier must not depend on:

email
username
organization
external provider identifier

An external identity provider identifier must be represented separately from the canonical EVOXA User identity.

12. Email Identity

Email may participate in authentication and account communication, but it must not be treated as an authorization boundary.

The system must define a canonical normalization strategy before enforcing uniqueness.

Important rules:

Email must not be used as a tenant identifier.
Email must not implicitly establish organization membership.
Email-only matching must not automatically link external identities.
Email changes are security-sensitive operations.

Email verification is governed separately by the Identity communication/recovery capabilities.

13. Account Lifecycle

The canonical account lifecycle is:

PENDING
   │
   ▼
ACTIVE
   │
   ├──────────────► SUSPENDED
   │                    │
   │                    ▼
   │                 ACTIVE
   │
   └──────────────► DISABLED

The exact transition rules must be explicitly enforced by the User aggregate/application layer.

14. Account States
14.1 PENDING

The account exists but has not completed all requirements necessary for normal activation.

Possible causes include:

invitation
registration completion
email verification
administrative provisioning
external provisioning

PENDING must not automatically mean authenticated.

14.2 ACTIVE

The account is eligible for normal authentication subject to:

authentication requirements
security policy
MFA requirements
membership requirements
rate limiting
risk controls

Conceptually:

User = ACTIVE
        ↓
Authentication may continue
14.3 SUSPENDED

The account is temporarily blocked from normal authentication or other security-sensitive operations according to security policy.

Conceptually:

User = SUSPENDED
        ↓
Authentication denied

Suspension may trigger:

session revocation
refresh-token revocation
security event
audit event
authorization cache invalidation

The exact propagation behavior must follow the applicable Security and Session ADRs.

14.4 DISABLED

The account is disabled and must not be normally authenticated.

Conceptually:

User = DISABLED
        ↓
Authentication denied

A disabled account must not be reactivated implicitly by login.

Reactivation must be an explicit authorized operation.

15. Account State and Authentication

Authentication must validate account state before considering the user authenticated.

Conceptually:

Credentials
     ↓
User Lookup
     ↓
Account State
     ↓
Security Policy
     ↓
Authentication

Examples:

ACTIVE
  ↓
Continue
SUSPENDED
  ↓
Authentication denied
DISABLED
  ↓
Authentication denied

This relationship is explicitly required by the Identity authentication model.

Authentication remains responsible for the authentication decision itself.

User & Account Management owns the User state.

16. Account State vs Membership State

User state and Membership state are independent concepts.

Example:

User       = ACTIVE
Membership = ACTIVE
Organization = ACTIVE

→ Access may continue to authorization

But:

User       = ACTIVE
Membership = SUSPENDED
Organization = ACTIVE

→ Organization access blocked

And:

User       = ACTIVE
Membership = ACTIVE
Organization = SUSPENDED

→ Organization access blocked

Therefore:

User
+
Membership
+
Organization

must not be collapsed into a single state.

Authentication identifies the User.

Membership establishes the organization's relationship.

Authorization determines what the actor can do.

17. Account State Transitions

All lifecycle transitions must be explicit.

Conceptual commands:

CreateUser
ActivateUser
SuspendUser
DisableUser
ReactivateUser
UpdateUser

Each transition must validate:

Current state
Requested transition
Actor authorization
Tenant/platform context where applicable
Security policy
Concurrency version
Required audit/security behavior
18. User Creation

Conceptual flow:

Create User
    ↓
Validate Input
    ↓
Normalize Identity Data
    ↓
Validate Uniqueness
    ↓
Create User Aggregate
    ↓
Persist
    ↓
Create Domain Event
    ↓
Create Outbox Record
    ↓
Commit

Creation must not automatically:

create an organization
create arbitrary memberships
assign unrestricted roles
authenticate the user
issue tokens

Those operations belong to their corresponding bounded capabilities.

19. User Activation

Activation must be an explicit state transition.

Conceptually:

PENDING
   ↓
Activation Requirements
   ↓
Security Policy
   ↓
ACTIVE

Activation may depend on requirements such as:

verified email
invitation acceptance
administrative approval
provisioning state

The exact policy is not hard-coded into the domain unless mandated by platform security requirements.

20. User Suspension

Suspension is a security-sensitive operation.

Conceptually:

ACTIVE
   ↓
SuspendUser
   ↓
Security Policy / Authorization
   ↓
SUSPENDED

Suspension may require propagation to:

Sessions
Refresh Token Families
Authorization Cache
Security Events
Audit

The owning module remains responsible for its own lifecycle.

User Management must not directly manipulate another module's private tables.

21. User Disabling

Disabling is a stronger account state transition.

ACTIVE
   ↓
DisableUser
   ↓
DISABLED

A disabled account must not be authenticated normally.

Where required by security policy, disabling may result in:

User Disabled
      ↓
Session Revocation
      ↓
Refresh Token Revocation
      ↓
Security Event
      ↓
Audit
      ↓
Authorization Cache Invalidation

The User module emits the relevant fact/event.

Other modules react through their contracts.

22. User Reactivation

Reactivation must be explicit.

SUSPENDED / DISABLED
        ↓
Authorized Reactivation
        ↓
Security Policy
        ↓
ACTIVE

Reactivation must not automatically restore:

previous sessions
revoked refresh-token families
MFA trust
security exceptions
authorization cache entries

Security state must be reconstructed according to the corresponding module policies.

23. Credential Boundary

User & Account Management owns the account's credential relationship but does not own cryptographic implementation.

Conceptually:

User
 ↓
Credential Reference
 ↓
Authentication
 ↓
Password Verification Service

Passwords must never be stored in plaintext.

Password protection uses:

Argon2id

The Domain layer must not depend directly on a specific hashing library.

Credential implementation belongs to Infrastructure/Security Services.

24. Password Security

The system must never:

store plaintext passwords
return passwords
log passwords
emit passwords
include password hashes in API responses
include password hashes in domain/integration events
expose password material to AI agents

Password hashing parameters remain governed implementation details.

Password compromise may trigger:

session revocation
refresh-token revocation
forced credential reset
MFA requirements
account suspension

according to the Security Policy.

25. Account Security State

Account security state must be distinguishable from ordinary profile data.

Examples include:

Account Status
Security Status
Credential Status
MFA Status
Risk State

These states must not be collapsed into arbitrary boolean flags.

Security-sensitive state changes must be explicit, auditable and versioned.

26. Security Policy Integration

User & Account Management does not independently decide all security policy.

Conceptually:

Account Operation
       ↓
Security Policy Engine
       ↓
Decision

Possible decisions include:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

Policy evaluation remains centralized in the Identity Authorization/Security Policy architecture.

27. Authorization Boundary

Administrative User operations must be authorized.

Examples:

user.create
user.update
user.activate
user.suspend
user.disable
user.reactivate
user.delete

Permission naming follows:

<resource>.<action>

User Management must not implement its own independent RBAC/ABAC engine.

Authorization is delegated to the central Authorization Runtime.

28. Tenant Boundary

The User is globally identified.

Organization access is established through Membership.

Therefore:

User ≠ Organization
User ≠ Membership

A user administration operation may be:

platform-scoped
organization-scoped
self-scoped

The effective scope must be determined by authorization policy.

Cross-tenant access is denied by default.

29. Self-Service vs Administrative Operations

User operations may have different authorization contexts.

Examples:

Self-Service
    ↓
User updates own permitted profile data

versus:

Administrative
    ↓
Administrator suspends another user

versus:

Platform Administration
    ↓
Platform operator performs controlled cross-tenant operation

These operations must not share an implicit authorization shortcut.

30. User Ownership

Ownership may be used by Authorization as a contextual condition.

Example:

actor.user_id == resource.user_id

Ownership does not automatically grant access.

The Authorization Runtime determines whether ownership is sufficient under the active policy.

31. Concurrency

User lifecycle operations must be concurrency-safe.

Conceptually:

User.version

Optimistic concurrency may be used to prevent:

Request A
    ↓
Reads version 5

Request B
    ↓
Reads version 5

Request A
    ↓
Updates version 6

Request B
    ↓
Must not silently overwrite version 6

Conflicts must return a controlled concurrency error.

32. Domain Events

User lifecycle changes produce domain facts.

Examples:

UserCreated
UserUpdated
UserActivated
UserSuspended
UserDisabled
UserReactivated

Events must contain only information required for integration.

Events must never contain:

passwords
password hashes
access tokens
refresh tokens
MFA secrets
private keys
recovery codes
33. Security Events

Security-sensitive operations may generate security events.

Examples:

UserSuspended
UserDisabled
UserReactivated
UserCredentialChanged
UserSecurityStateChanged

Security events must support:

actor
tenant context where applicable
timestamp
target
operation
result
correlation
causation
trace context

Sensitive credential material is prohibited.

34. Audit Integration

Administrative and security-sensitive User operations must be auditable.

Audit must distinguish:

Actor
Target User
Operation
Result
Tenant
Policy
Reason
Timestamp
Correlation
Trace

Audit records are not authorization decisions.

Audit storage and retention follow the Identity Audit and Compliance specification.

35. Repository Contract

The User module exposes a repository contract rather than direct database access.

Conceptually:

UserRepository
├── get_by_id()
├── get_by_email()
├── add()
├── update()
└── exists()

Exact methods must be defined during Technical Tasks based on actual use cases.

Repository contracts must remain aggregate-oriented.

36. Persistence

PostgreSQL is the authoritative transactional store for User state.

The User module must not use Redis as the authoritative source.

Redis may be used for:

cache
temporary state
rate limiting
coordination

but not as the canonical User state.

37. Transactional Outbox

Critical User state changes follow:

User Aggregate Change
        ↓
Database Transaction
        ├── User State
        └── Outbox Event
        ↓
Commit
        ↓
Event Publisher

State and outbox records must commit atomically.

The User module must not directly publish external events before the transaction commits.

38. Event Delivery

Integration events use:

At-Least-Once delivery
idempotent consumers
versioned contracts
retry handling
DLQ handling
replay support

Consumers must not assume global event ordering.

Ordering is scoped according to event-stream requirements.

39. Cache Invalidation

User security-state changes may affect authorization cache.

Examples:

User Suspended
User Disabled
User Reactivated
Credential Security Change

The invalidation architecture uses:

User State Change
      ↓
Transactional Outbox
      ↓
Event Platform
      ↓
Invalidation Consumer
      ↓
Security Epoch / Version
      ↓
Cache Invalidation

The cache is not the source of truth.

Critical revocations must not depend exclusively on TTL.

40. Rate Limiting and Abuse Protection

User-related security operations must support protection against:

brute force
account enumeration
credential abuse
automated account creation
automated recovery abuse

Exact rate limits are not fixed in this story.

Security Policy and Runtime determine appropriate limits.

41. Enumeration Protection

Public operations must avoid revealing whether sensitive account information exists when that information could facilitate abuse.

Examples:

User exists

versus:

User does not exist

must not necessarily produce distinguishable responses in security-sensitive flows.

Internal systems may retain detailed information for audit and security analysis.

42. Account Deletion

Deletion must be treated as a governed lifecycle operation.

The system must distinguish between:

Disable
Suspend
Delete
Anonymize
Retain

Deletion must respect:

legal retention
audit requirements
security evidence
organizational relationships
downstream dependencies
event history

Physical deletion must not be assumed to be the default.

The final retention/deletion policy follows ESP-0001 persistence and retention requirements.

43. External Identity Relationship

External identities must not replace the canonical User identity.

Conceptually:

User
  │
  └── ExternalIdentity
          ├── provider
          └── subject

External identity linking must use explicit identity proof.

Email-only matching must not automatically link an external identity to an existing User.

Federation is implemented by the Federation capability.

44. AI Integration

AI agents may interact with User operations only through authorized Identity contracts.

AI agents are represented as:

AI_AGENT

They do not receive:

unrestricted User access
implicit administrative privileges
direct database access
credential material
security bypass capabilities

AI requests must pass through the same authorization model.

AI may assist with:

analysis
recommendations
anomaly detection
administrative suggestions

but cannot independently weaken account security.

45. Observability

User operations must emit structured telemetry where appropriate.

Telemetry may include:

operation
actor type
user identifier
organization context
result
duration
correlation ID
trace ID
error category

Telemetry must never contain:

passwords
password hashes
tokens
MFA secrets
recovery codes
private keys
46. Application Use Cases

Conceptual application services:

CreateUser
GetUser
UpdateUser
ActivateUser
SuspendUser
DisableUser
ReactivateUser

Additional use cases may be introduced through governed evolution.

Application services coordinate:

Validation
→ Authorization
→ Aggregate
→ Repository
→ Unit of Work
→ Audit/Event
47. Domain Services

Domain services should only be introduced where behavior cannot naturally belong to the User aggregate or application layer.

Potential boundaries include:

UserLifecycleService
UserSecurityService

They must not become generic service containers.

48. API Boundary

The public API belongs to the Identity API contract layer.

Conceptual namespace:

/api/v1/users

Potential operations:

POST   /api/v1/users
GET    /api/v1/users/{user_id}
PATCH  /api/v1/users/{user_id}
POST   /api/v1/users/{user_id}/activate
POST   /api/v1/users/{user_id}/suspend
POST   /api/v1/users/{user_id}/disable
POST   /api/v1/users/{user_id}/reactivate

These are conceptual boundaries.

The final OpenAPI contract is governed by:

ESP-0001/10 — Identity API Contracts

and IS-IDENTITY-014.

49. API Security

All administrative User endpoints must require authentication and authorization.

The API must validate:

Authentication
      ↓
Tenant Context
      ↓
Authorization
      ↓
Policy
      ↓
Criticality
      ↓
Freshness
      ↓
Operation

No endpoint may bypass centralized authorization.

50. Error Handling

Expected error categories include:

USER_NOT_FOUND
USER_ALREADY_EXISTS
INVALID_USER_STATE
INVALID_STATE_TRANSITION
USER_SUSPENDED
USER_DISABLED
UNAUTHORIZED
FORBIDDEN
TENANT_ACCESS_DENIED
CONCURRENCY_CONFLICT
VALIDATION_ERROR
SECURITY_POLICY_VIOLATION

Public messages must not expose sensitive implementation details.

51. Testing Requirements

Testing must include:

Unit Tests
User creation
State transitions
Invalid transitions
Identity validation
Version handling
Domain events
Invariants
Application Tests
CreateUser
UpdateUser
ActivateUser
SuspendUser
DisableUser
ReactivateUser
Security Tests
Unauthorized operation
Cross-tenant operation
Disabled account
Suspended account
Privilege escalation
AI actor restrictions
Enumeration protection
Concurrency Tests
Concurrent updates
Concurrent suspension
Concurrent activation/reactivation
Version conflicts
Integration Tests
PostgreSQL persistence
Transactional outbox
Event publication boundary
Cache invalidation integration
Contract Tests
API DTOs
Error envelope
Domain events
Integration events
52. Security Invariants

The following invariants are mandatory:

A disabled user cannot be normally authenticated.
A suspended user cannot be normally authenticated.
User state cannot be changed without authorized operation.
Cross-tenant access is denied by default.
User identity is independent from Organization identity.
Membership is not replaced by User identity.
Authorization remains centralized.
AI agents cannot bypass authorization.
Passwords are never stored in plaintext.
Password hashes are never returned through public APIs.
Credential material never appears in events or telemetry.
User state changes are concurrency-safe.
Critical state changes and outbox records are atomic.
Cache is never the authoritative User state.
Revocation/security changes cannot be weakened by cache TTL.
External identities cannot be linked by email alone.
Security-sensitive operations are auditable.
Unknown authorization/security errors fail closed.
Organization configuration cannot weaken platform security minimums.
User deletion cannot bypass retention/security requirements.
53. Dependencies
Depends on
IS-IDENTITY-001 — Identity Domain Foundation

Core ESP:

ESP-0001/01 — Identity Domain Model
ESP-0001/02 — Identity Architecture
ESP-0001/04 — User & Organization Management
ESP-0001/09 — Identity Audit & Compliance
ESP-0001/11 — Identity Persistence & Data Model
ESP-0001/12 — Identity Events & Integration
ESP-0001/13 — Identity Testing & Quality
ESP-0001/14 — Identity Implementation Requirements
ESP-0001/15 — Identity Deployment & Operational Requirements

Relevant ADR families include:

ADR-IDENTITY-001
ADR-IDENTITY-005
ADR-IDENTITY-006
ADR-IDENTITY-007
ADR-IDENTITY-012
ADR-IDENTITY-013
ADR-IDENTITY-014
ADR-IDENTITY-016
ADR-IDENTITY-017
ADR-IDENTITY-027
ADR-IDENTITY-028
ADR-IDENTITY-033
ADR-IDENTITY-034
ADR-IDENTITY-035
ADR-IDENTITY-042
ADR-IDENTITY-043
ADR-IDENTITY-045
ADR-IDENTITY-047
ADR-IDENTITY-050..059
ADR-IDENTITY-062..069
54. Required By
IS-IDENTITY-004 — Membership Management
IS-IDENTITY-005 — Authentication
IS-IDENTITY-006 — Authorization & Permission Evaluation
IS-IDENTITY-008 — Session Management
IS-IDENTITY-009 — Refresh Token Lifecycle
IS-IDENTITY-010 — Multi-Factor Authentication
IS-IDENTITY-011 — Security Policy Engine
IS-IDENTITY-012 — Audit & Security Events
IS-IDENTITY-014 — Identity API Contracts

Authentication explicitly depends on account state validation.

55. Technical Tasks

The story is decomposed into the following technical tasks:

TASK-002-01 — User Domain Model
TASK-002-02 — User Aggregate
TASK-002-03 — User Value Objects
TASK-002-04 — Account Lifecycle State Machine
TASK-002-05 — User Repository Contract
TASK-002-06 — User Creation
TASK-002-07 — User Update
TASK-002-08 — User Activation
TASK-002-09 — User Suspension
TASK-002-10 — User Disablement
TASK-002-11 — User Reactivation
TASK-002-12 — Credential Boundary
TASK-002-13 — Account Security State
TASK-002-14 — User Authorization Integration
TASK-002-15 — Tenant Scope Integration
TASK-002-16 — User Domain Events
TASK-002-17 — User Security Events
TASK-002-18 — User Audit Integration
TASK-002-19 — User Persistence
TASK-002-20 — User Concurrency Control
TASK-002-21 — User Cache Invalidation Integration
TASK-002-22 — User API Contracts
TASK-002-23 — User Testing
TASK-002-24 — User Security Testing
TASK-002-25 — User Integration Testing
TASK-002-26 — User Observability
TASK-002-27 — User Documentation & Traceability
TASK-002-28 — IS-IDENTITY-002 Validation
56. Acceptance Criteria
AC-001 — User Aggregate

Given the Identity Domain is initialized, when User behavior is implemented, then User is represented as an explicit aggregate with controlled lifecycle transitions.

AC-002 — User Identity

Given a User, when identity is persisted, then its canonical identifier is independent of email, Organization and external provider identifiers.

AC-003 — Account States

Given an account, when its state is evaluated, then PENDING, ACTIVE, SUSPENDED and DISABLED are represented explicitly.

AC-004 — Invalid Transitions

Given an invalid account-state transition, when requested, then the operation is rejected.

AC-005 — Active Account

Given a User with ACTIVE state, when Authentication evaluates the account, then authentication may continue subject to security policy.

AC-006 — Suspended Account

Given a User with SUSPENDED state, when Authentication evaluates the account, then normal authentication is denied.

AC-007 — Disabled Account

Given a User with DISABLED state, when Authentication evaluates the account, then normal authentication is denied.

AC-008 — Reactivation

Given a suspended or disabled account, when reactivation is requested, then the operation requires explicit authorization and policy validation.

AC-009 — Tenant Isolation

Given an organization-scoped User operation, when the actor lacks valid tenant context, then the operation cannot produce an unsafe ALLOW.

AC-010 — Authorization

Given an administrative User operation, when authorization is required, then the central Authorization Runtime evaluates the request.

AC-011 — Credential Security

Given User credential information, when it is persisted or processed, then plaintext passwords are never stored or exposed.

AC-012 — Events

Given a User lifecycle change, when the transaction commits, then the corresponding domain/outbox behavior is consistent with the event architecture.

AC-013 — Event Security

Given a User event, when serialized, then passwords, tokens, hashes and other credential material are absent.

AC-014 — Audit

Given a security-sensitive User operation, when completed, then appropriate audit information is available.

AC-015 — Concurrency

Given concurrent User updates, when versions conflict, then the system returns a controlled concurrency conflict rather than silently overwriting state.

AC-016 — Persistence

Given User state, when persisted, then PostgreSQL remains the authoritative transactional source.

AC-017 — Cache

Given Redis is unavailable or inconsistent, when User security state is required, then Redis failure cannot create an authorization bypass.

AC-018 — External Identity

Given an external identity, when linked to a User, then email-only matching is insufficient for automatic linking.

AC-019 — AI Security

Given an AI agent performs a User operation, when authorization is evaluated, then the AI agent uses the same security model as other actors.

AC-020 — API

Given User API operations, when contracts are exposed, then they use /api/v1 and the standard EVOXA error envelope.

AC-021 — Testing

Given the User module, when CI executes, then lifecycle, security, tenant and concurrency tests are executed.

AC-022 — Traceability

Given any User implementation task, when reviewed, then it can be traced to ESP-0001, applicable ADRs, this IS and the corresponding Technical Task.

AC-023 — Implementation Readiness

Given TASK-002-01 through TASK-002-28 are complete, when validation executes, then the User & Account Management capability is ready to support Membership and Authentication without bypassing Identity architecture.

57. Definition of Done

The story is considered complete only when:

User aggregate is implemented.
Account lifecycle is implemented.
State transitions are validated.
User repository contract is implemented.
PostgreSQL persistence is implemented.
Concurrency control is implemented.
Authorization integration is implemented.
Tenant boundary is enforced.
Credential boundary is secure.
Domain events are implemented.
Security events are implemented.
Audit integration is implemented.
Outbox integration is validated.
Cache invalidation behavior is validated where applicable.
API contracts are implemented.
Unit tests pass.
Integration tests pass.
Security tests pass.
Concurrency tests pass.
Contract tests pass.
Coverage meets Identity quality requirements.
CI quality gates pass.
Traceability is complete.
Architecture review passes.
Security review passes.
QA validation passes.
58. Traceability Matrix
Layer	Reference
Blueprint	Architecture Map / Identity
Domain	Identity
ESP	ESP-0001
Predecessor	IS-IDENTITY-001
Current IS	IS-IDENTITY-002
Source	apps/api/app/domains/identity/account/
Tests	apps/api/tests/domains/identity/account/
Authentication Consumer	IS-IDENTITY-005
Membership Consumer	IS-IDENTITY-004
Authorization Consumer	IS-IDENTITY-006
Persistence	ESP-0001/11
Events	ESP-0001/12
Testing	ESP-0001/13
Implementation	ESP-0001/14
Operations	ESP-0001/15
59. Architectural Model

The User & Account Management model is:

                         USER
                          │
              ┌───────────┴───────────┐
              │                       │
        Account State             Security State
              │                       │
       ┌──────┼──────┐                │
       │      │      │                │
    ACTIVE  SUSPENDED DISABLED        │
       │      │      │                │
       └──────┴──────┴────────────────┘
                          │
                          ▼
                     MEMBERSHIP
                          │
                          ▼
                    ORGANIZATION

Authentication consumes User account state.

Membership establishes organizational relationship.

Authorization determines effective access.

60. Fundamental Separation

The Identity architecture must preserve:

USER
=
Who the actor is
ACCOUNT STATE
=
Whether the account may operate
MEMBERSHIP
=
Which organization relationship exists
AUTHENTICATION
=
How the actor proves identity
AUTHORIZATION
=
What the actor may do
POLICY
=
Under what conditions the operation is permitted
SESSION
=
How authenticated state is maintained

This separation prevents User Management from becoming a monolithic security subsystem.

61. Lifecycle Status
Draft
   ↓
Refined
   ↓
Approved
   ↓
Implemented
   ↓
Validated
   ↓
Released
   ↓
Archived

Current Status: Refined

This story must not be marked Implemented or Validated until the actual implementation in evoxa-platform, its tests and CI evidence have been reviewed.

62. Final Result

IS-IDENTITY-002 establishes the canonical EVOXA User and Account foundation required by the rest of Identity.

The dependency model is:

IS-IDENTITY-001
Identity Domain Foundation
          ↓
IS-IDENTITY-002
User & Account Management
          ↓
IS-IDENTITY-003
Organization & Tenant Management
          ↓
IS-IDENTITY-004
Membership Management
          ↓
IS-IDENTITY-005
Authentication
          ↓
IS-IDENTITY-006
Authorization & Permission Evaluation

The most important architectural rule is:

User Identity
      ≠
Organization
      ≠
Membership
      ≠
Authentication
      ≠
Authorization

Each capability remains independently governed while participating in the common Identity security model.


**Este es el que usaría como reemplazo del `002` actual.** La fuente recuperada también confirma que el modelo previo ya contemplaba `User → Membership → Organization` y que el estado de cuenta debía ser validado antes de autenticación. :contentReference[oaicite:3]{index=3}

Una corrección importante respecto a documentos antiguos: **no debemos mantener `/api/v1/identity` como namespace**; la arquitectura consolidada que ya fijamos usa `/api/v1/users`, mientras que el contrato definitivo queda en `IS-IDENTITY-014`.
