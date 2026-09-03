ADR-IDENTITY-020 — Event Schema Registry & Compatibility

Estado: Proposed
Versión: 1.0
Fecha: 2026-09-02
Dominio: Identity / Event Platform
Prioridad: Critical
Tipo: Event Architecture / Contract Governance / Compatibility / Reliability

1. Título

ADR-IDENTITY-020 — Event Schema Registry & Compatibility

2. Contexto

EVOXA adopta una arquitectura Event-Driven en la que los eventos constituyen contratos entre diferentes componentes y dominios de la plataforma.

La arquitectura definida previamente establece que los eventos deben ser:

inmutables;
versionados;
observables;
seguros;
compatibles con entrega at-least-once;
procesables mediante consumidores idempotentes;
capaces de ser reintentados;
capaces de ser enviados a DLQ;
capaces de ser reproducidos mediante replay.

En ADR-IDENTITY-018 — Event Broker Strategy se estableció:

Domain
   ↓
PostgreSQL
   ↓
Transactional Outbox
   ↓
Outbox Relay
   ↓
Event Broker
   ↓
Consumer Groups
   ↓
Consumers

En ADR-IDENTITY-019 — Event Partitioning, Ordering & Routing se estableció además que:

el ordering no será global;
puede existir ordering por aggregate/user/workflow;
los eventos poseen event_type;
los eventos poseen event_version;
aggregate_version y event_version son conceptos diferentes;
los eventos contienen metadata de correlación y causalidad.

Por lo tanto, EVOXA necesita una estrategia formal para gobernar la estructura de los eventos y su evolución.

3. Problema

Un evento es un contrato.

Por ejemplo:

UserCreated v1

podría ser consumido por:

Analytics
Notifications
Search
AI Context
Security

Si Identity modifica arbitrariamente su estructura:

UserCreated v1
       ↓
breaking change
       ↓
Consumers fail

pueden producirse:

errores de procesamiento;
mensajes en DLQ;
pérdida funcional;
incompatibilidad entre versiones;
fallos durante deployments;
replay imposible;
consumers desactualizados;
inconsistencias entre dominios.

Por ello:

La evolución de un evento debe gobernarse como la evolución de una API pública.

4. Decisión

EVOXA adoptará un Schema Registry y un modelo formal de compatibilidad de eventos.

El Schema Registry será responsable conceptualmente de:

registrar schemas;
versionarlos;
identificarlos;
validarlos;
controlar compatibilidad;
gestionar lifecycle;
impedir cambios incompatibles no autorizados;
proporcionar metadata para productores y consumidores.

La arquitectura será:

                 Event Schema
                     │
                     ▼
              Schema Registry
                     │
          ┌──────────┴──────────┐
          │                     │
       Producer              Consumer
          │                     │
          └──────────┬──────────┘
                     ▼
                 Event Broker
5. Schema Registry no es Event Broker

Se separarán claramente las responsabilidades.

Event Broker

Transporta eventos.

Publish
Transport
Delivery
Partition
Consumer Groups
Schema Registry

Gobierna contratos.

Schema
Version
Compatibility
Validation
Evolution
PostgreSQL / Outbox

Mantiene la transacción de negocio.

Domain State
+
Outbox Event
6. Schema como contrato

Cada evento deberá tener un schema formal.

Conceptualmente:

Event Type
    +
Event Version
    +
Schema
    +
Semantic Rules

Ejemplo:

UserCreated
Version: 1

El schema deberá describir, como mínimo:

campos;
tipos;
required/optional;
estructura;
restricciones relevantes;
metadata;
versión.
7. Event Envelope

El schema deberá distinguir entre:

Envelope

Información común del evento:

event_id
event_type
event_version
timestamp
tenant_id
actor
resource
metadata
Payload

Información específica:

payload

La arquitectura deberá evitar mezclar ambos conceptos.

8. Schema Version

Cada evolución incompatible deberá producir una nueva versión del contrato.

Ejemplo:

UserCreated v1
      ↓
UserCreated v2

La versión deberá ser explícita.

No se deberá modificar silenciosamente el significado de:

UserCreated v1

después de que ya haya sido publicado y consumido.

9. Inmutabilidad

Una vez que una versión de schema haya sido utilizada en producción, deberá considerarse inmutable.

Por ejemplo:

UserCreated v1

no deberá cambiar de significado.

Si se requiere un cambio incompatible:

UserCreated v2

Esto preserva la capacidad de:

replay;
auditoría;
debugging;
compatibilidad histórica.
10. Compatibility Strategy

La estrategia preferida será compatibilidad hacia atrás cuando sea posible.

Esto significa que una nueva versión del schema deberá poder coexistir con consumidores que todavía procesan la versión anterior, siempre que el cambio lo permita.

Conceptualmente:

Producer v2
     │
     ├── Consumer v1
     └── Consumer v2

durante una fase de transición.

La política exacta de compatibilidad será definida según las capacidades del Schema Registry seleccionado.

11. Backward Compatibility

Un cambio será considerado backward compatible cuando los consumidores existentes puedan continuar procesando correctamente los eventos.

Ejemplos conceptuales de cambios potencialmente compatibles:

Agregar campo opcional

frente a:

Eliminar campo obligatorio

que puede romper consumidores existentes.

La compatibilidad deberá validarse automáticamente cuando la tecnología lo permita.

12. Forward Compatibility

También deberá considerarse la capacidad de consumidores nuevos para trabajar con eventos producidos por versiones anteriores.

Ejemplo:

Consumer v2
       ↑
Event v1

Esto puede ser útil durante despliegues progresivos.

No obstante, la política exacta de forward compatibility dependerá del tipo de evento y consumidor.

13. Full Compatibility

Para contratos especialmente críticos podrá evaluarse una política más estricta:

Backward
+
Forward

No se establecerá que todos los eventos deban cumplir automáticamente el máximo nivel de compatibilidad.

La política dependerá de:

criticidad;
lifecycle;
consumers;
replay;
seguridad;
evolución prevista.
14. Breaking Changes

Un cambio será tratado como breaking change cuando pueda provocar que consumidores existentes:

no puedan deserializar;
no puedan validar;
interpreten incorrectamente un campo;
pierdan información requerida;
ejecuten comportamiento incorrecto.

Ejemplos conceptuales:

Rename field
Remove required field
Change field type
Change semantic meaning
Change enum incompatibly
Change nested structure incompatibly
15. Breaking Change Strategy

Un breaking change deberá seguir:

Identify breaking change
        ↓
Create new schema version
        ↓
Register
        ↓
Validate compatibility strategy
        ↓
Update consumers
        ↓
Deploy producer
        ↓
Observe
        ↓
Retire old version when safe

No se deberá modificar directamente un schema de producción ya utilizado.

16. Semantic Compatibility

La compatibilidad no debe limitarse al tipo de datos.

Un cambio puede ser técnicamente compatible y semánticamente incompatible.

Ejemplo:

status = "ACTIVE"

Si posteriormente significa algo diferente, el schema puede seguir siendo válido técnicamente pero romper la lógica del consumidor.

Por ello, el gobierno deberá controlar también:

Semantic compatibility.

17. Enum Evolution

Los enums requieren especial cuidado.

Ejemplo:

status:
  ACTIVE
  SUSPENDED

Agregar:

DISABLED

puede romper consumidores que asumen:

only ACTIVE or SUSPENDED

Por ello, la evolución de enums deberá considerarse potencialmente breaking.

18. Required vs Optional

La evolución deberá preferir:

new field
optional

cuando sea compatible.

En general, introducir un nuevo campo obligatorio sobre consumidores existentes aumenta el riesgo de incompatibilidad.

19. Default Values

Los defaults podrán utilizarse para facilitar evolución, pero no deben ocultar cambios semánticos.

Ejemplo:

new_field
default = X

El valor default deberá representar correctamente el significado de los eventos antiguos.

20. Event Type vs Schema Version

Debe distinguirse:

event_type

de:

event_version

Por ejemplo:

event_type = UserCreated
event_version = 1

Una evolución compatible puede mantener:

UserCreated

mientras cambia la versión.

Un cambio semántico radical puede justificar un nuevo event type.

La decisión deberá tomarse caso a caso mediante governance.

21. Schema ID

Cada schema deberá tener un identificador estable.

Conceptualmente:

schema_id
event_type
event_version

Esto permitirá:

lookup;
validation;
compatibility checks;
debugging;
replay.
22. Schema Registration

Los schemas deberán registrarse antes de que puedan utilizarse en producción.

Conceptualmente:

Developer
   ↓
Schema Definition
   ↓
CI Validation
   ↓
Schema Registry
   ↓
Compatibility Check
   ↓
Approved

Un schema incompatible deberá bloquearse automáticamente salvo proceso excepcional y explícitamente autorizado.

23. CI/CD Integration

El Schema Registry deberá integrarse con CI/CD.

Pipeline conceptual:

Commit
  ↓
Schema Validation
  ↓
Compatibility Check
  ↓
Contract Tests
  ↓
Build
  ↓
Integration Tests
  ↓
Deploy

Esto evita descubrir incompatibilidades después de desplegar.

24. Producer Validation

El productor deberá publicar eventos que cumplan el schema registrado.

Conceptualmente:

Application
    ↓
Event
    ↓
Schema Validation
    ↓
Broker

La validación puede realizarse en:

application;
serialization layer;
producer client;
broker ecosystem;

según la tecnología seleccionada.

25. Consumer Validation

Los consumidores deberán conocer qué versiones de eventos soportan.

Conceptualmente:

Consumer
 ├── UserCreated v1
 └── UserCreated v2

La compatibilidad soportada deberá quedar documentada.

26. Consumer Capability

El sistema deberá poder determinar conceptualmente:

Consumer A
supports UserCreated v1

Consumer B
supports UserCreated v1 + v2

Esto será especialmente útil durante migraciones.

27. Event Evolution Lifecycle

El lifecycle recomendado será:

DRAFT
   ↓
REVIEW
   ↓
COMPATIBILITY CHECK
   ↓
APPROVED
   ↓
ACTIVE
   ↓
DEPRECATED
   ↓
RETIRED

Una versión retirada deberá permanecer disponible para:

auditoría;
histórico;
replay cuando corresponda.

Retired no significa necesariamente borrado inmediato.

28. Deprecation

Antes de retirar una versión se deberá determinar:

consumers activos;
replay requirements;
historical retention;
dependencies;
migration status.

Ejemplo:

UserCreated v1
      ↓
Deprecated
      ↓
Consumers migrate
      ↓
No active consumers
      ↓
Retired
29. Replay Compatibility

La estrategia de schema debe permitir interpretar eventos históricos.

Esto es especialmente importante para:

Replay

Un evento producido hace meses o años podría utilizar:

UserCreated v1

mientras la aplicación actual utiliza:

UserCreated v3

El sistema debe conservar la capacidad de procesar o transformar correctamente eventos históricos según la política de replay.

30. Schema Transformation

Cuando un consumer moderno necesite convertir:

v1 → v3

podrá utilizar una capa de transformación.

Conceptualmente:

Historical Event v1
       ↓
Schema Adapter
       ↓
Internal Model v3

Esto permite evitar que todos los consumers tengan que conocer indefinidamente todas las versiones.

La estrategia concreta se definirá en ADR-021.

31. Schema Registry Security

El Schema Registry deberá aplicar:

autenticación;
autorización;
TLS;
audit;
least privilege.

No todos los usuarios o servicios deberán poder:

register
modify
delete
approve

schemas.

32. Schema Immutability

Los schemas activos no deberán poder modificarse arbitrariamente.

Especialmente:

Production schema

deberá tener controles adicionales.

Las modificaciones deberán producir:

New Version

cuando el cambio afecte el contrato.

33. Schema Deletion

No se deberá eliminar un schema simplemente porque ya no se publica.

Debe considerarse:

eventos históricos;
replay;
auditoría;
DLQ;
investigaciones;
consumers antiguos.

Por ello:

Schema retirement y schema deletion son decisiones diferentes.

34. Compatibility Rules

La política de compatibilidad deberá poder definir reglas como:

Allowed:
+ optional field

Review:
+ enum value
+ nested optional object

Breaking:
- required field
- field type change
- field rename
- semantic change

Esta matriz deberá formalizarse en el gobierno de eventos.

35. Event Documentation

Cada evento deberá disponer de documentación mínima:

Event Type
Description
Version
Producer
Consumers
Schema
Semantic Rules
Security Classification
Tenant Scope
Ordering Scope
Retention
Replay Policy
Owner

Esto está alineado con la gobernanza general de eventos de EVOXA.

36. Ownership

Cada schema deberá tener un owner responsable.

Conceptualmente:

Schema
  ↓
Domain Owner
  ↓
Lifecycle
  ↓
Compatibility
  ↓
Consumers

Para Identity:

Identity Domain

será responsable de los eventos que publique.

37. Contract Testing

Se deberán implementar event contract tests.

Ejemplo:

Producer
   ↓
UserCreated v2
   ↓
Consumer Contract
   ↓
PASS / FAIL

Los tests deberán detectar:

campos incompatibles;
tipos incompatibles;
cambios semánticos documentados cuando sea posible;
versiones no soportadas.
38. Compatibility Matrix

La plataforma deberá mantener una matriz conceptual:

Event	Version	Producer	Consumer	Supported
UserCreated	1	Identity	Analytics	Yes
UserCreated	2	Identity	Analytics	Yes
UserCreated	2	Identity	Legacy Consumer	No

Esto será útil durante migraciones.

39. Event Security Classification

El schema deberá poder asociarse a una clasificación de seguridad.

Ejemplo:

INTERNAL
CONFIDENTIAL
SECURITY_SENSITIVE

Esto permite aplicar políticas apropiadas.

Los eventos de Identity relacionados con:

authentication;
authorization;
MFA;
security;

podrán requerir controles adicionales.

40. Tenant Context

El schema deberá distinguir si un evento es:

TENANT_SCOPED
SYSTEM_SCOPED
PLATFORM_SCOPED

Cuando corresponda.

Esto ayuda a mantener:

routing;
authorization;
replay;
audit;
security.
41. Event Compatibility y Tenant Isolation

Una evolución de schema no debe eliminar accidentalmente información necesaria para tenant isolation.

Por ejemplo:

tenant_id

no deberá eliminarse de un evento tenant-scoped simplemente porque el nuevo consumer "ya sabe" el tenant por otra vía.

El contrato debe preservar las garantías de seguridad necesarias.

42. Performance

La validación de schemas no debe convertirse en un cuello de botella.

La arquitectura deberá evaluar:

schema caching;
serialization efficiency;
validation overhead;
registry availability.

El Schema Registry podrá ser utilizado durante:

build;
registration;
producer validation;

sin necesariamente convertirse en una dependencia síncrona obligatoria para cada evento, dependiendo de la tecnología seleccionada.

43. Registry Failure

Si el Schema Registry está temporalmente indisponible, debe definirse qué operaciones se ven afectadas.

La estrategia preferida deberá evitar que una indisponibilidad momentánea provoque indiscriminadamente:

Identity outage

si los schemas ya están conocidos y validados.

Por ello, será importante determinar:

local schema cache;
embedded schemas;
validation mode;
fail-open vs fail-closed.

Para eventos de seguridad crítica, la política puede ser más estricta.

44. Failure Strategy

Los errores de schema deberán distinguirse de errores de infraestructura.

Invalid Schema
     ↓
Contract Error

frente a:

Registry unavailable
     ↓
Infrastructure Error

El tratamiento deberá ser diferente.

45. Consecuencias positivas
Contratos explícitos

Los eventos dejan de ser estructuras implícitas.

Evolución segura

Los cambios incompatibles pueden detectarse antes del deployment.

Replay

Se conserva la capacidad de interpretar eventos históricos.

Multi-team development

Diferentes equipos pueden evolucionar sin romper contratos.

Governance

Cada evento tiene owner, versión y lifecycle.

Reliability

Se reducen errores de consumer provocados por cambios inesperados.

Security

El schema puede incorporar clasificación y tenant scope.

46. Consecuencias negativas

La estrategia introduce:

Schema Registry;
governance;
contract tests;
versioning;
compatibility checks;
lifecycle management.

También aumenta la disciplina requerida para crear nuevos eventos.

Sin embargo, esta complejidad es necesaria cuando los eventos se convierten en contratos entre múltiples dominios.

47. Alternativas consideradas
Alternativa A — Schemas implícitos en código

Rechazada.

No proporciona governance suficiente.

Alternativa B — JSON libre sin schema

Rechazada como estrategia general.

Aumenta el riesgo de incompatibilidad.

Alternativa C — Modificar schemas existentes

Rechazada para cambios breaking.

Rompe consumers históricos y replay.

Alternativa D — Versionar únicamente la API

Rechazada.

Los eventos son contratos independientes de las APIs HTTP.

Alternativa E — Schema Registry centralizado

Seleccionada.

Permite:

governance;
validation;
compatibility;
versioning;
lifecycle.
48. Non-Negotiables
Todo evento publicado debe tener un schema definido.
Los eventos deben ser versionados.
Los schemas utilizados en producción son inmutables.
Los breaking changes requieren nueva versión.
Debe existir compatibilidad controlada.
Debe existir validación automática cuando sea posible.
Deben existir contract tests.
Debe existir ownership.
Debe existir lifecycle.
Debe existir documentación.
event_version no debe confundirse con aggregate_version.
El schema debe conservar tenant context cuando corresponda.
No se deben eliminar schemas históricos sin analizar replay.
No se deben eliminar campos sensibles o de seguridad sin evaluación.
DLQ y replay deben ser compatibles con schema evolution.
Los eventos históricos deben seguir siendo interpretables según la estrategia de replay.
Schema Registry y Event Broker son componentes distintos.
El Registry no es source of truth del dominio.
Los cambios breaking deben ser gobernados.
No se deben introducir schemas incompatibles directamente en producción.
La seguridad del Registry debe seguir least privilege.
La disponibilidad del Registry no debe convertirse innecesariamente en un single point of failure del runtime.
49. Traceability
Blueprint

Relacionado con:

Event Architecture;
immutable events;
versioned events;
schema governance;
compatibility;
replay;
observability;
security;
integration contracts.

La arquitectura de eventos establece que:

los eventos son inmutables;
breaking changes requieren nueva versión;
los schemas deben gobernarse;
deben existir contract tests;
deben existir owners;
deben existir reglas de versioning;
debe existir replay.
ESP

ESP-0001 — Identity Domain

Principalmente:

Domain Events;
Integration Events;
Security Events;
Audit Events;
Outbox.
Implementation Stories

Principalmente:

IS-IDENTITY-013 — Domain Events & Outbox
IS-IDENTITY-014 — Identity API Contracts
IS-IDENTITY-016 — Identity Testing & Quality
IS-IDENTITY-017 — Identity Runtime & Deployment
50. ADR relacionados
ADR-IDENTITY-007 — Event Architecture
ADR-IDENTITY-018 — Event Broker Strategy
ADR-IDENTITY-019 — Event Partitioning / Ordering / Routing
ADR-IDENTITY-021 — Event Replay / Governance
ADR-IDENTITY-022 — Consumer Idempotency
ADR-IDENTITY-028 — Observability / Operational Security
ADR-IDENTITY-029 — Disaster Recovery / Backup
51. Dependencias

La arquitectura queda:

ADR-018
Event Broker
      │
      ▼
ADR-019
Partitioning / Ordering / Routing
      │
      ▼
ADR-020
Schema Registry / Compatibility
      │
      ├───────────────┐
      ▼               ▼
ADR-021          ADR-022
Replay            Consumer
Governance        Idempotency

El Schema Registry será un elemento transversal entre:

Producer
   ↕
Broker
   ↕
Consumer
52. Decisiones pendientes

Este ADR deja abiertas:

Schema Technology
formato de schema;
Registry tecnológico;
serialización.
Compatibility Mode
backward;
forward;
full;
custom.
Versioning
major/minor;
integer versions;
schema IDs;
event versions.
Validation
producer;
CI/CD;
broker;
consumer;
combinación.
Registry Availability
caching;
local schemas;
fallback;
fail behavior.
Governance
approval process;
ownership;
deprecation;
retirement.
53. Criterios de aceptación

ADR-IDENTITY-020 podrá pasar de Proposed a Accepted cuando exista:

 formato de schema seleccionado;
 Schema Registry seleccionado;
 schema lifecycle;
 schema versioning;
 compatibility policy;
 breaking-change policy;
 schema ownership;
 registration workflow;
 CI/CD integration;
 producer validation;
 consumer validation;
 contract testing;
 compatibility matrix;
 deprecation strategy;
 retirement strategy;
 replay compatibility;
 DLQ compatibility;
 security model;
 tenant scope;
 schema caching/failure strategy;
 observability;
 operational runbook.
54. Siguiente ADR

Con ADR-018, ADR-019 y ADR-020, ya tenemos definida la columna vertebral del transporte y contrato de eventos:

ADR-018
Event Broker
      ↓
ADR-019
Partitioning / Ordering / Routing
      ↓
ADR-020
Schema Registry / Compatibility

El siguiente paso lógico es:

ADR-IDENTITY-021 — Event Replay & Governance

Ahí definiremos cómo EVOXA podrá:

reproducir eventos;
reconstruir proyecciones;
recuperar consumers;
procesar DLQ;
hacer replay parcial;
hacer replay por tenant;
hacer replay por aggregate;
controlar quién puede ejecutar replay;
evitar efectos secundarios peligrosos;
preservar ordering;
mantener auditoría;
controlar retención;
diferenciar replay operacional de replay histórico.

La cadena continuará:

018 Event Broker
      ↓
019 Partitioning / Ordering / Routing
      ↓
020 Schema Registry / Compatibility
      ↓
021 Event Replay / Governance
      ↓
022 Consumer Idempotency

Estado final: Proposed.
