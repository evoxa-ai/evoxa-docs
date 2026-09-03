ADR-IDENTITY-042 — Identity Source of Truth, Attribute Ownership & Synchronization Conflict Resolution Strategy

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity
Priority: Critical
Type: Architecture / Security / Data Consistency

1. Context

ADR-IDENTITY-040 estableció que EVOXA soportará Federated Authentication with Local Identity and Local Authorization.

ADR-IDENTITY-041 definió el modelo de provisioning y deprovisioning, incluyendo:

Local Provisioning.
Invitation-Based Provisioning.
Just-In-Time Provisioning.
Pre-Provisioning.
Futura integración SCIM.
Deprovisioning.
Synchronization.
ExternalIdentity.
Lifecycle synchronization.

Sin embargo, todavía falta resolver una cuestión fundamental:

¿Qué sistema es la fuente de verdad para cada dato de identidad y qué ocurre cuando dos fuentes intentan modificar el mismo atributo?

En EVOXA pueden coexistir:

EVOXA
OIDC Provider
SAML Provider
SCIM Provider
Administración interna
Directory / Enterprise Identity

Si no se define explícitamente la autoridad sobre cada atributo, pueden producirse conflictos como:

SCIM → nombre = "Sebastian Silva"
Admin → nombre = "Sebastián Silva"
OIDC → nombre = "Sebastián A. Silva"

o:

SCIM → ACTIVE
Admin → SUSPENDED
External IdP → ACTIVE

El sistema necesita reglas deterministas para resolver estos conflictos sin comprometer seguridad, tenant isolation ni trazabilidad.

2. Problem Statement

Se necesita definir:

Fuentes de verdad.
Ownership de atributos.
Atributos administrados externamente.
Atributos administrados por EVOXA.
Atributos compartidos.
Prioridad entre fuentes.
Resolución de conflictos.
Sincronización.
Versionado.
Concurrencia.
Protección contra stale data.
Protección contra cambios maliciosos.
Auditoría.
Comportamiento durante fallos de sincronización.

La estrategia debe impedir que una fuente externa pueda modificar arbitrariamente información que pertenece al dominio de EVOXA.

3. Decision

EVOXA adoptará un modelo de:

Attribute-Level Source of Truth

La autoridad no será definida exclusivamente a nivel de usuario o proveedor.

Será definida por atributo y por contexto.

Esto significa que un mismo User puede tener atributos cuyo origen autorizado sea diferente.

Ejemplo:

User
│
├── email              → EVOXA / Verification
├── external_subject   → External IdP
├── nombre             → SCIM
├── apellidos          → SCIM
├── role               → EVOXA
├── permissions        → EVOXA
├── membership         → EVOXA
├── security_state     → EVOXA
└── MFA                → EVOXA

La fuente externa puede ser authoritative para determinados atributos, pero nunca adquiere autoridad global sobre el User.

4. Source of Truth Hierarchy

Se establece una jerarquía conceptual:

Platform Security Rules
        ↓
EVOXA Security Policy
        ↓
EVOXA Domain Authority
        ↓
Organization Policy
        ↓
Authorized External Source
        ↓
User-Supplied Attribute

Esta jerarquía no significa que EVOXA sobrescriba automáticamente todos los atributos externos.

Significa que una fuente inferior no puede contradecir una restricción superior.

Por ejemplo:

SCIM → role = ADMIN

no puede convertir automáticamente al usuario en ADMIN si EVOXA no permite ese mapping.

5. Global vs Attribute-Level Authority

No se adoptará un único:

User Source = SCIM

como mecanismo universal.

Se utilizará:

Attribute Authority

porque diferentes aspectos de Identity requieren diferentes niveles de control.

Ejemplo:

Atributo	Fuente autorizada
External Subject	IdP
Federation Provider	EVOXA Federation
User Identity State	EVOXA
Organization	EVOXA
Membership	EVOXA / Provisioning Policy
Role	EVOXA
Permission	EVOXA
MFA State	EVOXA
Security Policy	EVOXA
Nombre	Policy-defined
Apellidos	Policy-defined
Email	Policy-defined + verification
Avatar	Policy-defined
Metadata externa	External Provider

La tabla anterior es conceptual; el ownership definitivo por atributo debe quedar parametrizable por política donde corresponda.

6. Attribute Ownership Categories

Se definirán tres categorías principales.

6.1 EVOXA-Owned

Atributos cuya autoridad pertenece exclusivamente a EVOXA.

Ejemplos:

User lifecycle.
Organization membership.
Roles.
Permissions.
Security policies.
Sessions.
Refresh token state.
MFA state.
Security state.
Audit metadata.

Una fuente externa no puede sobrescribirlos directamente.

7. External-Owned

Atributos cuyo valor es administrado por un sistema externo autorizado.

Ejemplos potenciales:

External Subject.
External Directory Identifier.
Enterprise username.
determinados atributos corporativos.

Si el atributo es external-owned:

External Source
       ↓
Validated Synchronization
       ↓
Local Projection

La modificación local directa deberá estar prohibida o limitada según la política.

8. Shared / Policy-Owned

Algunos atributos pueden tener participación de múltiples fuentes.

Ejemplo:

email
nombre
apellidos
telefono

En estos casos no se utilizará un simple:

last write wins

por defecto.

La política deberá definir:

quién puede escribir;
quién puede validar;
quién puede sobrescribir;
qué evidencia se requiere;
cuándo vence la autoridad;
cómo se resuelven conflictos.
9. Identity Provider vs Provisioning Provider

Authentication Provider y Provisioning Provider pueden ser diferentes.

Por ejemplo:

OIDC IdP
    ↓
Authentication

SCIM
    ↓
Provisioning

Esto significa que no se debe asumir:

Authentication Source = Attribute Source

Un usuario puede autenticarse mediante OIDC mientras sus atributos corporativos son sincronizados mediante SCIM.

10. ExternalIdentity Authority

ExternalIdentity será la relación técnica que vincula:

Provider
+
External Subject
+
EVOXA User

La identidad externa deberá conservar su propio identificador estable.

Conceptualmente:

ExternalIdentity
├── provider_id
├── external_subject
├── user_id
├── status
├── created_at
└── updated_at

El email externo no sustituirá al external_subject como identificador de Federation.

11. No Automatic Account Linking

No se permitirá:

Same Email
     ↓
Automatic Account Linking

como regla universal.

El matching por email puede ser utilizado como señal de discovery, pero el linking de identidades deberá respetar los requisitos de seguridad definidos por la política.

Para escenarios de alto riesgo podrán requerirse:

autenticación adicional;
verificación del usuario;
evidencia de control;
aprobación administrativa;
MFA/step-up.
12. Organization Authority

Organization y Membership serán siempre responsabilidad de EVOXA.

Un proveedor externo puede proporcionar una señal:

User belongs to Group X

pero EVOXA decide:

Group X
   ↓
Mapping Policy
   ↓
Membership / Role

Por lo tanto:

External Group ≠ EVOXA Membership automáticamente.

13. Role Authority

Los roles serán EVOXA-owned.

Un proveedor externo podrá aportar grupos o atributos, pero solamente podrán influir en roles mediante un mapping explícito.

Ejemplo:

External Group:
"coaches"

        ↓

Organization Mapping Policy

        ↓

EVOXA Role:
Coach

La política puede además imponer:

External Group
+
Organization
+
Provider
+
Security Policy

antes de permitir el mapping.

14. Permission Authority

Las permissions serán exclusivamente derivadas desde el modelo de autorización de EVOXA.

No se aceptará:

External Claim
      ↓
Permission

directamente.

Debe existir:

External Claim
      ↓
Validated Mapping
      ↓
EVOXA Role
      ↓
EVOXA Permissions

Esto mantiene la separación entre Federation y Authorization.

15. Lifecycle Authority

El estado de seguridad local será propiedad de EVOXA.

Ejemplos:

ACTIVE
SUSPENDED
DISABLED

Un proveedor externo puede generar una señal que provoque una transición, pero la transición se realizará mediante el Identity Lifecycle Service.

Por lo tanto:

SCIM SUSPEND
     ↓
Lifecycle Service
     ↓
Policy
     ↓
User/Membership SUSPENDED

y nunca:

SCIM
 ↓
UPDATE users.status

directamente.

16. Conflict Resolution

Cuando dos fuentes intenten modificar el mismo atributo se aplicará una estrategia explícita.

Se consideran cuatro mecanismos:

16.1 Authority-Based Resolution

La fuente autorizada gana.

Authorized Source
       ↓
Accepted
16.2 Policy-Based Resolution

La política determina qué fuente tiene autoridad en ese contexto.

Source A
Source B
   ↓
Policy Engine
   ↓
Decision
16.3 Version-Based Resolution

Cuando sea posible, se utilizarán:

version;
sequence;
timestamp confiable;
ETag;
optimistic concurrency.

Esto permite detectar información stale.

16.4 Manual Resolution

Para conflictos de alto riesgo o ambiguos:

Conflict
   ↓
HOLD
   ↓
Review
   ↓
Authorized Decision

No se utilizará automáticamente last write wins para atributos críticos.

17. Last Write Wins

Last Write Wins no será el mecanismo universal.

Puede ser aceptable para determinados atributos no críticos cuando:

la fuente es confiable;
el atributo tiene ownership claro;
el cambio es reversible;
no afecta autorización;
no afecta seguridad.

Nunca será la estrategia predeterminada para:

roles;
permissions;
security state;
MFA;
membership;
tenant;
account recovery state.
18. Stale Data Protection

Una actualización externa puede llegar tarde.

Ejemplo:

T1:
SCIM → ACTIVE

T2:
SCIM → SUSPENDED

T3:
mensaje antiguo → ACTIVE

El sistema debe evitar que el evento antiguo reactive al usuario.

Se utilizarán, según el mecanismo disponible:

version;
event sequence;
source timestamp;
operation ID;
provider metadata;
current-state validation;
optimistic concurrency.

Una fuente que no proporcione suficiente información para resolver stale updates podrá requerir una estrategia adicional antes de habilitar sincronización automática.

19. Concurrent Updates

Ejemplo:

Admin → SUSPEND
SCIM  → ACTIVE

simultáneamente.

La operación no deberá depender simplemente del orden en que lleguen las peticiones.

El dominio deberá validar:

current_version
source_authority
policy
operation_context

antes de aplicar el cambio.

Un conflicto puede producir:

SYNC_CONFLICT

y requerir resolución.

20. Security State Precedence

Para atributos relacionados con seguridad, se aplicará una regla más estricta.

Una señal de seguridad local crítica no podrá ser sobrescrita automáticamente por una fuente externa de menor autoridad.

Ejemplo:

EVOXA Security Policy
        ↓
User SUSPENDED

y posteriormente:

SCIM ACTIVE

No significa automáticamente:

User ACTIVE

La reactivación deberá pasar por la política correspondiente.

21. Deprovisioning Precedence

El deprovisioning confirmado debe tener especial protección contra reactivación accidental.

Ejemplo:

SCIM DELETE
     ↓
Membership REMOVED

Posteriormente:

OIDC LOGIN

no debe recrear automáticamente el acceso si la policy requiere reprovisioning explícito.

Esto evita:

Access resurrection

22. Attribute Mapping

Cada integración podrá definir mappings explícitos.

Conceptualmente:

External Attribute
        ↓
Normalization
        ↓
Validation
        ↓
Mapping
        ↓
EVOXA Attribute

Ejemplo:

given_name → nombre
family_name → apellidos
email → email
department → metadata.department
groups → role mapping

El mapping debe ser:

explícito;
validado;
versionado;
auditable;
tenant-aware.
23. Attribute Validation

Antes de aceptar un atributo externo:

External Value
      ↓
Schema Validation
      ↓
Domain Validation
      ↓
Security Policy
      ↓
Ownership Check
      ↓
Apply

Se deberán rechazar:

tipos inválidos;
valores fuera de rango;
valores incompatibles con el dominio;
modificaciones de atributos no autorizados.
24. Normalization

Los atributos compartidos deberán utilizar reglas de normalización.

Ejemplos:

email;
identificadores externos;
teléfonos;
nombres de usuario.

La normalización debe evitar que:

User@Example.com
user@example.com

sean interpretados accidentalmente como identidades completamente distintas cuando el dominio considere equivalentes dichos valores.

Las reglas concretas quedan pendientes de la ESP correspondiente.

25. Attribute Provenance

EVOXA deberá poder determinar el origen de un valor relevante.

Conceptualmente:

Attribute Value
     +
Source
     +
Source Identifier
     +
Last Synchronized At
     +
Version

Esto permitirá responder:

¿De dónde proviene este dato?

Ejemplo:

email:
value = user@example.com
source = SCIM
provider = Enterprise Directory
synced_at = ...
version = ...

La persistencia exacta de provenance queda pendiente.

26. Synchronization Metadata

Para integraciones externas podrán utilizarse metadatos como:

source
provider_id
external_subject
external_version
source_updated_at
last_synced_at
sync_status

Estos datos no deben sustituir el estado de dominio.

Son información de sincronización.

27. Synchronization Status

Una sincronización podrá encontrarse conceptualmente en:

PENDING
PROCESSING
APPLIED
SKIPPED
CONFLICT
FAILED

Un FAILED de sincronización no implica necesariamente que el User deba suspenderse.

El tratamiento dependerá del tipo de cambio y de la policy.

28. Provider Outage

Un proveedor externo puede estar temporalmente indisponible.

No se inferirá:

Provider Unavailable
        =
Users Invalid

Durante un outage:

los datos existentes permanecen;
las políticas locales continúan;
las operaciones que requieran validación externa pueden fallar o degradarse de manera controlada;
no se realizará disablement masivo sin evidencia válida.
29. Emergency Security Signals

La estrategia anterior no impide una revocación urgente.

Si existe una señal de seguridad autenticada y confiable:

Provider
    ↓
Compromise Signal
    ↓
Validated
    ↓
Security Policy
    ↓
Revoke / Suspend

esta podrá tener prioridad sobre la sincronización normal.

La severidad y respuesta dependerán de la Security Policy.

30. Synchronization Direction

No todos los atributos deberán sincronizarse bidireccionalmente.

Se distinguen:

One-Way
External → EVOXA
EVOXA → External
EVOXA → External
Bidirectional
External ↔ EVOXA

La sincronización bidireccional requiere especial cuidado por posibles loops:

A → B
B → A
A → B
...

Por defecto, EVOXA preferirá sincronización unidireccional cuando no exista un requisito explícito para bidireccionalidad.

31. Synchronization Loop Prevention

Cuando EVOXA actualice un atributo recibido desde una fuente externa, no deberá generar automáticamente una actualización que vuelva al mismo proveedor sin distinguir el origen.

Podrán utilizarse:

source metadata;
correlation ID;
causation ID;
operation ID;
synchronization version.

Esto será especialmente importante en futuras integraciones bidireccionales.

32. Event-Driven Synchronization

La sincronización seguirá el modelo Event-Driven definido por EVOXA.

Conceptualmente:

External Provider
        ↓
Integration Adapter
        ↓
Lifecycle / Identity Application Service
        ↓
PostgreSQL Transaction
        +
Outbox Event
        ↓
Event Platform
        ↓
Consumers

Los eventos internos seguirán siendo hechos del dominio, no simples copias de mensajes externos.

33. Audit Requirements

Cada conflicto significativo deberá ser auditable.

Registrar conceptualmente:

conflict_id
tenant_id
user_id
attribute
source_a
source_b
value_a_metadata
value_b_metadata
authority
decision
policy
actor
timestamp
correlation_id

No se deberán almacenar secretos ni información sensible innecesaria.

34. Security Events

Los siguientes casos podrán generar Security Events:

intento de modificar atributo no autorizado;
conflicto de identidad;
account linking sospechoso;
provider inconsistency;
stale update crítico;
reactivación rechazada;
external identity collision;
cross-tenant mapping;
role mapping inválido;
provisioning masivo anómalo.
35. Tenant Isolation

Toda resolución deberá ejecutarse dentro del contexto de tenant correspondiente.

Nunca se permitirá:

Provider A / Organization A
        ↓
User / Organization B

por una coincidencia accidental de:

email;
nombre;
external subject;
username.

Los mappings de Federation y Provisioning deben estar explícitamente asociados al tenant cuando corresponda.

36. AI Agent Integration

Los AI Agents podrán analizar conflictos y sugerir:

SOURCE_A_WINS
SOURCE_B_WINS
MANUAL_REVIEW

pero no tendrán autoridad implícita para cambiar ownership.

Ejemplo:

AI detects conflict
       ↓
Recommendation
       ↓
Policy / Authorized Operator
       ↓
Decision

Para operaciones críticas, la decisión final debe provenir del mecanismo autorizado de EVOXA.

37. Persistence Model — Conceptual

Podrán requerirse estructuras adicionales relacionadas con:

IdentityProvider
ExternalIdentity
AttributeSource
AttributeMapping
SynchronizationState
ProvisioningOperation
IdentityConflict

Un modelo conceptual de AttributeOwnership podría incluir:

attribute
scope
source_type
provider_id
authority_level
writable
validation_policy
mapping_version

No se fija todavía este modelo como schema definitivo.

38. Performance

La resolución de ownership y policy no debe convertir cada autenticación en una consulta pesada a todos los proveedores externos.

Siempre que sea seguro:

mappings deben estar cacheados;
policies deben tener versionado;
JWKS debe utilizar caching;
datos externos previamente sincronizados pueden utilizarse como proyección;
las decisiones críticas deben seguir las reglas de fail-closed.

La autenticación normal no deberá depender de una sincronización completa en tiempo real si el protocolo y el modelo no lo requieren.

39. Disaster Recovery

El estado de ownership y synchronization debe formar parte de la recuperación.

Después de un restore se debe conservar:

ExternalIdentity mappings;
Membership state;
User lifecycle;
attribute provenance cuando exista;
synchronization state;
conflict state;
revocations.

No debe producirse una reactivación accidental por pérdida del estado de sincronización.

40. Consequences
Positivas
Define claramente la autoridad de cada dato.
Evita que Federation controle Authorization.
Reduce conflictos entre SCIM, OIDC, SAML y administración local.
Protege roles y permissions.
Permite múltiples proveedores.
Facilita Enterprise Identity.
Mejora trazabilidad.
Evita account resurrection.
Facilita futuras sincronizaciones bidireccionales controladas.
Negativas
Mayor complejidad conceptual.
Necesidad de metadata de provenance.
Necesidad de resolver conflictos.
Mayor cantidad de policies.
Requiere testing de concurrencia y sincronización.
SCIM no puede implementarse simplemente como CRUD directo.
41. Alternatives Considered
A. Una única fuente de verdad para todo User

Rejected.

No representa correctamente la separación entre Identity Provider, Provisioning y EVOXA Authorization.

B. Last Write Wins

Rejected as default.

Es inseguro para lifecycle, membership, roles y security state.

C. External Provider como autoridad total

Rejected.

Contradice el modelo de Local Authorization establecido en Federation.

D. EVOXA sobrescribe siempre todos los atributos externos

Rejected.

Eliminaría la utilidad del provisioning empresarial.

E. Sincronización bidireccional universal

Rejected.

Incrementa significativamente el riesgo de loops, conflictos y estados inconsistentes.

42. Non-Negotiables

No se permitirá:

Una fuente externa con autoridad global sobre Identity.
External claims → permissions directamente.
External groups → permissions directamente.
Email como único mecanismo universal de identity linking.
External provider sobrescribiendo Security State sin policy.
Reactivación automática después de deprovisioning crítico.
Last Write Wins para atributos de seguridad.
Cross-tenant synchronization.
Direct DB synchronization bypassing domain services.
Sincronización sin idempotencia.
Eventos externos que produzcan efectos duplicados.
Secrets dentro de synchronization metadata.
AI Agent modificando ownership sin autorización.
43. Traceability
Blueprint

Este ADR se relaciona principalmente con:

Identity.
Authentication.
Authorization.
Organization/Tenant Management.
Membership Management.
Federation.
Provisioning.
Security Policy.
Audit.
Event-Driven Architecture.
Tenant Isolation.
Previous ADRs
ADR-IDENTITY-040 — Identity Federation & External Identity Provider Strategy
ADR-IDENTITY-041 — Identity Provisioning, Deprovisioning & Lifecycle Synchronization Strategy
Related
ADR-IDENTITY-005 — Authorization Model
ADR-IDENTITY-007 — Domain Events & Outbox
ADR-IDENTITY-012 — PostgreSQL Schema
ADR-IDENTITY-017 — Runtime & Deployment
ADR-IDENTITY-021 — Controlled Event Replay
ADR-IDENTITY-022 — Consumer Idempotency
ADR-IDENTITY-028 — Observability & Operational Security
ADR-IDENTITY-029 — Disaster Recovery & Backup
ADR-IDENTITY-033 — Account Lifecycle
ADR-IDENTITY-039 — OAuth2.1 & OpenID Connect
ADR-IDENTITY-040 — Federation
ADR-IDENTITY-041 — Provisioning & Deprovisioning
44. Pending Decisions

Antes de pasar a implementación deberán definirse:

Catálogo definitivo de atributos.
Ownership exacto por atributo.
Source precedence.
Policy precedence.
Attribute provenance persistence.
SCIM attribute mappings.
OIDC claim mappings.
SAML attribute mappings.
Group-to-role mapping.
Conflict state machine.
Versioning strategy.
Stale-event window.
Manual conflict resolution workflow.
Bidirectional synchronization rules.
Loop prevention mechanism.
Provider outage behavior.
Automatic reactivation rules.
Deprovisioning precedence.
Attribute normalization.
Account-linking assurance levels.
Bulk synchronization limits.
Conflict retention.
Synchronization metrics and SLOs.
Exact API contracts.
Exact PostgreSQL schema.
45. Acceptance Criteria

El ADR podrá considerarse aceptado cuando:

 Exista ownership explícito por atributo.
 EVOXA sea autoridad local de Authorization.
 Roles y Permissions sean EVOXA-owned.
 Membership sea EVOXA-controlled.
 ExternalIdentity utilice provider + external_subject.
 Email no sea la única base de account linking.
 Group-to-role mapping sea explícito.
 Exista estrategia de conflict resolution.
 Last Write Wins no se utilice para atributos críticos.
 Exista protección contra stale updates.
 Exista control de concurrencia.
 Exista provenance para atributos relevantes.
 Exista idempotencia.
 Se eviten synchronization loops.
 Provider outage no provoque disablement masivo.
 Deprovisioning no pueda ser revertido accidentalmente.
 Cross-tenant synchronization sea imposible por diseño.
 Los conflictos sean auditables.
 Los cambios críticos generen Security Events.
 AI Agents puedan asistir pero no saltarse la autoridad de Identity.
 El modelo sea compatible con OIDC, SAML y futura integración SCIM.
46. Architectural Outcome

Con los ADR-040, 041 y 042, la arquitectura de Federation/Provisioning queda conceptualmente así:

                 ┌──────────────────────┐
                 │ External Identity    │
                 │ Providers            │
                 │ OIDC / SAML          │
                 └──────────┬───────────┘
                            │
                     Authentication
                            │
                            ▼
                  ┌──────────────────┐
                  │ ExternalIdentity │
                  └────────┬─────────┘
                           │
                           ▼
┌──────────────┐    ┌─────────────────────┐
│ SCIM /       │───▶│ Identity Lifecycle  │
│ Provisioning │    │ & Synchronization   │
└──────────────┘    └──────────┬──────────┘
                               │
                               ▼
                     ┌──────────────────┐
                     │ Source /         │
                     │ Attribute        │
                     │ Ownership        │
                     └────────┬─────────┘
                              │
                              ▼
                     ┌──────────────────┐
                     │ Conflict         │
                     │ Resolution       │
                     └────────┬─────────┘
                              │
                              ▼
                       ┌──────────────┐
                       │ EVOXA       │
                       │ Identity    │
                       └──────┬───────┘
                              │
                 ┌────────────┼─────────────┐
                 ▼            ▼             ▼
             Membership     Roles       Security
                 │            │             │
                 └────────────┼─────────────┘
                              ▼
                     Authorization Engine

La separación clave queda establecida:

Los proveedores externos pueden aportar identidad, atributos y señales de lifecycle; EVOXA conserva la autoridad sobre acceso, tenant, membership, roles, permissions y seguridad local.

Siguiente ADR recomendado

El siguiente paso lógico es bajar esta arquitectura a la relación entre Identity, Organization y Membership, especialmente para escenarios multi-organización:

ADR-IDENTITY-043 — Multi-Organization Identity, Membership & Cross-Tenant Access Strategy

Ahí podremos definir cómo un mismo User puede pertenecer a múltiples Organizations, cómo cambia el contexto de tenant durante una sesión y cómo se impide cualquier acceso accidental entre organizaciones.
