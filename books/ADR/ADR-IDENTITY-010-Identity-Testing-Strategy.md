ADR-IDENTITY-010 — Identity Testing Strategy

Status: Proposed
Version: 1.0.0
Domain: Identity
Decision Type: Architecture / Quality / Security
Scope: Unit, Component, Integration, API, Contract, E2E, Security, Performance, Resilience, MFA, Authorization, Events and Quality Gates
Related: ESP-0001 — Identity
Depends on:

ADR-IDENTITY-001 — Identity Architecture & Technology Decisions
ADR-IDENTITY-002 — JWT & Token Lifecycle
ADR-IDENTITY-003 — Refresh Token Security & Session Management
ADR-IDENTITY-005 — Authorization & Policy Engine
ADR-IDENTITY-006 — Identity Persistence Strategy
ADR-IDENTITY-007 — Identity Event & Outbox Strategy
ADR-IDENTITY-008 — Identity Cryptographic Key Management
ADR-IDENTITY-009 — Identity Client Token Storage
1. Context

Identity es uno de los componentes de mayor criticidad de EVOXA.

Un error en Identity puede permitir:

acceso no autorizado;
escalamiento de privilegios;
acceso entre tenants;
robo de sesiones;
reutilización de refresh tokens;
bypass de MFA;
exposición de información;
pérdida de trazabilidad;
comprometer otros dominios de la plataforma.

Por esta razón, las pruebas de Identity no pueden limitarse a comprobar que los endpoints funcionan.

El Blueprint establece una estrategia de calidad basada en múltiples niveles de testing y exige validación específica de seguridad, autorización, APIs, eventos, rendimiento, accesibilidad, resiliencia y capacidades de IA.

2. Problem Statement

Necesitamos garantizar simultáneamente:

Functional Correctness
+
Security
+
Authorization
+
Data Integrity
+
API Contract
+
Concurrency
+
Performance
+
Resilience
+
Observability

Por lo tanto:

Identity utilizará una estrategia de testing multinivel con quality gates automáticos antes de permitir la promoción de una versión.

3. Decision

Se adopta una Testing Pyramid ampliada:

                         E2E
                          ▲
                    API / Contract
                          ▲
                    Integration
                          ▲
                     Component
                          ▲
                       Unit

Complementada transversalmente por:

Security
Performance
Resilience
Concurrency
Accessibility
Observability
AI Security / Evaluation

La estrategia será:

Fast Tests
   ↓
Unit
   ↓
Component
   ↓
Integration
   ↓
API / Contract
   ↓
E2E
   ↓
Security / Performance / Resilience
4. Testing Principles

Identity seguirá:

Automated Testing First
Test Pyramid
Fail Fast
Security First
Risk-Based Testing
Deterministic Tests
Isolation
Repeatability
Traceability
Continuous Validation
5. Coverage Targets

Se establecen como objetivos:

Identity overall coverage ≥ 85%
Critical Identity areas ≥ 95%

Las áreas críticas incluyen especialmente:

authentication;
authorization;
token lifecycle;
refresh rotation;
tenant isolation;
MFA;
security policies;
session management.

Estos objetivos están alineados con los requisitos de testing definidos para EVOXA Identity.

6. Unit Testing

Los Unit Tests validarán comportamiento aislado.

Ejemplos:

PasswordService
TokenService
AuthorizationService
PolicyEngine
SessionService
RefreshTokenService
MFAService
AuditService

Ejemplo conceptual:

PasswordService
      ↓
correct password → VALID
wrong password   → INVALID

Los Unit Tests deben ser rápidos y no depender de infraestructura externa cuando no sea necesario.

7. Domain Testing

El dominio deberá probar sus invariantes.

Ejemplo:

User
 ├── ACTIVE
 ├── SUSPENDED
 └── DISABLED

Se probarán transiciones válidas e inválidas:

ACTIVE → SUSPENDED       ✓
SUSPENDED → ACTIVE       ✓
DISABLED → ACTIVE        ?

Las transiciones no permitidas deben producir errores controlados.

8. Component Testing

Los Component Tests validarán componentes completos con sus dependencias controladas.

Ejemplo:

Authentication Component
        │
        ├── User Repository
        ├── Password Service
        ├── Session Service
        └── Token Service

Esto permite validar flujos internos sin requerir todo el sistema EVOXA.

9. Authentication Testing

Se deberán probar:

Login exitoso
Valid Credentials
      ↓
Authenticated
      ↓
Session Created
      ↓
Tokens Issued
Credenciales incorrectas
Invalid Credentials
      ↓
Authentication Failed
Usuario suspendido
Suspended User
      ↓
Authentication Denied
Usuario deshabilitado
Disabled User
      ↓
Authentication Denied
MFA requerido
Credentials Valid
      ↓
MFA Required
      ↓
Challenge
10. Password Testing

Se deberán validar:

password hashing;
password verification;
incorrect password;
password policy;
password reset;
password change;
lockout;
rate limiting.

Nunca se deben utilizar passwords reales de usuarios en los tests.

11. Authorization Testing

Se probará la combinación:

RBAC
+
ABAC
+
Tenant
+
Ownership
+
Security Policy

Ejemplo:

Coach
  +
clients.read
  +
same tenant
  +
owned resource
       ↓
     ALLOW

Y:

Coach
  +
clients.read
  +
different tenant
       ↓
      DENY
12. RBAC Testing

Se probará:

Role → Permission

Casos:

permission granted;
permission missing;
multiple roles;
revoked permission;
inactive role;
invalid role;
unauthorized role assignment.

También deberá comprobarse que un usuario no pueda elevar sus propios privilegios.

13. ABAC Testing

Se probarán condiciones como:

organization_id
resource ownership
resource state
user attributes
operation sensitivity
MFA state
device context
risk context

Ejemplo:

training.update
+
client.owner = current_user
+
tenant = current_tenant

→ ALLOW

Mientras:

training.update
+
client.owner != current_user

→ DENY.

14. Tenant Isolation Testing

Esta será una de las áreas de máxima prioridad.

Se deben probar sistemáticamente:

Tenant A → Tenant A resource

→ permitido cuando corresponda.

Tenant A → Tenant B resource

→ siempre denegado salvo una política explícita y legítima que lo permita.

También:

invalid tenant IDs;
missing tenant;
suspended tenant;
removed membership;
cross-tenant API requests;
cross-tenant database queries;
cross-tenant events;
cross-tenant AI context.
15. Session Testing

Se probará el ciclo:

CREATED
   ↓
ACTIVE
   ↓
EXPIRED

y:

ACTIVE
   ↓
REVOKED

Casos:

session creation;
session expiration;
logout;
logout-all;
administrative revocation;
user suspension;
organization suspension;
membership removal;
concurrent sessions.
16. JWT Testing

Se deberá probar:

Valid JWT
signature ✓
issuer ✓
audience ✓
expiration ✓
claims ✓

→ ALLOW

Invalid JWT
signature ✗

→ DENY

También:

expired token;
malformed token;
wrong issuer;
wrong audience;
unknown kid;
wrong signature;
missing required claims;
algorithm mismatch.
17. Refresh Token Testing

Este será un componente crítico.

Debe probarse:

Token A
   ↓
Refresh
   ↓
Token B

y que:

Token A

ya no pueda reutilizarse.

18. Refresh Token Reuse Detection

Caso crítico:

Token A
   ↓
Refresh
   ↓
Token B

Posteriormente:

Attacker
   ↓
Reuse Token A

Resultado esperado:

REUSE_DETECTED
      ↓
Security Event
      ↓
Audit
      ↓
Family Revocation
      ↓
Session Revocation

Esto debe estar cubierto por pruebas automatizadas.

19. Concurrent Refresh Testing

Debe probarse:

Request A ─┐
Request B ─┼──> Refresh Token A
Request C ─┘

simultáneamente.

El sistema debe garantizar que no existan múltiples refresh exitosos indebidos utilizando el mismo token.

Se requieren pruebas reales de concurrencia, no solamente Unit Tests.

20. MFA Testing

Se probará:

Enrollment
Create Factor
   ↓
Verify
   ↓
Activate
Authentication
Password Correct
      ↓
MFA Required
      ↓
Challenge
      ↓
Verified

También:

incorrect OTP;
expired OTP;
reused OTP;
brute force;
recovery code;
factor revocation;
trusted device;
step-up authentication.
21. Security Policy Testing

Cada decisión posible del Policy Engine debe probarse:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

También:

Deny by Default
Fail Closed

Ejemplo:

Policy Engine unavailable
        ↓
      DENY

Nunca:

Policy Engine unavailable
        ↓
      ALLOW
22. API Contract Testing

Las APIs de Identity deben validarse contra sus contratos.

Se probarán:

HTTP methods;
paths;
request schema;
response schema;
status codes;
authentication;
authorization;
validation;
errors;
pagination;
filtering;
idempotency;
correlation IDs.

La arquitectura de EVOXA establece API-First y OpenAPI como parte del contrato de APIs.

23. Error Contract Testing

Las respuestas de error deberán respetar:

{
  "error": {
    "code": "...",
    "message": "...",
    "correlation_id": "..."
  }
}

Se probará que:

no se expongan secretos;
no se expongan tokens;
no se revelen detalles internos;
los códigos sean consistentes;
exista correlation ID cuando corresponda.
24. Idempotency Testing

Las operaciones que soporten idempotencia deben probar:

Request A
Request A duplicate

Resultado:

Same logical operation

y no:

Duplicate side effects

Especialmente importante para:

user creation;
membership operations;
role assignments;
MFA operations;
administrative operations.
25. Persistence Testing

Se deberán validar:

PostgreSQL constraints;
foreign keys;
uniqueness;
transactions;
rollback;
migrations;
concurrent updates;
tenant filtering;
repository behavior.

Ejemplo:

DB Transaction
    ↓
User Created
    +
Outbox Created

ambos deben existir después del commit.

26. Transactional Outbox Testing

Caso exitoso:

Business Change
      +
Outbox Event
      ↓
COMMIT

Caso fallido:

Business Change
      +
Outbox Event
      ↓
ROLLBACK

Resultado:

No Business Change
No Outbox Event

También:

publisher failure;
retry;
duplicate publish;
worker crash;
DLQ;
replay.
27. Event Testing

Se probarán:

event_id
event_type
event_version
timestamp
tenant_id
actor
resource
payload
metadata
correlation_id
causation_id
trace_id

Además:

schema validation;
compatibility;
duplicate events;
ordering;
consumer idempotency.
28. Cryptographic Testing

De acuerdo con ADR-IDENTITY-008:

Se probarán:

JWT signature;
public key validation;
kid;
key rotation;
retired key;
unknown key;
emergency rotation;
encryption/decryption;
secret protection;
MFA secret protection.

Nunca se deben imprimir claves reales en los tests.

29. Client Token Storage Testing
Web

Probar:

HttpOnly;
Secure;
SameSite;
CSRF;
access token memory storage;
logout;
refresh;
multi-tab;
concurrent refresh.
Mobile

Probar:

secure storage;
app restart;
refresh;
logout;
credential deletion;
concurrent refresh.

Esto valida la decisión definida en ADR-IDENTITY-009.

30. Security Testing

Identity tendrá una suite específica de seguridad.

Debe cubrir:

Authentication Bypass
Authorization Bypass
Privilege Escalation
Tenant Escape
Session Hijacking
Token Replay
Token Leakage
MFA Bypass
CSRF
XSS-related token exposure
Injection
Rate Limit Bypass
Secret Exposure

Las vulnerabilidades críticas y de alta severidad deberán bloquear una release.

31. Rate Limiting Testing

Se probarán límites sobre:

login;
password reset;
MFA;
token refresh;
API endpoints;
administrative operations.

Ejemplo:

100 failed logins
       ↓
Rate Limit
       ↓
Request blocked

También se probará que el rate limiter no pueda ser evadido mediante:

IP manipulation;
concurrent requests;
alternate endpoints;
tenant switching.
32. Integration Testing

Se validará la interacción con:

PostgreSQL
Redis
Event Platform
KMS / Secret Manager
External Identity Providers

Los tests deben poder ejecutarse en ambientes controlados y reproducibles.

33. External Identity Provider Testing

Para proveedores OIDC/SAML se deberán probar:

successful authentication;
invalid assertion;
invalid issuer;
invalid audience;
expired assertion;
missing claims;
user mapping;
organization mapping;
account linking;
provider failure.

La arquitectura contempla integración con proveedores como Entra ID, Google, Okta, Auth0, SAML2 y OIDC.

34. End-to-End Testing

Los E2E validarán flujos completos.

Ejemplo:

User Registration
       ↓
Login
       ↓
MFA
       ↓
Session
       ↓
Access Token
       ↓
API Request
       ↓
Authorization
       ↓
Business Operation
       ↓
Audit
       ↓
Domain Event

Otro:

Login
 ↓
Refresh
 ↓
Rotation
 ↓
Logout
 ↓
Refresh Attempt
 ↓
DENY
35. Performance Testing

Se validarán los objetivos:

Authentication P95 < 500 ms
General API P95 < 300 ms

Se probarán:

normal load;
peak load;
sustained load;
concurrent authentication;
concurrent token refresh;
authorization throughput;
Redis latency;
PostgreSQL performance.
36. Load Testing

Se deberá evaluar comportamiento con:

N concurrent users
N login requests
N refresh requests
N authorization decisions

La cifra concreta dependerá de la capacidad objetivo de cada entorno.

No se debe establecer artificialmente un número global sin conocer el capacity planning.

37. Stress Testing

Se incrementará progresivamente la carga hasta identificar:

Capacity Limit

Se observará:

latency;
error rate;
CPU;
memory;
DB connections;
Redis;
event backlog.
38. Resilience Testing

Se probarán fallas controladas:

PostgreSQL unavailable
Redis unavailable
Event Platform unavailable
KMS unavailable
External IdP unavailable
Network latency
Service restart
Publisher crash

El comportamiento esperado debe ser seguro:

Failure
  ↓
Controlled Degradation
  ↓
No Security Bypass
39. Chaos Testing

En etapas posteriores se podrán ejecutar pruebas de Chaos Engineering.

Ejemplos:

Kill Identity instance
Kill Redis
Delay database
Drop event broker connection
Rotate key during traffic

Objetivo:

verificar que el sistema pueda fallar sin producir una autorización insegura.

40. Regression Testing

Cada cambio de Identity deberá ejecutar una suite de regresión.

Especialmente cuando se modifiquen:

authentication;
roles;
permissions;
sessions;
JWT;
refresh tokens;
MFA;
security policies;
database migrations.
41. Observability Testing

Se deberá comprobar que operaciones críticas produzcan:

Logs
Metrics
Traces
Audit
Events

y que puedan correlacionarse mediante:

correlation_id
trace_id
request_id

sin incluir secretos.

42. Test Data Strategy

Los tests deberán utilizar datos sintéticos.

Ejemplo:

user@test.evoxa.local
tenant-test-001
client-test-001

No se utilizarán:

datos reales de clientes;
passwords reales;
tokens reales;
MFA secrets reales;
información clínica real.

Esto es especialmente importante para EVOXA debido al carácter potencialmente sensible de los datos manejados por la plataforma.

43. Test Environment

Se recomienda disponer de:

Local
Development
QA
Staging
Production-like

Los tests de integración y E2E deben ejecutarse en entornos reproducibles.

Los tests destructivos/Chaos no deben ejecutarse contra Production sin un procedimiento específico autorizado.

44. Quality Gates

Una versión de Identity no deberá avanzar si falla cualquiera de las condiciones críticas.

Conceptualmente:

Build
 ↓
Unit Tests
 ↓
Component Tests
 ↓
Integration
 ↓
API Contract
 ↓
Security
 ↓
Performance
 ↓
E2E
 ↓
Quality Gate

Resultado:

PASS → Release
FAIL → Block
45. Release Blocking Conditions

La release deberá bloquearse ante:

tests críticos fallidos;
cross-tenant access;
authorization bypass;
authentication bypass;
privilege escalation;
refresh-token reuse bypass;
MFA bypass;
critical/high security vulnerabilities;
API contract breaking change no aprobado;
corrupción de datos;
pérdida de eventos críticos;
incumplimiento de quality gates.
46. Traceability

Cada requisito crítico debe poder relacionarse:

Requirement
    ↓
Implementation
    ↓
Test
    ↓
Result
    ↓
Release

Ejemplo:

BR-IDENTITY-TENANT-001
        ↓
TenantAuthorizationPolicy
        ↓
TEST-AUTHZ-TENANT-001
        ↓
PASS

Esto es consistente con el modelo de trazabilidad del Blueprint:

Persona
 → User Story
 → Workflow
 → Business Rules
 → API
 → Events
 → Acceptance Tests
47. Test Naming

Se recomienda una nomenclatura consistente:

test_<component>_<scenario>_<expected_result>

Ejemplo:

test_refresh_token_reuse_revokes_family
test_cross_tenant_client_access_is_denied
test_mfa_required_for_sensitive_operation

Esto facilita mantenimiento y diagnóstico.

48. CI/CD Integration

Los tests deberán integrarse en CI/CD:

Commit
  ↓
Lint
  ↓
Unit
  ↓
Component
  ↓
Integration
  ↓
Security
  ↓
Contract
  ↓
Build
  ↓
Deploy QA
  ↓
E2E
  ↓
Quality Gate

La estrategia de release de EVOXA contempla CI/CD, quality gates, promoción de ambientes y verificación posterior al despliegue.

49. Test Reporting

Cada ejecución deberá producir información como:

Test Suite
Passed
Failed
Skipped
Duration
Coverage
Security Findings
Performance Metrics

Los resultados deberán poder relacionarse con:

commit
build
version
environment
deployment
50. Definition of Done

Una funcionalidad de Identity no estará terminada hasta cumplir:

Code
 +
Unit Tests
 +
Component Tests
 +
Integration Tests
 +
API Contract
 +
Security Tests
 +
Observability
 +
Documentation

y cuando corresponda:

Performance
E2E
Resilience
51. Consequences
Positivas
Mayor confianza en Identity.
Reducción del riesgo de vulnerabilidades.
Detección temprana de regresiones.
Protección explícita de tenant isolation.
Validación de token lifecycle.
Mayor seguridad en MFA.
Mejor calidad de APIs.
Mayor trazabilidad.
Releases más controladas.
Negativas
Mayor tiempo de desarrollo inicial.
Mayor complejidad de CI/CD.
Necesidad de ambientes de testing.
Necesidad de datos sintéticos.
Costos de ejecución de pruebas.
Las pruebas E2E, performance y chaos requieren mayor infraestructura.
52. Implementation Direction

La estructura conceptual podría ser:

apps/api/tests/

├── unit/
│   └── identity/
│
├── component/
│   └── identity/
│
├── integration/
│   └── identity/
│
├── contract/
│   └── identity/
│
├── e2e/
│   └── identity/
│
├── security/
│   └── identity/
│
├── performance/
│   └── identity/
│
└── resilience/
    └── identity/

Esto es una dirección de implementación derivada, no una estructura prescrita literalmente por el Blueprint.

53. Final Decision Summary

La estrategia queda:

                         IDENTITY TESTING
                                │
             ┌──────────────────┼──────────────────┐
             │                  │                  │
             ▼                  ▼                  ▼
          Functional         Security          Quality
             │                  │                  │
       ┌─────┼─────┐       ┌────┼────┐        ┌────┼────┐
       ▼     ▼     ▼       ▼    ▼    ▼        ▼    ▼    ▼
     Unit  API   E2E      Auth Tenant MFA    Perf Resilience
       │     │     │       │    │    │        │      │
       └─────┴─────┴───────┴────┴────┴────────┴──────┘
                                │
                                ▼
                         Quality Gates
                                │
                       ┌────────┴────────┐
                       ▼                 ▼
                     PASS              FAIL
                       │                 │
                       ▼                 ▼
                    Release             Block
Decisión principal

EVOXA Identity adoptará una estrategia de testing multinivel basada en Unit, Component, Integration, API/Contract y End-to-End Testing, complementada por pruebas específicas de Security, Performance, Concurrency, Resilience, Observability y, cuando corresponda, AI Security/Evaluation. Identity tendrá como objetivo ≥85% de cobertura general y ≥95% en áreas críticas. Los quality gates bloquearán releases ante fallos de seguridad críticos, bypass de autenticación/autorización, cross-tenant access, MFA bypass, problemas de refresh-token rotation, corrupción de datos, pérdida de eventos críticos o incumplimiento de contratos.

Estado: PROPOSED.

54. Cadena de ADRs de Identity hasta ahora

Con este ADR ya tenemos una cadena bastante coherente:

ADR-001
Identity Architecture & Technology
        ↓
ADR-002
JWT & Token Lifecycle
        ↓
ADR-003
Refresh Token Security & Sessions
        ↓
ADR-005
Authorization & Policy Engine
        ↓
ADR-006
Persistence Strategy
        ↓
ADR-007
Event & Outbox Strategy
        ↓
ADR-008
Cryptographic Key Management
        ↓
ADR-009
Client Token Storage
        ↓
ADR-010
Testing Strategy

Y el siguiente paso lógico de la serie sería ADR-IDENTITY-011 — Identity Deployment & Runtime Architecture, donde cerraríamos las decisiones de ejecución, ambientes, contenedores, configuración, health checks, escalamiento, observabilidad operacional y estrategia de despliegue de Identity.
