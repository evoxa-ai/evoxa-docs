ADR-IDENTITY-067 — Authorization Criticality Policy & Test Runtime Contracts

Status: Proposed
Version: 1.0
Date: 2026-09-07
Domain: Identity / Authorization / Security / Policy Engine / Criticality / Testing / Runtime
Priority: Critical
Type: Architecture / Security / Runtime Contract / Testing / Governance

1. Contexto

Los ADR anteriores han definido progresivamente el modelo de autorización y su mecanismo de verificación:

ADR-045: modelo de Authorization Policy y evaluación.
ADR-046: lenguaje, schema y governance de policies.
ADR-048: DSL y expression language.
ADR-049: runtime de evaluación de autorización.
ADR-053–057: cache, invalidation, consistency y freshness.
ADR-058: matriz de criticidad y operation criticality.
ADR-059: Authorization Criticality Policy Model y dynamic risk escalation.
ADR-062: catálogo de tests, Golden Scenarios e invariantes.
ADR-063: Test Execution Engine.
ADR-064: modelo machine-readable y evidence schema.
ADR-065: schema versioning, compatibility y migration.
ADR-066: Schema Registry, contract validation y governance gate.

Sin embargo, todavía falta definir formalmente el contrato de ejecución entre:

Authorization Policy
        ↓
Policy Engine
        ↓
Authorization Runtime
        ↓
Criticality Evaluation
        ↓
Freshness / Cache
        ↓
Test Runtime
        ↓
Expected Result
        ↓
Evidence

Sin este contrato existe riesgo de que:

el Policy Engine produzca una estructura y el Runtime espere otra;
el Test Engine interprete de manera diferente una Criticality Policy;
los Golden Scenarios no sean ejecutables de forma determinista;
una diferencia entre runtime de producción y runtime de testing quede oculta;
Criticality sea confundida con Authorization Decision;
una prueba pueda aprobarse utilizando reglas distintas de las utilizadas en producción.

Este ADR establece los contratos normativos entre esos componentes.

2. Problema

Necesitamos garantizar que:

El mismo modelo de seguridad utilizado para tomar decisiones reales pueda ser evaluado de forma reproducible por el sistema de testing.

El problema puede representarse así:

                    POLICY
                      │
                      ▼
             ┌─────────────────┐
             │ Policy Engine   │
             └────────┬────────┘
                      │
                      ▼
             ┌─────────────────┐
             │ Criticality     │
             │ Evaluation      │
             └────────┬────────┘
                      │
                      ▼
             ┌─────────────────┐
             │ Authorization   │
             │ Runtime         │
             └────────┬────────┘
                      │
                      ▼
                  Decision

Mientras que el testing necesita:

Scenario
   ↓
Input Context
   ↓
Policy Version
   ↓
Runtime Version
   ↓
Evaluation
   ↓
Actual Result
   ↓
Expected Result
   ↓
Comparison

Ambos caminos deben compartir contratos formales.

3. Decisión

Se establece un modelo de Runtime Contracts para Authorization Criticality.

Estos contratos serán:

explícitos;
versionados;
tipados;
deterministas;
tenant-aware;
security-sensitive;
compatibles con el Schema Registry;
independientes del lenguaje de programación;
reproducibles;
auditables.

El contrato será común conceptualmente para:

Production Authorization Runtime
Test Execution Runtime
Simulation Runtime
Shadow Validation Runtime

El Test Runtime no podrá implementar una lógica de autorización diferente a la del runtime autorizado por la arquitectura.

4. Principio fundamental

Se establece:

Test Runtime ≠ Production Runtime

pero:

Test Runtime debe evaluar los mismos contratos y reglas normativas del Production Authorization Runtime.

Es decir:

                  Shared Contracts
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
     Production Runtime        Test Runtime
             │                       │
             ▼                       ▼
        Actual Decision         Test Result

Esto permite mantener separación operacional sin duplicar la semántica de seguridad.

5. Separación de responsabilidades
5.1 Policy Engine

Es responsable de:

seleccionar policies;
validar policies;
evaluar condiciones;
resolver precedence;
producir policy results;
producir Criticality Policy result.

No ejecuta operaciones de negocio.

5.2 Authorization Runtime

Es responsable de:

validar request;
validar actor;
validar autenticación;
validar tenant;
construir contexto;
ejecutar autorización;
integrar RBAC;
ownership;
ABAC;
delegation;
Policy Engine;
Criticality;
freshness;
obligations;
construir Decision.
5.3 Criticality Engine

Es responsable de:

Base Criticality
+
Dynamic Risk
+
Security State
+
Tenant
+
Delegation
+
MFA
+
Context
+
Platform Minimum

y producir:

Effective Criticality

No puede producir por sí solo:

ALLOW
DENY

porque:

Criticality ≠ Authorization Decision.

5.4 Test Execution Engine

Es responsable de:

ejecutar escenarios;
proporcionar contextos controlados;
invocar el runtime;
capturar resultados;
comparar Expected vs Actual;
ejecutar invariantes;
evaluar regresiones;
generar evidence.

No puede modificar:

policies;
platform minimums;
invariants;
security semantics.
6. Runtime Contract Layers

Se establecen cinco contratos principales:

RC-01 Request Contract
RC-02 Authorization Context Contract
RC-03 Policy Evaluation Contract
RC-04 Criticality Result Contract
RC-05 Authorization Decision Contract

Y contratos complementarios:

RC-06 Freshness Contract
RC-07 Cache Eligibility Contract
RC-08 Test Execution Contract
RC-09 Evidence Contract
RC-10 Error Contract
7. RC-01 — Authorization Request Contract

El request debe representar como mínimo:

request_id
actor
organization
membership
operation
resource
action
authentication
session
mfa
delegation
environment
risk_context
timestamp
correlation

Conceptualmente:

{
  "request_id": "...",
  "actor": {},
  "organization": {},
  "membership": {},
  "resource": {},
  "action": "...",
  "authentication": {},
  "session": {},
  "mfa": {},
  "delegation": {},
  "environment": {},
  "risk": {},
  "timestamp": "...",
  "correlation": {}
}

La estructura definitiva deberá derivarse posteriormente en el ESP/API Contract.

8. Request Identity

Cada request debe tener una identidad única.

request_id
correlation_id
trace_id

Cuando corresponda:

causation_id

Esto permitirá relacionar:

Request
→ Policy Evaluation
→ Criticality
→ Decision
→ Audit
→ Test Execution
→ Evidence
9. RC-02 — Authorization Context Contract

El Authorization Context será una representación inmutable del contexto utilizado durante la evaluación.

Debe contener únicamente información autorizada y necesaria.

Categorías:

Actor
Organization
Membership
Authentication
Session
MFA
Request
Resource
Ownership
Delegation
Environment
Risk
Security State
Freshness
10. Context Immutability

Una vez construido:

AuthorizationContext

no debe modificarse durante la evaluación.

Si cambia una condición de seguridad:

Nuevo Context

deberá ser construido y evaluado nuevamente.

Esto evita que:

Policy A

sea evaluada con un contexto y:

Policy B

con otro contexto no registrado.

11. Tenant Context

El tenant/organization debe formar parte explícita del contexto.

El Runtime debe verificar:

actor
membership
organization
resource

antes de permitir una evaluación que pudiera cruzar límites de tenant.

Si existe incertidumbre:

Tenant = UNKNOWN

el resultado debe ser:

DENY

o una condición de seguridad equivalente que nunca permita un ALLOW inseguro.

12. RC-03 — Policy Evaluation Contract

El Policy Engine recibirá:

PolicyEvaluationRequest

y devolverá:

PolicyEvaluationResult

Conceptualmente:

PolicyEvaluationRequest
    ↓
Policy Selection
    ↓
Policy Evaluation
    ↓
PolicyEvaluationResult
13. PolicyEvaluationRequest

Debe identificar:

policy_set_version
policy_ids
policy_scope
authorization_context
runtime_version
evaluation_time

Cuando corresponda:

simulation_mode
test_mode

El test_mode nunca podrá cambiar las reglas de seguridad.

14. PolicyEvaluationResult

El resultado conceptual será:

MATCH
NO_MATCH
INDETERMINATE
ERROR

y deberá incluir:

policy_id
policy_version
effect
matched
reason_code
obligations
evaluation_metadata
15. INDETERMINATE y ERROR

Un resultado:

INDETERMINATE

o:

ERROR

no podrá transformarse automáticamente en:

ALLOW

Se mantiene:

Unknown Is Not Safe Allow.

y:

Error Is Not Safe Allow.

16. RC-04 — Criticality Evaluation Contract

El Criticality Engine recibirá un contexto y policies aplicables.

Entrada conceptual:

CriticalityEvaluationRequest

Salida:

CriticalityEvaluationResult
17. CriticalityEvaluationRequest

Debe contener como mínimo:

resource
action
actor
organization
membership
security_state
risk
mfa
session
delegation
ownership
environment
policy_version
runtime_version
18. CriticalityEvaluationResult

Debe devolver:

base_criticality
dynamic_criticality
effective_criticality
freshness_class
cacheability
security_overrides
platform_minimum
policy_version
reason_codes

Ejemplo conceptual:

{
  "base_criticality": "C2",
  "dynamic_criticality": "C0",
  "effective_criticality": "C0",
  "freshness_class": "F0",
  "cacheable": false
}
19. Criticality Calculation

Se mantiene la regla establecida:

Effective Criticality =
MAX(
  Base,
  Dynamic Risk,
  Context,
  Security State,
  Tenant,
  Delegation,
  MFA,
  Platform Minimum
)

La función debe ser monotónica:

C3 → C2 → C1 → C0

Nunca:

C0 → C1

por efecto de un contexto menos seguro.

20. Criticality Result ≠ Permission

Un resultado:

C0

no significa:

ALLOW

y:

C3

no significa:

DENY

Ejemplo:

User without permission
+
C3
=
DENY

y:

User with permission
+
C0
=
REQUIRE_MFA / REAUTHENTICATE / DENY

según las políticas aplicables.

21. RC-05 — Authorization Decision Contract

El Authorization Runtime producirá:

AuthorizationDecision

Las decisiones permitidas son:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE
22. Decision Structure

Conceptualmente:

{
  "decision": "REQUIRE_MFA",
  "criticality": "C0",
  "freshness": "F0",
  "policy_version": "...",
  "reason_codes": [],
  "obligations": [],
  "runtime_version": "..."
}

La respuesta final deberá evitar revelar información sensible innecesaria.

23. Decision Precedence

Se mantiene la precedencia arquitectónica:

Regulatory / Mandatory Security
        ↓
Platform Security Minimum
        ↓
Security State Override
        ↓
Tenant Isolation
        ↓
Resource Security Policy
        ↓
Dynamic Risk
        ↓
Context
        ↓
Organization Policy
        ↓
Default Operation Matrix

Las reglas superiores no pueden ser debilitadas por reglas inferiores.

24. DENY Precedence

Dentro del mismo nivel:

DENY > ALLOW

La existencia de un ALLOW no elimina un DENY de igual o mayor precedencia.

25. Security Guards

Antes de producir:

ALLOW

deben haberse evaluado los mandatory security guards.

Ejemplos:

Tenant Isolation
Account State
Membership State
Session State
MFA requirement
Delegation boundary
Security Policy
Platform Minimum
26. RC-06 — Freshness Contract

El resultado de autorización deberá incluir información suficiente para determinar su freshness.

Conceptualmente:

freshness_class
security_epoch
policy_version
entity_versions
context_fingerprint
cache_generation
evaluated_at
expires_at
27. Freshness Classes

Se mantienen:

F0 — Authoritative
F1 — Security Validated
F2 — Controlled Eventual
F3 — Restricted Degraded

Y los principios de ADR-057:

F0 → no stale ALLOW basado solamente en cache
F1 → requiere validaciones de seguridad
F2 → operaciones de menor riesgo
F3 → degradación restringida
28. Criticality → Freshness

La Criticality determina la exigencia mínima de freshness.

Conceptualmente:

Criticality	Freshness
C0	F0
C1	F1
C2	F2-SHORT
C3	F2-MEDIUM

La implementación concreta seguirá los perfiles definidos en ADR-057/058.

29. RC-07 — Cache Eligibility Contract

Antes de utilizar una decisión cacheada:

Criticality
        ↓
Freshness
        ↓
Cache Eligibility

El sistema deberá determinar si esa operación puede utilizar cache.

Para C0:

Cached ALLOW

no será suficiente por sí solo.

30. Cache Context Binding

Una decisión cacheada debe estar vinculada como mínimo conceptualmente a:

tenant
actor
membership
resource
action
policy_version
security_epoch
entity_versions
context_fingerprint
cache_generation

Un mismatch invalida la entrada.

31. RC-08 — Test Execution Contract

El Test Execution Engine deberá ejecutar un escenario mediante el mismo contrato conceptual utilizado por Authorization Runtime.

Flujo:

Scenario
   ↓
Input Context
   ↓
Build Authorization Request
   ↓
Authorization Runtime
   ↓
Actual Result
   ↓
Invariant Evaluation
   ↓
Expected vs Actual
   ↓
Test Result
32. Test Runtime Modes

Se establecen:

GOLDEN
INVARIANT
SCENARIO
GENERATED
PROPERTY_BASED
BOUNDARY
DIFFERENTIAL
CHAOS
RECOVERY

Cada modo utiliza los mismos contratos normativos.

33. Test Isolation

El Test Runtime deberá ser aislado.

No podrá:

modificar usuarios reales;
modificar tenants reales;
cambiar policies activas;
alterar platform minimums;
modificar Security Invariants;
alterar cache de producción.
34. Determinism

Una ejecución deberá ser reproducible utilizando:

policy_version
runtime_version
schema_version
catalog_version
scenario_version
invariant_version
generator_version
seed
input_context
evaluation_time

Cuando el tiempo sea relevante, deberá utilizarse un tiempo controlado/reproducible en el entorno de testing.

35. Runtime Version

El runtime_version será obligatorio en:

TestExecution
ScenarioExecution
Evidence
Baseline
RegressionResult

Esto permite distinguir:

Policy change

de:

Runtime behavior change
36. Differential Testing

El Test Engine podrá comparar:

Baseline Runtime
        vs
Candidate Runtime

utilizando:

same policy
same context
same scenario
same schema

para detectar cambios de comportamiento.

37. Expected Result Contract

El Expected Result debe poder representar como mínimo:

expected_decision
expected_criticality
expected_freshness
expected_cacheability
expected_security_behavior
expected_invariants

Pero se mantiene la separación:

Expected Criticality
        ≠
Expected Authorization Decision
38. Actual Result Contract

El Actual Result deberá incluir:

decision
criticality
freshness
cacheability
policy_versions
runtime_version
reason_codes
invariants
obligations

y metadata suficiente para reproducibilidad.

39. Result Comparison

El comparador deberá evaluar independientemente:

Decision
Criticality
Freshness
Cacheability
Security Behavior
Invariants

Ejemplo:

Expected:
C0 / F0 / REQUIRE_MFA

Actual:
C1 / F1 / ALLOW

Esto no es un simple mismatch.

Debe clasificarse como:

SECURITY_REGRESSION
40. Security Regression Rules

Como mínimo deberá bloquearse cuando:

C0 → C1
F0 → F1
DENY → ALLOW
REQUIRE_MFA → ALLOW
REAUTHENTICATE → ALLOW
Cross-tenant → ALLOW
Revoked → ALLOW
Unknown → ALLOW
Error → ALLOW

Estas reglas derivan de las invariantes establecidas en ADR-062 y de la política de criticidad/freshness.

41. Evidence Contract

Cada ejecución deberá generar evidencia suficiente para reproducibilidad.

Debe conservar:

execution_id
scenario_id
schema_version
policy_version
runtime_version
catalog_version
invariant_version
generator_version
input_hash
expected_hash
actual_hash
result
timestamp
environment
trace_reference
integrity

No deberá almacenar:

passwords;
refresh tokens;
MFA secrets;
API secrets;
private keys;
datos sensibles innecesarios.
42. Input Hash

El contexto utilizado para una prueba deberá poder identificarse mediante hash.

Conceptualmente:

input_hash =
HASH(canonical(input_context))

Esto permite comprobar que dos ejecuciones utilizaron el mismo input lógico sin almacenar necesariamente todo el contexto sensible.

43. Result Hash

Cuando corresponda también se podrá generar:

expected_hash
actual_hash

utilizando representación canónica.

Esto facilita:

comparación;
integridad;
reproducibilidad;
detección de drift.
44. RC-09 — Error Contract

Los errores deberán estar estructurados.

Conceptualmente:

{
  "code": "...",
  "message": "...",
  "correlation_id": "...",
  "category": "...",
  "security_impact": "..."
}

Categorías conceptuales:

VALIDATION_ERROR
AUTHENTICATION_ERROR
TENANT_ERROR
POLICY_ERROR
CRITICALITY_ERROR
FRESHNESS_ERROR
CACHE_ERROR
RUNTIME_ERROR
SCHEMA_ERROR
INTERNAL_ERROR
45. Error Security Behavior

Los errores relacionados con seguridad no podrán degradar a:

ALLOW

Ejemplo:

Policy Engine unavailable

no debe producir:

ALLOW

sino un comportamiento fail-closed según la criticidad y operación.

46. Test vs Simulation

Se diferencian explícitamente:

Test

Busca determinar si el comportamiento cumple una expectativa.

Simulation

Busca analizar el comportamiento hipotético de una policy/cambio.

Una simulación:

NO

puede activar una policy.

Tampoco puede modificar producción.

47. Shadow Runtime

Se permitirá un modo:

Shadow Evaluation

donde una decisión pueda evaluarse de forma no destructiva para comparar:

Current Runtime
vs
Candidate Runtime

El resultado shadow nunca deberá reemplazar directamente la decisión productiva.

48. Contract Compatibility

Los contratos de runtime tendrán su propio versionado.

Conceptualmente:

AuthorizationRequestContract v1.x
AuthorizationDecisionContract v1.x
CriticalityResultContract v1.x
TestExecutionContract v1.x
EvidenceContract v1.x

Los cambios seguirán las reglas del Schema Registry y ADR-065/066.

49. Contract Version Independence

Debe distinguirse:

Schema Version
Policy Version
Runtime Version
Contract Version

Por ejemplo:

Policy:       4.2.0
Runtime:      2.8.1
Contract:     1.3.0
Schema:       1.5.0

Ninguna de estas versiones sustituye a otra.

50. Runtime Compatibility Matrix

Antes de ejecutar una combinación:

Policy
+
Runtime
+
Contract
+
Schema
+
Catalog

deberá verificarse compatibilidad.

Conceptualmente:

Policy 4.x
     │
     ├── Runtime 2.x
     ├── Contract 1.x
     └── Schema 1.x

Si la combinación no está soportada:

EXECUTION_BLOCKED
51. Governance Gate

Un cambio en Runtime Contract deberá activar:

Schema Diff
Contract Diff
Compatibility Analysis
Security Impact Analysis
Golden Tests
Regression Tests
Invariant Tests
Approval

Los cambios security-sensitive no podrán aprobarse únicamente porque los tests funcionales sean exitosos.

52. Contract Testing Pipeline

El pipeline recomendado será:

Contract Change
      ↓
Schema Validation
      ↓
Contract Validation
      ↓
Compatibility Analysis
      ↓
Security Analysis
      ↓
Golden Scenarios
      ↓
Security Invariants
      ↓
Regression
      ↓
Mutation Tests
      ↓
Governance Gate
      ↓
APPROVE / BLOCK
53. Mutation Testing

El sistema deberá comprobar que los tests detectan cambios peligrosos.

Mutaciones mínimas:

C0 → C1
F0 → F1
DENY → ALLOW
Remove Tenant Guard
Remove MFA Guard
Lower Risk
Ignore Security State
MAX → MIN
Ignore Delegation Boundary
Ignore Session Revocation

Si una mutación peligrosa sobrevive:

MUTATION_SURVIVED

y el gate deberá bloquear cuando corresponda.

54. Property-Based Runtime Testing

Se deberán mantener propiedades como:

P1

Aumentar el riesgo nunca debe reducir Criticality.

Risk HIGH → Risk CRITICAL

Criticality(CRITICAL)
>=
Criticality(HIGH)
P2

Un tenant mismatch nunca puede producir ALLOW.

P3

Eliminar un permiso nunca debe aumentar privilegios.

P4

Revocar MFA nunca puede producir menor exigencia de seguridad.

P5

Revocar una sesión nunca debe aumentar autorización.

55. Concurrency Contract

El Runtime debe comportarse correctamente bajo concurrencia.

Escenarios:

Concurrent Policy Update
Concurrent Membership Revocation
Concurrent Session Revocation
Concurrent MFA Change
Concurrent Refresh Reuse
Concurrent Cache Invalidation

El Test Runtime deberá poder reproducir escenarios de concurrencia controlados.

56. Time Contract

La evaluación deberá utilizar una noción explícita de tiempo:

evaluation_time

Esto es necesario para:

policy effective_from;
effective_until;
session expiry;
MFA evidence;
trusted device TTL;
delegation expiry;
cache TTL;
freshness.

El Test Runtime debe poder congelar o controlar el tiempo cuando el escenario lo requiera.

57. Risk Contract

Risk será un input contextual.

Debe poder representar conceptualmente:

risk_level
risk_score
confidence
source
evaluated_at

El Risk Engine no podrá directamente emitir:

ALLOW

ni:

DENY

Su función es proporcionar contexto para Policy/Criticality.

58. MFA Contract

El runtime deberá poder representar:

mfa_required
mfa_verified
mfa_factor
mfa_assurance
mfa_verified_at
mfa_evidence_expiry

La evidencia MFA deberá respetar freshness.

MFA expirado no puede ser considerado evidencia válida simplemente porque el usuario la presentó anteriormente.

59. Session Contract

El contexto de sesión deberá permitir evaluar:

session_id
session_state
authenticated_at
last_activity
authentication_method
mfa_state
revoked

Una sesión revocada no puede producir un ALLOW basado únicamente en cache antigua.

60. Delegation Contract

El contexto de delegation deberá incluir conceptualmente:

delegation_id
delegator
delegatee
scope
resource
actions
tenant
valid_from
valid_until
revoked
depth

El Test Runtime deberá comprobar:

delegated_privilege <= delegator_effective_privilege

Nunca podrá existir privilege amplification.

61. Ownership Contract

Ownership deberá poder expresarse de forma determinista.

Ejemplos:

OWNER
NOT_OWNER
UNKNOWN

UNKNOWN no podrá convertirse en OWNER para producir ALLOW.

62. Security State Contract

Debe existir representación explícita para estados críticos:

ACTIVE
PENDING
SUSPENDED
DISABLED
REVOKED
EXPIRED

La interpretación exacta dependerá del aggregate.

Security State tiene precedencia sobre condiciones menos restrictivas cuando corresponda.

63. Observability Contract

Cada evaluación crítica debe poder relacionarse con:

trace_id
correlation_id
request_id
policy_version
runtime_version
criticality
decision

Esto permitirá investigar:

¿Por qué se permitió?

o:

¿Por qué se rechazó?

sin revelar secretos.

64. Deterministic Replay

El sistema deberá permitir:

Recorded Scenario
       ↓
Same Versions
       ↓
Same Context
       ↓
Replay
       ↓
Expected Same Result

El replay no deberá producir efectos de negocio.

Será:

non-destructive
auditable
idempotent
65. AI Runtime Testing

Las evaluaciones realizadas por agentes AI deberán utilizar exactamente estos contratos.

Un agente AI no tendrá un:

Authorization Runtime especial

ni:

Criticality bypass

La identidad AI seguirá:

RBAC
+
ABAC
+
Tenant
+
Ownership
+
Delegation
+
Policy
+
Criticality
+
Freshness
66. AI Test Generation

La IA podrá generar:

escenarios;
combinaciones;
casos frontera;
mutation candidates;
property tests;
análisis de regresión.

Pero no podrá:

cambiar Expected Security Behavior sin governance;
eliminar invariantes;
reducir platform minimum;
aprobar una regresión;
activar un contrato.
67. Seguridad del Test Runtime

El Test Runtime debe considerarse componente de seguridad.

Por lo tanto:

no puede acceder a secretos de producción;
no puede modificar políticas activas;
no puede modificar platform minimums;
no puede emitir credenciales;
no puede ejecutar operaciones de negocio reales durante tests;
debe utilizar identidades controladas;
debe quedar auditado.
68. Performance Contract

El runtime de autorización deberá mantener los objetivos establecidos anteriormente.

Como referencia:

Authorization evaluation:
< 50 ms objetivo

cuando el escenario sea compatible con evaluación local/cacheada.

El testing no deberá interpretar automáticamente una degradación de performance como regresión funcional, pero sí debe registrarla.

69. Contract Performance Tests

Se deberán probar:

Baseline
Normal load
High cardinality
Policy complexity
Large context
Cache miss
Cache hit
Redis unavailable
Event lag
PostgreSQL fallback
Concurrent requests
70. Failure Contract

El Runtime debe tener comportamiento definido ante:

Policy Engine unavailable
Risk Engine unavailable
MFA state unavailable
Session state unavailable
Redis unavailable
Event Platform unavailable
Schema Registry unavailable
Version/Epoch store unavailable
PostgreSQL unavailable

La respuesta dependerá de la criticidad, pero:

Ninguna falla podrá crear una elevación de privilegios.

71. Runtime Contract Security Invariants

Se establecen:

INV-RUNTIME-001

Criticality nunca concede permisos por sí sola.

INV-RUNTIME-002

Unknown no puede producir ALLOW seguro.

INV-RUNTIME-003

Error no puede producir ALLOW seguro.

INV-RUNTIME-004

Tenant mismatch nunca puede producir ALLOW.

INV-RUNTIME-005

C0 no puede degradarse silenciosamente.

INV-RUNTIME-006

F0 no puede degradarse silenciosamente.

INV-RUNTIME-007

Policy version debe estar identificada.

INV-RUNTIME-008

Runtime version debe estar identificada en evidencia.

INV-RUNTIME-009

Test Runtime y Production Runtime utilizan los mismos contratos normativos.

INV-RUNTIME-010

Test mode no puede reducir seguridad.

INV-RUNTIME-011

AI no puede obtener privilegios superiores mediante el Test Runtime.

INV-RUNTIME-012

Delegation no puede producir privilege amplification.

INV-RUNTIME-013

MFA evidence debe respetar freshness.

INV-RUNTIME-014

Revocation debe tener prioridad sobre cache TTL.

INV-RUNTIME-015

Un cambio de policy no puede utilizar silenciosamente una versión antigua.

INV-RUNTIME-016

Una versión incompatible de contrato bloquea la ejecución.

INV-RUNTIME-017

Replay no produce efectos de negocio.

INV-RUNTIME-018

Expected Security Behavior no puede modificarse silenciosamente.

INV-RUNTIME-019

Historical Evidence debe seguir siendo interpretable.

INV-RUNTIME-020

Runtime failures deben fallar closed cuando afectan seguridad.

72. Consecuencias positivas

Esta arquitectura permite:

contrato común entre producción y testing;
eliminación de ambigüedad;
reproducibilidad;
differential testing;
Golden testing;
property testing;
mutation testing;
trazabilidad;
detección de regresiones;
validación de Criticality;
validación de Freshness;
control de tenant isolation;
integración segura de IA.

Además, prepara directamente la transición hacia las Engineering Specifications.

73. Costos

Se añade complejidad mediante:

contratos versionados;
compatibility matrix;
contract testing;
runtime metadata;
deterministic replay;
test isolation;
evidence hashing;
governance.

Sin embargo, esta complejidad es necesaria porque estamos definiendo el comportamiento de seguridad que posteriormente utilizarán:

Web
Mobile
API
AI Agents
Internal Services
External Integrations
74. Alternativas rechazadas
A. Crear un Runtime exclusivo para Testing

Rechazado.

Podría producir falsos positivos:

Test PASS
Production FAIL
B. Duplicar la lógica del Policy Engine

Rechazado.

Generaría divergencia.

C. Permitir que Criticality determine ALLOW

Rechazado.

Criticality y Authorization Decision son conceptos distintos.

D. Ignorar Runtime Version

Rechazado.

No sería posible distinguir correctamente:

Policy Regression

de:

Runtime Regression
E. Permitir fallback ALLOW ante errores

Rechazado absolutamente.

75. No negociables
Existe un contrato formal entre Policy Engine y Runtime.
Existe un contrato formal entre Runtime y Test Engine.
Los contratos están versionados.
Criticality no concede autorización.
Test Runtime no tiene semántica de seguridad propia.
Production y Test Runtime comparten contratos normativos.
Tenant es obligatorio.
Context es inmutable durante una evaluación.
Policy Version es identificable.
Runtime Version es identificable.
Unknown no puede producir ALLOW.
Error no puede producir ALLOW.
C0 no puede degradarse silenciosamente.
F0 no puede degradarse silenciosamente.
Revocation tiene prioridad sobre TTL.
MFA freshness debe respetarse.
Delegation no puede ampliar privilegios.
AI usa el mismo modelo.
Replay no produce efectos de negocio.
Evidence es reproducible.
Contract incompatibility bloquea ejecución.
Security regressions bloquean governance.
Test mode no puede debilitar seguridad.
Platform minimum no puede modificarse mediante testing.
Active contract versions son inmutables.
Los cambios security-sensitive requieren aprobación.
76. Traceability
Blueprint
Identity
Authorization
Security Policy Engine
Testing
Security
Eventing
Observability
AI Governance
Implementation Stories
IS-006 — Authorization & Permission Evaluation
IS-011 — Security Policy Engine
IS-012 — Audit & Security Events
IS-013 — Domain Events & Outbox
IS-014 — Identity API Contracts
IS-016 — Identity Testing & Quality
IS-017 — Identity Runtime & Deployment
ADRs
ADR-045 — Authorization Policy Model
ADR-046 — Policy Language & Governance
ADR-048 — Policy DSL
ADR-049 — Authorization Evaluation Runtime
ADR-050–057 — Cache & Freshness
ADR-058 — Criticality Matrix
ADR-059 — Criticality Policy Model
ADR-062 — Test Catalog & Invariants
ADR-063 — Test Execution Engine
ADR-064 — Test Data & Evidence
ADR-065 — Schema Versioning & Migration
ADR-066 — Schema Registry & Governance
77. Dependencias

Este ADR depende de:

Authorization Policy Engine
Criticality Policy Engine
Authorization Runtime
Security Policy Engine
Risk Context
MFA Context
Session Context
Delegation Context
Tenant Context
Freshness Engine
Cache Strategy
Schema Registry
Test Execution Engine
Evidence System
Audit
Observability
CI/CD
Governance
78. Decisiones pendientes

Este ADR establece los contratos arquitectónicos, pero todavía debemos concretar:

78.1 Exact schemas

Definir los schemas machine-readable definitivos para:

AuthorizationRequest
AuthorizationContext
PolicyEvaluationRequest
PolicyEvaluationResult
CriticalityEvaluationRequest
CriticalityEvaluationResult
AuthorizationDecision
FreshnessResult
CacheEligibility
TestExecution
Evidence
78.2 Exact enums

Cerrar definitivamente:

Criticality
Freshness
Decision
PolicyResult
Error
SecurityState
Risk
MFAState
SessionState
78.3 Canonicalization

Definir exactamente cómo se calcularán:

input_hash
expected_hash
actual_hash
schema_hash
78.4 Contract storage

Definir dónde vivirán:

Contract Definitions
Schema Registry
Version Metadata
Compatibility Matrix
78.5 Runtime API

Convertir estos contratos conceptuales en contratos API/Service Interfaces definitivos.

78.6 Test adapter

Definir cómo el Test Execution Engine invocará el Authorization Runtime sin duplicar lógica.

78.7 Production/Test parity

Definir exactamente qué componentes serán compartidos y cuáles serán adapters específicos.

79. Acceptance Criteria
AC-01

Existe un Authorization Request Contract versionado.

AC-02

Existe un Authorization Context Contract versionado.

AC-03

Existe un Policy Evaluation Contract.

AC-04

Existe un Criticality Evaluation Contract.

AC-05

Existe un Authorization Decision Contract.

AC-06

Criticality está separada de Authorization Decision.

AC-07

El tenant forma parte del contexto.

AC-08

El contexto es inmutable durante una evaluación.

AC-09

Unknown no puede producir ALLOW.

AC-10

Error no puede producir ALLOW.

AC-11

C0 no puede degradarse silenciosamente.

AC-12

F0 no puede degradarse silenciosamente.

AC-13

Cache eligibility depende de Criticality/Freshness.

AC-14

Test Runtime utiliza los mismos contratos normativos que Production Runtime.

AC-15

Test mode no puede modificar las reglas de seguridad.

AC-16

Policy Version queda registrada.

AC-17

Runtime Version queda registrada.

AC-18

Schema Version queda registrada.

AC-19

Differential testing puede comparar runtimes.

AC-20

Mutation testing puede detectar security downgrades.

AC-21

Replay es determinista y no destructivo.

AC-22

Evidence permite reproducibilidad histórica.

AC-23

Contract incompatibility bloquea ejecución.

AC-24

Security regression bloquea governance.

AC-25

AI no puede modificar ni aprobar security contracts.

80. Resultado arquitectónico

Con ADR-067 cerramos el hueco que existía entre las políticas de autorización y el sistema de testing.

La arquitectura queda:

                 AUTHORIZATION POLICY
                         │
                         ▼
                POLICY EVALUATION
                         │
                         ▼
                CRITICALITY ENGINE
                         │
                         ▼
               FRESHNESS / CACHE
                         │
                         ▼
             AUTHORIZATION RUNTIME
                         │
                         ▼
                DECISION CONTRACT
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
        PRODUCTION              TEST RUNTIME
                                    │
                                    ▼
                              EXPECTED vs ACTUAL
                                    │
                                    ▼
                               INVARIANTS
                                    │
                                    ▼
                                REGRESSION
                                    │
                                    ▼
                               GOVERNANCE

Y, especialmente, queda establecido el principio que nos permitirá avanzar hacia implementación sin crear dos sistemas de seguridad diferentes:

El Test Runtime no inventa una autorización paralela; prueba los mismos contratos normativos que utiliza el Authorization Runtime.

ADR-IDENTITY-067 queda definido como Proposed.

Siguiente ADR recomendado

ADR-IDENTITY-068 — Identity Domain Integration & Module Boundaries

Este será un ADR importante porque empezará a cerrar cómo se divide realmente Identity en módulos internos, sus límites, dependencias y reglas de comunicación. Después de 068–069 y el cierre arquitectónico correspondiente, estaremos en una posición mucho más clara para pasar de ADR → ESP → modelo de datos/API → Implementation Stories → Technical Tasks → código.
