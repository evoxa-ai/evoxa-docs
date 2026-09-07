ADR-IDENTITY-063 — Authorization Criticality Policy Test Execution Engine, Scenario Generation & Regression Evaluation

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-07
Dominio: Identity / Authorization / Security / Policy Engine / Risk / Testing / Governance
Prioridad: Critical
Tipo: Architecture / Security / Testing / Policy Verification / Runtime

1. Contexto

Los ADR anteriores han definido progresivamente el modelo de autorización y, especialmente, el modelo de Criticality que determina qué nivel de frescura, validación y comportamiento de caché requiere una operación.

La arquitectura actualmente contempla:

RBAC.
ABAC.
Resource Ownership.
Delegation.
Tenant Isolation.
Security Policy Engine.
Authorization Runtime.
Risk Context.
MFA Context.
Session Context.
Security State.
Authorization Cache.
Freshness Classes F0–F3.
Criticality Classes C0–C3.
Security Epochs.
Policy Versions.
Entity Versions.
Context Fingerprints.
Golden Scenarios.
Security Invariants.
Regression Evaluation.

Los ADR-059 y ADR-062 establecieron que la Criticality Policy debe ser:

declarativa;
determinista;
versionada;
validable;
auditable;
monotónica respecto de requisitos de seguridad;
testeable mediante escenarios;
protegida contra regresiones.

ADR-062 definió el Authorization Criticality Policy Test Catalog, los Golden Scenarios y los Security Invariants.

Sin embargo, todavía falta definir cómo se ejecutan estos tests, cómo se generan escenarios adicionales, cómo se comparan resultados contra una versión anterior y cómo se determina si una modificación de políticas puede ser activada.

Este ADR define ese mecanismo.

2. Problema

No es suficiente disponer de una colección de tests.

Una plataforma de autorización crítica necesita responder de manera determinista preguntas como:

¿Qué escenarios deben ejecutarse ante un cambio de política?
¿Cómo se generan escenarios que no fueron escritos manualmente?
¿Cómo se ejecuta una política nueva contra la política actualmente activa?
¿Cómo se detecta un cambio esperado frente a una regresión de seguridad?
¿Cómo se comprueba que los Security Invariants siguen siendo verdaderos?
¿Cómo se prueban combinaciones de:
tenant;
actor;
recurso;
acción;
riesgo;
MFA;
sesión;
delegación;
ownership;
security state;
freshness?
¿Cómo se evita que una modificación aparentemente válida reduzca accidentalmente un requisito de seguridad?
¿Cómo se incorpora el resultado de la validación al lifecycle de una Criticality Policy?
¿Cómo se mantiene trazabilidad entre:
Policy Version;
Runtime Version;
Scenario Version;
Invariant Version;
Test Execution;
Approval;
Deployment?

Sin un mecanismo centralizado, diferentes equipos podrían interpretar o probar las políticas de forma distinta.

3. Decisión

Se establece un Authorization Criticality Policy Test Execution Engine como componente lógico centralizado para validar las Criticality Policies.

El motor será responsable de:

ejecutar escenarios deterministas;
ejecutar Golden Scenarios;
evaluar Security Invariants;
generar escenarios adicionales;
ejecutar property-based tests;
ejecutar boundary tests;
ejecutar mutation tests;
comparar versiones de políticas;
detectar regresiones;
producir un resultado de validación auditable;
bloquear activaciones que violen requisitos de seguridad.

El motor no autoriza operaciones reales y no reemplaza al Authorization Runtime.

Su función es verificar que el comportamiento esperado del Authorization/Criticality model sea correcto antes y durante su evolución.

4. Arquitectura conceptual

La arquitectura será:

                    ┌──────────────────────────────┐
                    │ Criticality Policy Repository│
                    └──────────────┬───────────────┘
                                   │
                                   ▼
                    ┌──────────────────────────────┐
                    │ Policy Validator / Compiler  │
                    └──────────────┬───────────────┘
                                   │
                                   ▼
              ┌────────────────────────────────────────┐
              │ Criticality Test Execution Engine      │
              │                                        │
              │  Scenario Loader                       │
              │  Scenario Generator                    │
              │  Golden Test Runner                    │
              │  Invariant Runner                      │
              │  Property Test Runner                  │
              │  Mutation Test Runner                  │
              │  Differential Runner                   │
              │  Regression Evaluator                  │
              │  Result Aggregator                     │
              └───────────────────┬────────────────────┘
                                  │
             ┌────────────────────┼─────────────────────┐
             ▼                    ▼                     ▼
       Expected Result       Candidate Result      Baseline Result
             │                    │                     │
             └────────────────────┼─────────────────────┘
                                  ▼
                    ┌──────────────────────────────┐
                    │ Regression Evaluation        │
                    └──────────────┬───────────────┘
                                   │
                 ┌─────────────────┼─────────────────┐
                 ▼                 ▼                 ▼
               PASS             REVIEW             BLOCK
                 │                 │                 │
                 └─────────────────┼─────────────────┘
                                   ▼
                         Governance / Approval
5. Principios fundamentales
5.1 Test Engine ≠ Authorization Runtime

El Test Execution Engine utiliza la misma semántica normativa del Authorization Runtime, pero permanece separado operacionalmente.

Esto evita que los tests modifiquen:

permisos reales;
sesiones;
memberships;
MFA;
security state;
producción.
5.2 Determinismo

Para una misma combinación de:

Policy Version
Runtime Version
Input Context
Scenario Version
Invariant Version

el resultado esperado debe ser determinista.

No deben existir tests cuya conclusión dependa de:

estado externo no controlado;
llamadas arbitrarias a Internet;
servicios externos no simulados;
tiempo real no congelado;
datos aleatorios no reproducibles.

Cuando se utilice aleatoriedad para generación de escenarios, deberá existir un:

seed

reproducible.

6. Modelo de Test Execution

Cada ejecución tendrá conceptualmente:

TestExecution

con:

Campo	Descripción
execution_id	Identificador único
policy_id	Policy evaluada
policy_version	Versión
baseline_policy_version	Versión de comparación
runtime_version	Versión del runtime
scenario_catalog_version	Versión del catálogo
invariant_version	Versión de invariantes
generator_version	Versión del generador
seed	Semilla cuando corresponda
started_at	Inicio
completed_at	Fin
environment	Entorno
result	Resultado global
severity	Severidad máxima
summary	Resumen
evidence_reference	Evidencia
correlation_id	Correlación
7. Tipos de ejecución

El Engine soportará diferentes modos.

7.1 Golden Execution

Ejecuta únicamente Golden Scenarios.

Objetivo:

Garantizar que los comportamientos críticos conocidos permanezcan estables.

Especialmente importante para:

tenant isolation;
security state;
MFA;
session revocation;
delegation;
policy precedence;
cache freshness.
7.2 Invariant Execution

Ejecuta los Security Invariants definidos en ADR-062.

Ejemplo:

EffectiveCriticality >= PlatformMinimum

Si el resultado viola el invariant:

INVARIANT_VIOLATION

El resultado debe bloquear cualquier activación de la política.

7.3 Scenario Execution

Ejecuta escenarios explícitamente definidos en el catálogo.

Cada escenario contiene conceptualmente:

Scenario
 ├── id
 ├── version
 ├── category
 ├── input_context
 ├── expected_criticality
 ├── expected_freshness
 ├── expected_cacheability
 ├── expected_security_behavior
 ├── invariants
 ├── severity
 └── golden
7.4 Generated Scenario Execution

El Engine podrá generar escenarios automáticamente.

Esto permitirá cubrir combinaciones que sería impráctico definir manualmente.

Las dimensiones principales serán:

Resource
Action
Risk
Security State
MFA
Session
Delegation
Tenant
Ownership
Freshness
Context
7.5 Property-Based Execution

En lugar de comprobar únicamente ejemplos concretos, el Engine comprobará propiedades generales.

Ejemplo:

Si:

Risk = LOW

produce:

C3

y posteriormente el riesgo aumenta a:

HIGH

el resultado no puede disminuir.

Formalmente:

Criticality(high_risk) >= Criticality(low_risk)

La propiedad puede comprobarse sobre múltiples escenarios generados.

7.6 Boundary Execution

Se probarán específicamente los límites.

Ejemplos:

C3 → C2
C2 → C1
C1 → C0

y:

F2 → F1
F1 → F0

También:

expires_at - 1ms
expires_at
expires_at + 1ms

para elementos relacionados con frescura.

7.7 Differential Execution

Una política candidata podrá ejecutarse contra una política baseline.

Baseline Policy
       │
       ├──── Scenario A → C2
       ├──── Scenario B → C1
       └──── Scenario C → C0

Candidate Policy
       │
       ├──── Scenario A → C2
       ├──── Scenario B → C0
       └──── Scenario C → C0

El Engine detectará:

A: NO_CHANGE
B: HARDENING
C: NO_CHANGE
8. Regression Evaluation

El Regression Evaluator clasificará los cambios.

8.1 NO_CHANGE

El resultado es idéntico al baseline.

C2 → C2
F2 → F2
cacheable → cacheable

Resultado:

NO_CHANGE
8.2 EXPECTED_CHANGE

Existe un cambio documentado y aprobado.

Ejemplo:

C2 → C1

debido a un cambio explícito de requerimiento.

Debe existir:

Change Request;
justificación;
análisis de impacto;
escenarios actualizados;
aprobación.
8.3 HARDENING

El comportamiento se vuelve más restrictivo sin violar ninguna garantía.

Ejemplo:

C2 → C1

o:

F2 → F1

El cambio puede ser válido, pero debe quedar registrado.

8.4 BEHAVIOR_CHANGE

Existe una modificación funcional no clasificada como:

NO_CHANGE;
EXPECTED_CHANGE;
HARDENING.

Requiere revisión.

8.5 SECURITY_REGRESSION

Se produce una reducción de seguridad.

Ejemplos:

C0 → C1
C1 → C2
F0 → F1
F1 → F2

cuando la operación requiere el nivel anterior.

También:

non-cacheable → cacheable

para una operación que no puede utilizar caché.

Resultado:

BLOCK
8.6 INVARIANT_VIOLATION

Cualquier Security Invariant se viola.

Resultado:

BLOCK

independientemente de que otros tests hayan pasado.

9. Scenario Generation Engine

El Scenario Generator generará escenarios a partir de los registros autorizados.

Inputs
Resource Registry
Action Registry
Attribute Registry
Criticality Policies
Security States
Risk Catalog
MFA States
Session States
Delegation States
Tenant States
Ownership States
Freshness Profiles
10. Generación combinatoria controlada

No se pretende ejecutar un producto cartesiano completo en cada ejecución.

La combinación:

Resource × Action × Risk × MFA × Session × Delegation
× Tenant × SecurityState × Ownership × Freshness

puede producir una cantidad enorme de casos.

Por ello se utilizará una estrategia combinada:

Nivel 1 — Golden

Casos críticos explícitos.

Nivel 2 — Pairwise

Combinaciones de dos dimensiones.

Nivel 3 — Critical Combinations

Combinaciones especialmente sensibles.

Nivel 4 — Property-Based

Propiedades matemáticas/de seguridad.

Nivel 5 — Randomized Reproducible

Generación controlada mediante seed.

Nivel 6 — Exhaustive Subsets

Para dominios pequeños o componentes críticos.

11. Critical Scenario Prioritization

No todos los escenarios tienen la misma prioridad.

Se establecerá:

Prioridad	Tipo
P0	Security invariants / tenant isolation / critical revocations
P1	C0/C1 / MFA / sessions / delegation
P2	Risk / precedence / freshness / cache
P3	Normal behavioral coverage
P4	Extended combinations

Los P0 deben ejecutarse siempre antes de permitir una activación.

12. Security Invariant Evaluation

Cada escenario puede estar asociado a uno o más invariants.

Ejemplo:

Scenario:
User is disabled

Expected:
Criticality = C0
Freshness = F0
Cache Allow = false

Invariants:

INV-CRITICALITY-01
INV-CRITICALITY-04
INV-CRITICALITY-13
INV-CRITICALITY-14

El Engine comprobará tanto:

Expected Result

como:

Security Invariants
13. Mutation Testing

El Engine deberá soportar mutation testing para comprobar que los tests realmente detectan modificaciones peligrosas.

Ejemplos de mutaciones:

C0 → C1
DENY → ALLOW
MAX → MIN
Security Override eliminado
Tenant Guard eliminado
Risk HIGH → LOW
MFA required → MFA optional

El resultado esperado es que los tests críticos detecten estas mutaciones.

Si una mutación peligrosa sobrevive:

MUTATION_SURVIVED

el catálogo de tests deberá considerarse insuficiente.

14. Differential Policy Testing

Cuando se propone una nueva versión:

Policy v10
     ↓
Candidate v11

el Engine ejecutará ambos sobre el mismo conjunto de escenarios.

Esto evita que un cambio en los inputs sea confundido con un cambio de política.

Conceptualmente:

Scenario Input
      │
 ┌────┴────┐
 ▼         ▼
v10       v11
 │         │
 ▼         ▼
Result    Result
 └────┬────┘
      ▼
Differential Comparator
15. Runtime Versioning

Los resultados no podrán interpretarse correctamente sin conocer la versión del runtime.

Por eso:

Policy Version
+
Runtime Version
+
Scenario Version
+
Invariant Version

formarán parte del baseline de regresión.

Una modificación del evaluator puede cambiar resultados incluso cuando la política no cambió.

Eso debe detectarse.

16. Golden Baseline

Cada entorno controlado tendrá un baseline conceptual:

RegressionBaseline

formado por:

Active Policy Version
Scenario Catalog Version
Invariant Version
Runtime Version
Generator Version

El baseline será inmutable una vez publicado.

Una nueva versión deberá compararse contra él.

17. Resultado de una ejecución

El resultado global podrá ser:

PASS
PASS_WITH_REVIEW
BLOCKED
FAILED

Con clasificación detallada:

NO_CHANGE
EXPECTED_CHANGE
HARDENING
BEHAVIOR_CHANGE
SECURITY_REGRESSION
INVARIANT_VIOLATION
TEST_INFRASTRUCTURE_FAILURE
18. Regla de bloqueo

La activación deberá bloquearse si ocurre cualquiera de los siguientes:

Security Regression
Invariant Violation
Tenant Isolation Failure
Privilege Amplification
Unsafe ALLOW
Critical Freshness Violation
Platform Minimum Violation
Policy Evaluation Error affecting security
Unknown Security State interpreted as safe
Mutation of critical guard survives
19. Unknown y Error

Los tests deben comprobar explícitamente:

UNKNOWN

y:

ERROR

No pueden transformarse silenciosamente en:

ALLOW

o:

C3

cuando eso implique reducir requisitos de seguridad.

Ejemplo:

Risk = UNKNOWN

no significa:

Risk = LOW

automáticamente.

20. Testing de Tenant Isolation

El Engine deberá generar escenarios como:

Actor Tenant A
Resource Tenant A

y:

Actor Tenant A
Resource Tenant B

El segundo caso debe respetar la regla de aislamiento.

También se deberán probar:

Cross-Tenant Admin
Cross-Tenant Delegation
Cross-Tenant Ownership
Cross-Tenant Impersonation
Unknown Tenant
Missing Tenant
Invalid Membership
21. Testing de Security State

Se generarán escenarios para:

ACTIVE
PENDING
SUSPENDED
DISABLED
REVOKED
COMPROMISED

cuando sean aplicables.

Particularmente:

User Disabled
Session Revoked
Membership Revoked
MFA Factor Revoked
Refresh Family Revoked
Delegation Revoked

deberán probar que las garantías de seguridad no disminuyen.

22. Testing de MFA

Se probarán estados como:

MFA_NOT_REQUIRED
MFA_REQUIRED
MFA_PENDING
MFA_VERIFIED
MFA_EXPIRED
MFA_REVOKED

El Engine deberá comprobar:

frescura de MFA;
expiración;
step-up;
factor revocado;
factor reemplazado;
recuperación;
trusted device cuando corresponda.

Una evidencia MFA inválida no puede convertirse en evidencia válida por defecto.

23. Testing de Session

Se probarán:

ACTIVE
EXPIRED
REVOKED
SUSPENDED
UNKNOWN

y escenarios de:

Session Revocation
Refresh Family Revocation
Logout
Logout All
Concurrent Refresh
Security Reset
24. Testing de Delegation

Se comprobará:

valid delegation
expired delegation
revoked delegation
scope mismatch
resource mismatch
action mismatch
tenant mismatch
delegation depth exceeded
privilege amplification

Una delegación nunca podrá proporcionar más privilegios que los permitidos al delegador.

25. Testing de Freshness

El Engine deberá generar escenarios para:

TTL valid
TTL expired
Security epoch changed
Policy version changed
Entity version changed
Invalidation received
Invalidation delayed
Invalidation duplicated
Invalidation out-of-order
Generation changed
Context fingerprint changed

Esto deberá validar ADR-056/057/058.

26. Testing de Cache

Se comprobará:

Fresh ALLOW
Fresh DENY
Acceptable ALLOW
Stale ALLOW
Stale DENY
Invalid ALLOW
Invalid DENY
Cache miss
Redis unavailable
L1 unavailable
Event Platform delayed

Especialmente:

Un ALLOW almacenado en caché no puede utilizarse para eludir una nueva restricción de seguridad.

27. Scenario Shrinking

Cuando un escenario generado falle, el Engine debería intentar reducirlo hasta encontrar la combinación mínima que reproduce el error.

Ejemplo:

Original:
HIGH risk
MFA expired
delegation active
tenant valid
session active
ownership valid
policy v12

Si la falla realmente depende solamente de:

HIGH risk + MFA expired

el Engine debería conservar una representación reducida.

Esto mejora:

debugging;
reproducibilidad;
análisis humano;
creación de Golden Scenario.

La estrategia exacta de shrinking queda pendiente.

28. Reproducibilidad

Toda ejecución generada debe poder reproducirse mediante:

policy_version
runtime_version
scenario_version
generator_version
seed
input_context

Un fallo debe poder reconstruirse posteriormente.

29. Test Evidence

Cada ejecución crítica deberá generar evidencia suficiente para auditoría.

Conceptualmente:

TestEvidence
 ├── execution_id
 ├── scenario_id
 ├── policy_version
 ├── runtime_version
 ├── input_hash
 ├── expected_result
 ├── actual_result
 ├── invariant_results
 ├── classification
 ├── severity
 ├── timestamp
 └── evidence_reference

Los resultados no deben contener:

passwords;
tokens;
MFA secrets;
recovery codes;
claves criptográficas;
información sensible innecesaria.
30. Policy Lifecycle Integration

El resultado del Test Execution Engine será integrado al lifecycle de Criticality Policy.

Conceptualmente:

DRAFT
  ↓
VALIDATING
  ↓
TESTING
  ↓
SIMULATION
  ↓
APPROVED
  ↓
ACTIVE

Una política que tenga:

SECURITY_REGRESSION

o:

INVARIANT_VIOLATION

no podrá avanzar automáticamente a:

ACTIVE
31. CI/CD Integration

El Engine deberá integrarse con CI/CD.

Ejemplo conceptual:

Pull Request
     ↓
Policy Validation
     ↓
Golden Tests
     ↓
Invariant Tests
     ↓
Generated Scenarios
     ↓
Differential Tests
     ↓
Mutation Tests
     ↓
Regression Evaluation
     ↓
Governance Gate

Una modificación crítica deberá superar los gates definidos.

32. Test Profiles

Se definirán perfiles de ejecución.

Profile: FAST

Para desarrollo:

Golden
P0
P1
Core invariants
Profile: STANDARD

Para CI:

Golden
P0-P2
Pairwise
Property-based
Differential
Profile: FULL

Para release:

All Golden
All Invariants
Generated scenarios
Property-based
Mutation
Differential
Boundary
Chaos-compatible suites

Los nombres son conceptuales y podrán evolucionar.

33. Production Verification

La producción no debe ejecutar indiscriminadamente escenarios que puedan modificar estado real.

La verificación productiva será:

no destructiva;
limitada;
auditable;
tenant-aware;
controlada.

Podrá utilizar:

shadow evaluation
synthetic scenarios
read-only validation
sampled comparisons

Nunca deberá generar una modificación de seguridad real como parte de un test.

34. Shadow Evaluation

Cuando sea necesario comparar:

Active Policy
vs
Candidate Policy

se podrá ejecutar la Candidate Policy en modo:

SHADOW

sin modificar la decisión efectiva.

Ejemplo:

Production Request
       │
       ├── Active Policy → Effective Decision
       │
       └── Candidate Policy → Shadow Decision

Los resultados pueden compararse para detectar:

unexpected divergence
security hardening
security regression

La política candidata nunca podrá reemplazar la decisión efectiva mediante shadow mode.

35. AI-Assisted Testing

La IA podrá utilizarse para:

proponer escenarios;
detectar combinaciones faltantes;
sugerir nuevos Golden Scenarios;
analizar diferencias;
identificar posibles regresiones;
recomendar mutations;
detectar gaps de cobertura.

Pero la IA no podrá:

aprobar automáticamente una Security Regression;
eliminar un Security Invariant;
reducir C0/C1 a una clase inferior;
cambiar Platform Minimum;
activar una política;
declarar seguro un escenario UNKNOWN;
modificar resultados esperados sin governance.

La ejecución normativa continuará perteneciendo al Test Engine determinista.

36. Coverage Model

La cobertura no se limitará a líneas de código.

Se medirán varias dimensiones:

Policy Coverage
Scenario Coverage
Resource Coverage
Action Coverage
Risk Coverage
Security-State Coverage
MFA Coverage
Session Coverage
Delegation Coverage
Tenant Coverage
Freshness Coverage
Invariant Coverage
Precedence Coverage
Boundary Coverage
Mutation Coverage

Una política puede tener 100% de cobertura de código y aun así presentar una cobertura de seguridad insuficiente.

37. Precedence Testing

Se deberán generar combinaciones donde múltiples reglas sean aplicables.

Ejemplo:

Organization Policy
       +
Resource Policy
       +
Security Override
       +
Platform Minimum

El Engine deberá comprobar que la precedencia definida en ADR-059 se respeta.

Particularmente:

Platform Security Minimum

no puede ser debilitado por:

Organization Policy
38. Monotonicity Testing

Se establecerán propiedades monotónicas.

Ejemplos:

Risk
LOW ≤ MEDIUM ≤ HIGH ≤ CRITICAL

Un aumento de riesgo no puede reducir Criticality.

Security State

Un estado más restrictivo no puede producir un requisito de seguridad inferior.

Tenant Isolation

Mayor incertidumbre sobre tenant no puede producir un ALLOW más permisivo.

MFA

Una evidencia MFA más antigua o inválida no puede aumentar privilegios.

39. Test Flakiness

Los tests de autorización crítica deberán evitar flakiness.

Los casos no deterministas deberán:

congelar el tiempo;
controlar seeds;
aislar dependencias;
controlar concurrencia;
utilizar fixtures reproducibles.

Un test flaky no debe marcar automáticamente una política como segura.

40. Failure Classification

Debe distinguirse entre:

POLICY_FAILURE
RUNTIME_FAILURE
TEST_FAILURE
INFRASTRUCTURE_FAILURE
GOVERNANCE_FAILURE

Ejemplo:

Si Redis está caído durante una prueba de infraestructura:

INFRASTRUCTURE_FAILURE

No debe interpretarse automáticamente como:

POLICY_PASS

ni como:

POLICY_FAIL

hasta completar la evaluación correspondiente.

41. Concurrency Testing

Se deberán probar escenarios concurrentes como:

Policy Update
+
Authorization Evaluation
Membership Revocation
+
Authorization Evaluation
MFA Revocation
+
Authorization Evaluation
Security Epoch Increment
+
Cache Read

El objetivo es comprobar que nunca se produce un ALLOW inseguro por una condición de carrera.

42. Performance Testing

El Test Engine deberá medir:

tiempo de evaluación;
cantidad de escenarios/segundo;
generación;
comparación;
property testing;
mutation testing;
almacenamiento de resultados.

No deberá confundirse:

Test Engine Performance

con:

Authorization Runtime Performance

aunque ambos deberán mantenerse compatibles con los objetivos definidos en ADR-049 y ADR-028.

43. Auditabilidad

Una activación de política deberá poder reconstruirse:

Policy
   ↓
Validation
   ↓
Scenario Generation
   ↓
Golden Tests
   ↓
Invariant Tests
   ↓
Simulation
   ↓
Differential Test
   ↓
Regression Evaluation
   ↓
Approval
   ↓
Deployment

Esto permitirá responder:

¿Por qué esta política está activa?

y:

¿Qué pruebas demostraron que podía activarse?

44. Seguridad del Test Engine

El Engine será considerado componente de seguridad.

Deberá utilizar:

identidad de servicio;
mínimo privilegio;
autenticación S2S;
autorización;
tenant awareness;
audit logging;
secrets management;
aislamiento de entornos.

No podrá recibir ni almacenar secretos innecesarios.

45. Modelo de decisión del Regression Gate

Conceptualmente:

if invariant_violation:
    BLOCK

elif security_regression:
    BLOCK

elif tenant_isolation_failure:
    BLOCK

elif unsafe_allow:
    BLOCK

elif critical_test_failure:
    BLOCK

elif expected_change:
    REVIEW / APPROVAL

elif hardening:
    PASS_WITH_REVIEW

elif no_change:
    PASS

else:
    REVIEW

La implementación exacta queda pendiente de la especificación del Governance Gate.

46. Non-Negotiables
El Test Engine debe ser determinista.
Los Security Invariants son obligatorios.
Una violación de invariant bloquea activación.
Una Security Regression bloquea activación.
Tenant Isolation no puede degradarse.
Platform Minimum no puede degradarse.
UNKNOWN no puede convertirse silenciosamente en ALLOW seguro.
ERROR no puede convertirse silenciosamente en ALLOW seguro.
Criticality no concede permisos.
Criticality determina requisitos de seguridad/frescura.
El Runtime sigue siendo la autoridad de autorización.
El Test Engine no puede modificar producción durante tests.
Las pruebas generadas deben ser reproducibles.
Las ejecuciones deben versionarse.
Policy Version debe quedar registrada.
Runtime Version debe quedar registrada.
Scenario Version debe quedar registrada.
Invariant Version debe quedar registrada.
Differential testing debe utilizar inputs equivalentes.
Mutation testing debe detectar mutaciones críticas.
Revocation scenarios deben tener prioridad P0/P1.
AI no puede aprobar regresiones.
AI no puede reducir requisitos de seguridad.
Los resultados críticos deben ser auditables.
La activación debe estar protegida por un governance gate.
47. Consecuencias positivas

Esta decisión proporciona:

validación sistemática;
regresión detectable;
reproducibilidad;
cobertura multidimensional;
protección de invariants;
validación de tenant isolation;
validación de freshness;
validación de cache;
detección de privilege amplification;
comparación entre policy versions;
integración con CI/CD;
evidencia auditable;
soporte para evolución segura de políticas.

Además, permite que el modelo de autorización evolucione sin depender exclusivamente de tests manuales.

48. Consecuencias negativas

El sistema introduce:

complejidad adicional;
necesidad de mantener catálogos de escenarios;
coste computacional;
necesidad de versionar múltiples artefactos;
necesidad de governance;
mayor complejidad en CI/CD;
necesidad de controlar generación combinatoria;
mantenimiento de Golden Scenarios;
mantenimiento de Security Invariants;
necesidad de infraestructura de almacenamiento de resultados.

Estas consecuencias se consideran aceptables debido a que Identity/Authorization es una capacidad crítica de seguridad.

49. Alternativas rechazadas
49.1 Solo unit tests

Rechazada.

No proporciona suficiente cobertura de:

policy interactions;
tenant isolation;
risk;
MFA;
delegation;
freshness;
regression.
49.2 Solo Golden Scenarios

Rechazada.

Los Golden Scenarios cubren comportamientos conocidos, pero no garantizan cobertura de nuevas combinaciones.

49.3 Solo property-based testing

Rechazada.

Las propiedades generales no sustituyen los escenarios de negocio y seguridad explícitamente definidos.

49.4 First-match regression

Rechazada.

La evaluación debe considerar precedencia y composición de políticas.

49.5 Tests manuales antes de cada release

Rechazada.

No proporciona reproducibilidad ni cobertura suficiente.

49.6 Permitir que AI determine PASS/FAIL

Rechazada.

La decisión normativa debe permanecer determinista y gobernada.

50. Dependencias

Este ADR depende directamente de:

ADR-045 — Authorization Policy Model/Evaluation.
ADR-046 — Policy Language/Schema/Governance.
ADR-047 — Policy Engine Technology & Implementation.
ADR-048 — Policy DSL.
ADR-049 — Evaluation Runtime.
ADR-050 — Cache Invalidation/Consistency.
ADR-051 — Cache Key/Context Fingerprinting.
ADR-052 — Cache Eligibility/TTL.
ADR-053 — Cache Invalidation Events.
ADR-054 — Cache Failure/Degradation.
ADR-055 — Cache Recovery.
ADR-056 — Cache Consistency/Freshness.
ADR-057 — Freshness Classes/SLAs.
ADR-058 — TTL Profiles/Criticality Matrix.
ADR-059 — Criticality Policy Model.
ADR-062 — Criticality Policy Test Catalog/Golden Scenarios/Invariants.

También depende de:

Policy Validator;
Policy Compiler;
Authorization Runtime;
Resource Registry;
Action Registry;
Attribute Registry;
Risk Context;
MFA Context;
Session Context;
Delegation Model;
Tenant Model;
Audit;
CI/CD;
Observability.
51. Trazabilidad
Blueprint
Identity
 └── Authorization
      ├── RBAC
      ├── ABAC
      ├── Tenant Isolation
      ├── Ownership
      ├── Delegation
      └── Security Policy Engine
Implementation Stories

Principalmente:

IS-IDENTITY-006
IS-IDENTITY-007
IS-IDENTITY-010
IS-IDENTITY-011
IS-IDENTITY-012
IS-IDENTITY-013
IS-IDENTITY-015
IS-IDENTITY-016
IS-IDENTITY-017
ADR chain
045
 ↓
046
 ↓
047
 ↓
048
 ↓
049
 ↓
050–056
 ↓
057
 ↓
058
 ↓
059
 ↓
062
 ↓
063

ADR-063 convierte el modelo conceptual de pruebas de ADR-062 en un mecanismo de ejecución, generación y evaluación de regresiones.

52. Decisiones pendientes

Antes de pasar a implementación deberán definirse formalmente:

formato machine-readable del Scenario Catalog;
formato machine-readable de Security Invariants;
estructura exacta del Test Execution Result;
framework de property-based testing;
algoritmo de pairwise generation;
estrategia de random generation;
seed management;
estrategia de scenario shrinking;
mutation operators definitivos;
mutation score mínimo;
regression thresholds;
clasificación exacta de HARDENING;
governance de EXPECTED_CHANGE;
almacenamiento de evidencias;
retención de resultados;
firma/integridad de artefactos;
CI/CD gates;
ejecución paralela;
límites de escenarios;
timeout por test;
retry policy;
flaky-test policy;
production shadow-testing;
sampling;
multi-region testing;
chaos integration;
performance thresholds;
cobertura mínima por dimensión;
algoritmo definitivo de shrinking;
integración exacta con Policy Lifecycle;
permisos necesarios del Test Engine;
modelo de service identity;
Governance Gate;
approval workflow;
formato de regression report.
53. Acceptance Criteria
AC-01 — Central Test Engine

Debe existir un modelo conceptual único para ejecutar Criticality Policy Tests.

AC-02 — Golden Tests

Debe ejecutar Golden Scenarios versionados.

AC-03 — Invariants

Debe ejecutar Security Invariants y bloquear violaciones.

AC-04 — Scenario Generation

Debe generar escenarios adicionales de forma reproducible.

AC-05 — Pairwise

Debe soportar cobertura pairwise.

AC-06 — Property Testing

Debe soportar propiedades de seguridad.

AC-07 — Boundary Testing

Debe cubrir límites de Criticality/Freshness.

AC-08 — Differential Testing

Debe comparar baseline y candidate.

AC-09 — Regression Classification

Debe distinguir NO_CHANGE, EXPECTED_CHANGE, HARDENING, BEHAVIOR_CHANGE y SECURITY_REGRESSION.

AC-10 — Tenant Isolation

Debe detectar violaciones cross-tenant.

AC-11 — Platform Minimum

Debe detectar cualquier downgrade bajo Platform Minimum.

AC-12 — Risk Monotonicity

Debe comprobar que aumentar riesgo no reduzca Criticality.

AC-13 — MFA

Debe validar frescura y estado de evidencia MFA.

AC-14 — Session

Debe validar revocaciones y estados de sesión.

AC-15 — Delegation

Debe validar límites de delegación y evitar privilege amplification.

AC-16 — Cache/Freshness

Debe validar TTL, epochs, versions e invalidation.

AC-17 — Mutation Testing

Debe detectar mutations de seguridad críticas.

AC-18 — Reproducibility

Toda ejecución generada debe poder reproducirse.

AC-19 — Governance Gate

Una Security Regression o Invariant Violation debe bloquear activación.

AC-20 — Auditability

Debe existir trazabilidad completa entre policy, runtime, scenarios, invariants, execution y resultado.

54. Resultado arquitectónico

ADR-063 establece que la validación de Authorization Criticality no será un conjunto aislado de pruebas, sino un sistema de verificación continuo y versionado.

La cadena pasa a ser:

Criticality Policy
       ↓
Validation
       ↓
Scenario Catalog
       ↓
Scenario Generation
       ↓
Golden Tests
       ↓
Security Invariants
       ↓
Property / Boundary Tests
       ↓
Differential Evaluation
       ↓
Mutation Testing
       ↓
Regression Classification
       ↓
Governance Gate
       ↓
Approval
       ↓
Activation
       ↓
Continuous Verification

La regla fundamental es:

Una política no se considera segura simplemente porque sus casos conocidos pasan; debe demostrar que conserva las invariantes de seguridad y que no introduce regresiones bajo escenarios conocidos, generados y diferenciales.

Y, de forma especialmente importante:

La IA puede ayudar a descubrir escenarios y analizar resultados, pero la autoridad para declarar una Criticality Policy válida permanece en el mecanismo determinista de validación y governance.

55. Siguiente ADR recomendado

El siguiente paso natural es:

ADR-IDENTITY-064 — Authorization Criticality Policy Test Data Model, Machine-Readable Catalog & Evidence Schema

Este ADR debería definir formalmente la estructura de los artefactos que ADR-063 ejecutará:

Scenario
Golden Scenario
Security Invariant
Test Case
Generated Scenario
Expected Result
Test Execution
Test Evidence
Regression Result
Baseline
Coverage Result
Mutation Result

Eso nos permitirá cerrar la parte conceptual del Test Execution Engine antes de entrar en implementación.
