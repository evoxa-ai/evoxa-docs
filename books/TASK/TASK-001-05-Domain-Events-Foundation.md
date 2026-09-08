TASK-001-05 — Domain Events Foundation

IS: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Dependencias: TASK-001-04 — Identity Shared Kernel
Prioridad: Critical
Estado: Ready for Implementation

Esta tarea establece la infraestructura conceptual y técnica para que Identity pueda producir eventos de dominio de forma consistente, sin acoplar el Domain al Event Platform.

La separación fundamental será:

Domain Event
     ↓
Aggregate
     ↓
Unit of Work
     ↓
Transactional Outbox
     ↓
Event Publisher
     ↓
Event Platform
     ↓
Consumers

Y una regla fundamental:

Un Domain Event es un hecho ocurrido en el dominio. No es un comando ni una instrucción para otro módulo.

1. Objetivo

Implementar la base para:

Domain Events;
eventos inmutables;
Aggregate Root event collection;
Event Metadata;
Event Envelope;
Outbox Contract;
Event Publisher Contract;
Event Consumer Contract;
idempotencia conceptual;
correlation/causation/trace context;
versionamiento;
separación Domain Event / Integration Event;
tests de eventos.

Todavía no conectaremos el Event Platform real.

Tampoco implementaremos todavía la persistencia PostgreSQL del Outbox. Eso dependerá de:

TASK-001-08 — PostgreSQL Infrastructure
TASK-001-07 — Unit of Work
2. Arquitectura

La arquitectura será:

                   DOMAIN
                     │
                     ▼
              Domain Event
                     │
                     ▼
               AggregateRoot
                     │
                     ▼
                UnitOfWork
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
   Domain State             Outbox Event
          │                     │
          │                     ▼
          │              Event Publisher
          │                     │
          ▼                     ▼
     PostgreSQL            Event Platform
                                │
                 ┌──────────────┼──────────────┐
                 ▼              ▼              ▼
             Consumer       Consumer       Consumer

El punto crítico es:

Estado de negocio + Outbox
            ↓
        misma transacción

Esto evita que el estado quede persistido sin su evento correspondiente.

3. Domain Event vs Integration Event

No vamos a mezclarlos.

Domain Event

Representa:

"algo ocurrió dentro de Identity"

Ejemplos futuros:

UserCreated
UserSuspended
MembershipCreated
MembershipRevoked
RoleAssigned
SessionRevoked
MFAFactorActivated
RefreshTokenReused
Integration Event

Representa el hecho que Identity publica hacia otros consumidores.

Domain Event
     ↓
Integration Event

El segundo incorpora el envelope y metadata necesarios para distribución.

4. Estructura

Ampliaremos:

identity/shared/

a:

identity/
├── shared/
│   ├── domain/
│   │   ├── entity.py
│   │   ├── aggregate.py
│   │   ├── value_object.py
│   │   ├── identifiers.py
│   │   ├── events.py
│   │   ├── exceptions.py
│   │   └── repositories.py
│   │
│   ├── application/
│   │   ├── ports.py
│   │   ├── result.py
│   │   └── event_ports.py
│   │
│   └── infrastructure/
│       ├── unit_of_work.py
│       └── events/
│           ├── __init__.py
│           └── publisher.py
│
└── integration/
    ├── __init__.py
    └── events/
        ├── __init__.py
        ├── envelope.py
        └── mapper.py

La carpeta integration pertenece al módulo Identity, pero no significa que el Domain dependa de ella.

5. Domain Event

Partimos del evento creado en TASK-001-04, pero ahora lo hacemos más robusto.

identity/shared/domain/events.py
from dataclasses import dataclass
from datetime import datetime, timezone
from uuid import UUID, uuid4


@dataclass(frozen=True)
class DomainEvent:
    """
    Base immutable domain event.

    A DomainEvent represents a fact that occurred
    inside the Identity domain.

    It is not an integration envelope.
    """

    event_id: UUID
    occurred_at: datetime

    @classmethod
    def create(cls) -> "DomainEvent":
        return cls(
            event_id=uuid4(),
            occurred_at=datetime.now(timezone.utc),
        )

La inmutabilidad es importante porque un evento representa un hecho histórico.

6. Evento concreto de ejemplo

Todavía no vamos a crear UserCreated, porque User pertenece a Account.

Pero necesitamos comprobar el mecanismo.

En tests:

from dataclasses import dataclass

from app.domains.identity.shared.domain.events import DomainEvent


@dataclass(frozen=True)
class ExampleCreated(DomainEvent):
    name: str = ""

Podemos entonces crear:

event = ExampleCreated(
    event_id=uuid4(),
    occurred_at=datetime.now(timezone.utc),
    name="example",
)

Esto nos permite probar el framework sin adelantar el módulo Account.

7. Aggregate Root

El Aggregate Root ya tiene:

record_event()
pull_domain_events()

Pero debemos establecer una regla:

El Aggregate registra eventos; no los publica.

Por tanto:

Aggregate
    │
    └── record_event()

pero nunca:

# ❌
aggregate.publish_to_kafka()

ni:

# ❌
aggregate.send_event()

El Domain no conoce el mecanismo de transporte.

8. Event Metadata

Necesitamos conservar contexto técnico.

Crearemos:

identity/integration/events/envelope.py
from dataclasses import dataclass
from datetime import datetime
from uuid import UUID


@dataclass(frozen=True)
class EventMetadata:
    correlation_id: str | None = None
    causation_id: UUID | None = None
    trace_id: str | None = None
    actor_type: str | None = None
    actor_id: UUID | None = None

Esto permite representar:

correlation
causation
trace
actor

sin colocar secretos.

9. Event Envelope

El envelope de integración:

from dataclasses import dataclass
from datetime import datetime
from typing import Any
from uuid import UUID


@dataclass(frozen=True)
class EventEnvelope:
    event_id: UUID
    event_type: str
    event_version: int
    occurred_at: datetime

    tenant_id: UUID | None

    actor_type: str | None
    actor_id: UUID | None

    resource_type: str | None
    resource_id: UUID | None

    payload: dict[str, Any]
    metadata: EventMetadata

Esto corresponde conceptualmente al envelope estándar de eventos de EVOXA.

10. Event Version

Los eventos serán versionados.

Ejemplo:

UserCreated v1
UserCreated v2

No debemos cambiar silenciosamente el significado de:

UserCreated v1

Si existe un cambio incompatible:

UserCreated v1
        ↓
UserCreated v2

El consumidor puede entonces saber qué contrato está procesando.

11. ¿Qué significa event_version?

Es la versión del contrato del evento, no la versión de la aplicación.

No significa:

API version
Database version
Application version

Son conceptos distintos.

12. Event Type

Usaremos un nombre estable.

Ejemplos conceptuales:

identity.user.created
identity.user.suspended
identity.membership.revoked
identity.session.revoked
identity.mfa.factor.activated

Pero no vamos a congelar todavía el catálogo definitivo ni los nombres exactos.

Eso debe consolidarse con el catálogo de eventos de Identity.

13. Event Mapper

El Domain Event debe transformarse en Integration Event fuera del Domain.

Crearemos:

identity/integration/events/mapper.py
from typing import Any
from uuid import UUID

from app.domains.identity.shared.domain.events import DomainEvent
from app.domains.identity.integration.events.envelope import (
    EventEnvelope,
    EventMetadata,
)


def map_domain_event(
    event: DomainEvent,
    *,
    event_type: str,
    tenant_id: UUID | None = None,
    actor_type: str | None = None,
    actor_id: UUID | None = None,
    resource_type: str | None = None,
    resource_id: UUID | None = None,
    payload: dict[str, Any] | None = None,
    correlation_id: str | None = None,
    causation_id: UUID | None = None,
    trace_id: str | None = None,
) -> EventEnvelope:

    metadata = EventMetadata(
        correlation_id=correlation_id,
        causation_id=causation_id,
        trace_id=trace_id,
        actor_type=actor_type,
        actor_id=actor_id,
    )

    return EventEnvelope(
        event_id=event.event_id,
        event_type=event_type,
        event_version=1,
        occurred_at=event.occurred_at,
        tenant_id=tenant_id,
        actor_type=actor_type,
        actor_id=actor_id,
        resource_type=resource_type,
        resource_id=resource_id,
        payload=payload or {},
        metadata=metadata,
    )

Esta transformación ocurre fuera del Domain.

14. Outbox Contract

Ahora viene una de las piezas fundamentales.

Creamos:

identity/shared/application/event_ports.py
from abc import ABC, abstractmethod
from typing import Protocol

from app.domains.identity.integration.events.envelope import EventEnvelope


class OutboxWriter(ABC):
    """
    Contract for storing events in the transactional outbox.
    """

    @abstractmethod
    async def append(
        self,
        event: EventEnvelope,
    ) -> None:
        raise NotImplementedError


class EventPublisher(ABC):
    """
    Contract for publishing integration events.
    """

    @abstractmethod
    async def publish(
        self,
        event: EventEnvelope,
    ) -> None:
        raise NotImplementedError

Esto establece:

Application
     │
     ▼
OutboxWriter
     │
     ▼
Infrastructure

y:

Publisher
     │
     ▼
Event Platform
15. Transactional Outbox

La regla será:

BEGIN TRANSACTION

    persist business state

    append outbox event

COMMIT

No:

persist business state
COMMIT

publish event

porque si falla el publisher:

Database = updated
Event Platform = no event

y tenemos inconsistencia.

16. Flujo correcto

Por ejemplo, futuro:

Create User
     │
     ▼
User Aggregate
     │
     ▼
UserCreated
     │
     ▼
UnitOfWork
     │
     ├── User INSERT
     │
     └── Outbox INSERT
     │
     ▼
COMMIT

Después:

Outbox Worker
      │
      ▼
Event Platform

El worker puede fallar y reintentar.

El estado de negocio no se pierde.

17. Outbox State

El modelo conceptual será:

PENDING
   ↓
PUBLISHED

y ante errores:

PENDING
   ↓
FAILED
   ↓
RETRY

No fijaremos todavía el esquema físico ni todos los estados porque eso pertenece a Persistence.

18. At-Least-Once

La publicación será:

AT-LEAST-ONCE

Por lo tanto:

Event may be delivered more than once.

Los consumidores deben ser idempotentes.

Ejemplo:

event_id = abc123

Si llega:

abc123
abc123

el segundo procesamiento no debe producir un segundo resultado de negocio.

19. Consumer Contract

Aunque la implementación de consumers llegará posteriormente, podemos establecer el contrato.

identity/shared/application/event_ports.py

Añadimos:

class EventConsumer(ABC):

    @abstractmethod
    async def handle(
        self,
        event: EventEnvelope,
    ) -> None:
        raise NotImplementedError

El consumer será responsable de:

receive
 ↓
validate
 ↓
deduplicate
 ↓
process
 ↓
ack
20. Idempotencia del consumidor

No vamos a utilizar:

event_id == business_id

Son cosas diferentes.

Conceptualmente:

Event ID
    ↓
Deduplication Record
    ↓
Business Processing

El consumidor deberá poder determinar:

¿Ya procesé este event_id?

La persistencia concreta será definida más adelante.

21. Causation vs Correlation

Debemos mantener ambos conceptos separados.

Correlation ID

Agrupa una operación completa.

HTTP Request
    │
    ├── Command
    ├── Event A
    ├── Event B
    └── Event C

Todos pueden compartir:

correlation_id
Causation ID

Indica qué evento/acción causó otro evento.

Event A
   │
   ▼
Event B

Entonces:

B.causation_id = A.event_id

Esto será extremadamente útil para debugging y trazabilidad.

22. Actor Context

El envelope permite representar:

USER
SERVICE
SYSTEM
AI_AGENT

como ya definimos para Identity.

Ejemplo:

{
  "actor_type": "USER",
  "actor_id": "..."
}

Para un proceso automático:

{
  "actor_type": "SYSTEM"
}

Para un agente:

{
  "actor_type": "AI_AGENT",
  "actor_id": "..."
}

El agente AI no obtiene privilegios especiales por ser AI_AGENT.

El evento solamente identifica quién actuó.

23. Tenant Context

Los eventos Identity deben ser tenant-aware cuando el recurso sea tenant-scoped.

Conceptualmente:

tenant_id

forma parte del envelope.

Esto será especialmente importante para:

Membership
Role
Permission
Delegation
Policy
Session
Authorization

Los eventos platform-scoped pueden no necesitar tenant_id.

No debemos inventar artificialmente un tenant para eventos globales.

24. No secrets

Nunca debemos colocar en eventos:

password
password hash
refresh token
access token
MFA secret
recovery code
private key
API secret
KMS secret

Tampoco:

authorization headers
session cookies

Los eventos deben contener únicamente la información necesaria para representar el hecho.

25. Eventos no son comandos

Incorrecto:

UserCreated
    ↓
"Create user in another domain"

El evento dice:

UserCreated

No:

CreateUser

Un consumidor decide qué hacer al recibir el hecho.

26. Replay

El Event Platform podrá permitir replay.

Pero:

Replay ≠ ejecutar nuevamente una operación de negocio ciegamente

El consumidor debe ser idempotente.

Por ejemplo:

UserCreated
   ↓
Replay
   ↓
Consumer
   ↓
deduplication / idempotent processing

No debemos crear un segundo User como consecuencia del replay.

27. Event Ordering

No asumiremos orden global.

El orden será:

scoped ordering

cuando sea requerido.

Por ejemplo:

User A
 ├── Event 1
 ├── Event 2
 └── Event 3

puede requerir un orden determinado.

Pero:

User A Event 3

no necesariamente debe bloquear:

User B Event 1

La estrategia concreta de particionado/routing vendrá posteriormente.

28. Event Publisher

Crearemos únicamente el adapter base.

identity/shared/infrastructure/events/publisher.py
from app.domains.identity.integration.events.envelope import EventEnvelope
from app.domains.identity.shared.application.event_ports import EventPublisher


class EventPlatformPublisher(EventPublisher):
    """
    Infrastructure adapter for the Event Platform.

    Concrete broker implementation is intentionally deferred.
    """

    async def publish(
        self,
        event: EventEnvelope,
    ) -> None:

        raise NotImplementedError(
            "Event Platform adapter is not configured yet."
        )

No Kafka.

No RabbitMQ.

No AWS EventBridge.

No proveedor específico todavía.

29. Test Domain Event
from uuid import uuid4

from app.domains.identity.shared.domain.events import DomainEvent


def test_domain_event_has_unique_id():

    first = DomainEvent.create()
    second = DomainEvent.create()

    assert first.event_id != second.event_id
30. Test Event Immutability
import pytest

from app.domains.identity.shared.domain.events import DomainEvent


def test_domain_event_is_immutable():

    event = DomainEvent.create()

    with pytest.raises(Exception):
        event.event_id = uuid4()
31. Test Aggregate Event Lifecycle
from uuid import uuid4

from app.domains.identity.shared.domain.aggregate import AggregateRoot
from app.domains.identity.shared.domain.events import DomainEvent


def test_aggregate_collects_events():

    aggregate = AggregateRoot(
        id=uuid4()
    )

    event = DomainEvent.create()

    aggregate.record_event(event)

    events = aggregate.pull_domain_events()

    assert events == [event]

Y:

def test_aggregate_event_collection_is_cleared():

    aggregate = AggregateRoot(
        id=uuid4()
    )

    aggregate.record_event(
        DomainEvent.create()
    )

    aggregate.pull_domain_events()

    assert aggregate.pull_domain_events() == []
32. Test Envelope
from uuid import uuid4

from app.domains.identity.shared.domain.events import DomainEvent
from app.domains.identity.integration.events.mapper import map_domain_event


def test_domain_event_maps_to_envelope():

    event = DomainEvent.create()

    envelope = map_domain_event(
        event,
        event_type="identity.example.created",
        tenant_id=uuid4(),
        correlation_id="correlation-123",
    )

    assert envelope.event_id == event.event_id
    assert envelope.event_type == "identity.example.created"
    assert envelope.event_version == 1
    assert envelope.metadata.correlation_id == "correlation-123"
33. Test no secret leakage

Los tests de eventos deberán comprobar que no se agreguen campos sensibles accidentalmente.

Por ejemplo:

FORBIDDEN_FIELDS = {
    "password",
    "password_hash",
    "access_token",
    "refresh_token",
    "mfa_secret",
    "recovery_code",
    "private_key",
}

Y posteriormente el schema validator deberá rechazar esos campos cuando corresponda.

34. Criterios de aceptación
AC-01 — Domain Events

Identity puede crear Domain Events inmutables.

AC-02 — Aggregate Events

Aggregate Roots pueden registrar y extraer eventos.

AC-03 — Domain Isolation

El Domain no conoce Event Platform.

AC-04 — Integration Envelope

Los Domain Events pueden transformarse a Integration Events.

AC-05 — Versioning

Cada Integration Event tiene versión de contrato.

AC-06 — Correlation

Los eventos soportan correlation context.

AC-07 — Causation

Los eventos soportan causation context.

AC-08 — Tenant Context

Los eventos pueden transportar tenant context cuando corresponde.

AC-09 — Actor Context

El envelope soporta USER/SERVICE/SYSTEM/AI_AGENT.

AC-10 — Outbox Contract

Existe contrato para almacenar eventos en Outbox.

AC-11 — Publisher Contract

Existe contrato de publicación.

AC-12 — Consumer Contract

Existe contrato de consumo.

AC-13 — At-Least-Once

El diseño permite entregas duplicadas y exige consumidores idempotentes.

AC-14 — No Secrets

No se permiten secretos en eventos.

AC-15 — Replay Safety

El diseño soporta replay controlado sin asumir que replay es una nueva operación de negocio.

35. Definition of Done
[ ] DomainEvent implementado
[ ] AggregateRoot integrado
[ ] EventMetadata
[ ] EventEnvelope
[ ] Event mapper
[ ] Event version
[ ] Outbox contract
[ ] Publisher contract
[ ] Consumer contract
[ ] Correlation ID
[ ] Causation ID
[ ] Trace ID
[ ] Actor context
[ ] Tenant context
[ ] No secrets
[ ] Aggregate event tests
[ ] Event immutability tests
[ ] Envelope tests
[ ] Architecture boundary tests
36. Lo que NO implementamos todavía

Muy importante:

❌ PostgreSQL Outbox
❌ Outbox Worker
❌ Event Broker
❌ Kafka
❌ RabbitMQ
❌ EventBridge
❌ Consumer database
❌ Deduplication table
❌ Retry worker
❌ DLQ
❌ Replay engine
❌ Event schema registry

Esas capacidades vendrán posteriormente, siguiendo las ADR ya establecidas.

37. Trazabilidad
IS-001
 │
 └── TASK-001-05
       │
       ├── ESP-0001/12
       │   Identity Events & Integration
       │
       ├── ESP-0001/14
       │   Identity Implementation Requirements
       │
       ├── ADR-007
       │   Transactional Outbox & Event Platform
       │
       ├── ADR-018
       │   Central Shared Event Broker
       │
       ├── ADR-019
       │   Partitioned Event Streaming
       │
       ├── ADR-020
       │   Schema Registry
       │
       ├── ADR-021
       │   Controlled Replay
       │
       └── ADR-022
           Consumer Idempotency & Deduplication
38. Estado después de TASK-001-05

Hasta ahora tenemos:

TASK-001-01
API Runtime
      │
      ▼
TASK-001-02
Identity Domain Package
      │
      ▼
TASK-001-03
Layer Architecture
      │
      ▼
TASK-001-04
Identity Shared Kernel
      │
      ▼
TASK-001-05
Domain Events Foundation

La base comienza a verse así:

                    EVOXA API
                       │
                       ▼
                Identity Domain
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
      Domain       Application   Infrastructure
        │              │              │
        │              │              │
        └──────► Domain Events ◄───────┘
                       │
                       ▼
                Transactional
                    Outbox
                       │
                       ▼
                 Event Platform
                       │
              ┌────────┼────────┐
              ▼        ▼        ▼
          Consumer Consumer Consumer

Punto clave: todavía no estamos publicando eventos reales. Hemos construido el contrato y las fronteras para poder hacerlo correctamente cuando lleguemos a Persistence/Event Infrastructure.

Siguiente tarea

TASK-001-06 — Repository Contracts

Ahí definiremos los contratos de persistencia de Identity, separando definitivamente:

Domain
   ↓
Repository Interface
   ↓
PostgreSQL Implementation

y estableceremos cómo los módulos podrán persistir sus Aggregates sin que el Domain conozca PostgreSQL.
