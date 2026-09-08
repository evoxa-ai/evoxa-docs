ESP-0001 / 09 — Identity Audit & Compliance

Estado: Draft — Consolidated
Dominio: Identity
Código: ESP-0001
Sección: 09 — Audit & Compliance
Versión: 1.1.0
Parent Specification: ESP-0001 — Identity Engineering Specification
Target Repository: evoxa-platform
Target Path: apps/api/app/domains/identity/

1. Propósito

Esta especificación define la capacidad de Audit & Compliance del dominio Identity de EVOXA.

Su objetivo es garantizar que las operaciones relacionadas con:

identidad;
autenticación;
autorización;
sesiones;
tokens;
MFA;
usuarios;
organizaciones;
memberships;
roles;
permisos;
delegaciones;
security policies;
recovery;
security state;

sean:

trazables;
auditables;
verificables;
correlacionables;
protegidas contra manipulación;
asociadas a un actor cuando corresponda;
asociadas a un tenant;
utilizables como evidencia de seguridad;
utilizables como evidencia de cumplimiento.

La especificación original ya establece estos objetivos y distingue explícitamente Audit Logging de Application Logging.

2. Objetivos

Identity Audit & Compliance debe permitir:

Registrar operaciones críticas.
Registrar autenticaciones relevantes.
Registrar decisiones de autorización relevantes.
Registrar cambios de privilegios.
Registrar cambios de MFA.
Registrar cambios de sesiones y tokens.
Registrar cambios de usuarios.
Registrar cambios de organizaciones y memberships.
Registrar cambios de roles y permisos.
Registrar cambios de delegaciones.
Registrar cambios de Security Policies.
Mantener tenant isolation.
Mantener correlación distribuida.
Proteger la integridad de los registros.
Facilitar investigaciones de seguridad.
Producir evidencia de compliance.
Permitir consultas autorizadas.
Permitir exportación controlada.
Integrarse con Event Platform.
Integrarse con Observability.
Mantener separación entre audit y technical logging.
Mantener trazabilidad de decisiones de autorización y seguridad.
3. Audit ≠ Application Logging

Esta separación es normativa.

Application Logging

Está destinado principalmente a:

Debugging
Errors
Diagnostics
Performance
Operational Troubleshooting

Ejemplo:

Database connection timeout
Audit Logging

Está destinado a:

Security
Accountability
Compliance
Forensics
Traceability
Non-repudiation where applicable

Ejemplo:

Administrator changed user's role

La arquitectura original ya establece esta separación explícitamente.

4. Audit Event

La unidad principal de auditoría será:

AuditEvent

Modelo conceptual:

AuditEvent
--------------------------------
event_id
event_type
event_version
timestamp
actor
tenant_id
organization_id
membership_id
target
action
result
source
session_reference
correlation_id
causation_id
trace_id
metadata
version

El modelo definitivo debe alinearse con el Event Architecture de EVOXA.

5. Actor

Cada operación auditable debe identificar el actor cuando sea posible.

Tipos conceptuales:

USER
SERVICE
SYSTEM
AI_AGENT

El actor puede contener:

type
id
authentication_context

No debe incluir información sensible innecesaria.

Un actor de tipo AI_AGENT debe conservarse explícitamente como tal para garantizar trazabilidad de acciones realizadas por sistemas de IA.

6. Target

El AuditEvent debe identificar el recurso afectado.

Ejemplo:

target:
  type: USER
  id: user-123

o:

target:
  type: ROLE
  id: role-admin

Esto permite responder:

¿Quién modificó qué?

7. Tenant Context

Toda operación que ocurra dentro de una organización debe mantener explícitamente el contexto correspondiente.

Conceptualmente:

tenant_id
organization_id
membership_id

La separación tenant-aware es obligatoria.

Un administrador de:

Organization A

no debe poder consultar:

Organization B

simplemente porque posee permisos administrativos dentro de A.

8. Audit Event Structure

El evento conceptual será compatible con el envelope de eventos de EVOXA:

{
  "event_id": "...",
  "event_type": "USER_ROLE_CHANGED",
  "event_version": 1,
  "timestamp": "...",
  "tenant_id": "...",
  "actor": {
    "type": "USER",
    "id": "..."
  },
  "resource": {
    "type": "USER",
    "id": "..."
  },
  "payload": {},
  "metadata": {
    "correlation_id": "...",
    "causation_id": "...",
    "trace_id": "..."
  }
}

El formato exacto será definido por ESP-0001/12 — Events & Integration.

9. Audit Result

Una operación auditable debe poder indicar su resultado.

Valores conceptuales:

SUCCESS
FAILURE
DENIED
BLOCKED
CHALLENGED

Ejemplos:

LOGIN → FAILURE
ROLE_CHANGE → SUCCESS
MFA_CHANGE → BLOCKED
AUTHORIZATION → DENIED

Registrar intentos fallidos es importante porque los intentos de ataque también forman parte de la evidencia de seguridad.

10. Authentication Audit

Deben auditarse operaciones importantes relacionadas con autenticación.

Ejemplos:

AuthenticationStarted
UserLoginSucceeded
UserLoginFailed
AuthenticationBlocked
AuthenticationRequired
PasswordChanged
PasswordResetRequested
PasswordResetCompleted

También:

MFAChallengeCreated
MFAChallengeSucceeded
MFAChallengeFailed

No significa que absolutamente cada evento técnico deba convertirse automáticamente en un AuditEvent.

La política de auditoría debe determinar qué eventos son auditables.

11. Authorization Audit

Las decisiones de autorización relevantes deben poder auditarse.

Ejemplos:

AuthorizationGranted
AuthorizationDenied
PermissionDenied
CrossTenantAccessDenied
PolicyDenied
StepUpRequired
ReauthenticationRequired
MFARequired

Especialmente deben poder reconstruirse operaciones:

Privilege Change
Administrative Action
Cross-Tenant Attempt
Security Configuration Change
Sensitive Operation
Critical Operation

No es necesario generar una auditoría individual para cada lectura normal de bajo riesgo.

12. Policy Decision Audit

La integración con Security Policy Engine requiere conservar, cuando corresponda:

policy_id
policy_version
decision
reason_code
criticality
freshness_class
cacheability
authorization_context_reference
runtime_version

Ejemplo:

Policy:
mfa-security

Version:
4

Decision:
REQUIRE_MFA

Criticality:
C0

Freshness:
F0

Esto es particularmente importante porque una decisión de autorización puede cambiar cuando cambia una política.

13. Criticality Audit

Cuando una operación crítica sea evaluada, la evidencia debe poder conservar:

base_criticality
dynamic_risk
security_state
platform_minimum
effective_criticality

Recordemos:

Criticality ≠ Authorization Decision

Por ejemplo:

Decision:
ALLOW

Criticality:
C0

es perfectamente posible.

La criticidad describe el nivel de garantías requeridas; no concede el permiso.

14. Freshness Audit

Para operaciones donde la frescura de autorización sea relevante, puede conservarse:

freshness_class
security_epoch
policy_version
entity_version
context_fingerprint
cache_generation
cacheability

Esto permite demostrar posteriormente que una decisión crítica no dependió de un estado de autorización obsoleto.

15. User Audit

Deben auditarse cambios significativos sobre usuarios:

UserCreated
UserUpdated
UserActivated
UserSuspended
UserDisabled
UserReactivated
UserDeleted

Y cuando corresponda:

EmailChanged
PasswordChanged
SecuritySettingsChanged
16. Organization Audit

Operaciones relevantes:

OrganizationCreated
OrganizationUpdated
OrganizationSuspended
OrganizationReactivated
OrganizationSecurityPolicyChanged

Los cambios de configuración de seguridad de una organización deben tener una trazabilidad reforzada.

17. Membership Audit

Deben registrarse cambios importantes en memberships:

MembershipCreated
MembershipActivated
MembershipSuspended
MembershipRemoved
MembershipRoleChanged

Esto permite reconstruir:

User
 ↓
Organization
 ↓
Membership
 ↓
Role
 ↓
Permission

durante una investigación.

18. Role Audit

Operaciones:

RoleCreated
RoleUpdated
RoleDeleted
RoleAssigned
RoleRemoved

Especial atención a:

AdminRoleGranted
AdminRoleRevoked

Estas operaciones deben clasificarse como altamente sensibles.

19. Permission Audit

Operaciones:

PermissionCreated
PermissionUpdated
PermissionAssigned
PermissionRevoked

Ejemplo:

Administrator
      ↓
Permission Granted
      ↓
AuditEvent

La evidencia debe permitir identificar:

quién otorgó el permiso;
a quién;
qué permiso;
en qué tenant;
cuándo;
bajo qué contexto;
cuál era el resultado.
20. Delegation Audit

Como el modelo de autorización ahora incorpora Delegation, deben auditarse:

DelegationCreated
DelegationModified
DelegationRevoked
DelegationExpired
DelegationUsed

Especialmente:

CrossTenantDelegation
PrivilegeBoundaryViolation
DelegationRevocation

La auditoría debe conservar suficiente información para demostrar que:

Delegated Capability
≤
Delegator Effective Capability
21. MFA Audit

Basado en ESP-0001/07:

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

Las operaciones relacionadas con:

MFA Disable
MFA Reset
Factor Replacement
Recovery

requieren trazabilidad reforzada.

22. Session Audit

Eventos relevantes:

SessionCreated
SessionAuthenticated
SessionRevoked
SessionExpired
Logout
GlobalLogout

Nunca deben almacenarse tokens completos.

Debe utilizarse una referencia segura:

session_id

o un identificador equivalente.

23. Token Audit

Eventos relevantes:

RefreshTokenRotated
RefreshTokenRevoked
TokenReuseDetected
TokenValidationFailed

Nunca:

access_token
refresh_token
api_secret
private_key

deben almacenarse como valores completos en Audit.

24. Refresh Token Reuse Audit

Un evento de reuse detection debe poder reconstruir:

Session
Token Family
Affected Actor
Tenant
Timestamp
Detection Result
Security Response

Por ejemplo:

Refresh Token Reuse Detected
        ↓
Security Event
        ↓
Audit Event
        ↓
Family Revoked
        ↓
Session Revoked

La auditoría debe registrar el hecho y la respuesta, nunca el secreto.

25. Security Policy Audit

Los cambios de políticas deben ser auditables:

SecurityPolicyCreated
SecurityPolicyUpdated
SecurityPolicyActivated
SecurityPolicySuperseded
SecurityPolicyRetired

Debe poder determinarse:

Which policy?
Which version?
Who changed it?
When?
Why?
What scope?
What result?
26. Administrative Audit

Las operaciones administrativas deben tener mayor nivel de trazabilidad.

Ejemplos:

GrantAdminRole
RevokeAdminRole
DisableUser
UnlockUser
ResetUserMFA
ModifyPermission
ModifySecurityPolicy
ChangeOrganizationSecurity
CrossTenantAdministrativeAction

La auditoría debe incluir el actor administrativo y el target afectado.

27. Before / After State

Para determinadas operaciones de modificación puede registrarse:

before
after

Ejemplo:

Role

BEFORE:
Viewer

AFTER:
Coach

Pero esto debe aplicarse con Data Minimization.

Nunca deben registrarse:

password
password_hash
TOTP_secret
private_key
refresh_token
access_token
recovery_code
28. Sensitive Data Protection

Audit Logs pueden contener información altamente sensible.

Por ello deben tratarse como información protegida.

Nunca deben contener:

credenciales;
tokens;
claves privadas;
MFA secrets;
recovery codes;
secretos de proveedores;
información de salud innecesaria;
datos personales que no sean necesarios para el propósito de auditoría.
29. Privacy by Design

Audit & Compliance debe aplicar:

Data Minimization
Purpose Limitation
Access Control
Retention Control
Tenant Isolation
Protected Storage
Controlled Export

La auditoría no debe convertirse en una copia completa de todos los datos del sistema.

30. Audit Retention

La arquitectura de EVOXA contempla como referencia:

Audit Logs Retention:
7 years

Pero este valor debe interpretarse como objetivo arquitectónico, no como una obligación universal.

La retención definitiva dependerá de:

regulación;
jurisdicción;
organización;
tipo de información;
contrato;
política de seguridad;
obligaciones legales.
31. Immutability

Los Audit Events críticos deben tratarse como registros append-only.

Conceptualmente:

Application
     ↓
Audit Event
     ↓
Append-Only Store
     ↓
Long-Term Storage

No debe existir una operación administrativa normal equivalente a:

UPDATE audit_event

o:

DELETE audit_event

para alterar evidencia histórica.

32. Tamper Protection

La arquitectura debe permitir evolucionar hacia mecanismos de protección contra manipulación.

Posibles mecanismos:

Hash Chaining
Digital Signatures
Write-Once Storage
Immutable Storage
External Archive

Sin embargo, no se debe declarar ninguno de ellos como implementación obligatoria todavía, porque el mecanismo físico concreto no quedó cerrado por los ADR existentes.

33. Audit Integrity

Cuando el mecanismo de integridad lo requiera, puede existir conceptualmente:

event_hash
previous_event_hash

Ejemplo:

Event A
   ↓
Hash A
   ↓
Event B
   ↓
Hash B
   ↓
Event C

Esto permite detectar alteraciones posteriores.

El algoritmo y mecanismo definitivo deben quedar definidos en una decisión de implementación/seguridad específica si fuera necesario.

34. Correlation

Una operación relevante debe poder correlacionarse a través de los diferentes subsistemas.

Modelo:

HTTP Request
      ↓
correlation_id
trace_id
span_id
      ↓
Authentication
      ↓
Authorization
      ↓
Business Operation
      ↓
Audit Event
      ↓
Domain / Integration Event

Esto permite reconstruir una operación completa durante una investigación.

35. Causation

Cuando una acción provoca otra, debe poder conservarse:

causation_id

Ejemplo:

Admin disables user
       ↓
UserDisabled
       ↓
SessionsRevoked
       ↓
RefreshFamiliesRevoked

Los eventos derivados pueden mantener la relación causal correspondiente.

36. Audit vs Domain Events

No deben confundirse:

Domain Event

y:

Audit Event
Domain Event

Representa:

Algo importante ocurrió en el dominio.

Audit Event

Representa:

Quién realizó una operación y qué ocurrió desde la perspectiva de seguridad/accountability.

Pueden estar relacionados:

UserRoleChanged
      ├── Domain Event
      └── Audit Event

pero no necesariamente deben ser el mismo objeto.

37. Audit vs Security Events

También debe existir separación conceptual entre:

Audit Event
Security Event
Application Log
Domain Event
Audit Event

Accountability y trazabilidad.

Security Event

Detección/respuesta ante condiciones de seguridad.

Application Log

Diagnóstico técnico.

Domain Event

Cambio significativo del dominio.

Esta separación evita mezclar responsabilidades.

38. Event Architecture Integration

Audit debe integrarse con el Event Architecture de EVOXA.

Los eventos deben respetar:

Immutable
Versioned
At-Least-Once
Idempotent Consumers
Retry
DLQ
Replay
Tenant Aware
Correlation
Causation

Los eventos no deben contener secretos.

39. Transactional Outbox

Cuando una operación de negocio produzca un hecho que debe ser auditado/publicado, la persistencia crítica debe utilizar el patrón:

Domain State Change
        +
Audit / Outbox State
        ↓
Same Transaction
        ↓
PostgreSQL
        ↓
Outbox Publisher
        ↓
Event Platform

Esto evita inconsistencias como:

Business change = committed
Audit/Event     = lost
40. Audit Storage

La arquitectura conceptual puede utilizar:

PostgreSQL
    ↓
Operational Audit

Event Platform
    ↓
Event Distribution / Processing

Immutable / Long-Term Storage
    ↓
Compliance Archive

PostgreSQL continúa siendo la fuente transaccional primaria.

La arquitectura exacta de almacenamiento de largo plazo queda pendiente de implementación/operación.

41. Audit Query

Los usuarios autorizados podrán consultar auditoría.

Conceptualmente:

GET /api/v1/identity/audit

Filtros potenciales:

actor
tenant
organization
user
event_type
action
result
target
date_from
date_to
correlation_id

Ejemplo:

GET /api/v1/identity/audit
    ?event_type=ROLE_CHANGED
    &organization_id=...

El contrato definitivo pertenece a ESP-0001/10.

42. Audit Access Control

La auditoría también necesita autorización.

No todos los usuarios pueden consultar Audit Events.

El flujo conceptual:

User
 ↓
Authentication
 ↓
Membership
 ↓
Role
 ↓
Permission
 ↓
Authorization Policy
 ↓
Audit Access

Y debe aplicarse:

Tenant Isolation
+
Ownership where applicable
+
Security Policy
+
Criticality
43. Platform vs Organization Audit

Debe existir una separación conceptual entre:

Platform Audit

Acciones de administradores globales.

Organization Audit

Acciones realizadas dentro de un tenant.

Ejemplo:

Platform
 └── Platform Administration

Organization A
 └── Organization Audit

Organization B
 └── Organization Audit

Un tenant no debe acceder a los registros de otro tenant.

44. Cross-Tenant Audit

Las operaciones cross-tenant son particularmente sensibles.

Cuando estén permitidas deben conservar:

Source Tenant
Target Tenant
Actor
Permission
Policy
Authorization Decision
Criticality
Timestamp
Reason
Correlation

Esto permite demostrar posteriormente por qué se permitió una operación excepcional.

45. Compliance Evidence

Audit & Compliance debe permitir producir evidencia de preguntas como:

Who accessed?
Who changed permissions?
Who granted admin privileges?
Who disabled the user?
When was MFA enabled?
When was MFA removed?
Who modified the security policy?
Which sessions were revoked?
Which authorization decision was made?
Which policy version was active?

Esto convierte Audit en una fuente de evidencia operacional.

46. Policy / Authorization Evidence

Para operaciones críticas debe ser posible reconstruir conceptualmente:

Actor
 ↓
Authentication Context
 ↓
Tenant
 ↓
Membership
 ↓
Roles
 ↓
Permissions
 ↓
Policy Version
 ↓
Authorization Decision
 ↓
Criticality
 ↓
Freshness
 ↓
Operation
 ↓
Audit

Esto es especialmente importante para investigaciones posteriores.

47. Test Evidence Integration

Los ADR-062 → 067 introducen un concepto adicional:

Test Evidence

No debe confundirse con Audit Event.

Test Evidence demuestra que una política/runtime fue probado.

Puede contener:

execution_id
scenario_id
policy_version
runtime_version
catalog_version
scenario_version
invariant_version
generator_version
input_hash
expected_hash
actual_hash
result
timestamp
integrity_reference

Audit demuestra qué ocurrió.

Test Evidence demuestra qué fue validado.

48. Security Invariant Evidence

Cuando se ejecuten invariantes de seguridad, la evidencia debe permitir demostrar resultados como:

Tenant Isolation
Platform Minimum
No Privilege Amplification
Revocation Priority
Criticality Monotonicity
Freshness Protection
No Unsafe ALLOW
AI Cannot Reduce Security

La evidencia debe ser reproducible mediante las versiones correspondientes.

49. Compliance Frameworks

La arquitectura de EVOXA contempla compatibilidad con marcos como:

GDPR
HIPAA
ISO 27001
SOC 2
Local Regulations

Pero:

EVOXA no es automáticamente compliant simplemente por implementar esta arquitectura.

El cumplimiento depende también de:

configuración;
despliegue;
procesos;
controles operacionales;
gestión de datos;
ubicación de datos;
contratos;
obligaciones regulatorias.

La especificación original establece explícitamente esta distinción.

50. Privacy / Data Subject Operations

Cuando corresponda, deben poder auditarse operaciones relacionadas con:

Data Access
Data Export
Data Rectification
Data Deletion
Data Retention
Consent

La auditoría debe registrar la operación sin convertirse en una copia completa de los datos personales afectados.

51. Audit Export

Los usuarios autorizados pueden necesitar exportar auditoría.

Formatos potenciales:

JSON
CSV
PDF

La exportación debe:

estar autorizada;
aplicar tenant isolation;
aplicar filtros;
proteger información sensible;
ser trazable;
generar su propia auditoría.

Flujo:

Audit Export Request
        ↓
Authorization
        ↓
Policy Evaluation
        ↓
Generate Export
        ↓
Audit Export Completed
52. Audit Export Security

Una exportación de auditoría debe tratarse como operación sensible.

Debe evaluarse mediante:

RBAC
+
ABAC
+
Tenant Isolation
+
Security Policy
+
Criticality
+
Freshness

Dependiendo de su alcance puede requerir:

REAUTHENTICATE

o:

REQUIRE_MFA
53. Audit Monitoring

Observability puede detectar patrones como:

Repeated Authentication Failures
Repeated Authorization Denials
Privilege Escalation
Mass Role Changes
Mass Permission Changes
Unexpected MFA Disable
Multiple Token Reuse Events
Cross-Tenant Attempts
Repeated Recovery Attempts
Suspicious Administrative Activity

AI puede asistir en detección de anomalías, pero no puede modificar la evidencia ni cambiar una decisión de autorización.

54. Security Event Severity

Security Events pueden clasificarse conceptualmente:

INFO
LOW
MEDIUM
HIGH
CRITICAL

La severidad no reemplaza:

Authorization Decision
Criticality
Audit Classification

Son dimensiones diferentes.

55. AI Auditability

Toda acción realizada por un AI Agent debe ser identificable como:

actor.type = AI_AGENT

cuando corresponda.

Debe ser posible responder:

Which AI Agent?
Which Service Identity?
Which Tenant?
Which User Context?
Which Operation?
Which Policy?
Which Decision?
Which Result?

La IA no debe poder eliminar o modificar su propia evidencia de auditoría.

56. AI Security Restrictions

AI Agents no pueden:

eliminar Audit Events;
alterar evidencia;
ocultar operaciones;
modificar Security Policy sin autorización;
reducir Criticality;
saltarse MFA;
manipular cache;
realizar cross-tenant access implícito;
aprobar Security Regression;
modificar Security Epoch para obtener autorización.
57. Audit Integrity During Failure

Un fallo de infraestructura no debe provocar silenciosamente la pérdida de evidencia crítica.

Deben utilizarse mecanismos como:

Transactional Outbox
Retry
At-Least-Once Delivery
Idempotent Consumers
DLQ
Replay
Reconciliation

El detalle operativo queda en Event Architecture y Deployment.

58. Audit Recovery

La recuperación debe permitir determinar:

What was committed?
What was published?
What was delivered?
What was duplicated?
What was delayed?
What was in DLQ?

Los eventos deben poder reprocesarse de forma idempotente.

El replay no debe crear duplicados de resultados de negocio.

59. Audit Access Logging

Consultar auditoría también puede ser una operación auditable.

Ejemplo:

Administrator
      ↓
Queries Audit Logs
      ↓
AuditAccessed

Esto evita que el mecanismo de auditoría se convierta en una zona sin trazabilidad.

60. Audit Retention vs Legal Deletion

La retención debe equilibrarse con:

Security
Compliance
Privacy
Legal Requirements
Data Minimization

Una política de retención no debe asumirse universalmente.

Cuando exista una obligación legal de eliminación o anonimización, el comportamiento debe estar definido por la política de privacidad/compliance correspondiente.

61. Audit Immutability vs Privacy

Debe reconocerse una tensión arquitectónica entre:

Audit Integrity

y:

Privacy / Data Subject Rights

La solución no debe ser simplemente modificar registros históricos.

Debe utilizar mecanismos controlados de:

Minimization
Pseudonymization
Redaction where legally required
Access Restriction
Retention Policies

manteniendo la integridad de la evidencia cuando sea legalmente necesario.

62. Observability Integration

Audit debe integrarse con:

Logs
Metrics
Traces
Security Events
Audit Events

pero estos sistemas siguen siendo conceptualmente diferentes.

Los identificadores comunes incluyen:

correlation_id
trace_id
span_id
causation_id
63. Audit Query Security

Las consultas deben estar protegidas contra:

cross-tenant access;
excessive data exposure;
unauthorized export;
enumeration;
privilege escalation;
injection;
uncontrolled bulk extraction.

Los filtros y ordenamientos deben estar controlados por el API contract.

64. Audit API Boundary

Los endpoints definitivos serán definidos en:

ESP-0001/10 — Identity API Contracts

La ESP-09 únicamente establece las capacidades.

Conceptualmente:

GET /api/v1/identity/audit
GET /api/v1/identity/audit/{id}
POST /api/v1/identity/audit/export

El API final debe aplicar:

Authentication
Authorization
Tenant Isolation
Rate Limiting
Pagination
Filtering
Audit Access Logging
65. Persistence Boundary

La estructura física de persistencia será definida en:

ESP-0001/11 — Identity Persistence & Data Model

Conceptualmente:

audit_events
security_events
outbox_events

y las estructuras auxiliares necesarias.

La implementación física de almacenamiento inmutable de largo plazo permanece pendiente.

66. Event Boundary

La definición final de:

event_type
event_version
payload
metadata
partitioning
ordering
retention
replay

pertenece a:

ESP-0001/12 — Identity Events & Integration

Esta ESP define únicamente las necesidades de Audit.

67. Testing

Audit & Compliance debe ser probado mediante:

Unit Tests
event construction;
classification;
minimization;
tenant context;
result mapping.
Integration Tests
PostgreSQL;
Outbox;
Event Platform;
audit persistence.
Security Tests
cross-tenant access;
unauthorized audit access;
privilege escalation;
tampering;
secret leakage.
E2E
Operation
 ↓
Authorization
 ↓
Business Action
 ↓
Audit
 ↓
Event
68. Audit Security Tests

Como mínimo deben validarse:

Password never logged
Token never logged
MFA secret never logged
Recovery code never logged
Private key never logged
Cross-tenant audit denied
Unauthorized audit query denied
Audit modification denied
Critical event preserved
Correlation preserved
Tenant context preserved
69. Golden Scenarios

El modelo de testing puede incorporar escenarios como:

GSC-AUDIT-001
Successful Login Audited

GSC-AUDIT-002
Failed Login Audited

GSC-AUDIT-003
Admin Role Grant Audited

GSC-AUDIT-004
MFA Disable Audited

GSC-AUDIT-005
Cross-Tenant Denial Audited

GSC-AUDIT-006
Security Policy Change Audited

GSC-AUDIT-007
Refresh Token Reuse Audited

GSC-AUDIT-008
Critical Authorization Audited

GSC-AUDIT-009
AI Agent Action Audited

GSC-AUDIT-010
Unauthorized Audit Access Denied

Los IDs definitivos pertenecerán al Test Catalog.

70. Security Invariants

Audit debe respetar invariantes como:

INV-AUDIT-001
Audit events are tenant-aware.

INV-AUDIT-002
Audit events cannot contain secrets.

INV-AUDIT-003
Critical security operations are auditable.

INV-AUDIT-004
Audit records cannot be modified through normal administrative operations.

INV-AUDIT-005
Audit access requires authorization.

INV-AUDIT-006
Cross-tenant audit access is denied by default.

INV-AUDIT-007
AI actions remain auditable.

INV-AUDIT-008
Audit correlation identifiers remain consistent.

INV-AUDIT-009
Audit replay is idempotent.

INV-AUDIT-010
Audit evidence does not create privilege escalation.
71. Evidence Integrity

La evidencia generada por Audit/Test debe poder relacionarse con:

Policy Version
Runtime Version
Schema Version
Catalog Version
Scenario Version
Invariant Version

Esto permite reconstruir posteriormente el contexto en que se produjo una decisión o validación.

72. Historical Reproducibility

Una investigación histórica debe poder determinar, cuando la información esté disponible:

Which Policy?
Which Policy Version?
Which Runtime?
Which Actor?
Which Tenant?
Which Membership?
Which Session?
Which Security State?
Which Criticality?
Which Freshness?
Which Decision?
Which Event?

Esto es especialmente importante después de cambios de Security Policy.

73. Compliance Reporting

El sistema podrá evolucionar hacia reportes como:

Authentication Report
Authorization Report
Privilege Change Report
MFA Report
Session Security Report
Security Policy Report
Administrative Activity Report
Cross-Tenant Activity Report
Security Incident Evidence

La generación de estos reportes debe utilizar las APIs/servicios de Audit y no acceder directamente a tablas internas desde otros dominios.

74. Business Domain Boundary

Identity Audit no debe convertirse en un sistema de auditoría universal de toda la aplicación.

Identity debe auditar principalmente:

Identity
Authentication
Authorization
Security
Access
Membership
Roles
Permissions
MFA
Sessions
Tokens
Policies
Recovery

Los dominios de negocio deben ser responsables de sus propios eventos de negocio y auditoría específica.

75. Module Boundary

Otros dominios no deben acceder directamente a:

audit_events
security_events

para implementar lógica de negocio.

La interacción debe utilizar:

Public Contracts
Domain Events
Integration Events
Audit Contracts

según corresponda.

76. Audit Service

El diseño puede exponer conceptualmente un:

AuditService

responsable de:

Create Audit Event
Validate Audit Context
Apply Data Minimization
Persist Audit
Publish / Outbox
Correlate

Pero no debe permitir que otros módulos escriban directamente en el repositorio físico.

77. Security Event Service

Debe existir una separación conceptual entre:

AuditService

y:

SecurityEventService

aunque inicialmente puedan compartir infraestructura.

Esto permite que:

Audit

represente accountability mientras:

Security Events

representa detección/respuesta de seguridad.

78. Compliance Architecture

La arquitectura consolidada queda:

                 Identity Operation
                        │
                        ▼
              Authentication / AuthZ
                        │
                        ▼
                Security Decision
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Audit         Security       Trace
       Event           Event        Context
          │             │             │
          └─────────────┼─────────────┘
                        ▼
                  Outbox / Events
                        │
                        ▼
              Compliance Evidence
79. Non-Negotiables

Los siguientes principios son obligatorios:

Audit ≠ Application Logging.
Security Events ≠ Audit Events.
Domain Events ≠ Audit Events.
Tenant context obligatorio cuando corresponda.
Audit Access requiere autorización.
Cross-tenant audit denied by default.
No secrets in audit.
Critical security actions deben ser auditables.
Audit records deben ser append-only donde corresponda.
No modificación normal de evidencia histórica.
Correlation debe preservarse.
Causation debe preservarse cuando corresponda.
Outbox para hechos transaccionales que deban publicarse.
At-least-once + idempotencia.
Replay seguro.
AI actions deben ser trazables.
Criticality debe poder preservarse en evidencia cuando sea relevante.
Freshness debe poder preservarse en evidencia cuando sea relevante.
Policy version debe conservarse para decisiones relevantes.
Audit nunca debe utilizarse para otorgar autorización.
Audit nunca debe convertirse en bypass de Tenant Isolation.
Privacy by Design.
Data Minimization.
Fail Closed para acceso a auditoría.
80. Traceability
Documento	Integración
ESP-0001/01	Identity Domain Model
ESP-0001/03	Authentication & Authorization
ESP-0001/05	Roles & Permissions
ESP-0001/06	Sessions & Tokens
ESP-0001/07	MFA
ESP-0001/08	Security Policies
ESP-0001/09	Audit & Compliance
ESP-0001/10	API Contracts
ESP-0001/11	Persistence
ESP-0001/12	Events
ESP-0001/13	Testing
ESP-0001/14	Implementation
ESP-0001/15	Deployment
ADR principales
ADR-007   Transactional Outbox
ADR-012   PostgreSQL Schema
ADR-013   Data Retention
ADR-014   Encryption & Secret Protection
ADR-018   Central Event Broker
ADR-019   Event Streaming / Routing
ADR-020   Event Schema Registry
ADR-021   Controlled Replay
ADR-022   Consumer Idempotency
ADR-028   Observability & Operational Security
ADR-029   Disaster Recovery & Backup
ADR-030   DR Testing
ADR-042   Attribute Ownership
ADR-045   Authorization Policy Model
ADR-049   Authorization Runtime
ADR-053   Cache Invalidation Events
ADR-054   Cache Failure
ADR-055   Cache Recovery
ADR-056   Freshness
ADR-057   Freshness SLAs
ADR-058   Operation Criticality
ADR-059   Dynamic Criticality
ADR-062   Test Catalog
ADR-063   Test Execution
ADR-064   Test Evidence
ADR-065   Schema Versioning
ADR-066   Schema Registry
ADR-067   Runtime Contracts
81. Implementation Readiness
Capability	Estado
Audit Event Model	🟢
Actor / Target	🟢
Tenant Isolation	🟢
Authentication Audit	🟢
Authorization Audit	🟢
Role / Permission Audit	🟢
Membership Audit	🟢
MFA Audit	🟢
Session Audit	🟢
Token Audit	🟢
Delegation Audit	🟢
Security Policy Audit	🟢
Criticality Evidence	🟢
Freshness Evidence	🟢
AI Auditability	🟢
Correlation	🟢
Causation	🟢
Event Integration	🟢
Transactional Outbox	🟢
Idempotency / Replay	🟢
Privacy / Minimization	🟢
Audit Access Control	🟢
Test Evidence Integration	🟢
API	🟡 — ESP-10
Physical Persistence	🟡 — ESP-11
Event Contracts	🟡 — ESP-12
Test Implementation	🟡 — ESP-13
Source Implementation	🟡 — ESP-14
Operational Retention/Archive	🟡 — ESP-15
Immutable Long-Term Storage	🟡 — implementación pendiente
82. Conclusión

La ESP-0001/09 original está bastante bien estructurada y, a diferencia de ESP-0001/03 o ESP-0001/08, no necesita una reconstrucción conceptual completa. Ya contiene los elementos fundamentales de auditoría: AuditEvent, actor, target, tenant, resultados, autenticación, autorización, usuarios, organizaciones, memberships, roles, permisos, MFA, sesiones, tokens, políticas, privacidad, retención, correlación, causación y separación respecto del logging técnico.

Lo que necesitaba era subirla al nivel arquitectónico que alcanzamos con los ADR posteriores.

Ahora la cadena queda mucho más sólida:

Authentication
       ↓
Authorization
       ↓
Security Policy
       ↓
Criticality
       ↓
Freshness
       ↓
Authorization Decision
       ↓
Business Operation
       ↓
Audit Event
       ↓
Security Event
       ↓
Transactional Outbox
       ↓
Event Platform
       ↓
Compliance Evidence

Y especialmente queda cerrado un punto que será muy importante cuando comencemos a programar EVOXA:

Audit no será simplemente una tabla de logs.

Será una capacidad de seguridad y accountability, integrada con Identity, Authorization, Policy Engine, Event Architecture, Observability, Testing y Compliance.

Estado recomendado

ESP-0001/09 v1.1.0 → 🟢 Arquitectónicamente consolidada

No veo necesidad de crear otro ADR únicamente para completar esta ESP.

El siguiente documento natural sería:

ESP-0001/10 — Identity API Contracts

y ahí sí tendremos que hacer algo especialmente importante: congelar los contratos HTTP/OpenAPI reales que después utilizaremos para comenzar la implementación.
