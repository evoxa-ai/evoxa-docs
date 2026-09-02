ADR-IDENTITY-001 — Identity Architecture & Technology Decisions

Proyecto: EVOXA Platform
Dominio: Identity
ADR: ADR-IDENTITY-001
Título: Identity Architecture & Technology Decisions
Estado: Proposed
Versión: 1.0.0
Fecha: 2026-09-01
Relacionado con: ESP-0001 — Identity
Tipo: Architecture Decision Record

1. Contexto

El dominio Identity es una capacidad transversal y crítica de EVOXA. Es responsable de establecer quién es un usuario, a qué organización pertenece, qué puede hacer, cómo se autentica, cómo se mantienen sus sesiones y cómo se registran las operaciones relacionadas con seguridad.

Los 15 capítulos de ESP-0001 — Identity ya definieron los requisitos de:

Domain Model
Architecture
Authentication & Authorization
User & Organization Management
Roles & Permissions
Sessions & Token Management
MFA
Security Policies
Audit & Compliance
API Contracts
Persistence & Data Model
Events & Integration
Testing & Quality
Implementation
Deployment & Operations

El Blueprint establece además que EVOXA debe utilizar una arquitectura:

Domain-Driven
Modular
API-First
Event-Driven
Cloud-Ready
AI-Native
Secure by Design
Observable
Continuously Evolvable.

Por lo tanto, antes de pasar a Implementation Stories y código, debemos convertir las decisiones tecnológicas y arquitectónicas abiertas de Identity en decisiones explícitas.

2. Problema

Si comenzamos directamente a programar Identity sin cerrar estas decisiones, existe riesgo de:

Acoplamiento excesivo.
Decisiones contradictorias entre módulos.
Dependencia de una tecnología específica.
Problemas de escalabilidad.
Inconsistencias de seguridad.
Cambios costosos posteriormente.
Pérdida de trazabilidad entre ESP y código.

El Blueprint establece precisamente una cadena de trazabilidad:

Blueprint
   ↓
Engineering Specifications
   ↓
Implementation Stories
   ↓
Technical Tasks
   ↓
Source Code
   ↓
Testing
   ↓
Deployment

3. Decisión
3.1 Arquitectura general

Identity será implementado como un Domain/Platform Service independiente, siguiendo arquitectura por capas:

┌─────────────────────────────┐
│       Presentation          │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│            API              │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│        Application          │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│           Domain            │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│       Infrastructure        │
└─────────────────────────────┘

Identity deberá ubicarse en:

apps/api/app/domains/identity/

Esta ubicación está explícitamente contemplada en el Architecture Map de EVOXA.

4. Decisión tecnológica
4.1 Backend
Decisión

Para la implementación de Identity se utilizará:

Python + FastAPI

Justificación

Esta decisión encaja con el modelo API-First y permite construir Identity como un servicio HTTP modular con contratos explícitos.

La arquitectura de EVOXA no prescribe en el Blueprint un lenguaje específico para Identity; por lo tanto:

Python + FastAPI es una decisión de implementación derivada, no un requisito literal del Blueprint.

Esta distinción deberá mantenerse en la documentación.

5. API Framework
Decisión

Utilizar:

FastAPI

para la capa API.

Responsabilidades:

HTTP.
Routing.
Request validation.
Response serialization.
OpenAPI.
Authentication middleware.
Authorization integration.
Error handling.

La lógica de dominio no deberá depender directamente de FastAPI.

6. API Architecture

Se utilizará:

REST
+
OpenAPI
+
Versioning

Base:

/api/v1/identity

El Blueprint establece API-First, resource-oriented URLs, versionamiento /api/v1, contratos OpenAPI y backward compatibility cuando sea posible.

7. API Contract

El contrato API será la fuente autoritativa de la interfaz pública.

Flujo:

API Contract
     ↓
OpenAPI
     ↓
Implementation
     ↓
Contract Tests

No se deberá implementar una API pública y posteriormente "documentarla".

8. Database
Decisión

Utilizar:

PostgreSQL

como base de datos transaccional principal de Identity.

Arquitectura:

Identity
   ↓
PostgreSQL

PostgreSQL ya está contemplado por la arquitectura de datos de EVOXA como datastore transaccional principal.

9. Data Access
Decisión propuesta

Utilizar una capa de acceso a datos desacoplada mediante repositories.

La infraestructura podrá utilizar un ORM compatible con Python/PostgreSQL, pero:

El dominio no deberá depender del ORM.

Estructura conceptual:

Domain
   ↓
Repository Interface
   ↓
Infrastructure
   ↓
ORM / PostgreSQL

La selección definitiva del ORM deberá registrarse en un ADR técnico específico si afecta transversalmente al proyecto.

10. Database Migration
Decisión

Las modificaciones de esquema deberán gestionarse mediante migrations versionadas.

No se permitirá que el funcionamiento normal de Production dependa de modificaciones manuales del esquema.

Flujo:

Migration
 ↓
CI
 ↓
QA
 ↓
Staging
 ↓
Production
11. Redis
Decisión

Utilizar:

Redis

como infraestructura complementaria para información temporal y operaciones de alta frecuencia.

Usos previstos:

Rate Limiting
MFA temporary state
Security counters
Short-lived state
Caching
Session acceleration

Redis no será el source of truth principal de Identity.

La arquitectura de datos de EVOXA contempla Redis como mecanismo de caching y datos de acceso rápido.

12. Authentication
Decisión

Identity tendrá una capa propia de Authentication desacoplada del framework HTTP.

Conceptualmente:

AuthenticationService

Será responsable de:

Validar credenciales.
Validar estado del usuario.
Validar contexto organizacional.
Aplicar MFA.
Crear sesión.
Emitir tokens.
Generar eventos.
Generar audit.
13. Password Hashing
Decisión

Las contraseñas utilizarán:

Argon2id

Nunca se almacenarán passwords en texto plano.

La arquitectura de seguridad de EVOXA contempla Argon2id para password hashing.

14. Access Tokens
Decisión

Los access tokens utilizarán:

JWT Bearer Tokens

Los tokens deberán ser:

Firmados.
Expirables.
Validados por issuer.
Validados por audience.
Asociados al subject.
Asociados a una sesión cuando corresponda.

Claims conceptuales:

sub
iss
aud
iat
exp
jti
session_id

No se almacenarán datos sensibles innecesarios en el JWT.

La duración exacta del access token queda como decisión configurable pendiente de un ADR de seguridad específico.

15. JWT Signing
Decisión provisional

La arquitectura utilizará claves asimétricas para la firma de tokens.

El Blueprint contempla:

Ed25519
RSA-4096

como mecanismos criptográficos.

La elección final entre ambos deberá quedar formalizada mediante un ADR específico de cryptographic key management.

Por lo tanto:

Decision:
Asymmetric signing

Pending:
Ed25519 vs RSA-4096
16. Refresh Tokens
Decisión

Los refresh tokens serán:

Persistidos de forma segura.
Asociados a sesiones.
Rotables.
Revocables.
Protegidos contra replay.
Susceptibles de detección de reuse.

Arquitectura:

Access Token
   ↓
short-lived

Refresh Token
   ↓
longer-lived
   ↓
Session
   ↓
Persistence

Los refresh tokens almacenados deberán utilizar una representación segura, evitando almacenar el secreto recuperable en texto plano.

17. Refresh Token Rotation
Decisión

Se utilizará Refresh Token Rotation.

Refresh A
   ↓
Refresh
   ↓
Refresh B

El token anterior deberá quedar invalidado según la política de rotation.

Si se detecta reuse:

Refresh A
   ↓
Already used
   ↓
TOKEN REUSE DETECTED

deberá generarse una señal de seguridad y aplicarse la política de revocación correspondiente.

18. Sessions
Decisión

Las sesiones serán entidades persistentes y revocables.

Lifecycle:

CREATED
   ↓
ACTIVE
   ↓
EXPIRED
   ↓
REVOKED

Esto permitirá:

Logout.
Logout all.
Remote session revocation.
Session listing.
Security investigation.
19. MFA
Decisión

MFA será un componente independiente dentro de Identity.

Factores contemplados:

TOTP
WebAuthn / Passkeys
Email OTP
SMS OTP
Recovery Codes
Trusted Devices

No se obliga a habilitar todos los factores en la primera release.

La arquitectura permitirá incorporarlos progresivamente.

20. Authorization
Decisión

Identity utilizará:

RBAC + ABAC

La arquitectura será:

RBAC
 ↓
Roles
 ↓
Permissions

combinada con:

ABAC
 ↓
Tenant
Ownership
Context
Security Policy

Esto es coherente con el modelo de seguridad definido para EVOXA.

21. Permission Model

Se utilizará inicialmente una convención:

<resource>.<action>

Ejemplos:

clients.read
clients.create
clients.update
clients.delete
training.read
training.update

Las permissions serán asignadas a roles mediante:

Role
 ↓
RolePermission
 ↓
Permission
22. Tenant Model
Decisión

La Organization será tratada como frontera de tenant.

Conceptualmente:

Tenant
   =
Organization

Toda operación que dependa de organización deberá tener un contexto de tenant validado.

No se permitirá acceso cross-tenant.

Esta decisión se encuentra fuertemente alineada con la arquitectura multi-tenant y los requisitos de aislamiento definidos en Identity.

23. User / Organization Relationship

Se mantendrá separación entre:

User
Organization
Membership
Role
Permission

Por tanto:

User
 │
 ├── Membership → Organization A
 │                    └── Roles
 │
 └── Membership → Organization B
                      └── Roles

La capacidad de un usuario de pertenecer a múltiples organizaciones queda como una decisión de modelo que deberá ser confirmada antes de implementación final.

24. Role Scope

Se contemplarán:

SYSTEM
ORGANIZATION

Los roles organizacionales estarán limitados a su tenant.

Los roles de sistema deberán reservarse para capacidades de plataforma.

25. Role Hierarchy
Decisión

No se implementará inicialmente una jerarquía de roles implícita.

Por ejemplo:

Admin
  ≠ automáticamente
Coach

Las permissions deberán ser asignadas explícitamente.

Esto evita escalamiento accidental de privilegios.

Una futura jerarquía requerirá un ADR específico.

26. Security Policy Engine

Identity tendrá una abstracción:

SecurityPolicyEngine

La evaluación podrá producir:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

El motor deberá aplicar:

Deny by Default
Fail Closed
Least Privilege
Zero Trust
27. Audit

Identity tendrá un servicio de auditoría separado:

AuditService

La información de audit deberá permitir reconstruir operaciones críticas sin almacenar secretos.

Nunca deberán registrarse:

password
password_hash
access_token
refresh_token
MFA_secret
recovery_codes
private_keys

La separación entre logs técnicos y audit events es obligatoria.

28. Event Architecture

Identity utilizará arquitectura Event-Driven.

Identity
   ↓
Domain Event
   ↓
Integration Event
   ↓
Event Platform
   ↓
Consumers

Los eventos deberán ser:

Versionados.
Inmutables.
Idempotentemente consumibles.
Correlacionables.
Tenant-aware.

El Blueprint define este modelo como parte de la arquitectura de eventos de EVOXA.

29. Event Delivery

Se utilizará un modelo at-least-once delivery.

Los consumidores deberán soportar:

Duplicate event
Retry
Delay
DLQ
Replay

No se asumirá exactamente-once delivery.

30. Transactional Outbox
Decisión provisional

Identity utilizará Transactional Outbox para los eventos donde la consistencia entre transacción de datos y publicación sea crítica.

┌────────────────────────┐
│ PostgreSQL Transaction │
│                        │
│ Identity change        │
│ +                      │
│ Outbox event           │
└───────────┬────────────┘
            ↓
      Event Publisher
            ↓
      Event Platform

La implementación concreta será definida en el ADR correspondiente.

31. Observability

Identity utilizará:

Logs
Metrics
Traces
Audit
Events

Cada operación relevante deberá poder correlacionarse mediante:

trace_id
correlation_id
request_id
session_id
tenant_id

Esto está alineado con los requisitos de observabilidad de EVOXA.

32. Health Checks

Se implementarán:

/health/live
/health/ready

y, cuando la plataforma de deployment lo requiera:

/health/startup
33. Containerization

Identity será distribuible como container image.

Flujo:

Source
 ↓
Build
 ↓
Container
 ↓
Security Scan
 ↓
Registry
 ↓
Deployment

El artefacto será versionado e inmutable.

34. Deployment

Identity seguirá:

Dev
 ↓
QA
 ↓
Staging
 ↓
Production

con quality gates entre ambientes.

El Blueprint define esta promoción de ambientes y estrategias como rolling, blue-green, canary y progressive delivery.

35. Secrets

Los secretos se gestionarán fuera del código fuente.

Incluye:

Database credentials.
JWT signing keys.
OAuth secrets.
MFA provider secrets.
External provider credentials.

La estrategia concreta de secrets manager será definida por infraestructura.

36. Testing

Identity deberá implementar:

Unit
Component
Integration
API Contract
Security
E2E
Performance
Resilience

Con objetivo:

Identity general ≥ 85%
Critical Identity components ≥ 95%

según la estrategia de calidad definida para EVOXA.

37. Performance

Objetivos:

Authentication P95 < 500 ms
API P95 < 300 ms

Estos son objetivos arquitectónicos de EVOXA.

38. Availability

Identity deberá diseñarse para alcanzar:

Availability ≥ 99.9%

en Production.

39. Disaster Recovery

Identity participará de la estrategia global de DR:

RTO < 1 hour
RPO < 15 minutes

Estos valores son objetivos de la arquitectura de EVOXA.

40. Architecture Summary

La decisión arquitectónica queda resumida:

                         EVOXA
                           │
                           ▼
                    ┌─────────────┐
                    │   Identity  │
                    └──────┬──────┘
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
    Authentication   Authorization       MFA
          │                │                │
          ▼                ▼                ▼
       Sessions      RBAC + ABAC       Factors
          │                │
          └───────┬────────┘
                  ▼
             Security
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
      Audit               Events
        │                   │
        └─────────┬─────────┘
                  ▼
             PostgreSQL
                  │
             ┌────┴────┐
             ▼         ▼
           Redis    Event Platform
41. Decisiones aceptadas en este ADR
Área	Decisión
Architecture	Layered + DDD
Domain location	apps/api/app/domains/identity/
Backend	Python
API framework	FastAPI
API style	REST
API contract	OpenAPI
API version	/api/v1
Database	PostgreSQL
Cache/temporary state	Redis
Password hashing	Argon2id
Access tokens	JWT
Token type	Bearer
Refresh tokens	Rotation
Sessions	Stateful
Authorization	RBAC + ABAC
Tenant boundary	Organization
Audit	Dedicated audit service
Events	Event-Driven
Event delivery	At-least-once
Outbox	Required where applicable
Deployment	Containerized
Environments	Dev → QA → Staging → Production
Testing	Multi-layer
Availability target	≥99.9%
42. Decisiones aún abiertas

Es muy importante no confundir las decisiones anteriores con las que todavía necesitan definición.

🔴 Pendientes
Cryptography
Ed25519 vs RSA-4096.
Key rotation period.
Key storage mechanism.
Tokens
Access token TTL.
Refresh token TTL.
Maximum session lifetime.
Concurrent session limits.
Identity
Multiple organizations per user.
Multiple roles per membership.
Exact System vs Organization role model.
MFA
Default MFA policy.
Mandatory MFA roles.
Recovery process.
Trusted-device lifetime.
Step-up requirements.
Authorization
ABAC implementation mechanism.
Policy engine technology.
Permission caching.
Policy cache invalidation.
Persistence
Exact ORM.
Connection pool configuration.
Audit storage implementation.
Outbox implementation.
External Identity
First OIDC provider.
SAML priority.
Enterprise federation strategy.
Infrastructure
Kubernetes strategy.
Cloud provider.
Container registry.
Secrets manager.
Observability stack.

Estas decisiones no deberán inventarse durante la programación.

43. Consequences
Positivas

Esta arquitectura permite:

Separación clara de responsabilidades.
Evolución independiente.
Testabilidad.
Seguridad centralizada.
Multi-tenancy.
Escalamiento horizontal.
Integración con otros dominios.
Integración futura con AI.
Observabilidad.
Deployment independiente.

La arquitectura modular de EVOXA precisamente busca que los dominios tengan ownership claro y colaboren mediante APIs y eventos.

Negativas

También introduce:

Mayor complejidad inicial.
Más componentes.
Mayor necesidad de observabilidad.
Necesidad de gestionar eventos.
Mayor esfuerzo de testing.
Necesidad de ADRs adicionales.
Requiere disciplina arquitectónica.

Esto es aceptable porque Identity es una capacidad de plataforma y no una funcionalidad aislada.

44. Riesgos
Riesgo 1 — Sobrearquitectura

Mitigación:

Implementar por fases y no activar componentes antes de necesitarlos.

Riesgo 2 — Complejidad de seguridad

Mitigación:

Security-first + tests + ADRs + audit.

Riesgo 3 — Dependencia tecnológica

Mitigación:

Mantener interfaces y dominio independientes del framework.

Riesgo 4 — Eventual inconsistencia

Mitigación:

Outbox + idempotent consumers + retries + DLQ.

Riesgo 5 — Cross-tenant access

Mitigación:

TenantContext + Authorization + automated security tests.

45. ADRs derivados

Este ADR será el ADR principal de Identity, pero no debe intentar contener todas las decisiones.

Se deberán crear ADR específicos:

ADR-IDENTITY-002
JWT & Token Lifecycle

ADR-IDENTITY-003
Refresh Token Security

ADR-IDENTITY-004
MFA Architecture

ADR-IDENTITY-005
Authorization & Policy Engine

ADR-IDENTITY-006
Identity Persistence Strategy

ADR-IDENTITY-007
Identity Event & Outbox Strategy

ADR-IDENTITY-008
Identity Cryptographic Key Management

ADR-IDENTITY-009
Identity Testing Strategy

ADR-IDENTITY-010
Identity Deployment & Infrastructure

Los números son una propuesta de organización, no IDs oficiales hasta que los incorporemos al repositorio.

46. Impacto en los 15 capítulos de ESP-0001
ESP-0001
│
├── 01 Domain Model          ← ADR influye
├── 02 Architecture          ← ADR principal
├── 03 Authentication        ← ADR
├── 04 Users/Organizations   ← ADR
├── 05 Roles/Permissions     ← ADR
├── 06 Sessions/Tokens       ← ADR
├── 07 MFA                   ← ADR
├── 08 Security Policies     ← ADR
├── 09 Audit                 ← ADR
├── 10 API                  ← ADR
├── 11 Persistence           ← ADR
├── 12 Events                ← ADR
├── 13 Testing               ← ADR
├── 14 Implementation        ← ADR
└── 15 Deployment            ← ADR

Por eso este ADR debe considerarse transversal a todo ESP-0001.

47. Estado

Status: Proposed

No debe marcarse todavía como Accepted.

Antes de aprobarlo, debemos cerrar las decisiones marcadas como pendientes que tengan impacto directo en la implementación inicial.

48. Próximo paso

Ahora tenemos una base importante:

ESP-0001 / 01–15
        │
        ▼
ADR-IDENTITY-001
        │
        ├── Architecture
        ├── Technology
        ├── Security
        ├── Persistence
        ├── API
        ├── Events
        ├── Testing
        └── Operations

Mi recomendación es no crear todavía las Implementation Stories.

Primero debemos completar los ADR específicos más críticos, comenzando por:

ADR-IDENTITY-002 — JWT & Token Lifecycle

Ahí podemos cerrar definitivamente access token, refresh token, expiraciones, rotation, reuse detection, claims, signing, revocation, sesiones y logout. Una vez cerrado ese ADR, podremos avanzar al siguiente sin dejar decisiones críticas ambiguas.
