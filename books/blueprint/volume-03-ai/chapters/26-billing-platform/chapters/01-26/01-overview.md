26/01 — Billing Overview
1. Billing Overview

Billing es la capacidad empresarial encargada de administrar el ciclo completo de monetización de una plataforma: desde la definición de productos, precios y planes, hasta la generación de cargos, facturas, cobros, créditos, impuestos, conciliación, ingresos y análisis financiero.

Dentro de una plataforma Enterprise como EVOXA, Billing no debe entenderse simplemente como un módulo de facturación. Debe funcionar como una plataforma financiera transaccional, capaz de soportar múltiples modelos comerciales, organizaciones, tenants, monedas, regiones, impuestos, métodos de pago y modelos de consumo.

La función central de Billing es transformar el uso y las obligaciones comerciales de una plataforma en eventos financieros trazables y ejecutables.

2. Billing como plataforma

Billing se posiciona entre las capacidades comerciales y financieras de la plataforma.

                         ENTERPRISE PLATFORM
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
     Commerce               Operations               Finance
        │                       │                       │
        └───────────────────────┼───────────────────────┘
                                │
                         BILLING PLATFORM
                                │
       ┌────────────────────────┼────────────────────────┐
       │                        │                        │
   Products                 Pricing                  Usage
   & Plans                 & Plans                 & Metering
       │                        │                        │
       └────────────────────────┼────────────────────────┘
                                │
                           Rating & Charges
                                │
                    ┌───────────┼───────────┐
                    │           │           │
                 Invoice     Payment     Credits
                    │           │           │
                    └───────────┼───────────┘
                                │
                    Revenue & Accounting
                                │
                     Financial Intelligence

Billing recibe información desde diferentes dominios y la convierte en información financiera.

3. Objetivo de Billing

El objetivo principal es proporcionar una infraestructura capaz de responder de forma consistente preguntas como:

¿Qué producto compró el cliente?
¿Qué plan tiene contratado?
¿Cuál es su precio?
¿Qué período está siendo facturado?
¿Cuánto consumió?
¿Qué cargos corresponden?
¿Qué descuentos aplican?
¿Qué impuestos deben cobrarse?
¿Cuánto debe pagar?
¿Qué facturas están pendientes?
¿Qué pagos fueron realizados?
¿Qué créditos posee?
¿Qué ingresos generó?
¿Qué ingresos deben reconocerse?
¿Qué dinero debe cobrarse?
¿Qué transacciones fueron rechazadas?
¿Qué ajustes fueron realizados?
¿Cuál es el MRR, ARR, ARPU, churn o revenue?
¿Cuál es el costo de servir al cliente?
¿Cuál es la rentabilidad del tenant?

Billing debe permitir responder estas preguntas de manera determinística, auditable y trazable.

4. Alcance funcional

Billing cubre principalmente:

Product & Catalog

Administración de:

productos
servicios
funcionalidades monetizables
SKUs
bundles
add-ons
planes
Pricing

Administración de:

precios
tarifas
pricing tiers
pricing rules
descuentos
promociones
precios regionales
monedas
Subscription

Administración de:

suscripciones
ciclos de facturación
upgrades
downgrades
renewals
cancellations
trials
pauses
reactivations
Usage

Administración de:

consumo
eventos de uso
métricas
unidades
medidores
períodos
límites
overages
Rating

Transformación del consumo en:

Usage
   ↓
Metering
   ↓
Rating
   ↓
Charge
Invoicing

Generación y administración de:

invoices
invoice lines
credit notes
debit notes
estados de factura
períodos de facturación
Payments

Gestión de:

payment intents
payment transactions
payment methods
payment authorization
payment capture
payment failures
payment reconciliation
Collections

Gestión de:

cuentas vencidas
retries
dunning
collection workflows
suspensión por falta de pago
recuperación de deuda
Revenue

Administración de:

revenue recognition
deferred revenue
earned revenue
recurring revenue
revenue allocation
Financial Integration

Integración con:

accounting
ERP
tax systems
payment gateways
banking
financial reporting
5. Billing Lifecycle

El ciclo principal puede representarse como:

Customer
   │
   ▼
Product
   │
   ▼
Plan
   │
   ▼
Subscription
   │
   ▼
Usage
   │
   ▼
Metering
   │
   ▼
Rating
   │
   ▼
Charges
   │
   ▼
Invoice
   │
   ▼
Payment
   │
   ▼
Settlement
   │
   ▼
Revenue
   │
   ▼
Accounting

Pero el ciclo no termina con el pago.

Posteriormente Billing debe continuar gestionando:

Payment
   │
   ├── Successful
   │
   ├── Failed
   │      │
   │      ▼
   │   Retry
   │      │
   │      ▼
   │   Dunning
   │      │
   │      ▼
   │   Collections
   │
   └── Disputed
          │
          ▼
       Dispute
6. Billing dentro del modelo Enterprise

Billing debe operar dentro de la jerarquía empresarial de la plataforma.

Enterprise
    │
    ├── Organization
    │       │
    │       ├── Business Unit
    │       │
    │       └── Tenant
    │               │
    │               ├── Customer
    │               │
    │               ├── Subscription
    │               │
    │               └── Billing Account
    │
    └── Finance
            │
            ├── Revenue
            ├── Accounting
            └── Reporting

Esto permite que Billing soporte escenarios como:

una empresa con múltiples tenants;
un tenant con múltiples clientes;
un cliente con múltiples subscriptions;
múltiples billing accounts;
diferentes monedas;
diferentes entidades legales;
diferentes métodos de pago;
diferentes reglas tributarias.
7. Billing Account

Uno de los conceptos centrales es el Billing Account.

El Billing Account representa la entidad financiera contra la cual se acumulan:

cargos;
facturas;
pagos;
créditos;
impuestos;
ajustes;
saldos;
deuda.

Puede existir una separación entre:

Customer
    │
    ├── Identity
    ├── Profile
    └── Billing Account
            │
            ├── Subscriptions
            ├── Invoices
            ├── Payments
            ├── Credits
            └── Balance

Esto es especialmente importante en modelos B2B.

8. Multi-Tenant Billing

Billing debe ser nativamente multi-tenant.

Cada operación financiera debe mantener contexto suficiente para determinar:

Enterprise
   ↓
Organization
   ↓
Tenant
   ↓
Billing Account
   ↓
Customer
   ↓
Transaction

Esto permite:

aislamiento financiero;
reporting por tenant;
límites de acceso;
configuración tributaria independiente;
pricing independiente;
monedas diferentes;
métodos de pago diferentes;
reglas comerciales diferentes.
9. Multi-Currency

Una plataforma Enterprise puede operar simultáneamente con:

USD
EUR
CLP
MXN
BRL
COP
PEN
ARS
GBP
...

Billing debe distinguir entre:

moneda del producto;
moneda del precio;
moneda de facturación;
moneda de pago;
moneda contable;
moneda de reporting.

Nunca debe asumirse que:

product_currency = billing_currency = accounting_currency
10. Billing Events

Billing debe funcionar principalmente alrededor de eventos financieros.

Ejemplos:

SubscriptionCreated
SubscriptionActivated
UsageRecorded
ChargeCreated
InvoiceGenerated
InvoiceFinalized
PaymentAuthorized
PaymentCaptured
PaymentFailed
PaymentRefunded
CreditApplied
InvoiceAdjusted
PaymentDisputed
SubscriptionRenewed
SubscriptionCancelled

Cada evento debe poseer:

identificador;
timestamp;
tenant;
billing account;
actor;
source;
correlation ID;
causation ID;
estado;
metadata.

Esto permite construir una arquitectura financiera auditable.

11. Billing Ledger

Una plataforma avanzada no debe depender únicamente del estado actual de las tablas.

Debe existir una representación histórica de las operaciones financieras.

Financial Event
       │
       ▼
Billing Transaction
       │
       ▼
Ledger Entry
       │
       ├── Debit
       └── Credit

El ledger proporciona:

trazabilidad;
auditoría;
reconstrucción histórica;
conciliación;
debugging financiero;
integración contable.
12. Billing State Machine

Los objetos financieros deben manejar estados explícitos.

Por ejemplo, una factura:

DRAFT
  │
  ▼
FINALIZED
  │
  ▼
OPEN
  │
  ├───────────────┐
  ▼               ▼
PAID            PAST_DUE
                  │
                  ▼
              COLLECTION
                  │
                  ▼
               WRITTEN_OFF

Una suscripción puede seguir:

TRIAL
  │
  ▼
ACTIVE
  │
  ├── PAUSED
  │      │
  │      ▼
  │    ACTIVE
  │
  ├── PAST_DUE
  │
  └── CANCELLED

Los estados deben ser controlados por reglas de negocio y no simplemente por modificaciones arbitrarias de columnas.

13. Billing e IA

En EVOXA, Billing puede evolucionar desde una plataforma transaccional hacia una AI-powered Billing Platform.

La IA puede participar en:

Revenue Intelligence

Predicción de:

MRR;
ARR;
churn;
expansion revenue;
contraction revenue.
Payment Intelligence

Predicción de:

payment failure;
probabilidad de recuperación;
mejor momento para retry;
riesgo de morosidad.
Pricing Intelligence

Análisis de:

elasticidad;
utilización;
pricing tiers;
oportunidades de upsell;
pricing optimization.
FinOps

Análisis de:

costo por tenant;
costo por cliente;
costo por feature;
margen;
consumo de infraestructura.
14. Billing Analytics

Billing debe producir métricas como:

Revenue
MRR
ARR
Gross Revenue
Net Revenue
Recognized Revenue
Deferred Revenue
Customer
ARPU
LTV
CAC
Churn
Expansion Revenue
Contraction Revenue
Billing
Invoice Volume
Invoice Value
Collection Rate
Payment Success Rate
Payment Failure Rate
DSO
Outstanding Balance
Usage
Usage Volume
Billable Usage
Free Usage
Overage
Usage Revenue
15. Billing como sistema financiero confiable

El principio fundamental debe ser:

Billing debe priorizar exactitud financiera sobre velocidad de implementación.

Una plataforma puede tolerar que una pantalla tarde algunos segundos más en cargar.

No puede tolerar:

$100 facturados → $120 cobrados

o:

Payment received
        ↓
Invoice remains unpaid

o:

Revenue recognized twice

Por eso Billing debe implementar:

idempotencia;
transaction integrity;
immutable financial records;
reconciliation;
audit trails;
event sourcing cuando corresponda;
retry safety;
duplicate prevention.
16. Principios arquitectónicos

Los principios fundamentales de Billing son:

1. Accuracy

Los cálculos financieros deben ser exactos.

2. Determinism

El mismo input debe producir el mismo resultado financiero.

3. Idempotency

Procesar dos veces un evento no debe generar dos cargos.

4. Auditability

Toda operación debe poder reconstruirse.

5. Traceability

Debe existir trazabilidad:

Usage
 → Charge
 → Invoice
 → Payment
 → Revenue
 → Accounting
6. Isolation

Los datos financieros deben respetar el aislamiento del tenant.

7. Extensibility

El sistema debe permitir nuevos modelos de pricing y monetización.

8. Compliance

Las operaciones deben poder adaptarse a requisitos fiscales y financieros.

17. Modelo conceptual

La estructura conceptual de Billing puede resumirse como:

                    BILLING PLATFORM
                           │
       ┌───────────────────┼───────────────────┐
       │                   │                   │
    Catalog             Pricing             Usage
       │                   │                   │
       └───────────────────┼───────────────────┘
                           │
                      Subscription
                           │
                           ▼
                       Metering
                           │
                           ▼
                        Rating
                           │
                           ▼
                        Charges
                           │
                           ▼
                        Invoice
                           │
                  ┌────────┴────────┐
                  ▼                 ▼
               Payment           Credit
                  │                 │
                  └────────┬────────┘
                           ▼
                       Settlement
                           │
                           ▼
                     Revenue Engine
                           │
                           ▼
                      Accounting
                           │
                           ▼
                  Financial Analytics
18. Enterprise Billing Vision

La visión final no es simplemente:

"generar facturas".

La visión es construir una Financial Monetization Platform capaz de administrar todo el ciclo económico de una plataforma digital.

                    MONETIZATION
                         │
                         ▼
                   BILLING PLATFORM
                         │
       ┌─────────────────┼─────────────────┐
       │                 │                 │
   Commercial         Financial          Intelligence
       │                 │                 │
       ▼                 ▼                 ▼
 Products            Payments          Analytics
 Pricing             Invoices          Forecasting
 Plans               Revenue           AI
 Subscriptions       Accounting        Optimization
 Usage               Collections       FinOps

Billing se convierte así en una de las plataformas fundamentales del ecosistema Enterprise, conectando Commerce + Operations + Finance + Data + AI.

19. Resultado de 26/01

Billing Overview establece las bases conceptuales para todo el capítulo 26.

El capítulo parte desde esta definición:

Billing es una plataforma financiera transaccional y de monetización que convierte productos, contratos, suscripciones y consumo en cargos, facturas, pagos, ingresos y conocimiento financiero, manteniendo exactitud, trazabilidad, seguridad y cumplimiento a escala Enterprise.

A partir de esta base, las siguientes secciones profundizan progresivamente en el modelo empresarial, arquitectura, organizaciones, tenants, cuentas, clientes, productos, pricing, subscriptions, invoicing, payments, usage, revenue, collections, analytics y evolución de la plataforma.
    
