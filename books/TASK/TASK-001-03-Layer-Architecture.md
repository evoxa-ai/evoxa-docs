TASK-001-03 — Layer Architecture

IS: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Dependencias: TASK-001-02 — Identity Domain Package
Prioridad: Critical
Estado: Ready for Implementation

En esta tarea vamos a convertir la estructura creada en TASK-001-02 en una arquitectura por capas real, estableciendo qué puede depender de qué y evitando que el código de EVOXA se acople desde el comienzo.

La regla central será:

API
 │
 ▼
Application
 │
 ▼
Domain
 ▲
 │
Infrastructure

Pero hay una precisión importante: Infrastructure implementa contratos definidos por Domain/Application; no significa que Domain importe Infrastructure.

1. Objetivo

Implementar la base arquitectónica de las cuatro capas:

Domain
Application
Infrastructure
API

y establecer:

responsabilidades;
dirección de dependencias;
contratos;
separación de infraestructura;
reglas de importación;
composición de dependencias;
tests arquitectónicos.

No implementaremos todavía funcionalidades de Identity.

2. Arquitectura objetivo

Para cada módulo:

identity/
└── account/
    ├── domain/
    ├── application/
    ├── infrastructure/
    └── api/

La dirección conceptual será:

                 ┌───────────────┐
                 │      API      │
                 └───────┬───────┘
                         │
                         ▼
                 ┌───────────────┐
                 │  Application  │
                 └───────┬───────┘
                         │
                         ▼
                 ┌───────────────┐
                 │    Domain     │
                 └───────────────┘
                         ▲
                         │
                 ┌───────┴───────┐
                 │ Infrastructure│
                 └───────────────┘

Esto materializa la arquitectura definida para Identity.

3. Responsabilidad de cada capa
Domain

El Domain contiene las reglas propias del negocio.

Podrá contener:

Entities
Value Objects
Aggregates
Domain Services
Domain Events
Repository Contracts
Domain Exceptions

No debe conocer:

FastAPI
HTTP
PostgreSQL
Redis
Kafka
Docker
AWS
Kubernetes

Por ejemplo:

# ❌ No permitido

from fastapi import HTTPException

dentro de:

identity/*/domain/
4. Application

Application coordina casos de uso.

Ejemplo futuro:

CreateUser
AuthenticateUser
CreateOrganization
AddMembership
AssignRole
AuthorizeRequest
EnrollMFA
RotateRefreshToken

Application puede utilizar:

Domain
Repository Contracts
Domain Services
Application Services

pero no debe contener directamente SQL o detalles HTTP.

Incorrecto:

class CreateUser:

    def execute(self):
        db.execute("INSERT INTO users...")

Correcto conceptualmente:

class CreateUser:

    def execute(self, command):
        user = User.create(...)
        self.user_repository.save(user)

La implementación concreta del repository estará en Infrastructure.

5. Infrastructure

Infrastructure contiene adaptadores tecnológicos.

Ejemplos futuros:

PostgreSQL
Redis
Event Platform
KMS
Secret Manager
Email Provider
OIDC Provider
SAML Provider
WebAuthn infrastructure

Por ejemplo:

identity/account/infrastructure/
└── persistence/
    └── postgres_user_repository.py

Infrastructure conoce tecnología.

Domain no.

6. API

API traduce el mundo HTTP al mundo Application.

Ejemplo:

HTTP Request
     │
     ▼
API
     │
     ▼
Command / Query
     │
     ▼
Application

API será responsable de:

HTTP;
request validation;
response mapping;
status codes;
authentication middleware integration;
correlation context;
API contracts.

No debe contener reglas de negocio.

7. Regla de dependencia

La regla será:

API → Application
Application → Domain
Infrastructure → Domain/Application Contracts

Y nunca:

Domain → API                    ❌
Domain → Infrastructure        ❌
Domain → FastAPI               ❌
Domain → PostgreSQL            ❌
Application → FastAPI          ❌
Application → SQL directo      ❌
8. Primer contrato arquitectónico

Para hacer esto verificable, vamos a crear un pequeño contrato de arquitectura.

identity/shared/domain/contracts.py
from abc import ABC, abstractmethod
from typing import Generic, TypeVar


T = TypeVar("T")


class Repository(ABC, Generic[T]):
    """
    Base contract for domain repositories.

    Concrete persistence implementations belong to Infrastructure.
    """

    @abstractmethod
    async def get_by_id(self, entity_id: str) -> T | None:
        raise NotImplementedError

    @abstractmethod
    async def save(self, entity: T) -> None:
        raise NotImplementedError

Esto no implementa PostgreSQL.

Solamente establece el contrato.

9. Domain Exceptions

Creamos una base para errores de dominio.

identity/shared/domain/exceptions.py
class DomainError(Exception):
    """
    Base exception for domain-level errors.
    """


class DomainRuleViolation(DomainError):
    """
    Raised when a domain invariant is violated.
    """

Esto evita que el Domain dependa de:

HTTPException
JSONResponse
FastAPI
10. Entity base

También podemos establecer una abstracción mínima para entidades.

identity/shared/domain/entity.py
from dataclasses import dataclass
from typing import Generic, TypeVar


EntityId = TypeVar("EntityId")


@dataclass
class Entity(Generic[EntityId]):
    id: EntityId

Es deliberadamente simple.

Todavía no estamos definiendo:

User
Organization
Membership

porque eso pertenece a las siguientes tareas.

11. Domain Event base

Identity tendrá posteriormente muchos eventos:

UserCreated
UserSuspended
MembershipRevoked
RoleAssigned
SessionRevoked
MFAFactorActivated
RefreshTokenReused

Pero en esta tarea solamente establecemos la abstracción.

identity/shared/domain/events.py
from dataclasses import dataclass
from datetime import datetime
from uuid import UUID, uuid4


@dataclass(frozen=True)
class DomainEvent:
    event_id: UUID
    occurred_at: datetime

    @classmethod
    def create(cls):
        return cls(
            event_id=uuid4(),
            occurred_at=datetime.utcnow(),
        )
Nota

Este objeto todavía no es el Event Envelope de integración.

El envelope completo definido para eventos de EVOXA será implementado cuando lleguemos a:

TASK-001-05 — Domain Events Foundation

Aquí únicamente establecemos el concepto de Domain Event.

12. Application Result

La Application Layer necesitará una forma consistente de representar resultados.

Podemos establecer:

identity/shared/application/result.py
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

No es todavía un contrato HTTP.

La conversión:

Application Result
       ↓
HTTP Response

corresponderá a API.

13. Application Ports

La Application Layer podrá necesitar servicios externos.

No queremos:

from redis import Redis

dentro de un caso de uso.

En su lugar:

Application
     │
     ▼
Port / Contract
     │
     ▼
Infrastructure Adapter

Ejemplo:

identity/shared/application/ports.py
from abc import ABC, abstractmethod


class UnitOfWork(ABC):

    @abstractmethod
    async def __aenter__(self):
        raise NotImplementedError

    @abstractmethod
    async def __aexit__(
        self,
        exc_type,
        exc_value,
        traceback,
    ):
        raise NotImplementedError

    @abstractmethod
    async def commit(self) -> None:
        raise NotImplementedError

    @abstractmethod
    async def rollback(self) -> None:
        raise NotImplementedError

Esto prepara:

Application
     │
     ▼
UnitOfWork contract
     │
     ▼
PostgreSQL implementation

La implementación real llegará en TASK-001-07.

14. Infrastructure Adapter

Podemos crear una implementación placeholder para demostrar la dirección.

identity/shared/infrastructure/unit_of_work.py
from app.domains.identity.shared.application.ports import UnitOfWork


class InfrastructureUnitOfWork(UnitOfWork):
    """
    Infrastructure implementation placeholder.

    Database-specific implementation will be introduced
    in the persistence tasks.
    """

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

    async def commit(self) -> None:
        raise NotImplementedError(
            "Persistence implementation is not configured yet."
        )

    async def rollback(self) -> None:
        raise NotImplementedError(
            "Persistence implementation is not configured yet."
        )

Esto es intencionalmente incompleto.

No vamos a inventar todavía el comportamiento PostgreSQL.

15. Dependency Injection

La composición concreta debe producirse fuera del Domain.

Conceptualmente:

                 Composition Root
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   Application      Repository       Services
        │              │              │
        └──────────────┼──────────────┘
                       ▼
                 Infrastructure

Para EVOXA, inicialmente la composición estará en la aplicación:

app/
└── core/
    └── dependencies/

No debemos introducir un framework DI adicional todavía.

FastAPI ya proporciona mecanismos suficientes para dependency injection a nivel API.

16. Composition Root

Crearemos:

apps/api/app/core/dependencies/
├── __init__.py
└── identity.py
identity.py
"""
Identity dependency composition.

Concrete infrastructure dependencies will be wired here
as the implementation progresses.
"""


def get_identity_dependencies() -> dict:
    """
    Composition point for Identity dependencies.
    """

    return {}

Por ahora es solamente el punto de composición.

17. Estructura final después de TASK-001-03

La estructura quedará:

apps/api/app/
│
├── api/
│   └── v1/
│
├── core/
│   ├── config.py
│   ├── errors.py
│   ├── middleware/
│   │   └── correlation.py
│   │
│   └── dependencies/
│       ├── __init__.py
│       └── identity.py
│
└── domains/
    └── identity/
        │
        ├── account/
        │   ├── domain/
        │   ├── application/
        │   ├── infrastructure/
        │   └── api/
        │
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
            ├── domain/
            │   ├── entity.py
            │   ├── events.py
            │   ├── exceptions.py
            │   └── contracts.py
            │
            ├── application/
            │   ├── ports.py
            │   └── result.py
            │
            └── infrastructure/
                └── unit_of_work.py
18. Regla especial de shared

Tenemos que controlar mucho esta carpeta.

No queremos terminar con:

shared/
├── user.py
├── organization.py
├── auth.py
├── permissions.py
├── everything.py

Eso destruiría los límites del Bounded Domain.

shared solamente contendrá abstracciones que:

sean realmente comunes;
tengan significado transversal;
no pertenezcan claramente a un módulo;
no creen dependencia circular.
19. Ejemplo de flujo correcto

Cuando eventualmente implementemos:

POST /api/v1/users

la arquitectura será:

HTTP
 │
 ▼
User API
 │
 ▼
CreateUserUseCase
 │
 ├── User domain
 │
 ├── UserRepository contract
 │
 └── UnitOfWork contract
 │
 ▼
Infrastructure
 │
 └── PostgreSQL

Nunca:

HTTP
 │
 ▼
FastAPI Controller
 │
 └── SQLAlchemy INSERT ❌
20. Ejemplo con eventos

Posteriormente:

CreateUserUseCase
       │
       ▼
   User Aggregate
       │
       ▼
 UserCreated Domain Event
       │
       ▼
 Unit of Work
       │
       ├── persist User
       │
       └── persist OutboxEvent

Después:

Outbox
   ↓
Event Platform
   ↓
Consumers

Eso será implementado en TASK-001-05 y las tareas posteriores.

21. Architecture Boundary Tests

Ahora debemos empezar a hacer que las reglas sean ejecutables.

Una primera prueba:

tests/domains/identity/test_layer_architecture.py
from pathlib import Path


IDENTITY_ROOT = (
    Path(__file__).resolve().parents[3]
    / "app"
    / "domains"
    / "identity"
)


MODULES = [
    "account",
    "organization",
    "membership",
    "authentication",
    "authorization",
    "role_permission",
    "policy",
    "session",
    "token",
    "mfa",
    "recovery",
    "federation",
    "provisioning",
    "audit",
    "security_events",
    "integration",
]


LAYERS = [
    "domain",
    "application",
    "infrastructure",
    "api",
]


def test_all_identity_modules_have_layers():
    for module in MODULES:

        module_path = IDENTITY_ROOT / module

        for layer in LAYERS:

            assert (
                module_path / layer
            ).is_dir(), (
                f"Missing {layer} layer "
                f"for module {module}"
            )


def test_shared_layers_exist():

    shared = IDENTITY_ROOT / "shared"

    assert (shared / "domain").is_dir()
    assert (shared / "application").is_dir()
    assert (shared / "infrastructure").is_dir()
22. Boundary enforcement

Más adelante agregaremos una herramienta como:

import-linter

o una implementación equivalente para verificar automáticamente:

domain
  ❌ fastapi
  ❌ sqlalchemy
  ❌ redis
  ❌ httpx

application
  ❌ fastapi
  ❌ sqlalchemy
  ❌ redis

API
  ✅ application

Infrastructure
  ✅ domain
  ✅ application

No fijamos todavía una herramienta concreta porque la decisión tecnológica del architecture linting no está cerrada en esta tarea.

23. Criterios de aceptación
AC-01 — Layer Structure

Cada módulo Identity tiene:

domain
application
infrastructure
api
AC-02 — Domain Independence

Domain no depende de frameworks HTTP ni infraestructura.

AC-03 — Application Isolation

Application no contiene SQL ni acceso directo a infraestructura tecnológica.

AC-04 — Infrastructure

Infrastructure implementa contratos sin introducir dependencias hacia API.

AC-05 — API

API solamente adapta HTTP hacia Application.

AC-06 — Shared Kernel

Shared contiene solamente abstracciones transversales.

AC-07 — Dependency Direction

Las dependencias respetan las reglas arquitectónicas.

AC-08 — Testability

Las reglas estructurales son verificables automáticamente.

24. Definition of Done
[ ] Domain layer definida
[ ] Application layer definida
[ ] Infrastructure layer definida
[ ] API layer definida
[ ] Shared Kernel mínimo
[ ] Entity base
[ ] Domain Event base
[ ] Domain exceptions
[ ] Repository contract
[ ] UnitOfWork contract
[ ] Infrastructure adapter placeholder
[ ] Composition Root inicial
[ ] Architecture tests
[ ] No lógica de negocio prematura
[ ] No dependencia Domain → Infrastructure
[ ] No dependencia Domain → FastAPI
[ ] No SQL en Application
25. Trazabilidad
IS-001
 │
 └── TASK-001-03
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
Resultado de TASK-001-03

Con esta tarea ya no tendremos solamente:

identity/
├── account/
├── authorization/
├── mfa/
└── ...

sino una arquitectura explícita:

                  EVOXA API
                     │
                     ▼
                Identity API
                     │
                     ▼
                Application
                     │
                     ▼
                   Domain
                     ▲
                     │
              Infrastructure
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
      PostgreSQL   Redis    Event Platform

Todavía no conectamos PostgreSQL, Redis ni Event Platform. Eso respeta el orden de las tareas.

Siguiente

TASK-001-04 — Identity Shared Kernel

Ahí definiremos con mayor precisión qué elementos pueden compartirse entre los módulos de Identity —por ejemplo Entity, AggregateRoot, ValueObject, DomainEvent, identificadores, errores y contratos comunes— y, sobre todo, qué queda explícitamente fuera del Shared Kernel para evitar crear un acoplamiento global.
