# 28/11 — SDK API Clients

## 1. Document Purpose

SDK API Clients defines the architecture, behavior, responsibilities and lifecycle of the client layer through which EVOXA applications communicate with EVOXA APIs.

The API Client is the primary execution interface between application code and the EVOXA platform.

```text id="r4m8q1"
Application
     ↓
SDK API Client
     ↓
SDK Core
     ↓
SDK Runtime
     ↓
Transport
     ↓
EVOXA API
```

The API Client layer must provide a consistent, secure, reliable and language-native interface for accessing EVOXA platform capabilities.

SDK API Clients builds on:

```text id="j7p3w2"
28/01 — SDK Overview
28/02 — SDK Business Model
28/03 — SDK Strategy
28/04 — SDK Architecture
28/05 — SDK Core
28/06 — SDK Runtime
28/07 — SDK Lifecycle
28/08 — SDK Versioning
28/09 — SDK Languages
28/10 — SDK Generation
```

---

# 2. SDK API Client Definition

An SDK API Client is the software interface responsible for translating developer operations into authenticated EVOXA API requests and converting API responses into language-native SDK objects.

```text id="q6t2m9"
Developer Intent
      ↓
API Client
      ↓
Request
      ↓
EVOXA API
      ↓
Response
      ↓
SDK Model
      ↓
Application
```

---

# 3. API Client Mission

The mission is:

> **Provide developers with a simple, reliable and strongly integrated interface for consuming EVOXA platform capabilities without requiring them to manually manage HTTP communication, authentication, serialization, retries or API contracts.**

---

# 4. API Client Principles

SDK API Clients follow:

```text id="b4n8x2"
Simple
Typed
Secure
Consistent
Observable
Resilient
Version-Aware
Language-Native
```

---

# 5. API Client Responsibilities

The API Client is responsible for:

```text id="z8m4q1"
Request Construction
Authentication Integration
Parameter Handling
Serialization
Transport Invocation
Response Processing
Error Mapping
Pagination
Streaming
Telemetry
```

---

# 6. API Client Boundary

The client must separate application logic from transport details.

```text id="p7w3k5"
Application Logic
       │
       ▼
    API Client
       │
       ▼
    SDK Core
       │
       ▼
    SDK Runtime
       │
       ▼
    Transport
```

---

# 7. API Client Architecture

```text id="c9v2m6"
                         APPLICATION
                              │
                              ▼
                       SDK API CLIENT
                              │
                ┌─────────────┼─────────────┐
                ▼             ▼             ▼
             Services       Models        Errors
                │             │             │
                └─────────────┼─────────────┘
                              ▼
                          SDK CORE
                              │
                              ▼
                        SDK RUNTIME
                              │
                 ┌────────────┼────────────┐
                 ▼            ▼            ▼
             Transport     Auth        Telemetry
                 │
                 ▼
                         EVOXA API
```

---

# 8. Primary Client

Each SDK should expose a primary client abstraction.

Conceptually:

```text id="f5m7q3"
EvoxaClient
```

The exact naming follows each language's conventions.

---

# 9. Client Construction

A client should be constructed with the minimum required configuration.

Conceptually:

```text id="w2k8n4"
Client
 ├── Credentials
 ├── Endpoint
 ├── Configuration
 └── Runtime
```

---

# 10. Client Configuration

Configuration may include:

```text id="n7p3x8"
API Endpoint
Authentication
Timeout
Retry Policy
Telemetry
Logging
User Agent
API Version
```

---

# 11. Default Configuration

The SDK should provide safe production-oriented defaults.

Developers should not need to configure every option manually.

---

# 12. Configuration Overrides

Advanced applications should be able to override supported settings.

---

# 13. Environment Configuration

Where appropriate, SDKs may support environment-based configuration.

```text id="j4q8m2"
Environment
     ↓
SDK Configuration
     ↓
API Client
```

---

# 14. Client Immutability

Client configuration should preferably be immutable after initialization unless the language ecosystem strongly favors another model.

---

# 15. Client Reuse

Clients should generally be reusable across multiple API operations.

```text id="r9m5k1"
Create Client
      ↓
Request
      ↓
Request
      ↓
Request
```

---

# 16. Connection Reuse

The runtime should reuse network connections where supported.

---

# 17. Client Thread Safety

Official SDKs should document whether clients are:

```text id="x6v2q8"
Thread-Safe
Concurrent-Safe
Not Thread-Safe
```

according to language runtime behavior.

---

# 18. Client Lifecycle

```text id="g3n8p5"
Create
 ↓
Configure
 ↓
Use
 ↓
Observe
 ↓
Close / Dispose
```

---

# 19. Resource Management

Where the language requires explicit resource management, the SDK should provide appropriate mechanisms.

---

# 20. Service Clients

Large APIs should be organized into service clients.

```text id="m8q4x2"
EvoxaClient
 ├── Users
 ├── Organizations
 ├── Billing
 ├── AI
 ├── Agents
 ├── Analytics
 └── Administration
```

---

# 21. Domain Client Model

Each domain should expose a coherent API surface.

```text id="t5k7n3"
client.billing
client.ai
client.agents
```

or the equivalent native structure.

---

# 22. Service Isolation

Domain clients should avoid unnecessary coupling.

---

# 23. Shared Client

All domain services should normally use the same underlying:

```text id="v3m9q7"
Authentication
Transport
Configuration
Runtime
Telemetry
```

---

# 24. Request Flow

The standard request flow is:

```text id="x2p8m5"
Developer
 ↓
Service Client
 ↓
API Method
 ↓
Request Builder
 ↓
SDK Core
 ↓
Runtime
 ↓
Authentication
 ↓
Transport
 ↓
EVOXA API
```

---

# 25. Response Flow

```text id="q8n3v6"
EVOXA API
 ↓
Transport
 ↓
Runtime
 ↓
Response Handler
 ↓
Deserializer
 ↓
SDK Model
 ↓
Developer
```

---

# 26. Request Builder

The request builder is responsible for constructing a valid API request.

---

# 27. Request Components

A request may contain:

```text id="k5m9x2"
Method
URL
Path Parameters
Query Parameters
Headers
Body
Authentication
```

---

# 28. HTTP Method Mapping

API operations should map naturally to:

```text id="g7p2v4"
GET
POST
PUT
PATCH
DELETE
```

where applicable.

---

# 29. URL Construction

Path parameters must be safely encoded.

---

# 30. Query Construction

Query parameters must be encoded consistently.

---

# 31. Header Management

The SDK should manage:

```text id="c8m4q1"
Authorization
Content-Type
Accept
User-Agent
Request ID
Tracing
```

where applicable.

---

# 32. User-Agent

Requests should identify the EVOXA SDK implementation.

Conceptually:

```text id="z5k8p3"
EVOXA-SDK/6.x
Language/Runtime
```

---

# 33. Request IDs

Where supported, each request should receive or propagate a request identifier.

---

# 34. Trace Context

SDK clients should support distributed tracing where the runtime and platform support it.

---

# 35. Authentication Integration

Authentication must be handled by the SDK security layer rather than application code.

```text id="y7m3q8"
Application
 ↓
API Client
 ↓
Auth Provider
 ↓
Credential
 ↓
Request
```

---

# 36. Credential Providers

The SDK may support:

```text id="v2n6k4"
API Key
OAuth
Access Token
Service Credential
Workload Identity
```

according to platform capabilities.

---

# 37. Credential Rotation

The client should support credential rotation without requiring unnecessary application restarts where technically possible.

---

# 38. Token Refresh

If the authentication model requires refresh tokens, the client should integrate refresh behavior through SDK Core.

---

# 39. Authentication Failure

Authentication failures should produce structured SDK errors.

---

# 40. Authorization Failure

Authorization failures should be distinguishable from authentication failures.

---

# 41. Serialization

Request objects must be serialized according to the authoritative API schema.

---

# 42. Deserialization

Responses should be converted into language-native SDK models.

---

# 43. Serialization Boundary

```text id="d8q4m2"
Native Object
      ↓
Serializer
      ↓
API Payload
```

---

# 44. Deserialization Boundary

```text id="j5v8n3"
API Payload
      ↓
Deserializer
      ↓
Native Model
```

---

# 45. Schema Enforcement

Serialization and deserialization should respect schema definitions.

---

# 46. Unknown Fields

SDKs should safely handle additional response fields when backward compatibility permits.

---

# 47. Missing Fields

Required missing fields should produce appropriate errors during deserialization or validation.

---

# 48. Type Conversion

The SDK should convert platform types into language-native representations.

---

# 49. Model Construction

Generated models should be created through predictable constructors, builders or equivalent language patterns.

---

# 50. Immutable Models

Where appropriate, response models should be immutable to prevent accidental mutation of server state assumptions.

---

# 51. Mutable Models

Mutable request models may be supported where they provide a natural developer experience.

---

# 52. Request vs Response Models

The SDK should distinguish between:

```text id="m6q2v9"
Request Models
Response Models
```

when their semantics differ.

---

# 53. API Method Design

API methods should expose domain-level intent rather than raw HTTP mechanics.

Prefer:

```text id="t3k7p1"
client.users.get(userId)
```

over requiring developers to manually construct HTTP requests.

---

# 54. Raw Request Access

Advanced developers may need controlled access to lower-level request mechanisms.

The SDK may provide an advanced escape hatch.

---

# 55. Raw Transport Access

Raw transport access should not be required for normal SDK usage.

---

# 56. Pagination

Clients should abstract pagination whenever possible.

```text id="q4m8x2"
API
 ↓
Pagination Handler
 ↓
Iterator / Page
 ↓
Developer
```

---

# 57. Page-Based APIs

SDKs may expose explicit page operations.

---

# 58. Iterator-Based APIs

SDKs should support idiomatic iteration where possible.

---

# 59. Automatic Pagination

Convenience APIs may retrieve multiple pages automatically.

---

# 60. Pagination Limits

Automatic pagination must respect:

```text id="h7p3m5"
Rate Limits
Memory
Cancellation
Timeouts
Application Limits
```

---

# 61. Streaming APIs

Streaming clients should expose native streaming abstractions.

```text id="n8k4v2"
EVOXA Stream
 ↓
SDK Stream
 ↓
Language Stream
 ↓
Application
```

---

# 62. Streaming Lifecycle

```text id="p2m7x5"
Connect
 ↓
Receive
 ↓
Process
 ↓
Reconnect if supported
 ↓
Close
```

---

# 63. Streaming Errors

Stream errors must be distinguishable from normal stream completion.

---

# 64. Stream Cancellation

Applications should be able to cancel streams.

---

# 65. Async Clients

SDK clients should support asynchronous execution where appropriate.

---

# 66. Sync Clients

Languages where synchronous APIs are idiomatic should provide them when practical.

---

# 67. Async / Sync Consistency

Sync and async clients should preserve equivalent semantic behavior.

---

# 68. Client Concurrency

Concurrent operations should be safe according to documented client guarantees.

---

# 69. Request Timeouts

Every request should have a bounded timeout or an explicit configurable policy.

---

# 70. Timeout Types

The SDK may distinguish:

```text id="w4n8k2"
Connection Timeout
Read Timeout
Write Timeout
Total Request Timeout
```

where the runtime supports them.

---

# 71. Retry Behavior

Transient failures may be retried automatically.

---

# 72. Retryable Errors

Typical retryable conditions may include:

```text id="c6p2m9"
Network Failure
Temporary Server Failure
Rate Limiting
Transient Gateway Error
```

according to platform policy.

---

# 73. Non-Retryable Errors

Examples:

```text id="y8k3q5"
Validation Error
Authentication Error
Authorization Error
Not Found
```

should generally not be automatically retried.

---

# 74. Exponential Backoff

Retries should use controlled backoff.

```text id="n5m7x1"
Attempt 1
 ↓
Wait
 ↓
Attempt 2
 ↓
Longer Wait
 ↓
Attempt 3
```

---

# 75. Jitter

Retry algorithms should support jitter to avoid synchronized retry storms.

---

# 76. Retry Limits

Retries must have bounded limits.

---

# 77. Retry Idempotency

The SDK must consider operation idempotency before automatically retrying requests.

---

# 78. Idempotency Keys

Where supported, clients should expose idempotency-key mechanisms for safe repeated operations.

---

# 79. Rate Limiting

The client should recognize platform rate-limit responses.

---

# 80. Rate Limit Metadata

Where available, SDK errors may expose:

```text id="r6p2k8"
Retry-After
Limit
Remaining
Reset
```

---

# 81. Rate Limit Handling

Applications may choose between:

```text id="x4m8v2"
Automatic Retry
Manual Retry
Backoff
Fail Fast
```

according to configuration.

---

# 82. Error Architecture

```text id="j7q3m5"
API Error
   ↓
Transport Error
   ↓
SDK Error Mapper
   ↓
Typed SDK Error
   ↓
Application
```

---

# 83. Error Categories

Common categories include:

```text id="m8v4k1"
Authentication
Authorization
Validation
Not Found
Conflict
Rate Limit
Timeout
Network
Server
Unknown
```

---

# 84. Error Consistency

The same platform error should map to semantically equivalent SDK errors across languages.

---

# 85. Error Context

Errors should provide enough context for debugging without exposing secrets.

---

# 86. Request Context

Errors should expose request identifiers when available.

---

# 87. Error Redaction

Sensitive data must not appear in exceptions or logs.

---

# 88. Logging

API clients should integrate with the SDK logging architecture.

---

# 89. Logging Levels

Potential levels:

```text id="p5n8x3"
Debug
Info
Warning
Error
```

---

# 90. Request Logging

Request logging should redact:

```text id="t3q7m1"
Authorization
Tokens
Secrets
Sensitive Payloads
```

---

# 91. Response Logging

Response logging should avoid sensitive content by default.

---

# 92. Debug Mode

Debug mode may expose additional request information with explicit developer configuration.

---

# 93. Telemetry

The API Client should emit structured telemetry.

```text id="f6m2q8"
Request
 ↓
Metrics
Logs
Traces
 ↓
Observability
```

---

# 94. Client Metrics

Useful metrics include:

```text id="k8p3v5"
Request Count
Success Rate
Error Rate
Latency
Retries
Timeouts
Rate Limits
```

---

# 95. SDK Version Telemetry

Requests should identify the SDK version where telemetry policy allows.

---

# 96. Language Telemetry

The SDK may identify:

```text id="w2m7x4"
Language
Runtime
SDK Version
```

---

# 97. Privacy

Telemetry must follow EVOXA privacy and data governance policies.

---

# 98. Client Health

The SDK may expose health-related metrics but should avoid introducing unnecessary network calls.

---

# 99. Endpoint Selection

Clients may support configurable API endpoints.

```text id="g4n8p2"
Production
Staging
Development
Private Enterprise
```

where supported.

---

# 100. Environment Isolation

Different environments must be clearly separated.

---

# 101. Region Selection

If EVOXA supports regional APIs, the client may support:

```text id="v7m3q5"
Region
 ↓
Endpoint
```

---

# 102. Multi-Region Clients

Enterprise SDKs may support region-aware routing.

---

# 103. Failover

Where supported, clients may fail over between endpoints according to platform policy.

---

# 104. Failover Safety

Failover must not duplicate non-idempotent operations unintentionally.

---

# 105. API Version Selection

Clients may expose API version selection where required.

```text id="m5q8x2"
Client
 ↓
API Version
 ↓
Request
```

---

# 106. Default API Version

The SDK should select a safe default API version.

---

# 107. Version Override

Advanced users may override the default when explicitly supported.

---

# 108. Unsupported API Version

The client must produce an actionable compatibility error.

---

# 109. Client Compatibility

The API Client must respect Chapter 08 versioning rules.

```text id="r3n7k1"
SDK Version
 ↓
API Version
 ↓
Compatibility
```

---

# 110. API Contract Alignment

Generated clients must remain synchronized with the authoritative API contract.

---

# 111. Client Generation

API Clients should be generated where appropriate.

```text id="q8m4v2"
API Contract
 ↓
Generator
 ↓
Client
```

---

# 112. Generated Client Components

Generation may create:

```text id="z6p2x5"
Methods
Models
Parameters
Serialization
Error Mappings
Documentation
Tests
```

---

# 113. Runtime Components

The generated client should rely on runtime components for:

```text id="n4k8m1"
Transport
Authentication
Retries
Timeouts
Telemetry
```

---

# 114. Generated vs Runtime

```text id="f3m7q9"
Generated Client
      ↓
SDK Core
      ↓
SDK Runtime
```

---

# 115. Client Extension

Developers should be able to extend supported client behavior without modifying generated code.

---

# 116. Middleware

Where useful, clients may support middleware/interceptor pipelines.

```text id="w5q2m8"
Request
 ↓
Middleware
 ↓
Auth
 ↓
Transport
 ↓
Response
 ↓
Middleware
 ↓
Application
```

---

# 117. Middleware Use Cases

Potential uses:

```text id="m9x3p6"
Logging
Tracing
Metrics
Headers
Caching
Custom Validation
```

---

# 118. Middleware Ordering

Middleware execution order must be deterministic.

---

# 119. Middleware Security

Middleware must not bypass SDK authentication or security controls accidentally.

---

# 120. Hooks

Language-specific SDKs may expose lifecycle hooks.

---

# 121. Request Hooks

Potentially:

```text id="g2k7n5"
Before Request
After Request
On Error
```

---

# 122. Response Hooks

Response hooks should not mutate critical platform semantics unexpectedly.

---

# 123. Caching

Caching may be supported selectively.

---

# 124. Cache Safety

Caching must respect:

```text id="x4m8p2"
Authentication
Tenant Isolation
Data Freshness
Privacy
Authorization
```

---

# 125. Client-Side Cache

Cache should never expose data across tenants or users.

---

# 126. Offline Behavior

Mobile SDKs may require controlled offline behavior.

```text id="p7n3m5"
Offline
 ↓
Queue / Cache
 ↓
Reconnect
 ↓
Synchronize
```

This must be explicitly defined for each API.

---

# 127. Mobile Client

Dart/Flutter clients require additional considerations:

```text id="v5q8m2"
Connectivity
Background Tasks
Battery
Secure Storage
Network Changes
```

---

# 128. Browser Client

Browser SDKs must consider:

```text id="m3x7p1"
CORS
Credential Exposure
Browser Security
Storage
```

---

# 129. Server Client

Server-side clients may safely support credential types that should never be exposed to browsers.

---

# 130. Client Environment Classification

The SDK should clearly identify supported environments:

```text id="k8n4q2"
Server
Browser
Mobile
Desktop
Serverless
Edge
```

---

# 131. Security Boundary

Browser and mobile SDKs must never expose server-only credentials.

---

# 132. Client Credential Isolation

The SDK should provide mechanisms that make insecure credential usage difficult.

---

# 133. Multi-Tenant Applications

API clients must support tenant context where required.

```text id="w6p2m9"
Application
 ↓
Tenant Context
 ↓
API Client
 ↓
EVOXA
```

---

# 134. Tenant Isolation

The SDK must never accidentally reuse tenant credentials or context across isolated clients.

---

# 135. Organization Context

Enterprise applications may specify:

```text id="q5m8x3"
Organization
Tenant
Workspace
Project
```

where supported.

---

# 136. User Context

Some operations may execute on behalf of a user.

The client must preserve appropriate identity context.

---

# 137. Delegated Access

Where supported:

```text id="n7k3p5"
Application
 ↓
Delegated Identity
 ↓
API Client
 ↓
EVOXA
```

---

# 138. Request Context

Applications may attach request-specific metadata through controlled APIs.

---

# 139. Correlation IDs

The client should support correlation IDs for distributed workflows.

---

# 140. Distributed Tracing

API clients should propagate trace context when enabled.

---

# 141. Agent API Clients

Agent applications may require specialized clients for:

```text id="x8m4q2"
Models
Tools
Memory
Execution
Policies
Workflows
```

---

# 142. AI API Clients

AI clients may expose:

```text id="p6n3v8"
Generate
Stream
Embed
Classify
Evaluate
```

according to EVOXA capabilities.

---

# 143. Agent Client

Conceptually:

```text id="j4q7m2"
client.agents
 ├── create
 ├── get
 ├── execute
 ├── stream
 └── stop
```

The exact API depends on EVOXA platform contracts.

---

# 144. AI Streaming

AI streaming clients should expose token/event streams through native language abstractions.

---

# 145. Tool Execution Clients

Agent tools should use the same authentication, telemetry and error infrastructure where appropriate.

---

# 146. Workflow Clients

Workflow clients should expose execution and status operations through consistent client abstractions.

---

# 147. Long-Running Operations

Some EVOXA APIs may return asynchronous operation handles.

```text id="z5m8p3"
Request
 ↓
Operation ID
 ↓
Poll / Wait
 ↓
Result
```

---

# 148. Operation Polling

The SDK may provide a convenience abstraction for long-running operations.

---

# 149. Operation Cancellation

Long-running operations should support cancellation where the platform permits it.

---

# 150. Operation Status

Clients should expose structured status:

```text id="v7q2m4"
Queued
Running
Succeeded
Failed
Cancelled
```

---

# 151. Batch APIs

Clients may expose batch operations.

```text id="m8p3k1"
Requests
 ↓
Batch
 ↓
EVOXA
 ↓
Results
```

---

# 152. Batch Error Handling

Partial failures must be represented explicitly.

---

# 153. Bulk Operations

Bulk APIs should preserve rate limits and retry semantics.

---

# 154. Client Resource Management

The client should minimize:

```text id="x2n7q5"
Connections
Threads
Memory
Background Tasks
```

---

# 155. Client Shutdown

Applications should have a clean mechanism to close clients where required.

---

# 156. Resource Cleanup

Cleanup should terminate:

```text id="p4m8x2"
Connections
Streams
Background Workers
Timers
```

where applicable.

---

# 157. Client Testing

API Clients should be highly testable.

---

# 158. Mock Transport

Developers should be able to test without contacting EVOXA.

```text id="n6q3m8"
Application
 ↓
API Client
 ↓
Mock Transport
```

---

# 159. Request Mocking

Tests should be able to verify:

```text id="w5m8p2"
Method
URL
Headers
Body
```

without exposing credentials.

---

# 160. Response Fixtures

SDKs should maintain representative API response fixtures.

---

# 161. Error Fixtures

Common API errors should have test fixtures.

---

# 162. Contract Testing

Every client implementation should run against authoritative API contracts.

---

# 163. Integration Testing

Official SDKs should test against real EVOXA environments where appropriate.

---

# 164. Client Regression Testing

SDK upgrades must ensure that existing client behavior remains stable unless intentionally changed.

---

# 165. Client Performance

API Client performance should be evaluated across:

```text id="g7p3n5"
Latency
Serialization
Memory
Connection Reuse
Concurrency
```

---

# 166. Client Scalability

The client must support workloads ranging from:

```text id="q4m8x2"
Single Request
 ↓
Interactive Application
 ↓
High-Throughput Service
 ↓
Enterprise Platform
```

---

# 167. Connection Pooling

Server SDKs should support connection pooling where appropriate.

---

# 168. Connection Limits

Applications should be able to configure connection limits when supported.

---

# 169. Backpressure

Streaming and high-throughput clients should support appropriate backpressure mechanisms.

---

# 170. Client Resilience

API clients should protect applications from transient platform failures through:

```text id="m5n8q3"
Timeouts
Retries
Backoff
Circuit Breaking where appropriate
```

---

# 171. Circuit Breaking

Advanced SDK Runtime implementations may provide circuit-breaking behavior.

---

# 172. Circuit Breaker Safety

Circuit breaking must not hide persistent authorization or validation problems.

---

# 173. Client Fault Isolation

Failures in one API operation should not unnecessarily destabilize the entire client.

---

# 174. Request Isolation

Independent requests should maintain independent error and cancellation contexts.

---

# 175. API Client Observability

The client layer should make operational behavior observable without exposing sensitive data.

---

# 176. Client Debugging

Developers should be able to diagnose:

```text id="v2m7p4"
Authentication
Request
Response
Latency
Retries
Errors
```

through supported tooling.

---

# 177. Request Replay

Where supported, sanitized request metadata may help developers reproduce failures.

---

# 178. Request Recording

Testing environments may support request recording with sensitive data redaction.

---

# 179. Client Documentation

Every API Client should document:

```text id="k8q3m5"
Installation
Configuration
Authentication
Methods
Models
Errors
Retries
Pagination
Streaming
Examples
```

---

# 180. API Reference

Reference documentation should be generated from the authoritative contract wherever possible.

---

# 181. Client Examples

Examples should demonstrate:

```text id="m4x7p2"
Basic Request
Authentication
Error Handling
Pagination
Streaming
Production Configuration
```

---

# 182. Client Migration

When client APIs change:

```text id="q6n3v8"
Old Client
 ↓
Migration Guide
 ↓
New Client
```

---

# 183. Client Deprecation

Deprecated methods should provide:

```text id="z5m8k2"
Deprecation Notice
Replacement
Migration Documentation
```

---

# 184. Client Version Compatibility

The client must communicate its:

```text id="x7p3m4"
SDK Version
API Version
Runtime Compatibility
```

---

# 185. Client Release

The API Client participates in the SDK release lifecycle:

```text id="n8q2m5"
Generate
 ↓
Validate
 ↓
Test
 ↓
Package
 ↓
Publish
```

---

# 186. Client Security Release

Security fixes should be distributed through patch releases whenever compatible.

---

# 187. Client Rollback

If a client release causes serious regressions, the ecosystem should be able to revert to a known-good version.

---

# 188. Client Registry

The EVOXA SDK Registry should track:

```text id="p5m7x2"
Client
Version
Language
API Version
Runtime
Status
```

---

# 189. API Client Marketplace

Future EVOXA developer infrastructure may expose client capabilities through a unified developer portal.

---

# 190. API Client Discovery

Developers should be able to discover:

```text id="w3n8q5"
Official SDK
Language
Version
Capabilities
Documentation
Examples
```

---

# 191. API Client Health

EVOXA should monitor:

```text id="g6m2p8"
Client Adoption
Errors
Versions
Compatibility
Performance
```

where telemetry policies permit.

---

# 192. API Client Analytics

Analytics may include:

```text id="v4q7m1"
Requests
Active Versions
Language
Feature Usage
Migration
Error Rates
```

---

# 193. Client Adoption

Client adoption provides signals for:

```text id="n8p3x5"
Version Support
Language Prioritization
Feature Development
Deprecation
```

---

# 194. Client Evolution

The API Client evolves through:

```text id="m5k8q2"
Raw API Wrapper
 ↓
Typed Client
 ↓
Domain Client
 ↓
Resilient Client
 ↓
Observable Client
 ↓
AI-Aware Client
 ↓
Autonomous Client
```

---

# 195. AI-Aware API Clients

Future clients may understand:

```text id="p7n4x8"
Model Capabilities
Agent Capabilities
Tool Compatibility
Context Requirements
Usage Limits
```

---

# 196. Intelligent Request Optimization

Future SDKs may optimize:

```text id="q3m8k5"
Batching
Caching
Retries
Model Selection
Endpoint Selection
```

within explicit platform policies.

---

# 197. AI-Assisted Error Resolution

An EVOXA developer assistant may interpret API errors and suggest:

```text id="w6p2n9"
Cause
Fix
Migration
Retry
Configuration
```

---

# 198. Autonomous API Client

Long-term:

```text id="z4m7q2"
Application
 ↓
Intelligent SDK Client
 ↓
Analyze Request
 ↓
Select Capability
 ↓
Execute
 ↓
Observe
 ↓
Adapt
```

The client remains governed by explicit application and platform policies.

---

# 199. Complete SDK API Client Architecture

```text id="f8q3m1"
                              APPLICATION
                                   │
                                   ▼
                            SDK API CLIENT
                                   │
             ┌─────────────────────┼─────────────────────┐
             ▼                     ▼                     ▼
          SERVICES              MODELS                 ERRORS
             │                     │                     │
             └─────────────────────┼─────────────────────┘
                                   ▼
                               SDK CORE
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
                AUTHENTICATION  SERIALIZATION  VERSIONING
                    │              │              │
                    └──────────────┼──────────────┘
                                   ▼
                             SDK RUNTIME
                                   │
             ┌─────────────────────┼─────────────────────┐
             ▼                     ▼                     ▼
         TRANSPORT              RETRIES              TELEMETRY
             │                     │                     │
             └─────────────────────┼─────────────────────┘
                                   ▼
                              EVOXA API
                                   │
             ┌─────────────────────┼─────────────────────┐
             ▼                     ▼                     ▼
            DATA                    AI                  AGENTS
             │                     │                     │
             └─────────────────────┼─────────────────────┘
                                   ▼
                              APPLICATION
```

---

# 200. Final SDK API Clients Definition

SDK API Clients are the **primary programmatic interface between applications and the EVOXA platform**, translating developer intent into secure, typed, observable and resilient API operations.

The fundamental architecture is:

```text id="a5m8q3"
                         APPLICATION
                              │
                              ▼
                       API CLIENT
                              │
                              ▼
                         SDK CORE
                              │
                              ▼
                       SDK RUNTIME
                              │
             ┌────────────────┼────────────────┐
             ▼                ▼                ▼
       AUTHENTICATION      TRANSPORT        TELEMETRY
             │                │                │
             └────────────────┼────────────────┘
                              ▼
                          EVOXA API
                              │
                              ▼
                       RESPONSE MODEL
                              │
                              ▼
                         APPLICATION
```

The complete request lifecycle is:

```text id="p7n4x2"
DEVELOPER INTENT
       ↓
API CLIENT
       ↓
REQUEST CONSTRUCTION
       ↓
AUTHENTICATION
       ↓
SERIALIZATION
       ↓
TRANSPORT
       ↓
EVOXA API
       ↓
RESPONSE
       ↓
DESERIALIZATION
       ↓
ERROR / RESULT MAPPING
       ↓
SDK MODEL
       ↓
APPLICATION
```

The resilience lifecycle is:

```text id="m8q3v5"
REQUEST
  ↓
TIMEOUT
  ↓
RETRY
  ↓
BACKOFF
  ↓
SUCCESS
     OR
FAILURE
  ↓
STRUCTURED ERROR
```

The evolution model is:

```text id="x4p7n2"
RAW API
   ↓
SDK API Client
   ↓
TYPED CLIENT
   ↓
DOMAIN CLIENT
   ↓
RESILIENT CLIENT
   ↓
OBSERVABLE CLIENT
   ↓
AI-AWARE CLIENT
   ↓
AUTONOMOUS CLIENT
```

The API Client therefore becomes the operational face of EVOXA SDKs.

Its strategic role is to hide unnecessary protocol complexity while preserving complete access to platform capabilities.

```text id="q6m3v8"
                    EVOXA PLATFORM
                           │
                           ▼
                       API CONTRACT
                           │
                           ▼
                    SDK GENERATION
                           │
                           ▼
                     API CLIENT
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
        TypeScript       Python          Java
             │             │             │
             ▼             ▼             ▼
            Go            C#             Dart
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                      SDK CORE
                           │
                           ▼
                    SDK RUNTIME
                           │
                           ▼
                        APPS
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
           WEB          MOBILE        ENTERPRISE
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                      EVOXA ECOSYSTEM
```

Therefore:

> **SDK API Clients are the execution interface of the EVOXA SDK Platform, providing applications with a secure, typed, version-aware, resilient and language-native mechanism for consuming EVOXA APIs, AI capabilities, agent infrastructure and future platform services.**

The ultimate objective is:

```text id="k5m8q2"
Developer
   ↓
Simple API Call
   ↓
Intelligent SDK Client
   ↓
Secure Runtime
   ↓
EVOXA Platform
   ↓
Structured Result
   ↓
Application
```

**SDK API Clients transform the complexity of distributed platform communication into a simple developer abstraction, allowing EVOXA to expose an increasingly sophisticated platform while keeping application development predictable, secure and productive.**
