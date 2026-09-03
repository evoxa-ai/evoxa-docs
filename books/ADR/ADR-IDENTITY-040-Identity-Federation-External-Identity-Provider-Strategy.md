ADR-IDENTITY-040 — Identity Federation & External Identity Provider Strategy

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity
Priority: Critical
Type: Security / Architecture / Federation

1. Context

EVOXA deberá poder integrarse con proveedores externos de identidad para permitir que determinados usuarios se autentiquen utilizando una identidad administrada fuera de EVOXA.

Casos potenciales:

Google
Microsoft
Apple
Enterprise IdP
OIDC Provider
SAML Provider

La federación permitirá:

External Identity Provider
          ↓
      Authentication
          ↓
     EVOXA Identity
          ↓
   Local User Identity
          ↓
      Session
          ↓
   EVOXA Authorization

Sin embargo, la existencia de una identidad autenticada externamente no debe otorgar automáticamente permisos internos dentro de EVOXA.

La arquitectura debe mantener la separación:

External Authentication
        ≠
EVOXA Identity
        ≠
EVOXA Authorization

Este ADR continúa especialmente:

ADR-IDENTITY-005 — Authorization Model;
ADR-IDENTITY-008 — Cryptographic Key Management;
ADR-IDENTITY-023 — MFA Factor Strategy;
ADR-IDENTITY-027 — Runtime Configuration, Secrets & KMS;
ADR-IDENTITY-028 — Observability & Operational Security;
ADR-IDENTITY-033 — Account Lifecycle;
ADR-IDENTITY-035 — Account Recovery;
ADR-IDENTITY-036 — Email Verification;
ADR-IDENTITY-037 — API Token & Service Credential Security;
ADR-IDENTITY-038 — Service-to-Service Authentication;
ADR-IDENTITY-039 — OAuth 2.1 & OpenID Connect Protocol Strategy.
2. Problem Statement

EVOXA necesita responder correctamente a situaciones como:

User authenticates with Microsoft
        ↓
How is the user represented in EVOXA?

o:

User authenticates with Google
        ↓
Does EVOXA create a new account?

o:

Enterprise IdP says:
department = sales
        ↓
Does EVOXA automatically assign:
role = Commercial?

La respuesta no debe ser implícita.

Una federación mal diseñada puede provocar:

account takeover;
account duplication;
incorrect account linking;
privilege escalation;
cross-tenant access;
trust boundary violations;
dependencia excesiva del IdP externo;
problemas de lifecycle cuando un usuario abandona una organización;
claims malinterpretados.
3. Decision

EVOXA adoptará un modelo de Federated Authentication with Local Identity and Local Authorization.

El principio principal será:

External Identity Providers authenticate users; EVOXA remains authoritative for local identity state, tenant membership and authorization.

Conceptualmente:

External IdP
     ↓
External Authentication
     ↓
Federation Adapter
     ↓
Identity Mapping
     ↓
EVOXA User
     ↓
Organization / Membership
     ↓
RBAC + ABAC + Policy

El IdP externo no será automáticamente la autoridad final sobre:

roles EVOXA;
permissions EVOXA;
tenant membership;
account lifecycle;
local security policy.
4. Supported Federation Protocols

EVOXA podrá soportar conceptualmente:

OpenID Connect
SAML 2.0

La estrategia preferida para nuevos proveedores será OIDC cuando el proveedor lo soporte adecuadamente.

SAML podrá utilizarse principalmente para integraciones Enterprise donde sea necesario.

La implementación concreta de cada protocolo queda pendiente.

5. Federation Provider

Se definirá conceptualmente una entidad/configuración:

IdentityProvider
├── id
├── organization_id
├── type
├── issuer
├── client_id
├── status
├── configuration
├── mapping_policy
├── created_at
└── updated_at

Tipos conceptuales:

OIDC
SAML

Los secrets de configuración deberán mantenerse fuera de la base de datos cuando corresponda, utilizando Secret Manager/KMS.

6. Provider Lifecycle

Un proveedor federado tendrá lifecycle explícito:

DRAFT
   ↓
CONFIGURED
   ↓
ACTIVE
   ↓
SUSPENDED
   ↓
DISABLED

Un proveedor deshabilitado no deberá permitir nuevas autenticaciones federadas.

Las sesiones existentes deberán seguir las políticas definidas por Identity.

7. Trust Boundary

La federación establece una relación de confianza:

External IdP
      │
      │ Authentication Assertion
      ▼
EVOXA Identity

EVOXA deberá validar cuidadosamente:

issuer;
signature;
audience;
expiration;
protocol state;
nonce cuando corresponda;
certificate/signing key;
subject;
tenant/provider context.

Nunca se deberá confiar simplemente en que:

email = user@example.com

es suficiente para considerar autenticada una identidad externa.

8. External Identity

EVOXA deberá mantener una representación explícita de la identidad externa.

Conceptualmente:

ExternalIdentity
├── id
├── provider_id
├── external_subject
├── user_id
├── created_at
├── last_authenticated_at
└── metadata

La combinación:

provider
+
external_subject

deberá identificar de forma única la identidad externa.

9. External Subject

El subject entregado por el proveedor deberá considerarse el identificador principal de la identidad federada.

Conceptualmente:

Provider A
+
subject 12345

representa una identidad diferente de:

Provider B
+
subject 12345

No se debe asumir que el mismo sub entre proveedores representa a la misma persona.

10. Email Is Not the Primary Federation Key

El email no debe ser utilizado como único identificador universal para federation.

Ejemplo peligroso:

Google:
user@example.com

Microsoft:
user@example.com

→ Assume same identity

Esto puede generar account linking incorrecto.

El mapping primario deberá utilizar:

Provider
+
External Subject

El email puede actuar como señal adicional según policy.

11. Identity Mapping

El proceso conceptual será:

External Assertion
       ↓
Validate Assertion
       ↓
Identify Provider
       ↓
Extract External Subject
       ↓
Find ExternalIdentity
       ↓
Existing?
   ┌───┴───┐
   │       │
  YES      NO
   │       │
   ▼       ▼
 User   Provision /
        Link Policy

El resultado final siempre deberá ser una identidad EVOXA válida.

12. Existing External Identity

Si existe:

Provider + Subject → User

el usuario podrá continuar el flujo de autenticación sujeto a:

User lifecycle;
Membership;
Tenant policy;
Security Policy;
MFA requirements.

Una autenticación externa válida no debe ignorar que el usuario EVOXA puede estar:

SUSPENDED
DISABLED
13. New External Identity

Si la identidad externa no existe, EVOXA deberá aplicar una política de provisioning.

Posibilidades:

ALLOW AUTO-PROVISION
REQUIRE INVITATION
REQUIRE ADMIN APPROVAL
DENY

La política puede variar por organización.

14. Just-in-Time Provisioning

EVOXA podrá soportar:

First Federated Login
        ↓
Identity Validation
        ↓
Create User
        ↓
Create Membership
        ↓
Assign Allowed Role
        ↓
Create Session

Esto se denomina conceptualmente Just-in-Time Provisioning.

No deberá habilitarse universalmente.

Debe depender de la policy del tenant.

15. Invitation-Based Federation

Para organizaciones que requieran mayor control:

Admin Invitation
      ↓
External Identity Authentication
      ↓
Match Invitation
      ↓
Accept Membership
      ↓
Activate Membership

Este modelo reduce el riesgo de que cualquier usuario del dominio corporativo pueda ingresar automáticamente.

16. Tenant Discovery

Cuando un usuario se autentique mediante un IdP Enterprise, EVOXA deberá determinar de manera segura a qué organización aplica el proveedor.

No debe confiar únicamente en:

email domain

cuando exista riesgo de ambigüedad.

El provider configuration deberá establecer explícitamente:

Provider
     ↓
Organization

cuando sea tenant-bound.

17. Domain-Based Discovery

EVOXA podrá utilizar el dominio del email como mecanismo de discovery:

user@empresa.cl
      ↓
empresa.cl
      ↓
Configured Enterprise IdP

pero esto debe considerarse:

Discovery mechanism, not authentication proof.

El dominio no reemplaza la validación criptográfica de la identidad federada.

18. Claim Mapping

Los proveedores pueden entregar claims como:

sub
email
name
given_name
family_name
preferred_username
groups
roles
department

EVOXA deberá utilizar un mapping explícito.

Conceptualmente:

External Claim
      ↓
Mapping Policy
      ↓
EVOXA Attribute

No todos los claims deben copiarse.

19. Claim Trust Levels

No todos los claims tienen el mismo nivel de confianza.

Por ejemplo:

sub

puede utilizarse para identificar la identidad federada.

Mientras:

department = sales

no debería convertirse automáticamente en:

role = Commercial

sin una política explícita.

20. External Groups to EVOXA Roles

Podrá existir mapping:

External Group
      ↓
Mapping Policy
      ↓
EVOXA Role

Ejemplo conceptual:

"evoxa-coaches"
       ↓
Coach

Pero este mapping deberá ser:

explícito;
tenant-scoped;
auditable;
versionado;
revocable.

Nunca deberá asumirse automáticamente.

21. Role Mapping Security

Un proveedor externo no podrá enviar arbitrariamente:

role = Administrator

y obtener privilegios de Administrator.

El sistema debe verificar:

Provider
+
Tenant
+
Mapping Policy
+
Allowed External Group
+
Platform Policy

antes de realizar la asignación.

22. Permission Mapping

La federación no debe asignar permissions directamente de forma arbitraria.

La arquitectura preferida será:

External Group
      ↓
EVOXA Role
      ↓
EVOXA Permissions

en lugar de:

External Claim
      ↓
Direct Permission

Esto mantiene centralizado el modelo RBAC/ABAC.

23. Local Authorization Authority

Una vez autenticado el usuario:

External IdP
      ↓
Authentication
      ↓
EVOXA User
      ↓
Local Authorization

EVOXA será la autoridad para:

Membership;
Roles;
Permissions;
Ownership;
Tenant isolation;
Security Policy.
24. Federation and MFA

La federación debe integrarse con MFA.

Existen dos posibilidades conceptuales:

External MFA
User
 ↓
External IdP MFA
 ↓
EVOXA
EVOXA MFA
User
 ↓
External IdP
 ↓
EVOXA MFA
 ↓
Session

La política deberá determinar qué nivel de assurance se considera suficiente.

No se debe asumir automáticamente que cualquier MFA externo equivale a cualquier requisito de MFA de EVOXA.

25. Authentication Assurance

Cuando un proveedor externo entregue información sobre el nivel de autenticación, EVOXA podrá utilizarla como parte del contexto.

Conceptualmente:

External Authentication Assurance
        ↓
Security Policy
        ↓
Sufficient?

Si no es suficiente:

REQUIRE_MFA

o:

REAUTHENTICATE
26. External IdP Compromise

EVOXA debe asumir que un proveedor externo puede eventualmente verse comprometido.

Por ello, la federación no elimina:

account lifecycle;
risk detection;
local policy;
session controls;
audit;
anomaly detection.

Ante un incidente del IdP:

Provider Compromise
      ↓
Suspend Federation
      ↓
Assess Existing Sessions
      ↓
Risk Evaluation
      ↓
Revoke Sessions if required
27. Provider Suspension

Si un IdP se suspende:

Provider = SUSPENDED

las nuevas autenticaciones deberán rechazarse.

Las sesiones existentes requieren una política explícita.

Para proveedores críticos puede ser necesario:

Provider Suspension
      ↓
Revoke related sessions

según riesgo.

28. Signing Key Rotation

OIDC/SAML providers utilizan material criptográfico para firmar assertions.

EVOXA deberá soportar rotation.

Para OIDC:

JWKS
   ↓
Current Signing Key
   ↓
Rotation
   ↓
New Key

EVOXA deberá poder manejar períodos de transición sin aceptar claves no autorizadas.

Para SAML se aplicará un modelo equivalente mediante certificados configurados/trusted metadata.

29. Federation Metadata

La configuración del provider deberá poder mantener:

issuer;
endpoints;
JWKS;
client configuration;
redirect URIs;
SAML metadata cuando corresponda;
certificate information;
allowed scopes;
mapping policies.

Los secretos deberán estar separados de configuración pública.

30. External Provider Secrets

Client secrets, private keys y otros secretos deberán seguir ADR-IDENTITY-027.

Nunca:

Git
Source Code
Docker Image
Frontend
Logs
Events

La aplicación deberá obtenerlos desde mecanismos seguros.

31. Account Linking

El linking entre una identidad externa y una cuenta EVOXA existente es una operación sensible.

Flujo recomendado:

Authenticated EVOXA User
       ↓
Link External Identity
       ↓
External Provider Authentication
       ↓
Validate Provider + Subject
       ↓
Security Policy
       ↓
Create ExternalIdentity Mapping

No se deberá permitir linking basado únicamente en email.

32. Automatic Account Linking

Automatic linking basado solamente en:

same email

será considerado inseguro para escenarios de alto riesgo.

Si se permite, deberá requerir condiciones adicionales definidas por policy.

Por defecto:

Existing account linking should require stronger evidence than email equality alone.

33. Account Takeover Protection

Un atacante que controle temporalmente una identidad externa no debería poder vincularla arbitrariamente a otra cuenta EVOXA.

Por eso:

External Identity
+
Existing EVOXA Account

requiere una operación explícita y protegida.

34. Unlinking

Eliminar una identidad federada también es una operación sensible.

Ejemplo:

User
 ├── Password
 ├── Google
 └── Microsoft

Si se elimina Google:

Google → unlinked

EVOXA debe comprobar que el usuario conserve al menos un mecanismo válido de acceso cuando la política lo requiera.

No se debe permitir accidentalmente:

Remove last authentication mechanism
        ↓
Account inaccessible
35. Federation and Account Recovery

Federation puede utilizarse como evidencia para recovery cuando la policy lo permita.

Pero:

External IdP Authentication

no debe convertirse automáticamente en:

Unlimited Account Recovery Authority

ADR-035 continúa aplicándose.

36. Federation and Email Verification

Un claim email_verified recibido desde OIDC puede formar parte de la evidencia de email ownership.

Pero EVOXA debe definir explícitamente cuánto confía en ese claim.

Conceptualmente:

External email_verified
       ↓
Provider Trust Policy
       ↓
EVOXA Email State

No todos los proveedores deben recibir idéntico nivel de confianza sin configuración.

37. User Provisioning

Si se permite provisioning automático, deberá crearse:

User
+
ExternalIdentity
+
Membership

y potencialmente:

Role assignment

según mapping policy.

El provisioning deberá respetar:

tenant;
account state;
role policy;
invitation policy;
audit.
38. Deprovisioning

Un usuario puede dejar de existir o dejar de pertenecer a una organización externa.

EVOXA deberá soportar:

External Deprovisioning
       ↓
Membership Suspension/Removal

cuando el proveedor y la integración lo permitan.

Debe distinguirse:

User Disabled

de:

Membership Removed

como ya se estableció en el modelo de lifecycle.

39. SCIM

EVOXA podrá soportar posteriormente SCIM para provisioning/deprovisioning Enterprise.

Conceptualmente:

Enterprise IdP
      ↓
SCIM
      ↓
EVOXA Identity
      ↓
Users / Memberships

SCIM deberá considerarse un mecanismo de lifecycle/provisioning, no un mecanismo de autenticación.

La decisión de soporte inicial queda pendiente.

40. Federation and Organization Membership

La autenticación externa no crea necesariamente membership.

Debe distinguirse:

External Authentication

de:

Organization Membership

Una persona puede estar autenticada correctamente y aun así no tener autorización para acceder a una organización concreta.

41. Multi-Organization Users

Un usuario EVOXA puede potencialmente tener:

User
 ├── Membership → Organization A
 ├── Membership → Organization B
 └── External Identity

La federación debe resolver el contexto organizacional de manera segura.

No se debe asumir que la identidad externa pertenece automáticamente a todas las organizaciones del usuario.

42. Session Creation

Después de una autenticación federada exitosa:

External Authentication
       ↓
Identity Mapping
       ↓
Policy
       ↓
User State
       ↓
Membership
       ↓
MFA / Step-Up if required
       ↓
EVOXA Session

La sesión será una sesión EVOXA y seguirá el lifecycle establecido por Identity.

43. Token Issuance

El proveedor externo no debe entregar directamente un token utilizable contra APIs EVOXA.

El flujo será:

External IdP
      ↓
Authentication Assertion
      ↓
EVOXA Identity
      ↓
Validate
      ↓
Create/Resolve EVOXA Identity
      ↓
EVOXA Session
      ↓
EVOXA Access Token

Esto mantiene el control de EVOXA sobre sus APIs.

44. Token Boundary

Debe existir una frontera clara:

External IdP Token
        ≠
EVOXA Access Token

Un access token emitido por Google/Microsoft/u otro proveedor no debe ser aceptado directamente por APIs EVOXA salvo que la arquitectura defina explícitamente esa relación de confianza.

La opción predeterminada será:

EVOXA APIs trust EVOXA-issued access credentials.

45. Federation Adapter

Los protocolos externos deberán aislarse mediante adapters:

External Provider
       ↓
OIDC/SAML Adapter
       ↓
Federation Application Service
       ↓
Identity Domain

El Domain no deberá depender directamente de:

Google SDK;
Microsoft SDK;
SAML library;
proveedor concreto.
46. Protocol Validation

La capa de federation deberá validar:

OIDC
issuer;
signature;
audience;
expiration;
nonce;
state;
authorization code;
PKCE;
discovery metadata según configuración.
SAML
signature;
issuer;
audience;
destination;
timestamps;
assertion conditions;
certificate trust;
replay controls.

Los detalles exactos se definirán en la correspondiente Protocol Specification.

47. Replay Protection

Assertions y authorization responses no deben poder reutilizarse indefinidamente.

Se deberán aplicar mecanismos apropiados como:

short expiration;
nonce;
state;
authorization code one-time use;
assertion ID tracking cuando corresponda;
audience restriction.
48. Clock Skew

La validación de assertions depende de timestamps.

EVOXA deberá establecer una tolerancia controlada para:

iat;
nbf;
exp;
SAML NotBefore;
SAML NotOnOrAfter.

La tolerancia exacta queda pendiente.

No deberá existir una tolerancia excesivamente amplia.

49. Federation Rate Limiting

Los endpoints federados deberán estar protegidos mediante:

Rate Limiting
+
Abuse Detection
+
Provider-specific controls

Especialmente:

authorization initiation;
callbacks;
account linking;
provider discovery;
provisioning.
50. Observability

Deberá poder observarse:

provider
protocol
authentication_success
authentication_failure
mapping_success
mapping_failure
account_link
account_unlink
provisioning
deprovisioning
provider_latency
provider_error
key_rotation
policy_denial

Nunca deberán registrarse:

ID tokens completos;
SAML assertions completas;
access tokens;
authorization codes;
client secrets.
51. Audit Events

Eventos conceptuales:

IDENTITY_PROVIDER_CREATED
IDENTITY_PROVIDER_UPDATED
IDENTITY_PROVIDER_ENABLED
IDENTITY_PROVIDER_DISABLED
FEDERATED_AUTHENTICATION_STARTED
FEDERATED_AUTHENTICATION_COMPLETED
FEDERATED_AUTHENTICATION_FAILED
EXTERNAL_IDENTITY_LINKED
EXTERNAL_IDENTITY_UNLINKED
FEDERATED_USER_PROVISIONED
FEDERATED_MEMBERSHIP_CREATED
FEDERATED_MEMBERSHIP_REMOVED
FEDERATION_MAPPING_CHANGED

Todos deberán incluir contexto suficiente para investigación.

52. Security Events

Casos de seguridad:

FEDERATION_SIGNATURE_FAILURE
FEDERATION_ISSUER_MISMATCH
FEDERATION_AUDIENCE_MISMATCH
FEDERATION_NONCE_FAILURE
FEDERATION_REPLAY_DETECTED
FEDERATION_ACCOUNT_LINKING_ANOMALY
FEDERATION_ROLE_MAPPING_VIOLATION
FEDERATION_CROSS_TENANT_ATTEMPT
FEDERATION_PROVIDER_COMPROMISE
FEDERATION_SUSPICIOUS_AUTHENTICATION
53. AI Integration

AI podrá ayudar a:

detectar anomalías;
identificar patrones de takeover;
analizar provider failures;
detectar cambios anormales de membership;
recomendar suspensión temporal.

Pero AI no podrá:

confiar automáticamente en un claim externo;
asignar Administrator;
vincular cuentas arbitrariamente;
cambiar mapping policies;
activar un IdP;
desactivar controles de seguridad;
convertir DENY en ALLOW.
54. Security Policy

Toda federated authentication deberá integrarse con:

External Identity
+
User
+
Organization
+
Membership
+
Authentication Assurance
+
Risk
+
Provider
+
Operation
+
Policy

La política puede determinar:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
55. Disaster Recovery

La recuperación debe preservar:

Identity Provider configuration;
provider status;
ExternalIdentity mappings;
organization bindings;
role mappings;
membership state;
trusted signing metadata;
audit events.

Los secretos deberán recuperarse mediante KMS/Secret Manager según ADR-027.

56. Consequences
Positivas
Permite Enterprise SSO.
Reduce necesidad de passwords locales.
Mejora interoperabilidad.
Permite Google/Microsoft/Enterprise IdPs.
Mantiene autorización bajo control EVOXA.
Permite provisioning futuro.
Facilita integración empresarial.
Permite AI-assisted federation security.
Mantiene tenant isolation.
Negativas
Mayor complejidad de Identity.
Dependencia de proveedores externos.
Federation introduce nuevas trust boundaries.
Claim mapping puede ser complejo.
Account linking es una superficie crítica.
Provisioning/deprovisioning requiere lifecycle adicional.
57. Alternatives Considered
A. No Federation

Rejected.

Limitaría las capacidades Enterprise y SSO de EVOXA.

B. Trust External IdP for Authorization

Rejected.

El IdP externo no debe convertirse automáticamente en autoridad de roles/permisos internos.

C. Email as Universal Identity Key

Rejected.

No garantiza identidad estable ni evita account-linking attacks.

D. Accept External Access Tokens Directly in EVOXA APIs

Rejected as default.

Las APIs EVOXA deberán confiar preferentemente en credentials emitidas por EVOXA.

E. Automatic Role Mapping from Arbitrary Claims

Rejected.

Puede provocar privilege escalation.

F. Automatic Account Linking by Email

Rejected as default.

Es una superficie potencial de account takeover.

58. Non-Negotiable Security Rules
External IdP authentication no equivale a EVOXA authorization.
provider + subject será la identidad externa principal.
Email no será el identificador universal de federation.
External claims deben tener mapping explícito.
External roles no se convierten automáticamente en EVOXA roles.
EVOXA mantiene autoridad sobre Membership.
EVOXA mantiene autoridad sobre Permissions.
EVOXA mantiene autoridad sobre Security Policy.
Account linking debe estar protegido.
Email equality no será suficiente para linking de alto riesgo.
External tokens no deben ser aceptados directamente por APIs EVOXA por defecto.
EVOXA emitirá sus propias sesiones/credentials.
Federation provider secrets deben utilizar Secret Manager/KMS.
Provider signing keys deben validarse.
Replay protection es obligatoria.
Federation debe respetar User Account Lifecycle.
Federation debe respetar Tenant Isolation.
Provider suspension debe bloquear nuevas autenticaciones.
Federation failures deben fail-closed.
AI no puede elevar privilegios mediante federation.
Federation events deben ser auditables.
Assertions/tokens no deben aparecer en logs.
Deprovisioning debe distinguir User de Membership.
Role mapping debe ser versionado y auditable.
External MFA no debe considerarse automáticamente equivalente a EVOXA MFA.
59. Traceability
Requirement	Source / Relationship
Identity Architecture	Identity ESP / ARCHITECTURE-MAP
Authorization	ADR-IDENTITY-005
Cryptographic Security	ADR-IDENTITY-008
MFA	ADR-IDENTITY-023/024
Secrets / KMS	ADR-IDENTITY-027
Observability	ADR-IDENTITY-028
Account Lifecycle	ADR-IDENTITY-033
Account Recovery	ADR-IDENTITY-035
Email Verification	ADR-IDENTITY-036
Service Credentials	ADR-IDENTITY-037
Service-to-Service	ADR-IDENTITY-038
OAuth/OIDC	ADR-IDENTITY-039
External Federation	ADR-IDENTITY-040

La trazabilidad continúa siguiendo el lifecycle definido por EVOXA:

ESP
 ↓
ADR
 ↓
Implementation Story
 ↓
Technical Tasks
 ↓
Source Code
 ↓
Tests

El Blueprint exige explícitamente esta relación de trazabilidad entre especificaciones, stories, tareas e implementación.

60. Dependencies

Principalmente:

ADR-IDENTITY-005
ADR-IDENTITY-008
ADR-IDENTITY-023
ADR-IDENTITY-024
ADR-IDENTITY-027
ADR-IDENTITY-028
ADR-IDENTITY-033
ADR-IDENTITY-035
ADR-IDENTITY-036
ADR-IDENTITY-037
ADR-IDENTITY-038
ADR-IDENTITY-039

La implementación deberá materializarse posteriormente mediante:

IS-IDENTITY-040
Technical Tasks
API Contracts
Persistence Specification
Federation Protocol Specification
Security Specification
Event Specification
Testing Specification
61. Pending Decisions

Quedan deliberadamente abiertos:

Providers
Google;
Microsoft;
Apple;
Enterprise OIDC;
Enterprise SAML;
catálogo inicial de proveedores.
Protocols
OIDC implementation profile;
SAML support;
exact federation flows;
logout federation.
Provisioning
Just-in-Time provisioning;
invitation-only provisioning;
SCIM;
deprovisioning behavior.
Mapping
claim catalog;
group-to-role mapping;
role mapping precedence;
mapping versioning;
tenant-specific mappings.
Account Linking
exact verification requirements;
email matching policy;
multi-factor requirements;
automatic vs manual linking.
Security
external MFA assurance;
provider trust levels;
clock skew;
assertion replay retention;
provider compromise response.
Infrastructure
discovery mechanism;
JWKS caching;
SAML metadata management;
certificate rotation;
Secret Manager integration.

Estas decisiones deberán cerrarse antes de la implementación definitiva.

62. Acceptance Criteria

El ADR se considerará correctamente implementado cuando:

 EVOXA pueda representar External Identity de forma independiente.
 provider + subject identifique inequívocamente una identidad externa.
 OIDC pueda utilizarse para federated authentication.
 SAML pueda incorporarse para Enterprise Federation cuando corresponda.
 External assertions sean validadas criptográficamente.
 Issuer y audience sean validados.
 Replay protection sea implementada.
 Account linking esté protegido.
 Email equality no sea suficiente para linking de alto riesgo.
 Claim mapping sea explícito.
 External groups no asignen privilegios arbitrariamente.
 EVOXA mantenga autoridad sobre roles y permissions.
 EVOXA mantenga autoridad sobre memberships.
 Tenant isolation sea obligatorio.
 JIT provisioning pueda estar sujeto a policy.
 Deprovisioning pueda suspender/remover memberships.
 Provider lifecycle sea administrable.
 Provider suspension bloquee nuevas autenticaciones.
 External MFA pueda evaluarse mediante assurance policy.
 EVOXA emita sus propias sesiones/tokens.
 External tokens no sean utilizados directamente por APIs por defecto.
 Provider secrets estén protegidos.
 Federation operations sean auditables.
 Security Events permitan detectar ataques.
 AI no pueda modificar trust/mapping/authorization arbitrariamente.
 Existan pruebas de account linking, replay, claim injection, role escalation, tenant isolation y provider compromise.
63. Next ADR

Con ADR-IDENTITY-040 queda definida la estrategia de Identity Federation y External Identity Providers, completando la frontera entre:

                    EVOXA IDENTITY
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
     Users            Services          External IdP
        │                 │                 │
      OIDC             M2M/OAuth          OIDC/SAML
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ▼
                  EVOXA Identity
                          │
              ┌───────────┴───────────┐
              │                       │
        Local Session          Local Authorization
                                      │
                         RBAC + ABAC + Tenant
                         + Ownership + Policy

El siguiente ADR natural es:

ADR-IDENTITY-041 — Identity Provisioning, Deprovisioning & Lifecycle Synchronization Strategy

Ahí podremos definir formalmente JIT Provisioning, Invitations, SCIM, creación de Users/Memberships, sincronización de cambios desde Enterprise IdPs, suspensión, eliminación, conflictos de identidad y qué ocurre cuando un usuario deja una organización, antes de avanzar a la siguiente capa de decisiones de Identity.
