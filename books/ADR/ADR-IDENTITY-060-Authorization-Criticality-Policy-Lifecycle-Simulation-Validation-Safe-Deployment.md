ADR-IDENTITY-060 — Authorization Criticality Policy Lifecycle, Simulation, Validation & Safe Deployment

Status: Proposed
Version: 1.0
Date: 2026-09-07
Domain: Identity / Authorization / Security / Policy Engine / Risk / Governance
Priority: Critical
Type: Architecture / Security / Policy Governance / Runtime Safety

1. Contexto

El ADR-IDENTITY-059 estableció un Authorization Criticality Policy Model centralizado para determinar la criticidad efectiva de una operación antes de decidir su elegibilidad para caché y su nivel de frescura.

La criticidad puede depender de:

Resource Criticality.
Action Criticality.
Dynamic Risk.
Context Criticality.
Security State.
Tenant Criticality.
Delegation Criticality.
MFA Criticality.
Platform Minimum.

El resultado puede elevar una operación entre:

C3 → C2 → C1 → C0

pero nunca reducir un requisito de seguridad obligatorio.

La criticidad es además un elemento de seguridad distribuida: determina qué clase de freshness puede utilizar Authorization Runtime y, por tanto, qué tan tolerante puede ser el sistema ante información potencialmente desactualizada.

Por esta razón, cambiar una Criticality Policy no puede tratarse como una simple modificación de configuración.

Una política incorrecta podría:

transformar una operación C0 en una operación aparentemente menos crítica;
permitir el uso de una caché que no corresponde;
aumentar la ventana de exposición ante una revocación;
modificar el comportamiento frente a riesgos elevados;
alterar el comportamiento de organizaciones o recursos;
introducir conflictos entre reglas;
producir shadowed rules;
generar políticas imposibles o inalcanzables;
provocar una regresión de seguridad durante un despliegue.

Por lo tanto, se requiere un lifecycle específico para las Criticality Policies que permita:

creación;
validación;
simulación;
revisión;
aprobación;
activación controlada;
observación;
rollback seguro;
retiro;
auditoría completa.
2. Problema

EVOXA necesita garantizar que las modificaciones de las reglas que determinan la criticidad de autorización sean:

deterministas;
versionadas;
validables antes de producción;
simulables contra escenarios conocidos;
comparables contra la política actualmente activa;
resistentes a configuraciones contradictorias;
monotónicas respecto de requisitos mínimos de seguridad;
auditables;
reversibles;
tenant-aware;
compatibles con el modelo RBAC + ABAC + Ownership + Delegation + Policy Engine;
seguras durante despliegues parciales o distribuidos.

Sin este mecanismo, una modificación aparentemente pequeña podría alterar indirectamente:

Criticality
    ↓
Freshness Class
    ↓
Cache Eligibility
    ↓
Authorization Evaluation Behavior
    ↓
Security Exposure

Por lo tanto, Criticality Policy debe ser tratada como una configuración de seguridad gobernada y no como configuración operacional ordinaria.

3. Decisión arquitectónica

Se establece un Authorization Criticality Policy Lifecycle formal y obligatorio.

Toda Criticality Policy deberá atravesar el siguiente lifecycle:

DRAFT
   ↓
VALIDATING
   ↓
SIMULATION
   ↓
APPROVAL_REQUIRED
   ↓
APPROVED
   ↓
DEPLOYING
   ↓
ACTIVE
   ↓
SUPERSEDED
   ↓
RETIRED

Con estados adicionales para situaciones excepcionales:

VALIDATING
    ↓
REJECTED

DEPLOYING
    ↓
DEPLOYMENT_FAILED

ACTIVE
    ↓
ROLLBACK_REQUESTED
    ↓
ROLLING_BACK
    ↓
ACTIVE(previous version)

La transición entre estados deberá estar controlada por reglas de governance y no podrá ser realizada arbitrariamente por una aplicación cliente.

4. Principios fundamentales
4.1 Security First

Ninguna política nueva podrá reducir una garantía mínima de seguridad establecida por la plataforma.

Una organización podrá endurecer una política.

No podrá debilitar un PLATFORM_MINIMUM.

Organization Policy
       ↓
may harden
       ↓
Platform Minimum

may NOT weaken
       ↓
Platform Minimum
4.2 Monotonicidad de seguridad

Las modificaciones de una Criticality Policy deberán preservar la monotonicidad de seguridad.

Permitido:

C3 → C2
C2 → C1
C1 → C0
C3 → C1
C3 → C0

No permitido cuando contradice un requisito obligatorio:

C0 → C1
C1 → C2
C2 → C3

La evaluación de una política no podrá utilizar un cambio de configuración para reducir una garantía impuesta por una política superior.

5. Modelo de Criticality Policy

Una Criticality Policy deberá mantener, como mínimo, la estructura conceptual definida en ADR-059:

CriticalityPolicy
├── id
├── name
├── version
├── status
├── scope
├── priority
├── target
├── conditions
├── base_criticality
├── elevations
├── security_overrides
├── freshness_profile
├── effective_from
├── effective_until
└── metadata

Se agregan para governance:

CriticalityPolicyGovernance
├── created_by
├── created_at
├── modified_by
├── modified_at
├── approved_by
├── approved_at
├── validation_result
├── simulation_result
├── deployment_id
├── parent_version
├── change_reason
├── rollback_reference
└── policy_hash

Los nombres concretos de persistencia quedan pendientes de la especificación de implementación.

6. Lifecycle de una Criticality Policy
6.1 DRAFT

Una nueva política comienza en:

DRAFT

En este estado:

puede ser creada;
puede ser modificada;
no afecta Authorization Runtime;
no puede modificar la criticidad efectiva;
no puede invalidar producción;
puede ser evaluada mediante herramientas de análisis;
puede ser preparada para simulación.

Una política DRAFT no debe ser considerada por el runtime de autorización.

7. VALIDATING

Al solicitar validación:

DRAFT
   ↓
VALIDATING

se ejecutan validaciones estáticas y estructurales.

Validaciones mínimas
Schema
estructura válida;
campos requeridos;
tipos correctos;
versiones válidas;
estados válidos.
Target

Validar:

resource;
action;
actor;
organization;
contexto;
scope.
Conditions

Validar:

referencias válidas;
tipos compatibles;
operadores permitidos;
funciones permitidas;
expresiones deterministas.
Criticality

Validar:

C0;
C1;
C2;
C3.

No deberán existir valores arbitrarios.

8. Validación de monotonicidad

El validator deberá verificar que la nueva política no reduzca requisitos mínimos.

Ejemplo:

Platform Minimum:
DELETE USER = C0

Una organización intenta:

DELETE USER = C2

Resultado:

REJECT

No se debe permitir que una política de menor autoridad reduzca la criticidad mínima.

9. Validación de precedencia

Las políticas deberán evaluarse según la precedencia establecida en ADR-059:

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
Contextual Policy
        ↓
Organization Policy
        ↓
Default Operation Matrix

La existencia de una política de menor prioridad no debe ocultar o invalidar una regla superior.

10. Detección de conflictos

El sistema deberá detectar al menos:

Contradictory Rules

Reglas que establecen resultados incompatibles para el mismo ámbito.

Ejemplo:

Rule A → operation = "export" → C1
Rule B → operation = "export" → C3

si ambas aplican al mismo contexto y no existe una resolución válida de precedencia.

Shadowed Rules

Una regla que nunca puede tener efecto porque otra regla superior siempre la cubre.

Rule A
priority = 100
target = users.*

Rule B
priority = 50
target = users.delete

Si A establece siempre una condición que hace imposible que B tenga efecto, B deberá ser reportada como SHADOWED.

Unreachable Rules

Reglas cuyas condiciones nunca pueden cumplirse.

Ejemplo conceptual:

risk = LOW
AND
risk = CRITICAL

Resultado:

UNREACHABLE
Redundant Rules

Reglas que producen exactamente el mismo resultado que otra regla de mayor prioridad y no agregan comportamiento.

Estas podrán generar:

WARNING

o ser rechazadas dependiendo de governance.

11. SIMULATION

Una política validada deberá poder pasar a:

SIMULATION

antes de solicitar aprobación.

La simulación permitirá comparar:

Current Policy
        VS
Candidate Policy

sobre un conjunto de escenarios.

12. Objetivo de la simulación

La simulación debe responder:

"¿Qué cambiaría si esta política estuviera activa?"

Debe permitir identificar:

cambios de C0/C1/C2/C3;
cambios de freshness;
operaciones que pasarían a ser cacheables;
operaciones que dejarían de ser cacheables;
cambios provocados por riesgo;
cambios por tenant;
cambios por MFA;
cambios por delegation;
cambios por security state.
13. Simulation Scenarios

Los escenarios deberán representar diferentes contextos.

Ejemplo:

Scenario:
actor = user
tenant = organization-A
resource = user
action = update
risk = LOW
mfa = VERIFIED
session = ACTIVE
ownership = TRUE

Resultado:

Current:
C2 / F2-SHORT

Candidate:
C1 / F1

El sistema deberá registrar el delta.

14. Security Regression Simulation

La simulación deberá buscar específicamente regresiones.

Ejemplo:

Current:
membership.revoke = C0

Candidate:
membership.revoke = C1

Resultado:

SECURITY REGRESSION

La política no podrá ser aprobada.

15. Simulation Result

El resultado conceptual deberá contener:

SimulationResult
├── policy_version
├── baseline_version
├── scenarios_evaluated
├── decisions_changed
├── criticality_changes
├── freshness_changes
├── cacheability_changes
├── security_regressions
├── conflicts
├── warnings
└── status

Estados posibles:

PASS
PASS_WITH_WARNINGS
FAIL
SECURITY_REGRESSION
16. APPROVAL_REQUIRED

Una política que haya pasado las validaciones y simulaciones deberá entrar en:

APPROVAL_REQUIRED

La activación no deberá producirse automáticamente.

La separación será:

Author
   ↓
Validator
   ↓
Simulator
   ↓
Approver
   ↓
Deployment

Esto evita que una misma operación pueda:

create → validate → activate

sin control adicional.

17. Separation of Duties

Para políticas de seguridad críticas, deberá existir separación entre:

quien crea;
quien valida;
quien aprueba;
quien despliega.

El nivel exacto de separación queda sujeto al modelo final de governance, pero la arquitectura debe soportarlo.

18. APPROVED

Una política aprobada:

APPROVED

ha superado:

validación;
análisis;
simulación;
revisión;
aprobación requerida.

Pero todavía no está activa.

Esto permite separar:

Policy Governance

de:

Policy Runtime Activation
19. DEPLOYING

La transición:

APPROVED
   ↓
DEPLOYING

representa la distribución controlada de la política.

El deployment deberá garantizar que Authorization Runtime nunca observe una política parcialmente escrita o inválida.

20. Atomic Policy Activation

La activación deberá ser atómica desde la perspectiva del runtime.

El runtime deberá observar:

Version N

o:

Version N+1

pero no un estado parcialmente mezclado:

N + N+1

Por tanto, la política deberá estar versionada y publicada como una unidad coherente.

21. Policy Version

Cada cambio significativo deberá generar una nueva versión.

Ejemplo:

Policy:
authorization-criticality

Versiones:

1.0
1.1
1.2
2.0

Las reglas exactas de semantic versioning quedan pendientes de definición.

La versión activa deberá estar explícitamente identificada.

22. Policy Hash

Cada versión desplegable deberá disponer de una representación canónica y un hash.

Conceptualmente:

Canonical Policy
       ↓
Canonical Serialization
       ↓
Hash

Esto permite verificar:

integridad;
consistencia entre nodos;
identificación exacta de versión;
detección de corrupción.

El algoritmo criptográfico exacto queda pendiente.

23. ACTIVE

Una política pasa a:

ACTIVE

cuando:

está aprobada;
fue desplegada;
fue validada en runtime;
su integridad fue comprobada;
la versión activa fue confirmada.

Solo una versión compatible con las reglas de governance deberá considerarse activa para un mismo ámbito.

24. Propagación distribuida

En un entorno distribuido:

Policy Store
      ↓
Policy Distribution
      ↓
Authorization Runtime instances
      ↓
Local compiled policy

cada instancia deberá conocer:

policy ID;
policy version;
policy hash;
effective time;
generation/version metadata.

Una instancia que no pueda validar la política recibida no deberá activarla.

25. Safe Deployment

La activación deberá utilizar un mecanismo de despliegue progresivo.

Modelo recomendado:

APPROVED
   ↓
DEPLOY
   ↓
VALIDATE
   ↓
CANARY
   ↓
OBSERVE
   ↓
PROMOTE
   ↓
ACTIVE

La implementación exacta de canary queda pendiente.

26. Canary Validation

Durante el canary se deberá observar:

errores de evaluación;
cambios inesperados de criticality;
policy conflicts;
cache invalidations;
freshness violations;
authorization mismatches;
latencia;
aumento de DENY;
reducción inesperada de DENY;
cambios por tenant;
anomalías de riesgo.

El objetivo no es únicamente verificar disponibilidad, sino detectar regresiones de seguridad o comportamiento.

27. Shadow Evaluation

Cuando sea viable, el sistema podrá ejecutar:

Current Policy

y:

Candidate Policy

en paralelo sin utilizar todavía el resultado candidato para autorizar.

Ejemplo:

Request
  ├── Current → C1
  └── Candidate → C0

Se registra:

DELTA = C1 → C0

Esto permite validar comportamiento antes de convertir la política en autoridad de runtime.

28. Regla para Shadow Evaluation

La evaluación shadow:

no modifica la decisión real;
no modifica permisos;
no concede acceso;
no revoca acceso;
no modifica sesiones;
no modifica MFA;
no modifica cachés de autorización;
no modifica estado de seguridad.

Es exclusivamente:

Observe → Compare → Analyze
29. Safe Promotion

La promoción de:

CANARY

a:

ACTIVE

requiere que se cumplan los criterios definidos para esa política.

Como mínimo:

no existen security regressions;
no existen validaciones críticas fallidas;
no existen conflictos no resueltos;
la integridad es válida;
las instancias pueden cargar la versión;
las métricas están dentro de los límites;
la política mantiene platform minimums.

Los umbrales exactos quedan pendientes.

30. Rollback

Toda política activa deberá tener un mecanismo de rollback seguro.

Ejemplo:

ACTIVE v12
     ↓
problem detected
     ↓
ROLLBACK_REQUESTED
     ↓
ROLLING_BACK
     ↓
ACTIVE v11

El rollback deberá tratarse como una nueva transición de seguridad, no como una simple restauración de memoria.

31. Rollback no significa reutilizar caché antigua

Una regla crítica:

El rollback de una política no autoriza automáticamente la reutilización de decisiones de autorización cacheadas bajo otra versión.

Ejemplo:

Policy v11
   ↓
Cache decisions

Policy v12
   ↓
Active

Rollback → v11

Las decisiones cacheadas deberán evaluarse según:

policy version;
security epoch;
entity version;
cache generation;
invalidation state;
freshness requirements.

Una entrada anterior no debe considerarse válida únicamente porque su política volvió a estar activa.

32. Policy Epoch / Generation

Los cambios relevantes de política deberán poder incrementar una versión de seguridad o generation cuando corresponda.

Conceptualmente:

Policy Generation 41
        ↓
Policy change
        ↓
Policy Generation 42

Las entradas incompatibles deberán quedar invalidadas.

Esto se integra con:

ADR-050;
ADR-053;
ADR-055;
ADR-056;
ADR-057;
ADR-058.
33. Emergency Security Change

Debe existir un mecanismo para cambios de emergencia.

Ejemplo:

ACTIVE
   ↓
EMERGENCY SECURITY CHANGE
   ↓
VALIDATE
   ↓
ACTIVATE

Un cambio de emergencia podrá reducir temporalmente algunas etapas de governance solo cuando sea estrictamente necesario, pero no podrá omitir:

validación estructural;
platform minimums;
integridad;
auditoría;
identificación del actor;
trazabilidad;
capacidad de rollback.

La revisión posterior deberá ser obligatoria.

34. Policy Retirement

Una política que deja de ser válida pasa a:

SUPERSEDED

y posteriormente:

RETIRED

Una política retirada:

no puede activarse normalmente;
no puede ser seleccionada por runtime;
permanece disponible para auditoría;
conserva su historial;
conserva su hash;
conserva sus relaciones de governance.
35. Audit Trail

Toda transición de lifecycle deberá generar un registro auditable.

Ejemplo:

POLICY_CREATED
POLICY_MODIFIED
POLICY_VALIDATION_STARTED
POLICY_VALIDATION_PASSED
POLICY_VALIDATION_FAILED
POLICY_SIMULATION_STARTED
POLICY_SIMULATION_COMPLETED
POLICY_APPROVAL_REQUESTED
POLICY_APPROVED
POLICY_REJECTED
POLICY_DEPLOYMENT_STARTED
POLICY_DEPLOYMENT_COMPLETED
POLICY_ACTIVATED
POLICY_ROLLBACK_REQUESTED
POLICY_ROLLED_BACK
POLICY_SUPERSEDED
POLICY_RETIRED

Estos eventos deberán seguir las reglas de Audit y Event Architecture ya definidas.

36. No secrets

Las Criticality Policies no deberán contener:

passwords;
tokens;
API keys;
credenciales;
secretos criptográficos;
datos sensibles innecesarios.

Tampoco deberán aparecer en:

logs;
traces;
eventos;
mensajes de error.
37. AI Governance

La IA podrá asistir en:

generar una propuesta de política;
detectar conflictos;
analizar simulaciones;
encontrar reglas redundantes;
detectar posibles regresiones;
explicar diferencias;
recomendar endurecimiento;
identificar patrones de riesgo.

Pero:

AI no puede activar directamente una Criticality Policy.

La IA tampoco podrá:

saltarse validaciones;
reducir Platform Minimum;
aprobarse a sí misma;
modificar directamente el runtime;
desactivar controles;
convertir una operación C0 en C1/C2/C3.

La decisión final deberá permanecer dentro del governance autorizado.

38. Policy Diff

Antes de aprobar una nueva versión deberá existir un diff estructurado.

Ejemplo:

Policy v10 → v11

users.update:
    C2 → C1

users.delete:
    C0 → C0

membership.revoke:
    C0 → C0

delegation.create:
    C1 → C1

Además:

Freshness:
F2-SHORT → F1

Cacheability:
Eligible → Restricted

Esto facilita revisión humana y auditoría.

39. Risk-aware Simulation

La simulación deberá probar diferentes niveles de riesgo:

LOW
MEDIUM
HIGH
CRITICAL

Ejemplo:

LOW:
operation → C3

HIGH:
operation → C1

CRITICAL:
operation → C0

El objetivo es verificar que una política dinámica mantenga la monotonicidad de seguridad.

40. MFA-aware Simulation

También deberán existir escenarios con:

MFA absent
MFA pending
MFA verified
MFA expired
MFA revoked

Una política no deberá reducir accidentalmente la criticidad de una operación cuando el contexto de autenticación sea insuficiente.

41. Delegation-aware Simulation

Las simulaciones deberán incluir:

actor sin delegation;
delegation válida;
delegation expirada;
delegation revocada;
delegation cross-tenant;
delegation con privilegios insuficientes.

La existencia de una delegation nunca deberá reducir una restricción superior.

42. Tenant-aware Validation

Toda política deberá ser evaluada considerando:

PLATFORM
ORGANIZATION
RESOURCE

Una política organizacional no podrá afectar accidentalmente otro tenant.

Un escenario con:

tenant mismatch

deberá producir comportamiento seguro:

DENY

cuando corresponda.

43. Deployment Failure

Si el deployment falla:

DEPLOYING
   ↓
DEPLOYMENT_FAILED

la política candidata no deberá convertirse en activa.

El runtime deberá mantener una versión conocida y válida.

Nunca:

invalid candidate
      ↓
ACTIVE
44. Partial Deployment

Si solo algunas instancias reciben la nueva política:

Node A → v12
Node B → v11
Node C → v11

el sistema deberá poder detectar:

policy version mismatch;
hash mismatch;
deployment lag;
inconsistent runtime state.

La arquitectura deberá evitar que una distribución parcialmente desplegada sea interpretada como despliegue exitoso.

Los criterios exactos de tolerancia quedan pendientes.

45. Policy Activation and Cache Invalidation

Una activación que cambie la criticidad efectiva o freshness profile deberá provocar las invalidaciones necesarias.

Flujo conceptual:

Policy Change
     ↓
Policy Version Change
     ↓
Security/Policy Epoch Update
     ↓
Transactional Outbox
     ↓
Event Platform
     ↓
Cache Invalidation
     ↓
Authorization Runtime

Esto mantiene consistencia con ADR-053.

46. Policy Tightening vs Policy Relaxation

Se distinguen dos clases:

Tightening

Ejemplo:

C2 → C1

Puede incrementar controles y reducir cacheability.

Relaxation

Ejemplo:

C1 → C2

Debe considerarse una modificación de seguridad de mayor riesgo y requerir validación adicional.

En particular:

Una reducción de criticidad nunca debe ser aprobada únicamente porque la nueva regla sea sintácticamente válida.

Debe demostrar que no viola:

Platform Minimum;
Security State;
Tenant Isolation;
Resource Security;
Dynamic Risk requirements.
47. Policy Change Risk Classification

Los cambios podrán clasificarse conceptualmente como:

LOW
MEDIUM
HIGH
CRITICAL

Un cambio:

C0 → C0

que no cambia comportamiento puede tener menor impacto.

Un cambio:

C1 → C3

deberá ser considerado potencialmente crítico y sujeto a controles adicionales.

La clasificación exacta queda pendiente.

48. Version Compatibility

Las políticas deberán ser compatibles con:

Authorization Runtime;
Policy DSL;
Resource Registry;
Action Registry;
Attribute Registry;
Freshness Profiles;
Criticality Matrix.

Una política que utilice una versión incompatible de alguno de estos componentes deberá ser rechazada antes de activación.

49. Migration

Cuando evolucione el modelo de Criticality Policy:

DSL v1
   ↓
DSL v2

no se deberá asumir que una política antigua es automáticamente compatible.

Deberá existir:

Policy Migration

o una estrategia explícita de compatibilidad.

Las migraciones deberán ser:

deterministas;
versionadas;
verificables;
auditables;
reversibles cuando sea posible.
50. Disaster Recovery

Las Criticality Policies activas deberán poder recuperarse junto con:

policy versions;
policy hashes;
security epochs;
deployment metadata;
audit history.

La recuperación no deberá activar automáticamente una política no validada.

Después de DR:

Restore
   ↓
Validate
   ↓
Verify Version
   ↓
Verify Hash
   ↓
Verify Epoch
   ↓
Validate Runtime
   ↓
Activate
51. Observability

Deberán existir métricas para:

Lifecycle
policies created;
policies validated;
validation failures;
simulation failures;
approval latency;
deployment failures;
rollback count.
Runtime
active policy version;
policy hash;
policy evaluation errors;
policy mismatches;
policy load failures.
Security
criticality changes;
security regressions;
unexpected C0/C1/C2/C3 distribution;
freshness profile changes;
cache invalidation triggered by policy changes.
52. Alertas

Se deberán considerar alertas para:

Unexpected criticality downgrade
Policy hash mismatch
Runtime policy version mismatch
Failed security validation
Deployment inconsistency
Unexpected increase in ALLOW
Unexpected decrease in DENY
Policy rollback
Criticality policy activation failure

Los thresholds concretos quedan pendientes.

53. Performance

La validación y simulación podrán ser operaciones costosas.

No deberán bloquear innecesariamente el Authorization Runtime.

La arquitectura deberá separar:

Policy Governance Plane

de:

Authorization Runtime Plane

El runtime deberá utilizar únicamente políticas ya validadas y activables.

54. Governance Plane vs Runtime Plane
┌─────────────────────────────┐
│     Policy Governance       │
│                             │
│ Draft                       │
│ Validation                  │
│ Simulation                  │
│ Approval                    │
│ Deployment                  │
│ Audit                       │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│     Policy Distribution     │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ Authorization Runtime       │
│                             │
│ Active Policies             │
│ Criticality Evaluation      │
│ Freshness Decision           │
│ Authorization Decision      │
└─────────────────────────────┘

Esto evita que la complejidad del governance contamine el path crítico de autorización.

55. No-Bypass Rule

Ningún consumidor deberá poder ejecutar:

activate_policy()

directamente contra el runtime sin pasar por el lifecycle correspondiente.

Tampoco podrá:

set_criticality(C3)

para evitar las reglas superiores.

El único mecanismo válido será el Authorization Policy Engine y su governance.

56. Consecuencias positivas

Esta decisión proporciona:

control formal de cambios de seguridad;
simulación antes de producción;
detección de regresiones;
separación de funciones;
trazabilidad completa;
rollback controlado;
despliegues progresivos;
protección contra políticas inválidas;
compatibilidad con cache consistency;
integración con risk;
integración con MFA;
integración con tenant isolation;
soporte para AI-assisted governance sin entregar autoridad a la IA.
57. Consecuencias negativas

Introduce:

mayor complejidad operacional;
necesidad de tooling de governance;
mayor tiempo para cambios críticos;
necesidad de mantener versiones;
necesidad de ejecutar simulaciones;
mayor cantidad de estados;
necesidad de auditoría y observabilidad.

Sin embargo, esta complejidad se considera justificada porque Criticality Policy forma parte indirecta del security decision path.

58. Alternativas rechazadas
58.1 Editar políticas directamente en producción

Rechazado.

No existe suficiente control, trazabilidad ni validación.

58.2 Activación automática después de validación

Rechazado.

La validación técnica no sustituye governance ni aprobación.

58.3 First-Match-Wins

Rechazado.

Puede ocultar reglas de seguridad superiores.

58.4 Last-Match-Wins

Rechazado.

Puede permitir que una regla posterior reduzca una restricción crítica.

58.5 TTL como mecanismo de seguridad

Rechazado.

ADR-056 establece que TTL no sustituye:

security epoch;
policy version;
entity version;
invalidation;
fresh authorization.
58.6 Rollback sin invalidación

Rechazado.

Puede dejar decisiones cacheadas incompatibles con la versión efectiva.

58.7 AI Autonomous Policy Activation

Rechazado.

La IA puede asistir, pero no puede tener autoridad para modificar controles de seguridad.

59. Non-Negotiables
Criticality Policy es una configuración de seguridad.
Toda política debe estar versionada.
Toda política debe pasar validación.
Toda política crítica debe poder ser simulada.
Las regresiones de seguridad deben bloquear la activación.
Platform Minimum no puede ser debilitado.
La criticidad es monotónica respecto de requisitos obligatorios.
Organization Policy no puede reducir Platform Minimum.
Tenant isolation no puede ser debilitado.
AI no puede activar políticas.
Shadow evaluation no puede modificar autorización real.
Policy activation debe ser atómica.
Runtime no debe consumir políticas inválidas.
Deployment parcial debe ser detectable.
Rollback no implica reutilización automática de caché.
Policy changes deben integrarse con cache invalidation.
Todo cambio debe ser auditable.
No secrets en policies/logs/events.
Recovery debe validar policy version/hash/epoch.
Unknown/invalid policy state debe fallar cerrado.
60. Trazabilidad
Blueprint
Identity
Authorization
Security Policy Engine
Event-Driven Architecture
Caching
Observability
Testing
Governance
Implementation Stories

Principalmente:

IS-IDENTITY-006
Authorization & Permission Evaluation

IS-IDENTITY-007
Role & Permission Management

IS-IDENTITY-010
Multi-Factor Authentication

IS-IDENTITY-011
Security Policy Engine

IS-IDENTITY-012
Audit & Security Events

IS-IDENTITY-013
Domain Events & Outbox

IS-IDENTITY-015
Identity Persistence

IS-IDENTITY-016
Identity Testing & Quality

IS-IDENTITY-017
Identity Runtime & Deployment
ADRs relacionados
ADR-IDENTITY-045
Authorization Policy Model/Evaluation

ADR-IDENTITY-046
Policy Language/Schema/Governance

ADR-IDENTITY-047
Authorization Policy Engine Technology

ADR-IDENTITY-048
Policy DSL & Expression Language

ADR-IDENTITY-049
Policy Evaluation Runtime

ADR-IDENTITY-050
Policy Cache & Consistency

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
61. Dependencias

Este ADR depende de:

Policy DSL;
Attribute Registry;
Resource Registry;
Action Registry;
Authorization Runtime;
Security Policy Engine;
Audit Service;
Event Platform;
Transactional Outbox;
Cache Invalidation;
Security Epoch;
Policy Versioning;
Risk Context;
MFA State;
Tenant Context;
Deployment/Release Infrastructure.
62. Decisiones pendientes

Este ADR deja explícitamente pendientes para futuras especificaciones:

Estados finales exactos del lifecycle.
Semantic versioning exacto.
Policy hash algorithm.
Canonical serialization.
Modelo definitivo de approval.
Número mínimo de aprobadores.
Separation of Duties exacta.
Thresholds de security regression.
Thresholds de canary.
Duración del shadow evaluation.
Criterios exactos de promotion.
Rollback automático vs manual.
Emergency change protocol detallado.
Policy migration protocol.
Deployment consistency SLA.
Policy propagation SLA.
Multi-region policy activation.
Policy generation/epoch exacto.
Retención de versiones históricas.
Retención de simulation results.
Exactos eventos de lifecycle.
Integración definitiva con CI/CD.
Política de organization-level approval.
Herramienta final de policy simulation.
Exacto catálogo de cambios considerados CRITICAL.

Estas decisiones no deben resolverse informalmente durante la implementación; deberán quedar documentadas en ADRs o Engineering Specifications correspondientes.

63. Acceptance Criteria
AC-01 — Lifecycle

Given una nueva Criticality Policy
When es creada
Then debe comenzar en DRAFT.

AC-02 — Validation

Given una política inválida
When se ejecuta validation
Then no puede pasar a APPROVED.

AC-03 — Platform Minimum

Given un Platform Minimum C0
When una organización intenta definir C2
Then la política debe ser rechazada.

AC-04 — Simulation

Given una política validada
When se solicita simulation
Then debe compararse contra la versión baseline.

AC-05 — Regression

Given una reducción de criticidad que viola seguridad
When se ejecuta simulation
Then debe producir SECURITY_REGRESSION.

AC-06 — Shadow

Given una candidate policy
When se ejecuta shadow evaluation
Then no debe modificar la autorización real.

AC-07 — Approval

Given una política simulada correctamente
When requiere aprobación
Then no debe activarse automáticamente.

AC-08 — Atomic Activation

Given una política aprobada
When se activa
Then runtime debe observar una versión coherente.

AC-09 — Integrity

Given una política desplegada
When hash no coincide
Then runtime debe rechazarla.

AC-10 — Partial Deployment

Given diferentes versiones en diferentes instancias
When se detecta inconsistencia
Then debe quedar observable y no declararse deployment completo.

AC-11 — Rollback

Given una política activa defectuosa
When se ejecuta rollback
Then debe restaurarse una versión válida y auditable.

AC-12 — Cache

Given un cambio de política que afecta freshness
When se activa
Then deben producirse las invalidaciones correspondientes.

AC-13 — Cache Reuse

Given rollback a una versión anterior
When existen decisiones cacheadas
Then no deben reutilizarse automáticamente sin validación de freshness/version/epoch.

AC-14 — Tenant Isolation

Given una política organization-scoped
When se evalúa otro tenant
Then no debe afectar su autorización.

AC-15 — Risk

Given risk CRITICAL
When se evalúa una operación
Then la política debe poder elevar la criticidad a C0 cuando corresponda.

AC-16 — MFA

Given MFA no válido
When la operación requiere seguridad reforzada
Then la política no puede degradar el requisito.

AC-17 — Delegation

Given una delegation
When se calcula criticality
Then delegation no puede reducir un Platform Minimum.

AC-18 — AI

Given una recomendación generada por AI
When propone una Criticality Policy
Then no puede activarla directamente.

AC-19 — Audit

Given cualquier transición del lifecycle
When ocurre
Then debe existir trazabilidad auditable.

AC-20 — Fail Closed

Given una política inválida, incompatible o indeterminada
When Authorization Runtime intenta utilizarla
Then no debe producir un ALLOW inseguro.

64. Resultado arquitectónico

Con este ADR, EVOXA pasa de tener únicamente un modelo de criticidad a disponer de un mecanismo completo para gobernar su evolución:

Criticality Policy
       │
       ▼
   Validation
       │
       ▼
   Simulation
       │
       ▼
 Security Regression
     Analysis
       │
       ▼
    Approval
       │
       ▼
  Safe Deployment
       │
       ▼
    Activation
       │
       ▼
Cache/Freshness
   Integration
       │
       ▼
 Observability
       │
       ▼
Rollback / Evolution

La consecuencia más importante es que la criticidad deja de ser una configuración estática y pasa a ser una política de seguridad versionada, simulable, auditable y desplegable de forma controlada.

Esto completa de forma natural la línea iniciada en ADR-IDENTITY-057 → 058 → 059 → 060.

Siguiente ADR recomendado

ADR-IDENTITY-061 — Authorization Criticality Policy Testing, Regression Detection & Continuous Verification

Este siguiente ADR debería definir cómo probar automáticamente las Criticality Policies, mantener un catálogo de escenarios de seguridad, detectar regresiones entre versiones y convertir las simulaciones/validaciones en una barrera permanente de CI/CD antes de permitir cualquier activación.
