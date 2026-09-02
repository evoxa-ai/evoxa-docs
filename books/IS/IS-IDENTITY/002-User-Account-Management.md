IS-IDENTITY-002 — User & Account Management

Implementation Story ID: IS-IDENTITY-002
Título: User & Account Management
Dominio: Identity
ESP: ESP-0001 — Identity
Prioridad: Critical
Estado: Draft
Versión: 1.0.0
Repositorio documental: evoxa-docs
Repositorio de implementación: evoxa-platform

1. Propósito

Esta Implementation Story define la capacidad de gestionar usuarios y sus cuentas dentro de EVOXA, estableciendo el comportamiento necesario para crear, consultar, actualizar, activar, suspender y deshabilitar cuentas.

La historia implementa la capa de gestión de cuentas sobre la fundación definida en IS-IDENTITY-001, pero no implementa todavía autenticación completa, sesiones, MFA ni autorización avanzada.

La cuenta de usuario constituye uno de los elementos fundamentales del dominio Identity y debe poder operar dentro del contexto de una organización/tenant.

2. Business Goal

EVOXA necesita una capacidad centralizada y segura para administrar las cuentas de las personas que utilizan la plataforma.

El sistema debe permitir:

Crear usuarios.
Mantener sus datos básicos.
Controlar su estado de cuenta.
Asociarlos posteriormente a organizaciones mediante Membership.
Evitar duplicidad de identidad.
Aplicar las reglas de ciclo de vida.
Registrar cambios relevantes.
Mantener aislamiento entre tenants.
Preparar la cuenta para autenticación, autorización y MFA.

El objetivo es que ningún otro dominio tenga que implementar su propio concepto de usuario.

3. Persona
Personas principales
Platform Administrator
Organization Administrator
Coach
Nutritionist
Physiotherapist
Team Manager
Usuario final

Los permisos concretos para ejecutar cada operación serán definidos por Authorization & Policy Engine y no por esta historia.

4. Journey
Crear cuenta
     ↓
Cuenta PENDING
     ↓
Completar información
     ↓
Activar cuenta
     ↓
Cuenta ACTIVE
     ↓
Uso normal de EVOXA
     ↓
Suspensión / Deshabilitación
     ↓
Cuenta SUSPENDED / DISABLED
5. Feature Pack

Identity → User & Account Management

Incluye:

User creation
User retrieval
User update
User lifecycle management
User status management
User identity uniqueness
User profile metadata
Tenant-aware user access
Account security state
Audit integration
6. Alcance
Incluido

Esta historia cubre:

Entidad User.
Identidad única del usuario.
Datos básicos de cuenta.
Estado de usuario.
Creación.
Consulta.
Actualización.
Activación.
Suspensión.
Deshabilitación.
Validaciones de dominio.
Reglas de transición de estado.
Repository contract.
Application services.
Domain events.
Audit integration.
Tenant/context awareness.
Tests.
Fuera de alcance

No se implementa aquí:

Login.
Password authentication.
JWT.
Refresh tokens.
Sessions.
MFA.
Authorization engine completo.
Role assignment.
Permission management.
Organization management.
Membership management.
OAuth/OIDC.
UI.
Mobile application.
Kubernetes.
CI/CD.

Estas capacidades serán desarrolladas mediante sus respectivas IS.

7. User Stories
US-IDENTITY-002-01 — Crear usuario

Como administrador autorizado
Quiero crear un usuario
Para permitirle utilizar EVOXA.

Acceptance Criteria

AC-001

Given que existe una solicitud válida de creación de usuario
When se crea el usuario
Then el sistema debe generar una identidad única
And el usuario debe quedar registrado en el dominio Identity.

AC-002

Given que el email ya pertenece a un usuario
When se intenta crear otro usuario con el mismo email
Then la operación debe ser rechazada.

AC-003

Given que faltan datos obligatorios
When se intenta crear el usuario
Then la operación debe ser rechazada
And debe devolverse un error de validación.
8. US-IDENTITY-002-02 — Consultar usuario

Como usuario o administrador autorizado
Quiero consultar información de una cuenta
Para conocer su estado y datos permitidos.

Acceptance Criteria

AC-004

Given que el usuario existe
When una solicitud autorizada consulta su cuenta
Then debe retornarse la información permitida.

AC-005

Given que el usuario no existe
When se consulta su identidad
Then debe retornarse UserNotFound.
9. US-IDENTITY-002-03 — Actualizar usuario

Como usuario o administrador autorizado
Quiero actualizar información de mi cuenta
Para mantener mis datos correctamente.

Acceptance Criteria

AC-006

Given que el usuario existe
When se modifica un campo permitido
Then el cambio debe persistirse.

AC-007

Given que se intenta modificar un atributo protegido sin autorización
When se procesa la solicitud
Then la operación debe ser rechazada.
10. US-IDENTITY-002-04 — Gestionar estado

Como administrador autorizado
Quiero cambiar el estado de una cuenta
Para controlar su acceso al sistema.

Estados definidos:

PENDING
ACTIVE
SUSPENDED
DISABLED
Transiciones
PENDING ───────→ ACTIVE
   │
   └────────────→ DISABLED

ACTIVE ────────→ SUSPENDED
   │
   └────────────→ DISABLED

SUSPENDED ─────→ ACTIVE
   │
   └────────────→ DISABLED

No todas las transiciones deben estar permitidas.

El dominio debe validar cada transición.

11. US-IDENTITY-002-05 — Suspender cuenta
Given que un usuario está ACTIVE
When un actor autorizado solicita suspenderlo
Then el usuario pasa a SUSPENDED
And debe generarse UserSuspended.

La suspensión deberá integrarse posteriormente con:

Session Management
Refresh Token Lifecycle
Authorization
Security Policy Engine

para impedir el uso indebido de una cuenta suspendida.

12. US-IDENTITY-002-06 — Deshabilitar cuenta
Given que un usuario puede ser deshabilitado
When un actor autorizado solicita deshabilitarlo
Then el usuario pasa a DISABLED
And debe generarse UserDisabled.

DISABLED representa un estado más restrictivo que SUSPENDED.

13. User Entity

La entidad conceptual:

User
├── id
├── email
├── status
├── created_at
├── updated_at
└── lifecycle metadata

Otros atributos de perfil pueden evolucionar posteriormente.

La entidad debe evitar contener información perteneciente a otros dominios.

Por ejemplo:

User
    └── Identity data

HumanDigitalTwin
    └── Health / physiological data

Nutrition
    └── Nutrition data

Training
    └── Training data

Esto mantiene la propiedad de datos por dominio.

14. User Identity

El identificador interno debe ser estable y no depender del email.

UserId

El email es un atributo de identidad/login, pero no debe utilizarse como primary identifier del dominio.

15. Email

Debe existir un Value Object:

Email

Responsabilidades:

Validar formato.
Normalizar representación.
Comparar correctamente.
Evitar estados inválidos.

La normalización exacta debe mantenerse centralizada para evitar que cada componente implemente reglas diferentes.

16. Account Lifecycle
PENDING

Cuenta creada pero todavía no completamente activada.

ACTIVE

Cuenta habilitada para operar normalmente.

SUSPENDED

Cuenta temporalmente restringida.

DISABLED

Cuenta deshabilitada.

17. Reglas de negocio
BR-IDENTITY-002-01

Cada usuario debe poseer un identificador único.

BR-IDENTITY-002-02

El email no puede duplicarse dentro del ámbito de identidad definido por la plataforma.

BR-IDENTITY-002-03

Un usuario DISABLED no puede pasar automáticamente a ACTIVE.

La reactivación de cuentas deshabilitadas deberá requerir un flujo explícito de seguridad/administración.

BR-IDENTITY-002-04

Las transiciones de estado deben ser validadas por el dominio.

BR-IDENTITY-002-05

Una cuenta SUSPENDED no debe considerarse equivalente a una cuenta ACTIVE.

BR-IDENTITY-002-06

Los cambios relevantes del lifecycle deben producir eventos de dominio.

BR-IDENTITY-002-07

Las operaciones administrativas deben ser auditables.

BR-IDENTITY-002-08

Ningún dominio debe acceder directamente a las tablas internas de Identity.

BR-IDENTITY-002-09

La información de un usuario debe respetar el contexto de organización/tenant cuando corresponda.

BR-IDENTITY-002-10

La cuenta no debe contener información sensible perteneciente al Human Digital Twin.

18. Application Layer

Se propone una separación mediante casos de uso:

CreateUser
GetUser
UpdateUser
ActivateUser
SuspendUser
DisableUser

Conceptualmente:

API
 ↓
Application Service
 ↓
Domain
 ↓
Repository
 ↓
Infrastructure

La Application Layer coordina la operación, pero las reglas fundamentales del lifecycle permanecen en el dominio.

19. Repository Contract

Se utilizará el contrato establecido en IS-IDENTITY-001:

UserRepository

Operaciones conceptuales:

create()
get_by_id()
get_by_email()
update()
exists()
save()

La implementación concreta queda fuera de la capa de dominio.

Por ejemplo:

domain/repositories/
    user_repository.py

mientras que la implementación PostgreSQL pertenecerá a Infrastructure.

20. Domain Events

Esta historia genera inicialmente:

UserCreated
UserUpdated
UserActivated
UserSuspended
UserDisabled

Cada evento deberá incluir el envelope definido para Identity:

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
21. UserCreated

Se produce cuando una nueva cuenta se crea correctamente.

Payload conceptual:

{
    user_id,
    status,
    created_at
}

No debe contener:

password
password hash
access token
refresh token
MFA secret
recovery codes
private keys
22. UserUpdated

Debe representar un cambio relevante de información de cuenta.

Se recomienda que el evento identifique qué atributos cambiaron, sin incluir información sensible innecesaria.

23. UserActivated

Se produce cuando:

PENDING → ACTIVE

o una transición válida de recuperación/reactivación definida por el dominio.

24. UserSuspended

Se produce cuando:

ACTIVE → SUSPENDED

Debe posteriormente activar las integraciones necesarias para:

revocar sesiones;
impedir refresh;
aplicar políticas de seguridad;
registrar actividad de seguridad.

Estas acciones pertenecen a otros componentes, no deben acoplarse directamente dentro de User.

25. UserDisabled

Representa una deshabilitación permanente/restrictiva de la cuenta.

Debe producir un evento de dominio que permita a otros componentes reaccionar.

26. Audit

Las siguientes operaciones deben ser auditables:

UserCreated
UserUpdated
UserActivated
UserSuspended
UserDisabled

La auditoría debe registrar:

actor
action
target
tenant
timestamp
correlation_id
result
metadata

Nunca:

password
password_hash
tokens
MFA secrets
recovery codes
private keys
27. Tenant Awareness

El modelo de Identity debe estar preparado para:

User
   │
   ├── Membership
   │       │
   │       └── Organization
   │
   └── Security context

Un usuario puede potencialmente pertenecer a más de una organización.

Por ello:

User ≠ Organization Membership

El usuario representa la identidad global.

La relación con una organización pertenece a:

Membership

y será implementada en:

IS-IDENTITY-004 — Membership Management.

28. Authorization Boundary

Esta historia no decide por sí sola quién puede crear o modificar usuarios.

La decisión debe pasar por:

Authorization
      +
Security Policy Engine

Por ejemplo:

Actor
 ↓
Authentication
 ↓
Authorization
 ↓
Policy Evaluation
 ↓
User Management

Esto evita introducir reglas de permisos directamente en la entidad User.

29. API Requirements

La API será desarrollada posteriormente bajo:

/api/v1/identity

Endpoints conceptuales:

POST   /users
GET    /users/{user_id}
PATCH  /users/{user_id}
POST   /users/{user_id}/activate
POST   /users/{user_id}/suspend
POST   /users/{user_id}/disable

Estos contratos deberán cumplir:

OpenAPI 3.1
estándar de errores EVOXA
autorización
validación
tenant isolation
correlation ID
idempotencia donde corresponda
auditabilidad

La definición final de contratos pertenece a IS-IDENTITY-014 — Identity API Contracts.

30. Error Model

Errores de dominio relacionados:

UserNotFound
UserAlreadyExists
InvalidUserState
InvalidUserTransition
InvalidEmail
UserSuspended
UserDisabled
Unauthorized
Forbidden
TenantAccessDenied

Los nombres definitivos deben mantener consistencia con la taxonomía establecida en IS-IDENTITY-001.

31. Persistence Requirements

La persistencia transaccional será responsabilidad de PostgreSQL según ADR-IDENTITY-006.

Conceptualmente:

users

Debe contener información necesaria para:

identidad;
lifecycle;
timestamps;
estado;
metadata de seguridad necesaria.

No debe convertirse en un contenedor de información de todos los dominios EVOXA.

32. Concurrency

Las operaciones de lifecycle deben protegerse contra actualizaciones concurrentes.

Ejemplo:

Request A: ACTIVE → SUSPENDED

Request B: ACTIVE → DISABLED

El sistema debe evitar estados inconsistentes.

La estrategia concreta de locking/concurrency será definida durante la implementación de persistence.

33. Idempotency

Las operaciones administrativas que modifiquen estado deberán considerar idempotencia.

Ejemplo:

Suspend user
Suspend user
Suspend user

No debería producir tres cambios de estado independientes.

El comportamiento debe ser determinista y auditable.

34. Security Requirements

Debe cumplirse:

fail closed;
least privilege;
tenant isolation;
no secretos en logs;
no secretos en eventos;
no secretos en respuestas API;
validación de entrada;
auditoría;
protección contra enumeración de usuarios cuando corresponda;
control de acceso antes de revelar información.
35. Acceptance Criteria Consolidated
ID	Criterio
AC-001	Se puede crear un usuario válido
AC-002	No se permiten identidades duplicadas
AC-003	Se validan campos obligatorios
AC-004	Se puede consultar un usuario autorizado
AC-005	Usuario inexistente produce UserNotFound
AC-006	Se puede actualizar información permitida
AC-007	Campos protegidos requieren autorización
AC-008	Lifecycle se valida mediante reglas de dominio
AC-009	Se pueden activar/suspender/deshabilitar cuentas
AC-010	Los cambios generan eventos de dominio
AC-011	Las operaciones relevantes generan auditoría
AC-012	Se mantiene tenant isolation
AC-013	No se exponen secretos
AC-014	Las operaciones críticas son seguras ante concurrencia
AC-015	La implementación mantiene separación Domain/Application/Infrastructure
36. Technical Tasks
TASK-IDENTITY-002-01 — Definir User Entity

Crear la entidad User utilizando las reglas establecidas en IS-IDENTITY-001.

TASK-IDENTITY-002-02 — Implementar UserId

Crear el Value Object:

UserId

Debe proporcionar identidad estable y validada.

TASK-IDENTITY-002-03 — Implementar Email Value Object

Crear:

Email

con validación y normalización centralizadas.

TASK-IDENTITY-002-04 — Implementar User Lifecycle

Implementar:

PENDING
ACTIVE
SUSPENDED
DISABLED

y sus transiciones válidas.

TASK-IDENTITY-002-05 — Implementar Domain Rules

Crear las reglas para:

creación;
actualización;
activación;
suspensión;
deshabilitación;
validación de estado.
TASK-IDENTITY-002-06 — Implementar UserRepository Contract

Definir la interfaz abstracta:

UserRepository

sin dependencia de PostgreSQL.

TASK-IDENTITY-002-07 — Implementar CreateUser Use Case

Crear el caso de uso:

CreateUser

Debe:

validar input;
validar email;
comprobar unicidad;
crear entidad;
persistir;
generar UserCreated;
preparar auditoría.
TASK-IDENTITY-002-08 — Implementar GetUser Use Case

Crear:

GetUser

respetando:

autorización;
tenant context;
minimización de datos.
TASK-IDENTITY-002-09 — Implementar UpdateUser Use Case

Crear:

UpdateUser

con control de campos modificables.

TASK-IDENTITY-002-10 — Implementar Lifecycle Use Cases

Crear:

ActivateUser
SuspendUser
DisableUser

Cada uno debe validar las transiciones permitidas.

TASK-IDENTITY-002-11 — Implementar Domain Events

Crear contratos para:

UserCreated
UserUpdated
UserActivated
UserSuspended
UserDisabled
TASK-IDENTITY-002-12 — Integrar AuditService

Integrar los casos de uso con:

AuditService

sin duplicar la implementación de auditoría.

TASK-IDENTITY-002-13 — Preparar Persistence Mapping

Definir el mapping conceptual:

User Domain Entity
        ↓
Persistence Model
        ↓
users

La implementación física pertenece a Infrastructure.

TASK-IDENTITY-002-14 — Implementar Concurrency Protection

Definir y probar el mecanismo que evite conflictos en cambios simultáneos de estado.

TASK-IDENTITY-002-15 — Implementar Idempotency

Definir comportamiento idempotente para operaciones de modificación de estado.

TASK-IDENTITY-002-16 — Crear Unit Tests

Cubrir:

User;
Email;
UserId;
lifecycle;
transitions;
validation;
domain errors.
TASK-IDENTITY-002-17 — Crear Application Tests

Probar:

CreateUser
GetUser
UpdateUser
ActivateUser
SuspendUser
DisableUser
TASK-IDENTITY-002-18 — Crear Security Tests

Validar:

tenant isolation;
unauthorized access;
forbidden access;
information leakage;
disabled account;
suspended account.
TASK-IDENTITY-002-19 — Crear Event Tests

Verificar que cada cambio relevante produzca el evento correcto.

TASK-IDENTITY-002-20 — Traceability

Actualizar:

ESP-0001
   ↓
IS-IDENTITY-002
   ↓
TASK-IDENTITY-002-01 ... 020
   ↓
Source Code
   ↓
Tests
37. Definition of Done

La historia estará terminada cuando:

 User Entity implementada.
 UserId implementado.
 Email implementado.
 Lifecycle implementado.
 Transiciones validadas.
 Repository contract implementado.
 CreateUser implementado.
 GetUser implementado.
 UpdateUser implementado.
 ActivateUser implementado.
 SuspendUser implementado.
 DisableUser implementado.
 Domain Events implementados.
 Audit integrado.
 Tenant isolation validado.
 Concurrency control probado.
 Idempotency validada.
 Unit tests ≥85%.
 Critical Identity tests ≥95%.
 Security tests aprobados.
 Event tests aprobados.
 Traceability actualizada.
 Architecture Review aprobado.
 QA Review aprobado.
 Security Review aprobado cuando corresponda.
38. Traceability Matrix
Elemento	Referencia
Blueprint	Architecture Map
Engineering Standard	Identity / User Management
ESP	ESP-0001
ADR	ADR-IDENTITY-001
ADR	ADR-IDENTITY-005
ADR	ADR-IDENTITY-006
ADR	ADR-IDENTITY-007
ADR	ADR-IDENTITY-010
Predecessor IS	IS-IDENTITY-001
Current IS	IS-IDENTITY-002
Next related IS	IS-IDENTITY-003
Domain	Identity
Source	apps/api/app/domains/identity/
Tests	apps/api/tests/identity/

La trazabilidad sigue la cadena definida por EVOXA:

Blueprint
   ↓
ESP-0001
   ↓
IS-IDENTITY-002
   ↓
Technical Tasks
   ↓
Source Code
   ↓
Tests

La documentación de EVOXA establece explícitamente esta trazabilidad entre ESP, Implementation Stories, Technical Tasks y código, y exige que las IS mantengan referencias a ESP, dominio y estándares de ingeniería.

39. Dependencies
Depends on
IS-IDENTITY-001
ADR-IDENTITY-001
ADR-IDENTITY-005
ADR-IDENTITY-006
ADR-IDENTITY-007
ADR-IDENTITY-010
Required by
IS-IDENTITY-003 — Organization & Tenant Management
IS-IDENTITY-004 — Membership Management
IS-IDENTITY-005 — Authentication
IS-IDENTITY-006 — Authorization
IS-IDENTITY-008 — Session Management
IS-IDENTITY-009 — Refresh Token Lifecycle
IS-IDENTITY-010 — MFA
40. Architectural Notes

Esta IS debe permanecer deliberadamente agnóstica de infraestructura dentro del Domain Layer.

La arquitectura general de EVOXA establece:

Presentation
      ↓
API
      ↓
Application
      ↓
Domain
      ↓
Infrastructure

y el dominio Identity se ubica bajo:

apps/api/app/domains/identity/

Por tanto, no debemos comenzar escribiendo directamente modelos SQL, endpoints FastAPI o código de autenticación. Primero deben quedar cerrados los contratos de dominio, casos de uso, reglas, eventos y tareas técnicas.

41. Relación con las siguientes IS

El orden recomendado ahora queda:

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
        ↓
IS-IDENTITY-007
Role & Permission Management
        ↓
IS-IDENTITY-008
Session Management
        ↓
IS-IDENTITY-009
Refresh Token Lifecycle
        ↓
IS-IDENTITY-010
Multi-Factor Authentication
        ↓
...

Este orden es importante porque Authentication necesita User, Membership necesita User + Organization, y Authorization necesita el contexto formado por User + Organization + Membership + Roles/Permissions.

Estado de esta IS

IS-IDENTITY-002 queda definida como Draft y lista para revisión arquitectónica.

El siguiente paso lógico es IS-IDENTITY-003 — Organization & Tenant Management, manteniendo exactamente esta misma estructura y nivel de detalle.
