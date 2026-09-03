ADR-IDENTITY-041 — Identity Provisioning, Deprovisioning & Lifecycle Synchronization Strategy

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity
Priority: Critical
Type: Architecture / Security / Lifecycle

1. Context

EVOXA debe soportar múltiples mecanismos mediante los cuales una identidad puede ser creada, activada, modificada, suspendida o eliminada.

La identidad puede originarse desde:

Registro local.
Invitación a una Organization.
Administración interna.
Identity Federation mediante OIDC/SAML.
Futuros mecanismos de provisioning empresarial, como SCIM.
Service Accounts y Workload Identities para integraciones máquina-a-máquina.

El problema arquitectónico es evitar que cada mecanismo implemente su propia lógica de ciclo de vida.

La existencia de un usuario en un proveedor externo no debe implicar automáticamente que ese usuario tenga acceso activo a EVOXA.

EVOXA mantiene la autoridad local sobre:

User.
Organization.
Membership.
Roles.
Permisos.
Estado de acceso.
Sesiones.
Credenciales y mecanismos de recuperación.
Políticas de seguridad.

Esto es especialmente importante para Federation, donde un Identity Provider externo autentica al usuario, mientras que EVOXA mantiene la autorización y el estado de acceso local.

2. Problem Statement

Se necesita definir una estrategia consistente para:

Provisionar identidades.
Activarlas.
Asociarlas a Organizations.
Sincronizar cambios provenientes de sistemas externos.
Suspender o desprovisionar accesos.
Evitar accesos residuales.
Mantener consistencia entre User, Organization y Membership.
Preservar auditoría y trazabilidad.
Evitar que una sincronización externa pueda saltarse las políticas internas de seguridad.

Sin una estrategia centralizada podrían producirse situaciones como:

Usuario externo eliminado pero Membership local permanece activa.
Usuario suspendido externamente continúa utilizando sesiones existentes.
Cambio de atributos externos sobrescribe información administrada localmente.
Un grupo externo obtiene permisos sin una política explícita.
Un usuario federado obtiene acceso a otra Organization.
La eliminación de un usuario destruye información histórica necesaria para auditoría.
Un fallo temporal del IdP provoca desactivaciones masivas incorrectas.
3. Decision

EVOXA adoptará un modelo de Provisioning y Lifecycle Synchronization centralizado, donde los sistemas externos pueden proporcionar señales de identidad y lifecycle, pero EVOXA conserva la autoridad final sobre el acceso local.

La estrategia se basa en cinco principios:

Identity Authority Separation
Explicit Provisioning
Local Lifecycle Authority
Controlled Synchronization
Fail-Safe Deprovisioning
4. Authority Model

Se establece la siguiente separación:

Área	Autoridad
Autenticación externa	Identity Provider
Identidad externa	Identity Provider
Identidad local	EVOXA
Organization	EVOXA
Membership	EVOXA
Roles	EVOXA
Permissions	EVOXA
Security Policies	EVOXA
Sessions	EVOXA
Refresh Tokens	EVOXA
MFA local	EVOXA
Access decision	EVOXA
Audit	EVOXA

Por lo tanto:

External Authentication ≠ EVOXA Authorization

Un IdP externo puede afirmar:

“Este usuario fue autenticado.”

Pero EVOXA debe decidir:

“¿Este usuario tiene actualmente autorización para acceder a esta Organization y realizar esta operación?”

5. Provisioning Model

El provisioning se realizará mediante mecanismos explícitos.

Los modelos principales serán:

5.1 Local Provisioning

Un usuario puede ser creado directamente dentro de EVOXA.

Flujo conceptual:

Registration / Admin
        ↓
User CREATED
        ↓
Email Verification / Security Checks
        ↓
User ACTIVE
        ↓
Organization Membership
        ↓
Roles
        ↓
Access
5.2 Invitation-Based Provisioning

Una Organization puede invitar a un usuario.

Organization Admin
        ↓
Invitation
        ↓
Invitation PENDING
        ↓
User accepts
        ↓
Identity established
        ↓
Membership ACTIVE
        ↓
Roles assigned

La invitación deberá estar:

ligada a una Organization;
ligada a un propósito;
protegida contra replay;
expirable;
revocable;
auditada.

La aceptación de una invitación no debe otorgar permisos arbitrarios. Los roles disponibles estarán sujetos a las políticas de la Organization y del Platform Security Model.

6. Federated Provisioning

Para Federation, EVOXA soportará conceptualmente dos modelos.

6.1 Just-In-Time Provisioning

El usuario se provisiona cuando realiza su primer login federado.

External IdP
     ↓
Authentication
     ↓
ExternalIdentity
     ↓
Mapping Policy
     ↓
User
     ↓
Membership
     ↓
Roles
     ↓
EVOXA Session

JIT provisioning solamente estará permitido cuando una política de Federation de la Organization lo permita.

No debe existir:

Successful External Login
        =
Automatic EVOXA Access
7. Pre-Provisioning

Para organizaciones Enterprise, EVOXA podrá soportar provisioning previo.

Ejemplo:

Enterprise Directory
        ↓
Provisioning Integration
        ↓
EVOXA User
        ↓
Membership
        ↓
Role Mapping

Esto permite que una Organization prepare usuarios antes de su primer login.

El provisioning previo no debe activar automáticamente acceso si faltan condiciones de seguridad requeridas.

8. SCIM Strategy

SCIM podrá utilizarse como mecanismo futuro para:

Crear usuarios.
Actualizar usuarios.
Suspender usuarios.
Reactivar usuarios.
Eliminar/desprovisionar usuarios.
Sincronizar memberships cuando el modelo lo soporte.

SCIM será considerado un canal de provisioning, no una autoridad independiente de autorización.

El flujo conceptual será:

SCIM Provider
      ↓
Provisioning Adapter
      ↓
Validation
      ↓
Lifecycle Service
      ↓
User / Membership
      ↓
Domain Events
      ↓
Audit / Security / Consumers

La implementación concreta de SCIM queda pendiente.

9. User vs Membership Lifecycle

Una decisión fundamental es separar:

User

Representa la identidad global/local de la persona.

Membership

Representa su relación con una Organization.

Por lo tanto:

User ACTIVE

no significa necesariamente:

Membership ACTIVE

Ejemplo:

User
 ├── Organization A → ACTIVE
 ├── Organization B → SUSPENDED
 └── Organization C → REMOVED

El usuario puede seguir siendo una identidad válida mientras una determinada relación organizacional esté suspendida o eliminada.

10. Provisioning State Model

El provisioning deberá distinguir al menos:

REQUESTED
    ↓
PENDING
    ↓
PROVISIONED
    ↓
ACTIVE

Con estados de error o cancelación conceptuales:

FAILED
REJECTED
CANCELLED

No necesariamente todos estos estados deberán convertirse en estados persistentes del dominio; algunos pueden pertenecer al proceso de provisioning.

La decisión final de modelado queda pendiente.

11. Deprovisioning

El deprovisioning significa retirar el acceso de una identidad o relación.

Puede originarse por:

Usuario eliminado externamente.
Usuario suspendido externamente.
Membership eliminada.
Organization suspendida.
Administración interna.
Compromiso de seguridad.
Violación de política.
Fin de contrato.
Expiración.
Revocación administrativa.

El flujo conceptual será:

External/Internal Signal
        ↓
Validate Signal
        ↓
Lifecycle Policy
        ↓
User/Membership State Change
        ↓
Session / Token Evaluation
        ↓
Security Events
        ↓
Audit
        ↓
Domain Events
12. Deprovisioning Security Rule

Cuando una Membership deja de estar autorizada:

Membership != ACTIVE

el usuario no debe continuar obteniendo nuevas sesiones o refresh tokens para esa Organization.

Las sesiones y refresh-token families asociadas deberán ser revocadas de acuerdo con la política aplicable.

Especialmente:

Membership Removed
        ↓
Refresh capability revoked
        ↓
Future access denied

Un Access Token JWT que ya fue emitido puede continuar siendo criptográficamente válido hasta su expiración, pero JWT válido no equivale a autorización actual.

Por eso la autorización debe considerar el estado actual de:

User.
Membership.
Organization.
Security Policy.
13. Soft Deprovisioning vs Hard Deletion

EVOXA preferirá deprovisioning lógico frente a eliminación física inmediata.

Ejemplo:

ACTIVE
  ↓
SUSPENDED
  ↓
DISABLED / REMOVED

La eliminación física deberá ser excepcional.

Esto permite preservar:

Audit history.
Security events.
Referencias históricas.
Integridad de relaciones.
Evidencia operacional.

La retención concreta continuará gobernada por ADR-IDENTITY-013.

14. External Lifecycle Synchronization

Cuando exista un sistema externo, EVOXA podrá recibir señales como:

USER_CREATED
USER_UPDATED
USER_SUSPENDED
USER_REACTIVATED
USER_DELETED
GROUP_CHANGED
MEMBERSHIP_CHANGED

Estas señales deberán pasar por un proceso de validación antes de modificar el dominio.

No se debe implementar:

External Event
      ↓
Direct Database UPDATE

Debe existir:

External Event
      ↓
Adapter
      ↓
Validation
      ↓
Mapping
      ↓
Lifecycle Policy
      ↓
Domain Operation
      ↓
Transaction
      ↓
Outbox
15. Idempotency

Los mensajes de provisioning/deprovisioning deben soportar procesamiento At-Least-Once.

Por lo tanto, un mismo evento puede llegar varias veces.

Ejemplo:

USER_SUSPENDED
USER_SUSPENDED
USER_SUSPENDED

El resultado debe ser equivalente a procesarlo una vez.

Debe existir deduplicación/idempotencia basada en identificadores del evento o de la operación.

Esto se alinea con:

ADR-IDENTITY-007
ADR-IDENTITY-022
ADR-IDENTITY-026
16. Ordering

No se asumirá orden global de eventos.

Sin embargo, determinados cambios requieren orden lógico.

Ejemplo:

CREATE
  ↓
SUSPEND

no debe terminar procesándose como:

SUSPEND
  ↓
CREATE

Los consumidores deberán utilizar mecanismos como:

versionado;
timestamps confiables;
sequence/version;
causation;
correlation;
estado actual;
optimistic concurrency.

La estrategia exacta de ordering queda pendiente de implementación.

17. Attribute Synchronization

No todos los atributos deben sincronizarse automáticamente.

Se define conceptualmente:

External-Owned

Atributos cuyo origen es el sistema externo.

Ejemplos potenciales:

external subject;
external username;
determinados atributos corporativos.
EVOXA-Owned

Atributos administrados localmente.

Ejemplos:

Organization Membership.
EVOXA roles.
EVOXA permissions.
Security policy state.
Local account status.
Shared

Atributos donde ambos sistemas pueden participar mediante reglas explícitas.

Ejemplo:

nombre;
apellido;
email.

La estrategia concreta de ownership de cada atributo deberá definirse antes de implementar provisioning.

18. Email como Identificador

El email no será considerado identificador universal de Federation.

La identidad federada deberá basarse preferentemente en:

provider + external_subject

y no simplemente:

email

Esto evita:

account takeover;
linking incorrecto;
conflictos entre proveedores;
cambios de email externos.

La vinculación de cuentas existentes requerirá controles adicionales.

19. Group-to-Role Synchronization

Los grupos externos no deberán convertirse automáticamente en permisos.

Flujo permitido:

External Group
      ↓
Explicit Mapping Policy
      ↓
EVOXA Role
      ↓
Permissions

No:

External Group
      ↓
Direct Permissions

Esto mantiene a EVOXA como autoridad local de autorización.

20. Synchronization Failures

Un fallo temporal del proveedor externo no debe provocar automáticamente una desactivación masiva.

Ejemplo:

IdP unavailable

no significa:

All federated users = DISABLED

Se deberá distinguir entre:

Authentication Failure

No se puede autenticar al usuario.

Synchronization Failure

No se pudo actualizar el lifecycle.

Security Signal

Existe evidencia de que el acceso debe ser revocado.

Cada situación tendrá tratamiento diferente.

21. Fail-Safe Deprovisioning

Las señales de revocación suficientemente confiables deberán tener prioridad sobre estados anteriores.

Por ejemplo:

External User Suspended
        ↓
Validate trusted source
        ↓
Local Membership SUSPENDED
        ↓
Revoke/Block refresh

Sin embargo, EVOXA no deberá aceptar cualquier evento externo como revocación válida.

La integración debe validar:

origen;
autenticidad;
tenant/provider binding;
integridad;
event identity;
timestamp;
replay;
mapping;
lifecycle policy.
22. Organization-Level Provisioning

Una Identity Provider Federation podrá estar asociada a una o varias Organizations, pero nunca se asumirá acceso global.

Ejemplo:

IdP
 ├── Organization A → ACTIVE
 ├── Organization B → NOT AUTHORIZED
 └── Organization C → SUSPENDED

La existencia de una identidad federada no constituye autorización transversal.

Esto mantiene el aislamiento multi-tenant.

23. Session and Token Integration

Provisioning y deprovisioning deberán integrarse con:

Session Management.
Refresh Token Lifecycle.
Account Lifecycle.
Authorization.
Security Policy Engine.

Una desactivación relevante podrá provocar:

User / Membership State Change
        ↓
Re-evaluate Authorization
        ↓
Revoke Sessions
        ↓
Revoke Refresh Families
        ↓
Security Event
        ↓
Audit Event

La decisión exacta sobre cuándo revocar sesiones deberá depender de la causa y de la Security Policy.

24. Event Model

Los cambios de lifecycle deberán generar eventos de dominio cuando correspondan.

Ejemplos conceptuales:

identity.user.provisioned.v1
identity.user.activated.v1
identity.user.suspended.v1
identity.user.disabled.v1
identity.user.deprovisioned.v1

identity.membership.provisioned.v1
identity.membership.activated.v1
identity.membership.suspended.v1
identity.membership.removed.v1

identity.federation.identity.linked.v1
identity.federation.identity.unlinked.v1

Los eventos deberán respetar ADR-IDENTITY-007 y no contener:

passwords;
refresh tokens;
MFA secrets;
recovery codes;
private keys;
otros secretos.
25. Audit & Security Events

Provisioning y deprovisioning son operaciones sensibles.

Se deberán registrar:

actor;
source;
provider;
organization;
user/membership;
operation;
previous state;
new state;
reason;
correlation ID;
causation ID;
timestamp;
result.

Las señales externas deberán conservar suficiente trazabilidad para determinar:

quién solicitó el cambio, desde dónde llegó y qué política permitió aplicarlo.

26. Administrative Provisioning

Los administradores podrán realizar operaciones de lifecycle únicamente mediante APIs y servicios autorizados.

No se permitirá:

Admin → Direct DB Update

como mecanismo operativo normal.

Debe utilizarse:

Admin
 ↓
API
 ↓
Authorization
 ↓
Policy
 ↓
Application Service
 ↓
Domain
 ↓
Persistence + Outbox

Las operaciones masivas deberán tener controles adicionales.

27. AI Agents

Los AI Agents podrán:

detectar inconsistencias;
identificar usuarios potencialmente desactualizados;
sugerir acciones;
detectar anomalías;
preparar operaciones para aprobación;
analizar diferencias entre sistemas.

No podrán, por sí solos:

otorgar acceso;
crear permisos arbitrarios;
saltarse MFA;
desactivar controles de seguridad;
realizar cross-tenant provisioning;
eliminar evidencias;
convertir una recomendación en una operación crítica sin autorización.

Una operación realizada por un AI Agent deberá ser atribuible y auditable.

28. Concurrency

El lifecycle synchronization deberá ser concurrency-safe.

Escenario:

Admin → ACTIVATE
SCIM  → SUSPEND

procesados simultáneamente.

El sistema deberá evitar estados inconsistentes y definir precedencia mediante:

optimistic concurrency;
versionado;
transacciones;
políticas de conflicto.

La resolución exacta de conflictos queda pendiente.

29. Data Model — Conceptual

Se consideran relevantes:

User
Organization
Membership
ExternalIdentity
IdentityProvider
Invitation
ServiceAccount
ProvisioningOperation

ExternalIdentity conceptualmente:

id
provider_id
external_subject
user_id
status
created_at
updated_at
metadata

La clave lógica principal será:

(provider_id, external_subject)

y no el email.

ProvisioningOperation podrá registrar:

operation_id
source
provider
tenant
subject
operation_type
status
idempotency_key
correlation_id
created_at
completed_at
error_code

El modelo final se definirá en los ADR de persistence/schema correspondientes.

30. Transaction Boundary

Los cambios críticos deberán ejecutarse dentro de una transacción.

Ejemplo:

User/Membership State Change
        +
Outbox Event

deben confirmarse atómicamente.

No:

DB update
   ↓
commit

event publish
   ↓
failure

sin una estrategia de outbox.

Esto sigue ADR-IDENTITY-007.

31. Security Non-Negotiables

No se permitirá:

Cross-tenant provisioning accidental.
External group → permission directo.
Email como única prueba de account linking de alto riesgo.
Federation bypassing local authorization.
Secrets en provisioning events.
Direct DB lifecycle manipulation como flujo normal.
Fallo de IdP convertido automáticamente en disablement masivo.
Reactivación automática de cuentas suspendidas por seguridad sin política.
AI Agent con autoridad ilimitada.
Replay de eventos provocando efectos duplicados.
Deprovisioning que reactive accidentalmente una identidad.
Pérdida de revocaciones durante recuperación/restore.
32. Consequences
Positivas
Lifecycle centralizado.
Menor duplicación entre local auth, federation y futuros SCIM.
Mejor aislamiento multi-tenant.
Deprovisioning seguro.
Menor riesgo de access residue.
Auditoría completa.
Compatible con Enterprise Identity.
Compatible con AI-native operations.
Compatible con event-driven architecture.
Facilita futuras integraciones con directorios empresariales.
Negativas
Mayor complejidad de lifecycle.
Necesidad de resolver conflictos entre fuentes.
Necesidad de idempotencia.
Necesidad de adapters para cada proveedor.
Mayor esfuerzo de testing.
Provisioning y deprovisioning requieren políticas explícitas.
33. Alternatives Considered
A. External IdP como autoridad completa

Rejected.

No es compatible con la necesidad de que EVOXA controle Organization, Membership, roles, permissions y policies.

B. Email como identidad federada

Rejected.

No proporciona una identidad federada suficientemente robusta.

C. Cada integración administra directamente User

Rejected.

Generaría lógica duplicada y estados inconsistentes.

D. Eliminación física inmediata

Rejected as default.

Compromete auditoría, integridad histórica y recuperación.

E. Sincronización eventual sin idempotencia

Rejected.

No es compatible con At-Least-Once Delivery.

34. Traceability
Blueprint
Identity Domain.
Authentication.
Authorization.
Organization/Tenant Management.
Membership Management.
Federation.
Session Management.
Audit.
Event-Driven Architecture.
Tenant Isolation.
Security by Design.
Architecture
Identity
 ├── User
 ├── Organization
 ├── Membership
 ├── ExternalIdentity
 ├── IdentityProvider
 └── Lifecycle Services
Related ADRs
ADR-IDENTITY-001 — Identity Architecture
ADR-IDENTITY-005 — Authorization Model
ADR-IDENTITY-007 — Domain Events & Outbox
ADR-IDENTITY-008 — Cryptographic Key Management
ADR-IDENTITY-009 — Token Storage
ADR-IDENTITY-012 — PostgreSQL Schema
ADR-IDENTITY-013 — Data Retention
ADR-IDENTITY-017 — Runtime & Deployment
ADR-IDENTITY-021 — Controlled Event Replay
ADR-IDENTITY-022 — Consumer Idempotency
ADR-IDENTITY-025 — Refresh Token Cryptography & Concurrency
ADR-IDENTITY-027 — Runtime Configuration, Secrets & KMS
ADR-IDENTITY-028 — Observability & Operational Security
ADR-IDENTITY-029 — Disaster Recovery & Backup
ADR-IDENTITY-031 — High Availability & Failover
ADR-IDENTITY-033 — Account Lifecycle
ADR-IDENTITY-035 — Account Recovery
ADR-IDENTITY-036 — Email Verification
ADR-IDENTITY-037 — API Tokens & Service Credentials
ADR-IDENTITY-038 — Service-to-Service Authentication
ADR-IDENTITY-039 — OAuth2.1 & OIDC
ADR-IDENTITY-040 — Federation & External Identity Providers
35. Pending Decisions

Este ADR no fija todavía:

SCIM versión y endpoints.
JIT provisioning exacto.
Pre-provisioning.
Mapping de atributos.
Attribute ownership.
Group-to-role mappings.
Conflict resolution.
Provisioning operation state machine definitiva.
Deprovisioning grace period.
Session revocation rules por tipo de evento.
Automatic reactivation.
Provider outage behavior.
Multi-provider linking.
Account linking assurance levels.
SCIM webhook/event strategy.
Bulk provisioning limits.
Exact idempotency model.
Exact ordering/versioning strategy.
Retention de provisioning history.
Dual-control para operaciones administrativas críticas.

Estos puntos deberán resolverse en ADRs posteriores o en las Engineering Specifications correspondientes.

36. Acceptance Criteria

El diseño podrá considerarse aceptado cuando:

 User y Membership tengan lifecycle claramente separado.
 EVOXA sea la autoridad local de autorización.
 Federation no otorgue acceso implícito.
 Provisioning pueda ser idempotente.
 Deprovisioning pueda revocar acceso futuro.
 Refresh tokens se integren con lifecycle.
 Sessions puedan revocarse según policy.
 ExternalIdentity utilice provider + external_subject.
 Group-to-role mapping sea explícito.
 Cross-tenant provisioning sea imposible por diseño.
 Cambios de lifecycle generen audit/security events.
 Domain state y outbox sean transaccionales.
 Eventos sean replay-safe.
 Fallos temporales del IdP no provoquen disablement masivo.
 Operaciones administrativas estén autorizadas y auditadas.
 AI Agents no puedan saltarse controles de Identity.
 El diseño sea compatible con futura implementación SCIM.
 Recovery/restore preserve estados de revocación.
 Existan pruebas de concurrencia, replay, deprovisioning y aislamiento tenant.
37. Next ADR

El siguiente paso natural después de definir cómo se provisionan y desprovisionan identidades es definir cómo se administrarán los conflictos y sincronización entre múltiples fuentes de identidad.

ADR-IDENTITY-042 — Identity Source of Truth, Attribute Ownership & Synchronization Conflict Resolution Strategy

Ese ADR debería resolver especialmente:

Local Identity
      +
OIDC Provider
      +
SAML Provider
      +
SCIM
      +
Administrative Changes
      ↓
¿Cuál fuente gana?
¿Qué atributos puede modificar cada una?
¿Cómo resolvemos conflictos?
¿Cómo evitamos que una sincronización sobrescriba
información administrada por EVOXA?

Con ADR-041 ya dejamos definido el marco de provisioning/deprovisioning; el 042 puede encargarse de la autoridad por atributo y la resolución de conflictos, sin mezclar ambas decisiones.
