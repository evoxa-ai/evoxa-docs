1. Billing Architecture

La Billing Architecture define la arquitectura tecnológica, funcional y financiera que permite transformar productos, precios, contratos, suscripciones y consumo en cargos, facturas, pagos, créditos, ingresos y registros financieros confiables.

La arquitectura debe soportar simultáneamente:

múltiples organizaciones;
múltiples tenants;
múltiples clientes;
múltiples productos;
múltiples modelos de pricing;
múltiples monedas;
múltiples jurisdicciones;
múltiples proveedores de pago;
múltiples ciclos de facturación;
billing recurrente;
usage-based billing;
billing híbrido;
contratos Enterprise;
grandes volúmenes de eventos;
procesamiento síncrono y asíncrono;
auditoría financiera completa.

La arquitectura debe diseñarse para que Billing pueda evolucionar sin romper las operaciones financieras existentes.

2. Architectural Vision

La arquitectura conceptual puede representarse:

                         ENTERPRISE PLATFORM
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                 │
          Commerce           Operations        Finance
              │                 │                 │
              └─────────────────┼─────────────────┘
                                │
                         BILLING PLATFORM
                                │
      ┌─────────────────────────┼─────────────────────────┐
      │                         │                         │
   Catalog                   Pricing                  Contracts
      │                         │                         │
      └─────────────────────────┼─────────────────────────┘
                                │
                         Subscription Engine
                                │
                         Usage & Metering
                                │
                           Rating Engine
                                │
                          Charge Engine
                                │
                        Invoice Engine
                                │
                         Payment Engine
                                │
                     Revenue & Accounting
                                │
                        Billing Analytics

Billing funciona como una plataforma intermedia entre Commerce y Finance.

3. Architectural Layers

La plataforma puede dividirse en diferentes capas.

┌──────────────────────────────────────────────┐
│              Experience Layer                │
│ Dashboard • Admin • Customer Portal • APIs   │
├──────────────────────────────────────────────┤
│              Billing Services                │
│ Subscription • Invoice • Payment • Credit    │
├──────────────────────────────────────────────┤
│             Billing Engines                 │
│ Rating • Charging • Tax • Revenue            │
├──────────────────────────────────────────────┤
│            Financial Core                   │
│ Ledger • Transactions • Balances             │
├──────────────────────────────────────────────┤
│              Data Layer                     │
│ PostgreSQL • Events • Data Warehouse         │
├──────────────────────────────────────────────┤
│          Infrastructure Layer               │
│ Queue • Cache • Storage • Observability      │
└──────────────────────────────────────────────┘

Cada capa debe tener responsabilidades claramente delimitadas.

4. Experience Layer

Es la capa mediante la cual usuarios, administradores, clientes y sistemas externos interactúan con Billing.

Incluye:

Billing APIs;
Admin Portal;
Customer Portal;
Partner APIs;
Webhooks;
Reporting APIs;
Financial dashboards.

Ejemplo:

Customer
   │
   ▼
Customer Portal
   │
   ▼
Billing API

Los clientes no deben acceder directamente a los componentes financieros internos.

5. API Layer

La API Layer proporciona una interfaz estable para:

crear subscriptions;
consultar invoices;
registrar pagos;
administrar payment methods;
consultar balances;
registrar usage;
aplicar credits;
realizar refunds;
consultar billing history.

Conceptualmente:

External Client
      │
      ▼
API Gateway
      │
      ▼
Authentication
      │
      ▼
Authorization
      │
      ▼
Billing APIs

Debe soportar:

OAuth/JWT;
API keys;
RBAC;
tenant isolation;
rate limiting;
idempotency keys;
request tracing.
6. Billing Domain Services

Billing debe dividirse en dominios funcionales.

Billing Platform
│
├── Catalog Service
├── Pricing Service
├── Subscription Service
├── Usage Service
├── Rating Service
├── Charge Service
├── Invoice Service
├── Payment Service
├── Credit Service
├── Tax Service
├── Collections Service
├── Revenue Service
├── Ledger Service
└── Billing Analytics

La separación permite que cada dominio evolucione independientemente.

7. Catalog Service

Administra aquello que puede ser vendido.

Product
   │
   ├── SKU
   ├── Feature
   ├── Plan
   ├── Add-on
   └── Package

El Catalog Service define:

productos;
versiones;
features;
bundles;
disponibilidad;
lifecycle.

No debe determinar por sí mismo cuánto se cobra.

8. Pricing Service

El Pricing Service determina las reglas económicas.

Product
   ↓
Pricing Model
   ↓
Price
   ↓
Pricing Rules

Puede administrar:

fixed pricing;
seat pricing;
usage pricing;
tier pricing;
volume pricing;
discounts;
promotions;
contract pricing;
regional pricing.
9. Subscription Service

Administra la relación contractual recurrente.

Customer
    │
    ▼
Subscription
    │
    ├── Plan
    ├── Price
    ├── Billing Cycle
    ├── Start Date
    ├── End Date
    └── Status

Debe manejar:

creation;
activation;
renewal;
upgrade;
downgrade;
pause;
resume;
cancellation;
expiration.
10. Usage & Metering Architecture

Para usage-based billing se requiere una arquitectura específica.

Application
    │
    ▼
Usage Event
    │
    ▼
Event Ingestion
    │
    ▼
Meter
    │
    ▼
Aggregation
    │
    ▼
Rated Usage

Ejemplo:

1,000 API calls
       ↓
API_CALLS meter
       ↓
1,000 billable units
       ↓
Pricing Rule
       ↓
Charge

El sistema debe ser capaz de procesar grandes volúmenes de eventos.

11. Rating Engine

El Rating Engine transforma unidades de consumo en valor monetario.

Usage
  │
  ▼
Metered Quantity
  │
  ▼
Pricing Rule
  │
  ▼
Rating
  │
  ▼
Charge Amount

Ejemplo:

Usage = 50,000 events


Tier:
0–10,000      → $0.010
10,001–50,000 → $0.008


Calculated Charge
= $420

El Rating Engine debe ser:

determinístico;
versionado;
auditable;
reproducible;
idempotente.
12. Charge Engine

El Charge Engine transforma resultados de pricing y rating en cargos financieros.

Rated Usage
     +
Subscription Fee
     +
Add-ons
     -
Discounts
     +
Taxes
     -
Credits
     =
Billable Amount

Debe generar un registro financiero identificable.

Charge
├── charge_id
├── billing_account_id
├── source
├── amount
├── currency
├── pricing_version
├── tax_amount
├── discount_amount
└── status
13. Invoice Engine

El Invoice Engine agrupa cargos en documentos financieros.

Charges
   │
   ▼
Billing Period
   │
   ▼
Invoice Generation
   │
   ▼
Invoice Finalization

Una factura puede contener:

Invoice
│
├── Subscription
├── Usage
├── Add-ons
├── Discounts
├── Taxes
├── Credits
└── Adjustments

La factura debe conservar referencias hacia los cargos originales.

14. Payment Architecture

El Payment Engine administra la interacción con proveedores externos.

Invoice
   │
   ▼
Payment Intent
   │
   ▼
Payment Provider
   │
   ├── Authorized
   ├── Captured
   ├── Failed
   └── Refunded

Puede integrarse con múltiples providers:

                Payment Engine
                       │
          ┌────────────┼────────────┐
          │            │            │
       Provider A   Provider B   Provider C

Esto evita dependencia de un único proveedor.

15. Payment Abstraction Layer

La arquitectura debe utilizar una abstracción:

Payment Service
       │
       ▼
Payment Provider Interface
       │
 ┌─────┼─────┐
 ▼     ▼     ▼
PSP-A PSP-B PSP-C

Así, el dominio Billing no necesita conocer detalles específicos de cada gateway.

16. Credit Architecture

Credits deben ser tratados como una entidad financiera independiente.

Credit Grant
     │
     ▼
Credit Balance
     │
     ▼
Credit Application
     │
     ▼
Invoice / Charge

Debe existir trazabilidad:

Credit Issued
     ↓
Credit Available
     ↓
Credit Applied
     ↓
Credit Remaining

Nunca se debe simplemente modificar un saldo sin registrar la transacción que produjo el cambio.

17. Tax Architecture

El Tax Engine debe permanecer desacoplado.

Charge
   │
   ▼
Tax Determination
   │
   ├── Jurisdiction
   ├── Tax Rules
   ├── Customer Type
   └── Product Tax Category
   │
   ▼
Tax Amount

Esto permite integrar:

tax providers;
reglas locales;
tax IDs;
exenciones;
jurisdicciones múltiples.
18. Billing Ledger

El Ledger representa la capa financiera fundamental.

                 BILLING TRANSACTION
                         │
                         ▼
                      LEDGER
                         │
              ┌──────────┴──────────┐
              │                     │
             Debit                Credit

Cada operación financiera debe poder rastrearse hasta su origen.

Ejemplo:

Usage Event
    ↓
Charge
    ↓
Invoice
    ↓
Payment
    ↓
Ledger

El ledger debe ser preferentemente append-oriented / immutable para operaciones financieras históricas.

19. Balance Engine

El Balance Engine calcula la posición financiera.

Opening Balance
      +
Charges
      +
Taxes
      -
Payments
      -
Credits
      -
Refunds
      +
Adjustments
      =
Current Balance

Puede mantener:

available balance;
outstanding balance;
credit balance;
prepaid balance;
overdue balance.

20. Event-Driven Billing

Billing debe utilizar arquitectura orientada a eventos para operaciones que no necesitan respuesta inmediata.

Subscription Service
        │
        ▼
SubscriptionActivated
        │
        ▼
Event Bus
        │
 ┌──────┼────────┐
 ▼      ▼        ▼
Billing Invoice Analytics

Eventos típicos:

SubscriptionCreated
SubscriptionRenewed
UsageRecorded
ChargeCreated
InvoiceCreated
InvoiceFinalized
PaymentSucceeded
PaymentFailed
RefundCreated
CreditApplied

Esto reduce acoplamiento entre servicios.

21. Synchronous vs Asynchronous Processing

No todo debe procesarse de la misma forma.

Synchronous

Para operaciones que necesitan respuesta inmediata:

Create Payment Intent
Get Invoice
Get Balance
Create Subscription
Asynchronous

Para procesos de mayor volumen:

Usage ingestion
Invoice generation
Payment reconciliation
Revenue processing
Analytics
Notifications
Collections

Modelo:

Request
  │
  ▼
API
  │
  ▼
Queue
  │
  ▼
Worker
  │
  ▼
Billing Engine
22. Billing Processing Pipeline

El pipeline principal puede representarse:

             COMMERCIAL EVENT
                    │
                    ▼
             EVENT INGESTION
                    │
                    ▼
                 METERING
                    │
                    ▼
                  RATING
                    │
                    ▼
                  CHARGE
                    │
                    ▼
                 INVOICE
                    │
                    ▼
                 PAYMENT
                    │
                    ▼
               SETTLEMENT
                    │
                    ▼
              REVENUE ENGINE
                    │
                    ▼
                 LEDGER
                    │
                    ▼
               ACCOUNTING
23. Data Architecture

Billing requiere una arquitectura de datos especializada.

Operational Database
        │
        ├── Customers
        ├── Subscriptions
        ├── Charges
        ├── Invoices
        └── Payments
                │
                ▼
           Event Stream
                │
        ┌───────┴────────┐
        ▼                ▼
 Data Warehouse      Analytics

La base transaccional debe estar optimizada para operaciones financieras, mientras que analytics debe utilizar estructuras apropiadas para consultas analíticas.

24. Core Billing Data Model

Modelo conceptual:

Organization
     │
     ▼
Tenant
     │
     ▼
Billing Account
     │
 ┌───┼────────────┐
 ▼   ▼            ▼
Customer Subscription Contract
          │
          ▼
         Usage
          │
          ▼
        Charge
          │
          ▼
       Invoice
          │
          ▼
       Payment

Y transversalmente:

Charge
  ├── Tax
  ├── Discount
  ├── Credit
  └── Adjustment
25. Billing Database Principles

La base de datos debe garantizar:

Atomicity

Una operación financiera debe completarse completamente o no realizarse.

Consistency

Las relaciones financieras deben mantenerse válidas.

Isolation

Los tenants deben permanecer aislados.

Durability

Una transacción confirmada no debe perderse.

Idempotency

Una solicitud repetida no debe producir duplicaciones.

26. Idempotency Architecture

Es uno de los componentes más importantes.

Request
   │
   ├── Idempotency-Key
   │
   ▼
Idempotency Store
   │
   ├── Existing → Return Original Result
   │
   └── New
         │
         ▼
      Process
         │
         ▼
      Persist

Ejemplo:

POST /payments


Idempotency-Key:
PAY-2026-000123

Si la misma solicitud llega tres veces:

Request 1 → Payment Created
Request 2 → Original Payment
Request 3 → Original Payment

No deben crearse tres pagos.

27. Versioned Pricing Architecture

Los precios deben ser versionados.

Product
   │
   ├── Price v1
   │
   ├── Price v2
   │
   └── Price v3

Una modificación futura no debe alterar retrospectivamente una factura histórica.

Ejemplo:

2026-01
Price v1 = $100


2026-07
Price v2 = $120

La factura de enero debe continuar asociada a Price v1.

28. Temporal Architecture

Billing es un sistema altamente dependiente del tiempo.

Debe manejar:

effective date;
billing period;
invoice date;
due date;
payment date;
settlement date;
recognition date;
cancellation date.

Ejemplo:

Subscription
Start: 01/01
Billing Period: 01/01 → 31/01


Invoice Date: 01/01
Due Date: 15/01
Payment Date: 10/01
Revenue Recognition: 01/01 → 31/01

Por ello, los timestamps deben almacenarse de forma consistente y con timezone explícito cuando corresponda.

29. Proration Engine

Cuando una subscription cambia durante un período:

Plan A
$100/month
     │
     │ Day 15
     ▼
Plan B
$200/month

Billing debe calcular:

Unused Plan A
      +
Used Plan B
      =
Proration Adjustment

El cálculo debe quedar registrado como una operación financiera independiente.

30. Billing Workflow Orchestration

Los procesos complejos pueden utilizar un Billing Workflow Engine.

Subscription Renewal
        │
        ▼
Validate Subscription
        │
        ▼
Calculate Charges
        │
        ▼
Apply Credits
        │
        ▼
Calculate Taxes
        │
        ▼
Generate Invoice
        │
        ▼
Attempt Payment
        │
   ┌────┴────┐
   ▼         ▼
Success    Failure
   │         │
   ▼         ▼
Complete   Retry
             │
             ▼
          Dunning

Esto permite administrar workflows largos sin concentrar toda la lógica en un único servicio.

31. Resilience Architecture

Billing debe diseñarse para fallos parciales.

Ejemplo:

Invoice
   │
   ▼
Payment Provider
   │
   X
Timeout

El sistema no debe asumir automáticamente que el pago falló.

Debe distinguir:

SUCCESS
FAILED
TIMEOUT
UNKNOWN
PENDING

Un estado UNKNOWN puede requerir conciliación posterior.

32. Reconciliation Architecture

La conciliación compara los registros internos con fuentes externas.

Billing Ledger
      │
      ├──────────────┐
      │              │
      ▼              ▼
Internal Payment   PSP Report
      │              │
      └──────┬───────┘
             ▼
       Reconciliation
             │
      ┌──────┴──────┐
      ▼             ▼
   Matched       Exception

Las excepciones deben generar casos para investigación.

33. Security Architecture

Billing maneja información altamente sensible.

Debe incluir:

encryption at rest;
encryption in transit;
RBAC;
tenant isolation;
secrets management;
API authentication;
audit logging;
payment tokenization;
least privilege;
privileged access controls.

La plataforma debe evitar almacenar información sensible de pago cuando pueda utilizar tokens proporcionados por el payment provider.

34. Audit Architecture

Cada modificación crítica debe producir un evento de auditoría.

User
 │
 ▼
Billing Action
 │
 ▼
Authorization
 │
 ▼
Financial Change
 │
 ▼
Audit Event

Ejemplos:

PriceChanged
DiscountApplied
InvoiceAdjusted
CreditGranted
RefundIssued
PaymentCaptured
SubscriptionCancelled

Debe conservarse:

actor;
timestamp;
resource;
previous state;
new state;
reason;
correlation ID.
35. Observability

Billing debe tener observabilidad financiera y técnica.

Technical Metrics
API latency;
error rate;
queue depth;
worker failures;
processing time.
Financial Metrics
invoices generated;
charges processed;
payments succeeded;
payments failed;
revenue processed;
reconciliation exceptions.

Ejemplo:

Technical Health
       +
Financial Health
       =
Billing Health
36. Scalability

La arquitectura debe poder escalar horizontalmente.

                    Billing API
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Instance A    Instance B    Instance C
          │             │             │
          └─────────────┼─────────────┘
                        ▼
                    Event Bus
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Worker A      Worker B      Worker C

Los componentes stateless deben poder escalar independientemente.

37. Multi-Region Architecture

Para una plataforma Enterprise global:

                 Global Billing Layer
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Americas        Europe          APAC
          │              │              │
       Billing        Billing         Billing
       Region         Region          Region

Debe contemplarse:

residencia de datos;
regulación;
moneda;
impuestos;
payment providers;
disponibilidad regional.
38. AI Billing Architecture

La arquitectura puede incorporar una capa de inteligencia:

Billing Data
     │
     ▼
Data Platform
     │
     ▼
AI / ML Layer
     │
 ┌───┼─────────────┐
 ▼   ▼             ▼
Forecasting   Anomaly Detection   Optimization

Aplicaciones:

revenue forecasting;
payment failure prediction;
churn prediction;
pricing optimization;
anomaly detection;
invoice anomaly detection;
customer segmentation;
cost-to-serve analysis.
39. FinOps Integration

Billing debe conectarse con FinOps.

Customer Revenue
       │
       ▼
Billing
       │
       ├───────────────┐
       ▼               ▼
Revenue             Cloud Cost
       │               │
       └───────┬───────┘
               ▼
          Unit Economics
               │
               ▼
          Profitability

Esto permite calcular:

Revenue per Tenant
Cost per Tenant
Margin per Tenant
Revenue per Feature
Cost per Feature
AI Cost per Customer
40. Reference Architecture

La arquitectura completa puede resumirse:

                         ┌─────────────────────┐
                         │   Customer / Admin  │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │     API Gateway     │
                         └──────────┬──────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 │                  │                  │
                 ▼                  ▼                  ▼
             Catalog             Pricing          Subscription
                 │                  │                  │
                 └──────────────────┼──────────────────┘
                                    │
                                    ▼
                              Usage / Metering
                                    │
                                    ▼
                               Rating Engine
                                    │
                                    ▼
                               Charge Engine
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
                 Credits           Tax           Discounts
                    │               │               │
                    └───────────────┼───────────────┘
                                    ▼
                              Invoice Engine
                                    │
                                    ▼
                              Payment Engine
                                    │
                         ┌──────────┴──────────┐
                         ▼                     ▼
                   Payment Providers      Collections
                         │
                         ▼
                    Reconciliation
                         │
                         ▼
                    Revenue Engine
                         │
                         ▼
                       Ledger
                         │
                ┌────────┴────────┐
                ▼                 ▼
            Accounting         Analytics
                                  │
                                  ▼
                             AI / FinOps
41. Arquitectura de referencia tecnológica

Para una implementación Enterprise moderna, una arquitectura posible sería:

API
│
├── REST APIs
├── GraphQL (optional)
└── Webhooks


Application
│
├── Billing Services
├── Workflow Engine
└── Rules Engine


Messaging
│
├── Event Bus
├── Message Queue
└── Dead Letter Queue


Data
│
├── PostgreSQL
├── Redis
├── Object Storage
└── Data Warehouse


Payments
│
├── PSP A
├── PSP B
└── Bank Integration


Analytics
│
├── BI
├── Data Lake
└── AI/ML


Infrastructure
│
├── Kubernetes / Containers
├── Observability
├── Secrets
└── Security

La implementación concreta puede variar, pero los principios arquitectónicos deben permanecer estables.

42. Core Architectural Principles

La arquitectura de Billing debe seguir estos principios:

1. Domain Separation

Cada dominio financiero debe tener responsabilidades claras.

2. Event-Driven

Los cambios importantes deben poder propagarse mediante eventos.

3. API-First

Toda capacidad relevante debe exponerse mediante APIs bien definidas.

4. Immutable Financial History

Los eventos financieros históricos no deben sobrescribirse arbitrariamente.

5. Idempotent Processing

Los procesos repetidos deben ser seguros.

6. Versioned Rules

Pricing, tax y billing rules deben ser versionables.

7. Multi-Tenant by Design

El aislamiento debe estar incorporado desde el diseño.

8. Provider Independence

Los proveedores externos no deben dominar el dominio interno.

9. Financial Determinism

Los cálculos deben poder reproducirse.

10. Auditability

Toda operación crítica debe ser explicable y rastreable.

43. Billing Architecture Evolution

La arquitectura debe permitir evolucionar progresivamente:

Stage 1
Monolithic Billing
       │
       ▼
Stage 2
Modular Billing
       │
       ▼
Stage 3
Service-Oriented Billing
       │
       ▼
Stage 4
Event-Driven Billing
       │
       ▼
Stage 5
Intelligent Billing
       │
       ▼
Stage 6
Autonomous Financial Platform

La evolución no significa necesariamente reemplazar todo el sistema. Debe poder realizarse incrementalmente.

44. Resultado de 26/03

Billing Architecture establece la estructura técnica sobre la cual se construirá todo el ecosistema financiero.

El principio central es:

Billing debe ser una arquitectura modular, multi-tenant, event-driven, idempotente, auditable y extensible, capaz de transformar eventos comerciales y de consumo en operaciones financieras confiables y posteriormente integrarlas con Payments, Revenue, Accounting, Analytics, FinOps e IA.

La arquitectura queda organizada alrededor de:

Catalog
   ↓
Pricing
   ↓
Subscription
   ↓
Usage
   ↓
Metering
   ↓
Rating
   ↓
Charges
   ↓
Invoice
   ↓
Payment
   ↓
Settlement
   ↓
Revenue
   ↓
Ledger
   ↓
Accounting
   ↓
Analytics / AI / FinOps

Con esto queda establecida la base arquitectónica para continuar con:

26/04 — Billing Organizations
