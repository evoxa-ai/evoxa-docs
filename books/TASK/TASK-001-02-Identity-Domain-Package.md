TASK-001-02 — Identity Domain Package

IS: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Dependencias: TASK-001-01 — API Runtime Foundation
Prioridad: Critical
Estado: Ready for Implementation

Esta tarea crea el paquete raíz del Identity Domain y establece sus límites físicos dentro del código.

La idea es que desde ahora EVOXA tenga una frontera clara:

apps/api/app/domains/identity/

y que ninguna funcionalidad externa pueda acceder directamente a las estructuras internas de Identity.

1. Objetivo

Crear el bounded package inicial de Identity con:

paquete raíz identity;
módulos internos definidos por arquitectura;
separación inicial Domain / Application / Infrastructure / API;
shared mínimo;
contratos de módulo;
reglas de dependencia;
tests arquitectónicos básicos.

Todavía no implementaremos entidades de negocio.

Por ejemplo, no crearemos todavía:

User
Organization
Membership
Role
Permission
Session
RefreshToken
MFAFactor

Eso llegará en las tareas correspondientes.

2. Ubicación

Partimos de:

apps/api/app/

y agregamos:

domains/
└── identity/

La estructura será:

apps/api/app/
│
├── api/
│   └── v1/
│
├── core/
│
└── domains/
    └── identity/
3. Estructura completa inicial

Según la arquitectura que venimos consolidando:

identity/
│
├── __init__.py
│
├── account/
│   ├── __init__.py
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── api/
│
├── organization/
│   ├── __init__.py
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── api/
│
├── membership/
│   ├── __init__.py
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── api/
│
├── authentication/
│   ├── __init__.py
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── api/
│
├── authorization/
│   ├── __init__.py
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── api/
│
├── role_permission/
│   ├── __init__.py
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── api/
│
├── policy/
│   ├── __init__.py
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── api/
│
├── session/
│   ├── __init__.py
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── api/
│
├── token/
│   ├── __init__.py
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── api/
│
├── mfa/
│   ├── __init__.py
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── api/
│
├── recovery/
│   ├── __init__.py
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── api/
│
├── federation/
│   ├── __init__.py
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── api/
│
├── provisioning/
│   ├── __init__.py
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── api/
│
├── audit/
│   ├── __init__.py
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── api/
│
├── security_events/
│   ├── __init__.py
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── api/
│
├── integration/
│   ├── __init__.py
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── api/
│
└── shared/
    ├── __init__.py
    ├── domain/
    ├── application/
    └── infrastructure/

Esto representa la frontera arquitectónica, no significa que cada carpeta tenga que llenarse ahora.

4. ¿Por qué crear todos los módulos ahora?

Porque estamos separando dos conceptos:

Estructura arquitectónica
Identity
 ├── Account
 ├── Organization
 ├── Membership
 ├── Authentication
 ├── Authorization
 ├── Role & Permission
 ├── Policy
 ├── Session
 ├── Token
 ├── MFA
 ├── Recovery
 ├── Federation
 ├── Provisioning
 ├── Audit
 ├── Security Events
 └── Integration
Implementación funcional

Se irá construyendo progresivamente.

Por tanto:

crear módulo ≠ implementar módulo

Esto nos permite mantener el mapa arquitectónico estable mientras desarrollamos.

5. Regla principal de dependencia

Dentro de cada módulo:

API
 ↓
Application
 ↓
Domain

e Infrastructure implementa contratos definidos por las capas superiores.

Conceptualmente:

              ┌──────────────┐
              │     API      │
              └──────┬───────┘
                     │
                     ▼
              ┌──────────────┐
              │ Application  │
              └──────┬───────┘
                     │
                     ▼
              ┌──────────────┐
              │    Domain    │
              └──────────────┘
                     ▲
                     │
              ┌──────┴───────┐
              │Infrastructure │
              └──────────────┘

La infraestructura no debe contaminar el Domain.

6. identity/__init__.py

Crearemos:

"""
EVOXA Identity Domain.

Bounded domain responsible for identity, authentication,
authorization, sessions, tokens, MFA, recovery,
federation, provisioning and identity security.
"""

No debemos colocar lógica de negocio aquí.

7. Crear los módulos

Por ejemplo:

identity/account/
identity/organization/
identity/membership/
identity/authentication/
...

Cada módulo tendrá inicialmente solamente:

__init__.py
domain/
application/
infrastructure/
api/
8. Ejemplo: Account
identity/account/__init__.py
"""
Identity Account module.
"""
identity/account/domain/__init__.py
"""
Account domain layer.

Contains domain entities, value objects, domain services,
domain events and repository contracts owned by Account.
"""
identity/account/application/__init__.py
"""
Account application layer.

Contains application use cases and orchestration.
"""
identity/account/infrastructure/__init__.py
"""
Account infrastructure layer.

Contains implementations of infrastructure contracts.
"""
identity/account/api/__init__.py
"""
Account API layer.
"""

Los demás módulos seguirán exactamente el mismo patrón.

9. Shared Kernel

No debemos convertir shared en un cajón de sastre.

Su función será contener solamente conceptos realmente compartidos.

Inicialmente:

shared/
├── domain/
├── application/
└── infrastructure/

Por ejemplo, eventualmente podrían vivir aquí conceptos como:

EntityId
DomainEvent
DomainError
ValueObject
AggregateRoot

pero no los implementaremos todavía si no son necesarios para esta tarea.

La regla será:

Si un concepto pertenece claramente a un módulo, permanece en ese módulo.

10. Contratos entre módulos

Los módulos no podrán hacer esto:

# ❌ PROHIBIDO
from identity.membership.infrastructure.models import MembershipModel

ni:

# ❌ PROHIBIDO
from identity.role_permission.infrastructure.repositories import RoleRepository

para manipular directamente infraestructura de otro módulo.

La interacción deberá producirse mediante contratos.

Conceptualmente:

Account
   │
   │ Contract
   ▼
Membership

o mediante eventos:

Account
   │
   │ Domain / Integration Event
   ▼
Security Events
11. Regla de acceso a persistencia

Un módulo no puede acceder directamente a las tablas de otro.

Incorrecto:

Membership
   ↓
SQL
   ↓
Role tables

Correcto:

Membership Application
   ↓
Role & Permission Contract
   ↓
Role & Permission

La persistencia será implementada posteriormente mediante los Repository Contracts definidos en TASK-001-06.

12. Authorization como frontera especial

Existe una regla especialmente importante.

Los demás dominios de EVOXA no deben implementar su propia lógica de autorización.

La autorización debe pasar por:

Identity
   │
   ▼
Authorization
   │
   ▼
Authorization Runtime

Por eso no crearemos:

orders/
   └── authorization.py   ❌

ni:

training/
   └── permission_engine.py   ❌

Los dominios consumirán el contrato central de Identity.

13. API del módulo

Cada módulo podrá tener su propia API interna:

identity/account/api/
identity/membership/api/
identity/authentication/api/

Pero las rutas públicas serán ensambladas posteriormente desde:

app/api/v1/

Por ejemplo:

/api/v1/auth/*
/api/v1/users/*
/api/v1/organizations/*
/api/v1/memberships/*

Esto evita que el router público quede acoplado directamente a detalles internos.

14. Domain vacío por ahora

Por ejemplo:

identity/account/domain/

no tendrá todavía:

user.py
user_service.py
user_repository.py

porque eso corresponde a las siguientes tareas.

Esta tarea únicamente establece el límite físico y arquitectónico.

15. Tests arquitectónicos

Necesitamos comprobar que la estructura no se rompa accidentalmente.

Inicialmente podemos crear:

tests/
└── domains/
    └── identity/
        └── test_identity_structure.py

Ejemplo:

from pathlib import Path


IDENTITY_ROOT = (
    Path(__file__).resolve().parents[3]
    / "app"
    / "domains"
    / "identity"
)


EXPECTED_MODULES = [
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
    "shared",
]


def test_identity_root_exists():
    assert IDENTITY_ROOT.exists()
    assert IDENTITY_ROOT.is_dir()


def test_identity_modules_exist():
    for module in EXPECTED_MODULES:
        module_path = IDENTITY_ROOT / module

        assert module_path.exists(), (
            f"Identity module missing: {module}"
        )


def test_identity_module_layers_exist():
    for module in EXPECTED_MODULES:
        module_path = IDENTITY_ROOT / module

        for layer in [
            "domain",
            "application",
            "infrastructure",
            "api",
        ]:
            layer_path = module_path / layer

            assert layer_path.exists(), (
                f"Missing layer {layer} "
                f"in module {module}"
            )
16. Architecture test más importante

Posteriormente agregaremos análisis automático de imports para impedir:

Domain → FastAPI             ❌
Domain → SQLAlchemy          ❌
Domain → Redis               ❌
Domain → HTTP client         ❌
Domain → Infrastructure      ❌

y permitir:

API → Application
Application → Domain
Infrastructure → Domain contracts

Esto será parte importante de los Architecture Boundary Tests.

17. Resultado esperado

Al terminar TASK-001-02 tendremos:

apps/api/
│
├── app/
│   ├── api/
│   ├── core/
│   │
│   └── domains/
│       └── identity/
│           │
│           ├── account/
│           ├── organization/
│           ├── membership/
│           ├── authentication/
│           ├── authorization/
│           ├── role_permission/
│           ├── policy/
│           ├── session/
│           ├── token/
│           ├── mfa/
│           ├── recovery/
│           ├── federation/
│           ├── provisioning/
│           ├── audit/
│           ├── security_events/
│           ├── integration/
│           └── shared/
│
└── tests/

Y podremos ejecutar:

pytest

y verificar que la estructura arquitectónica existe.

18. Criterios de aceptación
AC-01 — Identity Root

Given la API de EVOXA
When se inspecciona app/domains
Then debe existir identity.

AC-02 — Identity Modules

Given Identity
When se inspeccionan sus módulos
Then deben existir los módulos definidos por ESP.

AC-03 — Layer Separation

Cada módulo debe contener:

domain
application
infrastructure
api
AC-04 — Shared Kernel

Debe existir un shared independiente y mínimo.

AC-05 — No Business Logic

Ningún módulo debe contener todavía lógica funcional que pertenezca a futuras IS.

AC-06 — API Isolation

Las APIs públicas deben permanecer ensambladas desde app/api/v1.

AC-07 — Dependency Boundaries

Los tests arquitectónicos deben poder detectar violaciones de dependencias.

AC-08 — Testability

La estructura debe poder importarse y probarse independientemente.

19. Definition of Done
[ ] identity package creado
[ ] 16 módulos Identity creados
[ ] shared kernel creado
[ ] Domain/Application/Infrastructure/API separados
[ ] __init__.py creados
[ ] arquitectura sin lógica de negocio
[ ] tests estructurales
[ ] pytest ejecuta correctamente
[ ] no acceso directo cross-module
[ ] no acceso directo a infraestructura
[ ] documentación de límites
20. Trazabilidad
IS-001
 │
 └── TASK-001-02
       │
       ├── ESP-0001/01
       ├── ESP-0001/02
       ├── ESP-0001/14
       ├── ADR-067
       └── ADR-069

La decisión fundamental que estamos materializando aquí es:

Identity será un Bounded Domain modular, inicialmente implementado como Modular Monolith, con límites explícitos entre módulos y capas.

Y esto es importante porque no estamos creando simplemente carpetas: estamos estableciendo las fronteras que posteriormente impedirán que el código de EVOXA termine convertido en un monolito acoplado.

Próxima tarea

TASK-001-03 — Layer Architecture

Ahí pasaremos de tener solamente la estructura de módulos a implementar físicamente las capas:

Domain
Application
Infrastructure
API

con sus primeras reglas de dependencia y contratos, preparando el terreno para comenzar a crear los primeros elementos reales del dominio Identity.
