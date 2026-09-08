ESP-0001 / 08 — Identity Security Policies

Estado: Draft — Consolidated
Dominio: Identity
Código: ESP-0001
Sección: 08 — Security Policies
Versión: 1.1.0
Parent Specification: ESP-0001 — Identity Engineering Specification
Target Repository: evoxa-platform
Target Path: apps/api/app/domains/identity/

1. Propósito

Este documento define el modelo de Security Policies de EVOXA Identity y establece cómo las políticas de seguridad deben ser:

definidas;
versionadas;
validadas;
evaluadas;
combinadas;
priorizadas;
aplicadas;
auditadas;
probadas;
cacheadas;
invalidadas;
recuperadas.

Security Policies constituye una capacidad transversal de Identity y participa en:

Authentication;
Authorization;
MFA;
Sessions;
Tokens;
Users;
Organizations;
Memberships;
Roles & Permissions;
Recovery;
Delegation;
Audit;
Security Events;
Authorization Criticality;
Authorization Freshness.

La especificación original ya establecía que Security Policies debía actuar transversalmente sobre estas capacidades.

2. Principios normativos

Security Policies debe respetar los siguientes principios.

2.1 Deny by Default

Cuando una operación no está explícitamente autorizada:

DENY

No debe existir una interpretación implícita de ausencia de política como ALLOW.

2.2 Fail Closed

Los siguientes estados nunca deben convertirse en un ALLOW inseguro:

error de evaluación;
contexto incompleto;
tenant desconocido;
política inválida;
política no disponible;
error del Policy Engine;
error del Risk Engine;
estado MFA desconocido;
estado de sesión desconocido;
error de cache;
error de infraestructura relevante.

El principio ya estaba definido en la especificación original y ahora queda formalizado mediante el runtime de autorización de los ADR posteriores.

2.3 Least Privilege

Las políticas nunca deben conceder capacidades superiores a las necesarias.

La autorización efectiva resulta de combinar:

Identity
   ↓
Membership
   ↓
Roles
   ↓
Permissions
   ↓
Ownership / ABAC
   ↓
Delegation
   ↓
Security Policies
   ↓
Security Context
2.4 Zero Trust

Una identidad autenticada no implica autorización automática.

Tampoco debe asumirse confianza solamente por:

pertenecer a una organización;
utilizar un dispositivo conocido;
provenir de una IP determinada;
haber iniciado sesión previamente;
tener un rol administrativo.

Cada operación debe evaluarse según su contexto.

3. Security Policy Model

La política se define conceptualmente mediante:

SecurityPolicy
-------------------------
id
name
version
status
scope
priority
effect
target
conditions
obligations
effective_from
effective_until
metadata
created_at
updated_at

Este modelo sustituye la representación antigua basada simplemente en rules por una estructura declarativa y tipada.

4. Policy Scope

Las políticas pueden existir en diferentes niveles.

PLATFORM

Políticas globales de EVOXA.

ORGANIZATION

Políticas específicas de una organización.

RESOURCE

Políticas específicas de determinados recursos.

Los niveles de usuario y rol pueden participar como targets/conditions/contexto, pero no deben interpretarse como una jerarquía de autoridad independiente.

5. Policy Lifecycle

El lifecycle normativo será:

DRAFT
   ↓
VALIDATING
   ↓
APPROVED
   ↓
ACTIVE
   ↓
SUPERSEDED
   ↓
RETIRED

Una política:

DRAFT no debe utilizarse para autorización productiva.
VALIDATING está siendo validada.
APPROVED está aprobada pero aún no necesariamente activa.
ACTIVE puede participar en decisiones.
SUPERSEDED fue reemplazada.
RETIRED dejó de ser aplicable.

Una política histórica debe conservarse para permitir reproducibilidad y auditoría.

6. Policy Versioning

Cada versión de política debe ser identificable.

Ejemplo:

Policy:
  password-security

Version:
  3

Status:
  ACTIVE

Una decisión histórica debe poder identificar:

policy_id
policy_version
runtime_version
evaluation_timestamp

Esto permite reproducir posteriormente por qué una decisión fue tomada.

7. Policy Language

EVOXA utilizará un modelo declarativo, estructurado y tipado.

Las políticas no deben permitir:

código arbitrario;
Python;
SQL;
llamadas HTTP;
acceso directo a Redis;
acceso directo a PostgreSQL;
ejecución de comandos;
acceso a secretos;
efectos secundarios arbitrarios.

La política expresa qué debe evaluarse, no cómo ejecutar código.

8. Policy Conditions

Las condiciones podrán combinarse mediante expresiones estructuradas.

Conceptualmente:

AND
OR
NOT

Y referencias a:

subject.*
resource.*
organization.*
membership.*
session.*
request.*
environment.*
delegation.*

El catálogo definitivo de operadores y funciones permanece como detalle de implementación del DSL y no debe inventarse en esta ESP.

9. Policy Effects

Las políticas pueden producir:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

Esto mantiene el modelo original, que ya contemplaba decisiones más allá de ALLOW/DENY.

10. Policy Evaluation Context

El Policy Engine debe recibir un contexto explícito.

Conceptualmente:

AuthorizationContext
--------------------
actor
organization
membership
roles
permissions
request
resource
operation
session
authentication
mfa
ownership
delegation
risk
security_state
environment
timestamp
correlation

No todos los campos tienen que existir en todas las evaluaciones.

El motor debe utilizar únicamente los atributos necesarios para la política aplicable.

11. Authorization Policy Evaluation

Security Policy no debe evaluarse como una función aislada.

El runtime normativo será:

Authorization Request
        ↓
Request Validation
        ↓
Authentication Validation
        ↓
Tenant Guard
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
Authorization Decision

Este es el modelo consolidado de los ADR-045–059 y reemplaza el flujo simplificado de la ESP original.

12. Policy Evaluation Result

El Policy Engine debe distinguir:

MATCH
NO_MATCH
INDETERMINATE
ERROR

Regla fundamental:

INDETERMINATE ≠ ALLOW
ERROR ≠ ALLOW

Un error o incertidumbre no puede transformarse en autorización positiva.

13. Policy Precedence

La precedencia normativa será:

Regulatory / Mandatory Security
            ↓
Platform Security Minimum
            ↓
Security State
            ↓
Tenant Isolation
            ↓
Resource Security Policy
            ↓
Dynamic Risk
            ↓
Contextual Policy
            ↓
Organization Policy
            ↓
Default Operation Rules

La precedencia determina qué regla tiene autoridad.

No debe confundirse con la fórmula de criticidad.

14. Conflict Resolution

Cuando existen efectos incompatibles en el mismo nivel:

DENY > ALLOW

Pero no se trata de un simple modelo global de "deny wins".

La decisión debe considerar:

precedencia;
nivel de seguridad;
mandatory security guards;
platform minimums;
security state;
tenant isolation;
criticality;
freshness.
15. Platform Security Minimums

Una organización puede endurecer las políticas de plataforma.

No puede debilitarlas.

Ejemplo:

PLATFORM:
  Admin → REQUIRE_MFA

Una organización no puede definir:

ORGANIZATION:
  Admin → MFA optional

para neutralizar la política de plataforma.

Esto queda reforzado por el modelo de Platform Minimum definido en los ADR de Policy y Criticality.

16. Authentication Policies

Security Policies puede gobernar:

autenticación;
credenciales;
account state;
MFA;
intentos fallidos;
step-up;
risk;
creación de sesión;
recuperación.

Ejemplo conceptual:

IF account.status != ACTIVE
THEN DENY
17. Password Policies

Las políticas pueden definir conceptualmente:

minimum_length
maximum_length
complexity
history
expiration
reuse_prevention
breach_detection

El almacenamiento seguirá utilizando:

Argon2id

Los parámetros criptográficos exactos no deben fijarse arbitrariamente dentro de esta ESP.

18. Password History

La reutilización de contraseñas puede ser restringida mediante política.

Nunca deben almacenarse contraseñas anteriores en texto plano.

La comparación debe realizarse contra representaciones protegidas.

19. Password Reset

Password Reset es una operación de seguridad.

Flujo:

Reset Request
      ↓
Identity Verification
      ↓
Reset Challenge
      ↓
New Credential
      ↓
Security State Update
      ↓
Session / Token Policy
      ↓
Audit
      ↓
Security Event

La política puede determinar si deben revocarse sesiones y tokens existentes.

20. Account Security Policies

Estados principales:

PENDING
ACTIVE
SUSPENDED
DISABLED

Security Policy debe considerar el estado de cuenta antes de permitir operaciones.

Ejemplo:

SUSPENDED → DENY
DISABLED  → DENY

Una transición de seguridad crítica puede provocar:

Revoke Sessions
Revoke Refresh Tokens
Invalidate MFA Challenges
Invalidate Authorization Cache
Increment Security Epoch

según las políticas correspondientes.

21. Rate Limiting Policies

Security Policies puede definir controles sobre:

Login;
MFA;
Password Reset;
Recovery;
MFA Enrollment;
Token Refresh;
API operations;
Administrative operations.

El rate limiting puede considerar múltiples dimensiones:

IP
Actor
Organization
Endpoint
Operation
Session
Device
Risk

No debe depender exclusivamente de IP.

22. MFA Policies

Security Policy Engine determina cuándo debe exigirse MFA.

MFA se encarga de cómo verificarlo.

Ejemplo:

IF role == ADMIN
THEN REQUIRE_MFA

Otro:

IF operation == CHANGE_SECURITY_SETTINGS
THEN REQUIRE_MFA

Otro:

IF risk == HIGH
THEN REQUIRE_MFA

Esto mantiene la separación definida en ESP-0001/07.

23. Step-Up Authentication

Las políticas pueden requerir una elevación temporal de autenticación.

Authenticated Session
        ↓
Sensitive Operation
        ↓
Policy Evaluation
        ↓
REQUIRE_MFA
        ↓
MFA Challenge
        ↓
MFA Verified
        ↓
Temporary Security Evidence
        ↓
Operation

La evidencia de MFA debe tener una validez limitada.

24. Session Policies

Las políticas pueden controlar:

maximum session lifetime;
inactivity timeout;
concurrent sessions;
high-risk sessions;
reauthentication;
revocation;
logout-all;
security reset.

Los valores concretos de TTL y límites son parámetros de implementación y operación, no deben inventarse en esta ESP.

25. Token Policies

Security Policies debe integrarse con:

Access Tokens
Refresh Tokens
API Tokens
Service Credentials

Puede gobernar conceptualmente:

Lifetime
Rotation
Revocation
Reuse Detection
Audience
Issuer
Scopes

Pero debe respetar las decisiones ya establecidas en los ADR correspondientes.

26. Refresh Token Reuse

Un refresh token utilizado previamente no debe volver a utilizarse.

Flujo:

Refresh Token A
       ↓
     USED
       ↓
Refresh Token B

Si aparece nuevamente:

Refresh Token A
       ↓
REUSE DETECTED
       ↓
Security Event
       ↓
Revoke Family / Session

La rotación y detección pertenecen al Token/Session subsystem; Security Policy determina las condiciones de seguridad que pueden acompañar la respuesta.

27. Organization Security Policies

Las organizaciones pueden definir políticas más estrictas.

Ejemplo:

Organization A
 ├── MFA required
 ├── Strong authentication
 ├── Short sessions
 └── Restricted administration

Mientras otra organización puede utilizar valores menos restrictivos siempre que no viole los mínimos de plataforma.

28. Tenant Isolation

Tenant isolation es una protección obligatoria.

Una solicitud:

Tenant A
User A
   ↓
Resource B
Tenant B

debe producir:

DENY

salvo que exista una capacidad cross-tenant explícitamente definida, autorizada y auditada.

29. Cross-Tenant Policies

El acceso cross-tenant debe requerir explícitamente:

Explicit Permission
+
Explicit Policy
+
Tenant Context
+
Business Authorization
+
Audit

Tener un rol administrativo no debe generar automáticamente privilegios cross-tenant.

La incertidumbre sobre el tenant debe producir DENY.

30. Device Policies

Las políticas pueden utilizar información como:

Known Device
New Device
Trusted Device
Revoked Device

Ejemplo:

IF device.status == NEW
THEN REQUIRE_MFA

El estado de un dispositivo no sustituye una autenticación fuerte.

31. Network / IP Policies

Las políticas pueden utilizar:

IP;
región;
red corporativa;
VPN;
contexto de red.

Ejemplo:

IF operation == ADMIN_ACCESS
AND network.trust == UNKNOWN
THEN REQUIRE_MFA

Estos controles son complementarios y no deben convertirse en una única frontera de seguridad.

32. Risk-Based Policies

Security Policy Engine puede consumir señales de riesgo.

Ejemplos:

New Device
Unusual Location
Failed Logins
Token Anomaly
Suspicious Behavior

Conceptualmente:

LOW
MEDIUM
HIGH
CRITICAL

El Risk Engine proporciona contexto.

No puede decidir directamente:

ALLOW

ni reducir las garantías de seguridad por sí mismo.

33. Criticality

Security Policies se integra con el modelo de Authorization Criticality.

La criticidad efectiva es:

MAX(
    Base Criticality,
    Dynamic Risk,
    Context Criticality,
    Security State,
    Tenant Criticality,
    Delegation Criticality,
    MFA Criticality,
    Platform Minimum
)

Niveles:

C0 — Critical Security
C1 — High Security
C2 — Standard
C3 — Low Risk

La criticidad no es una autorización.

Especifica el nivel de garantías de seguridad requeridas.

34. Freshness

La criticidad determina las garantías de frescura necesarias.

Modelo:

Criticality	Freshness
C0	F0
C1	F1
C2	F2-SHORT
C3	F2-MEDIUM

Por tanto:

Criticality
     ↓
Freshness Requirement
     ↓
Cache Eligibility
     ↓
Authorization Decision
35. Security Freshness

La frescura no puede depender únicamente del TTL.

También debe considerar:

Security Epoch;
Policy Version;
Entity Version;
Context Fingerprint;
Generation;
Invalidation State;
Operation Criticality.

Una revocación de seguridad debe invalidar una autorización aunque el TTL de cache todavía no haya expirado.

36. Authorization Cache

Security Policy Engine puede utilizar cache para optimizar evaluaciones.

Pero:

Cache ≠ Source of Truth

La autoridad permanece en PostgreSQL y en el estado de seguridad correspondiente.

La cache puede incluir:

L1
L2 / Redis

según la arquitectura de implementación.

37. Cache Eligibility

La elegibilidad de una decisión para cache debe determinarse después de establecer:

Criticality
+
Freshness

Debe considerar como mínimo:

tenant;
actor;
membership;
security epoch;
policy version;
entity version;
context fingerprint;
cache generation;
invalidation state;
TTL;
operation criticality;
freshness class.

Un C0 no puede depender simplemente de un ALLOW cacheado.

38. Cache Invalidation

El cambio de una política de seguridad debe propagarse mediante el modelo de eventos.

Flujo:

State Change
     ↓
PostgreSQL Transaction
     ↓
Transactional Outbox
     ↓
Event Platform
     ↓
Invalidation Consumer
     ↓
Security Epoch / Version
     ↓
L1 / L2 Invalidation

Los eventos representan hechos del dominio, no instrucciones específicas para Redis.

39. Security Epoch

El Security Epoch proporciona una frontera de seguridad superior al TTL.

Conceptualmente:

Cache Epoch < Current Security Epoch
          ↓
        STALE
          ↓
     Re-evaluate

Esto permite invalidar múltiples decisiones relacionadas sin depender de eliminar físicamente cada entrada.

40. Cache Failure

Si Redis o una cache secundaria falla:

Redis Failure
     ↓
Fresh Evaluation
     ↓
PostgreSQL / Authoritative State

No debe producir:

Redis Failure → ALLOW Everything

El sistema puede degradar rendimiento, pero no garantías de seguridad.

41. Cache Recovery

La recuperación sigue el modelo:

NORMAL
  ↓
DEGRADED
  ↓
RECONCILING
  ↓
INVALIDATING
  ↓
REBUILD
  ↓
WARM-UP
  ↓
RECOVERED
  ↓
NORMAL

Una cache recuperada no se considera automáticamente correcta.

Debe validarse contra el estado autoritativo.

42. Policy Governance

Toda política debe pasar por:

Draft
 ↓
Validation
 ↓
Approval
 ↓
Activation

Las políticas de seguridad crítica no deben poder activarse directamente por una IA.

Debe existir:

Change Request;
validación;
simulación;
pruebas;
aprobación;
auditoría;
despliegue controlado.
43. Policy Testing

Las políticas deben integrarse con el catálogo de pruebas definido en ADR-062–067.

Se contemplan:

Golden Scenarios;
Security Invariants;
Scenario Tests;
Generated Tests;
Property-Based Tests;
Boundary Tests;
Mutation Tests;
Differential Tests;
Performance Tests;
Chaos Tests;
Recovery Tests.
44. Security Invariants

Entre las invariantes fundamentales:

Platform Minimum Immutability
Security Monotonicity
Tenant Isolation
Unknown Is Not Safe Allow
Error Is Not Safe Allow
Revocation Priority
Security Epoch Supersedes TTL
Policy Version Integrity
Delegation Boundary
MFA Evidence Freshness
No Privilege Amplification
Organization Cannot Weaken Platform
Security State Dominance
Critical ALLOW Requires Freshness
No Version Regression
Determinism
AI Cannot Reduce Security
Criticality Does Not Grant Permission
Cache Is Not Source of Truth
45. Policy Runtime Contract

Security Policy Engine debe cumplir los contratos definidos en ADR-067:

Request Contract
Authorization Context Contract
Policy Evaluation Contract
Criticality Result Contract
Authorization Decision Contract
Freshness Contract
Cache Eligibility Contract
Test Execution Contract
Evidence Contract
Error Contract

Esto permite que producción y testing utilicen los mismos contratos normativos sin convertir el Test Runtime en el Production Runtime.

46. Audit

Las decisiones relevantes deben poder responder:

Who?
What?
When?
Where?
Why?
Which Policy?
Which Version?
Which Runtime?
Which Decision?
Which Criticality?
Which Freshness?

Ejemplo:

Actor:
user-123

Organization:
org-456

Operation:
change_mfa

Policy:
mfa-security

Policy Version:
3

Decision:
REQUIRE_MFA

Criticality:
C0

Freshness:
F0

Correlation ID:
...
47. Security Events

Pueden existir eventos como:

SecurityPolicyCreated
SecurityPolicyUpdated
SecurityPolicyActivated
SecurityPolicySuperseded
SecurityPolicyRetired
SecurityPolicyEvaluated
SecurityPolicyDenied
SecurityChallengeRequired
AuthenticationBlocked
AccountLocked
TokenReuseDetected
SecurityStateChanged

No todos necesariamente deben exponerse como eventos públicos de dominio.

Los eventos deben cumplir las reglas de Event Architecture:

immutable;
versioned;
at-least-once;
idempotent consumers;
replayable;
tenant-aware;
sin secretos.
48. API Boundary

Las APIs administrativas de políticas serán definidas formalmente en ESP-0001/10 — Identity API Contracts.

Conceptualmente:

GET    /api/v1/.../security-policies
POST   /api/v1/.../security-policies
GET    /api/v1/.../security-policies/{id}
PATCH  /api/v1/.../security-policies/{id}

No se recomienda exponer al cliente final un endpoint genérico que permita ejecutar arbitrariamente:

POST /security-policies/evaluate

La evaluación normalmente debe producirse internamente dentro de los flujos de Identity/Authorization.

49. Persistence Boundary

La persistencia será definida en:

ESP-0001/11 — Identity Persistence & Data Model

Conceptualmente se requiere soporte para:

security_policies
policy_versions
policy_metadata
policy_state
security_versions / epochs

El diseño físico definitivo queda fuera de esta ESP.

50. AI Governance

Los agentes de IA pueden:

sugerir políticas;
analizar políticas;
detectar conflictos;
generar escenarios;
explicar decisiones;
analizar anomalías;
proponer hardening.

Pero no pueden:

desactivar MFA;
reducir Criticality;
convertir DENY en ALLOW;
modificar Security Epoch;
manipular cache para obtener autorización;
aprobar una política de seguridad;
aprobar una Security Regression;
saltarse Tenant Isolation;
emitir privilegios superiores a los autorizados.

La IA utiliza exactamente el mismo modelo de autorización que cualquier otro actor.

51. Authorization Boundary

Security Policies no debe absorber reglas de negocio.

Identity

Es responsable de:

Who?
Which Organization?
Which Membership?
Which Roles?
Which Permissions?
Which Security Conditions?
Which Authentication Assurance?
Which Security Policy?
Business Domain

Es responsable de:

Is this business operation valid?
Is the resource in the correct state?
Does the business rule allow it?
Does domain ownership permit it?

Por tanto:

Identity Authorization
        +
Business Domain Authorization

son complementarias, no sustitutas.

52. Observability

Security Policy Engine debe generar métricas y trazabilidad sobre:

policy evaluations;
ALLOW;
DENY;
REQUIRE_MFA;
REAUTHENTICATE;
REVOKE;
policy evaluation latency;
cache hit/miss;
stale decisions;
invalidations;
policy version;
criticality;
freshness;
evaluation errors;
policy conflicts.

Nunca deben registrarse:

passwords;
MFA secrets;
recovery codes;
refresh tokens;
API secrets;
private keys;
secretos de proveedores.
53. Performance

El objetivo definido para el Authorization Runtime es:

Authorization P95 < 50 ms

cuando la evaluación corresponda al runtime de autorización y las condiciones operacionales permitan dicha medición.

La cache debe mejorar rendimiento sin degradar las garantías de seguridad.

54. Security Non-Negotiables

Security Policies no puede violar:

Deny by Default.
Fail Closed.
Tenant Isolation.
No Privilege Amplification.
Platform Minimums.
Security Monotonicity.
Revocation Priority.
Criticality/Freshness requirements.
Policy Version Integrity.
Cache not Source of Truth.
No arbitrary policy code.
No secrets in policies/events/logs.
No AI security bypass.
No cross-tenant access by implication.
No unsafe ALLOW on unknown/error.
55. Traceability
Blueprint / Specification	Integración
ESP-0001/01	Domain Model
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
ADR-045 → Authorization Policy Model
ADR-046 → Policy Language & Governance
ADR-047 → Policy Engine Implementation
ADR-048 → DSL / Expression Language
ADR-049 → Evaluation Runtime
ADR-050 → Cache Strategy
ADR-051 → Cache Keys & Fingerprinting
ADR-052 → Cache Eligibility / TTL
ADR-053 → Cache Invalidation
ADR-054 → Cache Failure / Degradation
ADR-055 → Cache Recovery
ADR-056 → Consistency / Freshness
ADR-057 → Freshness Classes / SLAs
ADR-058 → Criticality Matrix
ADR-059 → Criticality Policy
ADR-062 → Test Catalog
ADR-063 → Test Execution
ADR-064 → Test Data / Evidence
ADR-065 → Schema Versioning
ADR-066 → Schema Registry
ADR-067 → Runtime Contracts
56. Implementation Readiness
Capability	Estado
Security Policy Model	🟢
Policy Lifecycle	🟢
Policy Versioning	🟢
Policy Scope	🟢
Policy DSL principles	🟢
Policy Evaluation	🟢
Policy Precedence	🟢
Conflict Resolution	🟢
Tenant Isolation	🟢
MFA Integration	🟢
Session Integration	🟢
Token Integration	🟢
Criticality	🟢
Freshness	🟢
Cache Eligibility	🟢
Cache Invalidation	🟢
Cache Recovery	🟢
AI Governance	🟢
Audit	🟢
Testing Model	🟢
API Contract	🟡 — ESP-10
Persistence Schema	🟡 — ESP-11
Event Contracts	🟡 — ESP-12
Exact DSL Schema	🟡 — implementation detail
Exact Operators/Functions	🟡 — implementation detail
Runtime Implementation	🟡 — ESP-14
Operational Thresholds	🟡 — ESP-15
57. Conclusión

La ESP-0001/08 original estaba conceptualmente bien encaminada, pero había quedado en un nivel demasiado general y todavía trataba varias decisiones como futuras. Por ejemplo, ya definía scopes, contexto, decisiones, MFA, sesiones, tokens, tenants, riesgo, precedencia y versionado, pero dejaba explícitamente algunas combinaciones y decisiones para ADR posteriores.

Con la consolidación v1.1.0, Security Policies queda alineada con la arquitectura que construimos en los ADR-045 → ADR-067:

Security Policy
       ↓
Policy Engine
       ↓
Authorization Runtime
       ↓
Criticality
       ↓
Freshness
       ↓
Cache Eligibility
       ↓
Authorization Decision
       ↓
Audit + Security Events

Y algo muy importante: no necesitamos crear un ADR-070 simplemente para completar esta ESP. Las decisiones arquitectónicas relevantes ya están cubiertas.

Estado recomendado

ESP-0001/08 v1.1.0 → 🟢 Arquitectónicamente consolidada

Queda pendiente principalmente la implementación y los detalles contractuales que corresponden a ESP-10, ESP-11, ESP-12, ESP-13, ESP-14 y ESP-15.

El siguiente paso lógico sería ESP-0001/09 — Identity Audit & Compliance, donde tendremos que consolidar especialmente auditoría, security events, trazabilidad de decisiones de Policy Engine, evidencias de Criticality/Freshness y requisitos de compliance.
