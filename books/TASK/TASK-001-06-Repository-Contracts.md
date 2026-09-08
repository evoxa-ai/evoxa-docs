TASK-001-06 — Repository Contracts

IS: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Dependencias: TASK-001-05 — Domain Events Foundation
Prioridad: Critical
Estado: Ready for Implementation

Esta tarea define los contratos de persistencia de Identity.

El objetivo no es conectar todavía PostgreSQL. El objetivo es establecer una frontera clara entre el dominio y la infraestructura:

┌──────────────────────────────┐
│           DOMAIN             │
│                              │
│ User / Organization /        │
│ Membership / Session / ...   │
│                              │
│ Repository Contract          │
└──────────────┬───────────────┘
               │
               │ Interface
               ▼
┌──────────────────────────────┐
│       INFRASTRUCTURE         │
│                              │
│ PostgreSQL Repository        │
│ SQL / ORM / Transactions     │
└──────────────────────────────┘

La decisión arquitectónica importante es:

El Domain define qué necesita para persistir; Infrastructure decide cómo hacerlo.

Esto mantiene alineada la implementación con la separación de módulos y contratos de Identity.

1. Objetivo

Implementar:

Repository base;
contratos específicos por módulo;
criterios de consulta;
especificación de paginación;
manejo conceptual de concurrencia;
contrato de Unit of Work;
separación Repository / Unit of Work;
reglas de tenant isolation;
contrato de persistencia de agregados;
arquitectura preparada para PostgreSQL.

Todavía no implementaremos SQL ni modelos ORM.

2. Qué problema estamos resolviendo

Queremos evitar esto:

class UserService:

    def create_user(self):
        db.execute(
            "INSERT INTO users ..."
        )

Porque entonces:

Application
   ↓
PostgreSQL

queda acoplado.

La arquitectura correcta será:

CreateUserUseCase
       │
       ▼
UserRepository
       │
       ▼
PostgreSQLUserRepository

El Use Case solamente conoce:

UserRepository

no:

PostgreSQL
SQLAlchemy
asyncpg
connection pool
3. Regla de ownership

Cada módulo es dueño de sus propios repositorios.

Por ejemplo:

Account
 └── UserRepository

Organization
 └── OrganizationRepository

Membership
 └── MembershipRepository

Session
 └── SessionRepository

Token
 └── RefreshTokenRepository

No tendremos:

shared/
└── repositories/
    ├── UserRepository
    ├── RoleRepository
    ├── MembershipRepository
    └── ...

Eso violaría el ownership de módulos.

El Shared Kernel solamente proporciona las abstracciones comunes.

4. Estructura

La estructura evolucionará a:

apps/api/app/domains/identity/
│
├── account/
│   └── domain/
│       └── repositories.py
│
├── organization/
│   └── domain/
│       └── repositories.py
│
├── membership/
│   └── domain/
│       └── repositories.py
│
├── authentication/
│   └── domain/
│       └── repositories.py
│
├── authorization/
│   └── domain/
│       └── repositories.py
│
├── role_permission/
│   └── domain/
│       └── repositories.py
│
├── policy/
│   └── domain/
│       └── repositories.py
│
├── session/
│   └── domain/
│       └── repositories.py
│
├── token/
│   └── domain/
│       └── repositories.py
│
├── mfa/
│   └── domain/
│       └── repositories.py
│
├── recovery/
│   └── domain/
│       └── repositories.py
│
├── federation/
│   └── domain/
│       └── repositories.py
│
├── provisioning/
│   └── domain/
│       └── repositories.py
│
├── audit/
│   └── domain/
│       └── repositories.py
│
├── security_events/
│   └── domain/
│       └── repositories.py
│
├── integration/
│   └── domain/
│       └── repositories.py
│
└── shared/
    ├── domain/
    │   └── repositories.py
    │
    └── application/
        └── ports.py

Pero no necesitamos crear todos los contratos específicos todavía.

En esta tarea vamos a establecer el patrón y crear únicamente los contratos necesarios para validar la arquitectura.

5. Repository base

Ya tenemos:

identity/shared/domain/repositories.py

Lo vamos a refinar.

from abc import ABC, abstractmethod
from typing import Generic, TypeVar

from app.domains.identity.shared.domain.identifiers import EntityId


T = TypeVar("T")


class Repository(ABC, Generic[T]):
    """
    Base repository contract.

    Concrete implementations belong to Infrastructure.
    """

    @abstractmethod
    async def get_by_id(
        self,
        entity_id: EntityId,
    ) -> T | None:
        raise NotImplementedError

    @abstractmethod
    async def save(
        self,
        entity: T,
    ) -> None:
        raise NotImplementedError

    @abstractmethod
    async def delete(
        self,
        entity_id: EntityId,
    ) -> None:
        raise NotImplementedError

Este contrato es deliberadamente pequeño.

No agregaremos:

find_everything()
execute_sql()
raw_query()

porque no pertenecen al contrato genérico.

6. ¿Por qué no update()?

No necesitamos obligar a todos los repositorios a implementar:

update()

porque en Domain normalmente trabajaremos con Aggregates:

load
 ↓
modify aggregate
 ↓
save

Por ejemplo:

user = await repository.get_by_id(user_id)

user.suspend()

await repository.save(user)

El Repository representa la persistencia del estado del Aggregate.

7. Account Repository

Ahora creamos el primer contrato específico.

identity/account/domain/repositories.py
from abc import abstractmethod

from app.domains.identity.shared.domain.repositories import Repository


class UserRepository(Repository):
    """
    Persistence contract owned by Account.
    """

    @abstractmethod
    async def get_by_email(
        self,
        email: str,
    ):
        raise NotImplementedError

Aquí todavía no importamos User, porque User será implementado en la tarea correspondiente.

Cuando exista el Aggregate:

class UserRepository(Repository[User]):
    ...

podremos tiparlo correctamente.

8. Regla importante sobre email

No debemos poner en el Repository:

get_by_email_and_password()

porque:

Password verification

es responsabilidad de Authentication/Account y sus servicios de seguridad, no una consulta de persistencia arbitraria.

El Repository solamente recupera el Aggregate.

9. Organization Repository
identity/organization/domain/repositories.py
from abc import abstractmethod

from app.domains.identity.shared.domain.repositories import Repository


class OrganizationRepository(Repository):

    @abstractmethod
    async def get_by_name(
        self,
        name: str,
    ):
        raise NotImplementedError

La regla exacta de unicidad de nombre todavía no queda definida aquí.

La unicidad será una combinación de:

Domain Rule
+
Database Constraint

cuando lleguemos a Persistence.

10. Membership Repository
identity/membership/domain/repositories.py
from abc import abstractmethod
from uuid import UUID

from app.domains.identity.shared.domain.repositories import Repository


class MembershipRepository(Repository):

    @abstractmethod
    async def get_by_user_and_organization(
        self,
        user_id: UUID,
        organization_id: UUID,
    ):
        raise NotImplementedError

    @abstractmethod
    async def list_for_user(
        self,
        user_id: UUID,
    ):
        raise NotImplementedError

    @abstractmethod
    async def list_for_organization(
        self,
        organization_id: UUID,
    ):
        raise NotImplementedError

Aquí aparece un concepto fundamental:

Membership = access boundary

Por lo tanto, las consultas deberán ser tenant-aware.

11. Tenant Isolation

No debemos permitir un Repository como:

list_all_memberships()

sin contexto.

En general, la consulta deberá conocer:

organization_id

o un TenantContext explícito cuando el contrato lo requiera.

Por ejemplo:

async def list_for_organization(
    self,
    organization_id: UUID,
):
    ...

Esto crea una primera barrera arquitectónica.

12. Session Repository

El módulo Session tendrá:

identity/session/domain/repositories.py
from abc import abstractmethod
from uuid import UUID

from app.domains.identity.shared.domain.repositories import Repository


class SessionRepository(Repository):

    @abstractmethod
    async def get_by_session_id(
        self,
        session_id: UUID,
    ):
        raise NotImplementedError

    @abstractmethod
    async def list_active_for_user(
        self,
        user_id: UUID,
    ):
        raise NotImplementedError

    @abstractmethod
    async def revoke(
        self,
        session_id: UUID,
    ) -> None:
        raise NotImplementedError

Aquí debemos notar algo.

revoke() es una operación con significado de dominio.

Por eso más adelante debemos decidir si:

Repository.revoke()

es realmente necesario o si el patrón correcto será:

load session
 ↓
session.revoke()
 ↓
repository.save()

La segunda opción será preferida cuando la operación implique una transición de Aggregate.

Por eso no debemos multiplicar métodos especializados innecesariamente.

13. Token Repository

Refresh Tokens tienen requisitos especiales.

El Repository deberá eventualmente soportar:

family
parent
token hash
status
expiration
rotation
reuse detection

pero todavía no definiremos el contrato completo.

Eso corresponde a:

ADR-025
+
Token implementation

Por ahora:

identity/token/domain/repositories.py
from abc import abstractmethod
from uuid import UUID

from app.domains.identity.shared.domain.repositories import Repository


class RefreshTokenRepository(Repository):

    @abstractmethod
    async def get_active_by_id(
        self,
        token_id: UUID,
    ):
        raise NotImplementedError

    @abstractmethod
    async def get_family(
        self,
        family_id: UUID,
    ):
        raise NotImplementedError
14. MFA Repository

No implementaremos aún todos sus métodos.

Pero el patrón será:

MFAFactorRepository
MFAChallengeRepository
RecoveryCodeRepository
TrustedDeviceRepository

No:

MFARepository

gigante.

Cada Aggregate/ownership boundary debe tener un contrato apropiado.

15. Policy Repository

Authorization Policy también será un módulo independiente.

Futuro:

identity/policy/domain/repositories.py

con contratos conceptuales como:

get_active_policy()
get_policy_version()
list_policies()

Pero el Repository no evaluará policies.

Esto es crucial.

Incorrecto:

policy_repository.authorize(...)

Correcto:

PolicyRepository
      ↓
retrieve policy
      ↓
Authorization Policy Engine
      ↓
evaluate

El Repository recupera estado.

El Policy Engine toma decisiones.

16. Repository ≠ Service

No debemos convertir los repositorios en servicios de negocio.

Repository:

retrieve
persist
delete

Application Service:

orchestrate use case

Domain:

enforce business invariants

Por ejemplo:

SuspendUserUseCase
      │
      ▼
UserRepository.get_by_id()
      │
      ▼
User.suspend()
      │
      ▼
UserRepository.save()
17. Query Contracts

Las consultas complejas deben tener criterios explícitos.

Podemos crear:

identity/shared/application/query.py
from dataclasses import dataclass
from typing import Any


@dataclass(frozen=True)
class QueryCriteria:
    filters: dict[str, Any]
    limit: int | None = None
    cursor: str | None = None

Pero debemos mantener esto genérico.

No permitiremos:

filters={
    "raw_sql": "..."
}

ni:

filters={
    "sql": ...
}
18. Filtering Allowlist

Los Repository implementations deberán trabajar únicamente con filtros permitidos.

Por ejemplo:

email
status
created_at
organization_id

y no:

arbitrary SQL

Esto conecta con las reglas API de filtrado y seguridad.

19. Pagination

Para colecciones grandes, el contrato preferirá:

cursor pagination

sobre offset.

Ejemplo:

@dataclass(frozen=True)
class PageRequest:
    limit: int = 50
    cursor: str | None = None

Y:

@dataclass(frozen=True)
class Page[T]:
    items: list[T]
    next_cursor: str | None

Si el runtime Python usado no permite la sintaxis genérica class Page[T], podemos mantener una versión compatible con Generic[T].

Para evitar fijar una versión Python todavía, recomiendo:

from dataclasses import dataclass
from typing import Generic, TypeVar


T = TypeVar("T")


@dataclass(frozen=True)
class PageRequest:
    limit: int = 50
    cursor: str | None = None


@dataclass(frozen=True)
class Page(Generic[T]):
    items: list[T]
    next_cursor: str | None = None
20. Repository Query Contract

Entonces podemos ampliar:

identity/shared/application/
├── ports.py
├── result.py
└── pagination.py
pagination.py
from dataclasses import dataclass
from typing import Generic, TypeVar


T = TypeVar("T")


@dataclass(frozen=True)
class PageRequest:
    limit: int = 50
    cursor: str | None = None


@dataclass(frozen=True)
class Page(Generic[T]):
    items: list[T]
    next_cursor: str | None = None

Los límites máximos serán definidos posteriormente en la API contract/configuration.

21. Unit of Work

El Repository no debe manejar por sí solo la transacción completa.

La transacción pertenece al:

Unit of Work

Conceptualmente:

Use Case
   │
   ▼
UnitOfWork
   │
   ├── UserRepository
   ├── MembershipRepository
   └── OutboxWriter
   │
   ▼
Commit

Esto será esencial para el Transactional Outbox.

22. Contrato Unit of Work

Mantenemos:

identity/shared/application/ports.py
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

Más adelante una implementación concreta podrá exponer:

uow.users
uow.organizations
uow.memberships

pero no lo haremos todavía porque esos repositorios aún no existen completamente.

23. Repository + Outbox

Este es uno de los puntos más importantes.

Una operación como:

Create User

no debe hacer:

repository.save()
     ↓
commit

outbox.append()
     ↓
commit

porque serían dos transacciones.

Debe ser:

UnitOfWork
│
├── UserRepository.save()
│
├── OutboxWriter.append()
│
└── commit()

Una única transacción.

24. Optimistic Concurrency

Identity tendrá entidades con una versión conceptual:

version

Por ejemplo:

User version 7

si dos procesos intentan modificarlo:

Process A → version 7 → save → version 8
Process B → version 7 → save

B debe detectar:

CONCURRENCY_CONFLICT

en lugar de sobrescribir silenciosamente A.

La implementación SQL concreta vendrá en Persistence.

El Repository Contract debe permitir expresar esa semántica.

25. Concurrency Exception

En:

identity/shared/domain/exceptions.py

añadiremos:

class ConcurrencyConflict(DomainError):
    """
    Raised when an aggregate cannot be persisted because
    its expected version is no longer current.
    """

Esto permite que Infrastructure traduzca:

PostgreSQL optimistic locking failure

a:

ConcurrencyConflict

sin filtrar detalles de la base de datos hacia Application.

26. Repository Implementation Boundary

Más adelante:

identity/account/
│
├── domain/
│   └── repositories.py
│
└── infrastructure/
    └── persistence/
        └── postgres_user_repository.py

Entonces:

Domain
  UserRepository
       ▲
       │ implements
       │
Infrastructure
  PostgreSQLUserRepository
27. Lo que Infrastructure podrá conocer

Sí:

PostgreSQL
SQLAlchemy / asyncpg
connection pool
transactions
indexes
constraints
serialization

No:

FastAPI request
HTTP response
API status codes

Infrastructure no es API.

28. Lo que Domain NO podrá conocer

No:

from sqlalchemy import ...

No:

from fastapi import ...

No:

import redis

No:

import httpx

No:

from psycopg import ...

La frontera es estricta.

29. Tests de Repository Contracts

En esta tarea no probaremos PostgreSQL.

Probaremos que los contratos pueden ser implementados.

Ejemplo:

from uuid import uuid4

from app.domains.identity.shared.domain.repositories import Repository


class FakeRepository(Repository):

    def __init__(self):
        self.items = {}

    async def get_by_id(self, entity_id):
        return self.items.get(entity_id)

    async def save(self, entity):
        self.items[entity.id] = entity

    async def delete(self, entity_id):
        self.items.pop(entity_id, None)

Test:

async def test_repository_contract():

    repository = FakeRepository()

    entity = type(
        "Entity",
        (),
        {"id": uuid4()},
    )()

    await repository.save(entity)

    result = await repository.get_by_id(
        entity.id
    )

    assert result is entity
30. Test de aislamiento

Podemos revisar imports.

from pathlib import Path


DOMAIN_ROOT = (
    Path(__file__).resolve().parents[3]
    / "app"
    / "domains"
    / "identity"
)


FORBIDDEN_DOMAIN_IMPORTS = [
    "fastapi",
    "sqlalchemy",
    "asyncpg",
    "psycopg",
    "redis",
    "httpx",
]


def test_identity_domain_has_no_infrastructure_dependencies():

    for file in DOMAIN_ROOT.rglob("domain/*.py"):

        content = file.read_text(
            encoding="utf-8"
        ).lower()

        for forbidden in FORBIDDEN_DOMAIN_IMPORTS:

            assert f"import {forbidden}" not in content
            assert f"from {forbidden}" not in content

Esto es una primera barrera.

Posteriormente utilizaremos un architecture linter más robusto.

31. Criterios de aceptación
AC-01 — Base Repository

Existe un Repository contract genérico.

AC-02 — Module Ownership

Los Repository Contracts específicos pertenecen a sus módulos.

AC-03 — No Cross-Module Persistence

Un módulo no accede directamente al Repository interno de otro módulo.

AC-04 — Infrastructure Isolation

Las implementaciones concretas pertenecen a Infrastructure.

AC-05 — Domain Independence

Domain no conoce PostgreSQL, Redis, FastAPI u otros adapters.

AC-06 — Unit of Work

Existe un contrato de transacción.

AC-07 — Outbox Compatibility

El diseño permite ejecutar:

business state + outbox

dentro de una misma transacción.

AC-08 — Concurrency

Existe una abstracción para conflictos de concurrencia.

AC-09 — Pagination

Existe un contrato de paginación basado en cursor.

AC-10 — Tenant Isolation

Los contratos tenant-scoped requieren contexto de organización/tenant cuando corresponda.

AC-11 — Testability

Los Repository Contracts pueden probarse mediante implementaciones fake.

AC-12 — No SQL Leakage

No existen consultas SQL dentro del Domain/Application.

32. Definition of Done
[ ] Repository base
[ ] EntityId integration
[ ] Account/User Repository pattern
[ ] Organization Repository pattern
[ ] Membership Repository pattern
[ ] Session Repository pattern
[ ] RefreshToken Repository pattern
[ ] QueryCriteria
[ ] PageRequest
[ ] Page
[ ] UnitOfWork contract
[ ] ConcurrencyConflict
[ ] Repository tests
[ ] Architecture boundary tests
[ ] No PostgreSQL implementation
[ ] No ORM models
[ ] No SQL in Domain
[ ] No cross-module persistence access
33. Trazabilidad
IS-001
 │
 └── TASK-001-06
       │
       ├── ESP-0001/01
       │
       ├── ESP-0001/11
       │   Identity Persistence & Data Model
       │
       ├── ESP-0001/14
       │   Identity Implementation Requirements
       │
       ├── ADR-006
       │   PostgreSQL as transactional source
       │
       ├── ADR-012
       │   PostgreSQL schema
       │
       ├── ADR-016
       │   Concurrency
       │
       └── ADR-017
           Database migrations
34. Estado de Identity después de TASK-001-06

Ya tenemos:

TASK-001-01
API Runtime
      ↓
TASK-001-02
Domain Package
      ↓
TASK-001-03
Layer Architecture
      ↓
TASK-001-04
Shared Kernel
      ↓
TASK-001-05
Domain Events
      ↓
TASK-001-06
Repository Contracts

Y la arquitectura empieza a quedar así:

                         EVOXA API
                            │
                            ▼
                     Identity API
                            │
                            ▼
                      Application
                            │
                ┌───────────┴───────────┐
                ▼                       ▼
             Domain              Application Ports
                │                       │
                │               ┌───────┴────────┐
                │               │                │
                ▼               ▼                ▼
          Repository        UnitOfWork       OutboxWriter
             Contract           │                │
                │               │                │
                └───────────────┼────────────────┘
                                ▼
                         Infrastructure
                                │
                       ┌────────┴────────┐
                       ▼                 ▼
                  PostgreSQL       Event Platform

La siguiente pieza es especialmente importante porque aquí vamos a cerrar la unidad transaccional real.

Siguiente: TASK-001-07 — Unit of Work / Transaction Boundary

Ahí definiremos cómo una operación de Identity agrupa:

Aggregate changes
      +
Repository persistence
      +
Domain Events
      +
Outbox records

en una única frontera transaccional, incluyendo commit, rollback y manejo de errores de concurrencia.
