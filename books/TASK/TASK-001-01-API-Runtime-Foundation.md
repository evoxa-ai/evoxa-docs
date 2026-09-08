TASK-001-01 — API Runtime Foundation

IS: IS-001 — Identity Domain Foundation
ESP: ESP-0001/14 — Identity Implementation Requirements
Capa: Presentation / API
Prioridad: Critical
Estado: Ready for Implementation

Esta tarea construye el runtime HTTP base de EVOXA Identity. No implementaremos todavía User CRUD, Login, JWT, MFA ni Authorization.

1. Objetivo

Crear la base ejecutable de la API Identity con:

FastAPI.
Punto de entrada de aplicación.
Configuración centralizada.
Router /api/v1.
Health endpoints.
Readiness endpoint.
Manejo global de errores.
Error envelope estándar de EVOXA.
Correlation ID.
Estructura preparada para DI y middleware.
Tests mínimos del runtime.

La arquitectura debe respetar:

Client
   │
   ▼
Presentation
   │
   ▼
API Runtime
   │
   ├── Middleware
   │
   ├── Routing
   │
   ├── Error Handling
   │
   └── Correlation / Trace Context
   │
   ▼
Application
2. Estructura inicial

Vamos a crear:

apps/api/
├── app/
│   ├── __init__.py
│   │
│   ├── main.py
│   │
│   ├── api/
│   │   ├── __init__.py
│   │   └── v1/
│   │       ├── __init__.py
│   │       ├── router.py
│   │       └── health.py
│   │
│   └── core/
│       ├── __init__.py
│       ├── config.py
│       ├── errors.py
│       └── middleware/
│           ├── __init__.py
│           └── correlation.py
│
└── tests/
    ├── __init__.py
    └── test_runtime.py

Todavía no crearemos:

domains/identity/account/
domains/identity/authentication/
domains/identity/authorization/
domains/identity/mfa/
...

Eso corresponde a las siguientes tareas.

3. Dependencias

La base utiliza:

FastAPI
Pydantic Settings
Uvicorn
Pytest
HTTPX

El requirements.txt inicial:

fastapi
pydantic-settings
uvicorn[standard]
pytest
httpx

No fijamos versiones todavía si el proyecto no tiene aún una política de versionado de dependencias definida.

4. Configuración
apps/api/app/core/config.py
from functools import lru_cache

from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    app_name: str = "EVOXA Identity API"
    app_version: str = "0.1.0"
    environment: str = "development"

    api_prefix: str = "/api/v1"

    debug: bool = False

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=False,
        extra="ignore",
    )


@lru_cache
def get_settings() -> Settings:
    return Settings()

La configuración queda centralizada.

Más adelante esta misma estructura podrá incorporar:

PostgreSQL
Redis
Event Platform
KMS
Secret Manager
JWT
MFA
observability

pero no los agregamos todavía a esta tarea.

5. Error Model

El Blueprint establece el envelope:

{
  "error": {
    "code": "...",
    "message": "...",
    "correlation_id": "..."
  }
}

Por lo tanto creamos una excepción propia.

apps/api/app/core/errors.py
from dataclasses import dataclass
from typing import Any


@dataclass
class AppError(Exception):
    code: str
    message: str
    status_code: int = 400
    details: Any | None = None

Ahora los errores internos de EVOXA podrán transformarse posteriormente en un contrato HTTP uniforme.

6. Correlation ID

Cada request debe poder seguirse a través de:

Request
   │
   ├── API
   ├── Application
   ├── Domain
   ├── Database
   ├── Events
   └── Logs

Para ello utilizaremos:

X-Correlation-ID

Si el cliente proporciona uno válido, lo reutilizamos.

Si no existe, generamos uno.

apps/api/app/core/middleware/correlation.py
from uuid import uuid4

from starlette.middleware.base import BaseHTTPMiddleware
from starlette.requests import Request
from starlette.responses import Response


CORRELATION_HEADER = "X-Correlation-ID"


class CorrelationIdMiddleware(BaseHTTPMiddleware):

    async def dispatch(
        self,
        request: Request,
        call_next,
    ) -> Response:

        correlation_id = request.headers.get(CORRELATION_HEADER)

        if not correlation_id:
            correlation_id = str(uuid4())

        request.state.correlation_id = correlation_id

        response = await call_next(request)

        response.headers[CORRELATION_HEADER] = correlation_id

        return response

Esto nos permite que posteriormente:

API
 ↓
Application
 ↓
Domain
 ↓
Outbox
 ↓
Event Platform

pueda conservar el mismo contexto de correlación.

7. Health API

Los endpoints operacionales son distintos de las funcionalidades de Identity.

Crearemos:

GET /health/live
GET /health/ready
apps/api/app/api/v1/health.py
from fastapi import APIRouter

from app.core.config import get_settings


router = APIRouter(
    prefix="/health",
    tags=["Health"],
)


@router.get("/live")
async def liveness():
    settings = get_settings()

    return {
        "status": "ok",
        "service": settings.app_name,
        "version": settings.app_version,
    }


@router.get("/ready")
async def readiness():
    settings = get_settings()

    return {
        "status": "ready",
        "service": settings.app_name,
        "version": settings.app_version,
    }
Importante

En esta primera tarea /health/ready no comprobará PostgreSQL ni Redis.

Eso será responsabilidad de:

TASK-001-08 — PostgreSQL Infrastructure
TASK-001-09 — Redis Infrastructure

Posteriormente podremos evolucionarlo a:

{
  "status": "ready",
  "checks": {
    "postgresql": "ok",
    "redis": "ok",
    "event_platform": "ok"
  }
}

sin adelantar responsabilidades.

8. API v1 Router
apps/api/app/api/v1/router.py
from fastapi import APIRouter

from app.api.v1.health import router as health_router


router = APIRouter()

router.include_router(health_router)

Esto permitirá posteriormente:

/api/v1/auth
/api/v1/users
/api/v1/organizations
/api/v1/memberships
/api/v1/roles
/api/v1/permissions
/api/v1/sessions
/api/v1/mfa
/api/v1/policies
/api/v1/authorization
/api/v1/audit

sin modificar el runtime principal.

9. Aplicación principal
apps/api/app/main.py
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

from app.api.v1.router import router as api_v1_router
from app.core.config import get_settings
from app.core.errors import AppError
from app.core.middleware.correlation import CorrelationIdMiddleware


settings = get_settings()


app = FastAPI(
    title=settings.app_name,
    version=settings.app_version,
    description="EVOXA Identity API",
)


app.add_middleware(CorrelationIdMiddleware)


@app.exception_handler(AppError)
async def app_error_handler(
    request: Request,
    exc: AppError,
):
    correlation_id = getattr(
        request.state,
        "correlation_id",
        None,
    )

    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": {
                "code": exc.code,
                "message": exc.message,
                "correlation_id": correlation_id,
            }
        },
    )


@app.exception_handler(Exception)
async def unhandled_exception_handler(
    request: Request,
    exc: Exception,
):
    correlation_id = getattr(
        request.state,
        "correlation_id",
        None,
    )

    return JSONResponse(
        status_code=500,
        content={
            "error": {
                "code": "INTERNAL_SERVER_ERROR",
                "message": "An internal server error occurred.",
                "correlation_id": correlation_id,
            }
        },
    )


app.include_router(
    api_v1_router,
    prefix=settings.api_prefix,
)
10. Resultado de las rutas

Con esto tendremos:

GET /api/v1/health/live
GET /api/v1/health/ready

Por ejemplo:

GET /api/v1/health/live

Respuesta:

{
  "status": "ok",
  "service": "EVOXA Identity API",
  "version": "0.1.0"
}

Y:

GET /api/v1/health/ready

Respuesta:

{
  "status": "ready",
  "service": "EVOXA Identity API",
  "version": "0.1.0"
}

Además:

X-Correlation-ID: 7b5c...

estará presente en la respuesta.

11. Tests
apps/api/tests/test_runtime.py
from fastapi.testclient import TestClient

from app.main import app


client = TestClient(app)


def test_liveness():
    response = client.get("/api/v1/health/live")

    assert response.status_code == 200

    body = response.json()

    assert body["status"] == "ok"
    assert "service" in body
    assert "version" in body


def test_readiness():
    response = client.get("/api/v1/health/ready")

    assert response.status_code == 200

    body = response.json()

    assert body["status"] == "ready"


def test_correlation_id_is_generated():
    response = client.get("/api/v1/health/live")

    assert response.status_code == 200
    assert "X-Correlation-ID" in response.headers

    correlation_id = response.headers["X-Correlation-ID"]

    assert len(correlation_id) > 0


def test_correlation_id_is_preserved():
    correlation_id = "test-correlation-123"

    response = client.get(
        "/api/v1/health/live",
        headers={
            "X-Correlation-ID": correlation_id,
        },
    )

    assert response.status_code == 200
    assert response.headers["X-Correlation-ID"] == correlation_id
12. Cómo levantar la API

Desde:

apps/api/

instalamos:

pip install -r requirements.txt

y ejecutamos:

uvicorn app.main:app --reload

La API quedará disponible en:

http://localhost:8000

Swagger:

http://localhost:8000/docs

OpenAPI:

http://localhost:8000/openapi.json
13. Validación de la tarea

La TASK-001-01 estará completa cuando podamos demostrar:

Validación	Resultado esperado
API inicia	✅
FastAPI funciona	✅
/api/v1 existe	✅
Liveness	✅
Readiness	✅
Correlation ID generado	✅
Correlation ID preservado	✅
Error envelope	✅
Configuración centralizada	✅
Tests runtime	✅
PostgreSQL	⏳ siguiente task
Redis	⏳ siguiente task
JWT	⏳ futura task
Authentication	⏳ futura task
Authorization	⏳ futura task
MFA	⏳ futura task
14. Lo que deliberadamente NO hacemos

Esto es importante para no romper el lifecycle de EVOXA.

TASK-001-01 no debe implementar:

❌ User
❌ Organization
❌ Membership
❌ Role
❌ Permission
❌ Password
❌ Login
❌ JWT
❌ Refresh Token
❌ Session
❌ MFA
❌ Authorization
❌ Policy Engine
❌ Redis
❌ PostgreSQL
❌ Event Broker

Es únicamente el runtime HTTP sobre el cual construiremos Identity.

15. Trazabilidad
Blueprint
   │
   ▼
Reference Architecture
   │
   ▼
Engineering Standards
   │
   ▼
ESP-0001
   │
   ├── ESP-0001/02 Architecture
   ├── ESP-0001/10 API Contracts
   ├── ESP-0001/13 Testing
   └── ESP-0001/14 Implementation
          │
          ▼
       IS-001
          │
          ▼
   TASK-001-01
          │
          ▼
   API Runtime Foundation

Y las siguientes tareas continuarán sobre esta base:

TASK-001-01  API Runtime Foundation
       ↓
TASK-001-02  Identity Domain Package
       ↓
TASK-001-03  Layer Architecture
       ↓
TASK-001-04  Identity Shared Kernel
       ↓
TASK-001-05  Domain Events Foundation
       ↓
TASK-001-06  Repository Contracts
       ↓
...

Siguiente paso recomendado: TASK-001-02 — Identity Domain Package, donde empezaremos a crear físicamente app/domains/identity/ y sus límites de módulo, todavía sin implementar las funcionalidades de negocio.
