ESP-0001 — Identity

Document ID: ESP-0001
Module: Identity
Version: 1.0.0
Status: Draft
Type: Engineering Specification
Parent: BP-0002 — Reference Architecture
Domain: Identity
Planned path: apps/api/app/domains/identity/

1. Propósito

El módulo Identity proporciona las capacidades necesarias para identificar, autenticar y autorizar a las personas y sistemas que interactúan con EVOXA.

Identity será utilizado transversalmente por el resto de la plataforma.

                    EVOXA
                      │
                      ▼
                  Identity
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
     Users      Organizations       Access
       │              │              │
       └──────────────┼──────────────┘
                      ▼
              Roles & Permissions
                      │
                      ▼
                Authentication
                      │
                      ▼
                Authorization
                      │
                      ▼
              Platform Domains

La arquitectura considera Identity como uno de los dominios iniciales de EVOXA y establece que cada dominio debe mantener su propia lógica y datos.

2. Responsabilidades

Identity será responsable de:

Identidad
Usuarios
Perfiles de identidad
Organizaciones
Membresías
Estado de identidad
Autenticación
Login
Logout
Sesiones
Tokens
Refresh tokens
MFA
Autorización
Roles
Permisos
RBAC
Políticas de acceso
Verificación de acceso
Seguridad
Protección de credenciales
Gestión de sesiones
Control de acceso
Protección contra accesos no autorizados
Auditoría relacionada con identidad
Multi-tenancy
Identificación del tenant
Asociación usuario → organización
Aislamiento entre organizaciones
Validación de pertenencia

Estas capacidades coinciden con el catálogo de capacidades de Identity definido en el Blueprint: Authentication, Authorization, Role Management, Permission Management, Tenant Management, Session Management, API Tokens y MFA.

3. Lo que Identity NO debe hacer

Esto es muy importante.

Identity no debe contener lógica de negocio de otros dominios.

Por ejemplo:

❌ Identity → crear entrenamiento
❌ Identity → calcular dieta
❌ Identity → generar recomendación
❌ Identity → modificar Digital Twin

En cambio:

✅ Identity → saber quién es el usuario
✅ Identity → saber a qué organización pertenece
✅ Identity → saber qué permisos posee
✅ Identity → autenticarlo
✅ Identity → autorizarlo

Esto respeta el principio de que los dominios poseen su lógica de negocio y colaboran mediante contratos explícitos.

4. Arquitectura interna

Identity seguirá las capas de EVOXA:

apps/api/app/domains/identity/

├── api/
│
├── application/
│
├── domain/
│
└── infrastructure/
API

Responsable de exponer los contratos HTTP.

POST   /api/v1/auth/login
POST   /api/v1/auth/logout
POST   /api/v1/auth/refresh

GET    /api/v1/users
GET    /api/v1/users/{id}
POST   /api/v1/users
PATCH  /api/v1/users/{id}

GET    /api/v1/organizations
GET    /api/v1/roles
GET    /api/v1/permissions

Los recursos deben utilizar nombres de negocio y URLs orientadas a recursos, siguiendo la estrategia API-first definida por EVOXA.

5. Domain

La capa Domain será el corazón de Identity.

Propuesta inicial:

domain/

├── entities/
│   ├── User
│   ├── Organization
│   ├── Membership
│   ├── Role
│   ├── Permission
│   └── Session
│
├── value-objects/
│   ├── UserId
│   ├── OrganizationId
│   ├── Email
│   └── RoleId
│
├── services/
│   ├── AuthenticationService
│   └── AuthorizationService
│
├── policies/
│
├── events/
│
└── repositories/

Esto es una propuesta de implementación derivada de la arquitectura, no una estructura que el Blueprint haya especificado literalmente.

La razón es que la arquitectura exige que el Domain contenga entidades, agregados, value objects, servicios de dominio, eventos y reglas de negocio, permaneciendo independiente de infraestructura.

6. Entidades principales
User

Representa la identidad de una persona dentro de EVOXA.

Conceptualmente:

User
├── id
├── email
├── status
├── profile
├── authentication
└── memberships

No debemos convertir inmediatamente esto en una tabla SQL.

Primero debemos definir correctamente la entidad y sus invariantes.

7. Organization

Representa el contexto organizacional dentro del cual opera un usuario.

Organization
├── id
├── name
├── status
└── configuration

Una organización puede tener múltiples usuarios.

Organization
      │
      ├── User
      ├── User
      ├── User
      └── User
8. Membership

Aquí hay una decisión arquitectónica importante.

No deberíamos hacer simplemente:

User.roleId

porque el mismo usuario podría eventualmente participar en distintas organizaciones con distintos permisos.

La relación debería conceptualizarse como:

User
 │
 ▼
Membership
 │
 ├── Organization
 ├── Roles
 └── Status

Por ejemplo:

Sebastian
   │
   ├── Organization A
   │      └── Coach
   │
   └── Organization B
          └── Administrator

Esto además prepara Identity para el modelo multi-tenant previsto por la arquitectura.

9. Roles

Identity proporciona administración de roles.

Ejemplo conceptual:

Role
├── id
├── name
├── description
└── permissions

La documentación establece explícitamente Role Management y Permission Management como capacidades de Identity.

10. Permissions

Los permisos deben ser independientes de los roles.

Permission
├── id
├── resource
├── action
└── description

Ejemplos:

users.read
users.create
users.update
users.delete

training.read
training.create
training.update

nutrition.read
nutrition.create

Entonces:

Role
   │
   └── RolePermission
            │
            ▼
       Permission

Esto permitirá posteriormente combinar RBAC con políticas más avanzadas.

11. Authentication

El flujo conceptual será:

Client
  │
  ▼
Login
  │
  ▼
Identity API
  │
  ▼
Credential Validation
  │
  ▼
Authentication
  │
  ├──────────────┐
  ▼              ▼
Access Token   Refresh Token
  │              │
  └──────┬───────┘
         ▼
      Client

La API Architecture contempla autenticación para las APIs públicas y versionamiento de endpoints.

12. Authorization

Cada solicitud protegida deberá pasar por autorización.

Conceptualmente:

Request
   │
   ▼
Authentication
   │
   ▼
User Identity
   │
   ▼
Tenant Context
   │
   ▼
Role
   │
   ▼
Permission
   │
   ▼
Policy
   │
   ▼
ALLOW / DENY

No basta con comprobar:

isAuthenticated == true

También debemos comprobar el contexto organizacional y los permisos correspondientes.

13. Events

Identity deberá generar eventos de dominio.

Por ejemplo:

UserCreated
UserUpdated
UserActivated
UserDeactivated

OrganizationCreated
MembershipCreated
MembershipRemoved

RoleCreated
RoleUpdated
PermissionAssigned
PermissionRevoked

UserAuthenticated
UserLoggedOut
SessionCreated
SessionRevoked

La arquitectura de EVOXA establece que las acciones empresariales significativas generan eventos de dominio inmutables y que los consumidores deben ser idempotentes.

Esto permitirá posteriormente que otros dominios reaccionen sin acoplarse directamente a Identity.

14. Ejemplo

Cuando se crea un usuario:

POST /api/v1/users
        │
        ▼
     API Layer
        │
        ▼
 CreateUserUseCase
        │
        ▼
    User Domain
        │
        ├── Validate
        ├── Create User
        └── Emit UserCreated
                     │
                     ▼
                 Event Bus
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
      Analytics   Notifications  Audit

La API no debería llamar directamente a Analytics o Notifications.

Ese desacoplamiento es justamente una de las razones para utilizar la arquitectura orientada a eventos.

15. Seguridad

Identity deberá diseñarse bajo:

Least Privilege
Zero Trust
Defense in Depth
Secure by Default
Privacy by Design
Auditability

La arquitectura de seguridad de EVOXA exige autenticación, autorización, validación y auditoría en cada solicitud, además de aislamiento entre tenants.

También se contemplan mecanismos como OAuth/OIDC, JWT, API keys, service accounts y MFA.

16. API Contract

Antes de escribir los controllers vamos a definir el contrato.

Por ejemplo:

POST /api/v1/auth/login

Request:

{
  "email": "user@example.com",
  "password": "********"
}

Response conceptual:

{
  "access_token": "...",
  "refresh_token": "...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "user": {
    "id": "...",
    "email": "user@example.com"
  }
}

Los errores deberán utilizar el modelo estandarizado:

{
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Invalid credentials.",
    "correlation_id": "..."
  }
}

Este patrón de error con code, message y correlation_id está definido en la arquitectura API.

17. Trazabilidad

Para que realmente estemos siguiendo EVOXA, cada implementación deberá poder responder:

¿Por qué existe este código?

La respuesta deberá poder recorrer:

BP-0002
   ↓
ESP-0001
   ↓
IS-0001
   ↓
Technical Task
   ↓
API Contract
   ↓
Domain
   ↓
Code
   ↓
Test

Esta trazabilidad está explícitamente definida por ARCHITECTURE-MAP.md.

18. Qué NO vamos a decidir todavía

Hay algunas cosas que no debemos inventar si el material proporcionado no las define todavía:

Framework concreto del backend
ORM concreto
estructura exacta de tablas
proveedor definitivo de Identity
proveedor definitivo de email
proveedor definitivo de MFA
proveedor definitivo de infraestructura
configuración definitiva de Kubernetes
proveedor definitivo de Event Bus

El Blueprint deliberadamente mantiene la arquitectura independiente de tecnologías.

Por eso esas decisiones deberán documentarse como Engineering Standards / ADR antes de convertirlas en decisiones permanentes.

19. Estado de ESP-0001

Yo lo dejaría en este estado:

Área	Estado
Propósito	✅
Responsabilidades	✅
Límites del dominio	✅
Capas	✅
Entidades conceptuales	🟡
Value Objects	🟡
Reglas de negocio	🟡
API Contracts	🟡
Events	🟡
Security	🟡
Persistence	⏳
Tests	⏳
Implementation Stories	⏳
Código	⏳

No deberíamos empezar a programar todavía.

El siguiente paso correcto es completar ESP-0001 hasta convertirlo en una especificación implementable.

Siguiente paso

Yo continuaría con:

ESP-0001 / 01 — Identity Domain Model

Ahí vamos a definir uno por uno:

User
Organization
Membership
Role
Permission
RolePermission
Session
Authentication
Identity Provider

y para cada uno:

propósito → atributos → relaciones → invariantes → lifecycle → reglas → eventos → repository → API → seguridad.

Después pasamos a ESP-0001 / 02 — Identity API Contracts, y recién entonces empezamos a generar el código.
