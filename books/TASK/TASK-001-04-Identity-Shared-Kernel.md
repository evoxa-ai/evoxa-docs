TASK-001-04 — Identity Shared Kernel

IS: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Dependencias: TASK-001-03 — Layer Architecture
Prioridad: Critical
Estado: Ready for Implementation

Esta tarea define el Shared Kernel de Identity.

Aquí debemos ser especialmente estrictos: el Shared Kernel no será un lugar donde poner todo lo que varios módulos necesitan. Será un conjunto pequeño de abstracciones verdaderamente comunes.

La regla será:

Compartir únicamente conceptos estables y transversales; si un concepto pertenece claramente a un módulo, permanece dentro de ese módulo.

Esto es coherente con la decisión de mantener Identity como un Modular Monolith inicialmente, con límites explícitos entre módulos.

1. Objetivo

Crear las abstracciones comunes necesarias para los módulos Identity:

Entity
AggregateRoot
ValueObject
EntityId
DomainEvent
DomainError
Repository
UnitOfWork

y establecer:

qué puede vivir en shared;
qué está prohibido;
convenciones de identidad;
igualdad de entidades;
inmutabilidad de Value Objects;
eventos de dominio;
contratos compartidos;
reglas para evitar un Shared Kernel excesivo.
2. Ubicación

Partimos de:

apps/api/app/domains/identity/shared/

La estructura será:

shared/
│
├── __init__.py
│
├── domain/
│   ├── __init__.py
│   ├── entity.py
│   ├── aggregate.py
│   ├── value_object.py
│   ├── identifiers.py
│   ├── events.py
│   ├── exceptions.py
│   └── repositories.py
│
├── application/
│   ├── __init__.py
│   ├── ports.py
│   └── result.py
│
└── infrastructure/
    ├── __init__.py
    └── unit_of_work.py
3. Qué pertenece al Shared Kernel

Inicialmente:

Elemento	Shared Kernel
Entity	✅
AggregateRoot	✅
ValueObject	✅
EntityId	✅
DomainEvent base	✅
DomainError base	✅
Repository contract base	✅
UnitOfWork contract	✅
Result base	✅
User	❌
Organization	❌
Membership	❌
Role	❌
Permission	❌
Session	❌
MFAFactor	❌
Policy	❌
Delegation	❌
RefreshToken	❌

Los últimos elementos pertenecen a módulos concretos.

4. Entity

Una Entity se distingue por su identidad.

shared/domain/entity.py
from dataclasses import dataclass
from typing import Generic, TypeVar


EntityId = TypeVar("EntityId")


@dataclass
class Entity(Generic[EntityId]):
    """
    Base domain entity.

    Entity equality is based on identity, not on all attributes.
    """

    id: EntityId

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, Entity):
            return NotImplemented

        return (
            type(self) is type(other)
            and self.id == other.id
        )

    def __hash__(self) -> int:
        return hash((type(self), self.id))
Regla

Dos entidades con el mismo id representan la misma identidad de entidad.

Sus demás atributos pueden cambiar.

5. EntityId

Necesitamos una abstracción común para IDs.

shared/domain/identifiers.py
from uuid import UUID, uuid4


EntityId = UUID


def new_entity_id() -> EntityId:
    return uuid4()

No vamos a decidir todavía que todas las tablas físicas utilicen necesariamente UUID como tipo PostgreSQL.

Aquí estamos definiendo el contrato de dominio.

La decisión física de persistencia pertenece a la capa de Persistence.

6. Aggregate Root

Los agregados necesitan una frontera explícita.

shared/domain/aggregate.py
from dataclasses import dataclass, field

from app.domains.identity.shared.domain.entity import Entity
from app.domains.identity.shared.domain.events import DomainEvent


@dataclass
class AggregateRoot(Entity):
    """
    Base class for Identity aggregate roots.

    Aggregate roots own the consistency boundary of an aggregate.
    """

    _domain_events: list[DomainEvent] = field(
        default_factory=list,
        init=False,
        repr=False,
    )

    def record_event(self, event: DomainEvent) -> None:
        self._domain_events.append(event)

    def pull_domain_events(self) -> list[DomainEvent]:
        events = list(self._domain_events)
        self._domain_events.clear()

        return events

Esto permitirá posteriormente:

User
 └── UserCreated

Organization
 └── OrganizationCreated

Membership
 └── MembershipRevoked

Session
 └── SessionRevoked

El Aggregate Root acumula eventos de dominio.

Todavía no los publica.

7. Domain Event
shared/domain/events.py

Vamos a mejorar el contrato que habíamos creado en TASK-001-03.

from dataclasses import dataclass
from datetime import datetime, timezone
from uuid import UUID, uuid4


@dataclass(frozen=True)
class DomainEvent:
    """
    Base immutable domain event.

    This represents a domain fact inside Identity.
    It is not the external integration-event envelope.
    """

    event_id: UUID
    occurred_at: datetime

    @classmethod
    def create(cls) -> "DomainEvent":
        return cls(
            event_id=uuid4(),
            occurred_at=datetime.now(timezone.utc),
        )
Importante

Tenemos dos conceptos diferentes:

Domain Event
      │
      ▼
Outbox Event
      │
      ▼
Integration Event
      │
      ▼
Event Platform

No debemos mezclarlos.

El Shared Kernel solamente define el primer concepto.

La publicación y el envelope de integración serán tratados en:

TASK-001-05 — Domain Events Foundation
8. Value Object

Un Value Object no tiene identidad propia.

Su igualdad depende de su valor.

shared/domain/value_object.py
from dataclasses import dataclass
from typing import Generic, TypeVar


T = TypeVar("T")


@dataclass(frozen=True)
class ValueObject(Generic[T]):
    """
    Immutable base Value Object.

    Equality is value-based.
    """

    value: T

Por ejemplo, posteriormente podríamos tener:

EmailAddress
UserStatus
OrganizationId
MembershipId
PermissionName

pero no los crearemos aquí todavía.

¿Por qué?

Porque:

EmailAddress

puede tener reglas específicas de Account.

Y:

PermissionName

pertenece a Role & Permission.

El Shared Kernel proporciona la abstracción, no todos los Value Objects del sistema.

9. Domain Exceptions
shared/domain/exceptions.py
class DomainError(Exception):
    """
    Base exception for domain errors.
    """


class DomainRuleViolation(DomainError):
    """
    Raised when a domain invariant is violated.
    """


class InvalidEntityState(DomainError):
    """
    Raised when an entity is in an invalid state
    for the requested operation.
    """

No hay ninguna referencia a:

HTTP
FastAPI
PostgreSQL
Redis

Esto es obligatorio.

10. Repository Contract

Los repositorios pertenecen conceptualmente a la frontera del Domain/Application.

El Shared Kernel puede proporcionar únicamente una abstracción mínima.

shared/domain/repositories.py
from abc import ABC, abstractmethod
from typing import Generic, TypeVar


T = TypeVar("T")


class Repository(ABC, Generic[T]):
    """
    Generic repository contract.

    Concrete implementations belong to Infrastructure.
    """

    @abstractmethod
    async def get_by_id(
        self,
        entity_id,
    ) -> T | None:
        raise NotImplementedError

    @abstractmethod
    async def save(
        self,
        entity: T,
    ) -> None:
        raise NotImplementedError

Los repositorios específicos estarán en cada módulo.

Por ejemplo, posteriormente:

account/domain/
└── repositories.py

podrá definir:

class UserRepository(Repository[User]):
    ...

No:

shared/domain/user_repository.py

porque UserRepository pertenece a Account.

11. Unit of Work

El contrato común:

shared/application/ports.py
from abc import ABC, abstractmethod


class UnitOfWork(ABC):

    async def __aenter__(self):
        return self

    async def __aexit__(
        self,
        exc_type,
        exc_value,
        traceback,
    ):
        if exc_type is not None:
            await self.rollback()

    @abstractmethod
    async def commit(self) -> None:
        raise NotImplementedError

    @abstractmethod
    async def rollback(self) -> None:
        raise NotImplementedError

La implementación PostgreSQL vendrá posteriormente.

12. Result

Mantenemos el resultado de Application:

shared/application/result.py
from dataclasses import dataclass
from typing import Generic, TypeVar


T = TypeVar("T")


@dataclass(frozen=True)
class Result(Generic[T]):
    value: T | None = None
    error: Exception | None = None

    @property
    def is_success(self) -> bool:
        return self.error is None

    @property
    def is_failure(self) -> bool:
        return self.error is not None

Esto es un contrato de Application, no de HTTP.

13. Infrastructure

Infrastructure puede implementar contratos.

Por ejemplo:

shared/infrastructure/unit_of_work.py
from app.domains.identity.shared.application.ports import UnitOfWork


class InfrastructureUnitOfWork(UnitOfWork):
    """
    Base infrastructure implementation.

    Concrete database behavior will be introduced
    by the persistence implementation.
    """

    async def commit(self) -> None:
        raise NotImplementedError(
            "Concrete persistence implementation is required."
        )

    async def rollback(self) -> None:
        raise NotImplementedError(
            "Concrete persistence implementation is required."
        )

Todavía no hay SQL.

14. Qué NO debe entrar al Shared Kernel

Esta es probablemente la parte más importante de esta tarea.

❌ No colocar:
User
Organization
Membership
Role
Permission
Session
RefreshToken
MFAFactor
MFAChallenge
SecurityPolicy
Delegation
AuditEvent
SecurityEvent
❌ Tampoco:
FastAPI
SQLAlchemy
PostgreSQL models
Redis clients
JWT implementation
OAuth implementation
WebAuthn implementation
Email providers
❌ Ni:
UserService
AuthenticationService
AuthorizationService
PolicyEngine
MFAService

Porque esos pertenecen a módulos específicos.

15. Regla de promoción al Shared Kernel

Antes de mover cualquier concepto a shared, deberá cumplir:

1. ¿Es utilizado por múltiples módulos?
        │
        ├── NO → permanece en su módulo
        │
        └── SÍ
             │
             ▼
2. ¿Tiene significado verdaderamente transversal?
             │
             ├── NO → permanece en módulo
             │
             └── SÍ
                  │
                  ▼
3. ¿Es estable?
                  │
                  ├── NO → no compartir
                  │
                  └── SÍ
                       │
                       ▼
                 Shared Kernel

Esto evita que Shared Kernel se convierta en una dependencia gigantesca.

16. Regla de dependencia

Dentro de Identity:

Account
Organization
Membership
Authentication
Authorization
...
       │
       ▼
Shared Kernel

Pero:

Shared Kernel
       │
       ├── ❌ Account
       ├── ❌ Membership
       ├── ❌ MFA
       └── ❌ Authorization

El flujo debe ser unidireccional.

17. Evitar dependencia circular

Incorrecto:

Account
   ↓
Shared
   ↓
Membership
   ↓
Account

Correcto:

Account ──────┐
Membership ───┤
MFA ──────────┤
Authorization ┤
              ▼
         Shared Kernel

El Shared Kernel no conoce los módulos concretos.

18. Aggregate Boundary

Esta abstracción será fundamental cuando empecemos a crear los agregados.

Por ejemplo:

User Aggregate
│
├── User identity
├── credentials state
├── account state
└── domain events

y:

Membership Aggregate
│
├── organization relationship
├── membership state
├── assigned roles
└── domain events

No significa que un agregado pueda modificar directamente otro.

La interacción será mediante:

Application
Contracts
Domain Events

según el caso.

19. Invariantes del Shared Kernel

Definimos estas reglas:

SHARED-001

Entities tienen identidad.

SHARED-002

Entity equality es identity-based.

SHARED-003

Value Objects son immutable.

SHARED-004

Aggregate Roots son fronteras de consistencia.

SHARED-005

Domain Events son immutable.

SHARED-006

Domain no conoce infraestructura.

SHARED-007

Shared Kernel no conoce módulos concretos.

SHARED-008

Los Repository Contracts no contienen detalles de persistencia.

SHARED-009

UnitOfWork es un contrato, no una implementación PostgreSQL.

SHARED-010

No se permiten secretos en objetos del Shared Kernel.

20. Tests

Crearemos:

apps/api/tests/domains/identity/
└── test_shared_kernel.py
Test Entity
from dataclasses import dataclass
from uuid import uuid4

from app.domains.identity.shared.domain.entity import Entity


@dataclass
class ExampleEntity(Entity):
    name: str = ""


def test_entities_with_same_id_are_equal():

    entity_id = uuid4()

    first = ExampleEntity(
        id=entity_id,
        name="first",
    )

    second = ExampleEntity(
        id=entity_id,
        name="second",
    )

    assert first == second

Esto demuestra:

same identity
≠
same attributes
21. Test Value Object
from app.domains.identity.shared.domain.value_object import ValueObject


def test_value_object_is_value_based():

    first = ValueObject("EVOXA")
    second = ValueObject("EVOXA")

    assert first == second

Y además:

import pytest


def test_value_object_is_immutable():

    value = ValueObject("EVOXA")

    with pytest.raises(Exception):
        value.value = "OTHER"
22. Test Aggregate Events
from uuid import uuid4

from app.domains.identity.shared.domain.aggregate import AggregateRoot
from app.domains.identity.shared.domain.events import DomainEvent


def test_aggregate_records_domain_events():

    aggregate = AggregateRoot(
        id=uuid4()
    )

    event = DomainEvent.create()

    aggregate.record_event(event)

    events = aggregate.pull_domain_events()

    assert events == [event]


def test_pull_events_clears_event_collection():

    aggregate = AggregateRoot(
        id=uuid4()
    )

    aggregate.record_event(
        DomainEvent.create()
    )

    aggregate.pull_domain_events()

    assert aggregate.pull_domain_events() == []
23. Test de inmutabilidad del evento
import pytest

from app.domains.identity.shared.domain.events import DomainEvent


def test_domain_event_is_immutable():

    event = DomainEvent.create()

    with pytest.raises(Exception):
        event.event_id = None
24. Test arquitectónico del Shared Kernel

Debemos verificar que shared no importe módulos concretos.

La prueba puede inspeccionar los archivos Python:

from pathlib import Path


SHARED_ROOT = (
    Path(__file__).resolve().parents[3]
    / "app"
    / "domains"
    / "identity"
    / "shared"
)


FORBIDDEN_IMPORTS = [
    "identity.account",
    "identity.organization",
    "identity.membership",
    "identity.authentication",
    "identity.authorization",
    "identity.role_permission",
    "identity.policy",
    "identity.session",
    "identity.token",
    "identity.mfa",
    "identity.recovery",
]


def test_shared_kernel_does_not_import_identity_modules():

    for python_file in SHARED_ROOT.rglob("*.py"):

        content = python_file.read_text(
            encoding="utf-8"
        )

        for forbidden in FORBIDDEN_IMPORTS:

            assert forbidden not in content, (
                f"Shared Kernel depends on "
                f"Identity module: {forbidden}"
            )

Esto establece una primera barrera automática.

25. Resultado final

Después de esta tarea tendremos:

identity/
│
├── account/
├── organization/
├── membership/
├── authentication/
├── authorization/
├── role_permission/
├── policy/
├── session/
├── token/
├── mfa/
├── recovery/
├── federation/
├── provisioning/
├── audit/
├── security_events/
├── integration/
│
└── shared/
    │
    ├── domain/
    │   ├── entity.py
    │   ├── aggregate.py
    │   ├── value_object.py
    │   ├── identifiers.py
    │   ├── events.py
    │   ├── exceptions.py
    │   └── repositories.py
    │
    ├── application/
    │   ├── ports.py
    │   └── result.py
    │
    └── infrastructure/
        └── unit_of_work.py
26. Definition of Done
[ ] Entity base
[ ] Entity identity semantics
[ ] EntityId
[ ] AggregateRoot
[ ] DomainEvent
[ ] ValueObject
[ ] Domain exceptions
[ ] Repository contract
[ ] UnitOfWork contract
[ ] Result
[ ] Infrastructure placeholder
[ ] Shared Kernel dependency rules
[ ] Entity tests
[ ] ValueObject tests
[ ] DomainEvent tests
[ ] Aggregate tests
[ ] Architecture tests
[ ] No module-specific business logic
[ ] No infrastructure dependency from Domain
27. Trazabilidad
IS-001
 │
 └── TASK-001-04
       │
       ├── ESP-0001/01
       ├── ESP-0001/02
       ├── ESP-0001/14
       │
       ├── ADR-067
       │   Identity Domain Integration
       │
       └── ADR-069
           Identity Module Dependency Matrix
Resultado arquitectónico

Con TASK-001-01 teníamos el runtime.

Con TASK-001-02 creamos el Bounded Domain y sus módulos.

Con TASK-001-03 establecimos las capas.

Ahora con TASK-001-04 tenemos el lenguaje técnico común mínimo:

                 EVOXA Identity
                       │
             ┌─────────┴─────────┐
             │   Shared Kernel   │
             │                   │
             │ Entity            │
             │ AggregateRoot     │
             │ ValueObject        │
             │ EntityId           │
             │ DomainEvent        │
             │ DomainError        │
             │ Repository         │
             │ UnitOfWork         │
             └─────────┬─────────┘
                       │
       ┌───────────────┼────────────────┐
       ▼               ▼                ▼
    Account       Membership      Authorization
       │               │                │
       └───────────────┼────────────────┘
                       ▼
                Identity Domain

Siguiente tarea: TASK-001-05 — Domain Events Foundation.

Ahí sí entraremos en la siguiente pieza importante: cómo un Aggregate genera eventos, cómo esos eventos llegan al Transactional Outbox, cómo se construye el envelope de integración y cómo se prepara la publicación hacia el Event Platform, respetando At-Least-Once, idempotencia y trazabilidad.
