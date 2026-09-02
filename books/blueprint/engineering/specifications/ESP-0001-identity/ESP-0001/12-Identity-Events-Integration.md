ESP-0001 / 12 — Identity Events & Integration

Estado: Draft
Dominio: Identity
Código: ESP-0001
Sección: 12 — Events & Integration
Parent Specification: ESP-0001 — Identity Engineering Specification

1. Propósito

Este documento define cómo el dominio Identity de EVOXA publica, consume y gestiona eventos e integraciones con el resto de la plataforma.

Identity no debe convertirse en un dominio aislado. Sus cambios relevantes deben poder propagarse de forma controlada hacia:

Human Digital Twin
Training
Nutrition
Recovery
Recommendation
AI Orchestrator
Notifications
Analytics
Administration
Security
Observability
Integrations

La arquitectura de EVOXA establece explícitamente un enfoque Event-Driven, donde las acciones de negocio significativas generan eventos y los consumidores reaccionan a ellos.

2. Principios

Identity Events debe cumplir:

Events are facts, not commands.
Inmutabilidad.
Versionamiento.
Idempotencia.
Tenant awareness.
Correlation.
Causation.
At-least-once delivery.
Retry.
Dead Letter Queue.
Observability.
Security.
No sensitive secrets in events.
3. Event-Driven Identity

El modelo general:

                 Identity
                    │
             Business Action
                    │
                    ▼
              Domain Event
                    │
                    ▼
             Event Platform
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
 Notifications   Analytics    Other Domains

Por ejemplo:

User Created
     │
     ▼
UserCreated
     │
     ├── Analytics
     ├── Notifications
     ├── Administration
     └── Other Consumers
4. Domain Events vs Integration Events

No todos los eventos internos deben convertirse automáticamente en eventos externos.

Domain Event

Representa un hecho ocurrido dentro de Identity.

UserActivated
Integration Event

Es la representación publicada para consumidores externos al dominio.

identity.user.activated.v1

La separación permite que Identity evolucione internamente sin romper consumidores.

5. Event Envelope

Todos los eventos publicados deben seguir un envelope estándar.

Conceptualmente:

{
  "event_id": "...",
  "event_type": "UserCreated",
  "event_version": 1,
  "timestamp": "...",
  "tenant_id": "...",
  "actor": {
    "type": "USER",
    "id": "..."
  },
  "resource": {
    "type": "User",
    "id": "..."
  },
  "payload": {},
  "metadata": {
    "correlation_id": "...",
    "causation_id": "...",
    "trace_id": "..."
  }
}

Este modelo sigue la estructura definida por la arquitectura de eventos de EVOXA.

6. Event ID

Cada evento debe tener un identificador único:

event_id

Debe permitir:

deduplicación;
trazabilidad;
auditoría;
debugging.
7. Event Type

Los tipos deben utilizar nombres semánticos.

Ejemplos:

UserCreated
UserUpdated
UserActivated
UserSuspended
UserDisabled

No:

UserTableUpdated
DatabaseRowChanged

El evento representa el hecho de negocio, no el mecanismo de persistencia.

8. Event Version

Todo evento debe estar versionado.

UserCreated
version = 1

Posteriormente:

UserCreated
version = 2

La evolución debe mantener compatibilidad cuando sea posible.

9. Timestamp

Debe registrarse el momento en que ocurrió el hecho:

timestamp

Se recomienda utilizar timestamps consistentes en UTC.

10. Tenant ID

Los eventos organization-scoped deben incluir:

tenant_id

Esto permite que los consumidores respeten el aislamiento multi-tenant.

Un consumidor nunca debería procesar un evento de una organización como si perteneciera a otra.

11. Actor

Debe identificar el origen de la acción:

USER
ADMIN
SERVICE
SYSTEM
AI

Ejemplo:

{
  "type": "USER",
  "id": "user-123"
}

Si la operación fue realizada automáticamente:

{
  "type": "SYSTEM",
  "id": "identity-service"
}
12. Resource

El evento debe identificar el recurso afectado.

Ejemplo:

{
  "type": "User",
  "id": "user-123"
}

Esto facilita correlacionar:

Event
 ↓
Resource
 ↓
Audit
13. Correlation ID

Los eventos deben mantener:

correlation_id

para relacionar múltiples operaciones de una misma interacción.

Ejemplo:

POST /users
    │
    ├── correlation_id
    │
    ├── UserCreated
    │
    ├── MembershipCreated
    │
    └── NotificationRequested
14. Causation ID

Cuando un evento produce otro evento:

causation_id

permite conservar la relación causal.

Ejemplo:

AdminSuspendedUser
        │
        ▼
UserSuspended
        │
        ▼
SessionsRevoked

SessionsRevoked puede identificar que fue provocado por UserSuspended.

15. Identity Event Catalog

El catálogo inicial será:

UserCreated
UserUpdated
UserActivated
UserSuspended
UserDisabled

OrganizationCreated
OrganizationUpdated
OrganizationSuspended
OrganizationReactivated

MembershipCreated
MembershipActivated
MembershipSuspended
MembershipRemoved
MembershipRoleChanged

RoleCreated
RoleUpdated
RoleDeleted
RoleAssigned
RoleRemoved

PermissionCreated
PermissionUpdated
PermissionAssigned
PermissionRevoked

UserAuthenticated
UserAuthenticationFailed
UserLoggedOut

SessionCreated
SessionRevoked
SessionExpired

RefreshTokenRotated
RefreshTokenRevoked
TokenReuseDetected

MFAFactorCreated
MFAFactorActivated
MFAFactorRevoked
MFAChallengeVerified
MFAChallengeFailed
MFARecoveryCompleted

SecurityPolicyCreated
SecurityPolicyUpdated
SecurityPolicyActivated
SecurityPolicyDeactivated
SecurityPolicyViolation

Este catálogo deberá refinarse durante la definición de Implementation Stories.

16. User Events
UserCreated

Se publica cuando un nuevo User queda creado.

Consumidores potenciales:

Analytics
Notifications
Administration
Audit
UserUpdated

Se publica cuando se modifica información relevante del usuario.

No debería incluir información sensible innecesaria.

UserActivated

Indica:

User
PENDING → ACTIVE
UserSuspended

Indica que el acceso del usuario ha sido suspendido.

Puede desencadenar:

Session Revocation
Token Revocation
Security Monitoring

según las políticas definidas.

UserDisabled

Representa la desactivación permanente/lógica de la cuenta.

17. Organization Events
OrganizationCreated
OrganizationUpdated
OrganizationSuspended
OrganizationReactivated

Estos eventos permiten que otros componentes conozcan cambios importantes del tenant.

18. Membership Events
MembershipCreated
MembershipActivated
MembershipSuspended
MembershipRemoved
MembershipRoleChanged

Ejemplo:

User
  │
  ▼
MembershipCreated
  │
  ▼
Organization
  │
  ▼
Role Assignment

Estos eventos son especialmente importantes para autorización y aplicaciones multi-tenant.

19. Role Events
RoleCreated
RoleUpdated
RoleDeleted
RoleAssigned
RoleRemoved

Un RoleAssigned puede incluir conceptualmente:

{
  "membership_id": "...",
  "role_id": "..."
}

No debe incluir automáticamente todos los permisos contenidos en el rol.

El consumidor puede resolverlos mediante APIs apropiadas.

20. Permission Events
PermissionCreated
PermissionUpdated
PermissionAssigned
PermissionRevoked

Cambios de permisos son eventos de seguridad relevantes.

21. Authentication Events
UserAuthenticated
UserAuthenticationFailed
UserLoggedOut

Estos eventos deben utilizarse con cuidado debido al volumen potencial.

No todos los intentos de autenticación necesariamente deben distribuirse a todos los consumidores.

Puede existir:

Security Event Stream

separado del stream de negocio.

22. Session Events
SessionCreated
SessionRevoked
SessionExpired

Consumidores potenciales:

Security;
Analytics;
Audit;
Notifications.

Nunca deben incluir tokens completos.

23. Token Events
RefreshTokenRotated
RefreshTokenRevoked
TokenReuseDetected

TokenReuseDetected debe considerarse principalmente un Security Event.

Puede alimentar:

Security Operations
Risk Engine
Observability
24. MFA Events

Basado en ESP-0001/07:

MFAFactorCreated
MFAFactorActivated
MFAFactorRevoked
MFAChallengeVerified
MFAChallengeFailed
MFARecoveryCompleted

Nunca:

TOTP Secret
Recovery Code
OTP

dentro del evento.

25. Security Policy Events
SecurityPolicyCreated
SecurityPolicyUpdated
SecurityPolicyActivated
SecurityPolicyDeactivated
SecurityPolicyViolation

Un evento puede incluir:

policy_id
policy_version
scope
decision

pero no debe incluir secretos ni información sensible innecesaria.

26. Event Payload Design

Los payloads deben ser:

mínimos;
semánticos;
estables;
versionables;
libres de secretos.

Ejemplo:

{
  "user_id": "...",
  "organization_id": "...",
  "status": "ACTIVE"
}

No:

{
  "password": "...",
  "password_hash": "...",
  "refresh_token": "..."
}
27. Event Ordering

La arquitectura de EVOXA establece que el orden debe garantizarse únicamente donde sea necesario, típicamente dentro de un aggregate.

Ejemplo:

UserCreated
      ↓
UserActivated

El sistema no debe asumir que eventos no relacionados de distintos usuarios llegarán globalmente ordenados.

28. Idempotency

Los consumidores deben ser idempotentes.

Ejemplo:

UserCreated
      │
      ├── Delivery 1
      └── Delivery 2

El consumidor debe producir el mismo estado final.

Puede utilizar:

event_id

como identificador de deduplicación.

29. At-Least-Once Delivery

La arquitectura de EVOXA contempla entrega at-least-once con reintentos y DLQ.

Por tanto:

Consumer
   │
   ▼
Process Event
   │
   ├── Success → ACK
   │
   └── Failure → Retry

El consumidor no debe asumir exactamente-once delivery.

30. Retry

Los errores temporales deben poder reintentarse.

Conceptualmente:

Attempt 1
   ↓
Failure
   ↓
Attempt 2
   ↓
Failure
   ↓
Attempt 3
   ↓
Success

La estrategia concreta de:

cantidad;
backoff;
máximo tiempo;

debe definirse en la arquitectura de Event Platform.

31. Dead Letter Queue

Si un evento no puede procesarse después de los reintentos:

Event
 ↓
Retry
 ↓
Retry
 ↓
Retry
 ↓
DLQ

La DLQ debe permitir:

inspección;
corrección;
replay;
trazabilidad.
32. Event Replay

Identity debe permitir reconstruir determinados procesos a partir de eventos cuando corresponda.

Pero no todos los eventos deben convertirse automáticamente en fuente de verdad.

El modelo inicial recomendado continúa siendo:

PostgreSQL
+
Domain Events
+
Audit

y no Event Sourcing completo.

Esto es coherente con la arquitectura de datos de EVOXA, que reserva event sourcing para determinados aggregates donde aporta valor.

33. Transactional Outbox

Para evitar:

Database committed
BUT
Event lost

se recomienda evaluar:

Database Transaction
      │
      ├── Identity Change
      │
      └── Outbox Event
                │
                ▼
          Event Publisher
                │
                ▼
          Event Platform

Esto debe convertirse en un ADR específico antes de implementación.

34. Consumers

Consumidores potenciales de Identity Events:

Notifications
UserCreated
PasswordReset
MembershipCreated
Analytics
Authentication
Session
Membership
Organization
Security
AuthenticationFailed
TokenReuseDetected
SecurityPolicyViolation
Administration
UserSuspended
RoleChanged
OrganizationChanged
AI Platform

En escenarios futuros:

User / Organization lifecycle signals

pero con controles estrictos de privacidad y minimización de datos.

35. Identity Consuming Events

Identity también puede consumir eventos de otros dominios.

Sin embargo, debe existir una separación clara entre:

Identity owns identity state

y:

Other domains notify Identity of relevant external facts

Ejemplos potenciales futuros:

OrganizationProvisioningRequested
ExternalIdentityLinked
EnterpriseUserProvisioned

Estos requieren contratos explícitos.

36. Integration with External Identity Providers

La arquitectura de EVOXA contempla proveedores como:

Entra ID
Google
Okta
Auth0
SAML2
OIDC

Identity debe encapsular estos proveedores detrás de una abstracción.

External Identity Provider
          │
          ▼
    Identity Adapter
          │
          ▼
      EVOXA Identity

No debe propagarse la lógica específica de un proveedor a todo el dominio.

37. Webhooks

Para integraciones externas puede ser necesario publicar eventos mediante webhooks.

Ejemplo:

Identity Event
      │
      ▼
Webhook Dispatcher
      │
      ▼
External System

Los webhooks deben incluir:

firma;
timestamp;
event ID;
retries;
replay protection.

La arquitectura API de EVOXA contempla webhooks firmados y reintentables.

38. External Event Consumers

Los consumidores externos no deben depender directamente de tablas de Identity.

Incorrecto:

External System
      ↓
Identity Database

Correcto:

Identity
      ↓
API / Events / Webhooks
      ↓
External System

Esto preserva el ownership de datos.

39. Event Security

Los eventos pueden contener información sensible.

Por ello deben protegerse mediante:

autenticación de consumers;
autorización;
encryption in transit;
encryption at rest;
tenant validation;
schema validation;
access control.

La arquitectura de seguridad exige TLS para datos en tránsito y controles de aislamiento de tenant.

40. Event Access Control

No todos los consumidores deben recibir todos los eventos.

Ejemplo:

Security Service
     → AuthenticationFailed

Analytics
     → Aggregated Authentication Metrics

Notification
     → UserCreated

Un consumer debe recibir únicamente los streams/topics necesarios.

41. Sensitive Event Filtering

Un evento de seguridad puede necesitar una versión:

Internal Security Event

y otra:

External Integration Event

con distinto nivel de detalle.

Esto evita filtrar información interna innecesaria.

42. Event Schema Registry

Se recomienda un mecanismo centralizado de schemas.

Conceptualmente:

Event Schema Registry
       │
       ├── UserCreated v1
       ├── UserCreated v2
       ├── RoleAssigned v1
       └── MFAFactorActivated v1

El mecanismo tecnológico concreto queda pendiente.

43. Schema Compatibility

Cambios de eventos deben clasificarse:

Compatible
Potentially Breaking
Breaking

Cuando sea necesario:

Event v1
Event v2

deben coexistir durante una transición.

44. Observability

Cada evento debe poder rastrearse.

HTTP Request
   │
   ▼
Application
   │
   ▼
Database
   │
   ▼
Outbox
   │
   ▼
Event Platform
   │
   ▼
Consumer

Los identificadores:

trace_id
span_id
correlation_id
causation_id
event_id

permiten seguir todo el flujo.

La arquitectura de observabilidad de EVOXA exige distributed tracing y correlación entre APIs, Event Bus, bases de datos, IA e integraciones externas.

45. Event Metrics

Identity debe producir métricas como:

identity_events_published_total
identity_events_failed_total
identity_events_retried_total
identity_events_dlq_total
identity_event_processing_latency
identity_event_consumer_failures

También por:

event_type
consumer
tenant
result
46. Failure Handling

Si Event Platform está temporalmente indisponible:

Identity Operation
       │
       ▼
Database
       │
       ▼
Outbox
       │
       ▼
Event Publisher
       │
       X
Event Platform unavailable

El evento debe permanecer pendiente para posterior publicación si se adopta Outbox.

La operación de Identity no debería perderse simplemente porque el broker estuvo temporalmente indisponible.

47. Integration Boundaries

Identity expone:

APIs
Events
Webhooks

y consume:

External Identity Providers
Platform Events
Provisioning Events

Pero no debe exponer directamente:

Database
Internal ORM
Internal Domain Objects
48. Testing
Unit Tests
Event creation.
Event validation.
Versioning.
Payload filtering.
Correlation.
Causation.
Integration Tests
Identity
 ↓
Outbox
 ↓
Event Publisher
 ↓
Event Platform
Consumer Tests
Idempotency.
Duplicate events.
Ordering.
Retry.
DLQ.
Contract Tests

Validar schemas.

Security Tests
Unauthorized consumer.
Cross-tenant event.
Event tampering.
Sensitive data leakage.
49. ADRs necesarios
ADR-IDENTITY-045

Identity Event Taxonomy

Definir formalmente:

Domain Events
Integration Events
Security Events
Audit Events
ADR-IDENTITY-046

Identity Event Transport

Definir tecnología y patrón del Event Platform.

ADR-IDENTITY-047

Transactional Outbox

Determinar si será obligatorio para Identity.

ADR-IDENTITY-048

Event Schema Registry

Definir cómo se versionan y validan los schemas.

ADR-IDENTITY-049

Event Retention

Definir cuánto tiempo se mantienen los eventos.

ADR-IDENTITY-050

Event Replay

Definir qué eventos pueden reproducirse y bajo qué controles.

ADR-IDENTITY-051

Identity Event Security

Definir:

encryption;
authentication;
authorization;
tenant isolation;
consumer access.
ADR-IDENTITY-052

External Webhooks

Definir firma, retry, replay protection y delivery model.

50. Criterios de aceptación

ESP-0001/12 estará listo para implementación cuando:

 Event envelope definido.
 Event taxonomy definida.
 Event catalog definido.
 Event versioning definido.
 Tenant context definido.
 Actor definido.
 Resource definido.
 Correlation definido.
 Causation definido.
 Ordering definido.
 Idempotency definida.
 Retry definido.
 DLQ definido.
 Replay definido.
 Outbox decidido.
 Schema Registry decidido.
 Event security definido.
 Consumer access definido.
 External integrations definidas.
 Webhooks definidos.
 Observability definida.
 Event retention definida.
 ADRs aprobados.
 Contract tests definidos.
 Integration tests definidos.
51. Trazabilidad
BP-0002 Reference Architecture
          │
          ▼
20 — Event Architecture
          │
          ▼
ESP-0001 Identity
          │
 ┌────────┼───────────────┐
 ▼        ▼               ▼
Domain   Security       Audit
Events   Events         Events
 │        │               │
 └────────┼───────────────┘
          ▼
   Integration Events
          │
          ▼
    Event Platform
          │
 ┌────────┼───────────────┐
 ▼        ▼               ▼
Analytics Security   Notifications
          │
          ▼
   Other EVOXA Domains
          │
          ▼
 External Integrations

La arquitectura de eventos de EVOXA establece precisamente este enfoque: eventos como hechos inmutables, consumidores idempotentes, entrega at-least-once, retries, DLQ, replay y correlación/causación.

52. Estado completo de ESP-0001
#	Sección	Estado
01	Identity Domain Model	Draft
02	Identity Architecture	Draft
03	Authentication & Authorization	Draft
04	User & Organization Management	Draft
05	Roles & Permissions	Draft
06	Sessions & Token Management	Draft
07	Multi-Factor Authentication	Draft
08	Security Policies	Draft
09	Audit & Compliance	Draft
10	API Contracts	Draft
11	Persistence & Data Model	Draft
12	Events & Integration	Draft
Siguiente sección

Con 01–12, Identity ya tiene cubiertos los principales bloques de diseño:

Domain
Architecture
Authentication
Authorization
Users
Organizations
Roles
Permissions
Sessions
Tokens
MFA
Security Policies
Audit
APIs
Persistence
Events
Integrations

La siguiente sección lógica sería:

ESP-0001 / 13 — Identity Testing & Quality Requirements

Ahí vamos a definir cómo se valida Identity antes de permitir que llegue a implementación, incluyendo unit tests, integration tests, API contract tests, security testing, MFA testing, tenant isolation, performance, resiliencia, auditoría y criterios de calidad. Esto conecta directamente con el capítulo 27 de Testing & QA del Blueprint y nos acerca al punto donde podremos comenzar a convertir ESP-0001 en Implementation Stories (IS).
