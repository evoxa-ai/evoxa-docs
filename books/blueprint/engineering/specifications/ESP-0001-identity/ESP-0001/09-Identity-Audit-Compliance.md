ESP-0001 / 09 — Identity Audit & Compliance

Estado: Draft
Dominio: Identity
Código: ESP-0001
Sección: 09 — Audit & Compliance
Parent Specification: ESP-0001 — Identity Engineering Specification

1. Propósito

Este documento define la capacidad de Audit & Compliance del dominio Identity de EVOXA.

Su objetivo es garantizar que las operaciones relacionadas con identidad, autenticación, autorización, sesiones, tokens, MFA, usuarios, organizaciones, roles y permisos sean:

trazables;
auditables;
verificables;
protegidas contra manipulación;
asociadas a un actor;
asociadas a un tenant cuando corresponda;
correlacionables con una petición, sesión o evento;
utilizables como evidencia de seguridad y cumplimiento.

La arquitectura de EVOXA establece Auditability como principio de seguridad y contempla auditoría de operaciones críticas.

Además, la arquitectura de observabilidad establece que los eventos relevantes deben poder correlacionarse mediante identificadores como trace_id, span_id y correlation_id, y contempla retención prolongada de registros de auditoría.

2. Objetivos

Identity Audit & Compliance debe permitir:

Registrar operaciones críticas.
Registrar autenticaciones exitosas y fallidas.
Registrar decisiones de autorización relevantes.
Registrar cambios de roles y permisos.
Registrar cambios de MFA.
Registrar creación, modificación, suspensión y desactivación de usuarios.
Registrar cambios de organizaciones y memberships.
Registrar sesiones y tokens relevantes.
Mantener trazabilidad de operaciones administrativas.
Proteger los registros contra manipulación.
Mantener contexto de tenant.
Facilitar investigaciones de seguridad.
Facilitar evidencia de cumplimiento.
Permitir consultas controladas.
Integrarse con Event Platform y Observability.
Mantener separación entre auditoría y logging técnico.
3. Auditoría ≠ Logging

Es importante separar ambos conceptos.

Application Logging

Sirve principalmente para:

Debugging
Errors
Performance
Operational Diagnostics
Audit Logging

Sirve para:

Security
Accountability
Compliance
Forensics
Traceability

Por ejemplo:

Application Log

"Database query failed"

mientras que:

Audit Log

"Administrator changed user's role"

Son mecanismos diferentes y no deben mezclarse.

4. Principios

Audit & Compliance debe respetar:

Immutable where required
Least Privilege
Privacy by Design
Security by Design
Tenant Isolation
Traceability
Accountability
Non-repudiation where applicable
Data Minimization
Secure by Default
Fail Closed
5. Audit Event

La unidad principal de auditoría será el:

AuditEvent

Modelo conceptual:

AuditEvent
----------
id
event_type
action
timestamp
actor
target
organization
result
source
ip
user_agent
session
correlation_id
trace_id
metadata
version
6. Actor

Cada operación auditable debe identificar quién la realizó cuando sea posible.

Ejemplos:

USER
ADMIN
SERVICE
SYSTEM
AI

Conceptualmente:

Actor
-----
type
id
name
authentication_context

No debe almacenarse información sensible innecesaria.

7. Target

El evento debe identificar sobre qué recurso ocurrió la operación.

Ejemplo:

Target
------
type: User
id: user_123

o:

Target
------
type: Role
id: role_admin

Esto permite responder:

¿Quién modificó qué?

8. Organization / Tenant Context

Cuando la operación ocurre dentro de una organización, el evento debe incluir el contexto correspondiente.

tenant_id
organization_id
membership_id

La información exacta dependerá del modelo final de Identity.

Esto es fundamental para impedir que una auditoría de una organización quede mezclada con otra.

9. Audit Event Structure

Propuesta conceptual:

{
  "event_id": "...",
  "event_type": "USER_ROLE_CHANGED",
  "event_version": 1,
  "timestamp": "...",
  "actor": {
    "type": "USER",
    "id": "..."
  },
  "tenant_id": "...",
  "target": {
    "type": "USER",
    "id": "..."
  },
  "action": "ROLE_CHANGED",
  "result": "SUCCESS",
  "correlation_id": "...",
  "trace_id": "...",
  "metadata": {}
}

El formato definitivo debe alinearse con el modelo de eventos de EVOXA, que contempla event_id, event_type, event_version, timestamp, tenant_id, actor, resource, payload, metadata, correlation_id y causation_id.

10. Result

Una operación auditable debe indicar su resultado.

SUCCESS
FAILURE
DENIED
BLOCKED
CHALLENGED

Ejemplo:

LOGIN
FAILURE

o:

ROLE_CHANGE
SUCCESS

Esto permite analizar tanto operaciones exitosas como intentos fallidos.

11. Authentication Audit

Identity debe registrar eventos importantes de autenticación.

Ejemplos:

UserLoginSucceeded
UserLoginFailed
AuthenticationStarted
AuthenticationBlocked
AuthenticationRequired
PasswordChanged
PasswordResetRequested
PasswordResetCompleted

También:

MFAChallengeCreated
MFAChallengeSucceeded
MFAChallengeFailed
12. Authorization Audit

Las decisiones de autorización sensibles deben ser auditables.

Ejemplos:

AuthorizationGranted
AuthorizationDenied
PermissionDenied
CrossTenantAccessDenied
PolicyDenied
StepUpRequired

No necesariamente debe registrarse cada lectura normal de cada endpoint.

La política de auditoría debe distinguir entre:

Normal Operations
Sensitive Operations
Critical Operations
13. User Audit

Operaciones importantes sobre usuarios:

UserCreated
UserUpdated
UserActivated
UserSuspended
UserDisabled
UserReactivated
UserDeleted

Cuando corresponda, también:

EmailChanged
PasswordChanged
SecuritySettingsChanged
14. Organization Audit

Operaciones sobre organizaciones:

OrganizationCreated
OrganizationUpdated
OrganizationSuspended
OrganizationReactivated
OrganizationSecurityPolicyChanged

También deberán auditarse operaciones administrativas sobre la organización.

15. Membership Audit

Cambios de pertenencia:

MembershipCreated
MembershipActivated
MembershipSuspended
MembershipRemoved
MembershipRoleChanged

Esto permite reconstruir la evolución de los accesos de una persona dentro de una organización.

16. Role Audit

Cambios de roles:

RoleCreated
RoleUpdated
RoleDeleted
RoleAssigned
RoleRemoved

Especialmente importante:

Admin Role Granted
Admin Role Revoked

Estas operaciones deben tener un nivel de auditoría reforzado.

17. Permission Audit

Cambios de permisos:

PermissionCreated
PermissionUpdated
PermissionAssigned
PermissionRevoked

Ejemplo:

Administrator
       │
       ▼
Permission Granted
       │
       ▼
Audit Event

Esto permite determinar quién otorgó privilegios.

18. MFA Audit

Basado en ESP-0001/07, deben auditarse:

MFAEnrollmentStarted
MFAFactorCreated
MFAFactorVerified
MFAFactorActivated
MFAFactorRevoked
MFAChallengeFailed
MFARecoveryInitiated
MFARecoveryCodeUsed
MFADisabled
TrustedDeviceRegistered
TrustedDeviceRevoked

Las operaciones de MFA son particularmente sensibles.

19. Session Audit

Eventos relacionados con sesiones:

SessionCreated
SessionAuthenticated
SessionRevoked
SessionExpired
Logout
GlobalLogout

No se deben almacenar tokens completos en el audit log.

Debe utilizarse un identificador seguro o referencia.

20. Token Audit

Deben registrarse determinados eventos de seguridad relacionados con tokens.

Por ejemplo:

RefreshTokenRotated
RefreshTokenRevoked
TokenReuseDetected
TokenValidationFailed

Nunca:

Access Token = almacenado en AuditLog

ni:

Refresh Token = almacenado en AuditLog

Los secretos no forman parte del registro de auditoría.

21. Security Policy Audit

Las modificaciones de políticas deben auditarse.

Ejemplos:

SecurityPolicyCreated
SecurityPolicyUpdated
SecurityPolicyActivated
SecurityPolicyDisabled
SecurityPolicyDeleted

También debe registrarse:

Which policy
Which version
Who changed it
When
Why / reason when required
22. Administrative Audit

Las operaciones administrativas requieren un nivel elevado de trazabilidad.

Ejemplos:

GrantAdminRole
RevokeAdminRole
ResetUserMFA
DisableUser
UnlockUser
ChangeOrganizationSecurity
ModifyPermission
ModifySecurityPolicy

La auditoría debe registrar el actor administrativo y el recurso afectado.

23. Before / After State

Para determinadas operaciones de modificación puede ser necesario registrar el cambio.

Ejemplo:

Role:

BEFORE
Viewer

AFTER
Coach

Pero no todos los atributos deben registrarse.

Nunca debería incluir:

password
password_hash
TOTP_secret
private_key
refresh_token
access_token
recovery_code

La auditoría debe aplicar data minimization.

24. Privacy

Audit logs pueden contener información personal.

Por ello deben tratarse como datos protegidos.

Debe controlarse:

quién puede consultarlos;
cuánto tiempo se almacenan;
qué campos contienen;
exportación;
eliminación cuando legalmente corresponda;
anonimización cuando sea posible.

La arquitectura de EVOXA establece Privacy by Design y contempla cumplimiento regulatorio dependiente del despliegue, incluyendo GDPR, HIPAA cuando corresponda, ISO 27001, SOC 2 y regulaciones locales.

25. Audit Retention

La arquitectura de observabilidad establece una referencia de:

Audit Logs Retention: 7 years

Sin embargo, esto debe tratarse como objetivo arquitectónico sujeto a las obligaciones legales y al despliegue específico.

No todas las organizaciones necesariamente tendrán exactamente las mismas obligaciones regulatorias.

26. Immutability

Los registros de auditoría críticos deberían ser inmutables.

Conceptualmente:

Application
     │
     ▼
Audit Event
     │
     ▼
Append Only Store
     │
     ▼
Long Term Storage

No debería existir una operación normal como:

UPDATE audit_event

ni:

DELETE audit_event

por usuarios administrativos comunes.

27. Tamper Protection

Como evolución de seguridad, pueden utilizarse mecanismos como:

Hash chaining
Digital signatures
Write-once storage
External immutable storage

El mecanismo concreto no está definido por el Blueprint y debe resolverse mediante ADR.

28. Audit Integrity

Una futura estrategia podría mantener:

event_hash
previous_event_hash

por ejemplo:

Event A
   │
   hash A
   │
   ▼
Event B
   │
   hash B
   │
   ▼
Event C

Esto permitiría detectar modificaciones posteriores.

Es una propuesta técnica, no una obligación explícita del Blueprint.

29. Correlation

Cada operación relevante debe poder correlacionarse.

Ejemplo:

HTTP Request
     │
     ├── trace_id
     ├── span_id
     └── correlation_id
            │
            ▼
       Audit Event
            │
            ▼
        Domain Event

Esto permite investigar una operación desde:

Request
 ↓
Authentication
 ↓
Authorization
 ↓
Business Action
 ↓
Audit
 ↓
Event

La arquitectura de observabilidad de EVOXA exige trazabilidad distribuida mediante estos identificadores.

30. Causation

Cuando una acción provoca otra, puede registrarse:

causation_id

Ejemplo:

Admin disables user
        │
        ▼
UserDisabled
        │
        ▼
SessionsRevoked

El segundo evento puede conservar la relación causal con el primero.

Esto sigue el modelo de eventos definido en la arquitectura de EVOXA.

31. Audit vs Domain Events

No debemos confundir:

Domain Event

con:

Audit Event

Un Domain Event representa:

Algo importante ocurrió en el negocio.

Un Audit Event representa:

Quién realizó una operación y qué ocurrió desde la perspectiva de seguridad y accountability.

Pueden estar relacionados:

UserRoleChanged
       │
       ├── Domain Event
       │
       └── Audit Event

Pero no necesariamente deben ser el mismo objeto.

32. Audit Storage

La arquitectura utiliza una estrategia de persistencia poliglota.

Para auditoría se propone conceptualmente:

PostgreSQL
     │
     └── Operational Audit

Event Platform
     │
     └── Audit Events

Object / Immutable Storage
     │
     └── Long-Term Compliance Archive

La tecnología exacta de almacenamiento inmutable todavía debe definirse.

33. Audit Query

Los usuarios autorizados podrán consultar auditoría.

Conceptualmente:

GET /api/v1/identity/audit

Filtros posibles:

actor
organization
user
event_type
action
result
target
date_from
date_to
correlation_id

Por ejemplo:

GET /api/v1/identity/audit
    ?event_type=ROLE_CHANGED
    &organization_id=...

El contrato definitivo se definirá posteriormente en la sección API Contracts.

34. Audit Access Control

No todos los usuarios deben poder consultar auditoría.

Modelo:

User
  │
  ▼
Role
  │
  ▼
Permission
  │
  ▼
Audit Access Policy
  │
  ▼
Audit Records

Los registros pueden contener información altamente sensible.

35. Platform vs Organization Audit

Debe existir separación conceptual entre:

Platform Audit

Operaciones realizadas por administradores globales.

Organization Audit

Operaciones dentro de un tenant.

Ejemplo:

Platform
 └── Platform Admin Actions

Organization A
 └── Organization Audit

Organization B
 └── Organization Audit

Un administrador de Organization A no debe consultar auditoría de Organization B.

36. Compliance Evidence

Audit & Compliance debe permitir producir evidencia de controles como:

Who accessed?
Who changed permissions?
Who disabled the user?
When was MFA enabled?
When was MFA removed?
Who modified security policy?
Which sessions were revoked?

Esto convierte la auditoría en una fuente de evidencia operativa.

37. Compliance Frameworks

La arquitectura de EVOXA contempla compatibilidad con marcos regulatorios y de seguridad como:

GDPR
HIPAA
ISO 27001
SOC 2
Local Health Regulations

pero no significa que EVOXA sea automáticamente compliant con todos ellos.

El cumplimiento depende de:

arquitectura desplegada;
configuración;
procesos organizacionales;
controles operacionales;
ubicación de datos;
obligaciones regulatorias;
políticas de la organización.

38. Data Subject / Privacy Operations

En escenarios regulados pueden existir operaciones relacionadas con:

Data Access
Data Export
Data Rectification
Data Deletion
Data Retention
Consent

La auditoría debe registrar dichas operaciones cuando corresponda.

Pero debe evitar registrar innecesariamente el contenido completo de los datos personales afectados.

39. Audit Export

Los administradores autorizados pueden necesitar exportar auditoría.

Formatos potenciales:

JSON
CSV
PDF

La exportación debe:

estar autorizada;
generar audit event;
aplicar filtros de tenant;
proteger información sensible;
ser trazable.

Ejemplo:

Admin exports audit
       │
       ▼
AuditExportRequested
       │
       ▼
Authorization
       │
       ▼
Generate Export
       │
       ▼
AuditExportCompleted
40. Audit Monitoring

El sistema de observabilidad puede detectar:

Repeated authentication failures
Repeated authorization denials
Privilege escalation
Mass role changes
MFA disabled unexpectedly
Multiple token reuse detections
Cross-tenant attempts
Administrative anomalies

Esto puede alimentar posteriormente el Risk Engine y AI Security.

41. AI y Audit

Como EVOXA es AI-Native, las acciones realizadas por sistemas de IA que afecten Identity deben poder distinguirse de acciones humanas.

Ejemplo:

Actor
type = AI
id = ai-agent-123

Sin embargo:

La IA no debe tener automáticamente permisos administrativos.

Cualquier acción realizada por un agente debe estar sometida a:

Authentication
Authorization
Policy
Audit

y a los controles de seguridad correspondientes.

La arquitectura de EVOXA establece AI Safety, accountability y trazabilidad como componentes esenciales del sistema de IA.

42. Audit Event Versioning

Los eventos deben versionarse.

USER_ROLE_CHANGED.v1
USER_ROLE_CHANGED.v2

Esto permite evolucionar el esquema sin romper consumidores históricos.

43. Audit Lifecycle

Conceptualmente:

Generated
   ↓
Validated
   ↓
Stored
   ↓
Replicated
   ↓
Archived
   ↓
Retained
   ↓
Disposed / Anonymized

El último estado dependerá de:

legislación;
política de retención;
contrato;
tipo de dato;
organización.
44. Failure Handling

Una pregunta crítica es:

¿Qué ocurre si el sistema no puede registrar una operación crítica?

Para operaciones de alta sensibilidad, la estrategia debe evitar que una acción crítica se ejecute sin trazabilidad cuando la política lo considere obligatorio.

Conceptualmente:

Critical Operation
       │
       ▼
Audit Required
       │
   ┌───┴────┐
   │        │
Success   Failure
   │        │
   ▼        ▼
Continue   DENY

Este comportamiento debe definirse formalmente mediante ADR.

45. Performance

Audit logging no debe degradar significativamente las APIs.

La arquitectura establece objetivos de rendimiento como:

API P95 < 300 ms
Authentication P95 < 500 ms

Por ello, para eventos que no requieren confirmación síncrona podría utilizarse:

Application
     │
     ▼
Event
     │
     ▼
Event Platform
     │
     ▼
Audit Consumer

Mientras que ciertas operaciones críticas pueden requerir confirmación antes de completar.

46. Security Events

Identity debe publicar eventos de seguridad importantes.

Ejemplos:

AuthenticationFailure
AccountLocked
TokenReuseDetected
MFAFailure
MFARevoked
PrivilegeEscalation
AuthorizationDenied
CrossTenantAccessAttempt
SecurityPolicyViolation

Estos pueden alimentar:

Observability
Security Operations
Risk Engine
Analytics
AI Security
47. Testing
Unit Tests
Audit event construction.
Event validation.
Sensitive-field filtering.
Tenant context.
Event versioning.
Integration Tests
Authentication → Audit.
MFA → Audit.
Authorization → Audit.
Role changes → Audit.
Policy changes → Audit.
Security Tests
Audit tampering.
Unauthorized audit access.
Cross-tenant audit access.
Sensitive data leakage.
Event injection.
Event deletion.
Privilege escalation.
Compliance Tests
Retention.
Export.
Access controls.
Immutability.
Traceability.
48. ADRs necesarios
ADR-IDENTITY-020

Audit Storage Architecture

Definir:

PostgreSQL;
Event Store;
Object Storage;
immutable storage.
ADR-IDENTITY-021

Audit Immutability

Definir cómo se evita:

UPDATE
DELETE
TAMPERING
ADR-IDENTITY-022

Audit Retention

Definir:

default;
regulatory;
organization-specific;
legal hold.
ADR-IDENTITY-023

Audit Event vs Domain Event

Definir cuándo un evento debe ser:

Domain Event
Audit Event
Security Event

o más de uno.

ADR-IDENTITY-024

Critical Operation Audit Failure

Definir si:

Audit unavailable
      ↓
DENY operation

para operaciones críticas.

ADR-IDENTITY-025

Audit Privacy & Data Minimization

Definir qué información puede registrarse y qué información debe excluirse o anonimizarse.

ADR-IDENTITY-026

Audit Integrity

Definir si se utilizarán:

Hash chaining
Digital signatures
WORM storage

u otra estrategia.

49. Criterios de aceptación

ESP-0001/09 estará listo para implementación cuando:

 Exista modelo definitivo de AuditEvent.
 Esté definido Actor.
 Esté definido Target.
 Esté definido Tenant Context.
 Estén definidos eventos de autenticación.
 Estén definidos eventos de autorización.
 Estén definidos eventos de usuarios.
 Estén definidos eventos de organizaciones.
 Estén definidos eventos de memberships.
 Estén definidos eventos de roles.
 Estén definidos eventos de permisos.
 Estén definidos eventos MFA.
 Estén definidos eventos de sesiones.
 Estén definidos eventos de tokens.
 Estén definidos eventos de Security Policies.
 Esté definida la política de datos sensibles.
 Esté definida la retención.
 Esté definida la inmutabilidad.
 Esté definido el acceso a auditoría.
 Esté definida la exportación.
 Estén definidos los eventos de seguridad.
 Estén definidos los ADR.
 Estén definidos los API Contracts.
 Existan pruebas de seguridad.
 Existan pruebas de integración.
 Existan pruebas de compliance.
50. Trazabilidad
BP-0002 Reference Architecture
          │
          ├── Security
          ├── Observability
          ├── Privacy
          └── Compliance
                  │
                  ▼
          ESP-0001 Identity
                  │
      ┌───────────┼────────────┐
      ▼           ▼            ▼
Authentication   MFA      Authorization
      │           │            │
      └───────────┼────────────┘
                  ▼
          ESP-0001/08
       Security Policies
                  │
                  ▼
          ESP-0001/09
       Audit & Compliance
                  │
       ┌──────────┼───────────┐
       ▼          ▼           ▼
     Audit     Security    Compliance
     Events     Events       Evidence
       │          │           │
       └──────────┼───────────┘
                  ▼
           Event Platform
                  │
                  ▼
            Observability
                  │
                  ▼
             Analytics

Esta trazabilidad sigue el principio del Architecture Map de EVOXA: Blueprint → Engineering Specifications → Implementation Stories → Technical Tasks → Source Code → Testing → Deployment → Monitoring → Continuous Evolution.

51. Estado de ESP-0001
Sección	Estado
01 — Identity Domain Model	Draft
02 — Identity Architecture	Draft
03 — Authentication & Authorization	Draft
04 — User & Organization Management	Draft
05 — Roles & Permissions	Draft
06 — Sessions & Token Management	Draft
07 — Multi-Factor Authentication	Draft
08 — Security Policies	Draft
09 — Audit & Compliance	Draft
Siguiente paso

La siguiente sección lógica sería:

ESP-0001 / 10 — Identity API Contracts

Ahí ya podemos empezar a transformar todo lo definido en 01–09 en contratos REST/OpenAPI concretos, incluyendo:

Authentication
Users
Organizations
Memberships
Roles
Permissions
Sessions
Tokens
MFA
Security Policies
Audit

Y esto será especialmente importante porque el Blueprint establece API-First: el contrato API debe definirse antes de la implementación y posteriormente utilizarse para documentación y pruebas de contrato.
