# 28/05 — SDK Core

## 1. Document Purpose

SDK Core defines the foundational runtime and programming components shared by all official EVOXA SDKs.

It establishes the common technical foundation that allows different language SDKs to provide a consistent EVOXA development experience while remaining idiomatic for each programming language.

SDK Core is responsible for the capabilities that should not be repeatedly implemented by every domain SDK:

```text
Configuration
Authentication
Authorization Context
Transport
Serialization
Validation
Errors
Retries
Timeouts
Pagination
Idempotency
Telemetry
Tracing
Logging
Request Context
Middleware
Resilience
Versioning
```

The architectural objective is:

```text
                    SDK CORE
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   TypeScript       Python          Java
        │              │              │
        └──────────────┼──────────────┘
                       ▼
                EVOXA PLATFORM
```

SDK Core therefore becomes the **common technical foundation of the EVOXA SDK ecosystem**.

---

# 2. SDK Core Definition

SDK Core is the reusable infrastructure layer that sits below domain-specific SDK clients.

```text
Application
     │
     ▼
Domain SDK
     │
     ▼
SDK Core
     │
     ▼
Transport / Platform Interfaces
     │
     ▼
EVOXA Platform
```

Examples of Domain SDKs:

```text
Users SDK
Organizations SDK
Billing SDK
Analytics SDK
Workflow SDK
Events SDK
AI SDK
Agent SDK
Marketplace SDK
```

All should rely on the same Core principles.

---

# 3. SDK Core Mission

The mission of SDK Core is to provide:

* consistency;
* reliability;
* security;
* performance;
* observability;
* compatibility;
* extensibility;
* predictable developer behavior.

The Core should make complex platform behavior simple for application developers.

---

# 4. SDK Core Design Principle

The central principle is:

> **Build platform complexity once in SDK Core and expose a simple, consistent developer experience above it.**

```text
Complexity
     ↓
SDK Core
     ↓
Simple API
```

---

# 5. SDK Core Responsibilities

SDK Core owns common capabilities:

```text
Configuration
Authentication
Transport
Request Building
Response Handling
Serialization
Validation
Errors
Retries
Timeouts
Pagination
Idempotency
Middleware
Telemetry
Tracing
Logging
Resilience
```

It should not own domain-specific business logic.

---

# 6. SDK Core Non-Responsibilities

SDK Core should not directly implement:

```text
Billing Rules
User Business Logic
Enterprise Workflows
Domain Policies
Marketplace Business Rules
AI Model Logic
Agent Planning
```

Those belong to higher-level SDK modules and platform services.

---

# 7. SDK Core Architecture

```text
┌─────────────────────────────────────────────┐
│              Domain SDKs                    │
│ API / Billing / AI / Agents / Events        │
├─────────────────────────────────────────────┤
│              SDK Core                       │
├─────────────────────────────────────────────┤
│ Configuration                               │
│ Authentication                              │
│ Request / Response                          │
│ Serialization                               │
│ Validation                                  │
│ Errors                                      │
│ Retry / Timeout / Idempotency               │
│ Middleware                                  │
│ Telemetry / Logging / Tracing               │
├─────────────────────────────────────────────┤
│              Transport                      │
├─────────────────────────────────────────────┤
│          EVOXA API Platform                 │
└─────────────────────────────────────────────┘
```

---

# 8. SDK Core Modules

SDK Core can be divided into:

```text
core/
├── config
├── auth
├── transport
├── request
├── response
├── serialization
├── validation
├── errors
├── retry
├── timeout
├── pagination
├── idempotency
├── middleware
├── telemetry
├── tracing
├── logging
├── resilience
└── versioning
```

---

# 9. Configuration Core

Configuration provides a single standardized mechanism for initializing SDK clients.

Example conceptual model:

```text
SDK Client
 ├── endpoint
 ├── credentials
 ├── environment
 ├── timeout
 ├── retry policy
 ├── telemetry
 ├── region
 └── tenant context
```

---

# 10. Configuration Sources

Configuration may come from:

```text
Environment Variables
Configuration Files
Application Code
Secret Providers
Managed Identity
Runtime Configuration
```

Configuration precedence must be explicitly defined.

---

# 11. Environment Configuration

SDK Core must support:

```text
development
testing
staging
production
```

The selected environment should determine appropriate defaults without silently creating security risks.

---

# 12. Environment Isolation

The Core should help prevent accidental use of production credentials against development or testing environments.

```text
Environment
     ↓
Endpoint
     ↓
Credential
```

The relationship must remain explicit.

---

# 13. Endpoint Management

SDK Core provides endpoint resolution.

```text
SDK
 ↓
Endpoint Resolver
 ↓
Regional / Environment Endpoint
```

Potential endpoint types:

```text
Public
Regional
Sandbox
Private
Enterprise
```

---

# 14. Region Configuration

Where supported, SDK Core should allow:

```text
Region
Data Residency
Endpoint
```

to be selected according to organization and tenant policies.

---

# 15. Credential Provider

Credentials should be resolved through an abstraction.

```text
CredentialProvider
       │
 ┌─────┼─────────────┐
 ▼     ▼             ▼
Env   Vault      Workload Identity
```

The application should not need to know how credentials are stored.

---

# 16. Authentication Core

Authentication provides the mechanism through which SDK requests establish identity.

```text
Application
 ↓
SDK Auth
 ↓
Credential
 ↓
Access Token
 ↓
API
```

---

# 17. Authentication Abstraction

SDK Core should support multiple authentication mechanisms without changing domain SDK interfaces.

Potential mechanisms:

```text
OAuth 2.0
API Keys
Service Accounts
Workload Identity
Managed Identity
JWT
```

The exact supported mechanisms depend on EVOXA Identity Platform.

---

# 18. Token Management

Where token-based authentication is used, Core should manage:

```text
Token Acquisition
Token Caching
Token Expiration
Token Refresh
Token Revocation
```

---

# 19. Token Refresh

The conceptual flow:

```text
Request
 ↓
Access Token
 ↓
Expired / Near Expiration
 ↓
Refresh
 ↓
New Token
 ↓
Request
```

Concurrent requests should avoid unnecessary duplicate refresh operations.

---

# 20. Authentication Failure

Authentication failures must generate standardized errors.

```text
Invalid Credential
Expired Credential
Missing Credential
Invalid Token
Revoked Token
```

---

# 21. Authorization Context

SDK Core should propagate authorization context where required.

```text
Identity
+
Tenant
+
Organization
+
Scopes
+
Policies
```

Core does not replace server-side authorization.

---

# 22. Tenant Context

Tenant context should be available to requests when the API contract requires it.

```text
Organization
 ↓
Tenant
 ↓
Application
 ↓
SDK
 ↓
API
```

---

# 23. Request Context

Each SDK operation should have a request context.

Conceptually:

```text
RequestContext
 ├── requestId
 ├── correlationId
 ├── traceId
 ├── tenantId
 ├── organizationId
 └── metadata
```

---

# 24. Request Builder

SDK Core should provide a standardized request construction process.

```text
Domain Method
 ↓
Request Builder
 ↓
Headers
 ↓
Authentication
 ↓
Query
 ↓
Body
 ↓
Transport
```

---

# 25. Request Headers

Core-managed headers may include:

```text
Authorization
Content-Type
Accept
User-Agent
SDK-Version
Request-ID
Trace Context
Idempotency-Key
```

Only applicable headers should be included.

---

# 26. User-Agent Strategy

SDK requests should identify:

```text
EVOXA SDK
Language
SDK Version
Runtime
```

This assists support, telemetry and compatibility analysis.

---

# 27. Transport Core

Transport provides network communication abstraction.

```text
SDK Core
 ↓
Transport Interface
 ↓
HTTP Client
 ↓
Network
```

---

# 28. Transport Interface

The domain SDK should not depend directly on a specific HTTP library.

Conceptually:

```text
Transport
 ├── send()
 ├── stream()
 └── close()
```

The actual implementation can differ by language.

---

# 29. HTTP Transport

HTTP/HTTPS should be the primary transport for REST-style APIs.

Core handles:

```text
Connection
Headers
Methods
Status Codes
Body
Timeouts
```

---

# 30. Streaming Transport

Streaming should be supported for workloads such as:

```text
AI Tokens
Events
Long-Running Operations
Realtime Updates
```

---

# 31. WebSocket Support

Where EVOXA services expose realtime communication, Core may provide a standardized realtime transport abstraction.

---

# 32. Serialization Core

Serialization translates between application models and transport payloads.

```text
Application Object
        ↕
SDK Model
        ↕
Serialized Payload
```

---

# 33. JSON Serialization

JSON is the primary serialization format for standard EVOXA APIs where applicable.

Core should consistently handle:

```text
Objects
Arrays
Enums
Dates
Nullable Values
Nested Objects
```

---

# 34. Date and Time Handling

SDK Core should provide consistent representations for:

```text
Timestamp
Date
Time
Duration
Timezone
```

Language-specific representations may differ.

---

# 35. Enum Handling

Enums should preserve API compatibility while remaining idiomatic for each language.

Unknown enum values should be handled safely when possible.

---

# 36. Nullable Fields

Core must distinguish:

```text
Missing
Null
Empty
Zero
False
```

when the API contract gives them different semantics.

---

# 37. Validation Core

Validation should catch invalid requests before network transmission whenever possible.

```text
Developer Input
 ↓
Validation
 ↓
Request
```

---

# 38. Validation Responsibilities

Core validation may verify:

```text
Required Fields
Types
Formats
Ranges
Enum Values
Structural Constraints
```

Business validation remains server-side.

---

# 39. Client vs Server Validation

```text
Client Validation
 ↓
Fast Feedback

Server Validation
 ↓
Authoritative Rule
```

The SDK must never assume client-side validation is sufficient for security.

---

# 40. Error Core

SDK Core provides a common error model.

```text
SDKError
 ├── AuthenticationError
 ├── AuthorizationError
 ├── ValidationError
 ├── NotFoundError
 ├── ConflictError
 ├── RateLimitError
 ├── TimeoutError
 ├── NetworkError
 └── ServerError
```

---

# 41. Error Metadata

Errors should expose useful structured metadata where available:

```text
Code
Message
HTTP Status
Request ID
Correlation ID
Retryable
Details
```

---

# 42. Error Preservation

The SDK must not discard meaningful server error information.

Developers should be able to diagnose failures without inspecting raw network traffic.

---

# 43. Retry Core

Retry behavior must be centralized.

```text
Request
 ↓
Failure
 ↓
Retry Policy
 ↓
Backoff
 ↓
Retry
```

---

# 44. Retryable Errors

Typical retry candidates may include:

```text
Temporary Network Failure
Timeout
503
429
Transient Infrastructure Failure
```

The actual retryability must follow service contracts.

---

# 45. Non-Retryable Errors

Examples:

```text
Invalid Authentication
Invalid Request
Permission Denied
Resource Not Found
Business Validation Failure
```

should generally not be blindly retried.

---

# 46. Exponential Backoff

Retry delays should normally use controlled backoff.

```text
Attempt 1
 ↓
Short Delay
 ↓
Attempt 2
 ↓
Longer Delay
 ↓
Attempt 3
```

Jitter should be supported to prevent synchronized retry storms.

---

# 47. Retry Limits

Every retry policy must have explicit limits.

```text
Maximum Attempts
Maximum Delay
Maximum Total Duration
```

---

# 48. Retry Safety

Retries must consider idempotency.

```text
Safe Operation
 ↓
Retry

Unsafe Operation
 ↓
Idempotency Protection
```

---

# 49. Timeout Core

Timeouts protect applications from indefinite waiting.

Core should support:

```text
Connect Timeout
Read Timeout
Write Timeout
Request Timeout
Operation Timeout
```

---

# 50. Timeout Propagation

Timeout configuration should be propagated consistently through the request lifecycle.

---

# 51. Cancellation

Long-running operations should support cancellation where the language runtime allows it.

```text
Operation
 ↓
Cancellation Signal
 ↓
Transport
 ↓
Stop
```

---

# 52. Idempotency Core

Idempotency prevents duplicate logical operations.

```text
Client
 ↓
Idempotency Key
 ↓
API
 ↓
Single Logical Operation
```

---

# 53. Idempotency Key Generation

The SDK may provide automatic key generation for operations explicitly marked as idempotent-capable.

Developers should also be able to supply their own keys.

---

# 54. Pagination Core

Pagination should be abstracted consistently.

Supported models may include:

```text
Cursor
Page
Token
Continuation
```

---

# 55. Iterator Model

Where supported by the language, SDK Core should expose idiomatic iteration.

Conceptually:

```text
for item in client.resources.list():
    process(item)
```

This hides pagination mechanics.

---

# 56. Pagination Safety

The SDK must avoid:

```text
Infinite Pagination
Duplicate Pages
Missing Pages
Unexpected Memory Growth
```

---

# 57. Middleware Core

Middleware provides extension points around requests.

```text
Before Request
      ↓
Middleware
      ↓
Transport
      ↓
After Response
```

---

# 58. Middleware Types

Potential middleware:

```text
Authentication
Logging
Tracing
Metrics
Retry
Caching
Validation
Rate Limit Handling
```

---

# 59. Middleware Ordering

Middleware ordering must be deterministic.

```text
Request
 ↓
Context
 ↓
Authentication
 ↓
Validation
 ↓
Telemetry
 ↓
Retry
 ↓
Transport
```

The exact order may vary by implementation.

---

# 60. Custom Middleware

Advanced developers may register custom middleware.

However, custom middleware must not bypass mandatory security controls.

---

# 61. Telemetry Core

Telemetry connects SDK activity to platform observability.

```text
SDK Operation
 ↓
Telemetry
 ↓
Metrics / Logs / Traces
```

---

# 62. Metrics

SDK Core should provide metrics such as:

```text
Request Count
Success Count
Error Count
Latency
Retry Count
Timeout Count
```

---

# 63. Tracing

Distributed tracing should propagate context across:

```text
Application
 ↓
SDK
 ↓
API Gateway
 ↓
Service
```

---

# 64. Trace Context

The SDK should support standardized trace propagation where EVOXA observability infrastructure supports it.

---

# 65. Logging Core

Logging should be:

```text
Structured
Configurable
Safe
Environment-Aware
```

---

# 66. Sensitive Data Protection

SDK Core must prevent accidental logging of:

```text
Access Tokens
Secrets
Passwords
Private Keys
Sensitive Payloads
```

unless explicitly enabled under safe debugging controls.

---

# 67. Debug Mode

Debugging can expose additional diagnostic information while respecting security boundaries.

```text
Normal
 ↓
Debug
 ↓
Trace
```

Each level should have clear behavior.

---

# 68. Resilience Core

SDK Core should provide resilience primitives:

```text
Timeout
Retry
Backoff
Circuit Breaker
Bulkhead
Cancellation
Idempotency
```

---

# 69. Circuit Breaker

A circuit breaker can prevent repeated requests to an unhealthy dependency.

```text
Healthy
 ↓
Failures
 ↓
Open
 ↓
Cooldown
 ↓
Half-Open
 ↓
Healthy
```

---

# 70. Bulkhead Isolation

Applications should be protected from one failing dependency exhausting all resources.

---

# 71. Connection Management

Transport implementations should support:

```text
Connection Pooling
Keep-Alive
Connection Reuse
Pool Limits
Connection Cleanup
```

---

# 72. Concurrency

SDK Core should safely support concurrent requests where the language and client implementation permit it.

Thread-safety expectations must be documented.

---

# 73. Async Support

Languages with asynchronous programming models should receive idiomatic async APIs.

```text
Async Client
 ↓
Await
 ↓
Result
```

---

# 74. Sync Support

Where appropriate, SDKs may also provide synchronous clients.

The underlying semantics should remain equivalent.

---

# 75. Resource Management

Clients should support deterministic cleanup.

```text
Create Client
 ↓
Use
 ↓
Close
```

Language-specific resource-management patterns should be supported.

---

# 76. Connection Lifecycle

The Core manages:

```text
Initialize
Connect
Reuse
Recover
Close
```

---

# 77. Caching

Caching may be supported for explicitly cacheable operations.

The SDK must never cache sensitive resources without clear policy.

---

# 78. Cache Policy

Cache behavior should define:

```text
TTL
Scope
Invalidation
Maximum Size
Consistency
```

---

# 79. Request Deduplication

For safe read operations, Core may optionally deduplicate concurrent identical requests.

---

# 80. Rate Limit Handling

SDK Core should interpret server rate-limit information where available.

```text
Rate Limit
 ↓
Retry Metadata
 ↓
Retry Policy
```

---

# 81. Quota Awareness

The SDK may expose quota information to applications.

```text
Used
Limit
Remaining
Reset
```

---

# 82. Request Budget

Applications may impose their own request budgets.

```text
Application Budget
 ↓
SDK Core
 ↓
Request
```

---

# 83. SDK Usage Metering

Core should expose sufficient telemetry to support platform usage metering.

```text
Request
 ↓
Usage Event
 ↓
Metering
```

---

# 84. Billing Context

Where applicable, requests should preserve the information required for billing attribution.

```text
Organization
Tenant
Application
Capability
```

---

# 85. Cost Context

AI and agent operations may additionally require:

```text
Model
Agent
Tool
Workflow
```

to support accurate cost allocation.

---

# 86. SDK Core Security Model

```text
Credential
 ↓
Authentication
 ↓
Request Context
 ↓
Authorization
 ↓
Policy
 ↓
Transport
 ↓
Platform
```

Security controls must be applied consistently.

---

# 87. Secure Defaults

SDK Core should use secure defaults for:

```text
TLS
Credential Handling
Logging
Timeouts
Certificate Validation
```

---

# 88. TLS

Production communication must use secure transport.

Certificate validation should not be disabled by default.

---

# 89. Credential Rotation

Credential providers should support rotation without requiring application redesign.

---

# 90. Secret Redaction

Core should automatically redact sensitive values from diagnostic output.

---

# 91. Supply Chain Security

SDK packages should be distributed with:

```text
Integrity
Signature
Version
Dependency Metadata
SBOM
```

where supported.

---

# 92. Dependency Management

SDK Core dependencies should be minimized and regularly reviewed.

Reducing dependencies decreases:

```text
Attack Surface
Compatibility Risk
Package Size
Maintenance Cost
```

---

# 93. SDK Core Versioning

Core versioning must preserve compatibility with domain SDKs.

```text
Domain SDK
 ↓
Core Version
 ↓
Runtime
```

---

# 94. Core Compatibility

A domain SDK should declare the compatible Core version range.

---

# 95. Internal vs Public APIs

SDK Core should distinguish:

```text
Public API
Internal API
Experimental API
```

Internal implementation details must not become accidental public contracts.

---

# 96. Stable Core Interfaces

The most important Core interfaces should remain stable:

```text
Transport
Credentials
Configuration
Middleware
Errors
Telemetry
```

---

# 97. Extension Points

SDK Core should provide controlled extension points.

```text
Core
 ↓
Interface
 ↓
Custom Implementation
```

---

# 98. Custom Transport

Advanced environments may require custom transports.

Examples:

```text
Proxy
Private Network
Enterprise Gateway
Testing Transport
```

---

# 99. Mock Transport

Testing environments should be able to replace network transport.

```text
Application
 ↓
SDK
 ↓
Mock Transport
```

This enables deterministic unit tests.

---

# 100. Testing SDK Core

Core must be tested independently of domain SDKs.

Testing categories:

```text
Unit
Integration
Contract
Concurrency
Security
Performance
Compatibility
```

---

# 101. Contract Testing

Core should validate assumptions against the EVOXA API contracts.

---

# 102. Error Testing

Every known API error class should have standardized mapping tests.

---

# 103. Retry Testing

Retry behavior must test:

```text
Retryable
Non-Retryable
Backoff
Jitter
Maximum Attempts
Timeout
```

---

# 104. Authentication Testing

Authentication tests should cover:

```text
Valid Token
Expired Token
Refresh
Invalid Token
Missing Credential
Credential Rotation
```

---

# 105. Serialization Testing

Serialization tests should include:

```text
Valid Data
Null Values
Missing Fields
Unknown Fields
Nested Objects
Dates
Enums
```

---

# 106. Concurrency Testing

Core should test:

```text
Concurrent Requests
Concurrent Token Refresh
Connection Pooling
Shared Client
Cancellation
```

---

# 107. Performance Testing

SDK Core performance should measure:

```text
Request Overhead
Serialization Cost
Memory Usage
Connection Reuse
Throughput
```

---

# 108. SDK Core Footprint

The Core should remain lightweight.

Avoid adding heavy dependencies simply for convenience.

---

# 109. Startup Performance

SDK initialization should be fast.

Avoid expensive initialization unless explicitly required.

---

# 110. Memory Management

Core should avoid unnecessary retention of:

```text
Requests
Responses
Logs
Telemetry
Cached Data
```

---

# 111. Streaming Memory Model

Streaming responses should be processed incrementally rather than loaded entirely into memory where possible.

---

# 112. Large Payload Handling

Core should support efficient handling of large responses and uploads.

---

# 113. File Transfer

Where EVOXA APIs support files, Core may expose standardized streaming upload/download abstractions.

---

# 114. Compression

Transport may support compression where beneficial and supported by the platform.

---

# 115. Proxy Support

Enterprise environments may require proxy configuration.

SDK Core should support configurable proxy behavior.

---

# 116. Network Configuration

Advanced network configuration may include:

```text
Proxy
DNS
TLS
Timeout
Connection Pool
Private Endpoint
```

---

# 117. Localization Context

Where APIs support localization, request context may include:

```text
Locale
Timezone
Language
```

---

# 118. User-Agent and Runtime Metadata

The SDK can expose runtime information for diagnostics:

```text
Language
SDK Version
Runtime Version
Operating Environment
```

Sensitive host information should not be exposed unnecessarily.

---

# 119. Request Metadata

Applications may attach safe metadata to requests where supported.

---

# 120. Correlation Architecture

Core should maintain:

```text
Request ID
Correlation ID
Trace ID
Operation ID
```

across supported operations.

---

# 121. Long-Running Operations

Core should provide a generic abstraction:

```text
Start
 ↓
Operation ID
 ↓
Poll / Stream
 ↓
Status
 ↓
Result
```

---

# 122. Operation Polling

Polling should respect:

```text
Backoff
Timeout
Cancellation
Retry
```

---

# 123. Operation Cancellation

Where platform APIs support cancellation:

```text
Application
 ↓
Cancel
 ↓
SDK Core
 ↓
Platform
```

---

# 124. Event Integration

SDK Core should provide the shared infrastructure needed by event-specific SDK modules.

```text
Core Transport
 ↓
Event Client
```

---

# 125. Webhook Verification

Core can provide shared cryptographic and request-validation primitives for webhook modules.

---

# 126. Signature Validation

Signature verification must use secure, constant-time comparison where appropriate.

---

# 127. SDK Core and AI

AI SDK builds on Core:

```text
AI SDK
 ↓
SDK Core
 ↓
Transport
 ↓
AI Gateway
```

---

# 128. SDK Core and Streaming AI

AI streaming uses Core transport and telemetry abstractions.

```text
AI Request
 ↓
Streaming Transport
 ↓
Incremental Output
```

---

# 129. SDK Core and Agents

Agent SDK uses Core for:

```text
Authentication
Transport
Tools
Telemetry
Errors
Policies
Usage
```

---

# 130. SDK Core and Tools

Tools can use standard Core request and response abstractions.

```text
Tool
 ↓
SDK Core
 ↓
API
```

---

# 131. SDK Core and Marketplace

Marketplace SDK clients rely on Core for:

```text
Authentication
Transport
Errors
Pagination
Telemetry
```

---

# 132. SDK Core and Extensions

Extensions should consume stable Core interfaces rather than internal implementation details.

---

# 133. SDK Core and CLI

CLI should reuse Core components where appropriate.

```text
CLI
 ↓
SDK Core
 ↓
Control APIs
```

This reduces duplicated platform logic.

---

# 134. SDK Core and Developer Portal

Portal backend services may use the same API contracts and conceptual Core patterns.

---

# 135. Cross-Language Core Model

Although implementations differ by language, the conceptual model should remain equivalent.

```text
TypeScript Core
Python Core
Java Core
Go Core
C# Core
Dart Core
```

---

# 136. Language-Specific Implementation

Each language can use native conventions.

For example:

```text
TypeScript → Promises / async-await
Python → async / sync
Java → CompletableFuture / synchronous APIs
Go → context.Context
C# → Task / async-await
Dart → Future / Stream
```

The platform semantics remain consistent.

---

# 137. Common Semantic Contract

All language SDKs should preserve:

```text
Authentication Semantics
Error Semantics
Retry Semantics
Pagination Semantics
Timeout Semantics
Telemetry Semantics
Versioning Semantics
```

---

# 138. SDK Core Package Model

Conceptually:

```text
@evoxa/core
```

or equivalent package naming per language.

Domain packages then depend on Core.

```text
@evoxa/core
@evoxa/api
@evoxa/ai
@evoxa/agents
```

The exact naming convention belongs to SDK packaging standards.

---

# 139. Package Dependency Model

```text
Application
 ├── Domain SDK
 │      ↓
 │   SDK Core
 │
 └── Optional Extensions
```

---

# 140. Core Dependency Rule

Domain SDKs should not duplicate Core infrastructure unless there is a justified language-specific requirement.

---

# 141. Core API Surface

Core public APIs should remain intentionally small.

```text
Client
Config
Credentials
Transport
Errors
Middleware
Telemetry
```

---

# 142. Developer Simplicity

Developers should normally interact with:

```text
Domain Client
```

rather than directly manipulating Core internals.

---

# 143. Advanced Developer Access

Advanced users may access Core configuration when required.

```text
Simple Mode
 ↓
Advanced Configuration
```

---

# 144. Default Client

SDKs should provide a straightforward default client configuration.

Conceptually:

```text
Client()
```

should be sufficient for common development scenarios once credentials are configured.

---

# 145. Explicit Client Configuration

Advanced applications should be able to configure:

```text
Endpoint
Credentials
Timeout
Retry
Telemetry
Proxy
Region
```

---

# 146. Multiple Clients

Applications may create multiple SDK clients for:

```text
Different Tenants
Different Credentials
Different Regions
Different Environments
```

---

# 147. Client Isolation

Clients should not unintentionally share:

```text
Credentials
Tenant Context
Cache
Telemetry
Configuration
```

unless explicitly designed to do so.

---

# 148. Thread Safety

Thread-safety guarantees must be documented for each language implementation.

---

# 149. Async Safety

Concurrent asynchronous operations must not corrupt:

```text
Token State
Request State
Configuration
Telemetry
```

---

# 150. Resource Lifecycle

A client lifecycle should be predictable:

```text
Initialize
 ↓
Authenticate
 ↓
Request
 ↓
Observe
 ↓
Close
```

---

# 151. Core Observability

SDK Core should make observability available without requiring developers to implement it manually.

```text
SDK Core
 ↓
Telemetry
 ↓
Platform Observability
```

---

# 152. OpenTelemetry Alignment

Where appropriate, Core should align with industry-standard telemetry concepts rather than inventing incompatible models.

---

# 153. Metrics Export

Applications may export SDK metrics to:

```text
EVOXA
External Observability Systems
```

according to configuration.

---

# 154. Logging Export

Structured logs can be integrated with application logging systems.

---

# 155. Trace Export

Distributed traces should integrate with supported tracing systems.

---

# 156. Privacy-Aware Telemetry

Telemetry must respect:

```text
Privacy
Data Residency
Security
Compliance
```

---

# 157. Telemetry Sampling

High-volume applications should be able to configure sampling.

```text
100%
 ↓
10%
 ↓
1%
```

according to operational requirements.

---

# 158. Telemetry Cost Control

Observability itself has a cost.

Core should allow control over:

```text
Log Level
Trace Sampling
Metric Frequency
Payload Capture
```

---

# 159. Diagnostic Mode

A controlled diagnostic mode can temporarily increase observability for troubleshooting.

---

# 160. Core Governance

Changes to SDK Core have ecosystem-wide impact.

Therefore Core changes require stronger governance than ordinary domain SDK changes.

---

# 161. Core Change Classification

Changes should be classified as:

```text
Patch
Minor
Major
Security
Breaking
```

---

# 162. Core Breaking Changes

Breaking Core changes can affect every domain SDK.

Therefore they require:

```text
Migration Plan
Compatibility Analysis
Release Notes
Testing
Communication
```

---

# 163. Core Release Strategy

A release should pass:

```text
Build
 ↓
Unit
 ↓
Integration
 ↓
Contract
 ↓
Security
 ↓
Compatibility
 ↓
Performance
 ↓
Documentation
 ↓
Release
```

---

# 164. Core Security Gates

Before release:

```text
Dependency Scan
SAST
Secret Scan
Supply Chain Validation
Artifact Signing
```

where applicable.

---

# 165. Core Documentation

Documentation should include:

```text
Configuration
Authentication
Transport
Errors
Retries
Timeouts
Pagination
Telemetry
Security
Advanced Usage
```

---

# 166. Core Examples

Examples should demonstrate:

```text
Basic Client
Authentication
Custom Configuration
Error Handling
Retries
Async Usage
Streaming
Telemetry
Testing
```

---

# 167. Core Troubleshooting

Troubleshooting should expose:

```text
Authentication Problems
Network Problems
Timeouts
Rate Limits
Serialization Errors
Version Conflicts
```

---

# 168. Core Diagnostic Information

Errors should provide enough information to answer:

```text
What happened?
Where did it happen?
Why did it happen?
Can I retry?
What should I do next?
```

---

# 169. Core Compatibility Matrix

EVOXA should maintain compatibility information across:

```text
SDK Core
Domain SDK
API Version
Language Runtime
Operating Environment
```

---

# 170. Core Lifecycle

```text
Design
 ↓
Implementation
 ↓
Testing
 ↓
Preview
 ↓
GA
 ↓
Maintenance
 ↓
Deprecation
 ↓
Retirement
```

---

# 171. Core Deprecation

Deprecated Core interfaces must provide:

```text
Replacement
Migration Guide
Deprecation Date
Retirement Date
```

---

# 172. Core Architecture Diagram

```text
                           APPLICATION
                                │
                                ▼
                         DOMAIN SDK
                                │
                                ▼
                       ┌─────────────────┐
                       │    SDK CORE     │
                       ├─────────────────┤
                       │ Configuration   │
                       │ Authentication  │
                       │ Context         │
                       │ Requests        │
                       │ Responses       │
                       │ Serialization   │
                       │ Validation      │
                       │ Errors         │
                       │ Retry           │
                       │ Timeout         │
                       │ Pagination      │
                       │ Idempotency     │
                       │ Middleware      │
                       │ Telemetry       │
                       │ Tracing         │
                       │ Logging         │
                       │ Resilience      │
                       └────────┬────────┘
                                │
                                ▼
                           TRANSPORT
                                │
                                ▼
                         EVOXA API LAYER
                                │
                                ▼
                        EVOXA PLATFORM
```

---

# 173. SDK Core Security Architecture

```text
                     APPLICATION
                           │
                           ▼
                    SDK CORE
                           │
                 ┌─────────┴─────────┐
                 ▼                   ▼
          Credential Provider    Request Context
                 │                   │
                 ▼                   ▼
          Authentication       Tenant / Identity
                 │                   │
                 └─────────┬─────────┘
                           ▼
                      Authorization
                           │
                           ▼
                         Policy
                           │
                           ▼
                       Transport
                           │
                           ▼
                       EVOXA API
```

---

# 174. SDK Core Request Lifecycle

```text
Application
     │
     ▼
Domain Method
     │
     ▼
Validation
     │
     ▼
Request Construction
     │
     ▼
Context Injection
     │
     ▼
Authentication
     │
     ▼
Middleware
     │
     ▼
Retry / Resilience
     │
     ▼
Transport
     │
     ▼
API
     │
     ▼
Response
     │
     ▼
Deserialization
     │
     ▼
Telemetry
     │
     ▼
Domain Result
```

---

# 175. SDK Core Error Lifecycle

```text
Platform Error
 ↓
Transport
 ↓
Error Mapper
 ↓
SDK Error
 ↓
Developer
```

The mapping should preserve relevant diagnostic information.

---

# 176. SDK Core Retry Lifecycle

```text
Request
 ↓
Failure
 ↓
Classify
 ↓
Retryable?
 ├── No → Error
 └── Yes
       ↓
    Backoff
       ↓
    Retry
       ↓
    Success / Failure
```

---

# 177. SDK Core Authentication Lifecycle

```text
Application
 ↓
Credential Provider
 ↓
Token
 ↓
Cache
 ↓
Request
 ↓
Expiration
 ↓
Refresh
 ↓
Retry Request
```

---

# 178. SDK Core Telemetry Lifecycle

```text
SDK Operation
 ↓
Create Span
 ↓
Request Metrics
 ↓
Transport
 ↓
Response
 ↓
Record Result
 ↓
Close Span
```

---

# 179. SDK Core and Enterprise Architecture

SDK Core integrates with enterprise capabilities:

```text
Identity
Security
Tenancy
Policy
Audit
Billing
FinOps
Observability
```

This ensures that SDK applications remain first-class EVOXA platform participants.

---

# 180. SDK Core and Multi-Tenant Architecture

```text
Developer
 ↓
Application
 ↓
SDK Core
 ↓
Tenant Context
 ↓
API Gateway
 ↓
Tenant-Aware Service
```

---

# 181. SDK Core and Policy

```text
Request
 ↓
Identity
 ↓
Tenant
 ↓
Policy
 ↓
Permission
 ↓
Execution
```

---

# 182. SDK Core and Billing

```text
SDK Request
 ↓
Usage Event
 ↓
Metering
 ↓
Billing
```

---

# 183. SDK Core and FinOps

```text
SDK Usage
 ↓
Cost Attribution
 ↓
FinOps
 ↓
Optimization
```

---

# 184. SDK Core and AI Economics

AI operations should preserve enough metadata to calculate:

```text
Model
Tokens
Requests
Latency
Cost
Tenant
Application
```

---

# 185. SDK Core and Agent Economics

Agent operations should preserve:

```text
Agent
Execution
Tool
Model
Steps
Usage
Cost
```

---

# 186. SDK Core and Marketplace

Marketplace operations should preserve:

```text
Vendor
Product
Customer
Installation
Version
Usage
```

---

# 187. SDK Core and Developer Experience

Core should remain largely invisible to normal developers.

The developer should experience:

```text
Simple
Predictable
Consistent
Reliable
```

behavior.

---

# 188. SDK Core Abstraction Principle

The architecture should hide:

```text
Networking Complexity
Token Management
Retry Logic
Serialization
Telemetry
```

while exposing:

```text
Business Capability
```

---

# 189. SDK Core Anti-Corruption Boundary

SDK Core should protect applications from unnecessary platform implementation details.

```text
EVOXA Internal Complexity
          ↓
       SDK Core
          ↓
Application Abstraction
```

---

# 190. SDK Core Evolution

The Core should evolve without forcing application rewrites.

```text
Application
 ↓
Stable SDK Contract
 ↓
Evolving Core
 ↓
Evolving Platform
```

---

# 191. SDK Core Extensibility

Future Core extensions may include:

```text
Offline Support
Advanced Caching
Local Emulation
Edge Execution
Realtime Transport
Binary Protocols
Advanced AI Streaming
```

These should be introduced without destabilizing the Core.

---

# 192. Local Development Architecture

SDK Core may eventually support local emulation.

```text
Developer
 ↓
SDK
 ↓
Local EVOXA Emulator
```

This can accelerate development and testing.

---

# 193. Offline Development

Where technically feasible, selected SDK features may support offline development through mocks and local environments.

---

# 194. Edge Architecture

SDK Core should not assume every application runs in a traditional server environment.

Potential environments include:

```text
Cloud
Server
Container
Serverless
Mobile
Desktop
Edge
AI Runtime
Agent Runtime
```

---

# 195. Mobile Architecture

Mobile SDK implementations require special handling for:

```text
Network Changes
Offline State
Credential Security
Background Execution
Battery
```

---

# 196. Serverless Architecture

Serverless applications should avoid expensive SDK initialization.

```text
Function Start
 ↓
SDK Initialization
 ↓
Request
 ↓
Response
```

---

# 197. Edge Architecture

Edge runtimes may impose:

```text
Memory Limits
Execution Limits
Restricted APIs
Low Latency Requirements
```

SDK Core implementations should adapt accordingly.

---

# 198. Agent Runtime Architecture

Agent runtimes may require:

```text
Streaming
Cancellation
Budget Enforcement
Tool Invocation
Telemetry
```

Core should support these primitives.

---

# 199. SDK Core Strategic Architecture

```text
                         SDK CORE
                            │
       ┌────────────────────┼────────────────────┐
       ▼                    ▼                    ▼
   SECURITY             RELIABILITY         OBSERVABILITY
       │                    │                    │
   Identity              Retry                Metrics
   Credentials           Timeout              Logs
   Context               Resilience            Traces
   Policy                Streaming             Usage
       │                    │                    │
       └────────────────────┼────────────────────┘
                            ▼
                       TRANSPORT
                            │
                            ▼
                       EVOXA APIs
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
             DATA           AI          AGENTS
```

---

# 200. Final SDK Core Definition

SDK Core is the **shared technical foundation of the EVOXA SDK Platform**.

It provides the reusable infrastructure required to make every official SDK:

```text
Secure
Reliable
Consistent
Observable
Performant
Compatible
Extensible
```

The final model is:

```text
                    EVOXA PLATFORM
                          │
                          ▼
                     API PLATFORM
                          │
                          ▼
                       SDK CORE
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
    DOMAIN SDKs        AI SDK            AGENT SDK
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ▼
                     APPLICATIONS
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
          EVENTS       WORKFLOWS      TOOLS
             │            │            │
             └────────────┼────────────┘
                          ▼
                   ENTERPRISE VALUE
```

SDK Core therefore becomes the **stable programmable foundation beneath the entire EVOXA developer ecosystem**.

Its most important architectural responsibility is not to expose more functionality, but to provide a **consistent execution contract** through which all EVOXA SDK capabilities can operate.

The Core establishes the invariant:

```text
Any EVOXA SDK
        ↓
Uses the same fundamental
        ↓
Identity
Authentication
Configuration
Transport
Errors
Resilience
Telemetry
Security
Versioning
        ↓
Regardless of domain
        ↓
Regardless of programming language
```

This allows EVOXA to evolve its APIs, AI capabilities, agents, marketplace and enterprise services while preserving a stable developer-facing foundation.

The long-term evolution is:

```text
SDK Core
   ↓
Unified SDK Infrastructure
   ↓
AI SDK Infrastructure
   ↓
Agent SDK Infrastructure
   ↓
Autonomous Runtime Infrastructure
   ↓
Self-Evolving Developer Platform
```

**SDK Core is therefore the foundational execution layer that makes the broader EVOXA SDK Platform coherent, secure, reusable and evolvable.**
