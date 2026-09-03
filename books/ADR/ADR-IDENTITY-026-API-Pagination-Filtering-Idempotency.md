ADR-IDENTITY-026 — API Pagination, Filtering & Idempotency

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-03
Dominio: Identity / API
Prioridad: Critical
Tipo: Architecture Decision Record

1. Título

ADR-IDENTITY-026 — API Pagination, Filtering & Idempotency

Define las convenciones arquitectónicas transversales para las APIs de Identity relacionadas con:

paginación;
filtrado;
ordenamiento;
búsqueda;
consistencia de resultados;
Idempotency-Key;
operaciones retryable;
prevención de efectos duplicados;
concurrencia;
propagación de correlation/trace IDs;
seguridad y tenant isolation.

Este ADR complementa directamente:

IS-IDENTITY-014 — Identity API Contracts
ADR-IDENTITY-016 — Concurrency & Transaction Boundaries
ADR-IDENTITY-022 — Consumer Idempotency & Deduplication
ADR-IDENTITY-025 — Refresh Token Cryptography & Concurrency
2. Contexto

La arquitectura API de EVOXA establece:

/api/v1

como base de versionamiento.

Las APIs deben ser:

consistentes;
seguras;
observables;
multi-tenant;
compatibles con retries;
adecuadas para clientes Web, Mobile y otros consumidores.

Identity tendrá endpoints de:

Users
Organizations
Memberships
Roles
Permissions
Sessions
MFA
Security Policies
Audit Events
Security Events

Algunos endpoints devolverán colecciones potencialmente grandes y otros ejecutarán operaciones que pueden ser reintentadas debido a:

timeouts;
pérdida de conexión;
retries automáticos;
errores de red;
fallos temporales;
clientes móviles con conectividad inestable.

Por ello, paginación e idempotencia deben formar parte de una estrategia común.

3. Problema

Sin una convención transversal podrían aparecer:

Paginación inconsistente
/users?page=1
/sessions?offset=20
/audit-events?cursor=abc

sin una semántica uniforme.

Resultados inestables

Un cambio en los datos mientras el cliente pagina podría provocar:

duplicados;
elementos omitidos;
resultados inconsistentes.
Operaciones duplicadas

Un cliente puede enviar:

POST /users

y no recibir respuesta.

El cliente reintenta:

POST /users

y termina creando dos usuarios.

Problemas de concurrencia

Dos requests podrían ejecutar simultáneamente la misma operación.

Por ello se necesita una estrategia común.

4. Decisión

EVOXA adoptará:

paginación basada en cursor como estrategia preferente para colecciones grandes o dinámicas;
offset/page únicamente cuando sea apropiado para casos simples;
ordenamiento determinista obligatorio;
filtros explícitos y allowlisted;
Idempotency-Key para operaciones mutantes retryable;
persistencia durable de la información necesaria para garantizar idempotencia;
tenant isolation aplicado antes de recuperar resultados;
correlation/trace IDs propagados durante toda la operación;
contratos uniformes definidos en OpenAPI.

La implementación concreta de cada endpoint deberá indicar cuál estrategia utiliza.

5. Paginación

Las colecciones deberán evitar respuestas ilimitadas.

Una API como:

GET /api/v1/identity/users

no deberá devolver arbitrariamente todos los registros.

Debe existir un mecanismo de paginación.

6. Cursor Pagination

Para colecciones grandes o altamente dinámicas se utilizará preferentemente:

cursor

Ejemplo conceptual:

GET /api/v1/identity/users?limit=50&cursor=ey...

Respuesta:

{
  "data": [],
  "pagination": {
    "limit": 50,
    "next_cursor": "...",
    "has_next": true
  }
}

El formato exacto del cursor será definido en el contrato API.

7. Propiedades del Cursor

El cursor deberá ser:

opaco;
no manipulable de manera útil por el cliente;
validable por servidor;
compatible con el ordenamiento utilizado;
preferentemente compacto;
sin información sensible.

El cliente no debe depender de la estructura interna del cursor.

8. Cursor y Tenant

Un cursor generado dentro de:

Organization A

no podrá utilizarse para acceder a:

Organization B

El servidor deberá validar el contexto asociado al cursor.

Conceptualmente:

Cursor
+
Authenticated Context
+
Tenant Context

deben ser compatibles.

9. Offset Pagination

Offset/page podrá utilizarse para colecciones pequeñas o escenarios donde resulte suficientemente eficiente.

Ejemplo:

GET /users?page=2&page_size=50

No será obligatorio utilizar cursor en absolutamente todos los endpoints.

Sin embargo, offset no deberá convertirse automáticamente en la estrategia para tablas de crecimiento elevado.

10. Selección de estrategia

La decisión deberá basarse en:

volumen;
crecimiento;
frecuencia de modificación;
patrón de consulta;
necesidad de navegación;
performance;
UX;
estabilidad de resultados.

Conceptualmente:

Small / Stable
     ↓
Offset acceptable

Large / Dynamic
     ↓
Cursor preferred
11. Límites

Las APIs deberán establecer:

default_limit
maximum_limit

El cliente no podrá solicitar un tamaño arbitrariamente grande.

Ejemplo conceptual:

?limit=1000000

deberá ser rechazado o limitado.

El valor exacto del límite máximo queda pendiente de validación de performance.

12. Ordenamiento

Toda colección paginada debe tener un orden determinista.

No se debe depender de:

database natural order

como comportamiento contractual.

Debe existir un orden explícito, por ejemplo conceptualmente:

created_at DESC
id DESC

El id puede actuar como desempate cuando sea necesario.

13. Ordenamiento estable

Si varios registros poseen el mismo valor de orden:

created_at

debe existir un criterio adicional para garantizar estabilidad.

Ejemplo:

ORDER BY created_at DESC, id DESC

Esto es especialmente importante para cursor pagination.

14. Sorting Allowlist

El cliente no podrá enviar arbitrariamente una expresión SQL.

Ejemplo peligroso:

?sort=raw_database_expression

La API deberá aceptar únicamente campos definidos explícitamente.

Conceptualmente:

sort=created_at
sort=email
sort=status

según el endpoint.

15. Dirección de orden

La API podrá utilizar:

sort=created_at
order=desc

o una convención equivalente.

La forma exacta será uniforme en todos los endpoints.

No deberán coexistir múltiples convenciones dentro de Identity.

16. Filtering

Los filtros deberán ser explícitos.

Ejemplo:

GET /users?status=ACTIVE

o:

GET /users?email=...

según el recurso.

Los filtros permitidos deberán formar parte del contrato OpenAPI.

17. No SQL dinámico desde el cliente

Nunca se deberá permitir que el cliente defina:

SQL;
expresiones arbitrarias;
nombres de columnas no allowlisted;
operadores no soportados;
joins arbitrarios.

La API traduce filtros conocidos a consultas internas.

18. Operadores

La arquitectura podrá soportar operadores como:

=
!=
>
>=
<
<=
contains
starts_with
in

pero únicamente cuando estén definidos para el recurso.

No todos los recursos necesitan todos los operadores.

19. Filtros multi-tenant

Los filtros proporcionados por el cliente nunca reemplazan el tenant context.

Ejemplo:

GET /users?organization_id=B

si el usuario está autorizado solamente para Organization A.

El servidor debe evaluar:

Authenticated Tenant Context
+
Authorization
+
Policy
+
Requested Filter

No confiar simplemente en organization_id enviado por el cliente.

20. Search

La búsqueda textual deberá estar limitada a campos explícitamente definidos.

Ejemplo:

GET /users?search=juan

No significa:

buscar en todas las columnas

a menos que el contrato lo establezca.

Esto permite controlar:

performance;
privacidad;
índices;
comportamiento.
21. Pagination Metadata

Las respuestas paginadas deberán tener metadata consistente.

Conceptualmente:

{
  "data": [],
  "pagination": {
    "limit": 50,
    "has_next": true,
    "next_cursor": "..."
  }
}

Para offset podrá incluirse:

{
  "page": 2,
  "page_size": 50,
  "has_next": true
}

No se debe prometer total_count si obtenerlo resulta costoso y no es necesario para el caso de uso.

22. Total Count

total_count será opcional.

Debe evaluarse:

costo de COUNT(*);
tamaño de tabla;
filtros;
necesidad real de UX.

Para grandes colecciones, la API podrá evitar calcularlo en cada request.

23. Consistencia durante paginación

La API no garantiza por defecto un snapshot transaccional completo entre todas las páginas.

Por tanto:

Request 1
↓
datos cambian
↓
Request 2

puede producir diferencias.

Cursor pagination y ordenamiento estable reducen el problema, pero no deben presentarse como snapshot global.

Si un caso de uso requiere snapshot consistente, deberá definirse explícitamente.

24. Idempotency-Key

Las operaciones mutantes retryable deberán soportar:

Idempotency-Key: <unique-key>

Especialmente:

POST;
operaciones de creación;
operaciones que producen side effects;
operaciones sensibles.

La necesidad concreta por endpoint será definida en el contrato.

25. Objetivo de Idempotency-Key

Ejemplo:

POST /users
Idempotency-Key: abc123

Primera request:

abc123 → SUCCESS

El cliente no recibe la respuesta.

Reintenta:

abc123 → same operation

El servidor deberá evitar crear otro usuario.

26. Idempotency Record

Conceptualmente deberá existir un registro:

IdempotencyRecord
├── key
├── actor
├── tenant
├── endpoint
├── operation
├── request_fingerprint
├── status
├── response_reference
├── created_at
└── expires_at

La estructura física puede residir en una tabla específica o mecanismo equivalente.

27. Scope de la Idempotency Key

Una misma key no debe ser globalmente reutilizable para cualquier operación.

La identidad lógica debe considerar como mínimo:

Idempotency-Key
+
Authenticated Actor
+
Tenant Context
+
Operation / Endpoint

Esto evita colisiones accidentales.

28. Request Fingerprint

La implementación deberá evaluar el request asociado a la key.

Ejemplo:

Primera request:

Idempotency-Key: ABC

email = user@example.com

Segunda:

Idempotency-Key: ABC

email = other@example.com

Debe producir un conflicto.

Conceptualmente:

same key
+
different request
=
IDEMPOTENCY_CONFLICT
29. Repetición idéntica

Si:

same key
+
same operation
+
same actor
+
same tenant
+
same request

el servidor deberá tratarla como retry de la misma operación.

La respuesta podrá reutilizar el resultado persistido.

30. Estado de Idempotency Record

Conceptualmente:

PENDING
   │
   ├──► COMPLETED
   ├──► FAILED
   └──► EXPIRED

La implementación puede utilizar otros estados equivalentes.

La transición debe ser concurrency-safe.

31. Concurrencia de Idempotency-Key

Escenario:

Request A ──┐
            ├── Key ABC
Request B ──┘

Debe evitarse que ambas ejecuten simultáneamente el side effect.

Conceptualmente:

Request A
   ↓
claims key
   ↓
executes operation

Request B
   ↓
sees key in progress
   ↓
wait / conflict / retry response

La política exacta de respuesta deberá definirse en API Contract.

32. Idempotencia y transacciones

Cuando la operación principal sea transaccional, el registro de idempotencia deberá coordinarse con la transacción siempre que sea posible.

Ejemplo:

BEGIN
   │
   ├── claim idempotency key
   ├── execute business operation
   ├── persist result
   └── outbox
COMMIT

Esto reduce la posibilidad de:

business success
+
idempotency record lost
33. Idempotencia y Outbox

Si una operación produce:

Domain Change
+
Domain Event

debe mantenerse:

Business State
+
Outbox
+
Idempotency State

de forma coherente cuando el caso de uso lo requiera.

Esto debe alinearse con:

ADR-IDENTITY-007

y:

IS-IDENTITY-013.

34. Idempotencia no significa exactamente-once

La Idempotency-Key no convierte el sistema entero en un sistema de exactly-once delivery.

Su objetivo es:

evitar efectos empresariales duplicados de una misma operación retryable.

Esto es diferente de:

exactly-once network delivery

o:

exactly-once event delivery
35. Idempotencia y DELETE

Muchas operaciones DELETE son naturalmente idempotentes desde el punto de vista HTTP.

Sin embargo, si producen side effects complejos o operaciones asíncronas, podrán requerir mecanismos adicionales.

No se deberá agregar Idempotency-Key automáticamente a todos los DELETE sin analizar el caso.

36. Idempotencia y PUT/PATCH

La semántica HTTP no elimina necesariamente la necesidad de control de concurrencia.

Por ejemplo:

PATCH /users/123

puede ser repetible pero aún puede existir un problema de:

lost update

Por ello, idempotencia y optimistic concurrency son conceptos diferentes.

37. Optimistic Concurrency API

Cuando sea necesario, la API podrá utilizar mecanismos como:

If-Match
ETag
version
updated_at

para evitar sobrescribir cambios concurrentes.

Ejemplo conceptual:

Resource Version = 7

Client sends:
If-Match: 7

Si el recurso ya está en versión 8:

412 Precondition Failed

El mecanismo exacto será definido por endpoint.

38. Idempotency TTL

Los registros de idempotencia no deben conservarse indefinidamente.

Deberán existir:

created_at
expires_at

La duración dependerá del tipo de operación y ventana razonable de retry.

El período exacto queda pendiente.

39. Reutilización después de expiración

Una vez expirada una key:

ABC
↓
EXPIRED

la reutilización deberá seguir una política explícita.

No se debe permitir que una nueva operación sea confundida con la operación histórica.

La estrategia podrá requerir una nueva key.

40. Seguridad de Idempotency-Key

Las keys deben tratarse como identificadores sensibles desde el punto de vista operacional.

No deben contener:

passwords;
tokens;
secretos;
PII innecesaria.

El cliente debe utilizar valores aleatorios adecuados.

41. Error Contract

Los errores deberán respetar el formato estándar de EVOXA:

{
  "error": {
    "code": "...",
    "message": "...",
    "correlation_id": "..."
  }
}

Ejemplos conceptuales:

INVALID_CURSOR
INVALID_FILTER
INVALID_SORT
INVALID_LIMIT
IDEMPOTENCY_CONFLICT
IDEMPOTENCY_IN_PROGRESS
RESOURCE_NOT_FOUND
FORBIDDEN

El catálogo definitivo se establecerá en API Contracts.

42. HTTP Status Codes

La elección deberá ser consistente.

Conceptualmente:

Situación	Status
Request válido	200 / 201 / 204
Parámetro inválido	400
No autenticado	401
No autorizado	403
Recurso inexistente	404
Conflicto de idempotencia/concurrencia	409
Precondición fallida	412
Rate limit	429
Error interno	500

La tabla definitiva pertenece al contrato API.

43. Rate Limiting

Paginación y búsqueda no deben permitir abuso.

Especialmente:

GET /audit-events
GET /security-events
GET /users

pueden consumir recursos importantes.

Rate limiting podrá aplicarse según:

actor;
tenant;
endpoint;
IP;
riesgo.
44. Authorization antes de Query

La arquitectura deberá evitar:

DB query all
     ↓
filter unauthorized records

La estrategia preferida es:

Authentication
     ↓
Tenant Context
     ↓
Authorization
     ↓
Security Policy
     ↓
Scoped Query

Esto reduce riesgo de exposición accidental.

45. Tenant-scoped Query

Las consultas deben incorporar el contexto de tenant cuando el recurso sea tenant-scoped.

Conceptualmente:

WHERE organization_id = authorized_organization

No:

WHERE organization_id = client_parameter

sin validar autorización.

46. Cursor Security

Los cursores no deben convertirse en mecanismo para saltarse autorización.

Un cursor de:

Organization A

no puede utilizarse para enumerar:

Organization B

aunque el cursor sea técnicamente válido.

47. API Observability

Toda request deberá poder correlacionarse mediante:

correlation_id
trace_id

cuando corresponda.

En operaciones complejas también:

causation_id

La información debe propagarse hacia:

application;
domain;
persistence;
events;
audit.
48. Audit

Operaciones sensibles relacionadas con:

creación;
eliminación;
cambio de roles;
permisos;
MFA;
sesiones;
security policies;

deberán producir auditoría según IS-IDENTITY-012.

El uso de Idempotency-Key no debe eliminar la auditoría.

49. Eventos

Una operación retryable debe producir un efecto de negocio único.

Ejemplo:

POST /users
Key ABC

puede producir:

UserCreated

una sola vez como efecto de negocio.

Un retry no debe generar:

UserCreated #1
UserCreated #2

por la misma operación lógica.

50. Event Idempotency vs API Idempotency

Son mecanismos diferentes.

API Idempotency

Protege:

Client → API
Consumer Idempotency

Protege:

Event Broker → Consumer

Por tanto:

API Idempotency
      +
Domain Transaction
      +
Outbox
      +
Consumer Idempotency

forman una cadena completa de protección.

51. AI Agents

Los AI Agents también estarán sujetos a:

autenticación;
autorización;
tenant isolation;
idempotency;
rate limiting;
audit.

Un agente no podrá utilizar una operación repetidamente para producir efectos duplicados.

Las operaciones AI de alto impacto deberán utilizar controles de idempotencia apropiados.

52. Performance

La estrategia debe cumplir los objetivos generales de API.

No se debe introducir una operación de:

COUNT(*)

costosa en cada página simplemente para proporcionar metadata que el cliente no necesita.

Tampoco se deberá realizar almacenamiento de idempotencia mediante operaciones innecesariamente pesadas.

53. Cache

Los resultados de GET podrán ser cacheados cuando sea seguro.

Pero:

Authorization
+
Tenant Context
+
Security Policy

deben formar parte del modelo de cache.

No se debe servir a:

User A

un resultado cacheado de:

User B

por una key incorrectamente diseñada.

54. No cachear datos sensibles indiscriminadamente

Recursos como:

Security Events;
Audit Events;
sesiones;
MFA;
credenciales;

requieren especial cuidado antes de utilizar caches.

Redis no debe convertirse automáticamente en fuente de verdad.

55. Compatibilidad

Las convenciones definidas aquí deberán documentarse en:

OpenAPI 3.1

y validarse mediante contract tests.

Los cambios breaking deberán seguir la estrategia de versionamiento de APIs.

56. Consecuencias positivas
API
comportamiento consistente;
colecciones controladas;
contratos previsibles;
mejor UX.
Seguridad
tenant isolation;
filtros controlados;
menor superficie de abuso;
no duplicación de operaciones sensibles.
Confiabilidad
retries seguros;
protección contra double submit;
concurrencia controlada.
Evolución

Las mismas convenciones pueden utilizarse posteriormente en otros dominios de EVOXA.

57. Consecuencias negativas

La estrategia añade:

complejidad en contratos;
almacenamiento de idempotency records;
gestión de expiración;
testing adicional;
complejidad de cursor;
manejo de concurrencia;
mayor disciplina API.

Sin embargo, esta complejidad es preferible a implementar diferentes comportamientos por endpoint.

58. Alternativas consideradas
Alternativa A — Offset obligatorio para todo

Rechazada.

No escala adecuadamente para todas las colecciones dinámicas.

Alternativa B — Cursor obligatorio para todo

Rechazada.

No todos los casos requieren cursor y puede aumentar innecesariamente la complejidad.

Alternativa C — No utilizar Idempotency-Key

Rechazada.

No protege suficientemente operaciones retryable con side effects.

Alternativa D — Redis como única fuente de idempotencia

Rechazada.

No debe ser la única autoridad para garantías críticas.

Alternativa E — Idempotencia únicamente en frontend

Rechazada.

El servidor debe protegerse independientemente del comportamiento del cliente.

59. No negociables
Las colecciones no deben tener respuestas ilimitadas.
Debe existir un límite máximo.
Toda paginación debe tener orden determinista.
Cursor debe ser opaco.
Cursor debe respetar tenant isolation.
Sorting debe ser allowlisted.
Filtering debe ser allowlisted.
El cliente no puede proporcionar SQL arbitrario.
Las queries deben respetar el authorization context.
Las operaciones retryable deben soportar idempotencia cuando corresponda.
Idempotency-Key debe estar asociada al contexto correcto.
Una misma key con diferente request debe generar conflicto.
Dos requests concurrentes con la misma operación no deben duplicar efectos.
Idempotency state debe ser durable para operaciones críticas.
Idempotency state debe tener expiración.
API idempotency no reemplaza consumer idempotency.
Idempotencia no implica exactly-once delivery.
Los efectos de negocio y Outbox deben permanecer consistentes.
Tenant isolation debe aplicarse antes de devolver datos.
AI Agents también deben respetar estas reglas.
Los errores deben usar el contrato estándar.
Las requests deben ser observables mediante correlation/trace IDs.
Las operaciones sensibles deben quedar auditadas.
Los secretos no deben aparecer en keys, cursores, logs o respuestas.
Las convenciones deben documentarse en OpenAPI y contract tests.
60. Decisiones pendientes

Quedan deliberadamente abiertos:

Pagination
cursor format;
offset support;
default limit;
maximum limit;
total_count;
snapshot semantics.
Sorting
campos permitidos por recurso;
formato final;
múltiples campos.
Filtering
operadores;
filtros por endpoint;
búsqueda;
case sensitivity.
Idempotency
almacenamiento;
TTL;
fingerprint;
estados;
respuesta para IN_PROGRESS;
comportamiento ante timeout;
recuperación después de crash.
Concurrency
optimistic concurrency;
ETag/If-Match;
version fields;
conflict semantics.
Performance
índices;
cursor strategy;
count strategy;
cache.

Estos valores deberán resolverse en IS-IDENTITY-014 — Identity API Contracts, IS-IDENTITY-015 — Identity Persistence y los Technical Tasks correspondientes.

61. Criterios de aceptación arquitectónica

Este ADR podrá considerarse satisfecho cuando:

 Todas las colecciones tengan estrategia de paginación.
 Exista límite máximo.
 Exista ordenamiento determinista.
 Cursor sea opaco.
 Cursor respete tenant isolation.
 Sorting esté allowlisted.
 Filtering esté allowlisted.
 No exista SQL arbitrario desde API.
 Exista contrato uniforme de pagination metadata.
 Exista Idempotency-Key.
 Exista idempotency record.
 Exista request fingerprint.
 Exista detección de conflictos.
 Exista protección ante requests concurrentes.
 Exista TTL de idempotency records.
 Exista integración con transacciones.
 Exista integración con Outbox.
 Exista integración con Audit.
 Exista integración con Authorization.
 Exista tenant isolation.
 Existan pruebas de retry.
 Existan pruebas de concurrencia.
 Existan pruebas de cross-tenant access.
 Existan contract tests.
 OpenAPI documente las convenciones.
62. Trazabilidad
Blueprint
API-First
Security by Design
Multi-Tenant
Observable
Continuously Evolvable
Engineering Specification
ESP-0001 — Identity
Implementation Stories
IS-IDENTITY-006
Authorization & Permission Evaluation

IS-IDENTITY-008
Session Management

IS-IDENTITY-009
Refresh Token Lifecycle

IS-IDENTITY-012
Audit & Security Events

IS-IDENTITY-013
Domain Events & Outbox

IS-IDENTITY-014
Identity API Contracts

IS-IDENTITY-015
Identity Persistence

IS-IDENTITY-016
Identity Testing & Quality
ADRs relacionados
ADR-IDENTITY-005
Authorization

ADR-IDENTITY-006
Persistence Strategy

ADR-IDENTITY-007
Events & Outbox

ADR-IDENTITY-012
PostgreSQL Schema

ADR-IDENTITY-013
Data Retention

ADR-IDENTITY-016
Concurrency

ADR-IDENTITY-020
Event Schema Compatibility

ADR-IDENTITY-022
Consumer Idempotency

ADR-IDENTITY-025
Refresh Token Cryptography & Concurrency
63. Siguiente ADR

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

Con ADR-IDENTITY-026 queda definido el bloque transversal de comportamiento de las APIs de Identity. El siguiente paso lógico es ADR-IDENTITY-027 — Runtime Configuration, Secrets & KMS, que cerrará cómo EVOXA administra configuración, secretos, credenciales y claves criptográficas fuera del código y del runtime.
