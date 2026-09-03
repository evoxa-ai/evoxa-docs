ADR-IDENTITY-025 — Refresh Token Cryptography & Concurrency

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-03
Dominio: Identity
Prioridad: Critical
Tipo: Architecture Decision Record

1. Título

ADR-IDENTITY-025 — Refresh Token Cryptography & Concurrency

Define la estrategia arquitectónica para la protección criptográfica, almacenamiento, rotación, concurrencia, reutilización y revocación de los Refresh Tokens de EVOXA.

Este ADR complementa:

ADR-IDENTITY-002 — JWT Strategy
ADR-IDENTITY-003 — Stateful Sessions & Refresh Tokens
ADR-IDENTITY-008 — Cryptographic Key Management
ADR-IDENTITY-014 — Encryption & Secret Protection
ADR-IDENTITY-016 — Concurrency & Transaction Boundaries
IS-IDENTITY-009 — Refresh Token Lifecycle
IS-IDENTITY-025? No se crea una nueva IS; la implementación corresponde principalmente a IS-IDENTITY-009, IS-IDENTITY-015 e IS-IDENTITY-016.
2. Contexto

EVOXA utilizará:

Access Token
+
Refresh Token
+
Stateful Session

El Access Token será un JWT de corta duración y el Refresh Token será una credencial persistente y revocable asociada a una sesión.

El modelo conceptual establecido anteriormente es:

Session
   │
   └── Refresh Token Family
           │
           ├── Token 1
           ├── Token 2
           ├── Token 3
           └── Token N

Cada renovación genera un nuevo Refresh Token y debe invalidar el anterior.

Esto introduce dos problemas críticos:

¿Cómo protegemos criptográficamente los Refresh Tokens?
¿Qué ocurre cuando dos requests intentan utilizar simultáneamente el mismo token?
3. Problema

Un Refresh Token tiene una vida considerablemente mayor que un Access Token y, por lo tanto, representa una credencial de alto valor.

Si un atacante obtiene un Refresh Token válido podría intentar:

generar nuevos Access Tokens;
mantener acceso;
reutilizar tokens antiguos;
competir con el usuario legítimo durante la rotación;
explotar condiciones de carrera;
utilizar tokens revocados;
realizar replay.

Además, una implementación incorrecta de la rotación puede provocar:

Request A → refresh
Request B → refresh

y terminar permitiendo dos rotaciones válidas sobre el mismo token.

Por ello, la criptografía y la concurrencia deben diseñarse conjuntamente.

4. Decisión

EVOXA adoptará una estrategia de:

Refresh Tokens opacos, persistidos de forma protegida, rotación obligatoria, token families, detección de reuse y operaciones de refresh concurrency-safe.

El flujo será:

Client
  │
  │ Refresh Token
  ▼
Identity
  │
  ├── Validate token
  ├── Validate session
  ├── Validate user
  ├── Validate membership/org
  ├── Validate policy
  │
  ▼
Atomic Rotation
  │
  ├── Old Token → USED
  └── New Token → ACTIVE
  │
  ▼
New Access Token
+
New Refresh Token
5. Refresh Token como credencial opaca

El Refresh Token no será un JWT de larga duración por defecto.

Será tratado como un bearer credential opaco.

Conceptualmente:

Client
   │
   ▼
Random Refresh Token
   │
   ▼
Protected representation
   │
   ▼
Database

Esto permite que el servidor controle:

estado;
revocación;
rotación;
familias;
reuse detection;
sesiones;
dispositivos.
6. Entropía criptográfica

El Refresh Token deberá generarse mediante un generador criptográficamente seguro.

No deberá utilizar:

Math.random;
timestamps;
IDs predecibles;
UUIDs no adecuados como secreto;
información del usuario;
información de la sesión;
datos derivados predecibles.

El material utilizado deberá tener suficiente entropía para impedir guessing attacks.

El tamaño/representación exactos quedan pendientes de la especificación criptográfica de implementación.

7. Hashing vs Encryption

La estrategia distingue dos conceptos:

Hashing

Cuando el servidor solo necesita comprobar que el token presentado corresponde al token almacenado.

Refresh Token
      ↓
Secure Hash / Representation
      ↓
Database

Esto evita tener que almacenar el secreto recuperable.

Encryption

Se utilizará cuando exista un requisito legítimo de recuperar el valor original.

Para Refresh Tokens, la decisión arquitectónica preferente será:

No almacenar el Refresh Token en texto plano ni requerir su recuperación desde la base de datos.

La estrategia concreta de hashing/HMAC se determinará bajo:

ADR-IDENTITY-014 — Encryption & Secret Protection

y la especificación criptográfica correspondiente.

8. No almacenar Refresh Tokens en texto plano

La base de datos no deberá contener:

refresh_token = "valor-secreto-original"

Debe existir una representación protegida.

Conceptualmente:

token_hash

o mecanismo equivalente aprobado.

Esto reduce el impacto de una filtración de la base de datos.

9. Pepper / Secret adicional

La implementación podrá utilizar un secreto adicional gestionado externamente para reforzar la protección de los tokens.

Si se adopta:

Refresh Token
     +
Server Secret
     ↓
Protected Representation

el secreto no deberá almacenarse en PostgreSQL.

Deberá estar administrado mediante:

KMS;
Secret Manager;
mecanismo equivalente.

Esta decisión concreta queda pendiente de la implementación criptográfica definitiva.

10. Identidad del token

Cada Refresh Token tendrá una identidad propia.

Conceptualmente:

RefreshToken
├── id
├── family_id
├── parent_token_id
├── token_hash
├── status
├── issued_at
├── expires_at
├── used_at
├── revoked_at
└── metadata

Debe distinguirse:

token_id
family_id
session_id
user_id

No representan la misma cosa.

11. Token Family

Una Token Family representa la cadena de rotaciones originada a partir de una sesión/autenticación.

Ejemplo:

Family F1

Token A
   │
   ▼
Token B
   │
   ▼
Token C
   │
   ▼
Token D

Estados:

A = USED
B = USED
C = USED
D = ACTIVE

La familia permite detectar comportamiento anómalo cuando un token antiguo vuelve a utilizarse.

12. Parent Token

Cada token rotado podrá mantener referencia a su token anterior:

Token B
parent = Token A

Token C
parent = Token B

Esto permite mantener lineage.

No debe utilizarse la lineage como único mecanismo de seguridad; el estado persistente y la política de reuse detection siguen siendo necesarios.

13. Estados

El ciclo conceptual será:

ACTIVE
   │
   ├──► USED
   ├──► EXPIRED
   └──► REVOKED

Un token USED no vuelve a ACTIVE.

Un token EXPIRED no vuelve a ACTIVE.

Un token REVOKED no vuelve a ACTIVE.

14. Rotación obligatoria

Cada refresh válido deberá producir:

Old Refresh Token
       ↓
USED

New Refresh Token
       ↓
ACTIVE

Por tanto:

Refresh A
   ↓
Access B
Refresh B

y no:

Refresh A
   ↓
Access B

Refresh A
   ↓
Access C

El segundo caso representaría reutilización del token anterior.

15. Atomicidad de la rotación

La rotación debe ser una operación atómica.

Conceptualmente:

BEGIN TRANSACTION

1. Locate token
2. Lock/validate current state
3. Validate session
4. Validate user
5. Validate organization/membership
6. Validate expiration
7. Validate security policy
8. Mark old token USED
9. Create new token ACTIVE
10. Persist required state/events

COMMIT

No debe existir un estado intermedio observable donde:

Old Token = ACTIVE
New Token = ACTIVE

como resultado de una misma rotación.

16. Concurrencia

Este es uno de los puntos más críticos del ADR.

Escenario:

                   ┌── Request A ──┐
Refresh Token ─────┤               ├── Identity
                   └── Request B ──┘

Ambos requests pueden llegar prácticamente al mismo tiempo.

La arquitectura debe garantizar:

Token A
   │
   ├── Request A → SUCCESS
   │
   └── Request B → REJECT / REUSE / CONFLICT

No:

Request A → SUCCESS
Request B → SUCCESS

sobre el mismo token.

17. Estrategia de locking

La implementación podrá utilizar mecanismos de concurrencia de PostgreSQL como:

row-level locking;
optimistic concurrency;
unique constraints;
estado condicional;
transacciones.

La elección exacta se definirá en la implementación y deberá cumplir ADR-IDENTITY-016.

No se utilizará un lock distribuido global como mecanismo principal.

18. Optimistic Concurrency

El modelo deberá permitir detectar que el token ya cambió de estado.

Conceptualmente:

UPDATE refresh_tokens
SET status = USED
WHERE id = ?
AND status = ACTIVE

Si:

rows_updated = 1

la transición fue obtenida.

Si:

rows_updated = 0

otro proceso ya consumió/modificó el token.

La implementación final podrá utilizar una variante equivalente.

19. Pessimistic Locking

Para operaciones especialmente sensibles podrá utilizarse locking pesimista.

Conceptualmente:

SELECT ...
FOR UPDATE

El lock deberá ser:

localizado;
corto;
dentro de una transacción;
liberado rápidamente.

No se deben mantener locks mientras se realizan llamadas externas.

20. No llamadas externas dentro de la transacción

La transacción crítica no debe contener llamadas prolongadas a:

proveedores externos;
APIs;
servicios de terceros;
sistemas AI;
email;
SMS.

Preferentemente:

DB Transaction
     │
     └── Commit
           ↓
External Side Effects

Esto reduce:

lock duration;
deadlocks;
latencia;
probabilidad de timeout.
21. Reuse Detection

Si un Refresh Token previamente marcado como USED vuelve a presentarse:

Token A
status = USED
       │
       ▼
Presented again
       │
       ▼
Reuse Detected

debe considerarse un evento de seguridad potencialmente grave.

22. Respuesta ante reuse

La política previamente establecida contempla:

USED token reused
       ↓
Security Event
       ↓
Audit
       ↓
Revoke Token Family
       ↓
Revoke Session

La familia completa podrá quedar invalidada.

Conceptualmente:

Family F1

A USED
B USED
C USED
D ACTIVE

A reused
   ↓
F1 REVOKED

Esto reduce el riesgo de que un atacante mantenga acceso mediante otro token de la misma familia.

23. Reuse vs Network Retry

Existe un caso legítimo que debe distinguirse:

Client
  │
  ├── Refresh request
  │
  └── Server succeeds
        │
        └── Response lost

El cliente puede repetir el request.

El servidor podría interpretar el token como USED.

Por ello, la implementación deberá distinguir cuidadosamente entre:

Replay potencialmente malicioso

y:

Retry legítimo debido a pérdida de respuesta.

La estrategia exacta queda pendiente.

No se deberá implementar una regla simplista que necesariamente destruya la sesión ante cualquier segundo request sin considerar el contexto definido por seguridad.

24. Idempotencia

La API de refresh deberá considerar el problema de retries.

El objetivo será:

Retry seguro

sin permitir:

Replay indefinido

Podrán evaluarse mecanismos como:

request idempotency;
refresh operation identifiers;
short-lived rotation grace;
resultado de rotación asociado;
client-side coordination.

La estrategia definitiva deberá mantener el principio:

La comodidad ante retries nunca debe convertirse en reutilización indefinida del Refresh Token.

25. Ventana de Grace

Podrá evaluarse una ventana extremadamente limitada para manejar determinados retries legítimos.

Si se adopta, deberá:

estar explícitamente definida;
tener duración mínima necesaria;
estar vinculada a una operación concreta;
no permitir múltiples rotaciones;
estar protegida contra replay.

No se considera obligatorio introducir una grace window.

Decisión pendiente.

26. Validaciones durante Refresh

Antes de emitir nuevos tokens se deberán validar:

Refresh Token
    │
    ├── Token status
    ├── Token expiration
    ├── Token family
    ├── Session status
    ├── User status
    ├── Membership status
    ├── Organization status
    └── Security Policy

Por tanto, poseer físicamente el Refresh Token no es suficiente.

27. User Suspension

Si el usuario pasa a:

SUSPENDED

los Refresh Tokens no deben continuar renovando sesiones normalmente.

Conceptualmente:

User SUSPENDED
      ↓
Refresh
      ↓
DENY

La política de revocación inmediata de tokens existentes podrá ejecutarse además mediante eventos.

28. Organization Suspension

Si una organización está suspendida:

Organization SUSPENDED
      ↓
Refresh for organization
      ↓
DENY

La validación deberá respetar el contexto organizacional de la sesión.

Esto evita que un token previamente emitido mantenga acceso efectivo a un tenant suspendido.

29. Membership Removal

Si la membership correspondiente pasa a:

REMOVED

el Refresh Token asociado a esa sesión deberá dejar de ser utilizable para acceder al contexto de esa organización.

30. Session Revocation

Si:

Session = REVOKED

el Refresh Token asociado debe dejar de ser válido.

Session REVOKED
      ↓
Refresh
      ↓
DENY

Esto mantiene la relación:

Session
   └── Refresh capability
31. Logout
Logout actual

Debe revocar:

Current Session
+
Refresh capability
Logout All

Debe revocar las sesiones/tokens correspondientes al usuario según la política.

Ejemplo:

User
 ├── Session A → revoke
 ├── Session B → revoke
 ├── Session C → revoke
 └── Session D → revoke
32. Access Token después de revocación

El Access Token es stateless y puede continuar siendo técnicamente válido hasta su expiración.

Por ello:

Refresh revoked
       ↓
No new Access Tokens

pero:

Existing Access Token
       ↓
may remain valid until expiration

salvo que EVOXA implemente posteriormente una estrategia de revocación inmediata.

Esto es coherente con:

ADR-IDENTITY-002

y:

ADR-IDENTITY-003.

33. TTL

La relación definida previamente es:

Access Token TTL
    <
Refresh Token TTL
    ≤
Session Maximum Lifetime

Los valores concretos no se fijan en este ADR.

Deberán definirse mediante:

configuración;
Security Policy;
requisitos de producto;
análisis de riesgo.
34. Expiración

Un Refresh Token debe pasar a:

EXPIRED

cuando alcance su expiración.

No debe poder utilizarse posteriormente.

La expiración lógica y la eliminación física siguen siendo conceptos distintos.

35. Revocación de familia

Una familia puede revocarse por:

reuse detection;
logout;
security incident;
user suspension;
administrative action;
policy;
session revocation.

Ejemplo:

Token Family
     ↓
REVOKED

Todos los tokens de la familia deben quedar efectivamente inutilizables.

36. Cryptographic Key Management

La estrategia criptográfica deberá integrarse con:

ADR-IDENTITY-008 — Cryptographic Key Management

y:

ADR-IDENTITY-014 — Encryption & Secret Protection

Los secretos utilizados para proteger tokens no deben estar:

en Git;
en código;
en imágenes;
en .env versionados;
en logs;
en eventos.

Deben administrarse mediante infraestructura segura.

37. Refresh Token y JWT Signing Keys

Debe existir separación entre:

JWT Signing Keys

y:

Refresh Token Protection Secrets

No debe asumirse que ambos utilizan exactamente la misma clave.

La arquitectura deberá permitir rotación independiente.

38. Rotación criptográfica

Cuando se roten secretos criptográficos, el sistema debe permitir una transición controlada.

No se debe provocar automáticamente:

Key rotation
      ↓
Logout all users

salvo que exista una razón de seguridad que lo requiera.

La estrategia de transición depende del mecanismo criptográfico definitivo.

39. Client Storage

Se mantiene la decisión de:

Web

Refresh Token:

HttpOnly
+
Secure
+
appropriate SameSite policy

preferentemente en cookie.

Mobile

Refresh Token:

OS Secure Storage

Nunca:

localStorage
URL
logs
plain text files

Esto sigue:

ADR-IDENTITY-009 — Token & Client Storage.

40. Tenant Isolation

El Refresh Token debe estar asociado al contexto adecuado.

Una request con un token perteneciente a:

Organization A

no puede convertirse en una sesión válida para:

Organization B

mediante manipulación del request.

El servidor debe obtener el contexto desde la sesión/token persistido y validarlo.

41. No confiar en datos del cliente

El cliente no puede definir libremente:

user_id
organization_id
membership_id
session_id
role

durante el refresh para alterar el contexto.

La autoridad será el estado persistido de Identity.

42. Auditoría

Operaciones relevantes:

RefreshTokenCreated
RefreshTokenRotated
RefreshTokenRevoked
RefreshTokenExpired
RefreshTokenReuseDetected
RefreshTokenFamilyRevoked

deberán integrarse con:

IS-IDENTITY-012 — Audit & Security Events

No se deberá registrar el token real.

43. Security Events

Especialmente importante:

RefreshTokenReuseDetected

Debe poder activar:

revocación de familia;
revocación de sesión;
alertas;
investigación;
análisis de riesgo.

La severidad concreta deberá ser definida por Security Policy.

44. Eventos de dominio

Los eventos de Refresh Token seguirán la estrategia definida por:

ADR-IDENTITY-018 — Event Broker

ADR-IDENTITY-019 — Event Partitioning

ADR-IDENTITY-020 — Event Schema Compatibility

ADR-IDENTITY-021 — Event Replay

ADR-IDENTITY-022 — Consumer Idempotency

Los eventos deberán ser:

versionados;
inmutables;
tenant-aware;
idempotentemente consumibles.
45. No incluir el token en eventos

Nunca:

payload.refresh_token

Debe utilizarse:

token_id
session_id
family_id
user_id
tenant_id

cuando sea necesario.

46. Persistencia

La estructura conceptual:

refresh_tokens

deberá soportar:

id
session_id
family_id
parent_token_id
token_hash
status
issued_at
expires_at
used_at
revoked_at
revoke_reason
created_at
metadata

Los nombres y tipos definitivos pertenecen a:

ADR-IDENTITY-012 — PostgreSQL Schema

y:

IS-IDENTITY-015 — Identity Persistence.

47. Índices

Los índices deberán diseñarse a partir de los patrones reales de acceso.

Como mínimo deberán evaluarse consultas por:

token identity / protected representation
session_id
family_id
status
expires_at

No se debe crear un índice indiscriminadamente sobre cada campo.

48. Constraints

Deberán existir restricciones que ayuden a garantizar:

identidad única;
relaciones válidas;
estados válidos;
integridad de sesión;
lineage;
concurrencia.

Las restricciones físicas definitivas serán definidas en Persistence.

49. Transaction + Outbox

Cuando una rotación produzca eventos relevantes, el estado transaccional y el outbox deberán seguir:

BEGIN
   │
   ├── Old Token → USED
   ├── New Token → ACTIVE
   ├── Session state if needed
   └── Outbox Event
   │
COMMIT

Esto evita:

DB success
+
Event lost

como dual write inconsistente.

50. Fallos
DB falla antes de commit
No rotation committed

El cliente puede reintentar según la política.

DB commit exitoso
Rotation committed

El estado persistido es autoridad.

Event Broker falla
Rotation committed
Outbox pending

No se revierte la rotación simplemente porque el broker esté temporalmente indisponible.

ACK perdido después de publicación

Puede producir:

duplicate event

Los consumidores deben ser idempotentes.

51. Observabilidad

Métricas conceptuales:

refresh_success_total
refresh_failure_total
refresh_expired_total
refresh_revoked_total
refresh_reuse_detected_total
refresh_rotation_conflict_total
refresh_latency
refresh_family_revoked_total

También:

concurrent_refresh_conflicts

para detectar problemas de clientes o infraestructura.

Nunca se debe registrar el token.

52. Seguridad operacional

Los logs podrán contener:

token_id
session_id
family_id
user_id
tenant_id
correlation_id
trace_id
result
reason

siempre que la clasificación de datos lo permita.

Nunca:

refresh_token
token_hash completo
secret
credential

si la representación pudiera permitir recuperación o abuso.

53. Testing

Este ADR exige pruebas específicas de concurrencia.

Caso normal
Refresh A
→ Success
→ New token
Token usado
Refresh A
→ USED
→ Reject
Token expirado
Refresh A
→ EXPIRED
→ Reject
Token revocado
Refresh A
→ REVOKED
→ Reject
Reuse
A USED
→ A presented again
→ Reuse Detection
→ Family Revoked
Concurrencia
100 concurrent requests
same refresh token

Debe existir como máximo una rotación válida.

54. Security Testing

Debe probarse:

token guessing;
token tampering;
replay;
race conditions;
stolen token;
revoked session;
suspended user;
suspended organization;
removed membership;
wrong tenant;
malformed token;
expired token;
invalid token representation;
database compromise assumptions;
logging leakage.
55. Performance

El refresh debe mantenerse dentro de los objetivos generales de Identity.

Se deberá evitar:

consultas innecesarias;
múltiples round-trips evitables;
locks prolongados;
llamadas externas dentro de la transacción;
dependencia de Redis para cada garantía crítica.

El objetivo de performance deberá validarse contra:

API P95 <300 ms

y los objetivos específicos de Authentication/Identity.

56. Consecuencias positivas
Seguridad
tokens no almacenados en claro;
rotación;
reuse detection;
revocación por familia;
sesiones stateful;
control centralizado.
Concurrencia
evita doble refresh;
permite detectar races;
protege contra replay.
Operación
auditoría;
observabilidad;
respuesta ante compromiso;
trazabilidad.
Evolución

La estrategia permite posteriormente implementar mecanismos más avanzados de:

detección de anomalías;
risk scoring;
device binding;
adaptive authentication.
57. Consecuencias negativas

La estrategia añade complejidad significativa:

persistencia;
transacciones;
locking;
token families;
reuse detection;
recovery de retries;
manejo de estados;
testing de concurrencia.

También requiere mayor disciplina operacional.

58. Alternativas consideradas
Alternativa A — Refresh Token como JWT stateless

Rechazada como estrategia principal.

Dificulta:

revocación;
rotation;
reuse detection;
family management;
control por sesión.
Alternativa B — Refresh Token persistido en texto plano

Rechazada.

Aumenta el impacto de una filtración de base de datos.

Alternativa C — Sin rotación

Rechazada.

Permitiría reutilización prolongada de una misma credencial.

Alternativa D — Redis como autoridad de refresh

Rechazada.

PostgreSQL es la fuente transaccional principal.

Alternativa E — Lock distribuido global

Rechazada.

Añadiría complejidad y dependencia operacional innecesaria.

La concurrencia debe resolverse principalmente mediante mecanismos transaccionales locales.

59. No negociables
Refresh Tokens son credenciales de alto valor.
No deben almacenarse en texto plano.
Deben utilizar generación criptográficamente segura.
Deben rotarse.
Deben pertenecer a una familia.
Deben soportar lineage.
Un token usado no vuelve a ser activo.
Un token expirado no vuelve a ser activo.
Un token revocado no vuelve a ser activo.
Debe existir reuse detection.
Reuse confirmado debe producir respuesta de seguridad.
La rotación debe ser concurrency-safe.
Solo una rotación válida debe ganar ante concurrencia.
Session state debe validarse.
User state debe validarse.
Organization/Membership state debe validarse cuando corresponda.
Security Policy debe poder bloquear refresh.
No se debe confiar en contexto sensible proporcionado por el cliente.
Los tokens nunca deben aparecer en logs.
Los tokens nunca deben aparecer en eventos.
Los tokens nunca deben exponerse a AI Agents.
PostgreSQL es la autoridad transaccional.
Redis no es la autoridad única.
Domain state + Outbox deben mantener atomicidad.
No deben realizarse llamadas externas largas dentro de la transacción crítica.
Logout debe revocar la capacidad de refresh.
Session revocation debe impedir nuevos refresh.
Tenant isolation es obligatorio.
La estrategia debe soportar pruebas de concurrencia.
La estrategia criptográfica definitiva debe alinearse con ADR-014.
60. Decisiones pendientes

Quedan deliberadamente abiertas:

Criptografía
algoritmo exacto de hash/HMAC;
tamaño exacto del token;
encoding;
uso de pepper;
KMS específico;
estrategia de rotación criptográfica.
Concurrencia
optimistic vs pessimistic locking por operación;
isolation level;
estrategia exacta de conflicto;
mecanismo de retry.
Retry
grace window;
request idempotency;
replay legítimo vs malicioso;
comportamiento cuando la respuesta se pierde.
TTL
Access Token TTL;
Refresh Token TTL;
Session maximum lifetime.
Reuse
severidad;
revocación automática;
alertas;
respuesta adaptativa.
Binding
device binding;
IP/context;
risk signals.

No se deben fijar estos valores arbitrariamente durante la codificación.

61. Criterios de aceptación arquitectónica

Este ADR podrá considerarse satisfecho cuando:

 Refresh Tokens sean opacos.
 Se generen criptográficamente seguros.
 No se almacenen en texto plano.
 Exista representación protegida.
 Exista token family.
 Exista parent/lineage.
 Exista rotación.
 Exista reuse detection.
 Exista revocación de familia.
 Exista validación de sesión.
 Exista validación de usuario.
 Exista validación de membership/organization.
 Exista integración con Security Policy.
 Exista protección contra concurrencia.
 Solo una rotación pueda ganar.
 Exista estrategia para retries.
 Exista estrategia de expiración.
 Exista integración con Audit/Security Events.
 Exista integración con Outbox.
 No existan secretos en logs/events.
 Exista tenant isolation.
 Existan pruebas de race conditions.
 Existan pruebas de replay.
 Existan pruebas de token compromise.
 Existan métricas y alertas.
62. Trazabilidad
Blueprint
Identity
 ├── Authentication
 ├── Session Management
 └── Security
Engineering Specification
ESP-0001 — Identity
Implementation Stories
IS-IDENTITY-005
Authentication

IS-IDENTITY-008
Session Management

IS-IDENTITY-009
Refresh Token Lifecycle

IS-IDENTITY-012
Audit & Security Events

IS-IDENTITY-013
Domain Events & Outbox

IS-IDENTITY-015
Identity Persistence

IS-IDENTITY-016
Identity Testing & Quality

IS-IDENTITY-017
Identity Runtime & Deployment
ADRs relacionados
ADR-IDENTITY-002
JWT Strategy

ADR-IDENTITY-003
Stateful Sessions & Refresh Tokens

ADR-IDENTITY-008
Cryptographic Key Management

ADR-IDENTITY-009
Token & Client Storage

ADR-IDENTITY-012
PostgreSQL Schema

ADR-IDENTITY-013
Data Retention & Lifecycle

ADR-IDENTITY-014
Encryption & Secret Protection

ADR-IDENTITY-016
Concurrency & Transaction Boundaries

ADR-IDENTITY-018
Event Broker

ADR-IDENTITY-019
Event Partitioning & Ordering

ADR-IDENTITY-020
Event Schema Compatibility

ADR-IDENTITY-022
Consumer Idempotency

ADR-IDENTITY-023
MFA Factor Strategy

ADR-IDENTITY-024
MFA Challenge TTL, Attempts & Recovery
63. Siguiente ADR

La secuencia de decisiones continúa:

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
        │
        ▼
ADR-IDENTITY-027
Runtime Configuration, Secrets & KMS
        │
        ▼
ADR-IDENTITY-028
Observability & Operational Security
        │
        ▼
ADR-IDENTITY-029
Disaster Recovery & Backup
        │
        ▼
ADR-IDENTITY-030
Autoscaling & Capacity

ADR-IDENTITY-025 cierra uno de los puntos más críticos de Identity: la renovación segura de sesiones. Una vez definido, el siguiente bloque lógico es ADR-IDENTITY-026 — API Pagination, Filtering & Idempotency, que permitirá cerrar aspectos transversales de los contratos API antes de entrar a la implementación.
