ADR-IDENTITY-061 — Authorization Criticality Policy Testing, Regression Detection & Continuous Verification

Status: Proposed
Version: 1.0
Date: 2026-09-07
Domain: Identity / Authorization / Security / Policy Engine / Risk / Testing / Governance
Priority: Critical
Type: Architecture / Security / Testing / Continuous Verification

1. Contexto

El ADR-IDENTITY-060 estableció el lifecycle de las Authorization Criticality Policies, incluyendo:

DRAFT;
VALIDATING;
SIMULATION;
APPROVAL_REQUIRED;
APPROVED;
DEPLOYING;
ACTIVE;
SUPERSEDED;
RETIRED;
mecanismos de rollback;
shadow evaluation;
canary deployment;
detección de security regressions.

Sin embargo, validar sintácticamente una política y simular algunos escenarios no garantiza que su comportamiento sea correcto ante todas las combinaciones relevantes de:

actor;
tenant;
membership;
role;
permission;
ownership;
delegation;
MFA;
session;
security state;
resource;
action;
environment;
risk;
policy version;
security epoch;
freshness class.

Por lo tanto, se requiere un mecanismo permanente de Testing + Regression Detection + Continuous Verification.

El objetivo es que una Criticality Policy no solamente sea:

válida

sino también:

demostrablemente segura
+
comportamentalmente consistente
+
compatible con las garantías anteriores
+
verificable continuamente
2. Problema

Una modificación de Criticality Policy puede producir regresiones que no sean evidentes mediante pruebas unitarias simples.

Por ejemplo:

membership.revoke
C0 → C0

puede parecer segura.

Pero una condición nueva podría provocar:

HIGH risk
      ↓
C1

cuando anteriormente:

HIGH risk
      ↓
C0

Esto podría modificar:

freshness;
cache eligibility;
authorization behavior;
security response;
exposición temporal a información desactualizada.

También pueden aparecer regresiones más sutiles:

una regla deja de aplicarse;
una regla queda shadowed;
una condición se vuelve unreachable;
una policy organization-scoped afecta otro tenant;
una policy de riesgo no eleva correctamente;
una revocación deja de ser C0;
una nueva versión acepta un contexto anteriormente rechazado;
una modificación cambia accidentalmente una freshness class;
una nueva combinación de atributos genera una criticidad inesperada.

Por ello, EVOXA necesita una estrategia de pruebas que detecte tanto:

regresiones de seguridad

como:

regresiones de comportamiento.

3. Decisión arquitectónica

Se establece un Authorization Criticality Continuous Verification Framework.

Toda Criticality Policy deberá poder ser evaluada mediante múltiples capas de testing:

Static Validation
       ↓
Unit Tests
       ↓
Policy Tests
       ↓
Scenario Tests
       ↓
Regression Tests
       ↓
Property-Based Tests
       ↓
Differential Tests
       ↓
Security Invariant Tests
       ↓
Integration Tests
       ↓
Performance Tests
       ↓
Chaos / Resilience Tests
       ↓
Continuous Verification

Ninguna política considerada CRITICAL deberá poder activarse si falla una prueba de seguridad obligatoria.

4. Principios
4.1 Test Before Activate

Una política no deberá llegar a ACTIVE si no cumple los controles obligatorios.

Candidate Policy
      ↓
Validation
      ↓
Testing
      ↓
Regression Analysis
      ↓
Approval
      ↓
Deployment
4.2 Security Regression Is a Blocking Failure

Una regresión de seguridad deberá bloquear:

approval;
deployment;
activation;
promotion.

No deberá tratarse simplemente como warning.

SECURITY_REGRESSION
        ↓
       BLOCK
5. Test Pyramid

Se establece una pirámide de pruebas:

                Continuous
                Verification
                     ▲
                Chaos / DR
                     ▲
              Integration Tests
                     ▲
             Regression Suites
                     ▲
              Scenario Tests
                     ▲
             Policy Tests
                     ▲
               Unit Tests
                     ▲
             Static Analysis

La mayor cantidad de pruebas deberá encontrarse en los niveles inferiores, mientras que las pruebas de integración y resiliencia validarán el comportamiento completo.

6. Static Policy Analysis

Antes de ejecutar escenarios deberá realizarse análisis estático.

Debe verificar:

schema;
tipos;
referencias;
operadores;
funciones;
recursos;
acciones;
atributos;
criticality values;
policy scope;
precedence;
contradictions;
unreachable conditions;
shadowed rules;
redundant rules;
invalid freshness profile;
invalid version references.

Resultado conceptual:

STATIC_ANALYSIS
├── PASS
├── WARNING
└── FAIL
7. Policy Unit Tests

Cada componente de una Criticality Policy deberá poder probarse aisladamente.

Ejemplos:

RiskElevationRule
SecurityOverrideRule
TenantRule
DelegationRule
MFAElevationRule
PlatformMinimumRule

Cada prueba debe verificar:

Input Context
      ↓
Rule
      ↓
Expected Criticality
8. Scenario Test Model

Se define un modelo estándar de escenario:

CriticalityTestScenario
├── scenario_id
├── description
├── actor_context
├── tenant_context
├── membership_context
├── session_context
├── mfa_context
├── delegation_context
├── resource
├── action
├── risk_context
├── environment_context
├── expected_criticality
├── expected_freshness
├── expected_cacheability
└── security_invariants

El modelo exacto de persistencia queda pendiente.

9. Golden Scenarios

EVOXA deberá mantener un conjunto de Golden Scenarios.

Estos representan comportamientos que nunca deberían cambiar accidentalmente.

Ejemplos:

user.disable → C0
membership.revoke → C0
session.revoke_all → C0
refresh_token.family_revoke → C0
MFA.factor_revoke → C0
cross_tenant_access → DENY / C0

Los valores concretos deberán evolucionar con el catálogo oficial.

10. Golden Test Suite

Una nueva policy deberá ejecutarse contra:

Current Golden Suite

antes de aprobarse.

Resultado:

PASS
FAIL
SECURITY_REGRESSION

Una modificación de comportamiento intencional deberá requerir actualización explícita del Golden Scenario, con governance y auditoría.

11. Regression Baseline

Cada versión activa deberá poder utilizarse como baseline.

Ejemplo:

ACTIVE v20

Candidate:

v21

Comparación:

v20
 ↓
Regression Engine
 ↓
v21

Debe identificar diferencias de:

criticality;
freshness;
cache eligibility;
security invariants;
tenant behavior;
risk behavior.
12. Differential Testing

El sistema deberá ejecutar:

Current Policy

y:

Candidate Policy

sobre los mismos escenarios.

Ejemplo:

Scenario 001

v20 → C1 / F1
v21 → C2 / F2-SHORT

Resultado:

CRITICALITY DOWNGRADE

El Regression Engine deberá decidir si:

está permitido;
requiere revisión;
constituye regresión;
bloquea activación.
13. Criticality Delta

Cada escenario deberá calcular:

Criticality Delta

Ejemplo:

C3 → C2
C2 → C1
C1 → C0

o:

C0 → C1

Las reducciones de criticidad deberán recibir un nivel de escrutinio superior.

14. Freshness Regression Detection

La criticidad no deberá analizarse aislada.

También deberá comprobarse:

Criticality
      ↓
Freshness Profile

Ejemplo:

v20:
C1 → F1

v21:
C2 → F2-SHORT

Esto deberá identificarse como una posible Freshness Regression.

No basta con comprobar que la criticidad sea aparentemente válida.

15. Cache Eligibility Regression

También deberá comprobarse:

Cacheable
Non-cacheable
Restricted
Fresh-only

Una modificación que transforme:

F0 → F2

debe ser considerada potencialmente crítica.

16. Security Invariants

Se establece un conjunto de invariantes de seguridad que ninguna Criticality Policy podrá violar.

Invariant I-01

Platform Minimum nunca puede ser reducido.

Invariant I-02

Tenant mismatch nunca puede convertirse en un resultado permisivo.

Invariant I-03

Security state crítico debe poder elevar la criticidad.

Invariant I-04

Risk CRITICAL nunca puede provocar un downgrade de seguridad.

Invariant I-05

Delegation nunca puede superar el privilege boundary del delegador.

Invariant I-06

MFA inválido no puede generar una relajación de seguridad.

Invariant I-07

Revocations críticas deben conservar requisitos F0/F1 definidos por governance.

Invariant I-08

Unknown nunca puede utilizarse para reducir criticidad.

Invariant I-09

Una policy de menor autoridad no puede debilitar una superior.

Invariant I-10

AI no puede modificar estos invariantes.

17. Property-Based Testing

Además de escenarios explícitos, se deberá soportar testing basado en propiedades.

En lugar de comprobar únicamente:

Scenario A → C0

se comprueba:

Para cualquier contexto que contenga una condición de seguridad obligatoria C0, la criticidad efectiva no puede ser menor que C0.

Conceptualmente:

∀ context
    SecurityRequirement(context) = C0
        ⇒
    EffectiveCriticality(context) = C0

Esto permite encontrar combinaciones que no fueron escritas manualmente.

18. Monotonicity Testing

Debe verificarse automáticamente la monotonicidad.

Ejemplo:

LOW risk → C2
HIGH risk → C1
CRITICAL risk → C0

Debe cumplirse:

CRITICAL ≥ HIGH ≥ LOW

No debería existir:

LOW → C1
HIGH → C2

cuando las reglas establecen que el aumento de riesgo debe elevar seguridad.

19. Context Mutation Testing

Se deberán modificar progresivamente los atributos del contexto:

MFA verified
      ↓
MFA revoked
risk LOW
      ↓
risk HIGH
session ACTIVE
      ↓
session REVOKED
membership ACTIVE
      ↓
membership REVOKED

El sistema deberá comprobar que los cambios produzcan la elevación o restricción esperada.

20. Tenant Isolation Testing

Se deberá probar sistemáticamente:

Tenant A
Tenant B

con los mismos:

actor;
resource;
action;
policy;
risk.

Una policy scoped a Tenant A no debe modificar el resultado de Tenant B.

Además:

tenant mismatch

deberá producir comportamiento seguro.

21. Delegation Testing

La suite deberá probar:

delegation válida;
delegation expirada;
delegation revocada;
delegation limitada;
delegation insuficiente;
delegation cross-tenant;
delegation con actor de alto riesgo.

Debe verificarse que:

delegation privilege
≤
delegator effective privilege
22. MFA Testing

Se deberá cubrir:

MFA absent
MFA pending
MFA verified
MFA expired
MFA revoked
MFA step-up valid
MFA step-up expired

Las Criticality Policies no podrán utilizar MFA stale o inválido como mecanismo para reducir seguridad.

23. Session Security Testing

Se deberá probar:

ACTIVE
REVOKED
EXPIRED
SUSPENDED

incluyendo:

session revoke;
revoke all;
suspicious session;
refresh reuse;
security reset.

Los estados críticos de sesión deberán conservar los requisitos establecidos por las políticas superiores.

24. Risk Testing

La suite de risk deberá cubrir:

LOW
MEDIUM
HIGH
CRITICAL
UNKNOWN

También deberá probar:

risk score;
confidence;
source;
timestamp;
stale risk;
conflicting risk signals.

Un UNKNOWN no deberá utilizarse para reducir una garantía.

25. Security State Testing

Debe comprobarse el comportamiento ante:

account suspended
account disabled
membership suspended
membership revoked
MFA revoked
session revoked
refresh family revoked
delegation revoked
security incident

Estos estados deberán tener prioridad sobre optimizaciones de caché.

26. Policy Precedence Testing

Se deberá probar la precedencia definida en ADR-059.

Ejemplo:

Platform = C0
Organization = C2

Resultado:

C0

Otro ejemplo:

Security State = C0
Risk = C2
Organization = C3

Resultado:

C0
27. Conflict Testing

El framework deberá generar deliberadamente conflictos entre reglas para comprobar:

detección;
precedencia;
resolución;
fail-closed.

Ejemplo:

Rule A → C0
Rule B → C2

La prueba debe verificar que la regla de mayor autoridad/minimum security prevalezca.

28. Unknown / Error Testing

Se deberán inyectar:

atributo desconocido;
policy inexistente;
policy incompatible;
versión incompatible;
risk unavailable;
tenant context unavailable;
malformed context;
evaluator error.

La condición:

UNKNOWN

no deberá convertirse en:

ALLOW

ni utilizarse para degradar criticidad.

29. Mutation Testing

El framework deberá soportar mutation testing de políticas.

Ejemplo:

Original:
risk = CRITICAL → C0

Mutation:

risk = CRITICAL → C2

La suite debería detectar la mutación.

Esto permite verificar que los tests realmente protegen las reglas críticas.

30. Test Coverage

La cobertura deberá evaluarse en diferentes dimensiones.

No será suficiente medir únicamente líneas de código.

Deberá existir cobertura conceptual de:

Policies
Rules
Resources
Actions
Criticality Levels
Risk Levels
Security States
Tenants
MFA States
Delegation States
Freshness Profiles
Precedence Paths
Failure Paths

Los targets exactos deberán alinearse con IS-016 y las futuras Engineering Specifications.

31. Continuous Verification

La verificación no termina con deployment.

Después de activar una policy, el sistema deberá continuar verificando:

policy version;
policy hash;
criticality distribution;
freshness behavior;
cacheability;
security invariants;
runtime errors;
policy mismatch;
unexpected deltas.
32. Runtime Verification

El runtime deberá poder comprobar periódicamente:

Active Policy
        vs
Expected Policy

Validando:

ID;
version;
hash;
generation;
policy status.

Una discrepancia deberá ser observable y tratada como incidente potencial.

33. Continuous Golden Testing

Los Golden Scenarios podrán ejecutarse periódicamente contra la versión activa.

Conceptualmente:

Active Policy
      ↓
Golden Suite
      ↓
Expected Results
      ↓
Compare

Si cambia inesperadamente:

ALERT

y, según criticidad:

AUTOMATED SAFE RESPONSE

podrá ser considerado.

34. Production Shadow Verification

Cuando sea seguro y apropiado, podrán ejecutarse evaluaciones shadow utilizando:

Active Policy

vs.

Reference / Candidate Policy

sin modificar decisiones reales.

Esto permitirá detectar divergencias utilizando tráfico representativo.

No deberán utilizarse datos sensibles innecesarios para generar estos análisis.

35. Drift Detection

Se deberá detectar policy drift.

Ejemplo:

Expected:
Policy v20 / hash ABC

Runtime Node A:
v20 / ABC

Runtime Node B:
v19 / XYZ

Resultado:

POLICY DRIFT

El sistema deberá:

registrar;
alertar;
impedir promoción;
determinar la acción segura correspondiente.
36. Continuous Verification of Cache/Freshness

La verificación deberá incluir:

Policy
 ↓
Criticality
 ↓
Freshness
 ↓
Cache Eligibility

Esto conecta directamente con:

ADR-056;
ADR-057;
ADR-058.

Una policy aparentemente correcta pero que produce un freshness profile incompatible deberá fallar la verificación.

37. Regression Severity

Las regresiones podrán clasificarse:

INFO
WARNING
HIGH
CRITICAL
BLOCKING
BLOCKING

Incluye como mínimo:

Platform Minimum violation;
tenant isolation regression;
C0 downgrade no autorizado;
security-state downgrade;
critical revocation freshness regression;
privilege boundary violation;
unknown/error resulting in unsafe downgrade;
policy integrity failure.
38. CI/CD Integration

La validación deberá integrarse al pipeline.

Conceptualmente:

Pull Request
     ↓
Policy Validation
     ↓
Static Analysis
     ↓
Unit Tests
     ↓
Scenario Tests
     ↓
Regression Suite
     ↓
Security Invariants
     ↓
Differential Testing
     ↓
Performance
     ↓
Approval

Una prueba crítica fallida deberá bloquear el pipeline.

39. Policy Test Artifact

Cada versión de policy deberá poder asociarse con sus resultados de testing.

Conceptualmente:

Policy v21
   │
   ├── Validation Result
   ├── Simulation Result
   ├── Regression Result
   ├── Security Test Result
   ├── Performance Result
   └── Approval Record

Esto crea trazabilidad:

Policy
   ↓
Tests
   ↓
Evidence
   ↓
Approval
   ↓
Deployment
40. Evidence Retention

Los resultados relevantes deberán conservarse para:

auditoría;
incident response;
compliance;
debugging;
rollback;
comparación histórica.

La duración exacta de retención queda pendiente.

41. Performance Testing

El framework deberá comprobar que la policy candidata no introduzca una degradación significativa del Authorization Runtime.

Debe medirse:

policy evaluation latency;
criticality calculation latency;
policy lookup;
compilation;
cache interaction;
memory consumption.

El target de autorización establecido previamente de <50 ms deberá mantenerse como referencia arquitectónica para el path correspondiente.

42. Load Testing

Se deberán probar:

alta concurrencia;
múltiples tenants;
muchas policies;
múltiples versiones;
alta frecuencia de cambios;
alta frecuencia de risk changes;
invalidation storms.

La carga no deberá provocar degradación que termine modificando las garantías de seguridad.

43. Chaos Testing

Deberán probarse escenarios como:

Redis unavailable
Event Platform unavailable
Policy Store unavailable
Policy Consumer paused
Delayed invalidation
Duplicate invalidation
Out-of-order events
Policy propagation delay
Runtime restart
Partial deployment
Policy corruption
Network partition

El objetivo es comprobar que las Criticality Policies mantengan:

fail-closed
+
tenant isolation
+
security freshness
44. Recovery Verification

Después de recuperación se deberá volver a ejecutar:

policy integrity;
policy version;
policy hash;
security epoch;
Golden Suite;
criticality invariants;
freshness verification;
cache verification.

No se considerará recuperado un runtime únicamente porque esté respondiendo HTTP 200.

45. AI-Assisted Verification

La IA podrá utilizarse para:

generar escenarios adicionales;
detectar patrones;
analizar diffs;
proponer casos límite;
identificar posibles regresiones;
generar mutation candidates;
analizar resultados históricos.

Pero los tests críticos deberán permanecer deterministas y reproducibles.

La IA no podrá:

marcar automáticamente una regresión como aceptable;
eliminar un test obligatorio;
reducir un Platform Minimum;
aprobar una policy;
desactivar continuous verification.
46. Test Determinism

Los tests de policy deberán ser reproducibles.

El resultado no deberá depender accidentalmente de:

estado mutable externo;
disponibilidad de un servicio no controlado;
resultado probabilístico de IA;
orden no determinista;
reloj no controlado;
información cross-tenant.

Cuando se utilicen componentes dinámicos, deberán existir mecanismos de snapshot/freeze/mock/control del contexto.

47. Time-Based Testing

Debido a que policies pueden tener:

effective_from
effective_until

deberán probarse escenarios:

before activation
at activation
during active window
at expiration
after expiration

También:

timezone;
boundary timestamps;
clock skew;
daylight saving cuando corresponda.
48. Version Regression Matrix

El framework deberá poder comparar:

vN-1
vN

y, cuando sea necesario:

vN-2
vN-1
vN

Esto permitirá detectar regresiones que una comparación simple podría ocultar.

49. Regression Baseline Integrity

La baseline utilizada para comparar deberá estar protegida.

No se deberá permitir que:

Candidate Policy

modifique silenciosamente:

Expected Results

para que los tests pasen.

Los Golden Scenarios y expected outcomes deberán tener governance independiente.

50. Safe Test Data

Los escenarios deberán utilizar datos sintéticos o anonimizados siempre que sea posible.

No deberán introducirse:

passwords;
tokens;
secrets;
datos de salud;
PII innecesaria;
credenciales reales.

Esto es especialmente importante debido a que Identity eventualmente interactuará con información altamente sensible del ecosistema EVOXA.

51. Observability

Se deberán medir como mínimo:

policy_tests_total
policy_tests_failed
security_regressions_total
criticality_downgrades
freshness_regressions
invariant_violations
mutation_tests_detected
golden_tests_failed
policy_drift_detected
shadow_differences
runtime_verification_failures

También:

test duration;
simulation duration;
regression analysis duration;
verification latency.
52. Automated Response

Cuando una verificación crítica falle en producción, la respuesta deberá estar gobernada por políticas de seguridad.

Posibles respuestas:

ALERT
↓
STOP PROMOTION
↓
MARK POLICY UNHEALTHY
↓
REQUIRE FRESH EVALUATION
↓
ROLLBACK

La acción exacta dependerá de la severidad.

No deberá existir una regla genérica de:

test failed → automatically rollback

sin considerar el contexto y la seguridad del rollback.

53. Continuous Verification State

Se establece conceptualmente:

UNKNOWN
    ↓
VERIFYING
    ↓
VALIDATED

o:

VERIFYING
    ↓
FAILED

Una policy en estado UNKNOWN no deberá interpretarse como segura.

54. Security Confidence

Podrá mantenerse un estado de confianza:

UNKNOWN
DEGRADED
VALIDATED
TRUSTED

Este estado será complementario al lifecycle de la policy.

No reemplaza:

ACTIVE;
APPROVED;
VALIDATING.
55. No Bypass

Ningún mecanismo de testing podrá ser desactivado para permitir una activación insegura.

No se permitirá:

tests failing
      ↓
force activate

excepto mediante un mecanismo de emergencia formalmente definido, auditado y sujeto a revisión posterior.

56. Consecuencias positivas

Este ADR proporciona:

detección temprana de regresiones;
protección continua;
evidencia objetiva para aprobación;
integración CI/CD;
pruebas de invariantes;
differential testing;
property-based testing;
mutation testing;
verificación post-deployment;
detección de drift;
protección de freshness;
mayor seguridad frente a cambios dinámicos de risk;
mayor confianza en AI-assisted policy management.
57. Consecuencias negativas

Introduce:

mayor cantidad de tests;
mayor coste computacional;
necesidad de mantener Golden Scenarios;
necesidad de mantener expected results;
complejidad adicional en CI/CD;
necesidad de infraestructura de simulación;
necesidad de observabilidad específica;
mayor disciplina para cambios de policies.

Esta complejidad se considera aceptable porque Criticality Policy afecta directamente el comportamiento de seguridad y freshness de Authorization.

58. Alternativas rechazadas
58.1 Unit Tests Only

Rechazado.

No cubren interacciones complejas entre policies, risk, tenant, MFA y delegation.

58.2 Manual Testing

Rechazado como mecanismo principal.

Puede complementar, pero no proporciona cobertura ni reproducibilidad suficientes.

58.3 Simulation Only

Rechazado.

La simulación no reemplaza pruebas automatizadas ni invariantes.

58.4 TTL-Based Monitoring Only

Rechazado.

TTL no demuestra que una policy mantenga garantías de seguridad.

58.5 AI-Based Validation Only

Rechazado.

La IA puede asistir, pero la seguridad crítica requiere pruebas deterministas.

58.6 Production-Only Verification

Rechazado.

Las regresiones deben detectarse antes de activation.

58.7 Automatic Rollback on Any Difference

Rechazado.

No toda diferencia es una regresión. Debe distinguirse entre:

expected change

y:

security regression
59. Non-Negotiables
Criticality Policies deben probarse antes de activarse.
Security regressions bloquean activation.
Platform Minimum debe tener tests obligatorios.
Tenant isolation debe tener tests obligatorios.
Criticality downgrade debe ser detectado.
Freshness downgrade debe ser detectado.
Cache eligibility regression debe ser detectada.
Golden Scenarios deben estar protegidos.
Baselines deben ser versionadas.
Differential testing debe estar disponible.
Security invariants no pueden ser desactivados por una policy.
Unknown no puede reducir seguridad.
Mutation testing debe utilizarse para validar la efectividad de la suite crítica.
Continuous Verification debe continuar después del deployment.
Policy drift debe ser detectable.
Policy version/hash deben verificarse.
Tests deben ser reproducibles.
No secrets en test artifacts.
AI no puede aprobar ni desactivar controles.
Chaos/recovery testing debe preservar fail-closed y tenant isolation.
60. Trazabilidad
Blueprint
Identity
Authorization
Security Policy Engine
Event-Driven Architecture
Security
Testing & Quality
Observability
Deployment
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

ADR-IDENTITY-055
Cache Recovery & Warm-Up

ADR-IDENTITY-056
Cache Consistency & Freshness

ADR-IDENTITY-057
Freshness Classes & Security SLAs

ADR-IDENTITY-058
TTL Profiles & Operation Criticality

ADR-IDENTITY-059
Criticality Policy Model & Dynamic Risk

ADR-IDENTITY-060
Policy Lifecycle, Simulation & Safe Deployment
61. Dependencias

Este ADR depende de:

Authorization Policy Engine;
Criticality Policy Model;
Policy DSL;
Policy Validator;
Simulation Engine;
Authorization Runtime;
Risk Context;
MFA Context;
Delegation Context;
Tenant Context;
Freshness Profiles;
Cache subsystem;
Event Platform;
Audit;
CI/CD;
Observability;
Deployment infrastructure.
62. Decisiones pendientes

Se dejan explícitamente pendientes:

Catálogo definitivo de Golden Scenarios.
Catálogo completo de Security Invariants.
Thresholds de regression severity.
Definición exacta de mutation testing.
Framework tecnológico de property-based testing.
Formato definitivo de CriticalityTestScenario.
Persistencia de test artifacts.
Retención de resultados.
Policy diff format.
Baseline management.
Shadow testing duration.
Production shadow sampling.
Continuous verification frequency.
Drift detection interval.
Performance thresholds.
Load-testing profiles.
Chaos-testing profiles.
Automated rollback criteria.
Emergency verification procedure.
Multi-region verification strategy.
CI/CD integration details.
Test-data generation strategy.
Time simulation mechanism.
Exact policy migration testing.
Formal definition of security regression.

Estas decisiones deberán concretarse en futuras ADRs o Engineering Specifications antes de implementar el framework definitivo.

63. Acceptance Criteria
AC-01 — Static Validation

Una policy inválida debe fallar antes de testing.

AC-02 — Unit Testing

Las reglas individuales deben disponer de pruebas deterministas.

AC-03 — Golden Scenarios

Toda policy candidata debe ejecutarse contra Golden Scenarios.

AC-04 — Regression Baseline

La candidate policy debe compararse con la versión baseline.

AC-05 — Criticality Delta

Los cambios C0/C1/C2/C3 deben ser identificados.

AC-06 — Security Regression

Una regresión de seguridad debe bloquear activation.

AC-07 — Freshness Regression

Un downgrade no autorizado de freshness debe bloquear activation.

AC-08 — Cache Regression

Un cambio inseguro de cache eligibility debe detectarse.

AC-09 — Tenant Isolation

Las pruebas deben demostrar aislamiento entre tenants.

AC-10 — Risk

Las pruebas deben cubrir LOW/MEDIUM/HIGH/CRITICAL/UNKNOWN.

AC-11 — MFA

Las pruebas deben cubrir los estados críticos de MFA.

AC-12 — Delegation

Las pruebas deben verificar el privilege boundary.

AC-13 — Security State

Revocations y security states deben mantener sus requisitos.

AC-14 — Property Testing

Las invariantes críticas deben poder verificarse sobre múltiples contextos.

AC-15 — Mutation Testing

Las mutaciones de seguridad conocidas deben ser detectables.

AC-16 — Differential Testing

Current vs Candidate debe producir un diff reproducible.

AC-17 — CI/CD

Security regressions deben bloquear el pipeline.

AC-18 — Runtime Verification

La policy activa debe verificarse después del deployment.

AC-19 — Drift

Policy version/hash mismatch debe ser detectable.

AC-20 — Recovery

Después de recovery debe ejecutarse nuevamente la verificación crítica.

64. Resultado arquitectónico

Con ADR-061 se completa el ciclo iniciado con los ADR anteriores:

ADR-059
Criticality Policy Model
        ↓
ADR-060
Lifecycle + Simulation + Safe Deployment
        ↓
ADR-061
Testing + Regression + Continuous Verification

El flujo completo queda:

                 ┌──────────────────────┐
                 │ Criticality Policy   │
                 │      Candidate       │
                 └──────────┬───────────┘
                            ↓
                       VALIDATION
                            ↓
                       SIMULATION
                            ↓
                    SECURITY TESTING
                            ↓
                   REGRESSION ANALYSIS
                            ↓
                  DIFFERENTIAL TESTING
                            ↓
                   SECURITY INVARIANTS
                            ↓
                       APPROVAL
                            ↓
                    SAFE DEPLOYMENT
                            ↓
                         CANARY
                            ↓
                      ACTIVATION
                            ↓
                 CONTINUOUS VERIFICATION
                            ↓
                  ┌─────────┴─────────┐
                  ↓                   ↓
                VALID               FAILED
                  ↓                   ↓
               ACTIVE             RESPONSE
                                      ↓
                              ROLLBACK / ISOLATE

La decisión fundamental es:

Una Authorization Criticality Policy no se considera segura simplemente porque sea válida. Debe demostrar, mediante pruebas deterministas, regresión controlada, invariantes de seguridad y verificación continua, que conserva las garantías arquitectónicas de EVOXA.

Esto convierte el sistema de Criticality Policies en un control de seguridad continuamente verificable, y no solamente en un mecanismo de configuración.

Siguiente ADR recomendado

ADR-IDENTITY-062 — Authorization Criticality Policy Test Catalog, Golden Scenarios & Security Invariants

Este sería el siguiente paso lógico: definir qué escenarios concretos debemos probar, cuáles son los Golden Scenarios obligatorios, qué invariantes son matemáticamente/normativamente inmutables y cómo construir la matriz de pruebas C0–C3 × Risk × MFA × Tenant × Delegation × Session × Resource × Action.
