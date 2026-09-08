ESP-0001 / 07 — Identity Multi-Factor Authentication (MFA)

Document ID: ESP-0001/07
Module: Identity
Version: 1.1.0
Status: Draft — Consolidated
Parent: ESP-0001 — Identity
Architecture Parent: BP-0002 — Reference Architecture

Target Repository: evoxa-platform
Target Path: apps/api/app/domains/identity/

1. Purpose

Esta especificación define la capacidad de Multi-Factor Authentication (MFA) de EVOXA Identity.

MFA proporciona una capa adicional de verificación sobre la autenticación primaria y debe integrarse con:

Authentication
Sessions
Token Management
Authorization
Security Policy Engine
Account Lifecycle
Recovery
Audit
Security Events
Event Platform
Tenant Isolation
AI Security

MFA no sustituye Authentication ni Authorization.

2. Fundamental Security Model

El modelo conceptual es:

USER
 │
 ├── MFA FACTORS
 │      ├── TOTP
 │      ├── WebAuthn
 │      ├── Passkey
 │      ├── Email OTP
 │      ├── SMS OTP
 │      ├── Push
 │      └── Hardware Key
 │
 ├── MFA CHALLENGES
 │
 ├── RECOVERY CODES
 │
 └── TRUSTED DEVICES

MFA complementa:

Authentication
      +
MFA Verification
      ↓
Authenticated Context
      ↓
Session

Pero:

MFA
   ≠
Authorization

Una autenticación MFA exitosa no significa automáticamente que una operación esté autorizada.

3. Scope

Esta especificación cubre:

MFA Factor Management
MFA Enrollment
MFA Verification
MFA Challenges
TOTP
WebAuthn / Passkeys
OTP-based factors
Recovery Codes
Trusted Devices
Step-Up Authentication
MFA Assurance
MFA Revocation
MFA Recovery
MFA Security Policies
MFA Rate Limiting
MFA Audit
MFA Security Events
MFA Tenant Isolation
MFA integration with Sessions
MFA integration with Authorization
MFA integration with Criticality/Freshness
MFA integration with AI

No redefine en detalle:

Authentication;
Session Management;
Token Management;
Authorization Runtime;
Policy DSL;
API Contracts;
Persistence;
Event infrastructure.

Esos elementos pertenecen a sus respectivas ESP/ADR.

4. Design Principles

MFA debe respetar:

Zero Trust
Least Privilege
Secure by Default
Defense in Depth
Fail Closed
Tenant Isolation
Privacy by Design
Security by Design
Auditability
No Security Bypass

Además:

Un fallo de un proveedor MFA nunca debe transformarse en un ALLOW.

5. Factor Abstraction

El diseño debe utilizar una abstracción común:

MFAFactor
    │
    ├── TOTP
    ├── WebAuthn
    ├── Passkey
    ├── SMS
    ├── Email
    ├── Push
    └── Hardware Key

Esto permite agregar nuevos factores sin modificar el modelo principal de:

User
Session
Authorization
6. MFA Factor Catalog

El catálogo conceptual es:

Factor	Tipo
TOTP	OTP basado en tiempo
WebAuthn	Criptografía de clave pública
Passkey	Credencial basada en WebAuthn
Email OTP	Código temporal
SMS OTP	Código temporal
Push	Aprobación externa
Hardware Key	Dispositivo criptográfico

No todos los factores deben ser obligatorios desde el MVP.

La disponibilidad de un factor puede depender de:

plataforma;
organización;
política;
proveedor;
nivel de seguridad requerido;
capacidad del cliente.
7. MFAFactor

MFAFactor representa un factor registrado por un usuario.

Modelo conceptual:

MFAFactor
├── id
├── user_id
├── organization_id / scope
├── type
├── status
├── display_name
├── created_at
├── verified_at
├── last_used_at
├── revoked_at
├── metadata
└── version

Estados:

PENDING
ACTIVE
REVOKED
EXPIRED

Un factor PENDING no puede utilizarse como factor MFA activo.

8. Factor Ownership

Un factor debe estar asociado inequívocamente a una identidad.

User
  ↓
MFAFactor

No se debe permitir:

User A
   ↓
Factor perteneciente a User B

La asociación debe validarse antes de cada operación relevante.

9. TOTP

TOTP permite generar códigos temporales utilizando un secreto compartido.

Flujo:

User
 ↓
TOTP Enrollment
 ↓
Secret
 ↓
Authenticator
 ↓
OTP
 ↓
Verification
 ↓
ACTIVE

El secreto debe protegerse durante todo su ciclo de vida.

10. TOTP Secret Security

El secreto TOTP es información altamente sensible.

Nunca debe aparecer en:

Logs
Events
Metrics
Analytics
JWT
Audit payload
AI context
Normal API responses

Debe existir protección criptográfica adecuada en almacenamiento.

11. WebAuthn / Passkeys

WebAuthn/Passkeys utilizan criptografía de clave pública.

Modelo:

Device
 ├── Private Key
 └── Public Key
          │
          ▼
       EVOXA

EVOXA almacena únicamente la información necesaria para verificar la autenticación.

La clave privada permanece bajo control del autenticador/dispositivo.

Este mecanismo es especialmente adecuado para:

administradores;
usuarios empresariales;
operaciones sensibles;
autenticación resistente al phishing.
12. Email OTP

Email OTP puede funcionar como factor adicional.

Debe cumplir:

Short-lived
Single-use
Purpose-bound
Rate-limited
Replay-resistant

El correo electrónico no debe considerarse automáticamente equivalente a un factor criptográficamente fuerte.

13. SMS OTP

SMS OTP puede estar soportado por la arquitectura.

Sin embargo:

su utilización debe ser configurable;
depende de proveedores externos;
debe estar sujeta a rate limiting;
no debe asumirse como factor obligatorio del MVP.

Un fallo del proveedor SMS no debe producir un ALLOW.

14. Future Factors

El modelo debe permitir incorporar posteriormente:

Hardware Security Keys
Push Authentication
Biometrics mediante WebAuthn
Smart Cards
Enterprise Authentication
Nuevos mecanismos estandarizados

La incorporación de un factor nuevo no debe requerir rediseñar:

User
Session
Authorization
15. MFA Challenge

Un MFAChallenge representa una instancia concreta de verificación.

MFAChallenge
├── id
├── user_id
├── session_id
├── factor_id
├── purpose
├── status
├── created_at
├── expires_at
├── verified_at
├── attempt_count
├── metadata
└── version
16. Challenge Lifecycle

El lifecycle consolidado:

CREATED
   │
   ▼
PENDING
 ┌─┴──────────────┐
 ▼                ▼
VERIFIED        FAILED
 │                │
 │                ▼
 │             terminal
 │
 └── terminal

PENDING
   ├── EXPIRED
   └── CANCELLED

Un challenge terminal no puede volver a PENDING.

Un challenge VERIFIED no puede verificarse nuevamente.

17. Challenge Purpose

Los challenges deben estar asociados a un propósito.

Conceptualmente:

LOGIN
STEP_UP
ENROLLMENT
FACTOR_CHANGE
SENSITIVE_OPERATION
RECOVERY

El purpose debe impedir que un challenge creado para una operación sea reutilizado para otra.

Ejemplo:

LOGIN challenge
     ≠
MFA factor deletion challenge
18. Challenge Expiration

Todo challenge debe tener:

created_at
expires_at

La validez temporal es:

created_at <= now < expires_at

Una vez expirado:

Challenge
   ↓
EXPIRED

No puede ser utilizado posteriormente.

El TTL exacto debe ser una configuración gobernada y no se fija arbitrariamente en esta ESP.

19. Challenge Attempts

Cada challenge debe controlar:

attempt_count
max_attempts

Un exceso de intentos debe impedir nuevas verificaciones.

Ejemplo:

attempt 1 → FAIL
attempt 2 → FAIL
attempt 3 → FAIL
...
max_attempts
       ↓
FAILED

Los límites exactos deben definirse mediante configuración/política.

20. Concurrent Verification

La verificación debe ser concurrency-safe.

Si:

Client A ─┐
          ├── Challenge C
Client B ─┘

llegan simultáneamente, no debe ser posible producir:

C → VERIFIED
C → VERIFIED

como dos consumos independientes.

Debe existir una única transición válida.

21. Idempotency and Replay

La verificación de MFA debe ser resistente a replay.

Una vez:

Challenge C
    ↓
VERIFIED

una segunda utilización debe rechazarse.

Lo mismo aplica a:

Recovery Code
OTP
Verification Evidence

cuando corresponda.

22. Enrollment

El registro de un factor requiere autenticación previa.

Flujo:

Authenticated User
       │
       ▼
Request Enrollment
       │
       ▼
Security Policy
       │
       ▼
Create Factor PENDING
       │
       ▼
Challenge
       │
       ▼
Verify
       │
       ▼
Factor ACTIVE

Crear un factor no significa activarlo.

23. TOTP Enrollment

Flujo:

Request TOTP
     ↓
Generate Secret
     ↓
Protect Secret
     ↓
Enrollment
     ↓
User Configures Authenticator
     ↓
OTP Verification
     ↓
Factor ACTIVE
     ↓
Recovery Codes

La activación sólo ocurre después de una verificación válida.

24. Factor Activation

Regla:

Factor Created
    ≠
Factor Active

Debe cumplirse:

Valid Enrollment
+
Successful Verification
=
ACTIVE

Un actor no puede activar directamente un factor sin completar el proceso requerido.

25. Login with MFA

El flujo consolidado:

Credentials
    │
    ▼
Authenticate User
    │
    ▼
Validate Account
    │
    ▼
Evaluate MFA Policy
    │
 ┌──┴───────────────┐
 │                  │
 ▼                  ▼
MFA Not Required   MFA Required
 │                  │
 ▼                  ▼
Session          MFA Challenge
                     │
                     ▼
                 Verification
                     │
                     ▼
                   Session
26. MFA Policy

Security Policy Engine determina cuándo MFA es requerido.

Ejemplos:

MFA required
MFA required for role
MFA required for operation
MFA required for risk level
MFA required for organization

La política puede producir:

ALLOW / CONTINUE
REQUIRE_MFA
REAUTHENTICATE
CHALLENGE
DENY

según el contexto.

27. MFA Policy Ownership

MFA Policy puede existir a diferentes scopes:

PLATFORM
ORGANIZATION
RESOURCE / OPERATION

Una organización puede endurecer requisitos de MFA.

No puede debilitar un requisito mínimo impuesto por la plataforma.

Esto sigue las reglas de Policy Precedence y Platform Minimum.

28. Step-Up Authentication

Una sesión existente puede necesitar MFA adicional.

Active Session
      │
      ▼
Sensitive Operation
      │
      ▼
Authorization Runtime
      │
      ▼
REQUIRE_MFA
      │
      ▼
MFA Challenge
      │
      ▼
Verification
      │
      ▼
Fresh MFA Evidence
      │
      ▼
Continue Authorization

La evidencia de step-up debe tener una validez limitada.

29. Sensitive Operations

Ejemplos:

Change Password
Change MFA Factor
Remove MFA
Register Factor
Security Configuration
Assign Privileges
Modify Roles
Administrative Operations
Financially Sensitive Operations
Security Reset

La lista definitiva no debe estar codificada dentro de MFA.

Debe provenir del modelo de autorización/criticality/policy.

30. MFA Assurance

El contexto de autenticación puede registrar el nivel de assurance alcanzado.

Conceptualmente:

PASSWORD
MFA
STRONG_MFA
PHISHING_RESISTANT

Esto permite políticas más precisas:

Operation requires STRONG_MFA

en lugar de:

MFA = true

La clasificación concreta de factores debe quedar gobernada por Security Policy.

31. MFA Evidence

Una verificación MFA puede producir evidencia temporal:

MFA Evidence
├── factor
├── method
├── authentication_time
├── assurance
├── purpose
└── validity

La evidencia no debe convertirse automáticamente en una autorización permanente.

32. Session Integration

La Session puede mantener:

authentication_method
mfa_verified
mfa_evidence
authentication_time
assurance_level

Pero no debe almacenar secretos MFA.

El Access Token tampoco debe convertirse en almacén de datos MFA.

33. JWT Integration

Cuando sea necesario transmitir contexto de autenticación, pueden utilizarse claims conceptuales como:

amr
acr

pero únicamente cuando sean necesarios.

Nunca:

TOTP secret
Recovery codes
Private key
OTP values
34. MFA and Authorization

El flujo completo:

Authentication
      ↓
MFA
      ↓
Session
      ↓
Tenant Context
      ↓
Authorization Runtime
      ↓
RBAC
      ↓
Ownership
      ↓
ABAC
      ↓
Delegation
      ↓
Policy
      ↓
Criticality
      ↓
Freshness
      ↓
Decision

Por lo tanto:

MFA Success
    ≠
Authorization Success
35. MFA Revocation

Un factor debe poder revocarse.

User
 ├── TOTP       ACTIVE
 ├── WebAuthn   ACTIVE
 └── Email      ACTIVE

Si TOTP está comprometido:

TOTP
 ↓
REVOKED

La revocación debe:

marcar el factor como REVOKED;
invalidar challenges asociados cuando corresponda;
generar Security Event;
generar Audit Event;
invalidar estado/caches relacionados;
aplicar la política de sesiones correspondiente.
36. MFA Factor Replacement

Cambiar un factor es una operación sensible.

Debe protegerse mediante:

Authentication
+
Security Policy
+
Step-Up
+
MFA / Recovery Evidence

No debe permitirse:

Compromised Session
       ↓
Delete MFA
       ↓
Register New Factor

sin controles adicionales.

37. MFA Disable

Desactivar MFA es una operación crítica.

Flujo:

Authenticated Session
        │
        ▼
Authorization
        │
        ▼
Criticality Evaluation
        │
        ▼
Step-Up / MFA
        │
        ▼
Disable MFA
        │
        ▼
Update Security State
        │
        ▼
Revoke / Update Factors
        │
        ▼
Security Event
        │
        ▼
Audit

Un AI Agent no puede desactivar MFA por su cuenta.

38. Recovery Codes

Los Recovery Codes proporcionan un mecanismo alternativo cuando el usuario pierde acceso a sus factores.

Modelo:

RecoveryCode
├── id
├── user_id
├── code_hash
├── status
├── created_at
├── used_at
└── revoked_at

Estados:

ACTIVE
USED
REVOKED
39. Recovery Code Security

Los Recovery Codes deben:

generarse criptográficamente;
almacenarse protegidos;
nunca almacenarse en texto plano;
ser de un solo uso;
no aparecer en logs;
no aparecer en eventos;
no aparecer en JWT;
no ser recuperables mediante una API normal.

Una vez utilizado:

ACTIVE
   ↓
USED

No puede reutilizarse.

40. Recovery Code Regeneration

Cuando se genere un nuevo conjunto de Recovery Codes, la estrategia preferida es invalidar el conjunto anterior.

Old Codes
    ↓
REVOKED

New Codes
    ↓
ACTIVE

Esto evita mantener múltiples generaciones válidas simultáneamente.

41. MFA Recovery

La recuperación no es un bypass de MFA.

Recovery
   ≠
Disable Security

Debe existir un proceso de assurance apropiado.

Posibles evidencias:

Recovery Code
Verified Channel
Trusted Device
Identity Verification
Administrative Recovery

según política.

42. Administrative Recovery

La recuperación administrativa es especialmente sensible para:

Platform Administrators
Organization Administrators
High-Privilege Users
Sensitive Accounts

Debe requerir:

Explicit Authorization
+
Tenant Validation
+
Security Policy
+
Audit
+
Security Event

Un administrador de Organization A no puede recuperar MFA de un usuario perteneciente a Organization B.

43. Trusted Devices

Un TrustedDevice permite reducir la frecuencia de challenges cuando la política lo permita.

Modelo:

TrustedDevice
├── id
├── user_id
├── device_identifier
├── created_at
├── last_used_at
├── expires_at
├── revoked_at
└── status

Estados:

ACTIVE
EXPIRED
REVOKED
44. Trusted Device ≠ Identity

Un Trusted Device:

≠
User
≠
Session
≠
MFA Factor
≠
Permanent Authentication

Sólo puede actuar como evidencia adicional según la política.

No debe permitir saltarse MFA indefinidamente.

45. Trusted Device Security

Debe existir:

expiración;
revocación;
asociación al usuario;
contexto de dispositivo;
protección contra replay;
auditoría;
aislamiento por tenant.

El dispositivo confiable nunca debe tener privilegios superiores a los del usuario.

46. Tenant Isolation

Todas las operaciones MFA deben respetar el contexto de tenant.

Platform
 ├── Organization A
 │      └── Users
 │
 └── Organization B
        └── Users

Una operación MFA debe validar:

Actor
+
Target User
+
Membership
+
Organization
+
Permission

La incertidumbre de tenant debe resultar en:

DENY
47. MFA and Criticality

Las operaciones MFA pueden elevar su Criticality.

Ejemplo:

Normal MFA verification
        ↓
C2/C1 según contexto

Factor revocation
        ↓
C0

MFA disable
        ↓
C0

Security recovery
        ↓
C0

La Criticality efectiva sigue el modelo central:

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

MFA no calcula una autorización independiente.

48. MFA and Freshness

MFA evidence debe respetar Freshness.

Para operaciones críticas:

Critical Operation
       ↓
Fresh MFA Evidence
       ↓
Authorization

Una evidencia antigua no debe utilizarse indefinidamente para justificar una operación crítica.

Esto se integra con:

F0
F1
F2
F3

según Criticality.

49. MFA and Cache

MFA state puede afectar decisiones de autorización.

Por tanto:

MFA Factor Revoked
       ↓
Security State Change
       ↓
Security Epoch / Version
       ↓
Cache Invalidation

No debe ocurrir:

MFA Revoked
+
Old Cached ALLOW
=
ALLOW

para una operación que requiere MFA válido.

50. Event Integration

MFA utiliza el Event Platform mediante Transactional Outbox.

Eventos conceptuales:

MFAEnrollmentStarted
MFAFactorCreated
MFAFactorVerified
MFAFactorActivated
MFAFactorRevoked

MFAChallengeCreated
MFAChallengeVerified
MFAChallengeFailed
MFAChallengeExpired

MFARecoveryCodeGenerated
MFARecoveryCodeUsed
MFARecoveryInitiated

MFADisabled

TrustedDeviceRegistered
TrustedDeviceRevoked

StepUpAuthenticationRequired
StepUpAuthenticationCompleted

Los nombres definitivos y payloads corresponden a ESP-0001/12.

51. Event Security

Los eventos MFA nunca deben contener:

TOTP secrets
Recovery codes
OTP values
Private keys
Raw credentials
Authentication secrets

El evento representa el hecho de seguridad, no el secreto que lo produjo.

52. Audit

Las siguientes operaciones deben ser auditables:

Factor Enrollment
Factor Activation
Factor Verification
Failed Challenge
Factor Revocation
Factor Replacement
MFA Disable
Recovery
Recovery Code Use
Recovery Code Regeneration
Trusted Device Registration
Trusted Device Revocation
Step-Up
Administrative Recovery

La auditoría debe registrar contexto suficiente para investigación sin exponer secretos.

53. Rate Limiting

MFA debe tener controles multidimensionales:

User
IP
Session
Factor
Challenge
Organization
Endpoint

Especialmente:

OTP Verification
Enrollment
Recovery
Factor Registration
MFA Disable
Password Recovery

Los valores concretos pertenecen a configuración/política.

54. Provider Failure

Los factores externos pueden depender de proveedores:

SMS
Email
Push
Enterprise Identity

La arquitectura debe aislarlos mediante adapters.

Regla:

Provider Failure
      ↓
NOT ALLOW

Nunca:

MFA Provider Error
      ↓
Skip MFA
      ↓
ALLOW
55. MFA Architecture

La separación interna recomendada:

MFA
├── Domain
│   ├── MFAFactor
│   ├── MFAChallenge
│   ├── RecoveryCode
│   └── TrustedDevice
│
├── Application
│   ├── Enrollment
│   ├── Verification
│   ├── Recovery
│   ├── Revocation
│   └── StepUp
│
├── Infrastructure
│   ├── TOTP
│   ├── WebAuthn
│   ├── Email
│   ├── SMS
│   └── Push
│
└── API
    └── MFA endpoints

El Domain no debe depender directamente de proveedores externos.

56. API Boundary

Los endpoints conceptuales pueden incluir:

POST   /api/v1/identity/mfa/factors
GET    /api/v1/identity/mfa/factors
DELETE /api/v1/identity/mfa/factors/{factor_id}

POST   /api/v1/identity/mfa/challenges
POST   /api/v1/identity/mfa/challenges/{challenge_id}/verify

POST   /api/v1/identity/mfa/recovery-codes
POST   /api/v1/identity/mfa/recovery

GET    /api/v1/identity/mfa/trusted-devices
DELETE /api/v1/identity/mfa/trusted-devices/{device_id}

Estos son contratos conceptuales, no el contrato API definitivo.

El contrato definitivo será congelado en:

ESP-0001/10 — Identity API Contracts

57. Persistence Boundary

Las entidades conceptuales son:

mfa_factors
mfa_challenges
recovery_codes
trusted_devices

El esquema físico, constraints, índices, encryption-at-rest y migrations pertenecen a:

ESP-0001/11 — Identity Persistence & Data Model

58. AI Integration

Los AI Agents pueden:

detectar anomalías;
recomendar step-up;
generar escenarios de prueba;
analizar eventos;
recomendar políticas.

Pero no pueden:

Disable MFA
Reveal MFA secrets
Reveal Recovery Codes
Approve MFA bypass
Reduce Criticality
Override Freshness
Manipulate MFA Cache
Approve Security Regression
Activate Security Policy

AI utiliza el mismo Authorization Runtime.

59. AI Risk Detection

Un AI/Risk Engine puede proporcionar:

risk_score
risk_level
confidence
source
timestamp

Pero:

Risk Engine
    ≠
Authorization Engine

El Risk Engine proporciona contexto.

La decisión final continúa perteneciendo al modelo de autorización.

60. Failure Model

MFA debe fallar cerrado.

Casos:

Unknown Factor
Unknown Challenge
Expired Challenge
Invalid OTP
Exceeded Attempts
Revoked Factor
Revoked Device
Provider Failure
Database Uncertainty
Tenant Uncertainty
Security State Uncertainty

No deben producir:

ALLOW

cuando MFA sea requerido.

61. Security Invariants

Se consolidan como mínimo:

INV-MFA-001
Inactive Factor Cannot Authenticate

INV-MFA-002
Revoked Factor Cannot Authenticate

INV-MFA-003
Expired Challenge Cannot Verify

INV-MFA-004
Verified Challenge Cannot Be Reused

INV-MFA-005
Recovery Code Is Single Use

INV-MFA-006
Revoked Recovery Code Cannot Be Used

INV-MFA-007
MFA Disable Requires Required Security Assurance

INV-MFA-008
Tenant Boundary Cannot Be Crossed

INV-MFA-009
Provider Failure Cannot Produce ALLOW

INV-MFA-010
MFA Secret Cannot Appear in Logs or Events

INV-MFA-011
MFA State Change Invalidates Required Cached Security State

INV-MFA-012
AI Cannot Reduce MFA Security

INV-MFA-013
MFA Evidence Has Bounded Freshness

INV-MFA-014
Concurrent Challenge Verification Is Atomic

INV-MFA-015
MFA Does Not Grant Authorization

INV-MFA-016
Unknown MFA State Cannot Produce Unsafe ALLOW
62. Testing Strategy
Factor Tests
Create
Enroll
Verify
Activate
Revoke
Replace
Expire
Challenge Tests
Create
Verify
Fail
Expire
Cancel
Replay
Concurrent verification
Attempt exhaustion
Recovery
Generate
Use
Reuse
Revoke
Regenerate
Administrative recovery
Trusted Devices
Register
Use
Expire
Revoke
Cross-user attempt
Cross-tenant attempt
Security
MFA bypass
Tenant isolation
Provider failure
Cache stale ALLOW
Security epoch
Step-up freshness
63. Golden Security Scenarios

Este ESP debe integrarse con el catálogo definido en ADR-062/063.

Ejemplos:

GSC-MFA-001
Active MFA factor + valid challenge → success

GSC-MFA-002
Revoked factor → DENY

GSC-MFA-003
Expired challenge → DENY

GSC-MFA-004
Reused challenge → DENY

GSC-MFA-005
Reused recovery code → DENY

GSC-MFA-006
MFA required + no evidence → REQUIRE_MFA

GSC-MFA-007
MFA required + stale evidence → REQUIRE_MFA

GSC-MFA-008
Cross-tenant MFA administration → DENY

GSC-MFA-009
MFA provider failure → no ALLOW

GSC-MFA-010
AI request to disable MFA → DENY
64. Integration with Test Runtime

MFA debe utilizar los contratos definidos en ADR-067:

Request Contract
Authorization Context
Policy Evaluation
Criticality Result
Authorization Decision
Freshness
Cache Eligibility
Test Execution
Evidence
Error Contract

El Test Runtime no sustituye al MFA Runtime.

Ambos deben compartir los mismos contratos normativos de seguridad.

65. Observability

Métricas conceptuales:

mfa_challenges_created
mfa_challenges_verified
mfa_challenges_failed
mfa_challenges_expired
mfa_factor_activations
mfa_factor_revocations
mfa_recovery_attempts
mfa_recovery_failures
mfa_step_up_requests
mfa_step_up_success
mfa_step_up_failure
mfa_provider_errors

Nunca se deben registrar:

OTP
TOTP Secret
Recovery Code
Private Key
66. Traceability
Área	Fuente
Identity Domain	ESP-0001/01
Authentication	ESP-0001/03
Sessions	ESP-0001/06
MFA Factors	ADR-023
MFA Challenges	ADR-024
Challenge TTL	ADR-024
Attempt Control	ADR-024
Recovery	ADR-024 / ADR-035
Cryptography	ADR-008 / Security
Authorization	ADR-045–049
Policy	ADR-046–048
Criticality	ADR-058–059
Freshness	ADR-056–057
Cache	ADR-050–056
Account Security	ADR-033–035
API	ESP-0001/10
Persistence	ESP-0001/11
Events	ESP-0001/12
Testing	ESP-0001/13
Runtime	ESP-0001/15
Test Catalog	ADR-062–067
Module Boundaries	ADR-068–069
67. Implementation Readiness
Área	Estado
MFA abstraction	🟢
MFAFactor	🟢
MFAChallenge	🟢
TOTP	🟢
WebAuthn/Passkey	🟢
OTP factors	🟢
Enrollment	🟢
Challenge lifecycle	🟢
TTL/attempt model	🟢
Replay prevention	🟢
Concurrent verification	🟢
Recovery Codes	🟢
Trusted Devices	🟢
Step-Up	🟢
Assurance	🟢
Revocation	🟢
Tenant isolation	🟢
Criticality	🟢
Freshness	🟢
Cache invalidation	🟢
Audit	🟢
Security Events	🟢
AI Security	🟢
API implementation	⏳ ESP-0001/10
Persistence implementation	⏳ ESP-0001/11
Event contracts	⏳ ESP-0001/12
Test implementation	⏳ ESP-0001/13
Deployment/runtime	⏳ ESP-0001/15
68. Qué cambió respecto al ESP-0001/07 original

El documento original ya tenía una buena base: factores, challenges, recovery codes, trusted devices, enrollment, login MFA, step-up, revocación y recuperación.

La consolidación agrega una diferencia importante:

Antes

Había varias formulaciones del tipo:

propuesta
debería
podría
ADR futuro
decisión posterior
Ahora

Las decisiones fundamentales quedan integradas como arquitectura normativa:

MFAFactor
MFAChallenge
RecoveryCode
TrustedDevice

Challenge expiration
Attempt limits
Replay prevention
Concurrent verification

Step-up
MFA assurance
Security policy
Criticality
Freshness
Cache invalidation

Tenant isolation
Audit
Security events
AI restrictions
Fail closed

Especialmente, ADR-023 y ADR-024 dejan de ser decisiones externas que todavía hay que tomar y pasan a formar parte explícita de esta ESP.

69. Decisiones que NO debemos inventar aquí

Todavía no corresponde fijar arbitrariamente:

TTL exacto de challenges;
número máximo de intentos;
TTL de trusted devices;
TTL de step-up evidence;
parámetros criptográficos exactos;
catálogo definitivo de proveedores;
SMS obligatorio/no obligatorio para producción;
catálogo definitivo de assurance por factor;
schemas JSON finales;
tablas SQL definitivas;
endpoints definitivos.

Eso se resolverá en las especificaciones correspondientes.

70. Resultado
ESP-0001/07 original

🟡 Draft — buena cobertura conceptual, pero parcialmente desactualizado frente a los ADR posteriores.

ESP-0001/07 v1.1.0

🟢 Arquitectónicamente alineado con el modelo actual de EVOXA Identity.

Y, al igual que hicimos con /06, no necesitamos crear otro ADR sólo para completar MFA.

El bloque Identity queda ahora mucho más coherente:

ESP-0001/01
Identity Domain Model
        │
        ▼
ESP-0001/03
Authentication & Authorization
        │
        ▼
ESP-0001/05
Roles & Permissions
        │
        ▼
ESP-0001/06
Sessions & Token Management
        │
        ▼
ESP-0001/07
Multi-Factor Authentication
        │
        ▼
ESP-0001/08
Security Policies

El siguiente paso lógico es ESP-0001/08 — Identity Security Policies, y aquí sí tendremos que hacer una consolidación especialmente profunda porque el ESP original quedó bastante atrás respecto de ADR-045 → ADR-059, incluyendo Policy Model, DSL, Evaluation Runtime, Criticality, Risk, Freshness y Cache Security.
