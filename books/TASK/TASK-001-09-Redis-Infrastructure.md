TASK-001-09 — Redis Infrastructure

IS: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Dependencias: TASK-001-08 — PostgreSQL Infrastructure
Prioridad: 🔴 Critical
Estado: Ready for Implementation

1. Objetivo

Implementar la infraestructura Redis de Identity para soportar:

cache;
estado temporal;
rate limiting;
coordinación de operaciones que lo requieran;
datos efímeros;
futuras necesidades de Authorization Cache.

La regla arquitectónica fundamental es:

Redis es infraestructura de optimización/estado temporal; no es la fuente autoritativa de Identity.

Por lo tanto:

                IDENTITY
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
   PostgreSQL               Redis
   SOURCE OF               CACHE /
    TRUTH                   TEMP
        │                     │
        └──────────┬──────────┘
                   ▼
             Event Platform

Si Redis desaparece:

Redis = ❌
PostgreSQL = ✅

Identity debe poder recuperarse.

2. Alcance

Esta tarea implementa:

Redis configuration
Redis client
Connection pool
Redis lifecycle
Health check
Timeouts
Retry policy base
Serialization boundary
Namespace/key boundary
Failure handling
Dependency injection
Testing
Observability foundation

No implementa todavía:

❌ Authorization Cache
❌ MFA challenge storage
❌ Rate limiting policies
❌ Session cache
❌ Refresh token cache
❌ Distributed locks específicos
❌ Application business data

Esas funcionalidades utilizarán esta infraestructura posteriormente.

3. Principio de diseño

Redis debe tratarse como:

Derived / Temporary State

y no como:

Authoritative Identity State

Ejemplo correcto:

PostgreSQL
    │
    ▼
Authoritative state
    │
    ▼
Redis
    │
    ▼
Performance optimization

No:

Redis
   │
   ▼
Source of truth
4. Casos de uso permitidos

Inicialmente Redis podrá soportar:

Cache
authorization cache
policy cache
permission cache
configuration cache
Temporary State
MFA challenge
OTP state
temporary workflow state
Rate Limiting
login attempts
MFA attempts
password reset abuse
API rate limits
Coordination

Cuando una futura funcionalidad lo necesite:

single-flight
distributed coordination
short-lived locks

Siempre bajo reglas específicas.

5. Casos de uso prohibidos

No debemos almacenar como único origen:

User
Organization
Membership
Role
Permission
Session
SecurityPolicy
RefreshToken
AuditEvent
SecurityEvent

Si alguno de estos objetos aparece en Redis:

PostgreSQL
    ↓
source of truth

Redis
    ↓
derived representation
6. Arquitectura
FastAPI
   │
   ▼
Application
   │
   ├───────────────┐
   ▼               ▼
PostgreSQL       Redis
   │               │
   ▼               ▼
Authoritative     Cache /
State             Temporary State

Y:

Redis failure
     │
     ▼
Controlled degradation
     │
     ├── PostgreSQL fallback
     ├── security-sensitive operations → fresh state
     └── no security bypass

Esto es especialmente importante para Authorization.

7. Estructura

La infraestructura puede quedar:

apps/api/app/core/
│
└── infrastructure/
    ├── database/
    │   ├── engine.py
    │   ├── session.py
    │   └── health.py
    │
    └── redis/
        ├── client.py
        ├── pool.py
        ├── health.py
        ├── exceptions.py
        ├── serialization.py
        └── keys.py

La lógica específica de Identity no debe vivir aquí.

Por ejemplo:

core/infrastructure/redis/

sabe hablar Redis.

Pero:

identity/authorization/

decide qué datos de Authorization pueden almacenarse.

8. Dependencia

Podemos utilizar un cliente Redis compatible con asyncio.

La implementación concreta podrá utilizar:

redis-py asyncio

manteniendo el contrato aislado en Infrastructure.

No necesitamos introducir un SDK de Redis directamente en Domain.

9. Configuración

Extenderemos Settings:

class Settings(BaseSettings):

    redis_host: str
    redis_port: int = 6379
    redis_database: int = 0

    redis_username: str | None = None
    redis_password: str | None = None

    redis_ssl: bool = True

    redis_max_connections: int = 50

    redis_socket_timeout: float = 2.0
    redis_socket_connect_timeout: float = 2.0

    redis_health_check_interval: int = 30

Los valores son baseline inicial, no valores finales de producción.

El dimensionamiento dependerá posteriormente de:

traffic
replicas
operation type
Redis deployment topology
10. Secrets

Si Redis requiere autenticación:

REDIS_PASSWORD

debe provenir de:

Secret Manager

en producción.

Nunca:

Git
Docker image
source code
logs
events
11. Redis URL

Infrastructure puede construir:

redis://...

o:

rediss://...

pero esa URL nunca debe registrarse completa.

Especialmente si contiene:

username
password
credentials
12. TLS

Para producción debemos soportar Redis sobre TLS:

Application
    │
    │ TLS
    ▼
Redis

La configuración exacta dependerá del proveedor/infraestructura.

No se debe deshabilitar la verificación de certificados como workaround permanente.

13. Redis Client

Conceptualmente:

import redis.asyncio as redis


def create_redis_client(settings):

    return redis.Redis(
        host=settings.redis_host,
        port=settings.redis_port,
        db=settings.redis_database,
        username=settings.redis_username,
        password=settings.redis_password,
        ssl=settings.redis_ssl,
        max_connections=settings.redis_max_connections,
        socket_timeout=settings.redis_socket_timeout,
        socket_connect_timeout=settings.redis_socket_connect_timeout,
        health_check_interval=settings.redis_health_check_interval,
        decode_responses=False,
    )

La infraestructura concreta puede evolucionar cuando definamos el deployment final.

14. Connection Pool

Redis utilizará un pool de conexiones.

Application
     │
     ▼
Redis Client
     │
     ▼
Connection Pool
 ┌───┼───┬───┐
 ▼   ▼   ▼   ▼
 R   R   R   R

Debemos evitar crear un cliente Redis por request:

async def endpoint():

    redis = create_redis_client(...)

Eso es incorrecto.

El cliente/pool será gestionado por el lifecycle de la aplicación.

15. Application Lifecycle

Startup:

Application startup
       │
       ├── load Settings
       ├── create PostgreSQL engine
       ├── create Redis client
       └── initialize infrastructure

Shutdown:

Application shutdown
       │
       ├── dispose PostgreSQL
       └── close Redis
16. Health Check

Agregaremos Redis a:

GET /api/v1/health/ready

La comprobación mínima:

PING

Resultado:

PONG
17. Redis Health ≠ API Liveness

Si Redis falla:

GET /health/live

puede continuar respondiendo:

200

porque el proceso sigue vivo.

La readiness debe reflejar si Redis es una dependencia necesaria para el nivel actual de servicio.

Esto debe ser tratado cuidadosamente porque Redis no siempre será una dependencia crítica para todas las operaciones.

18. Readiness degradable

No debemos convertir automáticamente:

Redis DOWN

en:

Entire API DOWN

si la operación puede funcionar de forma segura con PostgreSQL.

Por ejemplo:

Redis unavailable
      │
      ├── normal cached read
      │      ↓
      │   PostgreSQL fallback
      │
      └── critical authorization
             ↓
          fresh authoritative evaluation

El comportamiento específico será definido por cada módulo.

19. Fail Closed

Redis nunca debe utilizarse para convertir:

UNKNOWN

en:

ALLOW

Ejemplo:

Authorization cache unavailable
          │
          ▼
     PostgreSQL / fresh
          │
          ▼
      evaluate

Nunca:

Redis error
   ↓
ALLOW

Esto conecta directamente con las reglas de Authorization Cache.

20. Timeouts

Toda operación Redis deberá tener timeout.

No queremos:

Redis unavailable
      │
      ▼
request waits indefinitely

Sino:

Redis request
      │
      ▼
timeout
      │
      ▼
controlled fallback / failure

Los valores finales serán afinados mediante performance testing.

21. Retry Policy

No debemos aplicar retries ilimitados.

Incorrecto:

Redis error
 ↓
retry
 ↓
retry
 ↓
retry
 ↓
retry...

Eso puede producir:

latency amplification
connection exhaustion
cascading failure

Inicialmente:

bounded retry
+
timeout
+
backoff

y cada consumidor podrá definir si el retry es seguro.

22. Redis Exceptions

Crear:

core/infrastructure/redis/exceptions.py

Por ejemplo:

class RedisInfrastructureError(Exception):
    pass


class RedisUnavailable(RedisInfrastructureError):
    pass


class RedisTimeout(RedisInfrastructureError):
    pass

Infrastructure traducirá las excepciones del cliente.

Application no debería depender de:

redis.exceptions.ConnectionError

directamente.

23. Serialization Boundary

No almacenaremos objetos Python directamente como representación implícita.

Debe existir una frontera de serialización.

Domain/Application object
        │
        ▼
   Serialization
        │
        ▼
      Redis

Y:

Redis
  │
  ▼
Deserialization
  │
  ▼
Application representation
24. JSON como baseline

Para datos estructurados no sensibles, podemos utilizar JSON:

{
  "version": 1,
  "value": "..."
}

Pero cada módulo será responsable de definir el schema de su cache.

No debemos almacenar directamente:

pickle

como mecanismo general.

Esto evita riesgos de deserialización y acoplamiento de código.

25. Versioning

Los objetos cacheados deben poder evolucionar.

Ejemplo:

identity:authorization:v1:...
identity:authorization:v2:...

o mediante metadata:

{
  "schema_version": 2,
  "..."
}

La estrategia definitiva de keys/versioning será refinada en las tareas específicas de Authorization Cache.

26. Key Namespace

Nunca debemos utilizar keys genéricas:

user:123
session:123
policy:123

porque múltiples dominios podrían colisionar.

Utilizaremos namespace.

Por ejemplo:

evoxa:identity:authorization:...
evoxa:identity:mfa:...
evoxa:identity:ratelimit:...

La nomenclatura exacta de Authorization Cache se definirá posteriormente.

27. Key Builder

Podemos preparar:

core/infrastructure/redis/keys.py

con una función controlada:

def build_key(
    namespace: str,
    resource: str,
    identifier: str,
) -> str:

    return (
        f"evoxa:{namespace}:"
        f"{resource}:{identifier}"
    )

Pero los módulos no deben poder insertar SQL-like o datos arbitrarios sin validación.

28. Tenant-aware Keys

Cuando Redis almacene datos tenant-scoped:

tenant_id

deberá formar parte de la identidad de la key.

Conceptualmente:

evoxa:
identity:
authorization:
tenant:<tenant_id>:
...

Esto crea una defensa adicional contra contaminación de cache entre tenants.

29. Pero la Key NO es el control de seguridad

Muy importante:

tenant key isolation

no sustituye:

authorization
tenant guard
database isolation
policy evaluation

Una key incorrecta no puede otorgar acceso.

30. TTL

Los datos cacheados deben tener TTL apropiado.

No:

SET key value

sin expiración cuando el dato depende de freshness.

Debe existir:

SET key value EX ttl

cuando corresponda.

Sin embargo:

TTL no es equivalente a una revocación de seguridad.

Una invalidación de seguridad debe invalidar el cache independientemente del TTL.

31. Authorization Cache

Esto será especialmente importante más adelante.

La arquitectura será:

PostgreSQL
    │
    ▼
Authoritative authorization state
    │
    ▼
Redis
    │
    ▼
Authorization Cache

Con:

security epoch
entity version
policy version
context fingerprint
generation
TTL

según las reglas de ADR-050–059.

Esta tarea solamente proporciona el transporte Redis.

32. MFA Temporary State

Redis podrá utilizarse posteriormente para:

MFA challenge
OTP attempt counter
temporary enrollment state

pero la seguridad de esos objetos será definida por MFA.

Especialmente:

Redis loss

no debe permitir:

MFA bypass

Por ejemplo:

MFA challenge missing
       ↓
NOT
       ↓
MFA automatically accepted

Debe ocurrir:

challenge unavailable
       ↓
verification fails / controlled retry
33. Rate Limiting

Redis es un candidato natural para contadores temporales:

login attempts
MFA attempts
reset attempts
API requests

pero esta tarea no implementará todavía el algoritmo.

La futura implementación deberá seguir:

bounded
tenant-aware
actor-aware
endpoint-aware
operation-aware

y fallar de forma segura si Redis no está disponible.

34. No guardar secretos

Redis no deberá almacenar secretos en texto plano salvo que una futura especificación de seguridad explícitamente lo permita y establezca controles equivalentes.

Especialmente prohibido como cache:

password
password hash
access token
refresh token plaintext
MFA secret
recovery code
private key
KMS secret
API credential
35. Refresh Tokens

No debemos usar Redis como fuente de verdad para refresh tokens.

Correcto:

PostgreSQL
    │
    ▼
Refresh Token State

Redis podría utilizarse posteriormente para optimizaciones auxiliares, pero:

Redis ≠ Refresh Token Authority

Esto mantiene la coherencia con ADR-025.

36. Session State

La misma regla:

PostgreSQL
    ↓
Session Authority

Redis puede mantener representaciones derivadas, pero una pérdida de Redis no puede:

reactivate revoked session
37. Distributed Locks

Redis podrá eventualmente proporcionar coordinación distribuida, pero no debemos utilizar locks como sustituto de las garantías transaccionales PostgreSQL.

Incorrecto:

Redis lock
   ↓
assume database consistency

Correcto:

PostgreSQL transaction
   +
appropriate concurrency control

Redis lock puede ser una optimización/coordinación adicional.

38. Dependency Injection

La aplicación podrá obtener Redis mediante una dependencia central:

def get_redis():
    return redis_client

Pero los Use Cases no deberían hacer:

redis.Redis(...)

directamente.

39. Redis Port

Cuando una funcionalidad necesite Redis, preferiremos un port/interface.

Por ejemplo:

Application
     │
     ▼
CachePort
     ▲
     │
RedisAdapter

Esto permite:

FakeCache

en tests sin necesitar Redis real.

40. Arquitectura

La dirección de dependencia:

Application
     │
     ▼
Redis Port
     ▲
     │
Infrastructure
     │
     ▼
Redis

No:

Domain
   ↓
Redis
41. Testing Unitario

Podremos probar:

key generation
serialization
timeouts
exception mapping
cache adapter

sin Redis real.

42. Integration Tests

Con Redis real:

SET
GET
EXPIRE
DELETE
PING

y verificar:

connection
timeout
recovery
43. Failure Tests

Debemos simular:

Redis unavailable
Redis timeout
connection refused
pool exhausted
invalid response
serialization failure

El sistema debe producir comportamiento controlado.

Nunca:

Redis error
      ↓
security bypass
44. Recovery

Escenario:

Redis
  ↓
DOWN

posteriormente:

Redis
  ↓
UP

El sistema debe poder recuperar el servicio sin asumir que Redis contiene información autoritativa.

Para cache:

cache miss
    ↓
PostgreSQL / authoritative source
    ↓
rebuild cache
45. Cache Stampede

Más adelante, cuando implementemos Authorization Cache, tendremos que proteger:

Cache expires
       │
       ▼
10,000 requests
       │
       ▼
10,000 PostgreSQL queries

Esta tarea dejará preparada la infraestructura para mecanismos como:

single-flight
bounded concurrency
jitter
backpressure

pero no los implementaremos aquí todavía.

46. Observabilidad

Prepararemos métricas:

redis_commands_total
redis_errors_total
redis_timeouts_total
redis_connection_errors_total
redis_pool_exhaustion_total
redis_operation_duration
redis_health_status

También:

cache_hit
cache_miss

pero esos indicadores pertenecerán a los consumidores de cache, no necesariamente al adapter Redis.

47. Logging

Permitido:

Redis connection failed
Redis timeout
Redis health check failed
Redis pool exhausted

No permitido:

redis://user:password@...

ni valores sensibles almacenados en Redis.

48. Docker Development

Para desarrollo local podemos utilizar:

Redis container

por ejemplo:

redis:...

pero la versión exacta deberá fijarse en la infraestructura del proyecto.

La aplicación no debe depender de Redis instalado directamente en el sistema operativo del desarrollador.

49. Environment Separation

Configuraciones:

Development
QA
Staging
Production

deben tener Redis independiente.

Nunca:

Development → Production Redis
50. Security Boundary

Redis debe estar en una red privada cuando sea posible:

Internet
   │
   ✗
   │
Private Network
   │
   ▼
Application
   │
   ▼
Redis

No debería exponerse públicamente.

51. Acceptance Criteria
ID	Criterio
AC-01	Redis configuration está externalizada
AC-02	Credenciales nunca están hardcodeadas
AC-03	Existe Redis client central
AC-04	Existe connection pool
AC-05	Existe lifecycle de startup/shutdown
AC-06	Existe health check
AC-07	Existen timeouts
AC-08	Los retries son bounded
AC-09	Redis failure no genera security bypass
AC-10	Redis no es source of truth
AC-11	Existe boundary de serialización
AC-12	Existe namespace de keys
AC-13	Tenant-scoped keys pueden aislarse
AC-14	Secrets no se almacenan accidentalmente
AC-15	Redis puede recuperarse después de una caída
AC-16	Domain no importa Redis
AC-17	Application utiliza ports/adapters
AC-18	Existen tests unitarios
AC-19	Existen integration tests
AC-20	Existe observabilidad básica
52. Definition of Done
[ ] Redis settings
[ ] Redis credentials externalized
[ ] Redis client
[ ] Connection pool
[ ] TLS capability
[ ] Timeouts
[ ] Bounded retry
[ ] Redis exceptions
[ ] Health check
[ ] Startup integration
[ ] Shutdown integration
[ ] Serialization boundary
[ ] Key namespace
[ ] Tenant-aware key capability
[ ] Cache TTL capability
[ ] Failure handling
[ ] Recovery handling
[ ] Observability
[ ] Unit tests
[ ] Integration tests
[ ] Architecture tests
[ ] No Redis dependency in Domain
53. Trazabilidad
IS-001
 │
 └── TASK-001-09
       │
       ├── ESP-0001/11
       │     Identity Persistence
       │
       ├── ESP-0001/14
       │     Implementation Requirements
       │
       ├── ESP-0001/15
       │     Deployment & Operations
       │
       ├── ADR-006
       │     PostgreSQL Source of Truth
       │
       ├── ADR-027
       │     Runtime Configuration & Secrets
       │
       ├── ADR-032
       │     Rate Limiting / Abuse
       │
       ├── ADR-050
       │     Authorization Cache
       │
       ├── ADR-054
       │     Cache Failure & Degradation
       │
       ├── ADR-055
       │     Cache Recovery
       │
       ├── ADR-056
       │     Cache Consistency
       │
       ├── ADR-057
       │     Freshness / TTL
       │
       ├── ADR-058
       │     Criticality Matrix
       │
       └── ADR-059
             Dynamic Risk / Criticality
54. Resultado arquitectónico

Con TASK-001-09 terminamos el segundo componente de infraestructura importante:

                    EVOXA
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
     PostgreSQL                  Redis
       │                           │
       │                           │
 Source of Truth            Cache / Temporary
       │                           │
       └────────────┬──────────────┘
                    ▼
               Application

La diferencia queda claramente establecida:

PostgreSQL
AUTHORITATIVE
Redis
DERIVED / TEMPORARY / OPTIMIZATION

Y una regla que debemos conservar durante toda la implementación:

La pérdida, corrupción, retraso o indisponibilidad de Redis nunca puede aumentar los privilegios de un actor ni permitir un bypass de autenticación, MFA, autorización o tenant isolation.

Estado actual
TASK-001-01  API Runtime Foundation        ✅
TASK-001-02  Identity Domain Package       ✅
TASK-001-03  Layer Architecture            ✅
TASK-001-04  Identity Shared Kernel        ✅
TASK-001-05  Domain Events Foundation      ✅
TASK-001-06  Repository Contracts          ✅
TASK-001-07  Unit of Work                  ✅
TASK-001-08  PostgreSQL Infrastructure     ✅
TASK-001-09  Redis Infrastructure          ✅

Siguiente: TASK-001-10 — Configuration & Secrets. Ahí vamos a consolidar la configuración de toda la aplicación —API, PostgreSQL, Redis, seguridad, JWT/KMS, Event Platform y entornos— y establecer definitivamente la frontera entre configuración pública, configuración sensible y secretos, incluyendo validación de startup y reglas para que ningún secreto termine en Git, Docker, logs, eventos o contexto de IA.
