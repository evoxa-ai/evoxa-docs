ADR-IDENTITY-062 — Authorization Criticality Policy Test Catalog, Golden Scenarios & Security Invariants

Status: Proposed
Version: 1.0
Date: 2026-09-07
Domain: Identity / Authorization / Security / Policy Engine / Risk / Testing / Governance
Priority: Critical
Type: Architecture / Security / Testing / Policy Verification

1. Contexto

El ADR-IDENTITY-061 — Authorization Criticality Policy Testing, Regression Detection & Continuous Verification estableció que las Criticality Policies deben someterse a:

Static Analysis;
Unit Testing;
Scenario Testing;
Golden Scenarios;
Regression Testing;
Differential Testing;
Property-Based Testing;
Mutation Testing;
Security Invariant Testing;
Integration Testing;
Performance Testing;
Chaos Testing;
Continuous Verification.

Sin embargo, para implementar este modelo de forma consistente, EVOXA necesita definir qué se considera un escenario obligatorio, cuáles son los comportamientos que nunca pueden cambiar y qué invariantes deben protegerse independientemente de la versión de una policy.

Este ADR establece el Test Catalog, los Golden Scenarios y los Security Invariants para Authorization Criticality.

La intención es que estos elementos se conviertan en una referencia estable para:

Policy Authoring
      ↓
Validation
      ↓
Simulation
      ↓
Regression Testing
      ↓
Approval
      ↓
Deployment
      ↓
Continuous Verification
2. Problema

Sin un catálogo formal, cada nueva policy podría ser probada utilizando escenarios diferentes.

Esto genera el riesgo de que:

una policy pase sus propios tests;
pero rompa una garantía existente;
una nueva regla reduzca accidentalmente C0→C1;
un cambio de risk behavior no sea detectado;
un tenant pueda influenciar otro;
una delegation produzca privilege amplification;
una modificación de MFA reduzca seguridad;
una policy cambie la freshness class sin que el cambio sea detectado;
una regla crítica quede sin cobertura.

Por ello se necesita distinguir claramente entre:

Test Scenario

Caso específico utilizado para probar comportamiento.

Golden Scenario

Caso cuyo resultado esperado representa un comportamiento que EVOXA debe preservar salvo cambio arquitectónico explícitamente aprobado.

Security Invariant

Regla de seguridad que no puede ser violada por una Criticality Policy.

3. Decisión arquitectónica

Se establece un Authorization Criticality Policy Test Catalog central y versionado.

El catálogo estará compuesto por:

Test Catalog
├── Core Criticality Tests
├── Resource/Action Tests
├── Risk Tests
├── Security State Tests
├── MFA Tests
├── Session Tests
├── Tenant Tests
├── Delegation Tests
├── Policy Precedence Tests
├── Freshness Tests
├── Cache Eligibility Tests
├── Boundary Tests
├── Failure/Unknown Tests
├── Regression Tests
├── Property Tests
├── Mutation Tests
├── Performance Tests
└── Chaos/Recovery Tests

Dentro del catálogo existirán:

Golden Scenarios
Security Invariants
Regression Baselines
4. Modelo conceptual

Cada escenario deberá poder representarse conceptualmente como:

CriticalityTestCase
├── id
├── version
├── category
├── description
├── input_context
├── expected_criticality
├── expected_freshness
├── expected_cacheability
├── expected_security_behavior
├── invariants
├── severity
└── golden

Los campos definitivos de persistencia quedan pendientes.

5. Identificación de escenarios

Se establece una convención conceptual:

CTC-<CATEGORY>-<NUMBER>

Ejemplos:

CTC-CORE-001
CTC-RISK-001
CTC-MFA-001
CTC-TENANT-001
CTC-DELEGATION-001

Los Golden Scenarios utilizarán:

GSC-<CATEGORY>-<NUMBER>

Ejemplo:

GSC-SECURITY-001

Los Security Invariants:

INV-CRITICALITY-<NUMBER>
6. Test Catalog Categories
6.1 CORE

Pruebas fundamentales de criticidad.

Debe cubrir:

C0;
C1;
C2;
C3;
MAX combination;
default behavior;
unknown behavior.
6.2 RESOURCE

Pruebas relacionadas con Resource Criticality.

Ejemplo:

resource = user
resource_criticality = C0

Debe verificarse que una política inferior no reduzca ese requisito.

6.3 ACTION

Debe cubrir:

read;
list;
create;
update;
delete;
revoke;
suspend;
security change;
administrative operations.

El catálogo final de acciones deberá derivarse del Resource/Action Registry.

7. Golden Scenario Classification

Los Golden Scenarios se clasificarán en:

G0 — Platform Security
G1 — Tenant Isolation
G2 — Identity Security State
G3 — Authentication / MFA
G4 — Session / Token Security
G5 — Delegation
G6 — Risk Escalation
G7 — Policy Precedence
G8 — Freshness / Cache
G9 — Failure / Unknown

Los escenarios G0 y G1 tendrán carácter especialmente crítico.

8. GSC-SECURITY-001 — Platform Minimum

Scenario:

Resource: user
Action: disable
Platform Minimum: C0
Organization Policy: C2

Expected:

Effective Criticality = C0

Invariant:

Platform Minimum cannot be weakened.

Severity: BLOCKING.

9. GSC-SECURITY-002 — Critical Revocation

Scenario:

Resource: membership
Action: revoke

con requisito platform/security:

C0

Expected:

C0

No policy organizacional podrá convertirla en C1/C2/C3.

10. GSC-SECURITY-003 — Session Revocation
session.revoke

deberá conservar la criticidad mínima establecida para security state changes.

Resultado esperado:

C0

cuando corresponda al catálogo de operaciones críticas.

11. GSC-SECURITY-004 — Refresh Family Revocation
refresh_token.family_revoke

deberá mantener el requisito de seguridad correspondiente a revocación de credenciales/sesiones.

Una policy no podrá reducir este requisito únicamente para permitir mayor cacheability.

12. GSC-SECURITY-005 — MFA Factor Revocation
MFA.factor_revoke

deberá mantener su nivel mínimo de criticidad.

El cambio de una regla de MFA nunca deberá permitir:

C0 → C2

si contradice Platform Minimum.

13. GSC-TENANT-001 — Tenant Mismatch

Context:

actor.tenant = A
resource.tenant = B

Expected authorization behavior:

DENY

y la criticidad no podrá utilizarse para convertir el mismatch en un acceso permitido.

14. GSC-TENANT-002 — Organization Policy Isolation

Una policy:

scope = ORGANIZATION
tenant = A

no deberá modificar el resultado correspondiente a:

tenant = B
15. GSC-TENANT-003 — Cross-Tenant Administrative Access

Una operación cross-tenant deberá considerarse de seguridad elevada.

Debe comprobarse:

tenant context;
actor authority;
platform policy;
explicit exception;
audit requirement.

La ausencia de contexto válido debe producir comportamiento seguro.

16. GSC-SECURITY-006 — Security State Override

Si:

account = SUSPENDED

la policy no puede utilizar un contexto normal para reducir el requisito de seguridad.

El Security State Override tendrá precedencia sobre reglas de menor autoridad.

17. GSC-SECURITY-007 — Account Disabled
account = DISABLED

debe mantener las restricciones de seguridad correspondientes.

No podrá utilizarse una policy de cache para convertir un estado DISABLED en un contexto operativo válido.

18. GSC-RISK-001 — LOW Risk
risk = LOW

deberá producir la criticidad definida por la policy.

No se debe asumir automáticamente C3 si existe una regla superior.

19. GSC-RISK-002 — HIGH Risk Elevation
risk = HIGH

debe poder elevar la criticidad:

C3 → C1

cuando la policy lo establezca.

20. GSC-RISK-003 — CRITICAL Risk
risk = CRITICAL

deberá elevar la criticidad al máximo requerido por las reglas aplicables.

Cuando la policy establece C0:

CRITICAL → C0
21. GSC-RISK-004 — Risk Cannot Downgrade Security

Debe verificarse:

CRITICAL risk

nunca produce un nivel inferior al obtenido con:

LOW risk

cuando las demás condiciones son iguales y la policy define una relación monotónica.

22. GSC-RISK-005 — Unknown Risk
risk = UNKNOWN

no podrá utilizarse para reducir una garantía de seguridad.

Resultado:

No unsafe downgrade
23. GSC-MFA-001 — MFA Verified
MFA = VERIFIED

deberá producir el comportamiento esperado según la policy.

24. GSC-MFA-002 — MFA Missing
MFA = ABSENT

no podrá utilizarse para relajar una operación que requiere MFA.

25. GSC-MFA-003 — MFA Expired
MFA evidence = EXPIRED

deberá ser tratado como evidencia no válida.

La policy no podrá considerar esa evidencia como equivalente a MFA válido.

26. GSC-MFA-004 — MFA Revoked
MFA factor = REVOKED

deberá producir el comportamiento de seguridad correspondiente.

No se permite reutilizar evidencia de MFA invalidada.

27. GSC-MFA-005 — Step-Up MFA

Una operación que eleve su criticidad por contexto deberá poder requerir:

REQUIRE_MFA

o la freshness correspondiente según la policy.

28. GSC-SESSION-001 — Active Session
session = ACTIVE

deberá producir el comportamiento normal.

29. GSC-SESSION-002 — Revoked Session
session = REVOKED

no deberá permitir una reducción de seguridad basada únicamente en cache.

30. GSC-SESSION-003 — Expired Session
session = EXPIRED

deberá tratarse como estado no válido para operaciones que requieran sesión activa.

31. GSC-SESSION-004 — Refresh Reuse

Un refresh token detectado como reutilizado deberá mantener el comportamiento crítico definido en los ADRs de refresh/session security.

Una Criticality Policy no puede convertir una respuesta de seguridad en una operación normal.

32. GSC-DELEGATION-001 — Valid Delegation

Una delegation válida podrá permitir una operación si:

está vigente;
está dentro del scope;
pertenece al tenant correcto;
el delegador tiene el privilegio requerido.

La criticidad efectiva deberá seguir las reglas superiores.

33. GSC-DELEGATION-002 — Expired Delegation
delegation = EXPIRED

no debe proporcionar privilegios.

34. GSC-DELEGATION-003 — Revoked Delegation
delegation = REVOKED

deberá invalidarse inmediatamente según los requisitos de freshness aplicables.

35. GSC-DELEGATION-004 — Privilege Boundary

Debe mantenerse:

Delegated Privilege
≤
Delegator Effective Privilege

Nunca:

Delegated Privilege
>
Delegator Effective Privilege
36. GSC-DELEGATION-005 — Cross-Tenant Delegation

Una delegation cross-tenant deberá requerir una autorización explícita y válida.

No podrá derivarse implícitamente de una delegation normal.

37. GSC-PRECEDENCE-001 — Platform vs Organization
Platform = C0
Organization = C2

Expected:

C0
38. GSC-PRECEDENCE-002 — Security State vs Risk
Security State = C0
Risk = C2

Expected:

C0
39. GSC-PRECEDENCE-003 — Tenant Isolation

Si existe incertidumbre de tenant:

tenant = UNKNOWN

no podrá producirse un downgrade permisivo.

40. GSC-PRECEDENCE-004 — Multiple Elevations

Ejemplo:

Base = C2
Risk = C1
MFA context = C0
Security state = C0

Expected:

C0

La criticidad efectiva deberá respetar el máximo requisito aplicable.

41. GSC-FRESHNESS-001 — C0 Requires F0

Cuando una operación es C0 y el modelo establece F0:

C0
 ↓
F0

No deberá utilizarse una decisión stale de caché como ALLOW.

42. GSC-FRESHNESS-002 — Security Epoch

Si:

cache.security_epoch < current.security_epoch

la entrada deberá considerarse inválida independientemente de que TTL no haya expirado.

43. GSC-FRESHNESS-003 — Policy Version

Si:

cache.policy_version != active.policy_version

la decisión no deberá utilizarse cuando la diferencia afecta la autorización.

44. GSC-FRESHNESS-004 — TTL Not Sufficient

Una entrada:

pero con:

deberá invalidarse/no utilizarse para un ALLOW crítico.

45. GSC-CACHE-001 — Critical Operation Not Cached as Stale ALLOW

Una operación C0 no podrá utilizar un ALLOW stale únicamente porque la entrada exista en Redis/L1.

46. GSC-CACHE-002 — Stale DENY

Un DENY stale tampoco deberá considerarse eternamente válido.

Si el estado autoritativo cambió y permite la operación, la caché deberá poder invalidarse.

Esto evita convertir la seguridad en una fuente permanente de inconsistencias funcionales.

47. GSC-CACHE-003 — Invalidation Propagation

Una modificación de policy que cambie criticality/freshness deberá propagarse hacia:

L1
L2
Authorization Runtime

según el modelo definido en ADR-053.

48. GSC-CACHE-004 — Out-of-Order Invalidation

Si llega:

version 12

después de:

version 13

el sistema no deberá regresar a:

version 12
49. GSC-CACHE-005 — Duplicate Events

La recepción de dos eventos equivalentes no deberá producir efectos diferentes.

Debe mantenerse:

idempotent
50. Failure Scenarios

El catálogo deberá incluir:

Policy unavailable
Policy malformed
Policy incompatible
Risk unavailable
MFA unavailable
Session state unavailable
Redis unavailable
Event Platform unavailable
Policy consumer unavailable
Version store unavailable
Network partition

El objetivo común será:

No unsafe ALLOW
51. GSC-FAILURE-001 — Unknown Policy

Si no existe una policy aplicable válida:

UNKNOWN

no deberá asumirse:

C3

automáticamente.

Debe utilizarse el comportamiento seguro definido por governance.

52. GSC-FAILURE-002 — Evaluation Error

Si el evaluator falla:

ERROR

el resultado no podrá convertirse en:

ALLOW

mediante fallback inseguro.

53. GSC-FAILURE-003 — Redis Failure

Si Redis no está disponible:

Redis = DOWN

Authorization Runtime deberá continuar utilizando un camino seguro.

Nunca:

Redis unavailable → allow all
54. GSC-FAILURE-004 — Event Platform Failure

Si Event Platform está temporalmente indisponible:

Outbox

deberá preservar la información necesaria para recuperación.

Las operaciones críticas no podrán depender de una propagación eventual insegura.

55. GSC-FAILURE-005 — Policy Corruption

Si:

policy hash != expected hash

la policy no deberá activarse.

56. Security Invariants

Los siguientes invariantes forman parte del contrato arquitectónico.

INV-CRITICALITY-001 — Platform Minimum Immutability
Effective Criticality
≥
Platform Minimum

Una policy inferior nunca puede reducirlo.

INV-CRITICALITY-002 — Security Monotonicity

Un aumento de una condición de riesgo o seguridad no puede producir un downgrade cuando las reglas establecen una elevación monotónica.

INV-CRITICALITY-003 — Tenant Isolation
Tenant mismatch
→
No permissive downgrade
INV-CRITICALITY-004 — Unknown Is Not Safe Allow
UNKNOWN
≠
ALLOW
INV-CRITICALITY-005 — Error Is Not Safe Allow
ERROR
≠
ALLOW
INV-CRITICALITY-006 — Revocation Priority

Una revocación de seguridad deberá tener prioridad sobre una optimización de cache.

REVOCATION
>
CACHE OPTIMIZATION
INV-CRITICALITY-007 — Security Epoch Supersedes TTL
Security Epoch mismatch
>
Valid TTL
INV-CRITICALITY-008 — Policy Version Integrity

Una decisión cacheada bajo una policy incompatible no podrá utilizarse para generar un ALLOW inseguro.

INV-CRITICALITY-009 — Delegation Boundary
Delegated Privilege
≤
Delegator Effective Privilege
INV-CRITICALITY-010 — MFA Evidence Freshness

MFA expired/revoked no puede considerarse MFA válido.

INV-CRITICALITY-011 — No Privilege Amplification

Ninguna combinación de:

role;
delegation;
policy;
risk;
context;

podrá producir privilegios superiores a los permitidos por las reglas superiores.

INV-CRITICALITY-012 — Organization Cannot Weaken Platform
Organization Policy
≤
Platform Security Requirements

en el sentido de que puede endurecer, pero no debilitar.

INV-CRITICALITY-013 — Security State Dominance

Los estados de seguridad críticos deben prevalecer sobre optimizaciones normales.

INV-CRITICALITY-014 — Critical ALLOW Requires Required Freshness

Una operación que requiera F0 no puede autorizarse mediante una entrada stale.

INV-CRITICALITY-015 — No Version Regression

Nunca:

Current Version = N

debe convertirse nuevamente en:

N-1

por eventos atrasados.

INV-CRITICALITY-016 — Idempotent Verification

Repetir una prueba o verificación no debe alterar el estado de autorización.

INV-CRITICALITY-017 — Determinism

Mismo:

Policy
+
Policy Version
+
Context
+
Runtime Version

debe producir el mismo resultado.

INV-CRITICALITY-018 — AI Cannot Reduce Security

La IA no puede generar un resultado que reduzca un Platform Minimum ni modificar un Security Invariant.

INV-CRITICALITY-019 — Criticality Does Not Grant Permission

Criticality determina requisitos de seguridad/freshness.

No debe utilizarse como sustituto de:

RBAC;
ABAC;
ownership;
delegation;
authorization decision.
INV-CRITICALITY-020 — Cache Is Not Source of Truth
PostgreSQL / authoritative state
>
Cache

La caché no puede convertirse en autoridad de seguridad.

57. Property-Based Security Invariants

Los invariantes deberán poder expresarse como propiedades.

Ejemplo:

∀ context:
if platform_minimum(context) = C0
then effective_criticality(context) = C0

Otro:

∀ context:
if tenant_mismatch(context)
then authorization(context) != ALLOW

Otro:

∀ context:
if risk(context) increases
then security_requirement(context)
cannot decrease

Esto permitirá verificar cientos o miles de combinaciones automáticamente.

58. Test Matrix

La matriz conceptual deberá combinar:

Resource
×
Action
×
Risk
×
MFA
×
Session
×
Delegation
×
Tenant
×
Security State
×
Ownership
×
Freshness

Ejemplo simplificado:

Dimensión	Valores
Criticality	C0/C1/C2/C3
Risk	LOW/MEDIUM/HIGH/CRITICAL/UNKNOWN
MFA	ABSENT/PENDING/VERIFIED/EXPIRED/REVOKED
Session	ACTIVE/EXPIRED/REVOKED
Delegation	NONE/VALID/EXPIRED/REVOKED
Tenant	SAME/MISMATCH/UNKNOWN
Security State	NORMAL/SUSPENDED/DISABLED
Freshness	F0/F1/F2/F3/STALE/INVALID
Cache	HIT/MISS/INVALID
Policy	ACTIVE/UNKNOWN/INVALID

La matriz completa será potencialmente enorme.

Por ello, el framework deberá soportar:

combinaciones exhaustivas para subconjuntos críticos;
combinaciones generadas;
pairwise testing;
property-based testing;
escenarios Golden.
59. Test Prioritization

Se establece la siguiente prioridad:

P0 — Security Invariants
P0 — Golden Security Scenarios
P1 — Criticality / Freshness Regression
P1 — Tenant Isolation
P1 — Security State
P1 — MFA / Session / Delegation
P2 — Risk
P2 — Policy Precedence
P2 — Cache
P3 — Performance
P3 — Extended combinations

Las pruebas P0 son obligatorias antes de activation.

60. Golden Scenario Governance

Un Golden Scenario no podrá modificarse silenciosamente.

Para cambiarlo deberá existir:

Change Request
      ↓
Reason
      ↓
Impact Analysis
      ↓
Simulation
      ↓
Approval
      ↓
Golden Version Update

Esto evita que una regresión se "solucione" simplemente cambiando el expected result.

61. Golden Scenario Versioning

Los escenarios deberán estar versionados.

Ejemplo:

GSC-SECURITY-001 v1

Una evolución intencional:

v1 → v2

deberá conservar:

histórico;
motivo;
autor;
aprobador;
fecha;
impacto;
relación con policy version.
62. Regression Baseline

La baseline deberá estar compuesta por:

Active Policy Version
+
Golden Scenario Version
+
Invariant Version
+
Runtime Version

Una comparación será válida solamente si estos elementos están correctamente identificados.

63. Regression Classification

Cada delta deberá clasificarse:

NO_CHANGE
EXPECTED_CHANGE
HARDENING
BEHAVIOR_CHANGE
SECURITY_REGRESSION
INVARIANT_VIOLATION

SECURITY_REGRESSION e INVARIANT_VIOLATION son bloqueantes.

64. Continuous Verification

Después de activation:

ACTIVE POLICY
      ↓
Golden Tests
      ↓
Security Invariants
      ↓
Drift Detection
      ↓
Freshness Verification
      ↓
Runtime Monitoring

La policy deberá continuar siendo verificable durante toda su vida útil.

65. Production Verification

La verificación de producción no deberá ejecutar acciones destructivas.

Debe validar principalmente:

policy version;
policy hash;
evaluation behavior;
freshness;
cache consistency;
invariants;
drift.

Los escenarios destructivos deberán utilizar ambientes controlados.

66. Chaos Verification

El catálogo deberá incorporar chaos tests para:

Redis outage
Event delay
Event duplication
Event reordering
Policy propagation delay
Runtime restart
Database failover
Cache corruption
Network partition
Regional failure

Después de cada escenario deberán verificarse:

No privilege escalation
No cross-tenant access
No unsafe ALLOW
No security invariant violation
67. Performance Verification

Las pruebas deberán garantizar que aumentar el número de policies y escenarios no provoque:

latencias incompatibles;
timeouts;
comportamiento fail-open;
degradación insegura.

La validación deberá mantenerse separada del runtime crítico.

68. Test Artifact

Cada ejecución crítica deberá producir evidencia:

TestExecution
├── test_id
├── policy_id
├── policy_version
├── runtime_version
├── scenario_version
├── invariant_version
├── started_at
├── completed_at
├── result
├── severity
├── diff
└── evidence_reference

Los detalles exactos quedan pendientes.

69. Auditability

Debe ser posible responder:

¿Por qué esta policy fue considerada segura para activación?

La respuesta deberá poder reconstruirse mediante:

Policy Version
+
Validation Result
+
Simulation Result
+
Golden Tests
+
Security Invariants
+
Approval
+
Deployment
70. Consecuencias positivas

Este ADR proporciona:

catálogo estable;
pruebas reproducibles;
Golden Scenarios;
invariantes formales;
detección sistemática de regresiones;
protección contra cambios accidentales;
cobertura de risk/MFA/session/delegation;
cobertura tenant-aware;
integración con cache/freshness;
base para CI/CD;
evidencia auditable;
soporte para continuous verification.
71. Consecuencias negativas

Introduce:

mantenimiento del catálogo;
mantenimiento de Golden Scenarios;
mayor cantidad de tests;
necesidad de versionar expected results;
mayor coste de CI;
necesidad de mantener invariantes;
complejidad de la matriz combinatoria.

La complejidad se considera necesaria debido al impacto de Criticality Policies sobre el security decision path.

72. Alternativas rechazadas
72.1 Tests definidos individualmente por cada equipo

Rechazado.

Podría producir inconsistencias y pérdida de garantías globales.

72.2 Golden Scenarios sin invariants

Rechazado.

Los escenarios concretos no cubren todas las combinaciones posibles.

72.3 Invariants sin escenarios

Rechazado.

Los invariantes son demasiado abstractos para cubrir todo el comportamiento esperado.

72.4 Exhaustive Testing de todas las combinaciones

Rechazado como única estrategia.

La combinación:

Resource × Action × Risk × MFA × Session × Delegation × Tenant × ...

puede crecer exponencialmente.

Se utilizará una combinación de:

Golden;
property-based;
pairwise;
scenario;
regression;
mutation.
72.5 AI-Generated Tests Only

Rechazado.

Los tests críticos deben ser deterministas y gobernados.

73. Non-Negotiables
Debe existir un catálogo central de Criticality Tests.
Deben existir Golden Scenarios.
Deben existir Security Invariants.
Los Golden Scenarios deben estar versionados.
Los invariantes deben ser independientes de una policy específica.
Platform Minimum nunca puede reducirse.
Tenant isolation debe probarse obligatoriamente.
Critical security states deben probarse obligatoriamente.
Risk escalation debe probarse.
MFA debe probarse.
Session security debe probarse.
Delegation boundaries deben probarse.
Freshness debe probarse.
Cache eligibility debe probarse.
Unknown/error no pueden producir unsafe ALLOW.
Differential testing debe formar parte de regression analysis.
Security regressions bloquean activation.
Golden changes requieren governance.
Invariant changes requieren governance superior.
Continuous Verification debe continuar después de deployment.
AI no puede eliminar ni debilitar invariants.
Los tests críticos deben ser reproducibles.
Los resultados deben ser auditables.
Chaos testing debe preservar fail-closed.
El catálogo debe evolucionar junto con el Authorization Model.
74. Trazabilidad
Blueprint
Identity
Authorization
Security Policy Engine
Security
Testing & Quality
Event Architecture
Caching
Observability
AI Governance
Implementation Stories
IS-IDENTITY-006
Authorization & Permission Evaluation

IS-IDENTITY-011
Security Policy Engine

IS-IDENTITY-012
Audit & Security Events

IS-IDENTITY-013
Domain Events & Outbox

IS-IDENTITY-016
Identity Testing & Quality

IS-IDENTITY-017
Identity Runtime & Deployment
ADRs relacionados
ADR-IDENTITY-045
Authorization Policy Model/Evaluation

ADR-IDENTITY-046
Policy Language/Schema/Governance

ADR-IDENTITY-048
Policy DSL & Expression Language

ADR-IDENTITY-049
Policy Evaluation Runtime

ADR-IDENTITY-053
Cache Invalidation Event Model

ADR-IDENTITY-054
Cache Failure & Recovery

ADR-IDENTITY-055
Cache Recovery & Warm-Up

ADR-IDENTITY-056
Cache Consistency & Freshness

ADR-IDENTITY-057
Freshness Classes & Security SLAs

ADR-IDENTITY-058
TTL Profiles & Criticality Matrix

ADR-IDENTITY-059
Criticality Policy Model & Dynamic Risk

ADR-IDENTITY-060
Policy Lifecycle, Simulation & Safe Deployment

ADR-IDENTITY-061
Policy Testing, Regression & Continuous Verification
75. Dependencias

Este ADR depende de:

Authorization Policy Engine;
Criticality Policy Model;
Policy DSL;
Policy Validator;
Authorization Runtime;
Resource Registry;
Action Registry;
Attribute Registry;
Risk Context;
MFA Context;
Session Context;
Delegation Context;
Tenant Context;
Freshness Model;
Cache Model;
Event Platform;
Audit;
CI/CD;
Observability.
76. Decisiones pendientes

Quedan pendientes:

Catálogo completo de resources.
Catálogo completo de actions.
Lista definitiva de Golden Scenarios.
Lista definitiva de Security Invariants.
Versioning scheme de scenarios.
Versioning scheme de invariants.
Pairwise testing algorithm.
Property-based testing framework.
Mutation testing framework.
Exact severity model.
Test artifact storage.
Retention period.
Golden Scenario approval workflow.
Invariant approval workflow.
Runtime verification frequency.
Production shadow sampling.
Automated regression thresholds.
Exact C0/C1/C2/C3 mapping por operation.
Exact F0/F1/F2/F3 mapping.
Complete matrix generation strategy.
Chaos test catalog.
Performance thresholds.
Multi-region test strategy.
CI/CD implementation details.
Formal machine-readable format del catálogo.

Estas decisiones deberán desarrollarse en futuras ADRs/ESPs sin contradecir los invariantes definidos aquí.

77. Acceptance Criteria
AC-01 — Central Catalog

Existe un catálogo central versionado.

AC-02 — Golden Scenarios

Existen escenarios Golden para las categorías críticas.

AC-03 — Security Invariants

Existen invariantes formales de seguridad.

AC-04 — Platform Minimum

El catálogo detecta cualquier intento de downgrade respecto de Platform Minimum.

AC-05 — Tenant

El catálogo detecta cross-tenant security regressions.

AC-06 — Risk

El catálogo cubre LOW/MEDIUM/HIGH/CRITICAL/UNKNOWN.

AC-07 — MFA

El catálogo cubre estados críticos de MFA.

AC-08 — Session

El catálogo cubre estados críticos de Session.

AC-09 — Delegation

El catálogo verifica privilege boundaries.

AC-10 — Freshness

El catálogo detecta freshness regressions.

AC-11 — Cache

El catálogo detecta cache eligibility regressions.

AC-12 — Precedence

El catálogo verifica policy precedence.

AC-13 — Unknown

UNKNOWN no puede producir unsafe ALLOW.

AC-14 — Errors

Evaluation errors no pueden producir unsafe ALLOW.

AC-15 — Differential

Candidate vs baseline produce un resultado reproducible.

AC-16 — Property Testing

Las invariants críticas pueden verificarse sobre múltiples contextos.

AC-17 — Mutation

Las mutaciones críticas conocidas son detectables.

AC-18 — Governance

Golden e invariants no pueden modificarse silenciosamente.

AC-19 — Continuous Verification

La policy activa puede verificarse después de deployment.

AC-20 — Audit

Es posible reconstruir por qué una policy fue considerada segura.

78. Resultado arquitectónico

Con este ADR queda definida la tercera capa de la estrategia de Criticality Policy:

ADR-059
└── Criticality Model
        ↓
ADR-060
└── Lifecycle + Simulation + Safe Deployment
        ↓
ADR-061
└── Testing + Regression + Continuous Verification
        ↓
ADR-062
└── Test Catalog + Golden Scenarios + Security Invariants

El modelo final comienza a tomar esta forma:

                    CRITICALITY POLICY
                           │
                           ▼
                    ┌──────────────┐
                    │   Lifecycle  │
                    └──────┬───────┘
                           ↓
                    ┌──────────────┐
                    │  Validation  │
                    └──────┬───────┘
                           ↓
                    ┌──────────────┐
                    │  Simulation  │
                    └──────┬───────┘
                           ↓
             ┌─────────────┴─────────────┐
             ↓                           ↓
       Golden Scenarios            Security Invariants
             │                           │
             └─────────────┬─────────────┘
                           ↓
                   Regression Engine
                           ↓
                  Security Verification
                           ↓
                        Approval
                           ↓
                    Safe Deployment
                           ↓
                       Runtime
                           ↓
                Continuous Verification

La decisión fundamental de este ADR es:

Los Golden Scenarios representan comportamientos que EVOXA debe preservar; los Security Invariants representan garantías que una Criticality Policy nunca puede violar.

De esta forma, el sistema no depende exclusivamente de que alguien recuerde escribir el test correcto para cada nueva policy. Existe un contrato permanente de seguridad contra el cual todas las futuras versiones deben ser verificadas.

Siguiente ADR recomendado

ADR-IDENTITY-063 — Authorization Criticality Policy Test Execution Engine, Scenario Generation & Regression Evaluation

El siguiente paso lógico sería definir cómo se ejecutará técnicamente este catálogo: motor de escenarios, generación de contextos, ejecución de Golden Scenarios, property-based generation, comparación baseline/candidate, cálculo de deltas, clasificación automática de regresiones y generación de evidencia para CI/CD.
