IS-IDENTITY-010 — Multi-Factor Authentication

Estado: Draft
Prioridad: High
Dominio: Identity
Capability: Multi-Factor Authentication (MFA)
Dependencias: IS-IDENTITY-001 → IS-IDENTITY-009
Siguiente historia: IS-IDENTITY-011 — Security Policy Engine

1. Propósito

Definir el modelo funcional y técnico de Multi-Factor Authentication (MFA) para EVOXA, permitiendo elevar el nivel de seguridad de una autenticación mediante factores adicionales a las credenciales primarias.

MFA debe integrarse con:

Authentication.
Session Management.
Refresh Token Lifecycle.
Security Policy Engine.
Authorization.
Audit.
Security Events.

El objetivo no es crear un sistema de autenticación paralelo, sino incorporar MFA como una capacidad transversal del dominio Identity.

El Blueprint establece MFA como una capability de Identity y contempla entidades como MFAFactor, MFAChallenge, RecoveryCode y TrustedDevice; esta IS desarrolla el comportamiento necesario para esas piezas sin sustituir las decisiones que correspondan a los ADR posteriores.

2. Business Goal

Como plataforma EVOXA, necesitamos poder exigir una segunda o adicional evidencia de identidad cuando:

La política de seguridad lo determine.
El recurso sea sensible.
El riesgo de autenticación sea elevado.
El usuario configure MFA.
Se requiera una autenticación reforzada.

El flujo conceptual será:

Primary Authentication
        ↓
Security Policy
        ↓
MFA Required?
    ↙        ↘
  NO          YES
  ↓            ↓
Session      MFA Challenge
               ↓
          Verify Factor
               ↓
          MFA Verified
               ↓
             Session
3. Persona
Primary Personas
Coach
Nutritionist
Physiotherapist
Team Manager
Admin
Security Personas
Security Administrator
Organization Administrator
System Persona
Identity Service
4. User Journey
4.1 MFA no requerido
User
 ↓
Credentials
 ↓
Authentication
 ↓
Policy
 ↓
ALLOW
 ↓
Session
 ↓
Access Token
4.2 MFA requerido
User
 ↓
Credentials
 ↓
Authentication
 ↓
Policy
 ↓
REQUIRE_MFA
 ↓
MFA Challenge
 ↓
User provides factor
 ↓
Verify
 ↓
MFA VERIFIED
 ↓
Session
 ↓
Access Token
4.3 MFA requerido durante una operación sensible

MFA no debe limitarse necesariamente al login.

El modelo de autorización de EVOXA contempla decisiones como:

CHALLENGE
REAUTHENTICATE
REQUIRE_MFA

Por lo tanto:

Authenticated Session
        ↓
Sensitive Operation
        ↓
Authorization
        ↓
REQUIRE_MFA
        ↓
MFA Challenge
        ↓
Verified
        ↓
Continue Operation

Esto permite utilizar MFA como mecanismo de step-up authentication.

5. Feature Pack

Esta IS cubre:

MFA Core
MFA Factor.
MFA Challenge.
MFA verification.
MFA enrollment.
MFA activation.
MFA removal.
Recovery
Recovery Codes.
Recovery flow.
Revocation de recovery codes.
Trusted Devices
Registro de dispositivo confiable.
Validación.
Revocación.
Security
MFA required.
MFA verified.
Step-up authentication.
Rate limiting.
Anti-brute-force.
Audit.
Security Events.
6. Scope
In Scope
MFAFactor.
MFAChallenge.
RecoveryCode.
TrustedDevice.
Enrollment.
Verification.
Challenge lifecycle.
Factor lifecycle.
Recovery.
Step-up authentication.
Integration con Session.
Integration con Authentication.
Integration con Security Policy.
Audit.
Security events.
Observability.
Persistence.
Out of Scope

Esta IS no define completamente:

Password Authentication.
JWT.
Refresh Token.
RBAC.
ABAC.
Security Policy Engine completo.
OAuth/OIDC completo.
Servicio externo específico de SMS/email.
Un proveedor concreto de MFA.

Los mecanismos concretos de factor deben permanecer desacoplados del dominio.

7. MFA Conceptual Model

La arquitectura contempla:

User
 │
 ├── MFAFactor
 │      ├── Factor A
 │      ├── Factor B
 │      └── Factor N
 │
 ├── MFAChallenge
 │
 ├── RecoveryCode
 │
 └── TrustedDevice
8. MFAFactor

Entidad conceptual:

MFAFactor
├── id
├── user_id
├── type
├── status
├── created_at
├── activated_at
├── last_used_at
├── revoked_at
└── metadata

El secreto del factor no debe exponerse ni almacenarse en plaintext en logs, eventos o respuestas API.

9. MFA Factor Types

El modelo debe permitir distintos mecanismos.

Conceptualmente:

TOTP
WEBAUTHN
PASSKEY
SMS
EMAIL
PUSH
HARDWARE_KEY

Sin embargo:

La lista anterior representa mecanismos posibles y no debe interpretarse como catálogo normativo definitivo del Blueprint.

El catálogo final debe establecerse mediante decisión arquitectónica.

10. Factor Lifecycle

Se propone:

PENDING
   ↓
ACTIVE
   ↓
REVOKED
PENDING

Factor registrado pero aún no verificado.

ACTIVE

Factor confirmado y disponible para autenticación.

REVOKED

Factor invalidado y no utilizable.

11. MFA Enrollment

El enrollment debe seguir:

Authenticated User
       ↓
Request MFA Enrollment
       ↓
Security Policy
       ↓
Create Factor PENDING
       ↓
Challenge
       ↓
Verify
       ↓
Factor ACTIVE

La activación de un factor no debe ocurrir únicamente por haber iniciado el proceso de registro.

Debe existir una prueba de posesión/control del factor.

12. Enrollment Security

El enrollment de MFA es una operación sensible.

Dependiendo de la política, puede requerir:

Current Authentication
       +
Reauthentication
       +
Existing MFA

Por ejemplo, agregar un nuevo factor de autenticación no debería ser trivial para una sesión comprometida.

La política exacta será definida en IS-IDENTITY-011.

13. MFAChallenge

Entidad conceptual:

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
└── metadata
14. Challenge Purposes

Un challenge puede utilizarse para:

LOGIN
STEP_UP
ENROLLMENT
FACTOR_CHANGE
SENSITIVE_OPERATION
RECOVERY

El catálogo final puede ser refinado.

15. Challenge Lifecycle
CREATED
   ↓
PENDING
   ↓
VERIFIED

o:

PENDING
  ├──→ VERIFIED
  ├──→ EXPIRED
  ├──→ FAILED
  └──→ REVOKED
16. Challenge Rules
Created

Challenge generado correctamente.

Pending

Esperando la respuesta del usuario.

Verified

Factor verificado correctamente.

Failed

La verificación falló.

Expired

El challenge superó su TTL.

Revoked

El challenge fue invalidado por seguridad o por otra operación.

17. Challenge TTL

Los challenges deben tener una duración limitada.

Conceptualmente:

created_at
     +
challenge_ttl
     =
expires_at

El TTL exacto no debe inventarse en esta IS.

Debe convertirse en configuración/policy.

18. Verification

Flujo:

Challenge
   ↓
Validate status
   ↓
Validate expiration
   ↓
Validate factor
   ↓
Verify response
   ↓
Update challenge
   ↓
MFA Verified

La verificación debe ser:

Atómica.
Rate limited.
Auditada.
Observable.
19. Failed Attempts

Debe existir protección frente a:

Brute force.
Repeated guessing.
Automated attacks.

El challenge debe mantener información suficiente para detectar:

attempt_count

y permitir que Security Policy determine cuándo:

CHALLENGE
      ↓
FAILED
      ↓
BLOCK / REVOKE / REQUIRE_REAUTH

Los thresholds exactos deben ser configurables.

20. MFA Verified Context

Una vez verificado MFA, el sistema debe poder incorporar el resultado al Authorization Context.

Conceptualmente:

AuthorizationContext
├── authentication_method
├── mfa_status
├── mfa_verified_at
├── factor_type
└── challenge_id

Esto permite que una política determine:

MFA verified recently?

antes de permitir una operación sensible.

21. Step-Up Authentication

EVOXA debe soportar:

Normal Session
      ↓
Sensitive Resource
      ↓
Authorization
      ↓
REQUIRE_MFA
      ↓
Challenge
      ↓
Verified
      ↓
Authorization re-evaluation
      ↓
ALLOW

Es importante que la verificación MFA no otorgue permisos por sí misma.

MFA ≠ Authorization

MFA demuestra un nivel de autenticación.

Authorization determina si la operación está permitida.

22. Session Integration

Cuando MFA se verifica durante login:

MFA VERIFIED
      ↓
Session ACTIVE

Cuando MFA se verifica durante step-up:

Session ACTIVE
      ↓
MFA challenge
      ↓
MFA VERIFIED
      ↓
Session authentication context updated
23. Refresh Token Integration

Un Refresh Token no debe ignorar el estado de seguridad de MFA.

Si una Security Policy determina que una nueva autenticación reforzada es necesaria:

Refresh
 ↓
Policy
 ↓
REQUIRE_MFA
 ↓
MFA Challenge

El Refresh Token Lifecycle sigue siendo responsabilidad de IS-009.

24. Recovery Codes

El sistema debe permitir códigos de recuperación.

Conceptualmente:

User
 └── Recovery Codes
      ├── Code 1
      ├── Code 2
      ├── Code 3
      └── ...

Cada código debe ser:

De un solo uso.
Almacenado de forma segura.
Invalidable.
Auditable sin revelar su valor.
25. Recovery Code Lifecycle
AVAILABLE
    ↓
USED

o:

AVAILABLE
    ↓
REVOKED

Un código utilizado no puede reutilizarse.

26. Recovery Security

El recovery MFA es una operación de alto riesgo.

Debe estar protegido contra:

Enumeración.
Brute force.
Replay.
Credential stuffing.
Automated attempts.

Un intento de recuperación sospechoso debe generar un evento de seguridad.

27. Trusted Devices

El Blueprint contempla TrustedDevice.

Su objetivo es permitir que un usuario pueda registrar un dispositivo que haya completado correctamente una autenticación reforzada.

Conceptualmente:

MFA Verified
      ↓
Trust Device
      ↓
TrustedDevice
28. TrustedDevice Security

Un Trusted Device:

NO debe equivaler a una contraseña permanente.

Debe tener:

Identidad propia.
Estado.
Expiración.
Revocación.
Auditabilidad.

Conceptualmente:

ACTIVE
   ↓
EXPIRED
   ↓
REVOKED

La implementación concreta de device binding queda pendiente de decisión arquitectónica.

29. MFA Factor Removal

Eliminar un factor es una operación sensible.

Flujo:

Request Remove Factor
       ↓
Security Policy
       ↓
Reauthentication / MFA
       ↓
Revoke Factor
       ↓
Audit
       ↓
Security Event

No debe permitirse una eliminación de factor basada exclusivamente en una sesión posiblemente comprometida si la política exige una autenticación reforzada.

30. Last MFA Factor

La plataforma debe evitar dejar accidentalmente al usuario sin mecanismo de recuperación cuando la política exige MFA.

Ejemplo:

User
 └── Factor A

Si se solicita eliminar Factor A:

Remove Factor A
      ↓
No other factor
      ↓
Security Policy
      ↓
DENY / REQUIRE RECOVERY

La regla exacta debe ser definida por Security Policy.

31. Application Use Cases

Como mínimo:

EnrollMFAFactor
VerifyMFAFactor
CreateMFAChallenge
VerifyMFAChallenge
RevokeMFAFactor
ListMFAFactors
GenerateRecoveryCodes
UseRecoveryCode
RevokeRecoveryCodes
RegisterTrustedDevice
RevokeTrustedDevice
ListTrustedDevices

También:

RequireMFA
ValidateMFAContext

podrán existir como servicios internos.

32. Domain Services
MFAService

Responsable de:

Enrollment.
Factor lifecycle.
Challenge.
Verification.
Recovery.
MFAChallengeService

Responsable de:

Crear challenge.
Validar challenge.
Expirar.
Verificar.
Controlar intentos.
TrustedDeviceService

Responsable de:

Registrar.
Validar.
Expirar.
Revocar dispositivos.
33. Repository Contracts

Deben existir contratos conceptuales:

MFAFactorRepository
MFAChallengeRepository
RecoveryCodeRepository
TrustedDeviceRepository

Operaciones:

create()
get_by_id()
list_by_user()
update()
revoke()

y las operaciones específicas de challenge/recovery necesarias.

34. API Conceptual

Los contratos definitivos pertenecerán a IS-IDENTITY-014.

List Factors
GET /api/v1/identity/mfa/factors
Enroll
POST /api/v1/identity/mfa/factors
Verify
POST /api/v1/identity/mfa/challenges/{challenge_id}/verify
Revoke Factor
POST /api/v1/identity/mfa/factors/{factor_id}/revoke
Create Challenge
POST /api/v1/identity/mfa/challenges
Recovery Codes
POST /api/v1/identity/mfa/recovery-codes
Trusted Devices
GET /api/v1/identity/mfa/trusted-devices

Los endpoints son conceptuales, no contratos finales.

35. Events

Debe existir soporte para eventos como:

MFAFactorEnrolled
MFAFactorActivated
MFAFactorRevoked
MFAChallengeCreated
MFAChallengeVerified
MFAChallengeFailed
MFAChallengeExpired
RecoveryCodesGenerated
RecoveryCodeUsed
RecoveryCodesRevoked
TrustedDeviceRegistered
TrustedDeviceRevoked
36. Security Events

Especial atención a:

MFAChallengeFailed
MFAReuseAttempt
MFARecoveryAttempt
MFAFactorChanged
TrustedDeviceRegistered
TrustedDeviceRevoked

Los nombres finales pueden normalizarse en el catálogo global de eventos.

37. Audit

Debe registrarse:

Enrollment.
Activation.
Verification.
Failed attempts.
Revocation.
Recovery.
Trusted device registration.
Trusted device revocation.
Step-up authentication.

Nunca registrar:

TOTP secret
Recovery code plaintext
OTP value
Private key
Authentication secret
38. Security Rules
BR-010-001 — MFA Verification

Un factor solo puede considerarse válido después de una verificación exitosa.

BR-010-002 — Challenge Expiration

Un challenge expirado no puede verificarse.

BR-010-003 — Single-use Challenge

Un challenge completado no puede reutilizarse.

BR-010-004 — Rate Limiting

Los intentos de MFA deben estar limitados.

BR-010-005 — Recovery Code Single Use

Un recovery code utilizado queda invalidado.

BR-010-006 — Factor Revocation

Un factor revocado no puede utilizarse.

BR-010-007 — Tenant Isolation

MFA debe permanecer asociado al contexto de identidad correspondiente y no permitir cross-tenant escalation.

BR-010-008 — No Secret Leakage

Secretos MFA nunca deben aparecer en:

API responses.
Logs.
Events.
Audit.
BR-010-009 — Step-Up

Una solicitud REQUIRE_MFA no puede considerarse satisfecha sin una verificación MFA válida.

BR-010-010 — Authorization Separation

MFA verification no concede permisos adicionales.

BR-010-011 — Recovery Protection

Recovery debe tener controles equivalentes de seguridad contra abuso.

39. Acceptance Criteria
AC-010-01 — Enrollment

Given un usuario autenticado
When solicita registrar MFA
Then debe crearse un factor PENDING.

AC-010-02 — Activation

Given un factor PENDING
When el challenge de enrollment es verificado correctamente
Then el factor debe pasar a ACTIVE.

AC-010-03 — Invalid Challenge

Given un challenge expirado
When se intenta verificar
Then la operación debe rechazarse.

AC-010-04 — Successful MFA

Given un challenge válido
When el usuario proporciona una respuesta válida
Then el challenge debe quedar VERIFIED.

AC-010-05 — Failed MFA

Given un challenge válido
When la respuesta es incorrecta
Then debe registrarse un intento fallido y aplicarse rate limiting/policy.

AC-010-06 — Step-Up

Given una operación que requiere MFA
When Authorization devuelve REQUIRE_MFA
Then debe iniciarse un MFA Challenge.

AC-010-07 — Step-Up Completion

Given una operación pendiente de MFA
When el challenge es verificado correctamente
Then el contexto de autenticación debe reflejar MFA verificado y la autorización debe reevaluarse.

AC-010-08 — Factor Revocation

Given un factor activo
When se revoca
Then no debe poder utilizarse posteriormente.

AC-010-09 — Recovery Code

Given un recovery code disponible
When se utiliza correctamente
Then debe quedar USED y no poder reutilizarse.

AC-010-10 — Trusted Device

Given una autenticación MFA válida
When el usuario registra el dispositivo
Then debe crearse un Trusted Device según la política aplicable.

AC-010-11 — Suspicious Recovery

Given múltiples intentos fallidos de recovery
When se supera el límite configurado
Then deben aplicarse las medidas de seguridad correspondientes.

AC-010-12 — Secret Protection

Given cualquier operación MFA
When se generan logs/events/audit
Then no deben contener secretos MFA.

40. Technical Tasks
Domain

TSK-010-01 — Crear MFAFactor.

TSK-010-02 — Crear MFAChallenge.

TSK-010-03 — Crear RecoveryCode.

TSK-010-04 — Crear TrustedDevice.

TSK-010-05 — Definir Value Objects correspondientes.

TSK-010-06 — Definir lifecycle de factores.

TSK-010-07 — Definir lifecycle de challenges.

TSK-010-08 — Definir lifecycle de recovery codes.

TSK-010-09 — Definir lifecycle de trusted devices.

Application

TSK-010-10 — Implementar enrollment.

TSK-010-11 — Implementar activation.

TSK-010-12 — Implementar challenge creation.

TSK-010-13 — Implementar challenge verification.

TSK-010-14 — Implementar factor revocation.

TSK-010-15 — Implementar recovery codes.

TSK-010-16 — Implementar trusted devices.

TSK-010-17 — Implementar MFA context validation.

TSK-010-18 — Implementar step-up authentication.

Infrastructure

TSK-010-19 — Crear repositories.

TSK-010-20 — Crear persistence models.

TSK-010-21 — Crear índices.

TSK-010-22 — Implementar almacenamiento seguro de secretos.

TSK-010-23 — Implementar mecanismos de expiration.

Security

TSK-010-24 — Implementar rate limiting.

TSK-010-25 — Implementar brute-force protection.

TSK-010-26 — Implementar challenge replay protection.

TSK-010-27 — Implementar recovery abuse protection.

TSK-010-28 — Integrar con Security Policy Engine.

Integration

TSK-010-29 — Integrar con Authentication.

TSK-010-30 — Integrar con Session.

TSK-010-31 — Integrar con Refresh Token.

TSK-010-32 — Integrar con Authorization.

TSK-010-33 — Integrar con User lifecycle.

Events & Audit

TSK-010-34 — Implementar MFA domain events.

TSK-010-35 — Implementar security events.

TSK-010-36 — Integrar AuditService.

Observability

TSK-010-37 — Implementar métricas.

TSK-010-38 — Implementar logging seguro.

TSK-010-39 — Implementar tracing.

Testing

TSK-010-40 — Unit tests.

TSK-010-41 — Integration tests.

TSK-010-42 — API tests.

TSK-010-43 — Security tests.

TSK-010-44 — Rate-limit tests.

TSK-010-45 — Recovery tests.

TSK-010-46 — Step-up tests.

TSK-010-47 — Concurrency tests.

TSK-010-48 — Cross-tenant tests.

41. Testing Strategy
Unit Tests

Cubrir:

Factor lifecycle.
Challenge lifecycle.
Verification.
Expiration.
Recovery.
Trusted Device.
Integration Tests

Validar:

User
Session
MFAFactor
MFAChallenge
RecoveryCode
TrustedDevice
Security Tests

Especialmente:

Brute force.
Replay.
Challenge reuse.
Recovery abuse.
Factor removal.
Trusted device abuse.
Cross-tenant access.
Privilege escalation.
Authorization Tests

Validar:

REQUIRE_MFA
     ↓
Challenge
     ↓
Verified
     ↓
Authorization Re-evaluation
42. Observability
Metrics
identity.mfa.enrollment
identity.mfa.activation
identity.mfa.challenge.created
identity.mfa.challenge.success
identity.mfa.challenge.failure
identity.mfa.challenge.expired
identity.mfa.factor.revoked
identity.mfa.recovery.used
identity.mfa.recovery.failed
identity.mfa.trusted_device.created
identity.mfa.trusted_device.revoked
Security Metrics

Debe ser posible detectar tendencias como:

high MFA failure rate
high recovery usage
unusual device enrollment
repeated factor changes
43. Failure Modes
MFA Provider Failure

Si un proveedor externo participa en un factor:

Provider unavailable
       ↓
No verification
       ↓
No MFA bypass

Nunca se debe interpretar una falla del proveedor como MFA exitoso.

Database Failure

No debe aceptarse MFA basándose en estado inconsistente.

Challenge Replay

Debe rechazarse.

Expired Challenge

Debe rechazarse.

Concurrent Verification

Dos verificaciones concurrentes del mismo challenge deben terminar en un estado consistente.

44. Performance

Las operaciones MFA deben diseñarse para cumplir los objetivos generales de Identity.

Particularmente:

Challenge creation.
Challenge validation.
Factor lookup.
MFA verification.

Las dependencias externas deben manejar:

timeout;
retry controlado;
circuit breaking cuando corresponda;
observabilidad.

No se debe sacrificar seguridad por latencia.

45. Definition of Done
 MFAFactor implementado.
 MFAChallenge implementado.
 RecoveryCode implementado.
 TrustedDevice implementado.
 Factor lifecycle implementado.
 Challenge lifecycle implementado.
 Enrollment implementado.
 Verification implementada.
 Factor revocation implementada.
 Recovery implementado.
 Step-up authentication implementado.
 Session integration implementada.
 Authentication integration implementada.
 Authorization integration implementada.
 Refresh Token integration implementada.
 Security Policy integration implementada.
 Rate limiting implementado.
 Replay protection implementado.
 PostgreSQL persistence implementada.
 Secret storage seguro implementado.
 Events implementados.
 Audit implementado.
 Metrics implementadas.
 Logs seguros implementados.
 Tracing implementado.
 Unit tests implementados.
 Integration tests implementados.
 Security tests implementados.
 Recovery tests implementados.
 Step-up tests implementados.
 Concurrency tests implementados.
 Cross-tenant tests implementados.
 No MFA secrets aparecen en logs/events/API.
 No existen MFA bypasses.
 Documentación actualizada.
46. Traceability
Elemento	Referencia
Blueprint	Identity / Multi-Factor Authentication
ESP	ESP-0001 — Identity
ADR	ADR-IDENTITY-005
ADR	ADR-IDENTITY-006
ADR	ADR-IDENTITY-007
ADR	ADR-IDENTITY-008
ADR	ADR-IDENTITY-009
ADR	ADR-IDENTITY-010
Previous IS	IS-IDENTITY-005 — Authentication
Previous IS	IS-IDENTITY-006 — Authorization
Previous IS	IS-IDENTITY-008 — Session
Previous IS	IS-IDENTITY-009 — Refresh Token
Next IS	IS-IDENTITY-011 — Security Policy Engine
API	IS-IDENTITY-014
Persistence	IS-IDENTITY-015
Testing	IS-IDENTITY-016
Runtime	IS-IDENTITY-017

La trazabilidad mantiene la cadena ESP → IS → Technical Tasks → Source Code, que el Blueprint establece como requisito del lifecycle de implementación.

47. Architecture Model
                         ┌──────────────────────┐
                         │       Client         │
                         │     Web / Mobile     │
                         └──────────┬───────────┘
                                    │
                                    ↓
                         ┌──────────────────────┐
                         │    Identity API      │
                         └──────────┬───────────┘
                                    ↓
                         ┌──────────────────────┐
                         │    Application       │
                         │                      │
                         │ MFAService           │
                         │ ChallengeService     │
                         │ TrustedDeviceService │
                         └──────────┬───────────┘
                                    ↓
                         ┌──────────────────────┐
                         │       Domain         │
                         │                      │
                         │ MFAFactor            │
                         │ MFAChallenge          │
                         │ RecoveryCode          │
                         │ TrustedDevice         │
                         └──────────┬───────────┘
                                    ↓
              ┌─────────────────────┼──────────────────────┐
              ↓                     ↓                      ↓
       ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
       │ PostgreSQL   │      │    Redis     │      │ Event /      │
       │ Source       │      │ Rate Limit / │      │ Outbox       │
       │ of Truth     │      │ Temporary    │      │ Platform     │
       └──────────────┘      └──────────────┘      └──────────────┘
48. Identity Security Flow

Con las historias anteriores, Identity comienza a formar este flujo:

                   USER
                    │
                    ↓
              AUTHENTICATION
                    │
                    ↓
              SECURITY POLICY
                    │
          ┌─────────┴─────────┐
          │                   │
        ALLOW              REQUIRE_MFA
          │                   │
          │             MFA CHALLENGE
          │                   │
          │              MFA VERIFIED
          │                   │
          └─────────┬─────────┘
                    ↓
                  SESSION
                    │
                    ↓
             REFRESH TOKEN
                    │
                    ↓
               AUTHORIZATION
                    │
                    ↓
             RBAC + ABAC +
           TENANT + OWNERSHIP
                    │
                    ↓
              ALLOW / DENY

Esto es importante porque MFA no reemplaza ninguno de los mecanismos anteriores: aumenta el nivel de confianza de la autenticación y proporciona contexto para las decisiones de seguridad.

49. Architectural Decisions Pending

Antes de pasar esta IS a Approved, deberán definirse mediante ADR/configuración:

Catálogo definitivo de factores MFA.
Si TOTP será obligatorio como primer mecanismo.
Soporte WebAuthn/Passkeys.
Soporte SMS.
Soporte Email OTP.
TTL de MFA Challenge.
Número máximo de intentos.
Cooldown/lockout.
Política de Recovery Codes.
Cantidad de Recovery Codes.
Política de Trusted Devices.
Trusted Device TTL.
Device binding.
Reauthentication requirements.
Step-up MFA validity window.
MFA requirements por operación.
MFA requirements por rol.
MFA requirements por riesgo.
Política ante pérdida del segundo factor.
Integración con proveedores externos.

Estos puntos no deben cerrarse arbitrariamente porque varios dependen del futuro Security Policy Engine.

50. Dependency Flow

La arquitectura de Identity queda ahora:

001 Identity Foundation
        ↓
002 User
        ↓
003 Organization
        ↓
004 Membership
        ↓
005 Authentication
        ↓
006 Authorization
        ↓
007 Roles & Permissions
        ↓
008 Session
        ↓
009 Refresh Token
        ↓
010 MFA
        ↓
011 Security Policy
        ↓
012 Audit & Security Events
        ↓
013 Domain Events & Outbox
        ↓
014 API Contracts
        ↓
015 Persistence
        ↓
016 Testing & Quality
        ↓
017 Runtime & Deployment
Estado

IS-IDENTITY-010 — Multi-Factor Authentication queda definida como Draft — High.

La pieza clave que queda preparada es:

Authentication → MFA → Session → Refresh Token → Authorization → Step-Up Authentication

y el siguiente paso lógico es IS-IDENTITY-011 — Security Policy Engine, que será especialmente importante porque deberá decidir cuándo EVOXA permite, deniega, desafía, exige MFA, exige reautenticación o revoca una operación/sesión.
