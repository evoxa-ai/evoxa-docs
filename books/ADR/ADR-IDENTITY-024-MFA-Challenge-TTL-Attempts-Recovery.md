ADR-IDENTITY-024 — MFA Challenge TTL, Attempts & Recovery

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-03
Dominio: Identity
Prioridad: Critical
Tipo: Architecture Decision Record

1. Título

ADR-IDENTITY-024 — MFA Challenge TTL, Attempts & Recovery

Define la estrategia arquitectónica para:

expiración de MFA Challenges;
TTL según propósito;
número máximo de intentos;
protección contra brute force;
replay prevention;
bloqueo de challenges;
concurrencia durante la verificación;
recuperación mediante Recovery Codes;
pérdida de factores;
Trusted Devices;
step-up authentication;
comportamiento ante fallos;
auditoría y eventos de seguridad.

Este ADR complementa directamente:

ADR-IDENTITY-023 — MFA Factor Strategy

2. Contexto

EVOXA define MFA como una capacidad transversal de Identity.

Un MFAFactor representa una credencial/factor registrado, mientras que un MFAChallenge representa una solicitud concreta de verificación.

El modelo conceptual establecido previamente es:

MFAFactor
    │
    ├── Challenge
    ├── Challenge
    └── Challenge

Un challenge no puede permanecer indefinidamente válido.

Debe existir una estrategia explícita para:

cuánto tiempo permanece válido;
cuántos intentos permite;
qué ocurre cuando expira;
qué ocurre ante intentos incorrectos;
cómo evitar replay;
cómo proteger recuperación;
cómo manejar pérdida de factores;
cómo manejar múltiples requests concurrentes.
3. Problema

Sin una política uniforme de TTL e intentos, pueden producirse problemas como:

códigos válidos durante demasiado tiempo;
brute force;
replay de challenges;
múltiples verificaciones simultáneas;
abuso de recovery;
abuso de SMS/Email;
desafíos abandonados que permanecen activos;
recuperación que se convierte en bypass de MFA;
inconsistencia entre login y step-up.

Por tanto, Identity necesita una estrategia común y centralizada.

4. Decisión

EVOXA adoptará una estrategia de MFA Challenge de vida corta, de uso controlado y de un solo resultado válido.

El principio será:

Created
   │
   ▼
PENDING
   │
   ├──► VERIFIED
   ├──► FAILED
   ├──► EXPIRED
   └──► REVOKED

Un challenge:

tendrá TTL;
tendrá límite de intentos cuando el factor lo requiera;
no podrá verificarse después de expirar;
no podrá reutilizarse después de una verificación exitosa;
deberá ser protegido contra concurrencia;
generará auditoría y eventos relevantes.

Los valores numéricos concretos no se fijan aquí sin validación de seguridad/producto.

5. Principio de TTL

Todo MFAChallenge debe tener:

created_at
expires_at

La validez será:

created_at <= now < expires_at

Una vez alcanzado expires_at:

PENDING → EXPIRED

El sistema no deberá aceptar el challenge después de su expiración aunque el código presentado sea correcto.

6. TTL por propósito

No todos los desafíos tienen necesariamente el mismo nivel de riesgo.

La arquitectura permitirá definir TTL según:

Challenge Purpose
+
Factor Type
+
Security Policy
+
Risk Context

Ejemplo conceptual:

LOGIN
STEP_UP
ENROLLMENT
FACTOR_CHANGE
SENSITIVE_OPERATION
RECOVERY

Security Policy Engine podrá definir requisitos diferentes.

Por lo tanto:

El TTL debe ser configurable por política y no codificado rígidamente dentro del dominio.

7. TTL por factor

Diferentes factores pueden tener características diferentes.

Por ejemplo:

TOTP
WebAuthn
SMS
Email
Push

No se debe asumir que todos necesitan exactamente el mismo comportamiento.

La infraestructura deberá permitir que un factor defina restricciones específicas cuando corresponda.

La política resultante deberá seguir siendo evaluada por Identity/Security Policy.

8. No reutilización

Un challenge exitosamente verificado debe quedar consumido.

PENDING
   │
   ▼
VERIFIED

Una segunda utilización debe ser rechazada.

Ejemplo:

Challenge #123
   │
   └── verification #1 → VERIFIED

verification #2
   ↓
DENY

Esto evita que una misma evidencia MFA sea utilizada repetidamente como si fueran autenticaciones independientes.

9. Prevención de Replay

La protección contra replay deberá contemplar:

identificación única del challenge;
estado persistente;
expiración;
consumo único;
asociación con usuario;
asociación con sesión cuando corresponda;
asociación con propósito;
validación contextual.

No se debe aceptar simplemente:

"código correcto"

Debe validarse el contexto completo.

Conceptualmente:

Challenge
+
User
+
Session
+
Factor
+
Purpose
+
Expiration
+
Status
10. Número máximo de intentos

Los challenges que utilizan códigos verificables deberán tener un límite de intentos.

Conceptualmente:

attempt_count
max_attempts

Cada intento fallido:

attempt_count += 1

Cuando se alcanza el máximo:

PENDING → FAILED

No deberá existir una cantidad ilimitada de intentos.

11. Intentos incorrectos

Un intento incorrecto deberá:

incrementar el contador;
generar la telemetría correspondiente;
poder generar un Security Event;
verificar si se alcanzó el límite;
impedir continuar cuando el challenge quede bloqueado.

El sistema debe evitar proporcionar información que facilite enumeración.

Por ejemplo, no debería revelar innecesariamente si:

el usuario existe;
el factor existe;
el challenge pertenece a determinado usuario.
12. Rate Limiting

El límite de intentos del challenge no reemplaza el rate limiting.

Ambos mecanismos son complementarios.

Request Rate Limit
        +
Challenge Attempt Limit
        +
Security Policy

Esto permite controlar:

cantidad de requests;
cantidad de intentos por challenge;
frecuencia por usuario;
frecuencia por IP/device/contexto cuando corresponda.

Los valores concretos de rate limiting se definirán en la arquitectura/API correspondiente.

13. Brute Force

El sistema deberá proteger contra brute force en varios niveles.

Nivel 1 — Challenge
max_attempts
Nivel 2 — Usuario

Control de comportamiento anómalo por cuenta.

Nivel 3 — Request

Rate limiting.

Nivel 4 — Contexto

IP/device/session/risk cuando la política lo requiera.

Nivel 5 — Security Policy

Puede producir:

DENY
CHALLENGE
REAUTHENTICATE
REQUIRE_MFA
REVOKE
14. Bloqueo

Cuando el challenge alcance su límite:

PENDING
   ↓
FAILED

No deberá volver a PENDING.

Para continuar, deberá generarse un nuevo challenge si la política lo permite.

Ejemplo:

Challenge A
   ↓
FAILED

Create Challenge B
   ↓
PENDING

Esto evita reiniciar simplemente el contador.

15. Expiración

La expiración debe considerarse una transición de dominio.

PENDING → EXPIRED

No se debe interpretar un challenge expirado como un error técnico.

Es un estado válido del ciclo de vida.

Los challenges expirados deben quedar disponibles para:

auditoría;
métricas;
investigación de seguridad;
análisis de comportamiento.
16. Limpieza de challenges

La expiración lógica y la eliminación física son conceptos diferentes.

Esto mantiene coherencia con:

ADR-IDENTITY-013 — Data Retention & Lifecycle

Un challenge puede estar:

EXPIRED

sin que eso signifique:

DELETE inmediatamente

La eliminación/retención física seguirá la política centralizada de retención.

17. Verificación concurrente

La verificación MFA debe ser concurrency-safe.

Escenario:

Request A ──┐
            ├── Challenge
Request B ──┘

Ambos requests podrían intentar verificar simultáneamente.

El sistema debe garantizar que:

solo una transición válida

pueda producir el resultado final.

Por ejemplo:

PENDING
   │
   └── Request A → VERIFIED

Request B
   ↓
Challenge already consumed
   ↓
DENY
18. Atomicidad

La verificación debe realizarse dentro de una operación transaccional apropiada.

Conceptualmente:

BEGIN
   │
   ├── validate challenge
   ├── validate status
   ├── validate expiration
   ├── validate attempts
   ├── verify factor
   ├── consume challenge
   └── record result
COMMIT

No debe existir una ventana donde dos requests puedan consumir el mismo challenge.

Esto deberá alinearse con:

ADR-IDENTITY-016 — Concurrency & Transaction Boundaries

19. MFA y Step-Up

Cuando MFA se utiliza para step-up:

Authenticated Session
       │
       ▼
Sensitive Operation
       │
       ▼
REQUIRE_MFA
       │
       ▼
MFA Challenge
       │
       ▼
VERIFIED

La evidencia de MFA deberá estar asociada al contexto correspondiente.

No deberá convertirse automáticamente en:

MFA válido para siempre
20. Duración de evidencia Step-Up

La evidencia de step-up tendrá una duración limitada.

Conceptualmente:

MFA Verified
      │
      ▼
Step-Up Valid
      │
      ▼
Step-Up Expired

El TTL exacto queda pendiente.

Security Policy deberá poder determinar si una operación acepta una evidencia MFA reciente o requiere una nueva verificación.

21. Factor Change

El cambio de factor tendrá un nivel de seguridad superior al login normal.

Ejemplo:

Remove Existing Factor
        │
        ▼
Security Policy
        │
        ├── REAUTHENTICATE
        └── REQUIRE_MFA

Si el usuario no puede demostrar suficiente confianza:

DENY

No deberá permitirse convertir el proceso de recuperación en un mecanismo silencioso para eliminar MFA.

22. Recovery

La recuperación se considera un flujo de alta sensibilidad.

Debe existir una separación clara:

Normal Authentication
        ≠
MFA Recovery

La recuperación no debe convertirse en:

"no tengo MFA → entonces desactivo MFA"

Debe existir una política específica.

23. Recovery Codes

Los Recovery Codes serán una de las estrategias principales de recuperación contempladas por la arquitectura.

Cada código:

es de uso único;
debe almacenarse de forma protegida;
debe quedar marcado como utilizado;
no debe volver a aceptarse;
debe quedar auditado.

Flujo:

Recovery Code
      │
      ▼
Validate
      │
      ▼
Consume Atomically
      │
      ▼
Recovery Granted
24. Uso concurrente de Recovery Code

La utilización de un recovery code debe ser atómica.

Escenario:

Request A ──┐
            ├── Recovery Code
Request B ──┘

Resultado obligatorio:

A → SUCCESS
B → DENY

Nunca:

A → SUCCESS
B → SUCCESS

La implementación debe utilizar mecanismos transaccionales/concurrency-safe.

25. Recovery Codes y regeneración

Cuando se regeneren recovery codes:

Old Set
   ↓
Revoked
   ↓
New Set

Los códigos anteriores no deberán permanecer simultáneamente utilizables salvo que una política explícita lo permita.

La estrategia preferida será invalidar el conjunto anterior.

26. Recovery después de pérdida de factores

Si un usuario pierde todos sus factores:

No MFA Factor
      ↓
Recovery Request
      ↓
Security Policy
      ↓
Additional Verification
      ↓
Recovery Flow

El flujo deberá poder requerir:

identidad adicional;
reautenticación;
recovery code;
trusted device;
soporte administrativo;
otros controles.

La combinación exacta queda pendiente de la política de recuperación.

27. Trusted Device como recuperación

Un Trusted Device podrá ayudar a recuperar acceso cuando la política lo permita.

Pero:

Un Trusted Device no debe ser automáticamente equivalente a un factor MFA permanente.

Debe existir validación de:

estado;
expiración;
revocación;
contexto;
riesgo;
política.
28. Recuperación administrativa

EVOXA deberá contemplar la posibilidad de recuperación asistida por un actor autorizado.

Por ejemplo:

User
  │
  ▼
Recovery Request
  │
  ▼
Authorized Administrator
  │
  ▼
Security Policy
  │
  ▼
Recovery

Una recuperación administrativa deberá:

requerir permisos específicos;
quedar auditada;
identificar al actor;
registrar motivo;
aplicar tenant isolation;
poder requerir step-up/MFA al administrador.
29. AI y Recovery

Los AI Agents no podrán:

desactivar MFA;
emitir recovery codes;
revelar recovery codes;
modificar factores;
aprobar recuperación;
saltarse políticas.

Un agente podrá iniciar una operación únicamente si posee autorización explícita.

30. Recovery y Tenant Isolation

Una recuperación debe respetar completamente el aislamiento de tenants.

Una operación de recuperación no debe permitir que un actor autorizado en:

Organization A

modifique factores de:

Organization B

salvo que exista una capacidad explícitamente definida a nivel SYSTEM y autorizada por política.

31. Seguridad ante abuso de Recovery

Recovery debe tener controles contra:

enumeración de usuarios;
abuso de códigos;
ataques automatizados;
social engineering;
múltiples solicitudes;
cambios repetidos de factores;
creación/eliminación masiva de dispositivos.

Eventos sospechosos deberán alimentar Security Events.

32. MFA Challenge y Session

Cuando un challenge esté asociado a una sesión:

MFAChallenge
      │
      └── Session

La sesión debe permanecer válida para el propósito correspondiente.

Si la sesión es revocada mientras existe un challenge pendiente:

Session REVOKED
      ↓
Challenge
      ↓
REVOKED

o deberá ser rechazado al intentar verificarlo.

El comportamiento exacto puede implementarse mediante transición explícita o validación de sesión en tiempo de consumo, pero nunca debe permitir que una sesión revocada obtenga una nueva confianza.

33. MFA Challenge y User State

Los cambios en el estado del usuario deben afectar la validez del challenge.

Ejemplo:

User ACTIVE
    ↓
Challenge PENDING
    ↓
User SUSPENDED

Posteriormente:

Verify Challenge
    ↓
DENY

La verificación debe comprobar el estado actual del usuario.

34. MFA Challenge y Organization

Cuando el challenge dependa de contexto organizacional:

User
 │
 └── Membership
       │
       └── Organization

la organización debe seguir siendo válida.

Una organización suspendida no debe permitir que un challenge existente sea utilizado para recuperar acceso efectivo al tenant.

35. Auditoría

Se deberán registrar eventos como:

MFAChallengeCreated
MFAChallengeVerified
MFAChallengeFailed
MFAChallengeExpired
MFAChallengeRevoked

RecoveryCodeUsed
RecoveryCodesGenerated
RecoveryCodesRevoked

MFARecoveryRequested
MFARecoveryApproved
MFARecoveryDenied

TrustedDeviceRegistered
TrustedDeviceRevoked

Los eventos deben incluir contexto suficiente para investigación sin revelar secretos.

36. Security Events

Eventos especialmente relevantes:

MFABruteForceDetected
MFAChallengeReplayDetected
MFARecoveryAbuseDetected
MFAFactorChanged
MFAFactorRevoked
TrustedDeviceRevoked

La severidad deberá ser determinada por Security Policy/Event Classification.

37. No almacenar secretos en eventos

Los eventos nunca deberán incluir:

TOTP secrets;
recovery code values;
OTP values;
private keys;
authentication credentials.

Debe utilizarse referencia:

factor_id
challenge_id
user_id

en lugar del secreto.

38. Proveedores externos

Para SMS, Email, Push u otros mecanismos externos:

MFA Domain
      │
      ▼
Provider Port
      │
      ▼
External Provider

Un fallo del proveedor debe producir:

UNAVAILABLE / FAILED

y nunca:

ALLOW

por defecto.

39. Reintentos

Un fallo temporal de proveedor podrá ser reintentado bajo una política controlada.

Pero no debe confundirse:

Provider Retry

con:

MFA Attempt

Un reintento técnico para enviar un challenge no necesariamente representa un intento de autenticación.

Esto deberá mantenerse separado para evitar que los mecanismos operacionales alteren incorrectamente la seguridad.

40. Backoff

Los reintentos de proveedores deberán utilizar backoff y límites.

Conceptualmente:

Attempt 1
   ↓
wait
   ↓
Attempt 2
   ↓
backoff
   ↓
Attempt 3
   ↓
Stop / DLQ / failure

Los valores concretos se definirán en la infraestructura correspondiente.

41. Métricas

Se deberán monitorizar métricas como:

mfa_challenge_created
mfa_challenge_verified
mfa_challenge_failed
mfa_challenge_expired
mfa_challenge_replayed
mfa_challenge_rate_limited

mfa_recovery_requested
mfa_recovery_success
mfa_recovery_denied
mfa_recovery_abuse

recovery_code_used
recovery_code_invalid
recovery_code_replay

Las métricas no deben contener secretos.

42. Alertas

Podrán generarse alertas ante:

aumento anormal de MFA failures;
múltiples challenges por usuario;
múltiples recovery attempts;
recovery desde contextos de alto riesgo;
replay;
abuso de trusted devices;
fallo masivo de proveedor.

Las reglas concretas pertenecerán a la arquitectura de observabilidad/security operations.

43. Persistencia

Las estructuras relacionadas serán:

mfa_challenges
recovery_codes
trusted_devices

y deberán integrarse con:

users
sessions
mfa_factors
audit_events
security_events
outbox_events

La definición física queda en:

ADR-IDENTITY-012 — PostgreSQL Schema

y:

IS-IDENTITY-015 — Identity Persistence

44. Retención

Los estados:

VERIFIED
FAILED
EXPIRED
REVOKED

no significan automáticamente eliminación física.

La retención deberá seguir:

ADR-IDENTITY-013 — Data Retention & Lifecycle

La retención deberá considerar:

seguridad;
auditoría;
privacidad;
investigación;
volumen;
cumplimiento;
capacidad de almacenamiento.
45. Performance

El diseño de MFA debe respetar los objetivos generales de Identity.

Las operaciones críticas deberán evitar:

transacciones largas;
locks innecesarios;
llamadas externas dentro de transacciones;
dependencia innecesaria de Redis;
operaciones costosas durante cada verificación.

Las operaciones externas deberán estar desacopladas cuando sea posible.

46. Redis

Redis podrá utilizarse para:

rate limiting;
counters temporales;
caches;
throttling;
datos efímeros.

Pero:

Redis no será la única fuente de verdad para el consumo de un MFA Challenge crítico.

La garantía de consumo único debe descansar sobre almacenamiento durable y mecanismos transaccionales apropiados.

Esto mantiene coherencia con:

ADR-IDENTITY-006 — Persistence Strategy

y:

ADR-IDENTITY-016 — Concurrency & Transaction Boundaries

47. Fail Closed

Ante incertidumbre de seguridad:

Unknown
   ↓
DENY

Ejemplos:

challenge inexistente;
challenge expirado;
session inválida;
factor revocado;
usuario suspendido;
organización suspendida;
estado inconsistente;
proveedor no confirma una verificación requerida.

Nunca:

verification uncertainty → ALLOW
48. Consecuencias positivas

La decisión proporciona:

Seguridad
TTL controlado;
protección contra brute force;
replay prevention;
consumo único;
recovery controlado;
concurrencia segura;
fail closed.
Arquitectura
comportamiento centralizado;
políticas configurables;
separación entre factor y challenge;
integración limpia con Security Policy.
Operaciones
métricas;
auditoría;
investigación;
detección de abuso.
49. Consecuencias negativas

Introduce mayor complejidad en:

almacenamiento;
concurrencia;
rate limiting;
recovery;
proveedores externos;
políticas;
testing.

También requiere coordinación entre:

MFA
Authentication
Session
Authorization
Security Policy
Audit
Events
Persistence
Observability
50. Alternativas consideradas
Alternativa A — Challenge sin expiración

Rechazada.

Generaría una ventana de ataque innecesariamente grande.

Alternativa B — TTL fijo para todos los challenges

Rechazada.

No permite adaptar seguridad según:

propósito;
factor;
riesgo;
política.
Alternativa C — Intentos ilimitados

Rechazada.

Permitiría brute force.

Alternativa D — Recovery desactiva automáticamente MFA

Rechazada.

Representaría un bypass crítico de seguridad.

Alternativa E — Redis como única fuente para challenges

Rechazada.

No ofrece por sí solo la estrategia durable requerida para garantías críticas de Identity.

51. No negociables
Todo challenge debe tener expiración.
Challenge expirado no puede verificarse.
Challenge verificado no puede reutilizarse.
Debe existir protección contra brute force.
Debe existir límite de intentos para mecanismos basados en códigos.
Recovery Codes son de uso único.
Recovery Code usado no puede reutilizarse.
La verificación concurrente debe ser segura.
MFA no puede convertirse en bypass de Authorization.
Recovery no puede convertirse en bypass de MFA.
Trusted Device no equivale automáticamente a MFA permanente.
MFA Secrets nunca deben aparecer en logs/events.
AI Agents no pueden acceder a secretos MFA.
Fallos de proveedores nunca deben producir ALLOW.
Debe aplicarse tenant isolation.
Debe existir auditoría de operaciones sensibles.
Los cambios de factor deben estar protegidos.
El sistema debe fail closed ante incertidumbre de seguridad.
Redis no es autoridad única para garantías críticas.
Los valores numéricos de TTL/attempts deben estar gobernados por configuración/política, no dispersos en código.
52. Decisiones pendientes

Quedan explícitamente pendientes:

Challenge TTL
TTL LOGIN;
TTL STEP-UP;
TTL ENROLLMENT;
TTL FACTOR_CHANGE;
TTL SENSITIVE_OPERATION;
TTL RECOVERY.
Attempts
máximo de intentos por factor;
máximo por challenge;
bloqueo temporal;
bloqueo de cuenta;
escalamiento de riesgo.
Rate Limiting
por usuario;
IP;
dispositivo;
tenant;
factor;
endpoint.
Step-Up
TTL de evidencia;
operaciones sensibles;
niveles de assurance.
Recovery
número de recovery codes;
políticas de pérdida;
recuperación administrativa;
verificación adicional;
aprobación múltiple cuando corresponda.
Trusted Device
TTL;
renovación;
binding;
riesgo.
Proveedores
timeout;
retries;
backoff;
circuit breaker;
fallback.

Estos valores no se deben inventar durante la implementación. Deberán definirse mediante especificaciones y políticas aprobadas.

53. Criterios de aceptación arquitectónica

Este ADR podrá considerarse satisfecho cuando:

 Todos los MFA Challenges tengan created_at y expires_at.
 Exista una transición explícita a EXPIRED.
 Exista límite de intentos.
 Exista protección contra brute force.
 Exista replay prevention.
 Un challenge verificado no pueda reutilizarse.
 La verificación sea concurrency-safe.
 Recovery Codes sean de uso único.
 La regeneración invalide adecuadamente códigos anteriores.
 Exista estrategia para pérdida de factores.
 Trusted Devices estén integrados con Recovery/Policy.
 Step-Up tenga evidencia temporal.
 MFA se integre con Session.
 MFA se integre con Security Policy.
 MFA se integre con Authorization.
 MFA se integre con Audit/Security Events.
 Tenant isolation esté garantizado.
 Redis no sea autoridad única para garantías críticas.
 Exista fail-closed.
 Existan métricas y alertas.
 Existan decisiones pendientes documentadas.
54. Trazabilidad
Blueprint
   │
   └── Identity
        └── MFA
Engineering Specification
ESP-0001 — Identity
Implementation Stories
IS-IDENTITY-005 — Authentication
IS-IDENTITY-008 — Session Management
IS-IDENTITY-009 — Refresh Token Lifecycle
IS-IDENTITY-010 — Multi-Factor Authentication
IS-IDENTITY-011 — Security Policy Engine
IS-IDENTITY-012 — Audit & Security Events
IS-IDENTITY-013 — Domain Events & Outbox
IS-IDENTITY-015 — Identity Persistence
IS-IDENTITY-016 — Identity Testing & Quality
ADRs relacionados
ADR-IDENTITY-005  Authorization
ADR-IDENTITY-006  Persistence
ADR-IDENTITY-007  Events & Outbox
ADR-IDENTITY-013  Data Retention
ADR-IDENTITY-014  Encryption & Secret Protection
ADR-IDENTITY-016  Concurrency
ADR-IDENTITY-020  Event Schema Compatibility
ADR-IDENTITY-022  Consumer Idempotency
ADR-IDENTITY-023  MFA Factor Strategy
55. Siguiente ADR

La secuencia queda:

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

ADR-IDENTITY-025 será especialmente importante porque deberá cerrar la estrategia criptográfica y de concurrencia de los Refresh Tokens, complementando IS-IDENTITY-009 y los ADR-014/016.
