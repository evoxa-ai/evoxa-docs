ADR-IDENTITY-023 — MFA Factor Strategy

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-03
Dominio: Identity
Prioridad: High / Critical
Tipo: Architecture Decision Record

1. Título

ADR-IDENTITY-023 — MFA Factor Strategy

Define la estrategia arquitectónica para los factores de autenticación multifactor (MFA) de EVOXA, incluyendo:

tipos de factores soportados;
ciclo de vida;
enrolamiento;
activación;
verificación;
revocación;
recuperación;
dispositivos confiables;
step-up authentication;
integración con Authentication;
integración con Authorization;
integración con Security Policy Engine;
auditoría y eventos de seguridad;
protección de secretos MFA;
evolución futura de los factores.
2. Contexto

El Blueprint de EVOXA establece MFA como una capacidad transversal de Identity.

MFA no debe considerarse simplemente como una pantalla adicional durante el login. Forma parte del modelo de seguridad de Identity y debe integrarse con:

Authentication → Session → Authorization → Security Policy → Audit/Security Events

El diseño previamente establecido para IS-IDENTITY-010 — Multi-Factor Authentication contempla entidades como:

MFAFactor
MFAChallenge
RecoveryCode
TrustedDevice

y contempla que MFA pueda utilizarse tanto durante el login como posteriormente para operaciones que requieran un nivel de autenticación superior.

Además, ADR-IDENTITY-005 establece que las decisiones de autorización pueden producir:

ALLOW
DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE

Por lo tanto, MFA debe funcionar como una capacidad que puede ser solicitada dinámicamente por el modelo de seguridad.

3. Problema

EVOXA necesita definir una estrategia coherente para MFA que permita:

incorporar diferentes factores de autenticación;
evitar depender de un único proveedor;
proteger los secretos MFA;
permitir enrolamiento y revocación seguros;
soportar MFA durante login;
soportar MFA para operaciones sensibles;
soportar recuperación de cuenta;
permitir dispositivos confiables;
mantener trazabilidad y auditoría;
permitir incorporar nuevos factores sin rediseñar Identity;
permitir que Security Policy Engine determine cuándo MFA es obligatorio.

Sin esta decisión, cada mecanismo de MFA podría terminar implementándose de forma independiente, generando diferentes modelos de seguridad, almacenamiento, auditoría y recuperación.

4. Decisión

EVOXA adoptará una arquitectura de MFA basada en factores desacoplados, donde el dominio Identity define el contrato común de MFA y cada factor implementa su propio mecanismo de verificación.

La arquitectura será conceptualmente:

                    ┌──────────────────────┐
                    │ Security Policy      │
                    │ Engine               │
                    └──────────┬───────────┘
                               │
                         REQUIRE_MFA
                               │
                               ▼
                    ┌──────────────────────┐
                    │ MFA Orchestrator     │
                    └──────────┬───────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
           TOTP             WebAuthn         Passkey
              │                │                │
              └────────────────┼────────────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ MFA Challenge        │
                    └──────────┬───────────┘
                               │
                         verification
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Authentication /     │
                    │ Session Context      │
                    └──────────────────────┘

El dominio no debe depender directamente de un proveedor externo específico.

5. Modelo de factores

Los factores serán representados mediante un modelo común:

MFAFactor
├── id
├── user_id
├── type
├── status
├── enrolled_at
├── activated_at
├── revoked_at
├── last_used_at
├── metadata
└── security_metadata

Los detalles específicos del factor no deben contaminar el modelo común.

Por ejemplo:

MFAFactor
type = TOTP

puede tener información específica protegida para TOTP.

Mientras que:

MFAFactor
type = WEBAUTHN

utilizará información específica de WebAuthn.

6. Catálogo de factores

El Blueprint/IS-IDENTITY-010 contempla como posibilidades:

TOTP
WebAuthn
Passkey
SMS
Email
Push
Hardware Key

Estos factores deben considerarse parte del catálogo conceptual, no todos como una obligación de implementación inicial.

La arquitectura deberá permitir incorporarlos progresivamente.

6.1 Factores preferentes

La estrategia deberá favorecer factores de mayor seguridad y menor dependencia de canales externos.

Conceptualmente:

WebAuthn / Passkey
        ↓
Hardware Security Key
        ↓
TOTP
        ↓
Push
        ↓
Email / SMS

Este orden representa una dirección arquitectónica de seguridad, no una obligación normativa del Blueprint sobre el orden exacto de implementación.

La selección definitiva de factores iniciales queda pendiente de la especificación de producto y seguridad.

7. Factor vs desafío

Se establece una separación explícita entre:

MFA Factor

Representa el mecanismo registrado por el usuario.

Ejemplo:

Factor:
TOTP
User:
123
Status:
ACTIVE
MFA Challenge

Representa una solicitud concreta de verificación.

Ejemplo:

Challenge:
abc123
Factor:
factor-456
Purpose:
LOGIN
Status:
PENDING

Por lo tanto:

MFAFactor
     │
     ├── Challenge 001
     ├── Challenge 002
     └── Challenge 003

Un factor puede generar múltiples desafíos durante su vida útil.

8. Ciclo de vida del factor

El ciclo de vida conceptual será:

PENDING
   │
   ▼
ACTIVE
   │
   ▼
REVOKED
PENDING

Factor registrado pero todavía no verificado.

ACTIVE

Factor validado y disponible para autenticación.

REVOKED

Factor invalidado y no utilizable.

Un factor revocado no debe reutilizarse simplemente cambiando su estado.

Si el usuario desea volver a utilizar el mecanismo, deberá realizar un nuevo enrolamiento según las políticas vigentes.

9. Enrolamiento MFA

El proceso conceptual será:

Authenticated User
        │
        ▼
Policy Evaluation
        │
        ▼
Create MFA Factor
        │
        ▼
PENDING
        │
        ▼
Create Challenge
        │
        ▼
Verify Factor
        │
        ▼
ACTIVE
        │
        ▼
Audit + Event

El enrolamiento debe estar protegido.

No debe ser posible que un atacante que solamente haya obtenido una sesión parcial pueda registrar un nuevo factor sin superar los controles de seguridad correspondientes.

Security Policy Engine podrá exigir:

reautenticación;
MFA existente;
step-up;
dispositivo confiable;
otras condiciones contextuales.
10. Activación

Un factor no será considerado utilizable inmediatamente después de su creación.

Debe existir una prueba de posesión/verificación adecuada al tipo de factor.

Por ejemplo, conceptualmente:

Create Factor
     ↓
PENDING
     ↓
Verification
     ↓
ACTIVE

Esto evita que una configuración incompleta sea interpretada como una credencial válida.

11. MFA Challenge

El desafío MFA seguirá conceptualmente el modelo:

PENDING
   │
   ├────► VERIFIED
   │
   ├────► FAILED
   │
   ├────► EXPIRED
   │
   └────► REVOKED

El MFAChallenge deberá estar asociado, cuando corresponda, a:

usuario;
sesión;
factor;
propósito;
timestamps;
número de intentos;
resultado;
contexto de seguridad.
12. Propósitos de MFA

Los propósitos conceptuales definidos previamente incluyen:

LOGIN
STEP_UP
ENROLLMENT
FACTOR_CHANGE
SENSITIVE_OPERATION
RECOVERY

Estos propósitos permiten que una misma infraestructura MFA pueda utilizarse en diferentes escenarios.

LOGIN

MFA durante autenticación.

STEP_UP

Aumento temporal del nivel de confianza de una sesión.

ENROLLMENT

Verificación durante el registro de un nuevo factor.

FACTOR_CHANGE

Protección de modificaciones de factores existentes.

SENSITIVE_OPERATION

Protección de operaciones críticas.

RECOVERY

Proceso de recuperación de acceso.

El catálogo definitivo podrá evolucionar mediante las especificaciones correspondientes.

13. Step-Up Authentication

MFA deberá soportar step-up authentication.

Esto significa que un usuario puede tener una sesión válida pero no contar todavía con el nivel de autenticación necesario para una operación concreta.

Ejemplo:

Usuario autenticado
        │
        ▼
Solicita operación sensible
        │
        ▼
Authorization
        │
        ▼
Security Policy
        │
        ▼
REQUIRE_MFA
        │
        ▼
MFA Challenge
        │
        ▼
Verified
        │
        ▼
Authorization continúa

Esto evita exigir MFA para absolutamente todas las operaciones cuando la política de seguridad solo lo requiere para determinados riesgos.

14. Nivel de autenticación

La arquitectura deberá permitir distinguir entre:

Authenticated
Authenticated + MFA
Step-Up Authenticated
High Assurance Authentication

La definición exacta de niveles de assurance queda pendiente de una futura especificación de seguridad.

El modelo debe, sin embargo, ser suficientemente flexible para soportarlos.

15. Integración con Security Policy Engine

Security Policy Engine será responsable de determinar cuándo MFA es necesario.

MFA será responsable de ejecutar la verificación.

Por tanto:

Security Policy Engine
        │
        │ REQUIRE_MFA
        ▼
MFA
        │
        │ VERIFIED
        ▼
Authorization

No se duplicará la lógica de decisión entre MFA y Authorization.

MFA no decidirá por sí mismo que una operación de negocio requiere MFA salvo que exista una regla explícita definida por política.

16. Integración con Authorization

Una autorización podrá depender del estado MFA.

Ejemplo:

Permission:
billing.refund

MFA:
required

User:
authenticated

Result:
REQUIRE_MFA

Después de una verificación MFA válida:

Authorization
     ↓
ALLOW

La evidencia de MFA debe formar parte del contexto de autorización, no convertirse en un permiso permanente.

17. Protección de secretos MFA

Los secretos MFA son información altamente sensible.

Nunca deben almacenarse:

en texto plano;
en logs;
en eventos;
en respuestas API;
en trazas;
en mensajes de error;
en analytics;
en contextos de IA no autorizados.

Esto incluye, cuando corresponda:

secretos TOTP;
credenciales de recuperación;
información privada de autenticadores;
claves privadas;
códigos temporales.

La estrategia concreta de cifrado/protección se mantiene alineada con:

ADR-IDENTITY-014 — Encryption & Secret Protection

18. TOTP

Si TOTP es implementado, el secreto deberá tratarse como un secreto criptográfico.

Conceptualmente:

TOTP Secret
     ↓
Protected Storage
     ↓
Verification
     ↓
MFA Challenge

El sistema no debe almacenar códigos TOTP generados.

Debe almacenar únicamente el material necesario para realizar la verificación de forma segura.

Los parámetros criptográficos concretos quedan pendientes de la especificación de MFA.

19. WebAuthn / Passkeys

La arquitectura deberá poder soportar autenticadores basados en WebAuthn/Passkeys.

Estos mecanismos presentan una diferencia importante respecto de TOTP:

TOTP
Secret compartido

WebAuthn
Clave pública registrada
+
credencial del autenticador

El dominio Identity no debe asumir que todos los factores utilizan secretos compartidos.

La abstracción MFAFactor deberá permitir almacenar material específico según el tipo.

20. SMS y Email

SMS y Email podrán formar parte del catálogo de factores si se decide habilitarlos.

Sin embargo, su disponibilidad no deberá convertirse automáticamente en requisito de seguridad para todas las políticas.

Además, los mecanismos basados en canales externos deben considerar:

dependencia del proveedor;
disponibilidad;
rate limiting;
abuso;
interceptación;
costos;
privacidad;
recuperación;
observabilidad.

La decisión definitiva sobre su uso queda pendiente.

21. Push Authentication

Push podrá ser soportado como factor independiente.

El diseño debe permitir:

Create Challenge
       ↓
Push Provider
       ↓
User Approval
       ↓
Verification

La arquitectura deberá evitar que la disponibilidad de un proveedor push se convierta en una dependencia estructural del dominio.

22. Hardware Security Keys

Las claves físicas podrán representarse como factores MFA.

La arquitectura deberá permitir múltiples factores del mismo tipo.

Ejemplo:

User
 ├── Passkey A
 ├── Passkey B
 ├── Hardware Key A
 └── TOTP

Esto es importante para recuperación, reemplazo de dispositivos y escenarios de pérdida.

23. Múltiples factores

Un usuario podrá registrar múltiples factores.

No se deberá asumir:

1 usuario = 1 factor

El modelo será:

User
  │
  ├── MFAFactor 1
  ├── MFAFactor 2
  ├── MFAFactor 3
  └── MFAFactor N

Security Policy Engine podrá determinar qué combinación de factores es válida para una operación.

La lógica exacta de:

AND;
OR;
factor preferido;
factor obligatorio;
número mínimo de factores;

queda pendiente de una futura especificación de políticas.

24. Recovery Codes

Los códigos de recuperación se tratarán como credenciales sensibles.

Características conceptuales:

uso único;
almacenamiento protegido;
no recuperación del valor original;
auditoría de utilización;
revocación;
posibilidad de regeneración.

Flujo:

Generate
   ↓
Store Protected Representation
   ↓
User Receives Codes
   ↓
Use Code
   ↓
Mark Used

Un código utilizado no podrá volver a utilizarse.

25. Trusted Devices

EVOXA podrá soportar dispositivos confiables.

Un dispositivo confiable no equivale a una contraseña permanente ni a una sustitución absoluta de MFA.

Debe existir:

TrustedDevice
├── id
├── user_id
├── registered_at
├── last_seen_at
├── expires_at
├── status
└── device/security metadata

El dispositivo podrá reducir la frecuencia de desafíos MFA cuando una política lo permita.

26. Revocación de Trusted Device

Los dispositivos confiables deben poder ser revocados individualmente.

También podrán revocarse globalmente ante eventos como:

compromiso de cuenta;
cambio crítico de credenciales;
sospecha de fraude;
recuperación de cuenta;
política de seguridad.

Ejemplo:

Security Incident
       ↓
Revoke Trusted Devices
       ↓
Require MFA
       ↓
Reauthenticate
27. Cambio de factor

Cambiar un factor MFA se considerará una operación sensible.

Ejemplo:

Remove TOTP
Add Passkey

No deberá permitirse automáticamente solo porque exista una sesión autenticada.

Security Policy Engine podrá exigir:

REAUTHENTICATE
+
REQUIRE_MFA

antes de permitir el cambio.

28. Último factor

Debe evitarse que un usuario elimine accidentalmente todos sus mecanismos de recuperación cuando una política requiere MFA.

Ejemplo:

User has:
TOTP
Passkey
Recovery Codes

Remove TOTP
       ↓
Passkey + Recovery remain
       ↓
Allowed

Pero:

User has:
TOTP

Remove TOTP
       ↓
No recovery mechanism
       ↓
Policy evaluation
       ↓
DENY / REQUIRE RECOVERY

La regla exacta queda pendiente de definición.

29. Auditoría

Todas las operaciones relevantes de MFA deben generar trazabilidad.

Eventos conceptuales:

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

La información sensible del factor no debe incluirse en el evento.

30. Seguridad ante intentos repetidos

MFA debe estar protegido contra:

brute force;
replay;
enumeración;
abuso de challenges;
creación masiva de challenges;
abuso de recuperación;
abuso de SMS/Email;
automatización maliciosa.

Debe existir coordinación con:

rate limiting;
Security Policy Engine;
Authentication;
Audit;
Security Events.

Los límites exactos de intentos y TTL quedan deliberadamente pendientes.

31. MFA y sesiones

Una verificación MFA exitosa deberá producir evidencia asociada al contexto de autenticación.

Conceptualmente:

Session
   │
   ├── auth_method
   ├── mfa_status
   └── authentication_context

Esto permitirá que Authorization determine si una operación requiere una autenticación adicional.

La revocación de la sesión debe invalidar la capacidad de utilizar el contexto de esa sesión.

32. MFA y Refresh Tokens

Un refresh token no debe utilizarse para saltarse un requisito de MFA.

Durante un refresh, Identity debe considerar:

estado del usuario;
estado de la sesión;
membership;
organización;
políticas;
estado de MFA cuando corresponda.

Si la política exige una nueva autenticación o MFA:

Refresh
  ↓
Policy
  ↓
REAUTHENTICATE / REQUIRE_MFA

La sesión no debe obtener silenciosamente un nivel superior de confianza.

33. MFA y eventos

Los eventos MFA utilizarán el modelo de eventos definido en:

ADR-IDENTITY-018
ADR-IDENTITY-019
ADR-IDENTITY-020
ADR-IDENTITY-021
ADR-IDENTITY-022

Por tanto:

eventos versionados;
envelope estándar;
At-Least-Once;
consumidores idempotentes;
retry;
DLQ;
replay controlado.
34. MFA y AI

Los sistemas AI/Agents de EVOXA no podrán:

recuperar secretos MFA;
solicitar códigos MFA;
ver recovery codes;
acceder a secretos TOTP;
alterar MFA sin autorización;
bypass de Security Policy.

Un agente podrá conocer únicamente el contexto MFA permitido por Authorization.

Por ejemplo:

mfa_required = true
mfa_verified = false

pero no:

totp_secret = "..."
35. Persistencia

El modelo conceptual incluirá:

mfa_factors
mfa_challenges
recovery_codes
trusted_devices

Las relaciones principales serán:

User
 │
 ├── MFA Factors
 │
 ├── MFA Challenges
 │
 ├── Recovery Codes
 │
 └── Trusted Devices

Las estructuras específicas serán definidas en:

ADR-IDENTITY-012 — PostgreSQL Schema

y en:

IS-IDENTITY-015 — Identity Persistence

36. Cifrado y gestión de claves

La protección criptográfica de MFA se delega arquitectónicamente a la estrategia definida por:

ADR-IDENTITY-014 — Encryption & Secret Protection

No se definirá aquí una segunda estrategia criptográfica independiente.

La separación será:

ADR-023
MFA behavior & factor strategy
        │
        ▼
ADR-014
Cryptographic protection

Esto evita duplicación y contradicciones.

37. Integración con proveedores externos

Los factores que dependan de terceros deberán utilizar adaptadores.

Ejemplo:

Domain
  │
  ▼
MFA Provider Port
  │
  ├── SMS Provider
  ├── Email Provider
  ├── Push Provider
  └── Other Provider

El dominio Identity no deberá quedar acoplado directamente a SDKs de proveedores.

Los proveedores pertenecen a Infrastructure.

38. Disponibilidad y degradación

Un fallo de un proveedor externo no debe provocar un bypass de MFA.

Ejemplo:

SMS Provider DOWN
       ↓
NO MFA BYPASS
       ↓
Challenge Failed / Unavailable

La política puede permitir otro factor válido:

SMS unavailable
      ↓
TOTP available
      ↓
User verifies TOTP

Pero la decisión de fallback debe estar gobernada por políticas explícitas.

39. Consistencia y concurrencia

Las operaciones MFA sensibles deben ser concurrency-safe.

Especialmente:

uso de recovery code;
verificación de challenge;
revocación de factor;
activación de factor;
generación de recovery codes;
trusted-device registration;
trusted-device revocation.

Un código de recuperación utilizado simultáneamente no puede producir dos autenticaciones exitosas.

Esto deberá alinearse con:

ADR-IDENTITY-016 — Concurrency & Transaction Boundaries

40. Idempotencia

Las operaciones que puedan recibir reintentos deberán diseñarse para evitar efectos duplicados.

Ejemplos:

Enroll Factor
Generate Recovery Codes
Register Trusted Device
Revoke Factor
Revoke Trusted Device

La estrategia general deberá alinearse con:

ADR-IDENTITY-022 — Consumer Idempotency & Deduplication

y con el modelo de Idempotency-Key definido para APIs cuando corresponda.

41. Observabilidad

MFA deberá producir métricas y trazas operacionales sin exponer secretos.

Métricas conceptuales:

mfa_challenges_created
mfa_challenges_verified
mfa_challenges_failed
mfa_challenges_expired
mfa_factor_enrollments
mfa_factor_revocations
mfa_recovery_code_usage
mfa_provider_failures
mfa_step_up_requests

También deberá ser posible identificar:

factor type;
resultado;
latencia;
tenant;
región;
proveedor;

cuando sea apropiado y sin exponer información sensible.

42. Consecuencias positivas

Esta decisión proporciona:

Seguridad
separación entre autenticación y autorización;
MFA como capacidad transversal;
protección de secretos;
soporte para step-up;
recuperación controlada;
integración con políticas.
Arquitectura
factores desacoplados;
proveedores intercambiables;
dominio independiente de Infrastructure;
evolución progresiva.
Evolución

Será posible agregar nuevos mecanismos sin modificar fundamentalmente:

Authentication
Authorization
Session
Security Policy
Auditoría

Todas las operaciones relevantes quedan integradas con Audit/Security Events.

43. Consecuencias negativas

La solución introduce complejidad adicional:

múltiples tipos de factores;
múltiples flujos de enrolamiento;
recuperación;
dispositivos confiables;
proveedores externos;
seguridad criptográfica;
concurrencia;
compatibilidad;
soporte operacional.

También aumenta la superficie de testing.

Por ello, no todos los factores deben implementarse simultáneamente.

44. Alternativas consideradas
Alternativa A — Solo SMS

Rechazada.

Demasiada dependencia de un canal externo y menor flexibilidad arquitectónica.

Alternativa B — Solo TOTP

Rechazada como estrategia definitiva.

Puede ser un buen primer factor, pero no cubre la evolución hacia WebAuthn/Passkeys/hardware keys.

Alternativa C — MFA implementado dentro de Authentication

Rechazada.

Authentication debe orquestar la autenticación, pero MFA es una capacidad transversal reutilizable por:

login;
step-up;
operaciones sensibles;
cambios de seguridad;
recovery.
Alternativa D — MFA gestionado completamente por un proveedor externo

Rechazada como arquitectura base.

Puede existir integración con proveedores, pero Identity debe conservar el control sobre:

factores;
estado;
sesiones;
políticas;
autorización;
auditoría.
45. No negociables

Los siguientes principios son obligatorios:

MFA no puede ser bypass mediante Authorization.
MFA no puede ser bypass mediante Refresh Token.
Los secretos MFA nunca se almacenan en texto plano.
Los secretos MFA nunca aparecen en logs.
Los secretos MFA nunca aparecen en eventos.
Los recovery codes son de uso único.
Los MFA challenges deben expirar.
Las operaciones MFA sensibles deben ser concurrency-safe.
El enrolamiento debe estar protegido por Security Policy.
El cambio de factores debe ser una operación sensible.
La revocación debe quedar auditada.
Tenant isolation debe aplicarse a MFA.
AI Agents no pueden acceder a secretos MFA.
Los proveedores externos no deben acoplarse al dominio.
Un fallo externo nunca puede producir un bypass de MFA.
MFA debe integrarse con Audit/Security Events.
Los factores deben poder evolucionar sin rediseñar Identity.
El sistema debe soportar múltiples factores por usuario.
Las decisiones de obligatoriedad MFA deben pertenecer a Security Policy Engine.
MFA no debe convertirse en un mecanismo de autorización independiente.
46. Trazabilidad
Blueprint
Identity
 └── Multi-Factor Authentication
Engineering Specifications
ESP-0001 — Identity
Implementation Stories
IS-IDENTITY-005 — Authentication
IS-IDENTITY-006 — Authorization
IS-IDENTITY-008 — Session Management
IS-IDENTITY-009 — Refresh Token Lifecycle
IS-IDENTITY-010 — Multi-Factor Authentication
IS-IDENTITY-011 — Security Policy Engine
IS-IDENTITY-012 — Audit & Security Events
IS-IDENTITY-013 — Domain Events & Outbox
IS-IDENTITY-015 — Identity Persistence
IS-IDENTITY-016 — Identity Testing & Quality
ADRs
ADR-IDENTITY-005  Authorization
ADR-IDENTITY-006  Persistence Strategy
ADR-IDENTITY-007  Events & Outbox
ADR-IDENTITY-014  Encryption & Secret Protection
ADR-IDENTITY-016  Concurrency
ADR-IDENTITY-020  Event Schema Compatibility
ADR-IDENTITY-021  Event Replay
ADR-IDENTITY-022  Consumer Idempotency
47. Dependencias

Este ADR depende principalmente de:

ADR-IDENTITY-005
Authorization & Permission Evaluation

ADR-IDENTITY-008
Cryptographic Key Management

ADR-IDENTITY-009
Token & Client Storage

ADR-IDENTITY-014
Encryption & Secret Protection

ADR-IDENTITY-016
Concurrency & Transaction Boundaries

ADR-IDENTITY-018
Event Broker

ADR-IDENTITY-019
Event Partitioning & Ordering

ADR-IDENTITY-020
Event Schema Registry

ADR-IDENTITY-022
Consumer Idempotency
48. Decisiones pendientes

Este ADR define la arquitectura, pero deliberadamente no inventa valores que todavía no están definidos por el Blueprint.

Quedan pendientes:

Factores iniciales
TOTP;
WebAuthn;
Passkeys;
SMS;
Email;
Push;
Hardware Key.
Seguridad
algoritmos concretos;
almacenamiento de secretos;
KMS;
rotación;
protección de credenciales.
Challenges
TTL;
máximo de intentos;
políticas de bloqueo;
replay prevention.
Recovery
cantidad de recovery codes;
regeneración;
política de pérdida de factores;
validaciones adicionales.
Trusted Devices
TTL;
binding;
fingerprinting;
revocación;
renovación.
Step-Up
duración de la evidencia MFA;
qué operaciones requieren MFA;
qué roles requieren MFA;
qué niveles de riesgo requieren MFA.
Policy Engine
lenguaje formal de políticas;
operadores;
composición AND/OR;
precedencia entre factores;
fallback.

Estas decisiones deberán resolverse en las especificaciones y ADRs correspondientes, no mediante supuestos durante la implementación.

49. Criterios de aceptación arquitectónica

Este ADR podrá considerarse satisfecho cuando:

 MFA tenga una abstracción común de factor.
 MFAFactor y MFAChallenge estén conceptualmente separados.
 Exista lifecycle definido.
 Exista estrategia de enrolamiento.
 Exista estrategia de revocación.
 Exista soporte conceptual para múltiples factores.
 Exista soporte para step-up.
 Security Policy Engine controle cuándo se requiere MFA.
 Authorization pueda consumir el contexto MFA.
 Refresh Token no pueda utilizarse para bypass.
 Recovery Codes estén contemplados.
 Trusted Devices estén contemplados.
 Secretos MFA estén protegidos.
 Los eventos MFA estén definidos.
 Audit/Security Events estén integrados.
 Tenant isolation esté definido.
 AI Agents no tengan acceso a secretos.
 Existan límites para proveedores externos.
 Exista estrategia de concurrencia.
 Exista estrategia de idempotencia.
 Existan decisiones pendientes explícitamente documentadas.
50. Relación con los siguientes ADR

La continuación natural de esta serie queda:

ADR-IDENTITY-023
MFA Factor Strategy
        │
        ▼
ADR-IDENTITY-024
MFA Challenge TTL, Attempts & Recovery
        │
        ▼
ADR-IDENTITY-025
Refresh Token Cryptography & Concurrency
        │
        ▼
ADR-IDENTITY-026
API Pagination, Filtering & Idempotency

ADR-IDENTITY-024 deberá tomar este ADR como base y resolver específicamente los valores y políticas que aquí dejamos abiertos: TTL de challenges, número de intentos, expiración, bloqueo, recovery, trusted devices y comportamiento ante fallos.
