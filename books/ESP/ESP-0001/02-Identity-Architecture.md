ESP-0001 / 02 — Identity Architecture

Document ID: ESP-0001/02
Module: Identity
Version: 1.1.0
Status: Draft — Consolidated
Parent: ESP-0001 — Identity
Previous Specification: ESP-0001/01 — Identity Domain Model
Architecture Parent: BP-0002 — Reference Architecture
Repository: evoxa-platform
Target Path: apps/api/app/domains/identity/

1. Purpose

Este documento define la arquitectura interna y técnica del módulo Identity de EVOXA.

Su objetivo es transformar el modelo de dominio definido en:

ESP-0001/01 — Identity Domain Model

en una arquitectura implementable, manteniendo los principios establecidos por el Blueprint:

Domain-Driven Design;
API-First;
Event-Driven Architecture;
Security by Design;
Privacy by Design;
Zero Trust;
Least Privilege;
Observability;
Modularidad;
Evolución independiente;
Tenant Isolation;
Fail-Closed Authorization;
AI Security Governance.

Identity debe existir como una capacidad central reutilizable por los demás dominios de EVOXA.

No debe implementarse como una copia de lógica de autenticación/autorización dentro de cada dominio.

2. Architectural Position

Identity es un Shared Platform Domain.

Su posición conceptual es:

                         EVOXA PLATFORM
                               │
              ┌────────────────┴────────────────┐
              │                                 │
       Presentation                      Platform Services
              │                                 │
              ▼                                 ▼
        API Gateway ───────────────────────► Identity
                                                │
                 ┌──────────────────────────────┼────────────────────────┐
                 │                              │                        │
                 ▼                              ▼                        ▼
          Authentication                 Authorization               Sessions
                 │                              │                        │
                 └──────────────────────────────┼────────────────────────┘
                                                │
                                                ▼
                                        Identity Domain
                                                │
                         ┌──────────────────────┼─────────────────────┐
                         ▼                      ▼                     ▼
                    PostgreSQL               Redis             Event Platform

Identity proporciona capacidades centrales a:

Web;
Mobile;
APIs;
otros dominios;
servicios internos;
AI Agents;
integraciones externas autorizadas.
3. Architectural Principles

Identity debe cumplir los siguientes principios.

3.1 Domain Ownership

Identity es propietario de sus reglas y estado de identidad.

Los demás dominios no deben acceder directamente a sus tablas internas.

3.2 Centralized Authorization

La autorización debe utilizar el modelo central de Identity.

Los dominios consumidores no deben implementar su propio mecanismo alternativo de RBAC/ABAC que pueda entrar en conflicto con Identity.

El dominio propietario del recurso proporciona el contexto del recurso y sus reglas de negocio.

Identity determina la autorización de seguridad.

3.3 Explicit Contracts

La comunicación entre módulos debe realizarse mediante contratos explícitos.

No se permite:

Module A
   ↓
Direct DB Access
   ↓
Module B Tables

Debe utilizarse:

Module A
   ↓
Contract
   ↓
Module B

o:

Module A
   ↓
Domain/Event Contract
   ↓
Event Platform
   ↓
Module B

Esta regla está definida principalmente por ADR-068 y ADR-069.

4. Architectural Layers

Identity seguirá:

┌───────────────────────────────────────┐
│                API                    │
│ Routes / Controllers / Schemas        │
├───────────────────────────────────────┤
│             APPLICATION               │
│ Use Cases / Commands / Queries        │
├───────────────────────────────────────┤
│                DOMAIN                 │
│ Entities / Aggregates / Policies      │
│ Domain Services / Rules               │
├───────────────────────────────────────┤
│             INFRASTRUCTURE            │
│ DB / Cache / Crypto / Events / IdP    │
└───────────────────────────────────────┘

La dependencia conceptual es:

API
 ↓
Application
 ↓
Domain
 ↑
Infrastructure

Infrastructure implementa las interfaces definidas por el dominio/application.

5. Dependency Rule

El dominio no debe conocer:

PostgreSQL;
Redis;
HTTP;
FastAPI;
SDKs concretos;
proveedores cloud;
proveedores de identidad concretos;
Event Platform concreta;
mecanismos físicos de almacenamiento.

Ejemplo:

❌ Domain → PostgreSQL

❌ Domain → Redis

❌ Domain → FastAPI

❌ Domain → Google SDK

Debe utilizar:

Domain
   ↓
Interface
   ↑
Infrastructure

Por ejemplo:

UserRepository
      ▲
      │
PostgreSQLUserRepository
6. Identity Internal Modules

La arquitectura de Identity se divide en módulos:

Identity
│
├── Account
├── Organization
├── Membership
├── Authentication
├── Authorization
├── Role & Permission
├── Policy
├── Session
├── Token
├── MFA
├── Recovery
├── Federation
├── Provisioning
├── Audit
├── Security Events
└── Integration

Estos módulos corresponden a límites lógicos internos.

La arquitectura inicial será un Modular Monolith, no una colección obligatoria de microservicios.

La extracción futura de módulos sólo deberá realizarse cuando exista evidencia técnica, operativa o de escalabilidad que la justifique.

7. Module Boundary Rules

Cada módulo debe:

poseer sus propias reglas;
controlar sus entidades internas;
exponer únicamente contratos públicos;
evitar acceso directo a repositorios de otros módulos;
publicar eventos bajo su ownership;
respetar tenant isolation;
utilizar las interfaces de integración definidas;
evitar dependencias circulares.

Ejemplo:

Membership
     │
     │ Membership Contract
     ▼
Authorization

No:

Authorization
     │
     └── SELECT memberships ...
8. Module Dependency Direction

La dirección conceptual debe ser:

Account
   │
   ▼
Organization
   │
   ▼
Membership
   │
   ├──────────────► Role & Permission
   │
   ├──────────────► Session
   │
   └──────────────► Authorization
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
            Policy     MFA       Delegation
              │
              ▼
          Criticality
              │
              ▼
           Freshness
              │
              ▼
        Cache Eligibility

Esta representación es conceptual.

La matriz formal de dependencias se rige por ADR-069.

9. Authentication Architecture

Authentication responde:

¿Quién eres?

Flujo:

Client
   │
   ▼
Identity API
   │
   ▼
Authentication
   │
   ├── Credential Validation
   ├── Account State
   ├── Security Policy
   ├── MFA
   ├── Session
   └── Token Issuance
   │
   ▼
Authenticated Context

La autenticación no constituye autorización.

10. Authentication Providers

Identity debe utilizar una abstracción de proveedor:

IdentityProvider
      │
      ├── Local Identity
      ├── OIDC
      └── SAML

La arquitectura debe soportar conceptualmente:

EVOXA Identity;
OIDC providers;
SAML providers;
Entra ID;
Google;
otros proveedores compatibles.

El proveedor concreto no debe ser conocido por el Domain Layer.

11. Federation

La identidad federada debe convertirse en una identidad interna de EVOXA.

Conceptualmente:

External Provider
       │
       ▼
External Identity
       │
       ▼
Federation
       │
       ▼
EVOXA User
       │
       ▼
Membership
       │
       ▼
EVOXA Authorization

Un token externo no debe convertirse automáticamente en la autoridad final de autorización de EVOXA.

La autorización permanece bajo Identity.

Esto está alineado con ADR-040.

12. Authorization Architecture

Esta sección requiere una actualización importante respecto al documento anterior.

Authorization ya no debe representarse simplemente como:

RBAC
 ↓
ABAC
 ↓
Ownership
 ↓
Policy
 ↓
ALLOW / DENY

La arquitectura consolidada es:

Authorization Request
        │
        ▼
Request Validation
        │
        ▼
Authentication Validation
        │
        ▼
Tenant Guard
        │
        ▼
Context Construction
        │
        ▼
Policy Selection
        │
        ▼
Policy Evaluation
        │
        ▼
Mandatory Security Guards
        │
        ▼
RBAC
        │
        ▼
Ownership
        │
        ▼
ABAC
        │
        ▼
Delegation
        │
        ▼
Policy Composition
        │
        ▼
Conflict Resolution
        │
        ▼
Criticality
        │
        ▼
Freshness
        │
        ▼
Cache Eligibility
        │
        ▼
Authorization Decision

Esta evolución deriva de ADR-045 → ADR-059.

13. Authorization Policy Engine

Identity incorpora un Authorization Policy Engine central.

Conceptualmente:

                    Authorization
                          │
                          ▼
                  Policy Engine
                          │
              ┌───────────┼───────────┐
              ▼           ▼           ▼
             RBAC        ABAC      Policies
              │           │           │
              └───────────┼───────────┘
                          ▼
                    Decision Runtime

El Policy Engine:

selecciona políticas;
valida políticas;
evalúa condiciones;
aplica precedencia;
calcula efectos;
determina obligaciones;
participa en la determinación de criticality.

No debe ejecutar código arbitrario.

14. Policy Engine Boundary

El Policy Engine no debe:

ejecutar SQL arbitrario;
realizar llamadas de red durante evaluación;
acceder directamente a secretos;
ejecutar código proporcionado por usuarios;
modificar directamente el estado de negocio;
cambiar por sí mismo las políticas activas.

La evaluación debe ser:

Deterministic
Typed
Versioned
Tenant-Aware
Fail-Closed
Side-Effect Free
15. Policy Runtime

La evaluación utiliza conceptualmente:

AuthorizationRuntime
      │
      ├── RequestValidator
      ├── ContextBuilder
      ├── TenantGuard
      ├── PolicyMatcher
      ├── PolicyEvaluator
      ├── RBACEvaluator
      ├── OwnershipEvaluator
      ├── ABACEvaluator
      ├── DelegationEvaluator
      ├── ConflictResolver
      ├── EffectResolver
      ├── ObligationResolver
      └── DecisionBuilder

El AuthorizationRuntime constituye el punto central de evaluación.

16. Authorization Context

El contexto debe incluir, según corresponda:

Actor
Organization
Membership
Request
Resource
Action
Session
Authentication
MFA
Delegation
Ownership
Risk
Security State
Environment
Time

El contexto debe ser tratado como inmutable durante una evaluación.

17. Tenant Guard

Antes de ejecutar una autorización debe verificarse el tenant.

Request
   │
   ▼
Authenticated User
   │
   ▼
Membership
   │
   ▼
Organization
   │
   ▼
Tenant Context
   │
   ▼
Authorization

Nunca debe confiarse únicamente en:

organization_id

enviado por el cliente.

La incertidumbre del tenant debe resultar en comportamiento fail-closed.

18. Cross-Tenant Security

Por defecto:

Tenant A
   │
   ▼
Resource Tenant B
   │
   ▼
DENY

Cross-tenant access sólo puede existir cuando esté:

explícitamente definido;
autorizado;
limitado;
auditado;
sujeto a policy;
sujeto a los mínimos de seguridad de plataforma.
19. RBAC

RBAC se basa en:

User
 ↓
Membership
 ↓
Role
 ↓
Permission

Una Membership puede poseer múltiples Roles.

No existe una jerarquía implícita entre Roles.

Ejemplo:

Membership
 ├── Coach
 └── TeamManager

Los permisos efectivos deben ser evaluados por Authorization Runtime.

20. ABAC

ABAC permite incorporar atributos contextuales.

Ejemplo:

Actor
+
Organization
+
Membership
+
Resource
+
Ownership
+
Resource State
+
Environment
+
Risk

Ejemplo conceptual:

Coach
AND
belongs_to_organization
AND
owns_client
AND
client_active
21. Ownership

Ownership es una dimensión independiente de RBAC.

Por ejemplo:

Permission:
clients.update

Condition:
resource.owner == actor

Tener el permiso:

clients.update

no implica automáticamente ser propietario del recurso.

22. Delegation

Authorization debe soportar delegación explícita.

Actor A
   │
   │ Delegation
   ▼
Actor B

La delegación debe:

estar explícitamente creada;
estar limitada;
poder revocarse;
respetar tenant;
respetar temporalidad;
ser auditable;
no ampliar los privilegios del delegador.
Delegated Capability
        ≤
Delegator Effective Capability
23. Policy Precedence

La arquitectura utiliza precedencia explícita.

Conceptualmente:

Regulatory / Mandatory Security
             ↓
Platform Security Minimum
             ↓
Security State
             ↓
Tenant Isolation
             ↓
Resource Security
             ↓
Dynamic Risk
             ↓
Context
             ↓
Organization Policy
             ↓
Default

Un nivel inferior no puede debilitar una restricción superior.

24. Policy Effects

Los efectos disponibles son:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

DENY no debe ser confundido con una excepción de implementación.

25. Criticality Architecture

La arquitectura incorpora Authorization Criticality.

Criticality no determina por sí misma si una acción está permitida.

Determina el nivel de seguridad y freshness requerido.

Base Criticality
       +
Dynamic Risk
       +
Context
       +
Security State
       +
Tenant
       +
Delegation
       +
MFA
       +
Platform Minimum
       ↓
Effective Criticality

Niveles:

C0 — Critical Security
C1 — High Security
C2 — Standard
C3 — Low Risk

Regla:

C3 → C2 → C1 → C0

La criticality sólo puede elevarse por condiciones aplicables; nunca reducir un mínimo de plataforma.

26. Freshness Architecture

La arquitectura de autorización incorpora freshness:

F0 — Authoritative
F1 — Security Validated
F2 — Controlled Eventual
F3 — Restricted Degraded

Relación base:

C0 → F0
C1 → F1
C2 → F2-SHORT
C3 → F2-MEDIUM

La decisión final debe respetar la freshness requerida por la operación.

27. Authorization Cache

El cache es una optimización.

No constituye fuente de verdad.

PostgreSQL
    │
    │ Source of Truth
    ▼
Authorization State
    │
    ▼
Redis / L2 Cache
    │
    ▼
L1 Cache

La autorización no debe depender exclusivamente de TTL.

La validez del cache considera:

Tenant
Actor
Membership
Policy Version
Entity Version
Security Epoch
Context Fingerprint
Generation
TTL
Invalidation State
Operation Criticality

Esto deriva de ADR-050 → ADR-059.

28. Cache Invalidation

La propagación de cambios de seguridad utiliza:

State Change
     │
     ▼
PostgreSQL Transaction
     │
     ├── Domain State
     └── Outbox Event
             │
             ▼
       Event Platform
             │
             ▼
    Invalidation Consumer
             │
       ┌─────┴─────┐
       ▼           ▼
      L1           L2

Las invalidaciones deben considerar:

policies;
roles;
permissions;
memberships;
delegation;
ownership;
sessions;
MFA;
security state.

Los eventos son hechos del dominio; el consumidor de invalidación no se convierte en Policy Engine.

29. Security Epoch

Los cambios críticos de seguridad pueden provocar incrementos de Security Epoch.

Conceptualmente:

Security State Change
        ↓
Security Epoch++
        ↓
Existing Cache
        ↓
INVALID

Un cache con un epoch anterior no puede considerarse válido simplemente porque su TTL no haya expirado.

30. Session Architecture

Las Sessions son stateful.

User
 │
 ├── Session A
 ├── Session B
 └── Session C

Una Session puede:

crearse;
validarse;
expirar;
revocarse;
auditarse.

La revocación de Session es un evento de seguridad relevante.

31. Token Architecture

El modelo es:

Authentication
      │
      ▼
Session
      │
      ├──────────────┐
      ▼              ▼
Access Token     Refresh Token
Access Token
short-lived;
firmado asimétricamente;
validable mediante JWKS;
identificado mediante kid;
no contiene secretos;
no contiene el catálogo completo de permisos.
Refresh Token
stateful;
opaque;
protegido mediante hash/HMAC;
rotado;
asociado a Session;
asociado a Token Family;
sujeto a reuse detection.

Basado en ADR-002, ADR-003, ADR-008 y ADR-025.

32. Refresh Token Rotation

La arquitectura utiliza:

Token 1
   ↓
Token 2
   ↓
Token 3
   ↓
Token N

La rotación debe ser atómica.

Si se detecta reuse:

Reuse
 ↓
Security Event
 ↓
Audit
 ↓
Family Revocation
 ↓
Session Revocation

Los detalles de grace period o network retry deben permanecer sujetos a la definición específica de Token Management.

33. MFA Architecture

MFA es una capability interna de Identity.

Componentes:

MFAFactor
MFAChallenge
RecoveryCode
TrustedDevice

Flujo:

Authentication
      │
      ▼
Policy
      │
      ▼
MFA Required?
      │
      ├── NO ──► Session
      │
      └── YES
             │
             ▼
        MFA Challenge
             │
             ▼
          Verify
             │
             ▼
           Session

La Policy decide cuándo.

MFA decide cómo verificar.

34. Recovery Architecture

Recovery constituye una ruta alternativa de autenticación, no un bypass.

Debe soportar conceptualmente:

Password Recovery
Account Recovery
MFA Recovery
Administrative Recovery

Las operaciones de recovery deben:

ser purpose-bound;
ser temporales;
ser de un solo uso;
tener assurance apropiado;
ser auditables.
35. API Gateway Integration

Identity se integra con API Gateway:

Client
  │
  ▼
API Gateway
  │
  ├── TLS
  ├── Rate Limiting
  ├── Request Validation
  ├── Correlation
  └── Edge Controls
  │
  ▼
Identity / Domain API

El Gateway puede realizar validaciones preliminares.

Pero:

Un JWT válido no equivale a una autorización válida.

La autorización completa continúa en Identity/Authorization Runtime.

36. Application Layer

La Application Layer coordina casos de uso.

Ejemplos:

AuthenticateUser
CreateUser
ActivateUser
SuspendUser

CreateOrganization
CreateMembership
RemoveMembership

AssignRole
RevokeRole

CreateSession
RevokeSession
RefreshSession

EnrollMFAFactor
VerifyMFAChallenge

AuthorizeRequest

Debe:

coordinar;
validar;
iniciar transacciones;
invocar dominio;
publicar eventos;
coordinar repositorios.

No debe convertirse en un lugar para esconder reglas de dominio.

37. Domain Layer

La Domain Layer contiene:

Entities
Value Objects
Aggregates
Domain Services
Domain Policies
Domain Events
Repository Interfaces
Authorization Contracts

Ejemplo conceptual:

domain/
├── entities/
├── value_objects/
├── aggregates/
├── services/
├── policies/
├── events/
└── repositories/

La estructura exacta de carpetas se considera una decisión de implementación y podrá refinarse en ESP-0001/14.

38. Infrastructure Layer

Infrastructure implementará adaptadores para:

PostgreSQL
Redis
Cryptography
JWT/JWKS
External Identity Providers
Event Platform
Audit Storage
Secret Manager
KMS

Ejemplo:

Domain
  │
  ├── UserRepository
  ├── SessionRepository
  └── EventPublisher
        ▲
        │
Infrastructure
  │
  ├── PostgreSQL
  ├── Redis
  └── Event Platform
39. PostgreSQL Architecture

PostgreSQL constituye la fuente primaria de verdad transaccional de Identity.

Conceptualmente:

PostgreSQL
├── users
├── organizations
├── memberships
├── roles
├── permissions
├── role_permissions
├── membership_roles
├── sessions
├── refresh_tokens
├── mfa_factors
├── mfa_challenges
├── recovery_codes
├── trusted_devices
├── security_policies
├── audit_events
├── security_events
└── outbox_events

El esquema físico definitivo pertenece a:

ESP-0001/11 — Identity Persistence & Data Model

40. Redis Architecture

Redis puede utilizarse para:

caching;
rate limiting;
estado temporal;
MFA challenges;
aceleración de sesiones;
authorization cache.

Sin embargo:

Redis no constituye la fuente primaria de verdad para datos críticos de seguridad.

La pérdida de Redis no debe producir una autorización permisiva.

41. Event Architecture

Identity utiliza Event-Driven Architecture.

Identity
   │
   ▼
Domain Event
   │
   ▼
Transactional Outbox
   │
   ▼
Event Platform
   │
   ├── Analytics
   ├── Notifications
   ├── Audit
   ├── AI
   └── Other Domains

Los eventos:

representan hechos;
son inmutables;
están versionados;
son tenant-aware;
soportan at-least-once;
requieren consumidores idempotentes;
soportan retry/DLQ/replay.
42. Transactional Outbox

Los cambios de estado y los eventos correspondientes deben persistirse transaccionalmente cuando el caso lo requiera:

┌─────────────────────────────────┐
│ PostgreSQL Transaction          │
│                                 │
│ Identity State Change            │
│             +                   │
│ Outbox Event                    │
└────────────────┬────────────────┘
                 │
                 ▼
          Event Publisher
                 │
                 ▼
          Event Platform

Esto está definido por ADR-007 y ADR-013.

43. Audit Architecture

Identity debe mantener una capacidad de auditoría independiente de los logs operacionales.

Identity Operation
       │
       ├────────────► Business Result
       │
       ├────────────► Audit Event
       │
       └────────────► Security Event

Deben auditarse especialmente:

autenticación;
fallos de autenticación;
autorización sensible;
cambios de roles;
cambios de permisos;
cambios de membership;
cambios MFA;
recuperación;
revocación de Session;
refresh token reuse;
cambios de políticas;
delegaciones;
cambios de security state.
44. Observability

Identity debe soportar:

Logs
Metrics
Traces
Audit Events
Security Events
Domain Events

Las operaciones deben poder correlacionarse mediante:

trace_id
span_id
correlation_id
causation_id

Los AI Agents también deben quedar identificables como actores.

45. AI Integration

Los AI Agents utilizan Identity como cualquier otro actor autorizado.

AI Agent
   │
   ▼
Authentication
   │
   ▼
Authorization
   │
   ▼
Policy
   │
   ▼
Criticality
   │
   ▼
Freshness
   │
   ▼
Decision

No existe un canal privilegiado de AI.

AI no puede:

saltarse Authorization;
reducir Criticality;
desactivar MFA;
cambiar Platform Minimums;
activar directamente una Security Policy;
aprobar una Security Regression;
manipular el cache para obtener autorización.

AI puede ayudar a:

analizar políticas;
generar escenarios;
detectar anomalías;
sugerir cambios;
analizar resultados.

La decisión normativa permanece bajo los mecanismos de Identity.

46. Testing Architecture

Identity debe disponer de testing multinivel:

Unit
Component
Integration
API
E2E
Security
Performance
Chaos
Recovery

Authorization añade:

Golden Scenarios
Security Invariants
Generated Scenarios
Property-Based
Boundary
Mutation
Differential
Regression

El Test Runtime no debe convertirse en el Production Authorization Runtime.

Sin embargo, ambos deben utilizar los mismos contratos normativos.

Esto está definido en ADR-062 → ADR-067.

47. Runtime Contract

El contrato conceptual entre los runtimes es:

Authorization Request
        ↓
Authorization Context
        ↓
Policy Evaluation
        ↓
Criticality Result
        ↓
Freshness Result
        ↓
Cache Eligibility
        ↓
Authorization Decision

El Test Runtime debe poder reproducir esta evaluación utilizando:

Policy Version
Runtime Version
Schema Version
Scenario Version
Invariant Version
Generator Version
Seed
Input Context
Evaluation Time

Esto permite reproducibilidad histórica.

48. Failure Strategy

Identity debe aplicar Fail Closed.

Ejemplos:

Unknown tenant
     ↓
DENY
Authorization error
     ↓
No unsafe ALLOW
Policy unavailable
     ↓
Security-safe failure
Stale critical authorization
     ↓
Fresh evaluation / fail closed

Una degradación operacional nunca debe transformarse en una ampliación de privilegios.

49. Security Architecture

Identity debe aplicar:

Zero Trust
Least Privilege
Defense in Depth
Secure by Default
Fail Closed
Tenant Isolation
Continuous Verification
Auditability

Los secretos no pueden aparecer en:

Source Code
Logs
Traces
Events
Audit Payloads
API Responses
AI Context

La gestión criptográfica se rige por:

ADR-008;
ADR-014;
ADR-027.
50. Performance Requirements

Las operaciones deben respetar los objetivos definidos por la arquitectura de plataforma.

Objetivos principales:

General API P95        < 300 ms
Authentication P95     < 500 ms
Authorization          < 50 ms objective

El rendimiento no puede conseguirse sacrificando:

tenant isolation;
freshness;
revocation;
policy enforcement;
MFA;
auditability;
fail-closed behavior.
51. High Availability

Identity debe diseñarse para ejecución stateless en su capa API.

                Load Balancer
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       API-1       API-2      API-N
          │          │          │
          └──────────┼──────────┘
                     ▼
              PostgreSQL HA
                     │
               Redis HA
                     │
              Event Platform

Las dependencias críticas deben soportar failover.

La pérdida de un nodo API no debe provocar pérdida de estado de Identity.

52. Disaster Recovery

Identity debe cumplir los objetivos de plataforma:

Availability ≥ 99.9%
RTO < 1 hour
RPO < 15 minutes

La recuperación debe validar como mínimo:

Authentication;
Sessions;
Refresh Tokens;
Authorization;
Tenant Isolation;
MFA;
Outbox;
Event publishing;
Security state;
KMS/Secrets.

Un restore de infraestructura no implica automáticamente que el estado de autorización sea válido.

53. Architecture Governance

La arquitectura debe validarse automáticamente cuando sea posible.

Los controles deben detectar:

Circular Dependencies
Direct Cross-Module DB Access
Domain → Infrastructure Dependencies
Unauthorized Imports
Cross-Tenant Access Paths
Security Bypass
Missing Authorization

Los límites arquitectónicos deben poder formar parte de CI/CD.

Esto implementa los principios de ADR-068 y ADR-069.

54. Architectural Rules

Las siguientes reglas son no negociables:

AR-001

Identity es una capacidad central.

AR-002

Los módulos no acceden directamente a tablas de otros módulos.

AR-003

Authorization debe utilizar el Authorization Runtime central.

AR-004

Tenant context debe ser explícito.

AR-005

Cross-tenant access está denegado por defecto.

AR-006

Authorization failures deben fallar cerrado.

AR-007

Criticality no equivale a Authorization Decision.

AR-008

Cache no es source of truth.

AR-009

Security Epoch tiene precedencia sobre TTL.

AR-010

AI utiliza exactamente las mismas restricciones de Identity.

AR-011

Domain Layer no depende de Infrastructure.

AR-012

Events representan facts, no commands.

AR-013

Cambios transaccionales + eventos deben utilizar Outbox cuando corresponda.

AR-014

Los eventos deben ser idempotentemente consumibles.

AR-015

No se deben almacenar secretos en eventos, logs o contexto AI.

AR-016

Los módulos deben comunicarse mediante contratos explícitos.

AR-017

No debe existir privilege amplification mediante delegation.

AR-018

Organization policy no puede debilitar Platform Security Minimums.

AR-019

Test Runtime no puede modificar Production Authorization.

AR-020

Los resultados de seguridad deben ser reproducibles mediante versionado.

55. Architectural Traceability

Este documento se relaciona con:

Área	ADR
Identity Architecture	ADR-001
Authentication	ADR-002
Sessions	ADR-003
Authorization	ADR-005
Persistence	ADR-006
Events	ADR-007
Cryptography	ADR-008
Token Storage	ADR-009
Testing	ADR-010
Runtime	ADR-011
MFA	ADR-023–024
Refresh Tokens	ADR-025
API	ADR-026
Secrets/KMS	ADR-027
Observability	ADR-028
DR	ADR-029–030
HA	ADR-031
Rate Limiting	ADR-032
Account Lifecycle	ADR-033
Password Security	ADR-034
Recovery	ADR-035
Email Verification	ADR-036
API Tokens	ADR-037
S2S	ADR-038
OAuth/OIDC	ADR-039
Federation	ADR-040
Provisioning	ADR-041
Attribute Ownership	ADR-042
Multi-Organization	ADR-043
Roles/Delegation	ADR-044
Policy Model	ADR-045–049
Authorization Cache	ADR-050–057
Criticality	ADR-058–059
Criticality Testing	ADR-062–063
Test Data/Evidence	ADR-064
Schema Evolution	ADR-065
Schema Registry	ADR-066
Runtime Contracts	ADR-067
Module Boundaries	ADR-068
Dependency Matrix	ADR-069
56. Relationship with Other ESP Documents

Este documento no debe duplicar la definición detallada de:

ESP-0001/03
Authentication & Authorization

ESP-0001/05
Roles & Permissions

ESP-0001/06
Sessions & Token Management

ESP-0001/07
MFA

ESP-0001/08
Security Policies

ESP-0001/10
API Contracts

ESP-0001/11
Persistence

ESP-0001/12
Events

ESP-0001/13
Testing

ESP-0001/14
Implementation

ESP-0001/15
Deployment

Este documento establece la arquitectura y los límites.

Los documentos posteriores deben especializarla.

57. Architecture Decision Status

Después de los ADR-001 → 069, las siguientes decisiones ya no deben aparecer como preguntas abiertas en ESP-0001/02:

❌ ¿Identity será central?
❌ ¿Habrá Multi-Organization?
❌ ¿Habrá RBAC + ABAC?
❌ ¿Cómo se manejarán Sessions?
❌ ¿Cómo se manejarán Refresh Tokens?
❌ ¿Habrá MFA?
❌ ¿Existirá Policy Engine?
❌ ¿Habrá Delegation?
❌ ¿Existirá Criticality?
❌ ¿Habrá Authorization Cache?
❌ ¿Cómo se invalidará el cache?
❌ ¿Cómo se manejarán los tests de Criticality?
❌ ¿Cómo se definirán los límites de módulos?

Esas decisiones ya tienen ADR.

58. Remaining Specification Decisions

Quedan decisiones de especificación e implementación, no de arquitectura fundamental.

Principalmente:

1. Aggregate boundaries definitivos
2. Exact module package structure
3. Exact public module contracts
4. Exact Authorization API
5. Exact Policy DSL syntax
6. Exact AST representation
7. Exact PostgreSQL schema
8. Exact Event Catalog
9. Exact cache implementation
10. Exact infrastructure providers
11. Exact deployment topology

Estas deben resolverse en los ESP correspondientes y posteriormente en Technical Tasks.

59. Implementation Readiness

Con esta actualización, ESP-0001/02 pasa de:

Identity Architecture
       +
Several future ADR questions

a:

Identity Architecture
       │
       ├── Domain
       ├── Module Boundaries
       ├── Authentication
       ├── Authorization Runtime
       ├── Policy Engine
       ├── Criticality
       ├── Freshness
       ├── Cache
       ├── Sessions
       ├── Tokens
       ├── MFA
       ├── Events
       ├── Audit
       ├── AI
       ├── Testing
       └── Infrastructure
60. Veredicto de auditoría
ESP-0001/02 original

🟡 Parcialmente alineado

La arquitectura base era correcta, especialmente:

capas;
separación Domain/Infrastructure;
Identity central;
PostgreSQL;
Redis;
Event Platform;
Authentication;
Sessions;
API Gateway.

El documento original efectivamente contiene esos elementos.

ESP-0001/02 consolidado

🟢 Arquitectónicamente alineado con ADR-001 → 069

pero permanece:

Status: Draft

porque todavía depende de la consolidación de los ESP especializados.

61. Qué hemos conseguido con 01 + 02

Ahora tenemos:

ESP-0001/01
Identity Domain Model
        │
        ▼
ESP-0001/02
Identity Architecture
        │
        ├── Domain Boundaries
        ├── Module Boundaries
        ├── Authentication
        ├── Authorization
        ├── Policy Engine
        ├── Criticality
        ├── Freshness
        ├── Cache
        ├── Sessions
        ├── MFA
        ├── Events
        ├── Audit
        └── AI

Esto es importante porque ESP-0001/03 ya no debería inventar otra arquitectura de Authorization. Debe especializar exactamente esta arquitectura.

Siguiente paso

Continuaría con:

ESP-0001/03 — Identity Authentication & Authorization

Ahí debemos hacer una auditoría todavía más cuidadosa porque es probablemente el documento donde existe mayor diferencia entre el ESP original y los ADR-045 → 059. El objetivo será dejar completamente especificado el flujo:

Authentication → Context → Tenant → Policy → RBAC → ABAC → Ownership → Delegation → Criticality → Freshness → Decision,
