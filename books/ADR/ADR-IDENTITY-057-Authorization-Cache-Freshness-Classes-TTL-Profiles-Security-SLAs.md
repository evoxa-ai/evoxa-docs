ADR-IDENTITY-057 — Authorization Cache Freshness Classes, TTL Profiles & Security SLAs

Status: Proposed
Version: 1.0
Date: 2026-09-07
Domain: Identity / Authorization / Distributed Systems / Security / Performance
Priority: Critical
Type: Architecture / Security / Consistency / Performance

1. Contexto

Los ADR anteriores establecieron que el sistema de autorización de EVOXA utiliza caché para reducir latencia, pero que la caché no constituye la fuente de verdad.

La autoridad permanece en PostgreSQL y la decisión final pertenece al Authorization Policy Engine / Authorization Runtime.

Los ADR-050 a ADR-056 definieron progresivamente:

Cache L1/L2.
Versiones y epochs de seguridad.
Invalidación basada en eventos.
Consistencia y propagación.
Degradación y recuperación.
Reconciliación y warm-up.
Estados FRESH, ACCEPTABLE, STALE, INVALID.
Maximum Authorization Staleness (MAS).
Clases de frescura conceptuales F0, F1, F2, F3.
Prioridad de revocaciones sobre concesiones.
Requisito de que una autorización incierta nunca produzca un ALLOW en operaciones críticas.

Este ADR transforma esos principios en un modelo operativo de:

Freshness Classes
TTL Profiles
Security SLAs
Maximum Authorization Staleness
Revocation Propagation Guarantees
Grant Propagation Guarantees
Comportamiento bajo degradación

Los valores exactos establecidos aquí deberán considerarse objetivos arquitectónicos iniciales sujetos a validación mediante pruebas de carga, seguridad, distribución y operación.

2. Problema

Una autorización cacheada puede ser técnicamente válida desde el punto de vista de TTL y, sin embargo, ser incorrecta desde el punto de vista de seguridad.

Ejemplo:

10:00  Usuario autorizado
10:00  Decisión ALLOW almacenada en cache
10:02  Membership revocado
10:05  TTL todavía vigente

Una estrategia basada exclusivamente en TTL podría continuar devolviendo:

ALLOW

aunque el usuario ya no tenga acceso.

Por lo tanto:

TTL no puede definir por sí solo la frescura de una autorización.

La frescura debe considerar simultáneamente:

TTL
+
Security Epoch
+
Policy Version
+
Entity Version
+
Context Freshness
+
Invalidation State
+
Cache Generation
+
Operation Criticality
3. Decisión arquitectónica

EVOXA utilizará un modelo de Freshness Classes + TTL Profiles + Security SLAs.

La decisión de utilizar una entrada de caché deberá depender de:

Authorization Decision
        ↓
Operation Criticality
        ↓
Freshness Class
        ↓
TTL Profile
        ↓
Security Epoch
        ↓
Policy / Entity Versions
        ↓
Context Freshness
        ↓
Invalidation State
        ↓
Cache Generation
        ↓
ALLOW / DENY / CHALLENGE / REAUTHENTICATE / REQUIRE_MFA / REVOKE

La caché podrá optimizar una autorización solamente cuando todos los requisitos de seguridad correspondientes a su clase se cumplan.

4. Freshness Classes

Se establecen cuatro clases conceptuales.

Clase	Nombre	Uso	Nivel de frescura
F0	Authoritative	Operaciones críticas	Máxima
F1	Security Validated	Operaciones sensibles	Muy alta
F2	Controlled Eventual	Operaciones normales	Controlada
F3	Restricted Degraded	Degradación/incertidumbre	Restringida
5. F0 — Authoritative Freshness
5.1 Propósito

F0 representa la máxima garantía de frescura.

Debe utilizarse para operaciones donde una autorización incorrecta podría producir:

escalamiento de privilegios;
acceso cross-tenant;
reactivación de una cuenta revocada;
modificación de permisos;
alteración de configuración de seguridad;
bypass de MFA;
administración de identidad;
operaciones administrativas críticas.
5.2 Reglas

Para F0:

Cache != autoridad suficiente

La decisión debe validarse contra estado autoritativo o mediante un mecanismo equivalente que garantice frescura fuerte.

Debe validarse como mínimo:

tenant
actor
membership
session
security epoch
policy version
relevant entity version
MFA state
delegation state

Cuando alguno de estos elementos no pueda validarse:

DENY

o una decisión de seguridad equivalente:

REAUTHENTICATE
REQUIRE_MFA
CHALLENGE

según corresponda.

5.3 TTL

F0 no dependerá de un TTL convencional para determinar si un ALLOW sigue siendo válido.

Conceptualmente:

TTL = no security guarantee

El TTL puede existir como optimización técnica, pero no concede permiso para utilizar una decisión obsoleta.

6. F1 — Security Validated

F1 está destinada a operaciones sensibles que requieren una fuerte garantía de consistencia, aunque puedan tolerar una pequeña optimización mediante cache cuando el estado de seguridad ha sido validado.

Ejemplos:

cambios de información sensible;
operaciones sobre recursos importantes;
operaciones con privilegios elevados;
acciones relacionadas con configuración;
operaciones que requieren MFA;
operaciones relacionadas con delegaciones;
operaciones administrativas no críticas.
6.1 Requisitos

Una entrada F1 podrá utilizarse únicamente cuando:

tenant_id válido
AND
security_epoch válido
AND
policy_version válida
AND
entity_version válida
AND
context_fingerprint válido
AND
no existe invalidación pendiente
AND
cache_generation vigente
AND
TTL vigente

Si cualquiera de estas condiciones falla:

Fresh Evaluation
7. F2 — Controlled Eventual

F2 permite utilizar consistencia eventual controlada para operaciones normales.

Está orientada a:

lecturas normales;
consultas de recursos;
dashboards;
navegación;
operaciones de bajo riesgo;
consultas repetitivas;
escenarios de alta frecuencia.

F2 puede utilizar cache siempre que:

el tenant sea correcto;
el actor sea correcto;
membership sea válida;
security epoch sea válido;
policy version sea compatible;
entity version sea compatible;
no exista invalidación conocida;
el contexto sea compatible;
la entrada no haya superado el MAS;
la operación sea elegible para F2.
8. F3 — Restricted Degraded

F3 representa un estado de incertidumbre o degradación.

No significa:

"La autorización es válida durante más tiempo."

Significa:

"La infraestructura de autorización está degradada y las operaciones permitidas deben limitarse."

F3 puede producir:

DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA

según la operación.

No podrá utilizarse para convertir incertidumbre en:

ALLOW
9. Freshness State

Las Freshness Classes se combinan con los estados definidos en ADR-056.

                    ┌──────────────┐
                    │    FRESH     │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ ACCEPTABLE   │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │    STALE     │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │   INVALID    │
                    └──────────────┘

La transición puede producirse por:

expiración TTL;
cambio de security epoch;
cambio de policy version;
cambio de entity version;
invalidación;
cambio de membership;
revocación;
cambio de sesión;
cambio de MFA;
cambio de delegación;
cambio de contexto;
cache generation antigua.
10. TTL Profiles

Se define un sistema de perfiles en lugar de un único TTL global.

Esto permite que cada categoría de autorización tenga una política de frescura adecuada.

Perfil conceptual
Perfil	Freshness	Uso
TTL-F0	Authoritative	Sin dependencia de TTL
TTL-F1	Security validated	Sensible
TTL-F2-SHORT	Eventual corta	Normal
TTL-F2-MEDIUM	Eventual media	Bajo riesgo
TTL-F3	Restricted	Degradación
11. Valores iniciales propuestos

Como baseline operativo inicial:

F0
Security TTL:
0

Es decir:

No cached ALLOW sin validación autoritativa.
F1
TTL objetivo:
≤ 30 segundos

Siempre condicionado por:

epoch;
versión;
invalidación;
contexto;
criticidad.
F2-SHORT
TTL objetivo:
≤ 60 segundos

Para operaciones normales con sensibilidad moderada.

F2-MEDIUM
TTL objetivo:
≤ 5 minutos

Para operaciones de bajo riesgo y altamente repetitivas.

F3
No se concede un TTL adicional de seguridad.

La degradación no extiende artificialmente la autorización.

12. Maximum Authorization Staleness — MAS

Se define:

MAS = Maximum Authorization Staleness

como el máximo período durante el cual una decisión cacheada podría ser considerada utilizable bajo las reglas de su Freshness Class.

MAS no reemplaza:

security epoch;
invalidación;
policy version;
entity version;
contexto.

Por lo tanto:

MAS = upper bound

y no:

MAS = permission guarantee
13. MAS inicial

Baseline propuesto:

Clase	MAS objetivo
F0	0
F1	≤ 30 s
F2-SHORT	≤ 60 s
F2-MEDIUM	≤ 5 min
F3	No permite extensión de ALLOW

Estos valores deberán validarse posteriormente mediante:

performance testing;
security testing;
chaos testing;
invalidation latency testing;
multi-region testing;
producción controlada.
14. Security SLAs

Se definen SLAs específicos para propagación de cambios de seguridad.

No debe utilizarse solamente el SLA general de disponibilidad del servicio.

15. Revocation Propagation SLA

Las revocaciones tienen prioridad sobre las concesiones.

Se consideran revocaciones críticas:

user suspension;
user disable;
membership removal;
role removal;
permission removal;
session revocation;
refresh-token family revocation;
MFA revocation;
delegation revocation;
security policy tightening;
account compromise.

Objetivo inicial:

Security Revocation Propagation:
≤ 5 segundos

para propagación normal entre:

Authoritative State
        ↓
Outbox
        ↓
Event Platform
        ↓
Cache Consumer
        ↓
L1/L2
16. Critical Revocation Guarantee

Para operaciones F0:

No stale ALLOW

incluso si la propagación de invalidación supera el SLA.

Esto significa que:

Event delay
≠
Authorization bypass

Si la plataforma de eventos está retrasada y la operación requiere F0:

Fresh validation

debe prevalecer.

17. Grant Propagation SLA

Las nuevas concesiones de permisos pueden utilizar una consistencia ligeramente más flexible que las revocaciones.

Ejemplos:

nuevo role;
nuevo permission;
nueva membership;
nueva delegation;
nueva política permisiva.

Objetivo inicial:

Grant Propagation:
≤ 30 segundos

Una concesión puede tardar más que una revocación en llegar a todas las cachés sin representar necesariamente un riesgo de seguridad.

Por el contrario:

una revocación atrasada puede generar un acceso indebido.

18. Revocation vs Grant Priority

Se establece explícitamente:

REVOCATION > GRANT

Cuando existen estados concurrentes:

ALLOW + REVOKE

la revocación prevalece.

Cuando existe incertidumbre:

UNKNOWN + REVOKE possibility

no se debe producir un ALLOW para una operación crítica.

19. Policy Change SLA

Los cambios de políticas de seguridad deberán invalidar las entradas afectadas.

Objetivo:

Policy Security Invalidation:
≤ 5 segundos

especialmente para:

políticas restrictivas;
MFA obligatorio;
restricciones de tenant;
restricciones administrativas;
cambios de acceso privilegiado.
20. Membership Change SLA

Cambios en membership:

add
remove
suspend
restore
role assignment
role removal

deberán provocar invalidación correspondiente.

Para revocaciones:

≤ 5 segundos objetivo

Para nuevas concesiones:

≤ 30 segundos objetivo
21. Session Revocation SLA

La revocación de sesión debe ser tratada como evento de seguridad.

Objetivo:

≤ 5 segundos

para propagación normal.

Una sesión revocada no podrá recuperar acceso mediante:

cache antigua;
refresh token antiguo;
decisión de autorización antigua.
22. MFA Freshness SLA

Cambios críticos de MFA deben invalidar las decisiones afectadas.

Ejemplos:

factor revocado;
factor cambiado;
trusted device revocado;
MFA recovery;
cambio de assurance level.

Objetivo:

≤ 5 segundos

para propagación de la invalidación.

Las evidencias de MFA tendrán además su propia validez temporal y no podrán reutilizarse indefinidamente.

23. Context Freshness

La autorización no depende exclusivamente del estado almacenado.

Puede depender de:

IP
device
risk
location
session
MFA evidence
ownership
request attributes

Por ello, cada contexto deberá definir su propia política de frescura.

Ejemplo:

MFA evidence:
short-lived

Risk:
short-lived

Ownership:
version-based

Membership:
version + epoch

Policy:
policy version

Tenant:
tenant context + security epoch
24. Cache Eligibility Matrix
Operación	Clase	Cache ALLOW
Cambio de permisos	F0	No
Revocación de membership	F0	No
Revocación de sesión	F0	No
Cambio crítico de MFA	F0/F1	Validación fuerte
Administración de seguridad	F0	No
Operación privilegiada	F1	Condicional
Operación sensible	F1	Condicional
Lectura normal	F2	Sí
Dashboard	F2	Sí
Consulta de bajo riesgo	F2	Sí
Sistema degradado	F3	Restringido
25. DENY también necesita frescura

Un punto importante:

La política de frescura no se aplica únicamente a ALLOW.

Una decisión:

DENY

también puede quedar obsoleta.

Ejemplo:

10:00 DENY
10:01 usuario recibe nuevo permission
10:05 cache todavía contiene DENY

La autorización debería poder reevaluarse.

Por ello:

STALE DENY

debe poder invalidarse de forma equivalente a:

STALE ALLOW

aunque el riesgo sea diferente.

26. Security Epoch

El Security Epoch continúa siendo uno de los mecanismos de seguridad más importantes.

Ejemplo conceptual:

security_epoch = 42

Una autorización cacheada con:

epoch = 41

no puede considerarse válida aunque:

TTL > 0

Por tanto:

Epoch mismatch > TTL validity
27. Policy Version

Cada decisión asociada a una policy deberá registrar conceptualmente:

policy_id
policy_version

Una modificación incompatible deberá invalidar decisiones anteriores.

Ejemplo:

Policy v7 → cached ALLOW
Policy v8 → security restriction

Entonces:

cached v7 = STALE

aunque su TTL todavía no haya expirado.

28. Entity Version

Los recursos relevantes podrán utilizar versiones para evitar depender exclusivamente de TTL.

Ejemplo:

resource_version = 18

Si la caché contiene:

resource_version = 17

la decisión deberá considerarse potencialmente obsoleta.

Esto resulta especialmente importante para:

ownership;
membership;
delegations;
roles;
permisos;
recursos protegidos.
29. Context Fingerprint

Las decisiones que dependen de contexto deberán incluir un fingerprint conceptual.

Por ejemplo:

actor
tenant
device
session
risk
MFA assurance
resource
action

No deberán almacenarse atributos sensibles innecesarios dentro de la clave.

El fingerprint deberá ser:

determinista;
no reversible;
estable para el contexto relevante;
limitado a atributos necesarios.
30. Freshness Decision Algorithm

Conceptualmente:

EvaluateCacheEntry(entry):

    if tenant_mismatch:
        return INVALID

    if cache_generation_invalid:
        return INVALID

    if security_epoch_mismatch:
        return INVALID

    if policy_version_mismatch:
        return INVALID

    if entity_version_mismatch:
        return INVALID

    if context_mismatch:
        return INVALID

    if explicit_invalidation:
        return INVALID

    if TTL_expired:
        return STALE

    if MAS_exceeded:
        return STALE

    if operation_is_F0:
        return FRESH_VALIDATION_REQUIRED

    if operation_is_F1:
        return ACCEPTABLE_ONLY_IF_SECURITY_VALIDATED

    if operation_is_F2:
        return ACCEPTABLE

    if operation_is_F3:
        return RESTRICTED
31. Security SLA Hierarchy

Los SLAs deberán interpretarse de acuerdo con la siguiente prioridad:

1. Security Correctness
2. Revocation Safety
3. Tenant Isolation
4. Authorization Freshness
5. Availability
6. Performance Optimization

Nunca se deberá invertir el orden:

Performance > Security
32. Degradación

Cuando un componente falle:

Redis

Puede producir:

cache miss

y realizar evaluación alternativa.

No puede producir:

ALLOW all
Event Platform

El Outbox conserva el evento.

Mientras exista incertidumbre sobre invalidaciones críticas:

F0 → fresh validation
F1 → security validation
F2 → controlled degradation
Cache Consumer

Si existe backlog:

propagation lag increases

las entradas deberán perder confianza progresivamente.

No se deberá asumir:

eventual consistency forever
33. Staleness Budget

Cada Freshness Class tendrá un presupuesto máximo de staleness.

Conceptualmente:

Freshness Budget
    =
    TTL
    +
    propagation guarantees
    +
    context validity

pero nunca podrá superar:

MAS

ni invalidar un requisito F0.

34. Observabilidad

Se deberán medir como mínimo:

authorization_cache_hit_total
authorization_cache_miss_total
authorization_cache_fresh_total
authorization_cache_stale_total
authorization_cache_invalid_total
authorization_cache_f0_fresh_validation_total
authorization_cache_f1_validation_total
authorization_cache_f2_cached_allow_total
authorization_cache_invalidation_total
authorization_cache_invalidation_latency
authorization_cache_revocation_latency
authorization_cache_grant_latency
authorization_cache_policy_invalidation_latency
authorization_cache_event_lag
authorization_cache_version_mismatch
authorization_cache_epoch_mismatch
authorization_cache_context_mismatch
authorization_cache_generation_mismatch
authorization_cache_reconciliation_mismatch
35. Security Metrics

Deben existir métricas específicas para:

revocation propagation SLA violations
critical stale ALLOW attempts
stale cache usage
invalid cache usage
cross-tenant cache mismatch
authorization fallback
F3 activations
security epoch mismatches
policy version mismatches

Un intento de utilizar una decisión obsoleta para F0 deberá ser observable como evento de seguridad operacional.

36. Alertas

Deberán generarse alertas cuando:

revocation latency > SLA

o:

event propagation lag > threshold

o:

critical authorization fallback increases

o:

cache/version mismatch increases

o:

unexpected F3 frequency

o:

security epoch propagation is delayed
37. Performance Objectives

El sistema deberá preservar los objetivos generales del Blueprint.

El cache hit deberá permitir reducir la latencia de autorización sin degradar seguridad.

Objetivos conceptuales:

Authorization:
< 50 ms objetivo interno

y deberá contribuir al objetivo general de:

API P95 < 300 ms

La optimización de caché no deberá utilizarse para justificar decisiones inseguras.

38. Testing Requirements

Deberán existir pruebas para:

TTL
expiración exacta;
expiración concurrente;
clock skew;
renovación;
entradas antiguas.
Epoch
incremento;
invalidación inmediata;
entrada antigua;
eventos fuera de orden.
Policy
cambio de versión;
policy tightening;
policy relaxation;
policy retirement.
Membership
revocación;
restauración;
cambio de roles.
Session
revocación;
logout;
revoke-all.
MFA
factor revocado;
trusted device;
step-up expirado.
Delegation
creación;
expiración;
revocación.
Context
cambio de riesgo;
device;
sesión;
MFA assurance.
39. Chaos Testing

Se deberá comprobar el comportamiento frente a:

Redis outage
Event Platform outage
consumer pause
event duplication
event reordering
network partition
PostgreSQL failover
clock skew
cache corruption
cache loss
delayed invalidation
reconciliation failure
regional partition
cache stampede

Los resultados deben demostrar:

No unauthorized ALLOW
40. Multi-Region

En escenarios multi-región:

Security Epoch deberá propagarse de forma segura.
Las revocaciones deberán tener prioridad.
No se asumirá consistencia global instantánea.
Las operaciones F0 deberán poder requerir validación autoritativa.
Una región aislada no podrá generar autorizaciones privilegiadas basándose únicamente en información potencialmente obsoleta.

El comportamiento exacto multi-región queda para una decisión arquitectónica posterior.

41. AI Authorization

Los agentes de IA utilizarán exactamente las mismas Freshness Classes.

Un agente AI:

no puede
ampliar TTL;
ignorar security epoch;
saltarse invalidaciones;
convertir F3 en ALLOW;
ignorar policy version;
utilizar cache obsoleta para obtener privilegios.

La IA puede ayudar a:

detectar anomalías;
analizar freshness;
recomendar optimizaciones;
detectar propagation problems.

Pero:

AI no puede modificar los requisitos de seguridad de frescura.

42. Consecuencias positivas

Este ADR permite:

evitar dependencia exclusiva de TTL;
diferenciar operaciones críticas de normales;
controlar consistencia eventual;
establecer SLAs medibles;
priorizar revocaciones;
mejorar performance sin debilitar seguridad;
detectar stale authorization;
definir comportamiento bajo degradación;
facilitar capacity planning;
preparar multi-region;
proporcionar métricas operacionales claras.
43. Consecuencias negativas

Introduce mayor complejidad:

múltiples freshness classes;
perfiles TTL;
epochs;
entity versions;
policy versions;
context fingerprints;
métricas adicionales;
invalidación distribuida;
pruebas de consistencia;
reglas específicas por operación.

Esta complejidad se considera aceptable debido a que autorización es un componente de seguridad crítico.

44. Alternativas rechazadas
44.1 TTL único global

Rechazado.

No diferencia:

lectura normal

de:

revocación administrativa
44.2 TTL como única garantía

Rechazado.

Una revocación puede ocurrir antes de que expire TTL.

44.3 Consistencia fuerte para todo

Rechazado.

Produciría:

mayor latencia;
mayor carga;
menor escalabilidad;
peor eficiencia de cache.
44.4 Eventual consistency para todo

Rechazado.

No es aceptable para:

revocaciones;
tenant isolation;
privilegios;
MFA;
sesiones;
políticas críticas.
44.5 Extender TTL durante una caída

Rechazado.

Una falla de infraestructura no debe transformarse en una autorización prolongada.

45. Non-Negotiables
TTL no es la única garantía de frescura.
F0 no permite stale ALLOW.
Security Epoch tiene prioridad sobre TTL.
Policy Version tiene prioridad sobre TTL.
Entity Version tiene prioridad sobre TTL.
Revocations > Grants.
Tenant mismatch → DENY.
Unknown no puede producir ALLOW crítico.
F3 no extiende permisos.
PostgreSQL permanece como autoridad.
Redis no es fuente de verdad.
AI no puede reducir requisitos de frescura.
Los SLAs de revocación deben ser observables.
Las decisiones stale deben poder invalidarse.
La recuperación debe preservar las mismas garantías de seguridad.
No puede existir privilege escalation debido a cache.
No puede existir cross-tenant ALLOW debido a cache.
Una sesión revocada no puede recuperar autorización desde cache.
Una membership revocada no puede recuperar autorización desde cache.
La caché es una optimización, no una autoridad.
46. Traceability
Blueprint
Identity Domain
Authorization
RBAC
ABAC
Tenant Isolation
Security Policy Engine
Event-Driven Architecture
Performance
Observability
Security by Design
AI-Native Architecture
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
47. Pending Decisions

Los siguientes puntos quedan deliberadamente abiertos para ADRs posteriores:

Valores definitivos de TTL por operación.
MAS definitivo por recurso/acción.
Catálogo definitivo de operaciones F0/F1/F2/F3.
Revocation SLA definitivo por región.
Grant propagation SLA definitivo.
Policy propagation SLA.
MFA freshness TTL.
Risk-context freshness.
Clock-skew tolerance.
Multi-region freshness guarantees.
Cache refresh strategy.
Single-flight/stampede thresholds.
SLA breach escalation.
Automated security response.
Exact implementation of freshness profiles.

No se deben convertir estos valores pendientes en código definitivo sin su correspondiente decisión arquitectónica.

48. Acceptance Criteria
AC-01

Una entrada F0 no puede producir ALLOW basándose exclusivamente en TTL.

AC-02

Una entrada con security_epoch obsoleto debe considerarse inválida.

AC-03

Una entrada con policy_version obsoleta debe invalidarse cuando corresponda.

AC-04

Una entrada con entity_version obsoleta debe considerarse stale/invalid según criticidad.

AC-05

Una revocación debe tener prioridad sobre una concesión.

AC-06

El objetivo inicial de propagación de revocaciones críticas es ≤5 segundos.

AC-07

El objetivo inicial de propagación de nuevas concesiones es ≤30 segundos.

AC-08

F2 puede utilizar consistencia eventual controlada.

AC-09

F3 no puede extender un ALLOW más allá de los límites de seguridad.

AC-10

Un tenant mismatch produce DENY.

AC-11

Una sesión revocada no puede reutilizar una decisión cacheada.

AC-12

Una membership revocada no puede reutilizar una decisión cacheada.

AC-13

Un cambio crítico de MFA invalida las decisiones afectadas.

AC-14

Un cambio crítico de policy invalida las decisiones afectadas.

AC-15

Los stale DENY también pueden invalidarse.

AC-16

Las métricas permiten medir propagation latency.

AC-17

Las violaciones de Security SLA generan observabilidad y alertas.

AC-18

Una caída de Redis no provoca autorización global.

AC-19

Una caída del Event Platform no provoca automáticamente un ALLOW inseguro.

AC-20

La IA utiliza exactamente las mismas Freshness Classes y Security SLAs.

49. Resultado arquitectónico

Con este ADR, EVOXA pasa de un modelo genérico de:

Authorization Cache + TTL

a un modelo de:

             ┌──────────────────────┐
             │ Authorization Request│
             └──────────┬───────────┘
                        ↓
             ┌──────────────────────┐
             │ Operation Criticality│
             └──────────┬───────────┘
                        ↓
             ┌──────────────────────┐
             │ Freshness Class      │
             │ F0 / F1 / F2 / F3    │
             └──────────┬───────────┘
                        ↓
        ┌───────────────┼────────────────┐
        ↓               ↓                ↓
      Epoch           Policy          Entity
      Version         Version         Version
        └───────────────┼────────────────┘
                        ↓
               Context Freshness
                        ↓
                Invalidation State
                        ↓
                  Cache Generation
                        ↓
                 TTL / MAS Check
                        ↓
              ┌────────────────────┐
              │ Freshness Decision │
              └─────────┬──────────┘
                        ↓
          ┌─────────────┴─────────────┐
          ↓                           ↓
     Cache Eligible              Fresh Evaluation
          ↓                           ↓
      Authorization Runtime / PDP
                        ↓
             ALLOW / DENY / STEP-UP

La regla fundamental queda establecida:

La caché puede acelerar una decisión de autorización; nunca puede redefinir su validez de seguridad.

Siguiente ADR recomendado

El siguiente paso lógico es:

ADR-IDENTITY-058 — Authorization Cache TTL Profiles & Operation Criticality Matrix

Ahí podemos definir de manera mucho más concreta qué operaciones de EVOXA pertenecen a F0/F1/F2/F3, qué TTL/MAS tendría cada una y cómo se determina automáticamente la criticidad de una combinación:

Actor + Tenant + Resource + Action + Context + Risk

Eso nos dejaría prácticamente cerrada la política arquitectónica de freshness + cache + autorización antes de pasar a la siguiente capa de implementación.
