ESP-0001 / 02 — Identity Architecture

Document ID: ESP-0001/02
Module: Identity
Version: 1.0.0
Status: Draft
Parent: ESP-0001 — Identity
Previous Specification: ESP-0001/01 — Identity Domain Model
Architecture Parent: BP-0002 — Reference Architecture
Repository: evoxa-platform
Target Path: apps/api/app/domains/identity/

1. Purpose

Este documento define la arquitectura interna y técnica del módulo Identity de EVOXA.

Su objetivo es transformar el modelo de dominio definido en ESP-0001/01 en una arquitectura implementable, manteniendo los principios establecidos por el Blueprint:

Domain-Driven Design
API-First
Event-Driven Architecture
Security by Design
Privacy by Design
Zero Trust
Least Privilege
Observability
Modularidad
Evolución independiente

La arquitectura de referencia establece que EVOXA debe organizarse alrededor de dominios de negocio, con límites claros y responsabilidades independientes.

2. Architectural Position

Identity es un Shared Platform Domain.

Su posición conceptual es:

                    EVOXA PLATFORM
                          │
        ┌─────────────────┴─────────────────┐
        │                                   │
 Presentation                         Platform Services
        │                                   │
        ▼                                   ▼
      API Gateway ───────────────► Identity
                                      │
                    ┌─────────────────┼─────────────────┐
                    │                 │                 │
              Authentication    Authorization       Sessions
                    │                 │                 │
                    └─────────────────┼─────────────────┘
                                      │
                                      ▼
                               Identity Domain
                                      │
                     ┌────────────────┼────────────────┐
                     ▼                ▼                ▼
                 PostgreSQL         Redis         Event Platform

Identity no debe ser implementado como una librería aislada dentro de cada dominio.

Debe existir como capacidad central reutilizable.

3. Architectural Layers

Identity seguirá las capas arquitectónicas definidas por EVOXA:

┌───────────────────────────────┐
│             API               │
│ Routes / Controllers / Schemas│
├───────────────────────────────┤
│         APPLICATION           │
│ Use Cases / Commands / Queries│
├───────────────────────────────┤
│            DOMAIN             │
│ Entities / Rules / Policies   │
├───────────────────────────────┤
│        INFRASTRUCTURE         │
│ DB / Cache / Crypto / Events  │
└───────────────────────────────┘

La arquitectura de EVOXA define explícitamente las capas:

Presentation → API → Application → Domain → Infrastructure.

4. Dependency Rule

La dependencia debe dirigirse hacia el dominio:

API
 ↓
Application
 ↓
Domain
 ↑
Infrastructure

El dominio no debe conocer:

PostgreSQL
Redis
HTTP
FastAPI
proveedores externos
infraestructura cloud
SDKs concretos

Por ejemplo:

❌ Domain → PostgreSQL

✅ Domain
      ↓
 UserRepository interface
      ↑
 PostgreSQL implementation

Esto permite evolucionar la infraestructura sin modificar las reglas de negocio.

5. Identity Components

La arquitectura propuesta se divide en los siguientes componentes:

Identity
│
├── Authentication
│
├── Authorization
│
├── User Management
│
├── Organization Management
│
├── Membership Management
│
├── Role Management
│
├── Permission Management
│
├── Session Management
│
├── Token Management
│
├── MFA
│
├── Identity Policies
│
├── Audit
│
└── Identity Events

Las capacidades de Authentication, Authorization, Role Management, Permission Management, Tenant Management, Session Management, API Tokens y MFA forman parte del catálogo de capacidades de plataforma.

6. High-Level Architecture
                    ┌──────────────────┐
                    │   Web / Mobile   │
                    │ Coach / Admin    │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │   API Gateway    │
                    └────────┬─────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │      Identity API    │
                  └──────────┬───────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
       Authentication   Authorization   Sessions
              │              │              │
              └──────────────┼──────────────┘
                             ▼
                    ┌──────────────────┐
                    │ Identity Domain  │
                    └────────┬─────────┘
                             │
             ┌───────────────┼────────────────┐
             ▼               ▼                ▼
       PostgreSQL          Redis        Event Platform
7. Authentication Architecture

Authentication responde:

¿Quién eres?

El flujo conceptual será:

User
 │
 │ Credentials
 ▼
Identity API
 │
 ▼
Authentication Service
 │
 ├── Validate credentials
 │
 ├── Validate account status
 │
 ├── Validate MFA if required
 │
 ├── Create authentication context
 │
 ├── Create session
 │
 └── Issue tokens
 │
 ▼
Authenticated User

La API Architecture establece que las APIs protegidas requieren autenticación y contempla OAuth2, JWT Bearer, refresh tokens, API keys y service accounts.

8. Authentication Providers

La arquitectura debe permitir que EVOXA utilice diferentes mecanismos de identidad.

Conceptualmente:

                 Authentication
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
  EVOXA Identity    OIDC           SAML
        │              │              │
        │        ┌─────┴─────┐        │
        │        ▼           ▼        │
        │     Google       Entra ID   │
        │
        └──────────────┬──────────────┘
                       ▼
                 EVOXA Identity
                       │
                       ▼
                Internal User

La arquitectura de seguridad contempla EVOXA Identity y proveedores externos como Entra ID, Google, Okta y Auth0, además de OIDC y SAML2.

Importante: el proveedor primario definitivo todavía no está fijado por el Blueprint. Por ello, esta especificación debe mantener una abstracción de IdentityProvider.

9. Authentication Abstraction

Propuesta:

IdentityProvider
       │
       ├── LocalIdentityProvider
       │
       ├── OIDCIdentityProvider
       │
       └── SAMLIdentityProvider

El dominio no debería conocer el proveedor concreto.

Por ejemplo:

AuthenticationService
        ↓
IdentityProvider
        ↓
Google / Entra / EVOXA / etc.

La elección del proveedor concreto deberá formalizarse mediante un ADR.

10. Authorization Architecture

Authorization responde:

¿Qué puedes hacer?

Flujo:

Request
   │
   ▼
Authentication
   │
   ▼
Identity Context
   │
   ├── User
   ├── Organization
   ├── Roles
   └── Claims
   │
   ▼
Authorization
   │
   ├── RBAC
   │
   ├── ABAC
   │
   ├── Ownership
   │
   └── Tenant Policy
   │
   ▼
ALLOW / DENY

La arquitectura de seguridad define RBAC + ABAC, tenant isolation, organization membership y ownership como elementos del modelo de autorización.

11. RBAC

RBAC proporciona autorización basada en roles.

User
 │
 ▼
Membership
 │
 ▼
Role
 │
 ▼
Permissions

Ejemplo:

Coach
 ├── clients.read
 ├── clients.update
 ├── training.read
 ├── training.create
 └── training.update

El sistema debe evitar que la lógica de permisos quede codificada directamente dentro de los controllers.

En lugar de:

if user.role == "coach":

debe utilizarse:

AuthorizationService
        ↓
Permission
        ↓
Policy
12. ABAC

RBAC no es suficiente para todos los escenarios de EVOXA.

ABAC permite considerar atributos contextuales.

Ejemplo:

User
  +
Organization
  +
Resource
  +
Ownership
  +
Resource State

Ejemplo conceptual:

Coach
AND
belongs_to_organization
AND
owns_client
AND
client_active

Resultado:

ALLOW

o:

DENY
13. Tenant Context

Toda request autenticada que acceda a recursos organizacionales debe establecer un contexto de tenant.

HTTP Request
     │
     ▼
Authentication
     │
     ▼
User Identity
     │
     ▼
Tenant Resolver
     │
     ▼
Organization Context
     │
     ▼
Authorization

Ejemplo:

tenant_id = organization-123

Los servicios posteriores deben recibir ese contexto.

14. Cross-Tenant Protection

La arquitectura debe impedir:

Tenant A
   │
   │ request
   ▼
Resource Tenant B
   │
   ▼
DENY

Nunca debemos confiar únicamente en un organization_id enviado por el cliente.

El contexto debe derivarse de la identidad autenticada y sus memberships/autorizaciones.

Esta exigencia deriva directamente de las reglas de tenant isolation y cross-tenant restrictions establecidas en Security Architecture.

15. Session Architecture

Las sesiones permiten controlar los contextos autenticados.

User
 │
 ├── Session A → Web
 ├── Session B → Mobile
 └── Session C → Coach Portal

Una sesión debe poder:

crearse;
validarse;
renovarse;
revocarse;
expirar;
auditarse.
16. Token Architecture

La API Architecture contempla:

Access Token
+
Refresh Token

Conceptualmente:

Login
 │
 ▼
Authentication
 │
 ▼
Session
 │
 ├──────────────┐
 ▼              ▼
Access Token   Refresh Token

El access token se utiliza para acceder a APIs.

El refresh token permite obtener un nuevo access token sin repetir todo el proceso de autenticación.

La estrategia exacta de almacenamiento, rotación y revocación de refresh tokens deberá quedar definida en la especificación de Authentication/API y su correspondiente ADR.

17. API Gateway Integration

Identity se integra con el API Gateway:

Client
  │
  ▼
API Gateway
  │
  ├── TLS
  ├── Rate Limiting
  ├── Request Validation
  ├── Token Validation
  └── Correlation ID
  │
  ▼
Domain API

Sin embargo, la validación completa de autorización no debe delegarse exclusivamente al Gateway.

El dominio que posee el recurso debe poder aplicar sus propias reglas.

Ejemplo:

Gateway
  → "JWT válido"

Training Domain
  → "¿Este usuario puede modificar ESTE programa?"
18. Identity API

La API debe seguir el estándar API-First de EVOXA.

Las APIs deben ser:

versionadas;
stateless;
seguras;
orientadas a recursos;
observables;
documentadas;
compatibles hacia atrás cuando sea posible.

Base:

/api/v1

Ejemplos conceptuales:

POST   /api/v1/auth/login
POST   /api/v1/auth/logout
POST   /api/v1/auth/refresh

GET    /api/v1/users
GET    /api/v1/users/{id}

GET    /api/v1/organizations
GET    /api/v1/organizations/{id}

GET    /api/v1/roles
GET    /api/v1/permissions

Los contratos definitivos pertenecen a ESP-0001/13 — Identity API Contracts, no a este documento.

19. Application Layer

La Application Layer coordina los casos de uso.

Ejemplo:

LoginUser
CreateUser
ActivateUser
DeactivateUser

CreateOrganization
AddMembership
RemoveMembership

AssignRole
RevokeRole

CreateSession
RevokeSession
RefreshSession

La Application Layer:

recibe comandos;
valida contexto;
coordina servicios de dominio;
utiliza repositories;
publica eventos;
coordina transacciones.

No debe contener reglas fundamentales que pertenezcan al Domain Layer.

20. Domain Layer

Contendrá:

Entities
Value Objects
Aggregates
Domain Services
Policies
Domain Events
Repository Interfaces

Ejemplo:

domain/
├── entities/
├── value_objects/
├── aggregates/
├── services/
├── policies/
├── events/
└── repositories/

Esta capa representa las reglas de Identity independientemente de la infraestructura.

21. Infrastructure Layer

Infrastructure implementará:

PostgreSQL repositories
Redis session/cache repositories
Cryptographic services
Token services
External Identity Providers
Event Publisher
Audit persistence

Conceptualmente:

                 Domain
                   │
          ┌────────┴─────────┐
          ▼                  ▼
     Repository          EventPublisher
      Interface             Interface
          ▲                  ▲
          │                  │
    PostgreSQL          Event Platform
22. PostgreSQL

PostgreSQL será la persistencia principal propuesta para los datos transaccionales de Identity.

Modelo conceptual:

users
organizations
memberships
roles
permissions
role_permissions
sessions

Importante: PostgreSQL está establecido en la arquitectura de plataforma como datastore principal, pero este documento no debe considerarse todavía la definición final del esquema físico.

Eso pertenecerá a:

ESP-0001 / Persistence Specification.

23. Redis

Redis puede utilizarse para capacidades que requieran acceso rápido y/o estado temporal, por ejemplo:

Session state
Token metadata
Rate limiting
Temporary authentication state
MFA challenges
Caching

Esto es una decisión de diseño propuesta, no una obligación textual del Blueprint para cada uno de estos casos.

El Blueprint sí contempla Redis dentro de la infraestructura de plataforma.

24. Event Platform

Identity publica eventos hacia la plataforma central de eventos:

Identity
   │
   ▼
Event Publisher
   │
   ▼
Event Platform
   │
   ├── Analytics
   ├── Audit
   ├── Notifications
   ├── AI
   └── Other Domains

Esto evita acoplamiento directo:

Identity → Training

y favorece:

Identity → Event Platform → Training

La arquitectura de EVOXA establece Event-Driven Architecture y eventos de dominio inmutables.

25. Example Event Flow

Cuando se crea un usuario:

POST /users
      │
      ▼
UserApplicationService
      │
      ▼
User Aggregate
      │
      ▼
UserCreated
      │
      ▼
Event Platform
      │
      ├── Analytics
      ├── Audit
      └── Notifications

Identity no necesita conocer directamente a cada consumidor.

26. Audit Architecture

La arquitectura debe registrar operaciones críticas:

Authentication
Authorization failures
Role changes
Permission changes
Membership changes
MFA changes
Session revocation
Account status changes

Flujo:

Identity Operation
       │
       ├──────────────► Business Result
       │
       └──────────────► Audit Event
                              │
                              ▼
                         Audit Store

La auditoría es parte explícita de los requisitos de seguridad y observabilidad de EVOXA.

27. MFA Architecture

MFA se considera una capability de Identity.

Flujo:

Login
 │
 ▼
Credentials Valid
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
   MFA Verify
       │
       ▼
     Session

Los mecanismos específicos de MFA todavía deben definirse en una especificación de Authentication/MFA.

28. Security Boundary

Identity representa una frontera de seguridad crítica.

             UNTRUSTED
                 │
                 ▼
          API Gateway
                 │
                 ▼
        Authentication
                 │
                 ▼
          Identity Context
                 │
                 ▼
         Authorization
                 │
                 ▼
             TRUSTED

La confianza nunca debe derivarse solamente de datos enviados por el cliente.

Esto sigue el modelo Zero Trust definido para EVOXA.

29. Cryptography

La implementación deberá respetar los mecanismos definidos por Security Architecture:

Password hashing
→ Argon2id

Data at rest
→ AES-256

Transport
→ TLS 1.3

Signing
→ Ed25519 / RSA-4096

La selección concreta para cada uso debe documentarse posteriormente.

30. Observability Architecture

Identity deberá producir:

Logs
Metrics
Traces
Audit Events
Security Events
Business Events

Cada request deberá poder correlacionarse:

trace_id
span_id
correlation_id

Ejemplo:

Mobile
 │
 │ correlation_id
 ▼
API Gateway
 │
 ▼
Identity
 │
 ▼
PostgreSQL
 │
 ▼
Event Platform

La arquitectura de observabilidad establece distributed tracing y correlation IDs a través de los diferentes componentes.

31. Error Handling

Identity utilizará el contrato de error común de EVOXA:

{
  "error": {
    "code": "IDENTITY_ERROR",
    "message": "Human readable message",
    "correlation_id": "..."
  }
}

La API Architecture establece este formato estándar.

Nunca deberían exponerse detalles internos como:

SQL errors
stack traces
password validation internals
internal provider details

en respuestas públicas.

32. Rate Limiting

Identity debe estar protegido especialmente contra abuso.

Áreas críticas:

/login
/refresh
/password-reset
/mfa

El API Architecture contempla rate limiting como mecanismo de seguridad.

Los límites concretos deben definirse posteriormente.

33. Failure Handling

Identity debe diseñarse para fallos parciales.

Ejemplo:

Identity
 │
 ├── PostgreSQL unavailable
 │
 ├── Redis unavailable
 │
 ├── Event Platform unavailable
 │
 └── External IdP unavailable

Cada dependencia deberá tener una estrategia definida.

Particularmente importante:

Authentication
Authorization

no deben degradarse de manera insegura.

Fail-open para autorización no será permitido salvo que una decisión arquitectónica futura y explícita lo justifique.

34. Performance

Identity es una dependencia transversal.

Por tanto, su latencia afecta a toda la plataforma.

La arquitectura de Performance establece como objetivo:

Authentication
P95 < 500 ms

General API
P95 < 300 ms

La implementación deberá evitar consultas innecesarias y permitir caching donde sea seguro.

35. Scalability

Identity debe soportar escalamiento horizontal:

             Load Balancer
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
    Identity 1 Identity 2 Identity 3
        │          │          │
        └──────────┼──────────┘
                   ▼
              PostgreSQL
                   +
                 Redis

Los servicios deben permanecer stateless cuando sea posible.

Esto coincide con los principios de escalabilidad horizontal y servicios stateless definidos en la arquitectura.

36. Proposed Project Structure

La arquitectura técnica resultante sería:

apps/api/app/domains/identity/

├── api/
│   ├── controllers/
│   ├── routes/
│   ├── schemas/
│   └── dependencies/
│
├── application/
│   ├── commands/
│   ├── queries/
│   ├── services/
│   └── handlers/
│
├── domain/
│   ├── entities/
│   ├── value_objects/
│   ├── aggregates/
│   ├── services/
│   ├── policies/
│   ├── events/
│   └── repositories/
│
└── infrastructure/
    ├── persistence/
    │   ├── models/
    │   └── repositories/
    │
    ├── authentication/
    │   ├── providers/
    │   └── tokens/
    │
    ├── sessions/
    ├── security/
    ├── messaging/
    └── audit/

Esta estructura es Derived Design y no debe confundirse con una estructura literalmente definida por el Blueprint.

37. Interaction With Other Domains

Identity debe ser consumido por los demás dominios mediante contratos.

Ejemplo:

                 Identity
                    │
        ┌───────────┼────────────┐
        ▼           ▼            ▼
     Training    Nutrition    Recovery
        │           │            │
        └───────────┼────────────┘
                    ▼
             Authorization

Los dominios no deberían acceder directamente a tablas internas de Identity.

❌ Incorrecto:

Training → SELECT * FROM identity.users

✅ Correcto:

Training
   ↓
Identity Context / Authorization
   ↓
ALLOW / DENY

o mediante los contratos/eventos establecidos.

38. Architecture Principles

Identity deberá cumplir:

AP-001 — Domain Isolation

Identity no implementa lógica de otros dominios.

AP-002 — API First

Todo acceso externo debe realizarse mediante contratos API.

AP-003 — Security by Default

Toda operación comienza considerándose no autorizada.

AP-004 — Least Privilege

Los usuarios reciben solamente los permisos necesarios.

AP-005 — Tenant Isolation

No existe acceso implícito entre organizaciones.

AP-006 — Auditability

Las operaciones críticas deben ser trazables.

AP-007 — Event Driven

Los cambios significativos generan eventos de dominio.

AP-008 — Infrastructure Independence

El dominio no depende directamente de infraestructura concreta.

39. Architecture Decisions Still Open

Antes de pasar a código debemos cerrar:

ID	Decisión
ADR-IDENTITY-001	¿Membership será aggregate/entity formal?
ADR-IDENTITY-002	¿Usuarios multi-organización?
ADR-IDENTITY-003	¿Roles globales, organizacionales o ambos?
ADR-IDENTITY-004	Modelo definitivo de Permission
ADR-IDENTITY-005	JWT como mecanismo principal
ADR-IDENTITY-006	Estrategia de Refresh Token
ADR-IDENTITY-007	Proveedor primario de Identity
ADR-IDENTITY-008	Estrategia MFA
ADR-IDENTITY-009	PostgreSQL schema definitivo
ADR-IDENTITY-010	Estrategia Redis
ADR-IDENTITY-011	Estrategia de autorización RBAC + ABAC

No deberíamos convertir ninguna de estas decisiones en una dependencia irreversible del código sin documentarla.

40. Implementation Readiness

Con este documento tenemos:

ESP-0001/01
Identity Domain Model
        │
        ▼
ESP-0001/02
Identity Architecture
        │
        ▼
       API
        │
        ▼
    Persistence
        │
        ▼
      Events
        │
        ▼
      Security
Estado
Área	Estado
Architectural position	✅
Layers	✅
Authentication architecture	🟡
Authorization architecture	🟡
RBAC	✅
ABAC	✅
Tenant isolation	✅
Sessions	🟡
Tokens	🟡
MFA	🟡
PostgreSQL	🟡
Redis	🟡
Events	🟡
Audit	✅
Observability	✅
Scalability	✅
Security	✅
API contracts	⏳ Próximo
ADRs	⏳ Pendientes
Próximo documento

El siguiente paso debería ser:

ESP-0001 / 03 — Identity Authentication & Authorization

Ahí ya podemos entrar profundamente en cómo funcionará el login real de EVOXA, incluyendo:

Login
 ↓
Credentials
 ↓
Password Hashing
 ↓
Account Validation
 ↓
MFA
 ↓
Session
 ↓
Access Token
 ↓
Refresh Token
 ↓
Token Rotation
 ↓
Revocation
 ↓
Authorization
 ↓
RBAC + ABAC
 ↓
Tenant Context
