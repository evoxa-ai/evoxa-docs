ADR-IDENTITY-059 — Authorization Criticality Policy Model, Rule Precedence & Dynamic Risk Escalation

Status: Proposed
Version: 1.0
Date: 2026-09-07
Domain: Identity / Authorization / Security / Policy Engine / Risk
Priority: Critical
Type: Architecture / Security / Policy / Runtime

1. Contexto

Los ADR anteriores establecieron la arquitectura progresiva del Authorization Policy Engine de EVOXA.

En particular:

ADR-045 estableció el modelo general de autorización.
ADR-046 definió el modelo y governance de políticas.
ADR-047 definió la implementación conceptual del Policy Engine.
ADR-048 definió el DSL declarativo.
ADR-049 definió el runtime y algoritmo de evaluación.
ADR-050–056 definieron la arquitectura de cache, invalidación, recuperación y consistencia.
ADR-057 definió Freshness Classes, TTL Profiles y Security SLAs.
ADR-058 definió la Operation Criticality Matrix.

ADR-058 estableció que una autorización debe clasificarse considerando:

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
+
Ownership
+
Delegation
+
MFA Assurance

y obtener una criticidad:

C0 — Critical Security
C1 — High Security
C2 — Standard
C3 — Low Risk

La siguiente necesidad arquitectónica es determinar cómo se expresan esas reglas, cómo se resuelven cuando varias reglas aplican simultáneamente y cómo una condición dinámica puede elevar la criticidad durante runtime.

2. Problema

Una matriz estática no es suficiente.

Ejemplo:

profile.read

puede normalmente ser:

C3

pero el mismo request podría convertirse en:

C1

si:

risk = HIGH

o:

C0

si:

cross_tenant = true

Otro ejemplo:

user.update

podría ser:

C2

pero elevarse a:

C1

si afecta información sensible.

Por lo tanto, EVOXA necesita un mecanismo determinista para:

identificar reglas aplicables;
evaluar condiciones;
calcular criticidad base;
aplicar elevaciones;
resolver conflictos;
determinar Freshness Class;
seleccionar TTL/MAS;
impedir downgrades de seguridad.
3. Decisión arquitectónica

EVOXA utilizará un Authorization Criticality Policy Model centralizado dentro del Authorization Policy Engine.

El modelo seguirá el mismo principio que el DSL definido en ADR-048:

Declarativo, tipado, determinista, versionado y sin código arbitrario.

Las reglas de criticidad serán tratadas como políticas.

Conceptualmente:

Authorization Request
        ↓
Context
        ↓
Criticality Policy Selection
        ↓
Applicable Rules
        ↓
Rule Evaluation
        ↓
Base Criticality
        ↓
Dynamic Risk Escalation
        ↓
Security Overrides
        ↓
Effective Criticality
        ↓
Freshness Profile
4. Separación entre autorización y criticidad

La criticidad no concede ni deniega permisos.

Su responsabilidad es determinar:

qué nivel de garantía de autorización necesita

Por lo tanto:

Criticality ≠ Authorization Decision

Ejemplo:

Criticality = C0

no significa:

ALLOW

Significa:

la autorización requiere garantías F0

La decisión final continúa siendo responsabilidad del Authorization Runtime.

5. Modelo conceptual

Una CriticalityPolicy tendrá conceptualmente:

CriticalityPolicy
-------------------------
id
name
version
status
scope
priority
target
conditions
base_criticality
elevations
security_overrides
freshness_profile
effective_from
effective_until
metadata

El modelo deberá estar versionado.

6. Status de las políticas

Se utilizará el lifecycle definido anteriormente:

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

Solo las políticas:

ACTIVE

participan en decisiones de producción.

7. Scope

Las políticas podrán existir conceptualmente en:

PLATFORM
ORGANIZATION
RESOURCE

Sin embargo, su capacidad de modificar mínimos de seguridad estará limitada.

PLATFORM

Define:

mínimos de seguridad;
operaciones C0;
reglas obligatorias;
límites máximos de TTL;
restricciones de downgrade.
ORGANIZATION

Puede añadir restricciones más estrictas.

No puede debilitar:

Platform Security Minimums
RESOURCE

Puede definir requisitos específicos de un recurso.

8. Base Criticality

Cada operación tendrá una criticidad base determinada principalmente por:

Resource
+
Action

Ejemplo:

resource = profile
action = read
base = C3

o:

resource = membership
action = revoke
base = C0

La base constituye el punto inicial, no necesariamente el resultado final.

9. Dynamic Elevation

Las condiciones dinámicas podrán elevar la criticidad.

Ejemplo:

IF risk.level == HIGH
THEN criticality = C1

Otro:

IF cross_tenant == true
THEN criticality = C0

Otro:

IF mfa_assurance < required
THEN criticality = C1

La elevación siempre podrá aumentar la criticidad.

10. Regla de monotonicidad

La evaluación será monotónica respecto a seguridad.

Es decir:

Dynamic Rule

puede:

C3 → C2
C2 → C1
C1 → C0
C3 → C0

pero no:

C0 → C1
C1 → C2
C2 → C3

cuando la elevación proviene de una condición de seguridad obligatoria.

11. Effective Criticality

La criticidad efectiva será:

Effective Criticality =
MAX(
    Base Criticality,
    Dynamic Risk Criticality,
    Context Criticality,
    Security State Criticality,
    Tenant Criticality,
    Delegation Criticality,
    MFA Criticality,
    Platform Minimum
)

La función MAX se interpreta según el orden:

C0 > C1 > C2 > C3
12. Rule Categories

Las reglas se clasificarán conceptualmente en:

BASE
ELEVATION
SECURITY_OVERRIDE
PLATFORM_MINIMUM
CONTEXT_OVERRIDE
RISK_ELEVATION
13. BASE Rules

Definen la criticidad inicial.

Ejemplo:

RULE:
resource = "profile"
action = "read"

base_criticality = C3

Otro:

resource = "user"
action = "update"

base_criticality = C2
14. ELEVATION Rules

Incrementan la criticidad si se cumple una condición.

Ejemplo:

IF resource.sensitivity == "HIGH"
THEN minimum_criticality = C1

Otro:

IF risk.level == "HIGH"
THEN minimum_criticality = C1
15. SECURITY_OVERRIDE Rules

Son reglas obligatorias.

Ejemplo:

IF cross_tenant == true
THEN minimum_criticality = C0

Otro:

IF action == "revoke"
THEN minimum_criticality = C0

Estas reglas no pueden ser debilitadas por políticas organizacionales.

16. Platform Minimum

La plataforma puede establecer mínimos.

Ejemplo:

membership.revoke
minimum = C0

Una organización no puede establecer:

membership.revoke
C2

para reducir la garantía.

El resultado seguirá siendo:

C0
17. Rule Precedence

Cuando existen múltiples reglas, la precedencia será:

1. Regulatory / Mandatory Security
2. Platform Security Minimum
3. Security State Override
4. Tenant Isolation
5. Resource Security Policy
6. Dynamic Risk Elevation
7. Contextual Elevation
8. Organization Policy
9. Default Operation Matrix

Sin embargo, la precedencia no significa que una regla inferior pueda reducir una regla superior.

La combinación final continuará respetando:

MAX(Criticality)
18. Precedence vs MAX

Es importante distinguir:

Precedence

Determina:

qué regla tiene autoridad
MAX

Determina:

qué criticidad efectiva resulta de múltiples requisitos de seguridad

Por ejemplo:

Platform minimum = C1
Risk rule = C0
Organization rule = C2

Resultado:

C0

La organización no puede reducirlo a C2.

19. Explicit Deny / Security Block

La criticidad no reemplaza las decisiones:

DENY

Si una policy establece:

cross_tenant = true
DENY

el sistema no debe interpretar esto simplemente como:

C0

Son dos conceptos distintos:

Criticality = required security assurance
Decision = authorization outcome
20. Risk Model

El riesgo dinámico se representará mediante un nivel normalizado.

Baseline:

LOW
MEDIUM
HIGH
CRITICAL

Orden:

CRITICAL > HIGH > MEDIUM > LOW
21. Risk → Criticality Mapping

Baseline inicial:

Risk	Criticality mínima
LOW	C3
MEDIUM	C2
HIGH	C1
CRITICAL	C0

Esta conversión será configurable mediante políticas.

22. Risk Elevation Example

Request:

resource = workout
action = read

Base:

C3

Risk:

LOW

Resultado:

C3

Pero:

risk = HIGH

produce:

C1

Y:

risk = CRITICAL

produce:

C0
23. Risk Sources

El Risk Context puede incluir:

authentication anomalies
login failures
MFA anomalies
device reputation
session anomalies
IP reputation
location anomalies
impossible travel
refresh token reuse
account compromise signals
behavior anomalies
request velocity
privilege escalation indicators

No todos estos factores serán necesariamente implementados inicialmente.

24. Risk Engine Boundary

El Risk Engine no decidirá directamente:

ALLOW
DENY

Su responsabilidad será aportar:

risk context

El Authorization Runtime utilizará ese contexto para determinar la criticidad y la autorización.

Por tanto:

Risk Engine
     ↓
Risk Context
     ↓
Authorization Runtime

y no:

Risk Engine → ALLOW
25. Risk Confidence

El riesgo puede tener incertidumbre.

Conceptualmente:

risk.level
risk.score
risk.confidence
risk.source
risk.timestamp

Cuando el riesgo sea:

UNKNOWN

no deberá reducir la criticidad.

Para operaciones críticas:

UNKNOWN → Fresh Evaluation / Restrictive

Nunca:

UNKNOWN → C3
26. Risk Freshness

El riesgo es temporal.

Por ello:

risk.timestamp

deberá ser considerado.

Un risk assessment antiguo no debe reutilizarse indefinidamente.

La freshness del riesgo deberá ser compatible con la operación.

Ejemplo:

high-risk operation

puede requerir:

fresh risk evaluation

mientras una operación de bajo riesgo puede tolerar contexto de riesgo ligeramente más antiguo.

27. Risk Escalation vs TTL

Un cambio de riesgo debe poder elevar una autorización incluso si la cache entry tiene TTL vigente.

Ejemplo:

10:00
risk = LOW
cache = C3

10:01
risk = CRITICAL

10:02
TTL still valid

La decisión C3 debe dejar de ser utilizable para una operación que ahora requiere C0.

Por tanto:

Risk Change > TTL Validity

cuando la nueva evaluación eleva la criticidad.

28. Security State Escalation

Los estados de seguridad tienen precedencia especial.

Ejemplo:

account_compromised = true

debe producir una elevación inmediata según policy.

Otros ejemplos:

refresh_token_reuse_detected
membership_revoked
mfa_recovery
session_revoked
security_incident

Estos estados pueden incluso producir:

REVOKE
DENY
REAUTHENTICATE
REQUIRE_MFA

y no solamente C0.

29. Tenant Isolation

Cualquier incertidumbre relacionada con tenant deberá elevar la seguridad.

Ejemplo:

tenant_id missing

o:

resource.tenant_id != context.tenant_id

produce:

DENY

La criticidad no puede utilizarse para "resolver" una violación de tenant.

30. Organization Policy

Una organización puede definir reglas más estrictas.

Ejemplo:

Platform:
profile.read = C3

Organización:

profile.read
IF sensitive_context
THEN C1

Resultado:

C1

La organización puede endurecer, pero no debilitar.

31. Rule Conflict

Ejemplo:

Rule A:
profile.update → C2

Rule B:
profile.update + HIGH_RISK → C1

Rule C:
profile.update + cross_tenant → C0

Resultados:

normal:
C2

high risk:
C1

cross tenant:
C0

No existe conflicto real porque las reglas se combinan monotónicamente.

32. Contradictory Rules

Si se detecta:

Rule A → C0
Rule B → C3

para exactamente la misma condición y scope, el sistema deberá identificarlo durante validación.

No se debe depender únicamente del runtime.

El Policy Validator deberá detectar:

contradicciones;
reglas inalcanzables;
reglas redundantes;
shadowing;
condiciones imposibles.
33. Shadowed Rule

Ejemplo:

Rule A:
cross_tenant → C0

Rule B:
cross_tenant + HIGH_RISK → C1

B está shadowed porque:

cross_tenant → C0

siempre domina.

La herramienta de validación deberá advertirlo.

34. Rule Ordering

No se utilizará:

first-match-wins

como mecanismo principal.

La evaluación será:

collect applicable rules
        ↓
evaluate all relevant security rules
        ↓
combine requirements
        ↓
resolve precedence
        ↓
calculate effective criticality

Esto evita que una regla permisiva aparezca antes que una regla de seguridad.

35. Default Deny for Missing Criticality

Si el sistema no puede determinar una criticidad requerida para una operación desconocida:

unknown operation

no deberá asumir:

C3

Deberá producir:

configuration error

o utilizar un mínimo seguro definido por plataforma.

Para operaciones desconocidas en producción:

fail closed

es el comportamiento preferido.

36. Criticality Policy Example

Conceptualmente:

{
  "id": "criticality-membership-revoke",
  "version": 1,
  "status": "ACTIVE",
  "scope": "PLATFORM",

  "target": {
    "resource": "membership",
    "action": "revoke"
  },

  "base_criticality": "C0"
}
37. Dynamic Risk Policy Example
{
  "id": "criticality-high-risk",
  "version": 1,
  "status": "ACTIVE",

  "target": {
    "action": "*"
  },

  "conditions": [
    {
      "attribute": "risk.level",
      "operator": "EQUALS",
      "value": "HIGH"
    }
  ],

  "minimum_criticality": "C1"
}

El formato final deberá respetar el DSL definitivo de ADR-048.

38. Security Override Example
IF
    request.cross_tenant == true
THEN
    DENY
    AND
    minimum_criticality = C0

Esto demuestra nuevamente que:

Decision

y:

Criticality

son dimensiones independientes.

39. Dynamic Elevation Pipeline

La evaluación será:

1. Identify operation
2. Resolve base criticality
3. Validate tenant
4. Evaluate security state
5. Evaluate risk
6. Evaluate MFA/session state
7. Evaluate ownership/delegation
8. Evaluate contextual rules
9. Apply platform minimums
10. Apply organization restrictions
11. Calculate effective criticality
12. Resolve freshness profile
13. Continue authorization evaluation
40. Criticality Before Cache

La criticidad debe determinarse antes de aceptar una decisión cacheada.

No:

Cache HIT
   ↓
ALLOW
   ↓
Criticality

Sino:

Request
   ↓
Context
   ↓
Criticality
   ↓
Freshness Requirements
   ↓
Cache Eligibility
   ↓
Cache / Fresh Evaluation

Esto es esencial.

41. Cache Key Implications

La cache deberá ser consciente de los elementos que pueden cambiar la criticidad.

Conceptualmente:

actor
tenant
resource
action
context fingerprint
risk context
policy version
security epoch

No todos los valores necesariamente forman parte de una única key; algunos serán validaciones/versiones asociadas.

La decisión concreta sigue las reglas de ADR-051/052.

42. Risk Change Invalidation

Cuando un cambio de riesgo pueda elevar la criticidad:

Risk Update
    ↓
Risk Version / Timestamp
    ↓
Cache Eligibility Check
    ↓
Potential Invalidation

No será obligatorio invalidar todas las entradas por cualquier cambio de riesgo.

La invalidación deberá ser:

scoped

y determinada por las políticas aplicables.

43. Risk Escalation Rules

Las reglas podrán especificar:

minimum_criticality

pero no:

maximum_criticality

para reducir seguridad.

Ejemplo permitido:

risk.high → minimum C1

Ejemplo no permitido:

risk.critical → maximum C1

si una plataforma requiere C0.

44. Maximum Security Guarantee

Las políticas podrán establecer:

minimum criticality

y el runtime calculará:

MAX(all applicable minimums)

Esto crea un modelo monotónico y seguro.

45. Relation with Freshness Profiles

Una vez calculada:

Effective Criticality

se resuelve:

Criticality	Freshness Profile
C0	F0
C1	F1
C2	F2-SHORT
C3	F2-MEDIUM

Luego se aplican las validaciones de:

TTL
MAS
Epoch
Policy Version
Entity Version
Context
Invalidation
Generation

definidas en ADR-057/058.

46. Auditability

Cada evaluación crítica deberá poder producir metadata como:

criticality = C0
base = C2

elevations = [
  "CROSS_TENANT",
  "HIGH_RISK"
]

effective = C0

freshness = F0

policy_versions = [...]

La información sensible no deberá exponerse innecesariamente.

47. Explainability

El sistema deberá poder explicar internamente:

Why C0?

por ejemplo:

BASE_C2
+
RISK_HIGH → C1
+
CROSS_TENANT → C0
=
EFFECTIVE_C0

Esta explicación es especialmente útil para:

auditoría;
debugging;
security operations;
testing;
AI-assisted analysis.
48. AI-Assisted Criticality

La IA podrá:

sugerir nuevas reglas;
detectar patrones;
detectar reglas shadowed;
recomendar elevaciones;
analizar anomalías;
simular escenarios.

Pero:

AI suggestion
    ≠
ACTIVE security policy

Una política propuesta por IA requerirá:

validation
+
human approval
+
activation
49. AI Runtime

Durante runtime:

AI Agent
    ↓
normal Authorization Request
    ↓
Criticality Engine
    ↓
same rules

No existe:

AI bypass

ni:

AI reduced TTL

ni:

AI criticality exemption
50. Governance

Los cambios de Criticality Policies deberán seguir:

DRAFT
→ VALIDATING
→ APPROVED
→ ACTIVE

y registrar:

policy id
version
author
approver
timestamp
reason
scope
changes

Los cambios de:

C0
C1
TTL
MAS
security minimum
cross-tenant rules

deberán considerarse cambios de alta sensibilidad.

51. Versioning

Las políticas de criticidad serán versionadas.

Ejemplo:

criticality-policy-v4

Una nueva versión incompatible deberá producir:

policy version change
+
affected cache invalidation

según ADR-053.

52. Rollback

El rollback de una política de criticidad no debe asumir que:

old cache = safe

Después de un rollback:

new policy version

deberá provocar la invalidación/revalidación correspondiente.

El rollback debe ser tratado como un cambio de seguridad.

53. Testing

Deberán existir pruebas de:

Base
resource/action → expected Cx
Elevation
risk → Cx
Overrides
cross tenant → C0
Precedence
organization vs platform
Conflicts
contradictory rules
Dynamic changes
risk LOW → HIGH
Cache
criticality changed while TTL valid
Recovery
policy version changed during cache recovery
54. Chaos Testing

Se deberán probar escenarios como:

risk update during authorization
policy update during request
policy propagation delay
cache stale during risk escalation
event duplication
event reordering
Redis outage
risk engine unavailable
policy engine unavailable
cross-region risk inconsistency

El sistema deberá demostrar:

No security downgrade
55. Performance

La evaluación de criticidad deberá ser suficientemente eficiente para integrarse en cada authorization request.

Objetivo:

Criticality evaluation:
negligible relative overhead

y contribuir al objetivo interno:

Authorization P95 < 50 ms

No deberá introducir llamadas externas innecesarias durante cada request.

56. Failure Modes
Risk Engine unavailable

Si la operación es crítica:

fresh / restrictive

No debe asumirse:

risk = LOW
Policy Store unavailable

Para operaciones críticas:

fail closed / authoritative validation
Cache unavailable

Se aplica el modelo de ADR-054.

Context unavailable

Si el contexto es obligatorio:

DENY / CHALLENGE / REAUTHENTICATE

según policy.

Nunca:

ALLOW

por defecto.

57. Consecuencias positivas

Este ADR proporciona:

modelo formal de criticidad;
reglas dinámicas;
risk-based escalation;
precedencia determinista;
protección contra downgrade;
separación entre criticidad y autorización;
integración natural con cache;
integración con MFA/session/risk;
governance;
versionado;
auditabilidad;
soporte AI seguro.
58. Consecuencias negativas

Introduce:

mayor complejidad del Policy Engine;
necesidad de un catálogo de recursos/acciones;
necesidad de un Risk Context;
validación de reglas;
versionado;
testing combinatorio;
más eventos de invalidación;
mayor complejidad operacional.

Se considera necesario debido a la naturaleza crítica de Identity y Authorization.

59. Alternativas rechazadas
59.1 Criticality hardcoded

Rechazado.

Produciría reglas dispersas y difíciles de gobernar.

59.2 Criticality definida por endpoint

Rechazado.

No permite evaluar contexto dinámico.

59.3 Risk Engine decide directamente ALLOW/DENY

Rechazado.

El Risk Engine debe proporcionar contexto, no reemplazar al Authorization Runtime.

59.4 First-match-wins

Rechazado.

Una regla permisiva podría ocultar una regla de seguridad.

59.5 Last-match-wins

Rechazado.

El orden accidental podría producir un downgrade.

59.6 Organization puede reducir C0

Rechazado.

Los mínimos de seguridad de plataforma son inmutables desde el punto de vista organizacional.

60. Non-Negotiables
Criticality no equivale a Authorization Decision.
Criticality se calcula centralmente.
C0 > C1 > C2 > C3.
Las reglas de seguridad son monotónicas.
Las reglas pueden elevar, no reducir mínimos de seguridad.
Platform minimums no pueden ser debilitados.
Cross-tenant es C0 por defecto.
Revocations son C0.
Security changes son C0.
Risk HIGH puede elevar a C1.
Risk CRITICAL puede elevar a C0.
Unknown risk no puede reducir criticidad.
Risk Engine no decide ALLOW/DENY.
First-match-wins no es el mecanismo principal.
Las reglas deben ser deterministas.
No se permite código arbitrario dentro de policies.
La criticidad se determina antes de aceptar una cache entry.
Un cambio de criticidad puede invalidar cache.
Las políticas deben estar versionadas.
Cambios críticos deben ser auditables.
AI no puede activar directamente políticas.
AI no puede reducir criticality.
Tenant uncertainty no puede producir ALLOW.
Security State puede elevar criticidad.
Una operación desconocida no debe asumir C3 inseguro.
61. Traceability
Blueprint
Identity Domain
Authorization
RBAC
ABAC
Tenant Isolation
Resource Ownership
Delegation
Security Policy Engine
Risk-Aware Security
Event-Driven Architecture
AI-Native Security
Observability
Secure by Design
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
ADR-IDENTITY-058 — Cache TTL Profiles & Operation Criticality Matrix
62. Pending Decisions

Quedan deliberadamente pendientes:

Catálogo definitivo de Resource.
Catálogo definitivo de Action.
Catálogo definitivo de Risk Signals.
Risk scoring definitivo.
Risk confidence model.
Risk freshness TTL.
Catálogo definitivo de C0–C3.
DSL exacto para Criticality Policies.
Sintaxis de minimum_criticality.
Sintaxis de security overrides.
Rule conflict analyzer.
Rule shadow analyzer.
Policy simulation engine.
Policy testing framework.
Criticality Matrix versioning strategy.
Dynamic risk event model.
Risk-to-cache invalidation protocol.
Exact cross-region behavior.
Exact configuration rollout strategy.
Final Security SLA values.
63. Acceptance Criteria
AC-01

Toda autorización recibe una criticidad efectiva antes de determinar cache eligibility.

AC-02

La criticidad base se obtiene de Resource + Action.

AC-03

Risk puede elevar la criticidad.

AC-04

Security State puede elevar la criticidad.

AC-05

Cross-tenant puede elevar a C0.

AC-06

Revocation se mantiene como C0.

AC-07

Platform minimums no pueden ser reducidos.

AC-08

Organization policies pueden endurecer requisitos.

AC-09

Organization policies no pueden debilitar platform security minimums.

AC-10

Las reglas son monotónicas.

AC-11

No se utiliza first-match-wins como mecanismo de seguridad.

AC-12

Los conflictos se detectan durante policy validation.

AC-13

Las reglas shadowed pueden detectarse.

AC-14

Unknown risk no produce downgrade.

AC-15

Risk Engine no puede producir directamente ALLOW.

AC-16

Cambios de risk que eleven criticidad pueden invalidar cache.

AC-17

Criticality policies están versionadas.

AC-18

Cambios críticos son auditables.

AC-19

La criticidad se determina antes de utilizar cache.

AC-20

AI Agents utilizan exactamente el mismo modelo de criticidad y risk escalation.

64. Resultado arquitectónico

Con ADR-059 queda definida la pieza que conecta la Operation Criticality Matrix con el Policy Engine dinámico.

El flujo completo pasa a ser:

                    AUTHORIZATION REQUEST
                             │
                             ▼
                    CONTEXT CONSTRUCTION
                             │
                             ▼
                ┌─────────────────────────┐
                │ Criticality Policy      │
                │ Selection               │
                └────────────┬────────────┘
                             │
                             ▼
                    BASE CRITICALITY
                             │
                             ▼
              ┌────────────────────────────┐
              │ Dynamic Factors             │
              │                            │
              │ Risk                       │
              │ Security State             │
              │ Tenant                     │
              │ MFA                        │
              │ Session                    │
              │ Ownership                  │
              │ Delegation                 │
              └──────────────┬─────────────┘
                             │
                             ▼
                  SECURITY OVERRIDES
                             │
                             ▼
                 PLATFORM MINIMUMS
                             │
                             ▼
                EFFECTIVE CRITICALITY
                    C0 / C1 / C2 / C3
                             │
                             ▼
                 FRESHNESS PROFILE
                    F0 / F1 / F2 / F3
                             │
                             ▼
                  TTL / MAS / SECURITY SLA
                             │
                             ▼
                    CACHE ELIGIBILITY
                             │
                 ┌───────────┴───────────┐
                 ▼                       ▼
             CACHE VALID            FRESH EVAL
                 │                       │
                 └───────────┬───────────┘
                             ▼
                  AUTHORIZATION RUNTIME
                             │
                             ▼
       ALLOW / DENY / CHALLENGE / REAUTH / MFA / REVOKE
Principio arquitectónico definitivo

La criticidad de una autorización es una propiedad dinámica y evaluada centralmente. Las políticas pueden elevar las garantías requeridas según riesgo, contexto y estado de seguridad, pero ninguna política de menor autoridad puede reducir los mínimos de seguridad establecidos por la plataforma.

Esto deja preparado el siguiente paso lógico: ADR-IDENTITY-060 — Authorization Criticality Policy Lifecycle, Simulation, Validation & Safe Deployment, donde podemos definir cómo estas políticas se crean, validan, simulan, prueban, aprueban, versionan, despliegan, activan, hacen rollback y se auditan sin poner en riesgo el Authorization Runtime.
