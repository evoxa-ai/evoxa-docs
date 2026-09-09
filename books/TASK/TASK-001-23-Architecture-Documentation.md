TASK-001-23 — Architecture Documentation

Identity Implementation Story: IS-001 — Identity Domain Foundation
ESP principal: ESP-0001/14 — Identity Implementation Requirements
ESP relacionados: ESP-0001/01, /02, /03, /05, /06, /07, /08, /09, /10, /11, /12, /13, /15
Dependencias: TASK-001-01 → TASK-001-22
Prioridad: 🔴 Critical
Estado: 🟢 Ready for Implementation

1. Objetivo

Establecer la documentación arquitectónica oficial de la implementación de Identity, de manera que un desarrollador pueda entrar al proyecto y entender:

qué es Identity;
cómo está organizado;
dónde colocar cada pieza de código;
qué dependencias están permitidas;
qué dependencias están prohibidas;
cuáles son los contratos públicos;
cómo funcionan las transacciones;
cómo se manejan eventos;
cómo se manejan errores;
cómo se aplica seguridad;
cómo se prueba;
cómo se ejecuta la CI;
qué decisiones están respaldadas por ADRs.

La documentación no debe ser un documento descriptivo separado del código.

Debe convertirse en una fuente operativa de orientación para la implementación.

2. Principio fundamental

La documentación debe responder rápidamente:

“Si mañana un desarrollador tiene que implementar una nueva funcionalidad de Identity, ¿puede saber dónde ponerla y qué reglas debe respetar sin preguntarle a otra persona?”

El resultado esperado:

Developer
    ↓
Architecture Documentation
    ↓
Module Boundary
    ↓
Layer
    ↓
Contract
    ↓
Implementation
    ↓
Tests
3. Fuentes de autoridad

La documentación de Identity debe respetar la jerarquía:

Blueprint
   ↓
Reference Architecture
   ↓
Engineering Standards
   ↓
ESP-0001
   ↓
ADR
   ↓
Implementation Story
   ↓
Technical Tasks
   ↓
Source Code

La documentación no puede contradecir una decisión arquitectónica vigente.

Si existe una contradicción, debe corregirse la documentación o abrirse el proceso formal de cambio.

4. Documentación ≠ nueva arquitectura

TASK-001-23 no debe utilizarse para introducir nuevas decisiones arquitectónicas.

Su propósito es:

CONSOLIDATE

no:

INVENT

Por ejemplo, no debemos aprovechar este Task para decidir nuevamente:

PostgreSQL vs otra DB;
Redis vs otra cache;
JWT vs otro mecanismo;
RBAC vs otro modelo.

Esas decisiones ya están representadas en ESPs y ADRs.

5. Architecture Overview

La documentación debe incluir una visión general:

                    EVOXA
                      │
                      ↓
                 Identity Domain
                      │
       ┌──────────────┼──────────────┐
       ↓              ↓              ↓
    Account      Organization     Membership
       │              │              │
       └──────────────┼──────────────┘
                      ↓
                 Authentication
                      │
                      ↓
                 Authorization
                      │
       ┌──────────────┼──────────────┐
       ↓              ↓              ↓
     Policy          MFA          Sessions
       │              │              │
       └──────────────┼──────────────┘
                      ↓
               Audit / Security
                      │
                      ↓
                  Integration
6. Arquitectura por capas

Debe quedar documentado explícitamente:

Presentation
      ↓
API
      ↓
Application
      ↓
Domain
      ↓
Infrastructure

Con las reglas:

API → Application
Application → Domain
Infrastructure → Domain/Application contracts
Domain → nothing external
7. Domain Layer

La documentación debe establecer que Domain contiene:

Entities;
Value Objects;
Aggregates;
Domain Services;
Domain Events;
Domain Rules;
Repository Contracts.

El Domain no debe contener:

❌ FastAPI
❌ SQLAlchemy
❌ Redis
❌ HTTP
❌ PostgreSQL driver
❌ external APIs
8. Application Layer

Application contiene:

Use Cases;
Commands;
Queries;
orchestration;
transaction boundaries;
ports;
application contracts.

Ejemplo:

HTTP
 ↓
API
 ↓
Application Use Case
 ↓
Domain
 ↓
Repository Port

Application no debe convertirse en un segundo Domain.

Las reglas de negocio fundamentales pertenecen al Domain.

9. Infrastructure Layer

Infrastructure contiene implementaciones concretas:

PostgreSQL
Redis
Event Platform adapters
External providers
Cryptographic infrastructure
Persistence

Infrastructure implementa los contratos definidos por las capas superiores.

10. API Layer

API debe encargarse de:

HTTP;
request validation;
authentication transport;
DTOs;
response mapping;
error mapping;
routing;
headers;
OpenAPI.

No debe contener:

❌ business logic
❌ direct SQL
❌ authorization logic duplicada
❌ repository orchestration
11. Identity Module Map

La documentación debe incluir el mapa oficial:

identity/
│
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
12. Module Responsibility Matrix

Cada módulo debe tener una responsabilidad clara.

Module	Responsabilidad
Account	Identidad y ciclo de vida de User
Organization	Organizaciones / tenant
Membership	Relación User ↔ Organization
Authentication	Autenticación
Authorization	Evaluación de acceso
Role & Permission	RBAC
Policy	Security Policies
Session	Sesiones
Token	Access/Refresh Token lifecycle
MFA	Multi-Factor Authentication
Recovery	Recuperación
Federation	Identidades externas
Provisioning	Alta/baja/sincronización
Audit	Evidencia de auditoría
Security Events	Eventos de seguridad
Integration	Integración externa

Esta matriz deberá permanecer sincronizada con la arquitectura real.

13. Module Boundary Rules

La documentación debe indicar:

Module A
   ↓
Public Contract
   ↓
Module B

No:

Module A
   ↓
Module B Database

Ni:

Module A
   ↓
Module B Repository Implementation
14. Cross-Module Communication

Debe documentarse la elección entre:

SYNC
QUERY
EVENT
PUBLISH

según la naturaleza de la interacción.

Ejemplo:

Membership
   ↓
MembershipQuery

o:

Membership
   ↓
MembershipChanged Event
   ↓
Consumer

No debe utilizarse un evento para reemplazar innecesariamente una consulta sincrónica.

15. Forbidden Dependencies

Debe existir una sección explícita:

Prohibido
Domain → API
Domain → Infrastructure
Domain → ORM
Domain → Redis
Domain → HTTP

Module A → Module B DB
Module A → Module B infrastructure

API → Repository
API → DB

Esto debe estar acompañado por architecture tests.

16. Shared Kernel

Debe documentarse:

identity/shared/

como un mecanismo deliberadamente pequeño.

Permitido:

Entity
AggregateRoot
ValueObject
EntityId
DomainEvent
DomainError
Repository contracts
UnitOfWork
Result

No permitido:

User
Membership
Role
Permission
Session
MFA
Policy
Delegation
17. Aggregate Map

La documentación debe incluir los aggregates principales:

User
Organization
Membership
Session
RefreshTokenFamily
MFAFactor
MFAChallenge
SecurityPolicy
Role
Permission
Delegation
AuditEvent
SecurityEvent
OutboxEvent

Debe aclararse que:

La lista representa aggregates conceptuales definidos por la arquitectura; la granularidad final de implementación debe respetar los ESP y contratos correspondientes.

18. Persistence Boundary

Debe documentarse:

PostgreSQL
    ↓
Source of Truth

y:

Redis
    ↓
Cache / Temporary State / Optimization

Además:

Event Platform
    ↓
Integration / Event Distribution
19. Transaction Boundary

Debe existir un diagrama:

Application Use Case
        │
        ↓
   Unit of Work
        │
   ┌────┴─────┐
   ↓          ↓
Business    Outbox
 State       Event
   │          │
   └────┬─────┘
        ↓
     COMMIT

Regla:

Business state y Outbox deben confirmarse dentro de la misma transacción cuando formen parte del mismo cambio de negocio.

20. Event Architecture

Debe documentarse:

Domain Event
      ↓
Aggregate
      ↓
Unit of Work
      ↓
Transactional Outbox
      ↓
Event Publisher
      ↓
Event Platform
      ↓
Consumers

Y:

At-Least-Once
+
Idempotent Consumers
21. Event Rules

Debe quedar explícito:

Events = Facts

No:

Events = Commands

Además:

No global ordering
Scoped ordering when required
Versioned contracts
No secrets
22. Authorization Architecture

La documentación debe incluir el flujo completo:

Authorization Request
        ↓
Request Validation
        ↓
Actor/Auth Validation
        ↓
Tenant Context Validation
        ↓
Context Construction
        ↓
Policy Selection
        ↓
Policy Evaluation
        ↓
Mandatory Security Guards
        ↓
RBAC
        ↓
Ownership
        ↓
ABAC
        ↓
Delegation
        ↓
Policy Composition
        ↓
Conflict Resolution
        ↓
Criticality
        ↓
Freshness
        ↓
Cache Eligibility
        ↓
Decision
        ↓
Audit / Telemetry

Este diagrama debe ser considerado normativo para Identity.

23. Authorization Decisions

Debe documentarse:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

y:

ERROR
INDETERMINATE
UNKNOWN

no pueden transformarse en un ALLOW inseguro.

24. RBAC + ABAC + Ownership + Delegation

Debe quedar explícitamente documentado que Identity no utiliza únicamente RBAC.

Modelo:

RBAC
 +
ABAC
 +
Ownership
 +
Delegation
 +
Policy Engine
 +
Tenant Isolation

La autorización centralizada evita que cada dominio cree su propio sistema de permisos.

25. Criticality Architecture

Debe documentarse:

C0
C1
C2
C3

y:

F0
F1
F2
F3

La criticidad debe determinarse antes de decidir si una decisión puede utilizar cache.

Principio:

Criticality
    ↓
Freshness
    ↓
Cache Eligibility
26. Cache Architecture

Debe documentarse:

PostgreSQL
    ↓
Authoritative State
    ↓
Authorization Runtime
    ↓
Cache

Nunca:

Cache
 ↓
Authority

También deben aparecer:

Security Epoch
Entity Version
Policy Version
Context Fingerprint
Generation
TTL

como conceptos de consistencia.

27. Security State

La documentación debe indicar que:

Revocation

tiene prioridad sobre:

Grant

y que los estados de seguridad pueden invalidar cache aunque el TTL todavía no haya expirado.

28. Authentication Architecture

Debe documentarse:

Request
 ↓
Credential Validation
 ↓
Account State
 ↓
Security Policy
 ↓
MFA if required
 ↓
Session
 ↓
Access Token

Y para refresh:

Refresh Token
 ↓
State Validation
 ↓
Rotation
 ↓
Reuse Detection
 ↓
New Access Token
29. Token Architecture

Debe quedar documentado:

Access Token
    ↓
short-lived
stateless JWT
asymmetric signing

y:

Refresh Token
    ↓
stateful
opaque
rotated
family-based
reuse detection

No se debe documentar un refresh token como un JWT si contradice la decisión establecida.

30. MFA Architecture

Debe aparecer:

MFA Factor
       ≠
MFA Challenge

y:

Policy
  ↓
MFA Required?
  ↓
Challenge
  ↓
Verification
  ↓
Authentication / Step-Up Evidence

Nunca:

Provider failure
 ↓
ALLOW
31. Audit Architecture

Debe existir separación:

Application Logs
Security Events
Audit Events
Domain Events
Metrics
Traces

La documentación debe explicar la finalidad de cada uno.

32. Observability Architecture

Debe documentarse:

Logs
Metrics
Traces
Audit
Security Events

con:

request_id
correlation_id
trace_id
causation_id

cuando corresponda.

33. Error Architecture

Debe incluir:

Domain Error
      ↓
Application Error
      ↓
Mapper
      ↓
API Error Contract

Y:

Internal details
      ❌
Public API
34. Public Contract Architecture

Debe documentarse:

Identity
   │
   └── Public Contracts
           │
           ├── REST API
           ├── Application Contracts
           ├── Authorization Contracts
           ├── Event Contracts
           └── S2S Contracts

Y la regla:

Los consumidores dependen del contrato, no de la implementación.

35. API Documentation

Debe existir una referencia central para:

/api/v1/auth/*
/api/v1/users/*
/api/v1/organizations/*
/api/v1/memberships/*
/api/v1/roles/*
/api/v1/permissions/*
/api/v1/sessions/*
/api/v1/mfa/*
/api/v1/authorization/*
/api/v1/policies/*
/api/v1/audit/*

No necesariamente todos estarán implementados en IS-001.

La documentación debe distinguir:

Defined
Implemented
Planned
36. Testing Architecture

Debe documentarse:

Unit
Integration
API
Architecture
Security
Contract
Performance
Chaos
Recovery

y:

Golden Scenarios
Security Invariants
Mutation
Property-Based
Regression

según las etapas del proyecto.

37. CI Architecture

Debe incluir:

Pull Request
     ↓
CI
     ├── Lint
     ├── Type Check
     ├── Architecture
     ├── Unit
     └── Coverage
             ↓
        Quality Gate

Posteriormente:

Integration
Security
Contract
Performance

se añadirán al pipeline correspondiente.

38. Source Tree Documentation

La documentación debe contener el árbol de referencia:

apps/api/
│
├── app/
│   ├── main.py
│   │
│   ├── api/
│   │   └── v1/
│   │
│   ├── core/
│   │   ├── config/
│   │   ├── security/
│   │   ├── observability/
│   │   ├── dependencies/
│   │   └── infrastructure/
│   │
│   └── domains/
│       └── identity/
│           ├── account/
│           ├── organization/
│           ├── membership/
│           ├── authentication/
│           ├── authorization/
│           ├── role_permission/
│           ├── policy/
│           ├── session/
│           ├── token/
│           ├── mfa/
│           ├── recovery/
│           ├── federation/
│           ├── provisioning/
│           ├── audit/
│           ├── security_events/
│           ├── integration/
│           └── shared/
│
└── tests/
39. Developer Guide

Debe existir una guía sencilla:

Para agregar una nueva funcionalidad
1. Identify module
2. Identify aggregate
3. Identify use case
4. Define domain rules
5. Define repository contract
6. Implement infrastructure adapter
7. Define API contract if required
8. Define event if required
9. Add tests
10. Run CI checks

Esto ayuda a mantener el lifecycle:

IS
 ↓
Technical Task
 ↓
Code
 ↓
Tests
40. “Dónde colocar el código”

La documentación debe permitir responder:

¿Tengo una regla de negocio?
Domain
¿Tengo una operación/caso de uso?
Application
¿Necesito PostgreSQL?
Infrastructure
¿Necesito un endpoint?
API
¿Necesito un contrato transversal?
Shared / Public Contract
¿Necesito publicar un evento?
Domain Event
→ Outbox
→ Integration
41. Architecture Decision Index

Debe existir un índice de ADRs relacionados con Identity.

Por ejemplo:

ADR-001 → Identity Domain
ADR-002 → Access Tokens
ADR-003 → Sessions
...
ADR-045 → Authorization Policy Model
...
ADR-059 → Criticality
...
ADR-062 → Test Catalog
...
ADR-066 → Test Schema Governance
ADR-067 → Module Boundaries
ADR-069 → Dependency Matrix

No es necesario copiar los ADR completos.

La documentación debe indicar:

qué decisión existe
dónde está definida
qué parte de la implementación afecta
42. Traceability Matrix

Debe existir una matriz:

Artefacto	Relación
Blueprint	Arquitectura
ESP-0001	Especificación
ADR	Decisión
IS	Capability implementation
Task	Technical implementation
Code	Implementation
Test	Verification
CI	Quality Gate

Esto permitirá navegar:

Requirement
    ↓
ADR
    ↓
IS
    ↓
Task
    ↓
Code
    ↓
Test
43. Documentation Governance

La documentación debe tener lifecycle:

Draft
 ↓
Reviewed
 ↓
Approved
 ↓
Published
 ↓
Maintained
 ↓
Superseded

Una documentación obsoleta no debe permanecer como si fuera normativa.

44. Change Management

Cuando cambie una decisión importante:

Change Request
      ↓
Impact Analysis
      ↓
ADR Update/New ADR
      ↓
ESP Update
      ↓
Architecture Docs
      ↓
Implementation
      ↓
Tests

No:

Code changed
↓
Documentation forgotten
45. Documentation Quality Gates

La documentación deberá fallar la validación si:

❌ contradicts active ADR
❌ references obsolete architecture
❌ contains undocumented critical dependency
❌ exposes secrets
❌ points to nonexistent module
❌ describes unapproved implementation as final
46. Diagram Standards

Los diagramas deben ser:

simples;
legibles;
consistentes;
versionables;
sin secretos;
sin datos reales.

Preferencia:

Mermaid

cuando sea adecuado, porque permite almacenar los diagramas junto al código/documentación.

47. Documentation Location

Propuesta:

books/
└── blueprint/
    └── engineering/
        └── specifications/
            └── ESP-0001-identity/

La documentación de arquitectura de implementación deberá ubicarse de acuerdo con la estructura documental real del repositorio, evitando crear una segunda fuente de verdad.

Dentro del código puede existir además:

apps/api/README.md

como guía operativa para desarrolladores.

48. No duplicar ESPs

Una regla importante:

ESP
 ≠
README

El README debe explicar cómo trabajar.

El ESP define requisitos.

Los ADRs explican decisiones.

La IS define el trabajo.

Los Tasks definen implementación.

Cada artefacto mantiene su propósito.

49. Documentation Map

El resultado final debería ser navegable:

Identity
│
├── Architecture
│
├── Modules
│
├── Layers
│
├── Aggregates
│
├── Persistence
│
├── Events
│
├── Authentication
│
├── Authorization
│
├── MFA
│
├── API Contracts
│
├── Testing
│
├── Observability
│
├── CI
│
├── Security
│
└── ADR / ESP Traceability
50. Acceptance Criteria
AC-01

Existe documentación arquitectónica oficial de Identity.

AC-02

La documentación describe la arquitectura por capas.

AC-03

Los módulos de Identity están documentados.

AC-04

Las responsabilidades de los módulos están documentadas.

AC-05

Los boundaries entre módulos están documentados.

AC-06

Shared Kernel está documentado.

AC-07

Los aggregates principales están documentados.

AC-08

Los límites de persistencia están documentados.

AC-09

Unit of Work y transaction boundaries están documentados.

AC-10

La arquitectura de eventos está documentada.

AC-11

La arquitectura de Authorization está documentada.

AC-12

Criticality y Freshness están documentadas.

AC-13

La arquitectura de Authentication/MFA/Token está documentada.

AC-14

Los contratos públicos están documentados.

AC-15

Testing y CI están documentados.

AC-16

Observability está documentada.

AC-17

Las dependencias prohibidas están documentadas.

AC-18

Existe traceability con ESPs y ADRs.

AC-19

La documentación no contradice decisiones vigentes.

AC-20

Un desarrollador nuevo puede utilizar la documentación para comenzar una implementación sin necesitar rediseñar la arquitectura.

51. Definition of Done
[ ] Architecture overview
[ ] Layer documentation
[ ] Module map
[ ] Module responsibilities
[ ] Boundary rules
[ ] Shared Kernel rules
[ ] Aggregate map
[ ] Persistence architecture
[ ] Transaction architecture
[ ] Event architecture
[ ] Authentication architecture
[ ] Authorization architecture
[ ] MFA architecture
[ ] Token architecture
[ ] Audit architecture
[ ] Observability architecture
[ ] API contract architecture
[ ] Testing architecture
[ ] CI architecture
[ ] Source tree
[ ] Developer guide
[ ] ADR index
[ ] ESP traceability
[ ] Architecture diagrams
[ ] Documentation governance

Y:

[ ] no critical contradiction
[ ] no undocumented architectural boundary
[ ] no obsolete normative reference
[ ] documentation review completed
52. Traceability
Blueprint
   ↓
Reference Architecture
   ↓
Engineering Standards
   ↓
ESP-0001
   ↓
ADR-IDENTITY-001 → 069
   ↓
IS-001
   ↓
TASK-001-23
   ↓
Architecture Documentation

Especialmente:

ESP-0001/01 → Domain Model
ESP-0001/02 → Architecture
ESP-0001/03 → Authentication / Authorization
ESP-0001/10 → API Contracts
ESP-0001/11 → Persistence
ESP-0001/12 → Events
ESP-0001/13 → Testing
ESP-0001/14 → Implementation
ESP-0001/15 → Deployment
53. Resultado esperado

Después de TASK-001-23 deberíamos poder abrir la documentación y obtener una visión como:

                    IDENTITY
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
      Domain       Application    Infrastructure
        │              │              │
        │              │              │
        └──────────────┼──────────────┘
                       ↓
                      API
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
        Web          Mobile        AI

Y para cada flecha:

¿Qué contrato?
¿Qué dependencia?
¿Qué seguridad?
¿Qué test?
¿Qué ADR?

debe existir una respuesta documentada.

54. Estado final de IS-001 antes de Validation
Task	Descripción	Estado
001-01	API Runtime Foundation	✅
001-02	Identity Domain Package	✅
001-03	Layer Architecture	✅
001-04	Shared Kernel	✅
001-05	Domain Events Foundation	✅
001-06	Repository Contracts	✅
001-07	Unit of Work	✅
001-08	PostgreSQL Infrastructure	✅
001-09	Redis Infrastructure	✅
001-10	Configuration & Secrets	✅
001-11	Security Foundation	✅
001-12	Tenant Context	✅
001-13	Actor Context	✅
001-14	Correlation & Trace Context	✅
001-15	Exception Model	✅
001-16	Dependency Injection	✅
001-17	Architecture Boundary Tests	🟢
001-18	Initial Observability	✅
001-19	Migration Framework	✅
001-20	Test Foundation	✅
001-21	CI Foundation	✅
001-22	Identity Public Contracts	✅
001-23	Architecture Documentation	🟢 Ready
001-24	IS-001 Validation	⏭️ Next
55. Punto importante antes de 001-24

Aquí hay una distinción que debemos mantener muy clara:

TASK-001-23 documenta la arquitectura.

TASK-001-24 la valida.

No debemos declarar que IS-001 está validada solamente porque:

24 Tasks
=
24 documentos

La validación real deberá contrastar:

Documentación
      +
Repositorio
      +
Código
      +
Tests
      +
CI
      +
Traceability

Por eso TASK-001-24 será el verdadero Quality Gate de IS-001.

Si pasa, podremos marcar:

╔════════════════════════════════════╗
║ IS-001 — Identity Domain Foundation║
║                                    ║
║              VALIDATED             ║
╚════════════════════════════════════╝

y entonces sí comenzar IS-002 — User & Account Management, que será el primer paso donde empezaremos a construir funcionalidad de negocio real sobre esta fundación.
