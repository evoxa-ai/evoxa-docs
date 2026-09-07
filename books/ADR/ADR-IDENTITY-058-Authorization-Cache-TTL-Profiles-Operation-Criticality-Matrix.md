ADR-IDENTITY-058 — Authorization Cache TTL Profiles & Operation Criticality Matrix

Status: Proposed
Version: 1.0
Date: 2026-09-07
Domain: Identity / Authorization / Security / Distributed Systems / Performance
Priority: Critical
Type: Architecture / Security / Consistency / Performance

1. Contexto

El ADR-IDENTITY-057 estableció el modelo de Freshness Classes, TTL Profiles, Maximum Authorization Staleness (MAS) y Security SLAs para la caché de autorización.

Se definieron cuatro clases:

F0 — Authoritative
F1 — Security Validated
F2 — Controlled Eventual
F3 — Restricted Degraded

También se establecieron como baseline inicial:

Clase	TTL / MAS objetivo
F0	0 / validación autoritativa
F1	≤ 30 s
F2-SHORT	≤ 60 s
F2-MEDIUM	≤ 5 min
F3	Sin extensión de ALLOW

Sin embargo, todavía falta una decisión fundamental:

¿Cómo determina EVOXA qué clase de frescura y qué TTL corresponden a cada operación de autorización?

No resulta suficiente definir TTL por recurso de manera aislada.

La criticidad de una autorización depende de la combinación:

Actor
+
Tenant
+
Resource
+
Action
+
Context
+
Risk
+
Security State

Por ello este ADR establece la primera Operation Criticality Matrix de EVOXA.

2. Problema

Una política de cache demasiado agresiva puede producir:

Performance ↑
Security ↓

Mientras que una política excesivamente conservadora puede producir:

Security ↑
Performance ↓
Scalability ↓

El objetivo es encontrar una clasificación explícita que permita decidir:

Operation
   ↓
Criticality
   ↓
Freshness Class
   ↓
TTL Profile
   ↓
MAS
   ↓
Cache Eligibility

La clasificación debe además ser dinámica cuando el contexto cambie.

Por ejemplo:

GET /athletes/123

puede ser F2 en una consulta normal.

Pero:

DELETE /athletes/123

puede ser F1 o F0 dependiendo del actor, tenant, ownership y riesgo.

3. Decisión arquitectónica

EVOXA utilizará una Operation Criticality Matrix centralizada dentro del Authorization Runtime.

La criticidad será determinada mediante:

Operation Criticality =
f(
    actor,
    tenant,
    resource,
    action,
    context,
    authentication_assurance,
    MFA_assurance,
    ownership,
    delegation,
    risk,
    security_state
)

La clasificación no será responsabilidad del frontend ni del servicio de negocio.

La decisión será centralizada en Identity/Authorization.

4. Principio fundamental

La criticidad será:

la criticidad efectiva de la autorización solicitada, no simplemente la criticidad del recurso.

Por ejemplo:

Resource = User

no significa automáticamente:

F0

Una lectura limitada puede ser F2.

Pero:

Resource = User
Action = disable

será crítica y deberá elevarse a F0.

5. Niveles de Operation Criticality

Se establecen cuatro niveles:

Nivel	Nombre	Freshness
C0	Critical Security	F0
C1	High Security	F1
C2	Standard	F2-SHORT
C3	Low Risk	F2-MEDIUM

F3 no representa una criticidad normal.

F3 representa:

Degraded / Restricted State

y se activa por condiciones operacionales o de seguridad.

6. C0 — Critical Security

C0 representa operaciones donde una autorización incorrecta puede producir un impacto de seguridad significativo.

Estas operaciones requieren:

Freshness = F0
TTL = 0
MAS = 0

No se permite un ALLOW basado únicamente en cache.

6.1 Operaciones C0

Se consideran C0 como baseline:

Identidad
user.disable
user.suspend
user.reactivate
user.delete
user.security_state_change
Membership
membership.revoke
membership.suspend
membership.delete
membership.security_change
Roles
role.permission_remove
role.security_change
membership.role_remove
membership.role_replace
Permissions
permission.revoke
permission.security_change
Sessions
session.revoke
session.revoke_all
session.security_reset
Refresh Tokens
refresh_token.revoke
refresh_token.family_revoke
refresh_token.reuse_response
MFA
mfa.factor_revoke
mfa.factor_replace
mfa.recovery
mfa.security_reset
trusted_device.revoke
Security Policy
policy.activate_restrictive
policy.retire_security_policy
policy.security_change
Tenant
tenant.security_change
tenant.isolation_change
tenant.access_boundary_change
Cross-Tenant
cross_tenant_access
cross_tenant_administration
platform_impersonation
platform_security_operation
Delegation
delegation.revoke
delegation.security_change
delegation.cross_tenant
7. C1 — High Security

C1 corresponde a operaciones sensibles que no necesariamente cambian directamente la frontera de seguridad, pero pueden tener consecuencias importantes.

Baseline:

Freshness = F1
TTL ≤ 30 s
MAS ≤ 30 s

Ejemplos:

user.update_sensitive
membership.update
role.assign
role.modify
permission.assign
permission.modify
delegation.create
delegation.modify
session.inspect_sensitive
mfa.step_up
security_policy.simulate
security_configuration.update

La entrada cacheada podrá utilizarse solamente cuando:

epoch valid
AND
policy version valid
AND
entity version valid
AND
context valid
AND
no invalidation
AND
TTL valid
AND
MAS valid
8. C2 — Standard

C2 corresponde a operaciones normales del sistema.

Baseline:

Freshness = F2-SHORT
TTL ≤ 60 s
MAS ≤ 60 s

Ejemplos:

user.read
membership.read
role.read
permission.read
organization.read
session.list
resource.read
resource.update

La clasificación exacta dependerá del recurso.

Una operación de escritura podrá ser elevada automáticamente a C1 cuando:

el recurso sea sensible;
el actor tenga privilegios elevados;
exista riesgo elevado;
exista delegación;
exista contexto de seguridad especial.
9. C3 — Low Risk

C3 representa operaciones de bajo riesgo.

Baseline:

Freshness = F2-MEDIUM
TTL ≤ 5 min
MAS ≤ 5 min

Ejemplos:

catalog.read
metadata.read
non_sensitive_dashboard.read
low_risk_configuration.read
non_sensitive_reference_data.read

C3 no debe utilizarse para:

información sensible;
permisos;
memberships;
sesiones;
MFA;
políticas de seguridad;
operaciones cross-tenant.
10. F3 — Restricted Degraded

F3 no se asigna normalmente a una operación.

Se activa cuando existe una condición de degradación como:

Redis unavailable
Event Platform lag
cache consumer failure
security epoch uncertainty
reconciliation incomplete
regional partition
authorization infrastructure degradation

En F3:

C0 → DENY / fresh evaluation
C1 → fresh validation / restrictive
C2 → controlled behavior
C3 → potentially cached if explicitly allowed

La degradación nunca debe ampliar privilegios.

11. TTL Profile Matrix

La matriz inicial queda:

Criticality	Freshness	TTL	MAS	Cache ALLOW
C0	F0	0	0	No
C1	F1	≤30 s	≤30 s	Condicional
C2	F2-SHORT	≤60 s	≤60 s	Sí
C3	F2-MEDIUM	≤5 min	≤5 min	Sí
F3	Restricted	No extension	No extension	Restringido
12. Resource Sensitivity

La criticidad también dependerá de la sensibilidad del recurso.

Se establece conceptualmente:

R0 — Security Critical
R1 — Sensitive
R2 — Normal
R3 — Low Risk
R0
users.security
memberships
roles
permissions
sessions
refresh_tokens
mfa
security_policies
security_events
audit_access
tenant_security
R1
user_sensitive_data
delegations
organization_configuration
security_configuration
protected_resources
R2
normal_domain_resources
standard_profile_data
normal_configuration
R3
metadata
catalogs
reference_data
non-sensitive read models
13. Action Sensitivity

Las acciones también tendrán una clasificación.

Acción	Base
read	C2/C3
list	C2/C3
search	C2/C3
create	C1/C2
update	C1/C2
delete	C1
assign	C1
revoke	C0
disable	C0
suspend	C0
restore	C1/C0
grant	C1
remove	C0
security_change	C0
impersonate	C0
cross_tenant	C0

La acción específica puede elevar la criticidad del recurso.

14. Actor Sensitivity

El actor también influye en la evaluación.

Se consideran:

USER
SERVICE
AI_AGENT
ADMIN
PLATFORM_ADMIN
SYSTEM

Pero:

el hecho de ser administrador no reduce las garantías de frescura.

Por el contrario, una operación administrativa puede requerir una clase superior.

Ejemplo:

normal user → profile.read → C3
admin → user.disable → C0
15. AI Agent

Los agentes AI tendrán el mismo modelo.

Ejemplo:

AI_AGENT
+
user.disable

continúa siendo:

C0 / F0

El agente no podrá obtener:

TTL extension

ni:

freshness exemption

por ser un sistema automatizado.

16. Tenant Sensitivity

Las operaciones cross-tenant se clasifican como C0 por defecto.

tenant A
   ↓
resource tenant B

Si no existe una autorización explícita:

DENY

Incluso una cache entry aparentemente válida no puede eliminar esta comprobación.

17. Ownership

Ownership puede elevar la criticidad.

Ejemplo:

resource.owner == actor

puede permitir:

C2

para una operación normal.

Pero:

resource.owner != actor

requiere:

role / permission / delegation / policy

y puede elevar la criticidad a:

C1

o:

C0

dependiendo de la operación.

18. Delegation

Las delegaciones requieren especial cuidado.

Una operación realizada bajo delegation deberá considerar:

delegation validity
delegation scope
delegator authority
delegate authority
expiration
revocation
tenant
resource
action

Una delegación revocada produce:

C0

para la operación de revocación.

Una operación ejecutada bajo una delegación puede ser:

C1

o superior dependiendo del recurso.

La delegación nunca puede elevar el privilegio efectivo del delegador.

19. MFA Assurance

La autorización puede depender de MFA.

Ejemplo:

operation requires MFA

Entonces deberá comprobarse:

MFA factor
MFA challenge
assurance level
evidence freshness
session binding

Una evidencia MFA expirada no puede convertirse en:

ALLOW

por una cache entry.

20. Risk Elevation

El riesgo puede elevar dinámicamente una operación.

Ejemplo:

Base:
C2

pero:

risk = HIGH

produce:

C1

o:

C0

dependiendo de la política.

Se propone:

LOW      → sin elevación
MEDIUM   → C2
HIGH     → C1
CRITICAL → C0

La clasificación definitiva de risk scoring queda sujeta a la arquitectura del Risk Engine.

21. Security State Override

Determinados estados de seguridad fuerzan una clase superior.

Ejemplos:

account compromised
membership revoked
session revoked
MFA recovery active
refresh reuse detected
policy change pending
tenant isolation incident

En estos casos:

cached decision
        ↓
security state override
        ↓
fresh evaluation
22. Criticality Escalation Rule

La criticidad efectiva será:

Effective Criticality =
MAX(
    Resource Criticality,
    Action Criticality,
    Context Criticality,
    Risk Criticality,
    Security State Criticality,
    Tenant Criticality,
    Delegation Criticality
)

Esto evita que una dimensión de bajo riesgo reduzca otra dimensión crítica.

Ejemplo:

Resource = R3
Action = revoke

Resultado:

C0

No:

C3
23. Criticality Downgrade

No se permitirá un downgrade arbitrario.

Por ejemplo:

security policy says C0

no puede convertirse en:

C2

porque:

TTL = 60s

La política de seguridad siempre puede elevar la criticidad.

Una política organizacional no puede reducir los mínimos establecidos por la plataforma.

24. Matrix de combinación

La matriz conceptual será:

Resource	Action	Base Criticality
R0	read	C1
R0	create	C1
R0	update	C1
R0	delete	C0
R0	revoke	C0
R0	security_change	C0
R1	read	C2
R1	create	C1
R1	update	C1
R1	delete	C1
R2	read	C2
R2	create	C2
R2	update	C2
R2	delete	C1
R3	read	C3
R3	create	C2
R3	update	C2
R3	delete	C1

Esta tabla constituye el baseline, no una lista inmutable.

25. Ejemplos
Ejemplo 1 — Lectura normal
Actor: User
Tenant: Organization A
Resource: profile
Action: read
Risk: LOW
Ownership: valid

Resultado:

C3
F2-MEDIUM
TTL ≤ 5 min
Ejemplo 2 — Modificación normal
Resource: profile
Action: update

Resultado:

C2
F2-SHORT
TTL ≤ 60 s
Ejemplo 3 — Eliminar usuario
Resource: user
Action: delete

Resultado:

C0
F0
TTL = 0
Ejemplo 4 — Revocar membership
Resource: membership
Action: revoke

Resultado:

C0
F0
Ejemplo 5 — Administrador leyendo usuarios
Actor: ADMIN
Resource: user
Action: read

No necesariamente:

C0

Puede ser:

C1

porque la operación sigue siendo una lectura.

Pero si el contexto indica:

sensitive data

puede elevarse.

Ejemplo 6 — AI modificando una configuración
Actor: AI_AGENT
Resource: security_configuration
Action: update

Resultado:

C0/C1

según la configuración concreta.

Nunca:

C3

por tratarse de una operación automatizada.

26. Cache Eligibility

La elegibilidad final no dependerá solamente de la clase.

Conceptualmente:

CacheEligible =
    ClassAllowsCache
    AND TTLValid
    AND MASValid
    AND EpochValid
    AND PolicyVersionValid
    AND EntityVersionValid
    AND ContextValid
    AND NoInvalidation
    AND GenerationValid
    AND TenantValid

Para C0:

CacheEligible(ALLOW) = false

salvo que exista una validación autoritativa equivalente explícitamente aprobada.

27. Security SLA Matrix
Evento	Objetivo
Security revocation	≤5 s
Session revocation	≤5 s
Membership revocation	≤5 s
MFA revocation	≤5 s
Delegation revocation	≤5 s
Security policy tightening	≤5 s
Permission removal	≤5 s
New grant	≤30 s
New role assignment	≤30 s
New membership	≤30 s
Non-critical policy relaxation	≤30 s

Los valores son objetivos iniciales y deberán validarse operacionalmente.

28. Security SLA vs TTL

Es importante diferenciar:

TTL

de:

Security SLA

Ejemplo:

TTL = 5 min
Revocation SLA = 5 sec

Si ocurre:

revocation at T+10s

la entrada debe invalidarse inmediatamente.

No se espera a:

T+5min

Por lo tanto:

TTL ≠ Revocation SLA
29. Grant vs Revocation

La arquitectura acepta que:

Grant propagation

sea ligeramente más lenta.

Por ejemplo:

10:00 role granted
10:00:10 cache still old

puede generar:

DENY

temporalmente.

Esto es preferible a:

ALLOW

a un usuario cuyo permiso acaba de ser revocado.

30. Fail-Closed Matrix
Estado	C0	C1	C2	C3
Normal	Fresh	F1	F2	F2
Redis down	Fresh	Fresh	fallback	fallback
Event lag	Fresh	Fresh	restricted	restricted
Epoch unknown	DENY/Fresh	Fresh	restricted	restricted
Policy version unknown	DENY/Fresh	Fresh	restricted	restricted
Tenant uncertain	DENY	DENY	DENY	DENY
Cache corrupted	Fresh	Fresh	Fresh	Fresh
Recovery incomplete	Fresh	Fresh	restricted	restricted
31. Cache Stampede

Cuando una gran cantidad de entradas expiren simultáneamente:

TTL expiration
       ↓
mass cache miss
       ↓
PostgreSQL overload

el sistema deberá utilizar mecanismos como:

single-flight;
request coalescing;
bounded concurrency;
jitter;
progressive warm-up;
backpressure.

Pero estos mecanismos no podrán modificar:

Criticality
TTL
MAS
Security SLA
32. Configuration Model

Los TTL Profiles deberán ser configurables centralmente.

Conceptualmente:

FreshnessProfile
----------------
id
criticality
freshness_class
ttl
mas
cache_allow
requires_epoch
requires_policy_version
requires_entity_version
requires_context_validation
enabled
version

La configuración deberá estar versionada.

Los cambios de seguridad deberán quedar auditados.

33. Policy Override

Una policy puede elevar los requisitos.

Ejemplo:

Base:
C2 / TTL 60s

Policy:

resource = financial_profile
risk = high

Resultado:

C1 / TTL 30s

Una policy no puede debilitar un mínimo de seguridad establecido por la plataforma.

34. Configuration Governance

Los cambios sobre:

TTL;
MAS;
criticality;
cache eligibility;
security SLA;

deberán seguir governance.

No deberán modificarse silenciosamente en runtime.

Cambios relevantes deberán producir:

audit event
+
configuration version
+
effective timestamp
35. Testing Matrix

Cada combinación crítica deberá probarse.

Ejemplo:

Resource
×
Action
×
Actor
×
Tenant
×
Risk
×
Ownership
×
MFA
×
Delegation

No será necesario probar todas las combinaciones exhaustivamente si el motor utiliza reglas composables, pero deberán existir pruebas de:

límites;
conflictos;
elevaciones;
overrides;
precedence;
invalidación.
36. Security Test Cases
TC-01

Una operación C0 nunca utiliza stale ALLOW.

TC-02

Un cambio de epoch invalida cualquier entrada anterior.

TC-03

Una revocación de membership invalida autorización inmediatamente.

TC-04

Una policy tightening eleva correctamente la criticidad.

TC-05

Un tenant mismatch siempre produce DENY.

TC-06

Un AI Agent recibe las mismas restricciones.

TC-07

Un usuario delegado no obtiene privilegios superiores al delegador.

TC-08

Un riesgo HIGH eleva la criticidad.

TC-09

Una entrada C3 no puede utilizarse para una operación que la policy haya elevado a C1.

TC-10

Una caída de Event Platform no permite stale ALLOW para C0.

37. Observabilidad

Cada decisión de criticidad deberá poder ser explicada operacionalmente mediante identificadores y metadata no sensible.

Ejemplo conceptual:

criticality = C0
freshness_class = F0
reason_codes =
[
  "RESOURCE_SECURITY_CRITICAL",
  "ACTION_REVOKE",
  "MEMBERSHIP_BOUNDARY"
]

No deberán exponerse secretos ni información sensible innecesaria.

38. Reason Codes

Se propone un catálogo inicial:

RESOURCE_SECURITY_CRITICAL
ACTION_SECURITY_CRITICAL
ACTION_REVOKE
ACTION_DELETE
ACTION_DISABLE
CROSS_TENANT
HIGH_RISK
CRITICAL_RISK
MFA_REQUIRED
MFA_STALE
DELEGATION_SENSITIVE
SESSION_SECURITY
POLICY_SECURITY_CHANGE
MEMBERSHIP_BOUNDARY
PRIVILEGED_OPERATION
SECURITY_STATE_OVERRIDE

El catálogo definitivo será gestionado posteriormente.

39. Consecuencias positivas

Este ADR proporciona:

clasificación objetiva de operaciones;
TTLs diferenciados;
reglas claras de cache eligibility;
prioridad de seguridad;
soporte para risk-based authorization;
soporte para AI agents;
protección cross-tenant;
protección de delegaciones;
relación clara entre criticidad y freshness;
SLAs medibles;
mejor capacidad de testing;
reducción de decisiones arbitrarias.
40. Consecuencias negativas

La arquitectura incorpora mayor complejidad:

matriz de criticidad;
clasificación de recursos;
clasificación de acciones;
risk elevation;
policy overrides;
governance;
reason codes;
más escenarios de testing.

Además, la matriz deberá evolucionar con EVOXA.

Por ello no debe implementarse como un conjunto rígido de if/else dispersos por los dominios.

Debe existir un mecanismo central.

41. Alternativas rechazadas
41.1 TTL por endpoint

Rechazado.

No considera:

actor
risk
tenant
ownership
delegation
security state
41.2 TTL por recurso

Rechazado como mecanismo único.

La misma entidad puede tener operaciones con riesgos completamente diferentes.

41.3 TTL por rol

Rechazado.

El rol no determina por sí solo la criticidad.

41.4 TTL único

Rechazado.

No permite optimizar operaciones de bajo riesgo sin debilitar las críticas.

41.5 Criticidad definida por frontend

Rechazado.

El frontend no es una autoridad de seguridad.

42. Non-Negotiables
La criticidad se determina centralmente.
Resource + Action son solamente el punto de partida.
Contexto puede elevar la criticidad.
Risk puede elevar la criticidad.
Security State puede elevar la criticidad.
Cross-tenant es C0 por defecto.
Revocations son C0.
Security changes son C0.
F0 no permite stale ALLOW.
TTL nunca reduce una garantía de seguridad.
Revocation SLA tiene prioridad sobre TTL.
Grant propagation puede ser eventualmente consistente.
Tenant mismatch produce DENY.
AI Agents utilizan la misma matriz.
Delegation no puede ampliar privilegios.
Platform security minimums no pueden ser debilitados por una organización.
F3 no extiende permisos.
Redis no es fuente de verdad.
PostgreSQL continúa siendo autoridad.
Los cambios de criticality/TTL deben ser auditables.
43. Traceability
Blueprint
Identity
Authorization
RBAC
ABAC
Tenant Isolation
Ownership
Delegation
Security Policy Engine
Event-Driven Architecture
Performance
Observability
AI-Native Security
Implementation Stories
IS-IDENTITY-006 — Authorization & Permission Evaluation
IS-IDENTITY-007 — Role & Permission Management
IS-IDENTITY-008 — Session Management
IS-IDENTITY-010 — Multi-Factor Authentication
IS-IDENTITY-011 — Security Policy Engine
IS-IDENTITY-012 — Audit & Security Events
IS-IDENTITY-013 — Domain Events & Outbox
IS-IDENTITY-015 — Identity Persistence
IS-IDENTITY-016 — Identity Testing & Quality
IS-IDENTITY-017 — Identity Runtime & Deployment
Related ADRs
ADR-IDENTITY-045 — Authorization Policy Model/Evaluation
ADR-IDENTITY-046 — Policy Language/Schema/Governance
ADR-IDENTITY-047 — Policy Engine Technology & Implementation
ADR-IDENTITY-048 — Policy DSL & Expression Language
ADR-IDENTITY-049 — Policy Evaluation Runtime
ADR-IDENTITY-050 — Cache Invalidation & Consistency
ADR-IDENTITY-051 — Cache Key Schema & Context Fingerprinting
ADR-IDENTITY-052 — Cache Eligibility & TTL
ADR-IDENTITY-053 — Cache Invalidation Event Model
ADR-IDENTITY-054 — Cache Failure, Degradation & Recovery
ADR-IDENTITY-055 — Cache Recovery, Reconciliation & Warm-Up
ADR-IDENTITY-056 — Cache Consistency Model & Freshness Guarantees
ADR-IDENTITY-057 — Freshness Classes, TTL Profiles & Security SLAs
44. Pending Decisions

Quedan para decisiones posteriores:

Catálogo definitivo de recursos R0–R3.
Catálogo definitivo de acciones.
Catálogo completo C0–C3.
Matriz completa Resource × Action.
Risk Engine y scoring definitivo.
Definición exacta de HIGH y CRITICAL risk.
Context attributes definitivos.
TTLs definitivos por dominio.
MAS definitivo por operación.
SLA multi-región.
SLA para event propagation bajo incidentes.
Política de TTL jitter.
Política de cache warming.
Política de configuration rollout.
Versionado de Criticality Matrix.
Governance para modificaciones.
Catálogo definitivo de reason codes.
45. Acceptance Criteria
AC-01

Toda autorización recibe una clasificación de criticidad efectiva.

AC-02

La criticidad nunca puede ser reducida por TTL.

AC-03

C0 utiliza F0.

AC-04

C1 utiliza F1.

AC-05

C2 utiliza F2-SHORT.

AC-06

C3 utiliza F2-MEDIUM.

AC-07

F0 no permite stale ALLOW.

AC-08

Cross-tenant se clasifica C0 por defecto.

AC-09

Revocation se clasifica C0.

AC-10

Security policy change se clasifica C0.

AC-11

Risk HIGH puede elevar una operación.

AC-12

Security State puede elevar una operación.

AC-13

Delegation nunca puede ampliar privilegios.

AC-14

AI Agents reciben exactamente las mismas reglas.

AC-15

Una policy puede elevar requisitos de freshness.

AC-16

Una policy organizacional no puede reducir platform security minimums.

AC-17

Los TTL Profiles están versionados.

AC-18

Los cambios de TTL/Criticality son auditables.

AC-19

Las métricas permiten comprobar los Security SLAs.

AC-20

Una degradación F3 no puede producir una autorización insegura.

46. Resultado arquitectónico

Con ADR-057 y ADR-058, el modelo queda:

                    AUTHORIZATION REQUEST
                             │
                             ▼
                 ┌─────────────────────┐
                 │ Context Construction│
                 └──────────┬──────────┘
                            │
                            ▼
              ┌──────────────────────────┐
              │ Operation Criticality    │
              │                          │
              │ Resource                 │
              │ Action                   │
              │ Actor                    │
              │ Tenant                   │
              │ Ownership                │
              │ Delegation               │
              │ Risk                     │
              │ MFA                      │
              │ Security State           │
              └────────────┬─────────────┘
                           │
                           ▼
                ┌────────────────────┐
                │ C0 / C1 / C2 / C3 │
                └─────────┬──────────┘
                          │
                          ▼
                ┌────────────────────┐
                │ F0 / F1 / F2 / F3 │
                └─────────┬──────────┘
                          │
                          ▼
                ┌────────────────────┐
                │ TTL / MAS / SLA    │
                └─────────┬──────────┘
                          │
                          ▼
             ┌──────────────────────────┐
             │ Cache Eligibility        │
             │                          │
             │ Epoch                    │
             │ Policy Version           │
             │ Entity Version           │
             │ Context                  │
             │ Invalidation             │
             │ Generation               │
             └────────────┬─────────────┘
                          │
                 ┌────────┴────────┐
                 ▼                 ▼
             CACHE OK          FRESH EVAL
                 │                 │
                 └────────┬────────┘
                          ▼
                AUTHORIZATION RUNTIME
                          │
                          ▼
       ALLOW / DENY / CHALLENGE / REAUTH / MFA / REVOKE
Principio definitivo de esta etapa

La criticidad determina la frescura requerida; la frescura determina la elegibilidad de la caché; y ninguna optimización de caché puede reducir una garantía de seguridad.

Con esto queda definida la conexión entre Operation Criticality → Freshness Class → TTL → MAS → Security SLA → Cache Eligibility, que era la pieza que faltaba después de ADR-057.

Siguiente ADR recomendado:

ADR-IDENTITY-059 — Authorization Criticality Policy Model, Rule Precedence & Dynamic Risk Escalation

Este sería el siguiente paso lógico para definir cómo se representa técnicamente esta matriz como políticas, cómo se resuelven conflictos entre reglas y cómo se aplica la elevación dinámica por riesgo/contexto dentro del Authorization Runtime.
