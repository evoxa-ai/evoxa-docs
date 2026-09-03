ADR-IDENTITY-035 — Identity Account Recovery & Credential Recovery Strategy

Status: Proposed
Version: 1.0
Date: 2026-09-03
Domain: Identity
Priority: Critical
Type: Security / Architecture / Identity

1. Context

EVOXA Identity debe permitir que un usuario recupere el acceso a su cuenta cuando pierde o no puede utilizar sus mecanismos normales de autenticación.

La recuperación es una superficie de seguridad especialmente sensible porque, en la práctica, constituye una ruta alternativa hacia una cuenta.

Por lo tanto:

Account Recovery must provide a secure alternative authentication path without becoming an authentication bypass.

La estrategia debe integrarse con:

User Account Lifecycle
Authentication
Password Security
MFA
Sessions
Refresh Tokens
Security Policy Engine
Rate Limiting & Abuse Prevention
Audit & Security Events
Tenant Isolation
KMS / Secret Management
Event / Outbox Architecture

Este ADR continúa directamente las decisiones de:

ADR-IDENTITY-010 — Testing & Quality
ADR-IDENTITY-023 — MFA Factor Strategy
ADR-IDENTITY-024 — MFA Challenge TTL, Attempts & Recovery
ADR-IDENTITY-025 — Refresh Token Cryptography & Concurrency
ADR-IDENTITY-027 — Runtime Configuration, Secrets & KMS
ADR-IDENTITY-028 — Observability & Operational Security
ADR-IDENTITY-032 — Rate Limiting, Abuse Prevention & Brute-Force Protection
ADR-IDENTITY-033 — Account Lifecycle, Lockout & Suspicious Account Management
ADR-IDENTITY-034 — Password Security & Credential Lifecycle
2. Problem Statement

Un usuario puede perder acceso por diferentes motivos:

olvidar su contraseña;
perder acceso a su correo;
perder un dispositivo MFA;
perder un factor TOTP;
perder una Passkey/WebAuthn;
consumir o perder recovery codes;
perder un trusted device;
sospecha de compromiso;
suspensión temporal de la cuenta;
problemas con proveedores externos de autenticación;
pérdida de acceso a un mecanismo de autenticación administrativo.

No todos estos escenarios tienen el mismo nivel de riesgo.

Por ejemplo:

Forgot Password
      ↓
Password Reset

no debe tener necesariamente el mismo nivel de confianza que:

Lost MFA Factor
      ↓
Identity Recovery
      ↓
New MFA Enrollment

La arquitectura debe impedir que:

Recovery → Authentication Bypass

y establecer:

Recovery → Identity Verification → Policy Evaluation → Controlled Credential Change
3. Decision

EVOXA adoptará una estrategia de recuperación basada en niveles de confianza, verificación de identidad, políticas de seguridad y evidencia de recuperación.

La recuperación será tratada como un flujo de seguridad independiente, aunque integrado con Authentication, MFA y Credential Management.

El sistema deberá distinguir como mínimo entre:

Password Recovery
MFA Factor Recovery
Account Access Recovery
Credential Compromise Recovery
Administrative Recovery

No todos los tipos de recuperación podrán utilizar los mismos mecanismos.

4. Recovery Architecture

Conceptualmente:

                    ┌─────────────────────┐
                    │ Recovery Request    │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Identify Account    │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Rate Limit / Abuse   │
                    │ Protection           │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Risk Assessment      │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Recovery Policy      │
                    │ Evaluation           │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
           ALLOW            CHALLENGE         DENY
              │                │
              │                ▼
              │       Identity Verification
              │                │
              └────────┬───────┘
                       ▼
             ┌─────────────────────┐
             │ Recovery Evidence   │
             └──────────┬──────────┘
                        │
                        ▼
             ┌─────────────────────┐
             │ Credential Change   │
             │ / Factor Recovery   │
             └──────────┬──────────┘
                        │
                        ▼
             ┌─────────────────────┐
             │ Session / Token     │
             │ Security Actions   │
             └──────────┬──────────┘
                        │
                        ▼
             ┌─────────────────────┐
             │ Audit + Security    │
             │ Event               │
             └─────────────────────┘
5. Recovery Types
5.1 Password Recovery

El caso más común será:

Forgot Password
        ↓
Password Recovery Request
        ↓
Identity / Recovery Verification
        ↓
New Password
        ↓
Credential Update

Debe integrarse con ADR-IDENTITY-034.

El usuario nunca debe recibir información que permita determinar si una cuenta específica existe.

Por ejemplo, una solicitud de recuperación debe utilizar respuestas genéricas:

"If the account can be recovered, further instructions will be provided."

La respuesta no debe revelar:

si el email existe;
estado de la cuenta;
estado de MFA;
existencia de factores;
motivo del rechazo;
información del tenant.
6. Recovery Token

Los mecanismos de recuperación basados en enlaces o códigos deberán utilizar secretos criptográficamente seguros.

El recovery token deberá ser:

aleatorio;
impredecible;
de corta duración;
de un solo uso;
asociado a una cuenta;
asociado a un propósito;
protegido contra replay;
invalidable;
nunca almacenado en texto plano cuando sea posible.

Conceptualmente:

RecoveryToken
├── id
├── user_id
├── purpose
├── token_hash
├── status
├── created_at
├── expires_at
├── consumed_at
├── revoked_at
└── metadata

Estados conceptuales:

ISSUED
ACTIVE
CONSUMED
EXPIRED
REVOKED

Un token consumido no puede volver a utilizarse.

7. Recovery Purpose

Cada mecanismo debe declarar explícitamente su propósito.

Ejemplos:

PASSWORD_RESET
MFA_RECOVERY
ACCOUNT_RECOVERY
EMAIL_CHANGE_RECOVERY
FACTOR_CHANGE_RECOVERY
COMPROMISE_RECOVERY
ADMIN_RECOVERY

Un token generado para:

PASSWORD_RESET

no puede utilizarse para:

MFA_RECOVERY

o:

EMAIL_CHANGE

El propósito debe formar parte de la validación de seguridad.

8. Identity Verification

La recuperación debe basarse en evidencia suficiente para el nivel de riesgo.

Las posibles evidencias incluyen:

recovery code;
factor MFA existente;
trusted device;
sesión autenticada válida;
email recovery;
passkey/WebAuthn;
mecanismos externos autorizados;
verificación administrativa;
identity proofing cuando sea requerido.

La disponibilidad de una sola evidencia no implica automáticamente autorización para cualquier operación.

Por ejemplo:

Password Reset

puede requerir una evidencia diferente de:

Disable MFA

o:

Change Primary Email
9. Recovery Assurance Levels

Se adoptará conceptualmente un modelo de niveles de confianza.

Level 1 — Low Risk Recovery

Ejemplo:

Forgot password
+
valid approved recovery mechanism

Puede permitir:

password reset

pero no necesariamente:

disable MFA
change security factors
change organization ownership
Level 2 — Elevated Recovery

Cuando el usuario perdió un factor importante:

Lost MFA
+
secondary identity evidence

puede requerirse:

múltiples evidencias;
step-up;
trusted device;
recovery code;
identity verification adicional.
Level 3 — High Risk Recovery

Casos como:

pérdida total de factores;
sospecha de compromiso;
cambio simultáneo de credenciales;
cambio de email principal;
actividad anómala;
recuperación de una cuenta administrativa.

deberán estar sujetos a una política más estricta.

El resultado puede ser:

CHALLENGE

o:

DENY

hasta completar una verificación adicional.

10. Recovery Codes

Los recovery codes definidos en ADR-023/024 serán considerados un mecanismo de recuperación de alta sensibilidad.

Cada código debe:

ser único;
ser de un solo uso;
almacenarse de forma protegida;
invalidarse después de su utilización;
no aparecer en logs;
no aparecer en eventos;
no aparecer en traces;
no aparecer en respuestas de API después de su creación;
no ser accesible por agentes AI.

Conceptualmente:

RecoveryCode
     ↓
USED
     ↓
Cannot be reused

La regeneración de un nuevo conjunto deberá preferentemente invalidar el conjunto anterior.

11. MFA Factor Recovery

La pérdida de un factor MFA no debe permitir simplemente desactivar MFA.

Flujo:

User reports lost factor
        ↓
Recovery request
        ↓
Policy evaluation
        ↓
Identity verification
        ↓
Recovery evidence
        ↓
Security decision
        ↓
New factor enrollment

El nuevo factor deberá pasar por el proceso normal de enrollment definido en ADR-023.

12. Recovery vs MFA Bypass

Una regla crítica:

Recovery must never silently downgrade the security posture of the account.

Por ejemplo:

User has MFA
       ↓
Lost TOTP
       ↓
Recovery
       ↓
New TOTP

es válido.

Pero:

User has MFA
       ↓
Lost TOTP
       ↓
Disable MFA permanently

no debe ser el comportamiento predeterminado.

13. Recovery and Sessions

Una recuperación de credenciales puede indicar posible compromiso.

Por lo tanto, dependiendo de la política, una recuperación exitosa puede provocar:

Password reset
      ↓
Revoke existing sessions
      ↓
Revoke refresh token families
      ↓
Require new authentication

Especialmente cuando:

el usuario reporta compromiso;
se pierde un factor;
se cambia una credencial crítica;
existe evidencia de takeover.

No se debe asumir automáticamente que todas las sesiones deben mantenerse activas.

La decisión será policy-driven.

14. Recovery and Refresh Tokens

Si la recuperación representa un cambio de seguridad significativo:

Recovery
   ↓
Credential change
   ↓
Refresh token invalidation

deberá poder revocarse:

sesión actual;
otras sesiones;
refresh-token family;
todas las refresh-token families.

Esto deberá ejecutarse de forma consistente con ADR-IDENTITY-025.

Un refresh token previamente revocado nunca podrá volver a convertirse en válido como consecuencia de un proceso de recuperación.

15. Recovery and Account Lifecycle

Recovery debe respetar el estado de la cuenta.

Por ejemplo:

DISABLED

no debe poder convertirse automáticamente en:

ACTIVE

simplemente mediante un password reset.

Del mismo modo:

SUSPENDED

no implica que el usuario pueda eliminar la suspensión mediante recuperación.

La recuperación de credenciales y la recuperación del estado de cuenta son operaciones diferentes.

16. Suspicious Account Recovery

Si existen señales de compromiso:

Refresh Token Reuse
Login Anomalies
MFA Abuse
Credential Stuffing
Suspicious Device
Password Reset Abuse

el Security Policy Engine puede elevar el nivel de recuperación.

Ejemplo:

Normal:
Password Recovery → ALLOW

High Risk:
Password Recovery → CHALLENGE

Confirmed Compromise:
Password Recovery → DENY / MANUAL REVIEW

La evaluación deberá integrarse con ADR-IDENTITY-033.

17. Administrative Recovery

Los administradores podrán disponer de mecanismos de recuperación administrativa únicamente cuando exista una política explícita que lo permita.

Un administrador no debe poder:

ver passwords;
ver password hashes;
ver MFA secrets;
ver recovery codes;
extraer refresh tokens;
crear credenciales secretas sin trazabilidad;
saltarse políticas de tenant;
realizar recovery sin audit trail.

Administrative Recovery debe incluir:

Authorized Actor
+
Explicit Operation
+
Policy Evaluation
+
Tenant Context
+
Audit Event
+
Security Event when appropriate
18. Dual Control for Critical Recovery

Para operaciones de alto impacto, EVOXA podrá soportar posteriormente un modelo:

Administrator A
       +
Administrator B
       ↓
Critical Recovery

Esto puede aplicarse a:

cuentas administrativas críticas;
recuperación de ownership;
cambios de seguridad de alto riesgo;
recuperación de organizaciones críticas.

Esta capacidad queda como extensión futura, no como requisito obligatorio de la primera implementación.

19. Email Recovery

El email puede ser utilizado como mecanismo de recuperación únicamente cuando:

haya sido previamente verificado;
la política permita utilizarlo;
no exista evidencia que indique compromiso del canal;
se apliquen TTL y límites de intentos;
el token sea de un solo uso.

El sistema debe evitar:

Email change
     ↓
Immediately trust new email
     ↓
Use new email to bypass MFA

Los cambios de email deberán considerarse operaciones sensibles.

20. Recovery Abuse Protection

Todo endpoint de recovery debe estar protegido por ADR-IDENTITY-032.

Dimensiones potenciales:

IP
User
Email
Device
Session
Tenant
Endpoint
Recovery Purpose
Risk

Se deberá prevenir:

enumeración de usuarios;
spam de emails;
token guessing;
brute force;
repeated recovery;
MFA recovery abuse;
account takeover automation.

La recuperación nunca debe utilizarse como una vía para eludir los controles de rate limiting.

21. Anti-Enumeration

El sistema deberá evitar respuestas diferentes para:

existing account

vs.

non-existing account

Las respuestas públicas deben minimizar información.

Por ejemplo:

Recovery request accepted.

en lugar de:

This email does not exist.

Los detalles internos pueden registrarse en Audit/Security Events bajo los controles correspondientes.

22. Recovery Evidence

Una recuperación exitosa deberá generar una evidencia conceptual:

RecoveryEvidence
├── recovery_id
├── user_id
├── purpose
├── assurance_level
├── verification_method
├── verified_at
├── policy_id
├── policy_version
├── risk_context
└── expires_at

Esta evidencia puede permitir una operación posterior durante un período limitado.

Ejemplo:

Recovery verified
       ↓
Temporary recovery evidence
       ↓
Reset password
       ↓
Evidence expires

La evidencia no debe convertirse en una credencial permanente.

23. Recovery Evidence Expiration

La evidencia de recovery deberá tener una validez limitada.

Recovery Evidence
       ↓
ACTIVE
       ↓
EXPIRED

Después de expirar:

Credential Change
→ DENY

o deberá iniciarse nuevamente la recuperación.

El TTL exacto queda pendiente de definición mediante política.

24. Security Policy Engine Integration

El Recovery Service no decidirá por sí solo qué recuperación está permitida.

La decisión deberá integrar:

Recovery Request
+
User
+
Organization
+
Membership
+
Account State
+
Existing Factors
+
Session
+
Device
+
Risk
+
Recovery Evidence
+
Requested Operation
+
Security Policy

El resultado podrá ser:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE
25. AI Integration

Los AI Agents podrán ayudar a:

detectar patrones anómalos;
priorizar recovery cases;
detectar posible account takeover;
recomendar controles adicionales;
analizar Security Events.

Pero un AI Agent no podrá:

revelar recovery codes;
recuperar passwords;
generar credenciales privilegiadas sin autorización;
desactivar MFA;
saltarse identity verification;
convertir DENY en ALLOW;
modificar políticas críticas;
aprobar recuperación crítica sin las reglas correspondientes.

La decisión de seguridad deberá permanecer dentro de Identity/Security Policy.

26. Audit Events

Toda recuperación relevante deberá producir Audit Events.

Ejemplos:

ACCOUNT_RECOVERY_REQUESTED
ACCOUNT_RECOVERY_VERIFIED
ACCOUNT_RECOVERY_FAILED
PASSWORD_RECOVERY_REQUESTED
PASSWORD_RESET_COMPLETED
MFA_RECOVERY_REQUESTED
MFA_RECOVERY_COMPLETED
RECOVERY_TOKEN_CONSUMED
RECOVERY_TOKEN_REVOKED
ADMIN_RECOVERY_EXECUTED

Los eventos deberán incluir contexto suficiente para investigación, pero nunca secretos.

27. Security Events

Determinados escenarios deberán generar Security Events:

Repeated Recovery Attempts
Recovery Abuse
Recovery Token Reuse
MFA Recovery Anomaly
Suspicious Password Reset
Administrative Recovery
High Risk Account Recovery
Possible Account Takeover

El severity level será determinado por la política de seguridad.

28. Event Architecture

Los cambios transaccionales deberán seguir:

Database Transaction
       │
       ├── Domain State
       │
       └── Outbox Event
              ↓
        Event Platform

No se deberá publicar un evento de recuperación exitoso si la operación transaccional no fue confirmada.

Los consumidores deberán ser idempotentes.

29. Tenant Isolation

Toda recuperación deberá respetar:

User
Organization
Membership
Tenant Policy

Un actor de una organización no puede recuperar o administrar credenciales de otra organización salvo que exista una autorización explícita de plataforma.

El recovery context deberá incluir tenant context cuando corresponda.

30. Data Protection

Nunca deberán registrarse:

recovery tokens;
recovery codes;
passwords;
password hashes;
MFA secrets;
TOTP secrets;
private keys;
refresh tokens;
API secrets.

El sistema deberá registrar únicamente referencias y metadatos necesarios para auditoría.

31. Persistence

La persistencia conceptual puede requerir:

recovery_requests
recovery_tokens
recovery_evidence
recovery_attempts

Además de las entidades ya definidas:

users
sessions
refresh_tokens
mfa_factors
mfa_challenges
recovery_codes
trusted_devices
audit_events
security_events
outbox_events

Los nombres definitivos de tablas quedan sujetos a IS-015 y al diseño de persistence.

32. Concurrency

Recovery deberá ser concurrency-safe.

Casos críticos:

Two recovery requests
Two token consumptions
Two password resets
Two MFA enrollments
Recovery + password change
Recovery + account suspension
Recovery + session revocation

Ejemplo:

Recovery Token
      │
      ├── Request A → CONSUME
      │
      └── Request B → REJECT

Solo una operación podrá consumir exitosamente un token one-time.

33. Idempotency

Las operaciones retryable deberán integrarse con ADR-IDENTITY-026.

Particularmente:

recovery confirmation;
password reset completion;
administrative recovery;
sensitive recovery operations.

Un retry debido a timeout no debe crear dos efectos de negocio.

Pero:

same Idempotency-Key
+
different request

debe generar:

IDEMPOTENCY_CONFLICT
34. Recovery Failure

Una recuperación fallida no debe revelar información sensible.

El sistema podrá devolver:

RECOVERY_FAILED

o:

RECOVERY_CHALLENGE_REQUIRED

pero los detalles internos deberán permanecer restringidos.

Los intentos deberán ser contabilizados cuando corresponda.

35. Recovery Completion

Una recuperación completada debe ejecutar una secuencia controlada:

Verify Evidence
      ↓
Evaluate Policy
      ↓
Validate Account State
      ↓
Perform Credential/Factor Change
      ↓
Apply Session/Token Security Actions
      ↓
Persist Audit
      ↓
Persist Security Event when required
      ↓
Publish Outbox Event

Todo debe ejecutarse de manera consistente y transaccional donde corresponda.

36. Consequences
Positivas
Reduce account takeover.
Evita que recovery se convierta en MFA bypass.
Separa password reset de account recovery.
Permite diferentes niveles de assurance.
Integra recovery con Risk/Policy.
Permite recuperación futura de nuevos factores.
Mantiene trazabilidad completa.
Soporta multi-tenant.
Facilita detección de abuso.
Compatible con AI-assisted security sin entregar control crítico al AI.
Negativas
Mayor complejidad de implementación.
Más estados y entidades.
Mayor cantidad de políticas.
Requiere testing extensivo.
Recovery de alto riesgo puede requerir intervención humana.
Dependencia de mecanismos externos como email o WebAuthn cuando sean utilizados.
37. Alternatives Considered
A. Password Reset Only

No adoptado.

No cubre pérdida de MFA, account takeover ni recuperación de factores.

B. Disable MFA During Recovery

Rejected.

Crearía un bypass directo de seguridad.

C. Admin Can Reset Anything

Rejected.

Viola least privilege, separación de responsabilidades y auditability.

D. Permanent Recovery Tokens

Rejected.

Incrementaría significativamente el impacto de un token comprometido.

E. Recovery Without Risk Assessment

Rejected.

Todos los escenarios de recuperación no tienen el mismo nivel de riesgo.

F. AI-Based Recovery Approval

Rejected.

AI puede asistir en análisis, pero no debe convertirse en autoridad de seguridad independiente.

38. Non-Negotiable Security Rules
Recovery nunca debe convertirse en authentication bypass.
Recovery tokens deben ser one-time.
Recovery tokens deben expirar.
Recovery codes deben ser one-time.
No recovery secrets en logs/events/traces.
MFA no puede ser deshabilitado automáticamente como mecanismo genérico de recuperación.
Recovery debe respetar Account Lifecycle.
Recovery debe respetar Tenant Isolation.
Recovery debe estar sujeto a Rate Limiting.
Recovery debe integrarse con Security Policy Engine.
High-risk recovery debe poder requerir challenge o manual review.
Administrative Recovery debe ser auditado.
AI no puede saltarse las políticas de Identity.
Recovery debe ser concurrency-safe.
Credential changes críticos deben poder revocar sesiones/tokens según política.
Un token/recovery evidence consumido no puede reutilizarse.
Failures deben ser fail-closed.
Recovery no debe permitir enumeración de cuentas.
Secrets nunca deben quedar almacenados en plaintext.
Recovery debe poder ser investigado mediante Audit/Security Events.
39. Traceability
Requirement	Source / Relationship
Identity lifecycle	ARCHITECTURE-MAP / Identity ESP
Authentication	IS-IDENTITY-005
Authorization	IS-IDENTITY-006
MFA	IS-IDENTITY-010
MFA challenge/recovery	ADR-IDENTITY-024
Sessions	IS-IDENTITY-008
Refresh Tokens	IS-IDENTITY-009 / ADR-025
API Idempotency	ADR-IDENTITY-026
Secrets/KMS	ADR-IDENTITY-027
Observability	ADR-IDENTITY-028
Disaster Recovery	ADR-IDENTITY-029/030
Rate limiting	ADR-IDENTITY-032
Account lifecycle	ADR-IDENTITY-033
Password security	ADR-IDENTITY-034
Recovery	ADR-IDENTITY-035

La estructura general de trazabilidad del Blueprint exige mantener la relación entre ESP → IS → Technical Tasks → Source Code.

40. Dependencies

Este ADR depende principalmente de:

ADR-IDENTITY-023
ADR-IDENTITY-024
ADR-IDENTITY-025
ADR-IDENTITY-026
ADR-IDENTITY-027
ADR-IDENTITY-028
ADR-IDENTITY-032
ADR-IDENTITY-033
ADR-IDENTITY-034

Y deberá materializarse posteriormente mediante:

IS-IDENTITY-035
Technical Tasks
API Contracts
Persistence Specification
Testing Specification
41. Pending Decisions

Los siguientes valores no deben inventarse todavía y quedan pendientes de especificación:

Recovery
exact recovery token TTL;
exact recovery evidence TTL;
token entropy/encoding;
token hashing/HMAC strategy;
pepper strategy;
recovery request limits;
retry limits;
recovery attempt windows.
Password Recovery
exact password reset policy;
session revocation behavior;
refresh family revocation behavior;
notification strategy;
email provider.
MFA Recovery
exact minimum evidence;
required assurance level;
factor replacement rules;
trusted-device rules;
recovery-code policy.
High-Risk Recovery
exact risk thresholds;
manual review requirements;
dual-control requirements;
administrative recovery workflow.
Infrastructure
recovery token persistence;
Redis usage;
notification provider;
identity-proofing provider;
KMS integration;
regional recovery behavior.

Estas decisiones deberán cerrarse mediante ADRs o especificaciones posteriores cuando corresponda.

42. Acceptance Criteria

El ADR se considerará correctamente implementado cuando:

 Password Recovery tenga un flujo independiente y seguro.
 MFA Recovery no permita bypass de MFA.
 Recovery Tokens sean one-time.
 Recovery Tokens tengan expiración.
 Recovery Evidence tenga expiración.
 Recovery respete User Account Lifecycle.
 Recovery respete Tenant Isolation.
 Recovery esté protegido contra brute force y abuse.
 Recovery evite account enumeration.
 Recovery esté integrado con Security Policy Engine.
 High-risk recovery pueda requerir CHALLENGE o manual review.
 Recovery pueda revocar sessions/refresh tokens cuando la política lo requiera.
 Administrative Recovery esté autorizado y auditado.
 Recovery Codes sean single-use.
 Recovery secrets no aparezcan en logs/events/traces.
 Recovery sea concurrency-safe.
 Recovery sea idempotente cuando corresponda.
 Domain changes y Outbox Events sean consistentes.
 Security Events permitan investigar abusos.
 AI no pueda bypass Identity controls.
 Existan pruebas de seguridad, concurrencia, replay, tenant isolation y abuse prevention.
43. Next ADR

Con este ADR queda definida la estrategia conceptual de recuperación de cuentas y credenciales.

El siguiente paso natural es:

ADR-IDENTITY-036 — Identity Email Verification & Communication Security Strategy

para definir formalmente:

email verification;
email change;
verification tokens;
communication security;
notification security;
anti-enumeration;
email ownership;
recovery email;
notification events;
protección frente a email takeover.

Esto nos permitirá cerrar otra pieza importante antes de entrar a las Implementation Stories y Technical Tasks de Identity.
