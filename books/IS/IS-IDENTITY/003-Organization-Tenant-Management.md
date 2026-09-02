IS-IDENTITY-003 — Organization & Tenant Management

Implementation Story ID: IS-IDENTITY-003
Título: Organization & Tenant Management
Dominio: Identity
ESP: ESP-0001 — Identity
Prioridad: Critical
Estado: Draft
Versión: 1.0.0
Repositorio documental: evoxa-docs
Repositorio de implementación: evoxa-platform

1. Propósito

Esta Implementation Story define la capacidad de crear, administrar y controlar organizaciones dentro de EVOXA, estableciendo la organización como el límite lógico de tenant isolation de la plataforma.

La organización representa el contexto empresarial o institucional dentro del cual operan usuarios, memberships, roles, permisos y posteriormente los demás dominios de EVOXA.

La historia debe establecer una separación clara entre:

User
  ↓
Identity

Organization
  ↓
Tenant

Membership
  ↓
Relationship User ↔ Organization

La documentación arquitectónica de EVOXA establece que Identity es responsable de Identity, Organizations, Memberships, Roles, Permissions, Sessions y Security, mientras que el aislamiento de tenants forma parte de las responsabilidades de seguridad de la plataforma.

2. Business Goal

EVOXA necesita una estructura multi-tenant que permita que diferentes organizaciones utilicen la plataforma sin mezclar sus datos ni sus contextos de seguridad.

La capacidad debe permitir:

Crear organizaciones.
Identificar organizaciones de manera única.
Administrar su ciclo de vida.
Consultar organizaciones.
Actualizar información organizacional.
Suspender organizaciones.
Reactivar organizaciones.
Aplicar tenant isolation.
Servir como contexto para Membership.
Servir como contexto para autorización.
Permitir que los dominios posteriores puedan asociar sus recursos a una organización.
Mantener trazabilidad y auditoría.
3. Persona
Personas principales
Platform Administrator
Organization Administrator
Organization Owner
Team Manager
Coach
Nutritionist
Physiotherapist

La autorización concreta para cada operación será responsabilidad de Authorization & Policy Engine.

4. Journey
Crear Organization
       ↓
Organization PENDING
       ↓
Configuración inicial
       ↓
Organization ACTIVE
       ↓
Operación normal
       ↓
SUSPENDED
       ↓
Reactivated / Administrative Resolution
       ↓
ACTIVE
5. Feature Pack

Identity → Organization & Tenant Management

Incluye:

Organization creation
Organization identity
Organization lifecycle
Organization retrieval
Organization update
Organization suspension
Organization reactivation
Tenant context
Tenant isolation
Organization metadata
Audit integration
Domain events
6. Alcance
Incluido

Esta historia cubre:

Entidad Organization.
OrganizationId.
Identidad única.
Datos básicos de organización.
Estado.
Lifecycle.
Creación.
Consulta.
Actualización.
Suspensión.
Reactivación.
Repository contract.
Application use cases.
Domain rules.
Domain events.
Tenant context.
Audit integration.
Tests.
Fuera de alcance

No se implementa aquí:

Membership.
Invitaciones de usuarios.
Role assignment.
Permission assignment.
Authentication.
JWT.
Sessions.
MFA.
Billing.
Subscription management.
Organización física/deportiva específica.
Human Digital Twin.
Nutrition.
Training.
UI.
Mobile.
Kubernetes.

Membership será desarrollado en:

IS-IDENTITY-004 — Membership Management.

7. Organization Entity

La entidad conceptual:

Organization
├── id
├── name
├── status
├── created_at
├── updated_at
└── lifecycle metadata

La organización no debe almacenar información propia de otros dominios.

Por ejemplo:

Organization
    └── Identity / Tenant information

HumanDigitalTwin
    └── Health information

Training
    └── Training information

Nutrition
    └── Nutrition information

Esto mantiene el principio de Domain Ownership.

8. OrganizationId

Se debe implementar un Value Object:

OrganizationId

Características:

Único.
Estable.
Independiente del nombre.
Independiente del dominio.
No reutilizable.
No dependiente de datos modificables.

El nombre de una organización puede cambiar.

Su OrganizationId no.

9. Organization Name

El nombre es un atributo mutable de la organización.

Debe:

Validarse.
Tener longitud mínima/máxima definida por el contrato.
Evitar valores vacíos.
Permitir actualización.
No utilizarse como identificador técnico.

Las reglas exactas de longitud/formato deberán quedar establecidas en el contrato de dominio/API correspondiente; el Blueprint disponible no prescribe aquí valores numéricos específicos.

10. Organization Lifecycle

Se establece:

PENDING
ACTIVE
SUSPENDED
PENDING

Organización creada pero todavía no completamente activada.

ACTIVE

Organización habilitada para operar normalmente.

SUSPENDED

Organización temporalmente restringida.

11. Lifecycle Transitions

Transiciones conceptuales:

PENDING ─────→ ACTIVE
   │
   └──────────→ SUSPENDED

ACTIVE ──────→ SUSPENDED

SUSPENDED ───→ ACTIVE

El dominio debe controlar las transiciones válidas.

No se debe permitir modificar el estado simplemente asignando un string arbitrario.

Ejemplo incorrecto:

organization.status = "whatever"

El cambio debe pasar por una operación de dominio.

12. User Stories
US-IDENTITY-003-01 — Crear organización

Como administrador autorizado
Quiero crear una organización
Para establecer un nuevo tenant dentro de EVOXA.

Acceptance Criteria

AC-001

Given que existe una solicitud válida
When se crea una organización
Then se genera un OrganizationId único
And la organización queda registrada.

AC-002

Given que faltan datos obligatorios
When se intenta crear la organización
Then la operación debe ser rechazada.

AC-003

Given que la organización fue creada correctamente
When finaliza la operación
Then debe generarse OrganizationCreated.
13. US-IDENTITY-003-02 — Consultar organización

Como actor autorizado
Quiero consultar una organización
Para conocer su información y estado.

Acceptance Criteria

AC-004

Given que la organización existe
When un actor autorizado la consulta
Then el sistema devuelve la información permitida.

AC-005

Given que la organización no existe
When se consulta su OrganizationId
Then debe devolverse OrganizationNotFound.
14. US-IDENTITY-003-03 — Actualizar organización

Como administrador autorizado
Quiero modificar los datos de una organización
Para mantener su información actualizada.

Acceptance Criteria

AC-006

Given que la organización existe
When se modifica un atributo permitido
Then el cambio debe persistirse.

AC-007

Given que un actor no tiene autorización
When intenta modificar la organización
Then la operación debe ser rechazada.
15. US-IDENTITY-003-04 — Suspender organización

Como administrador autorizado
Quiero suspender una organización
Para impedir temporalmente sus operaciones.

Acceptance Criteria

AC-008

Given que la organización está ACTIVE
When un actor autorizado la suspende
Then pasa a SUSPENDED
And se genera OrganizationSuspended.

La suspensión de una organización debe convertirse posteriormente en una señal para:

Authorization.
Membership.
Sessions.
Refresh Tokens.
Security Policies.
Dominios dependientes.

Estos componentes deberán reaccionar mediante sus propios mecanismos.

16. US-IDENTITY-003-05 — Reactivar organización

Como administrador autorizado
Quiero reactivar una organización suspendida
Para permitir nuevamente sus operaciones.

Acceptance Criteria

AC-009

Given que la organización está SUSPENDED
When un actor autorizado solicita su reactivación
Then pasa a ACTIVE
And se genera OrganizationReactivated.
17. Tenant Boundary

La organización constituye el tenant boundary lógico.

Conceptualmente:

                    EVOXA
                      │
        ┌─────────────┼─────────────┐
        │             │             │
    Tenant A      Tenant B      Tenant C
        │             │             │
      Users         Users         Users
        │             │             │
      Data          Data          Data

Un usuario que pertenece a:

Organization A

no debe obtener acceso automáticamente a:

Organization B
18. User ≠ Organization

Es fundamental mantener esta separación:

User
  │
  │
  └── Membership
          │
          ↓
     Organization

Un usuario puede potencialmente pertenecer a varias organizaciones.

Por ello:

Organization no debe contener una lista persistente de usuarios como mecanismo de autoridad.

La relación pertenece a Membership.

19. Tenant Context

Las operaciones de negocio que dependan de una organización deben poder recibir un contexto equivalente a:

TenantContext
├── organization_id
├── actor_user_id
├── membership_id
└── authorization_context

Este contexto será utilizado posteriormente por:

Authorization.
Business Domains.
Event Platform.
Audit.
API.
Data access.
20. Tenant Isolation

La arquitectura de EVOXA establece como requisito de seguridad la separación entre tenants.

Por tanto, cualquier acceso a recursos organizacionales debe validar:

Actor
   ↓
Membership
   ↓
Organization
   ↓
Resource

No es suficiente confiar únicamente en:

organization_id

en una solicitud HTTP.

Debe existir una validación de contexto y autorización.

21. Cross-Tenant Access

Por defecto:

Organization A
       X
Organization B

No se permite acceso cruzado.

Un acceso cross-tenant solo podrá existir mediante una capacidad explícitamente autorizada por la arquitectura/política de EVOXA.

Por ejemplo, determinadas operaciones administrativas de plataforma podrían requerir acceso SYSTEM o una política especial.

22. Organization Status Effects

Cuando una organización pasa a SUSPENDED, los componentes dependientes deberán poder detectar esta condición.

Conceptualmente:

OrganizationSuspended
        │
        ├── Authorization
        ├── Membership
        ├── Session Management
        ├── Refresh Token Lifecycle
        ├── Notifications
        └── Analytics

La organización no debe llamar directamente a todos estos servicios.

El mecanismo preferente será mediante eventos y políticas.

Esto mantiene el desacoplamiento establecido por la arquitectura event-driven.

23. Domain Rules
BR-IDENTITY-003-01

Cada organización debe tener un OrganizationId único.

BR-IDENTITY-003-02

El nombre de la organización no es su identidad técnica.

BR-IDENTITY-003-03

Las transiciones de estado deben ser validadas por el dominio.

BR-IDENTITY-003-04

Una organización SUSPENDED no debe considerarse operativamente equivalente a ACTIVE.

BR-IDENTITY-003-05

El acceso a recursos de una organización debe respetar tenant isolation.

BR-IDENTITY-003-06

La relación User ↔ Organization debe realizarse mediante Membership.

BR-IDENTITY-003-07

Los cambios relevantes del lifecycle deben producir eventos.

BR-IDENTITY-003-08

Las operaciones administrativas deben ser auditables.

BR-IDENTITY-003-09

La organización no debe almacenar información perteneciente a otros dominios.

BR-IDENTITY-003-10

Los dominios externos no deben acceder directamente a las tablas internas de Organization.

24. Application Layer

Casos de uso:

CreateOrganization
GetOrganization
UpdateOrganization
SuspendOrganization
ReactivateOrganization

Flujo:

API
 ↓
Application Service
 ↓
Authorization / Policy
 ↓
Organization Domain
 ↓
Repository
 ↓
Infrastructure
25. Repository Contract

Debe definirse:

OrganizationRepository

Operaciones conceptuales:

create()
get_by_id()
update()
exists()
save()

La interfaz pertenece al dominio.

La implementación concreta pertenece a Infrastructure.

26. Domain Events

Esta historia genera:

OrganizationCreated
OrganizationUpdated
OrganizationSuspended
OrganizationReactivated

Los eventos deben utilizar el envelope definido en la arquitectura de Identity:

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
27. OrganizationCreated

Se genera al crear correctamente una organización.

Payload conceptual:

{
    organization_id,
    status,
    created_at
}

No debe contener secretos ni información sensible innecesaria.

28. OrganizationUpdated

Se genera cuando se modifica información relevante de la organización.

Debe permitir identificar qué cambió sin exponer datos innecesarios.

29. OrganizationSuspended

Se genera cuando:

ACTIVE → SUSPENDED

Este evento permite que los componentes dependientes reaccionen sin introducir acoplamiento directo.

30. OrganizationReactivated

Se genera cuando:

SUSPENDED → ACTIVE

Los consumidores pueden utilizar este evento para restablecer comportamientos dependientes de la organización, sujeto a sus propias reglas de seguridad.

31. Audit

Las operaciones siguientes deben ser auditables:

OrganizationCreated
OrganizationUpdated
OrganizationSuspended
OrganizationReactivated

El registro debe incluir conceptualmente:

actor
organization
action
timestamp
result
correlation_id
metadata

Nunca:

password
tokens
MFA secrets
recovery codes
private keys
32. Authorization Boundary

Organization Management no implementa su propio sistema de permisos.

Debe consumir el contexto de autorización proporcionado por Identity.

Ejemplo:

Actor
 ↓
Authentication
 ↓
Security Context
 ↓
Authorization
 ↓
Policy Engine
 ↓
Organization Management

Esto es consistente con la decisión de separar Authentication de Authorization y utilizar RBAC + ABAC + Tenant Isolation.

33. API Requirements

El namespace será:

/api/v1/identity

Endpoints conceptuales:

POST   /organizations
GET    /organizations/{organization_id}
PATCH  /organizations/{organization_id}
POST   /organizations/{organization_id}/suspend
POST   /organizations/{organization_id}/reactivate

La definición definitiva de OpenAPI pertenece a:

IS-IDENTITY-014 — Identity API Contracts.

Debe utilizar:

OpenAPI 3.1.
Standard error envelope.
Authorization.
Tenant isolation.
Correlation ID.
Validación.
Idempotency-Key cuando corresponda.
Auditability.
34. Error Model

Errores conceptuales:

OrganizationNotFound
OrganizationAlreadyExists
InvalidOrganizationState
InvalidOrganizationTransition
OrganizationSuspended
Unauthorized
Forbidden
TenantAccessDenied

Los códigos definitivos deberán mantener la taxonomía común de Identity.

35. Persistence Requirements

PostgreSQL será la fuente transaccional de verdad según ADR-IDENTITY-006.

Tabla conceptual:

organizations

Campos conceptuales:

id
name
status
created_at
updated_at

La definición física definitiva deberá realizarse durante IS-IDENTITY-015 — Identity Persistence.

36. Concurrency

Los cambios de lifecycle deben ser seguros frente a operaciones simultáneas.

Ejemplo:

Request A:
ACTIVE → SUSPENDED

Request B:
ACTIVE → SUSPENDED

El sistema debe producir un resultado consistente y evitar eventos duplicados incorrectos.

También debe protegerse el caso:

Request A:
ACTIVE → SUSPENDED

Request B:
ACTIVE → ACTIVE

La estrategia concreta de optimistic/pessimistic locking deberá definirse en la implementación de Persistence.

37. Idempotency

Las operaciones administrativas deberán tener comportamiento determinista.

Ejemplo:

Suspend Organization A
Suspend Organization A
Suspend Organization A

No deberían generarse múltiples cambios semánticos de estado.

La implementación definitiva de Idempotency-Key y persistencia de idempotencia será cubierta por API/Persistence.

38. Security Requirements

La implementación debe garantizar:

Fail closed.
Least privilege.
Tenant isolation.
No cross-tenant access accidental.
Validación de autorización.
Auditabilidad.
No secretos en eventos.
No secretos en logs.
No información sensible innecesaria.
Protección contra enumeración de tenants.
Validación de entrada.
Control de acceso antes de revelar información.
39. Acceptance Criteria Consolidated
ID	Criterio
AC-001	Se puede crear una organización válida
AC-002	Se genera OrganizationId único
AC-003	Se validan datos obligatorios
AC-004	Se puede consultar una organización autorizada
AC-005	Organización inexistente produce OrganizationNotFound
AC-006	Se puede actualizar información permitida
AC-007	Se puede suspender una organización
AC-008	Se puede reactivar una organización suspendida
AC-009	Las transiciones inválidas son rechazadas
AC-010	Se generan eventos de lifecycle
AC-011	Las operaciones relevantes son auditables
AC-012	Se mantiene tenant isolation
AC-013	User ↔ Organization se relaciona mediante Membership
AC-014	No se exponen secretos
AC-015	Las operaciones son seguras frente a concurrencia
AC-016	La implementación respeta Domain/Application/Infrastructure
40. Technical Tasks
TASK-IDENTITY-003-01 — Definir Organization Entity

Crear la entidad Organization.

TASK-IDENTITY-003-02 — Implementar OrganizationId

Crear el Value Object:

OrganizationId
TASK-IDENTITY-003-03 — Definir Organization Status

Implementar:

PENDING
ACTIVE
SUSPENDED
TASK-IDENTITY-003-04 — Implementar Lifecycle Rules

Definir y validar las transiciones permitidas.

TASK-IDENTITY-003-05 — Definir OrganizationRepository

Crear el contrato abstracto:

OrganizationRepository
TASK-IDENTITY-003-06 — Implementar CreateOrganization

El caso de uso debe:

validar input;
crear OrganizationId;
construir entidad;
validar estado inicial;
persistir;
producir OrganizationCreated;
preparar auditoría.
TASK-IDENTITY-003-07 — Implementar GetOrganization

Debe respetar:

autorización;
tenant context;
minimización de datos.
TASK-IDENTITY-003-08 — Implementar UpdateOrganization

Debe permitir únicamente atributos modificables.

TASK-IDENTITY-003-09 — Implementar SuspendOrganization

Debe validar:

ACTIVE → SUSPENDED

y producir:

OrganizationSuspended
TASK-IDENTITY-003-10 — Implementar ReactivateOrganization

Debe validar:

SUSPENDED → ACTIVE

y producir:

OrganizationReactivated
TASK-IDENTITY-003-11 — Implementar Domain Events

Crear:

OrganizationCreated
OrganizationUpdated
OrganizationSuspended
OrganizationReactivated
TASK-IDENTITY-003-12 — Implementar Tenant Context

Definir el contexto necesario para representar:

organization_id
actor_user_id
membership_id
authorization_context
TASK-IDENTITY-003-13 — Integrar AuditService

Conectar las operaciones de Organization con AuditService.

TASK-IDENTITY-003-14 — Definir Persistence Mapping

Preparar:

Organization Entity
        ↓
Persistence Model
        ↓
organizations
TASK-IDENTITY-003-15 — Implementar Concurrency Protection

Proteger cambios concurrentes del lifecycle.

TASK-IDENTITY-003-16 — Implementar Idempotency

Definir y probar operaciones administrativas idempotentes.

TASK-IDENTITY-003-17 — Unit Tests

Cubrir:

Organization.
OrganizationId.
Lifecycle.
State transitions.
Validation.
Errors.
TASK-IDENTITY-003-18 — Application Tests

Cubrir:

CreateOrganization
GetOrganization
UpdateOrganization
SuspendOrganization
ReactivateOrganization
TASK-IDENTITY-003-19 — Tenant Isolation Tests

Probar:

Tenant A → Tenant A   ✅
Tenant A → Tenant B   ❌

incluyendo escenarios con diferentes memberships y actores administrativos.

TASK-IDENTITY-003-20 — Event Tests

Verificar:

OrganizationCreated
OrganizationUpdated
OrganizationSuspended
OrganizationReactivated
TASK-IDENTITY-003-21 — Security Tests

Validar:

Unauthorized.
Forbidden.
Cross-tenant access.
Suspended organization.
Information leakage.
Privilege escalation.
TASK-IDENTITY-003-22 — Traceability

Actualizar:

ESP-0001
    ↓
IS-IDENTITY-003
    ↓
TASK-IDENTITY-003-01 ... 022
    ↓
Source Code
    ↓
Tests
41. Definition of Done

La historia estará terminada cuando:

 Organization Entity implementada.
 OrganizationId implementado.
 Lifecycle implementado.
 Transiciones validadas.
 Repository contract definido.
 CreateOrganization implementado.
 GetOrganization implementado.
 UpdateOrganization implementado.
 SuspendOrganization implementado.
 ReactivateOrganization implementado.
 Domain Events implementados.
 Tenant Context definido.
 Audit integrado.
 Tenant isolation probado.
 Concurrency protection implementada.
 Idempotency validada.
 Unit tests implementados.
 Application tests implementados.
 Security tests implementados.
 Event tests implementados.
 Coverage ≥85% general.
 Coverage ≥95% en áreas críticas de Identity.
 Traceability actualizada.
 Architecture Review aprobado.
 QA Review aprobado.
 Security Review aprobado cuando corresponda.
42. Traceability Matrix
Elemento	Referencia
Blueprint	Architecture Map
Domain	Identity
ESP	ESP-0001
Predecessor	IS-IDENTITY-001
Predecessor	IS-IDENTITY-002
Current IS	IS-IDENTITY-003
ADR	ADR-IDENTITY-001
ADR	ADR-IDENTITY-005
ADR	ADR-IDENTITY-006
ADR	ADR-IDENTITY-007
ADR	ADR-IDENTITY-010
Source	apps/api/app/domains/identity/
Tests	apps/api/tests/identity/

La arquitectura de EVOXA establece que las Implementation Stories forman parte de la cadena formal:

Blueprint
 ↓
Reference Architecture
 ↓
Engineering Standards
 ↓
ESP
 ↓
Implementation Stories
 ↓
Technical Tasks
 ↓
Source Code
 ↓
Testing

y que Identity corresponde al dominio apps/api/app/domains/identity/ y a ESP-0001.

43. Dependencies
Depends on
IS-IDENTITY-001 — Identity Domain Foundation
IS-IDENTITY-002 — User & Account Management
ADR-IDENTITY-001
ADR-IDENTITY-005
ADR-IDENTITY-006
ADR-IDENTITY-007
ADR-IDENTITY-010
Required by
IS-IDENTITY-004 — Membership Management
IS-IDENTITY-005 — Authentication
IS-IDENTITY-006 — Authorization & Permission Evaluation
IS-IDENTITY-007 — Role & Permission Management
IS-IDENTITY-008 — Session Management
IS-IDENTITY-009 — Refresh Token Lifecycle
IS-IDENTITY-010 — Multi-Factor Authentication
44. Architectural Decision Notes

Hay una decisión especialmente importante aquí:

Organization será tratada como el límite de tenant de EVOXA, pero la relación entre User y Organization será responsabilidad de Membership.

Por lo tanto, no construiremos algo como:

User.organization_id

como mecanismo general de pertenencia.

La arquitectura será:

                  User
                   │
                   │
             Membership
              │         │
              │         │
              ↓         ↓
          Organization  Role
              │
              ↓
          Tenant Context
              │
              ↓
       Business Resources

Esto permitirá posteriormente que un mismo usuario pueda participar en múltiples organizaciones sin romper el modelo de identidad.

45. Resultado esperado

Al finalizar IS-IDENTITY-003, EVOXA tendrá formalmente definida la segunda pieza fundamental del modelo multi-tenant:

┌──────────────────────────────────────────────┐
│                    Identity                  │
│                                              │
│   User                                       │
│    │                                         │
│    │                                         │
│ Membership                                   │
│    │                                         │
│    ▼                                         │
│ Organization                                 │
│    │                                         │
│    ▼                                         │
│ Tenant Boundary                              │
└──────────────────────────────────────────────┘

Y la secuencia de implementación queda:

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

IS-IDENTITY-003 queda en estado Draft y, al igual que IS-IDENTITY-001/002, todavía no implica comenzar a programar. Primero debemos completar las IS y Technical Tasks necesarias para que el código posterior pueda trazarse formalmente hasta ESP-0001. La documentación de EVOXA exige precisamente esa trazabilidad ESP → IS → Technical Tasks → Source Code.
