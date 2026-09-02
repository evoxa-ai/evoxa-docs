ESP-0001 / 07 — Identity Multi-Factor Authentication (MFA)

Estado: Draft
Dominio: Identity
Código: ESP-0001
Sección: 07 — Multi-Factor Authentication
Parent Specification: ESP-0001 — Identity Engineering Specification

1. Propósito

Este documento define la especificación técnica de Multi-Factor Authentication (MFA) para EVOXA Identity.

MFA añade una segunda capa de verificación a la autenticación de usuarios, reduciendo el riesgo asociado al compromiso de contraseñas, credenciales, sesiones o dispositivos.

La capacidad MFA pertenece al dominio Identity y debe integrarse con:

Authentication
Sessions
Token Management
Authorization
Security Policies
Audit
Event Platform
API Gateway
Administración de usuarios y organizaciones

La arquitectura de EVOXA establece MFA como una capacidad de Identity.

2. Objetivos

MFA debe permitir:

Registrar factores adicionales de autenticación.
Verificar factores durante el login.
Requerir MFA según políticas.
Ejecutar step-up authentication para operaciones sensibles.
Administrar dispositivos confiables.
Permitir mecanismos de recuperación.
Generar códigos de recuperación.
Revocar factores comprometidos.
Auditar todas las operaciones relacionadas con MFA.
Integrarse con sesiones y tokens.
Mantener aislamiento entre tenants.
Permitir futuras extensiones de factores sin rediseñar Identity.
3. Principios

MFA debe respetar los principios generales de EVOXA:

Zero Trust
Least Privilege
Security by Design
Privacy by Design
Secure by Default
Defense in Depth
Fail Closed
Auditability
Tenant Isolation

Estos principios están alineados con la arquitectura de seguridad definida para EVOXA.

4. Modelo conceptual

MFA introduce los siguientes conceptos:

User
 │
 ├── MFA Configuration
 │
 ├── MFA Factors
 │      ├── TOTP
 │      ├── WebAuthn / Passkey
 │      ├── Email OTP
 │      ├── SMS OTP
 │      └── Future Factors
 │
 ├── Recovery Codes
 │
 ├── Trusted Devices
 │
 └── MFA Challenges

El usuario continúa siendo la identidad principal.

MFA no reemplaza:

User
Session
Authentication
Authorization
Roles
Permissions

MFA complementa la autenticación.

5. Factores de autenticación

La plataforma debe diseñar MFA mediante una abstracción de factores.

5.1 TOTP

Time-based One-Time Password.

Ejemplos conceptuales:

Authenticator Apps

El usuario registra un secreto y genera códigos temporales.

Características:

Código de un solo uso.
Tiempo limitado.
No requiere conexión permanente con EVOXA.
Debe protegerse el secreto durante almacenamiento.
Debe existir protección contra brute force.
5.2 WebAuthn / Passkeys

Factor basado en criptografía de clave pública.

Modelo:

Device
   │
   ├── Private Key
   │
   └── Public Key
             │
             ▼
          EVOXA

La clave privada no debe almacenarse en EVOXA.

El servidor conserva únicamente la información necesaria para verificar la autenticación.

Este factor debe considerarse especialmente para:

Administradores
Personal interno
Organizaciones empresariales
Operaciones críticas
5.3 Email OTP

Código temporal enviado al correo electrónico registrado.

Puede utilizarse como:

MFA
recuperación
verificación adicional

Pero debe considerarse un factor de menor resistencia que WebAuthn/passkeys.

5.4 SMS OTP

Código temporal enviado mediante SMS.

La arquitectura puede soportarlo como factor, pero su disponibilidad, seguridad y costo deben quedar sujetos a una decisión posterior.

No debe asumirse como factor obligatorio del MVP.

6. Factores futuros

El diseño debe permitir agregar posteriormente:

Hardware security keys
Push authentication
Biometrics mediante WebAuthn
Enterprise identity providers
Smart cards
Otros mecanismos compatibles con estándares futuros

La incorporación de un nuevo factor no debería obligar a modificar el modelo principal de User o Session.

7. Entidades
7.1 MFAFactor

Representa un factor registrado por un usuario.

Conceptualmente:

MFAFactor
---------
id
user_id
type
status
name
created_at
verified_at
last_used_at
revoked_at
metadata
type

Ejemplos:

TOTP
WEBAUTHN
EMAIL_OTP
SMS_OTP
status
PENDING
ACTIVE
REVOKED
EXPIRED
8. MFA Challenge

Un challenge representa un intento concreto de verificar MFA.

MFAChallenge
------------
id
user_id
session_id
factor_id
type
status
created_at
expires_at
verified_at
attempt_count
metadata

Estados:

CREATED
PENDING
VERIFIED
FAILED
EXPIRED
CANCELLED

Un challenge debe tener una duración limitada.

9. Recovery Codes

Los códigos de recuperación permiten al usuario recuperar acceso cuando no tiene disponible su factor principal.

Conceptualmente:

RecoveryCode
------------
id
user_id
code_hash
status
created_at
used_at

Estados:

ACTIVE
USED
REVOKED

Los códigos deben almacenarse hasheados, no en texto plano.

Un código utilizado no puede reutilizarse.

10. Trusted Devices

EVOXA puede permitir que un usuario marque un dispositivo como confiable.

Conceptualmente:

TrustedDevice
-------------
id
user_id
device_identifier
created_at
last_used_at
expires_at
revoked_at
status

Estados:

ACTIVE
EXPIRED
REVOKED

El dispositivo confiable no debe considerarse una identidad independiente.

Su función es reducir la frecuencia de challenges MFA cuando la política lo permita.

11. Enrollment

El registro de MFA debe seguir un flujo controlado.

User
 │
 ▼
Request MFA Enrollment
 │
 ▼
Authentication
 │
 ▼
Create Factor
 │
 ▼
Verify Factor
 │
 ▼
Activate Factor
 │
 ▼
Audit Event

Un factor no debe considerarse activo simplemente porque fue creado.

Debe existir una verificación inicial.

12. Ejemplo TOTP

Flujo conceptual:

1. User requests TOTP
2. Identity generates secret
3. Secret delivered through enrollment flow
4. User configures authenticator
5. User provides OTP
6. Identity verifies OTP
7. Factor becomes ACTIVE
8. Recovery codes generated
9. Audit event emitted

El secreto TOTP debe estar protegido durante todo su ciclo de vida.

13. Login con MFA

El login debe extender el flujo definido en ESP-0001/03.

Credentials
    │
    ▼
Authenticate User
    │
    ▼
Check Account Status
    │
    ▼
Check MFA Policy
    │
 ┌──┴─────────────┐
 │                │
No MFA           MFA
 │                │
 ▼                ▼
Session        MFA Challenge
                 │
                 ▼
              Verification
                 │
                 ▼
              Session

Por tanto:

Authentication
      +
MFA Verification
      =
Authenticated Session
14. MFA Policy

Identity debe permitir políticas que determinen cuándo MFA es obligatorio.

Ejemplos conceptuales:

MFA_REQUIRED = true

o:

MFA_REQUIRED_FOR_ROLE = ADMIN

o:

MFA_REQUIRED_FOR_OPERATION = SECURITY_SENSITIVE

Las políticas pueden aplicarse a diferentes niveles:

Platform

Para administradores globales.

Organization

Para una organización determinada.

User

Configuración específica del usuario.

Operation

Para operaciones especialmente sensibles.

15. Step-Up Authentication

Una sesión ya autenticada puede necesitar una verificación MFA adicional.

Ejemplo:

User logged in
      │
      ▼
Normal session
      │
      ▼
Sensitive operation
      │
      ▼
Require MFA
      │
      ▼
MFA Challenge
      │
      ▼
Operation allowed

Ejemplos de operaciones potencialmente sensibles:

Cambiar contraseña.
Cambiar factores MFA.
Desactivar MFA.
Cambiar información crítica de seguridad.
Administrar usuarios.
Modificar permisos.
Operaciones administrativas críticas.
Acciones financieras críticas.
Cambiar configuración de seguridad.

La lista exacta debe definirse mediante políticas y ADRs.

16. MFA y Authorization

MFA no reemplaza autorización.

El flujo completo será:

Authentication
      ↓
MFA
      ↓
Session
      ↓
Tenant Context
      ↓
RBAC
      ↓
ABAC
      ↓
Ownership / Resource Policy
      ↓
ALLOW / DENY

Esto mantiene la separación definida en ESP-0001/03.

17. MFA Assurance Level

Como evolución del diseño, Identity puede mantener información sobre el nivel de autenticación alcanzado.

Conceptualmente:

Authentication Assurance
------------------------
PASSWORD
MFA
STRONG_MFA
PHISHING_RESISTANT

Esto permitiría posteriormente expresar políticas como:

Operation requires STRONG_MFA

en lugar de simplemente:

MFA = true

Esto es una propuesta derivada y no una exigencia explícita del Blueprint.

18. MFA y Sessions

La sesión debe conocer si MFA fue completado.

Conceptualmente:

Session
-------
authentication_method
mfa_verified
mfa_factor_id
authentication_time
assurance_level

Sin embargo, no se recomienda almacenar información excesiva dentro del JWT.

La arquitectura de sesiones definida anteriormente contempla un contexto de seguridad asociado a la sesión.

19. MFA y Tokens

El Access Token puede transportar únicamente claims necesarios para que otros componentes conozcan el contexto de autenticación.

Por ejemplo, conceptualmente:

amr
acr

cuando corresponda.

No debe incluir:

Secretos MFA.
Recovery codes.
TOTP secrets.
Datos sensibles del usuario.
Catálogos completos de permisos.

Esto mantiene el principio establecido en ESP-0001/06 de no convertir el JWT en un almacén de información sensible.

20. Revocación

Un factor debe poder revocarse.

Ejemplo:

User
 │
 ├── TOTP → ACTIVE
 ├── WebAuthn → ACTIVE
 └── Email → ACTIVE

Si el teléfono/dispositivo se pierde:

TOTP → REVOKED

La revocación debe:

Marcar el factor como revocado.
Invalidar challenges relacionados.
Generar evento de seguridad.
Registrar auditoría.
Aplicar las políticas de sesión correspondientes.
21. Desactivación de MFA

Desactivar MFA es una operación sensible.

Debe requerir una política de seguridad reforzada.

Flujo recomendado:

Authenticated Session
        │
        ▼
Step-Up Authentication
        │
        ▼
Verify MFA / Recovery
        │
        ▼
Disable MFA
        │
        ▼
Revoke / Update Factors
        │
        ▼
Audit
        │
        ▼
Security Event
22. Recuperación

Si el usuario pierde todos sus factores:

User
 │
 ▼
Recovery
 │
 ├── Recovery Code
 │
 ├── Verified Email
 │
 └── Administrative Recovery

El método exacto de recuperación administrativa debe diseñarse cuidadosamente.

Especialmente para:

Platform Administrators
Organization Administrators
usuarios con acceso a información sensible.
23. Protección contra ataques

MFA debe incorporar controles contra:

Brute Force

Limitar intentos.

OTP Replay

Un código utilizado no puede reutilizarse.

Challenge Replay

Un challenge verificado no puede volver a verificarse.

Challenge Expiration

Los challenges deben expirar.

Enumeration

Las respuestas no deben revelar información innecesaria.

Abuse

Aplicar rate limiting.

Credential Stuffing

MFA debe actuar como segunda barrera, no como sustituto de controles de credenciales.

Session Hijacking

Las operaciones sensibles pueden exigir step-up MFA.

24. Rate Limiting

Debe existir limitación por:

User
IP
Session
Challenge
Factor
Organization

Especialmente para:

OTP verification
MFA enrollment
Recovery
Password reset
MFA disable
Factor registration

Los límites concretos requieren definición mediante configuración/política y no deben fijarse arbitrariamente en esta especificación.

25. Eventos

MFA debe integrarse con Event Platform.

Eventos propuestos:

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

Los eventos siguen el modelo de eventos definido en EVOXA:

event_id
event_type
event_version
timestamp
tenant_id
actor
resource
payload
metadata
correlation_id
causation_id

26. Auditoría

Las operaciones MFA deben ser auditables.

Especialmente:

Enrollment
Verification
Failure
Factor revocation
MFA disable
Recovery
Trusted device
Step-up authentication
Administrative recovery

Ejemplo conceptual:

AuditLog
--------
actor
user
organization
action
resource
result
timestamp
ip
user_agent
correlation_id

La arquitectura de EVOXA exige auditabilidad para operaciones críticas y contempla retención de auditoría de largo plazo.

27. Seguridad de secretos

Los siguientes valores son extremadamente sensibles:

TOTP Secret
Recovery Codes
Private Authentication Material
OTP Secrets

Nunca deben:

aparecer en logs;
aparecer en analytics;
incluirse en eventos de negocio;
almacenarse en texto plano cuando no sea estrictamente necesario;
incluirse en JWT;
devolverse nuevamente mediante APIs normales.
28. API conceptual

La API pública de Identity podría evolucionar hacia endpoints como:

POST /api/v1/identity/mfa/factors
GET  /api/v1/identity/mfa/factors
DELETE /api/v1/identity/mfa/factors/{factor_id}

POST /api/v1/identity/mfa/challenges
POST /api/v1/identity/mfa/challenges/{challenge_id}/verify

POST /api/v1/identity/mfa/recovery-codes
POST /api/v1/identity/mfa/recovery

GET  /api/v1/identity/mfa/trusted-devices
DELETE /api/v1/identity/mfa/trusted-devices/{device_id}

Importante: estos son contratos conceptuales derivados para continuar el diseño. No deben considerarse todavía contratos API definitivos.

El Blueprint establece que las APIs deben diseñarse como contratos versionados y que la implementación debe seguir al contrato.

29. Administración

Los administradores autorizados podrán necesitar capacidades como:

View MFA status
Require MFA
Reset MFA
Revoke factor
Revoke trusted device
Initiate recovery

Pero estas operaciones deben estar sujetas a:

RBAC
+
ABAC
+
Tenant Policy
+
Audit
+
Step-Up Authentication

Un administrador de una organización no debe poder administrar MFA de usuarios pertenecientes a otro tenant.

30. Multi-Tenant

El contexto de tenant debe estar presente en todas las operaciones administrativas relacionadas con MFA.

Platform
 ├── Organization A
 │     ├── User 1
 │     └── User 2
 │
 └── Organization B
       ├── User 3
       └── User 4

Una operación de:

Reset MFA

debe validar:

Actor
 ↓
Organization
 ↓
Membership
 ↓
Permission
 ↓
Target User
 ↓
Policy
31. Observabilidad

MFA debe producir métricas técnicas y de seguridad.

Ejemplos:

mfa_enrollment_success_total
mfa_enrollment_failure_total
mfa_challenge_total
mfa_challenge_success_total
mfa_challenge_failure_total
mfa_recovery_total
mfa_factor_revocation_total
mfa_step_up_total

También:

MFA verification latency
Challenge expiration rate
Failure rate
Recovery rate

Los logs deben ser estructurados y correlacionables mediante:

trace_id
span_id
correlation_id

como establece la arquitectura de observabilidad.

32. Persistencia propuesta

Para el almacenamiento transaccional:

PostgreSQL

Para información temporal de alta frecuencia:

Redis

Conceptualmente:

PostgreSQL
 ├── MFAFactor
 ├── RecoveryCode
 └── TrustedDevice

Redis
 ├── MFAChallenge
 ├── OTP state
 ├── Rate limits
 └── Temporary security state

Esto sigue la estrategia de persistencia poliglota de EVOXA, pero la separación exacta de responsabilidades debe quedar definida posteriormente.

33. Integración con Identity

La arquitectura completa quedaría:

                    ┌──────────────┐
                    │     User     │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │Authentication│
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │     MFA      │
                    └──────┬───────┘
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
           TOTP         WebAuthn       OTP
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                    ┌──────────────┐
                    │   Session    │
                    └──────┬───────┘
                           ▼
                    ┌──────────────┐
                    │    Tokens    │
                    └──────┬───────┘
                           ▼
                    ┌──────────────┐
                    │Authorization │
                    └──────────────┘
34. Dependencias

ESP-0001/07 depende directamente de:

ESP-0001/01 Identity Domain Model
ESP-0001/02 Identity Architecture
ESP-0001/03 Authentication & Authorization
ESP-0001/04 User & Organization Management
ESP-0001/05 Roles & Permissions
ESP-0001/06 Sessions & Token Management

Y posteriormente deberá integrarse con:

ESP-0001/08 Identity Security Policies
ESP-0001/09 Identity Audit & Compliance
ESP-0001/10 Identity API Contracts

según la descomposición que adoptemos para ESP-0001.

35. ADRs necesarios

Antes de implementar MFA, recomiendo crear ADRs para las decisiones que el Blueprint no determina explícitamente.

ADR-IDENTITY-007

MFA Factor Strategy

Decidir factores soportados en MVP:

TOTP
WebAuthn
Email OTP
SMS OTP

y cuáles quedan para fases posteriores.

ADR-IDENTITY-008

MFA Enforcement Policy

Determinar:

MFA opcional
MFA obligatorio
MFA por organización
MFA por rol
MFA por operación
ADR-IDENTITY-009

Recovery Strategy

Definir:

Recovery codes
Email recovery
Administrative recovery
Identity-provider recovery
ADR-IDENTITY-010

Trusted Device Policy

Definir:

duración;
número máximo;
revocación;
comportamiento después de cambio de contraseña;
comportamiento después de cambio de MFA.
ADR-IDENTITY-011

Step-Up Authentication

Definir qué operaciones requieren una nueva autenticación MFA.

ADR-IDENTITY-012

Authentication Assurance

Determinar si EVOXA utilizará:

amr
acr

o un modelo propio de assurance levels.

36. Criterios de aceptación

ESP-0001/07 estará listo para implementación cuando:

 Exista modelo definitivo de MFAFactor.
 Exista modelo definitivo de MFAChallenge.
 Exista modelo definitivo de RecoveryCode.
 Exista modelo definitivo de TrustedDevice.
 Estén definidos los factores del MVP.
 Esté definido el enrollment.
 Esté definido el login con MFA.
 Esté definido step-up authentication.
 Esté definido recovery.
 Estén definidos los eventos.
 Esté definido audit logging.
 Estén definidas las políticas de rate limiting.
 Esté definida la estrategia de secretos.
 Estén definidos los contratos API.
 Estén definidos los ADR correspondientes.
 Existan pruebas de seguridad.
 Existan pruebas de integración.
 Existan pruebas de API.
 Existan pruebas de recuperación.
 Existan pruebas de revocación.
37. Trazabilidad
BP-0002 Reference Architecture
        │
        ├── Identity
        │
        ├── Authentication
        │
        ├── Authorization
        │
        ├── Security
        │
        └── Audit
                │
                ▼
        ESP-0001 Identity
                │
        ┌───────┴────────┐
        ▼                ▼
 ESP-0001/03       ESP-0001/06
 Authentication    Sessions & Tokens
        │                │
        └───────┬────────┘
                ▼
        ESP-0001/07
             MFA
                │
                ▼
        API Contracts
                │
                ▼
        Implementation Stories
                │
                ▼
             Code
                │
                ▼
             Tests

La trazabilidad es especialmente importante porque el mapa oficial de arquitectura establece que la implementación debe avanzar desde Blueprint → ESP → Implementation Stories → Technical Tasks → Source Code → Testing → Deployment → Monitoring.

Estado de ESP-0001 hasta ahora

Con este capítulo tenemos:

Sección	Estado
01 — Identity Domain Model	Draft
02 — Identity Architecture	Draft
03 — Authentication & Authorization	Draft
04 — User & Organization Management	Draft
05 — Roles & Permissions	Draft
06 — Sessions & Token Management	Draft
07 — Multi-Factor Authentication	Draft

Conclusión: todavía no conviene programar MFA directamente. Primero debemos cerrar las decisiones arquitectónicas mediante los ADRs y después convertir esta especificación en Implementation Stories + API Contracts + Persistence Model + Events + Tests. Esto mantiene la metodología de EVOXA y evita terminar con un backend que luego tengamos que rehacer.
