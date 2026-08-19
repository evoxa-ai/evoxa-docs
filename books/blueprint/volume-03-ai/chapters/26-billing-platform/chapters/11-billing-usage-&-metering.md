1. Billing Usage & Metering

Billing Usage & Metering define la capacidad de la plataforma para capturar, normalizar, medir, agregar y preparar para monetización el consumo real de productos y servicios.

Esta capa es especialmente importante para EVOXA porque muchos servicios pueden monetizarse en función del consumo:

eventos;
API calls;
usuarios;
almacenamiento;
campañas;
audiencia;
movilidad;
procesamiento;
consultas;
tokens de IA;
ejecuciones de agentes;
datos procesados.

La relación fundamental es:

Customer
   │
   ▼
Subscription
   │
   ▼
Product / Plan
   │
   ▼
Meter Definition
   │
   ▼
Usage Events
   │
   ▼
Usage Aggregation
   │
   ▼
Rated Usage
   │
   ▼
Charge
   │
   ▼
Invoice

La diferencia fundamental es:

Usage registra cuánto se utilizó. Metering determina cómo medir ese consumo. Rating determina cuánto vale ese consumo.

2. Propósito

Billing Usage & Metering debe proporcionar:

captura de consumo;
ingestión de eventos;
normalización;
validación;
deduplicación;
identificación del tenant;
identificación del customer;
asociación a subscription;
asociación a product;
definición de unidades;
agregación;
ventanas temporales;
cálculo de consumo facturable;
control de límites;
detección de anomalías;
trazabilidad;
idempotencia;
procesamiento histórico;
integración con Pricing;
preparación para Rating.
3. Usage vs Metering

Son conceptos diferentes.

Usage

Es el consumo real.

Customer A
utilizó
125,000 API calls
Metering

Es la forma en que ese consumo es medido.

Meter
API_CALL
Unit = request
Aggregation = SUM

Por lo tanto:

Usage
   ↓
Meter
   ↓
Measurement
4. Usage vs Rating

También deben separarse:

Usage
= 1,500,000 events

Mientras:

Rating
= 1,500,000 × $0.0005
= $750

La arquitectura debe evitar mezclar:

Measurement
≠
Pricing
5. Usage dentro de Billing

La cadena completa:

Usage Event
     │
     ▼
Usage Validation
     │
     ▼
Normalization
     │
     ▼
Meter
     │
     ▼
Aggregation
     │
     ▼
Rated Usage
     │
     ▼
Charge
     │
     ▼
Invoice

Esto permite mantener una clara separación entre:

datos de consumo;
reglas de medición;
reglas comerciales;
operaciones financieras.
6. Usage Sources

El consumo puede provenir de diferentes fuentes:

                 USAGE
                   │
      ┌────────────┼────────────┐
      ▼            ▼            ▼
     API        Application    Devices
      │            │            │
      ▼            ▼            ▼
   Platform      Services      IoT
      │            │            │
      └────────────┼────────────┘
                   ▼
             Usage Ingestion

También:

data pipelines;
mobile applications;
backend services;
AI engines;
cloud providers;
external APIs;
partner systems;
marketplaces.
7. Usage Event

El elemento básico es el Usage Event.

Conceptualmente:

UsageEvent
│
├── event_id
├── tenant_id
├── customer_id
├── billing_account_id
├── subscription_id
├── product_id
├── meter_id
├── metric
├── quantity
├── unit
├── occurred_at
├── source
└── metadata

Ejemplo:

event_id       = EVT-10001
tenant_id      = TENANT-01
subscription   = SUB-100
metric         = API_CALL
quantity       = 150
unit           = request
timestamp      = 2026-08-17 13:00
8. Usage Event Identity

Cada evento debe tener una identidad única:

event_id

Esto es fundamental para evitar doble facturación.

Si el mismo evento llega dos veces:

EVT-10001
EVT-10001

el sistema debe reconocer la duplicación.

9. Idempotency

El procesamiento debe ser idempotente.

Event Received
      │
      ▼
Check event_id
      │
 ┌────┴────┐
 ▼         ▼
New     Duplicate
 │         │
 ▼         ▼
Process   Ignore

Esto es crítico cuando:

se reintentan requests;
se procesan mensajes;
se recuperan fallos;
se reprocesan archivos;
se utilizan queues.
10. Usage Ingestion

La ingestión puede ocurrir mediante:

API
POST /billing/usage/events
Batch
CSV
JSON
Parquet
Event Streaming
Kafka
Pub/Sub
Event Bus
Queue
Internal Events
Application Event
       ↓
Billing Usage
11. Real-Time Usage

Para ciertos productos se requiere medición casi en tiempo real.

Application
    │
    ▼
Usage Event
    │
    ▼
Event Stream
    │
    ▼
Metering Engine
    │
    ▼
Current Usage

Esto permite mostrar:

API Usage
750,000 / 1,000,000

en el portal.

12. Batch Usage

Otros consumos pueden procesarse periódicamente:

Daily Usage
     │
     ▼
Batch Processing
     │
     ▼
Aggregation
     │
     ▼
Billing Usage

Esto puede utilizarse para:

grandes datasets;
movilidad;
procesamiento de datos;
analytics;
almacenamiento.
13. Usage Normalization

Las fuentes pueden utilizar diferentes unidades.

Source A → 1,000 requests
Source B → 1K API calls
Source C → 1000 req

El sistema debe normalizar:

Canonical Metric
API_CALL


Canonical Unit
request

Así todas las fuentes producen:

1,000 requests
14. Canonical Usage Model

El modelo canónico:

metric
unit
quantity
timestamp
tenant
customer
subscription
product
source

Esto desacopla Billing de los sistemas que producen el consumo.

15. Meter Definition

Un Meter define cómo debe medirse una métrica.

Meter
│
├── metric
├── unit
├── aggregation
├── dimensions
├── filters
├── reset_period
└── status

Ejemplo:

Meter
Metric = API_CALL
Unit = request
Aggregation = SUM
16. Meter Types

Los meters pueden medir:

Count

Cantidad de eventos.

API calls = 1,000
Sum

Suma de cantidades.

Data Processed = 500 GB
Gauge

Valor en un momento determinado.

Active Users = 250
Maximum

Máximo durante un período.

Peak Concurrent Users = 500
Average

Promedio durante un período.

Average Storage = 1.2 TB
17. Meter Aggregation

La agregación define cómo se transforma una colección de eventos.

Events:
10
20
30
40
SUM
100
COUNT
4
MAX
40
AVG
25

La elección de la agregación puede cambiar completamente el resultado financiero.

18. Meter Dimensions

El consumo puede medirse con dimensiones:

Usage
│
├── Region
├── Product
├── Model
├── Environment
├── API
├── Tenant
└── Customer

Ejemplo:

AI Tokens
│
├── Model A
├── Model B
└── Model C

Esto permite pricing diferenciado.

19. Meter Filters

Un meter puede medir únicamente determinados eventos.

Meter
Metric = API_CALL


Filter:
environment = production

Por lo tanto:

Production Calls
→ Billable


Development Calls
→ Not Billable
20. Billable vs Non-Billable Usage

No todo usage debe generar cargos.

Usage
│
├── Billable
├── Included
├── Promotional
├── Internal
├── Free
└── Excluded

Ejemplo:

1M events included
500K events overage
21. Included Usage

Un plan puede incluir una cantidad:

Included
1,000,000 API calls

El sistema calcula:

Actual Usage
      -
Included Usage
      =
Billable Overage

Ejemplo:

Actual = 1,250,000
Included = 1,000,000


Billable = 250,000
22. Usage Entitlements

El mismo usage puede controlar acceso:

Usage
   │
   ├── Billing
   │
   └── Entitlement

Ejemplo:

80% → Warning
100% → Limit
>100% → Overage

La política puede determinar si se:

permite;
advierte;
factura;
bloquea.
23. Usage Windows

El consumo debe agruparse por períodos.

Hourly
Daily
Weekly
Monthly
Billing Period
Custom

Ejemplo:

Billing Period
01/08 → 31/08

Todos los eventos dentro de esa ventana pueden agregarse para billing.

24. Usage Periods

Un evento puede pertenecer a:

occurred_at
       │
       ▼
Billing Period
       │
       ▼
Usage Window

Esto es importante cuando:

el ciclo no comienza el día 1;
existe proration;
hay cambios de plan;
existen múltiples subscriptions.
25. Usage Time

Debe distinguirse entre:

Event Time

Cuándo ocurrió el consumo.

Ingestion Time

Cuándo llegó al sistema.

Processing Time

Cuándo fue procesado.

Ejemplo:

Event Time      12:00
Ingestion       12:05
Processing      12:06

Billing debe utilizar normalmente event time para determinar a qué período pertenece el consumo.

26. Late-Arriving Usage

Puede ocurrir:

Event occurred
31 Aug

pero llega:

02 Sep

El sistema debe poder incorporar el evento al período correcto.

Esto es especialmente importante para billing mensual.

27. Usage Corrections

Si un evento fue incorrecto:

Original Usage
10,000

no debería simplemente eliminarse.

Debe generarse:

Correction
-2,000

Resultado:

Net Usage = 8,000

Esto mantiene trazabilidad.

28. Usage Adjustments

También pueden existir ajustes manuales:

Measured Usage
100,000


Approved Adjustment
-5,000


Billable Usage
95,000

Los ajustes deben requerir:

actor;
razón;
aprobación cuando corresponda;
timestamp.
29. Usage Validation

Cada evento debe validarse.

Usage Event
   │
   ├── Tenant valid?
   ├── Subscription valid?
   ├── Meter valid?
   ├── Metric valid?
   ├── Unit valid?
   ├── Quantity valid?
   └── Timestamp valid?

Los eventos inválidos deben ir a una cola de errores.

30. Usage Rejection

Ejemplo:

Unknown Meter

Resultado:

Usage Status = REJECTED
Reason = METER_NOT_FOUND

Debe ser posible reprocesarlo después de corregir la configuración.

31. Usage Status

Los eventos pueden tener:

RECEIVED
VALIDATED
ACCEPTED
AGGREGATED
RATED
BILLED
REJECTED
CORRECTED
VOIDED

Esto permite seguimiento end-to-end.

32. Usage Pipeline
Source
  │
  ▼
Ingestion
  │
  ▼
Validation
  │
  ▼
Normalization
  │
  ▼
Deduplication
  │
  ▼
Metering
  │
  ▼
Aggregation
  │
  ▼
Rating
  │
  ▼
Charge
  │
  ▼
Invoice

Este pipeline es uno de los componentes críticos de Billing.

33. Usage Aggregation

En lugar de procesar millones de eventos directamente durante la facturación:

1,000,000 Events
       │
       ▼
Aggregation
       │
       ▼
Monthly Usage

Resultado:

API_CALL
1,000,000 requests

Esto mejora:

performance;
costo;
escalabilidad.
34. Usage Aggregation Dimensions

Puede agregarse por:

Tenant
Customer
Subscription
Product
Meter
Region
Model
Environment
Billing Period

Ejemplo:

Customer
   │
   ├── API = 500K
   ├── AI = 10M tokens
   └── Storage = 2TB
35. Usage Ledger

Puede existir un ledger específico de consumo:

Usage Ledger
│
├── Event
├── Quantity
├── Adjustment
├── Net Usage
├── Meter
├── Period
└── Status

Esto permite reconstruir el consumo facturable.

36. Usage Snapshot

Antes de generar una factura:

Billing Period Closed
       │
       ▼
Usage Snapshot
       │
       ▼
Rating

El snapshot representa el consumo utilizado para generar cargos.

Esto ayuda a mantener consistencia.

37. Usage Finalization

Al cerrar un período:

OPEN
  │
  ▼
CLOSING
  │
  ▼
FINALIZED

Una vez finalizado:

se limita la modificación;
las correcciones deben utilizar ajustes;
el snapshot debe conservarse.
38. Usage Reprocessing

Si una regla fue configurada incorrectamente:

Incorrect Meter
      │
      ▼
Detect Error
      │
      ▼
Reprocess Usage
      │
      ▼
New Rating

La operación debe ser controlada para evitar doble facturación.

39. Usage and Rating Separation

La arquitectura debe separar:

Usage
   │
   ▼
How Much?
   │
   ▼
Metering
   │
   ▼
Rating
   │
   ▼
How Much Money?

Ejemplo:

Usage = 250,000 events


Metering
= 250,000 billable events


Rating
= 250,000 × $0.0005


Charge
= $125
40. Usage-Based Pricing

La relación completa:

Product
   │
   ▼
Plan
   │
   ▼
Meter
   │
   ▼
Usage
   │
   ▼
Aggregation
   │
   ▼
Pricing Rule
   │
   ▼
Rated Usage
   │
   ▼
Charge

Esto conecta directamente con 26/09 — Billing Pricing & Plans.

41. Usage + Subscription

Cada evento debe poder determinar qué subscription corresponde.

Usage Event
     │
     ▼
Tenant
     │
     ▼
Subscription
     │
     ▼
Product
     │
     ▼
Meter

Si existen varias subscriptions, deben utilizarse reglas explícitas de resolución.

42. Usage Allocation

Cuando múltiples subscriptions comparten consumo:

Usage
   │
   ├── Subscription A
   ├── Subscription B
   └── Subscription C

La asignación puede basarse en:

API key;
project;
tenant;
product;
environment;
customer;
contract.
43. Usage Attribution

Toda unidad de consumo debe poder atribuirse.

Usage
│
├── Enterprise
├── Organization
├── Tenant
├── Customer
├── Billing Account
├── Subscription
├── Product
└── Meter

Esto permite responder:

¿Quién consumió?

¿Qué consumió?

¿Cuánto consumió?

¿Por qué se facturó?

44. Usage for EVOXA

Para EVOXA pueden existir meters como:

EVOXA Usage Meters
│
├── API_REQUEST
├── AI_INPUT_TOKEN
├── AI_OUTPUT_TOKEN
├── AI_AGENT_EXECUTION
├── MOBILITY_EVENT
├── AUDIENCE_QUERY
├── OOH_ANALYSIS
├── REPORT_GENERATED
├── MAP_REQUEST
├── DATA_EXPORT
├── STORAGE_GB
└── DATA_PROCESSING_GB

Esto permite monetización flexible.

45. AI Usage Metering

Los productos de IA requieren mayor granularidad.

AI Usage
│
├── Model
├── Input Tokens
├── Output Tokens
├── Requests
├── Context Tokens
├── Agent Runs
└── Compute

Ejemplo:

Model = GPT-like model
Input = 10M tokens
Output = 4M tokens
Requests = 50K

Cada dimensión puede tener pricing independiente.

46. Mobility Usage Metering

Para EVOXA Mobility:

Mobility Usage
│
├── Events Processed
├── Devices
├── Trajectories
├── OD Queries
├── H3 Queries
├── Heatmaps
└── Data Exports

Esto puede permitir monetizar tanto:

volumen de datos;
consultas;
resultados analíticos.
47. OOH Usage Metering

Para OOH:

OOH Usage
│
├── Supports Analyzed
├── Campaigns
├── Audience Queries
├── Reach Calculations
├── Frequency Calculations
└── Exports

Esto puede utilizarse para pricing híbrido.

48. Data Usage Metering

Para productos de datos:

Data Product
│
├── Records
├── GB Processed
├── GB Delivered
├── API Requests
├── Downloads
└── Queries
49. Usage Quotas

Los clientes pueden tener cuotas:

Monthly Quota
1,000,000 API Calls

El sistema mantiene:

Used
750,000


Remaining
250,000

Esto puede exponerse en el portal.

50. Usage Alerts

Se pueden generar alertas:

Usage Thresholds
│
├── 50%
├── 75%
├── 80%
├── 90%
└── 100%

Ejemplo:

Usage = 90%
→ Alert

Las alertas pueden dirigirse a:

Customer;
Tenant admin;
Finance;
Operations.
51. Usage Anomaly Detection

La plataforma puede identificar:

Normal
100K events/day


Observed
2.5M events/day

Resultado:

Usage Anomaly
│
├── Expected
├── Actual
├── Variance
├── Metric
├── Tenant
└── Risk

Esto ayuda a detectar:

errores;
abuso;
integraciones defectuosas;
consumo inesperado;
fraude.
52. Usage Forecasting

El sistema puede proyectar consumo:

Historical Usage
      │
      ▼
Forecast Model
      │
      ▼
Expected Usage
      │
      ▼
Expected Billing

Ejemplo:

Current Usage
750K


Forecast
1.25M


Expected Overage
250K

Esto permite anticipar revenue y costos.

53. Usage-Based Revenue Forecast

Puede calcularse:

Expected Usage
       ×
Applicable Rate
       =
Expected Revenue

Esto alimenta:

revenue forecasting;
financial planning;
FinOps;
capacity planning.
54. Usage Security

El sistema debe proteger contra manipulación.

Usage Source
     │
     ▼
Authentication
     │
     ▼
Authorization
     │
     ▼
Validation
     │
     ▼
Usage Accepted

Deben existir controles contra:

falsificación;
duplicación;
replay;
timestamps inválidos;
cantidades imposibles.
55. Usage Audit

Eventos importantes:

UsageReceived
UsageValidated
UsageRejected
UsageCorrected
UsageAggregated
UsageFinalized
UsageReprocessed
UsageAdjusted
UsageBilled

Cada evento debe ser auditable.

56. Usage Data Model

Modelo conceptual:

UsageEvent
│
├── id
├── event_id
├── organization_id
├── tenant_id
├── customer_id
├── billing_account_id
├── subscription_id
├── subscription_item_id
├── product_id
├── meter_id
├── metric
├── quantity
├── unit
├── occurred_at
├── received_at
├── processed_at
├── source
├── status
├── idempotency_key
└── metadata
57. Meter Data Model
Meter
│
├── id
├── meter_code
├── name
├── metric
├── unit
├── aggregation_type
├── reset_period
├── dimensions
├── filter_definition
├── status
├── version
├── effective_from
└── effective_to
58. Usage Aggregate Model
UsageAggregate
│
├── id
├── tenant_id
├── customer_id
├── subscription_id
├── product_id
├── meter_id
├── period_start
├── period_end
├── quantity
├── billable_quantity
├── included_quantity
├── overage_quantity
├── adjustment_quantity
├── status
└── version

Este objeto es muy importante para separar el consumo bruto del consumo facturable.

59. Usage Reconciliation

Antes de facturar:

Source Usage
      │
      ▼
Billing Usage
      │
      ▼
Reconciliation
      │
 ┌────┴────┐
 ▼         ▼
Match    Difference
           │
           ▼
        Exception

Esto permite detectar pérdida o duplicación de eventos.

60. Usage Completeness

El sistema debe poder responder:

¿Recibimos todo el consumo esperado?

Ejemplo:

Expected Events
1,000,000


Received
995,000


Missing
5,000

Antes de cerrar billing puede generarse una excepción.

61. Usage Processing Architecture
             USAGE SOURCES
                  │
      ┌───────────┼───────────┐
      ▼           ▼           ▼
     API        Events       Batch
      │           │           │
      └───────────┼───────────┘
                  ▼
             INGESTION
                  │
                  ▼
             VALIDATION
                  │
                  ▼
            DEDUPLICATION
                  │
                  ▼
            NORMALIZATION
                  │
                  ▼
               METERING
                  │
                  ▼
             AGGREGATION
                  │
                  ▼
              FINALIZE
                  │
                  ▼
                RATING
                  │
                  ▼
               CHARGING
62. Usage Scalability

La arquitectura debe soportar desde:

100 events/day

hasta:

billions of events/day

Para esto se requieren:

procesamiento asíncrono;
particionamiento;
colas;
batching;
agregación;
almacenamiento optimizado;
idempotencia;
backpressure.
63. Usage Storage Architecture

Puede separarse:

Raw Usage
    │
    ▼
Event Storage
    │
    ▼
Normalized Usage
    │
    ▼
Aggregated Usage
    │
    ▼
Billing Usage Store

Esto permite mantener el evento original y al mismo tiempo disponer de estructuras optimizadas para billing.

64. Usage Retention

No todo usage necesita mantenerse con el mismo nivel de granularidad para siempre.

Puede existir:

Raw Events
     │
     ▼
90 Days
     │
     ▼
Aggregated Data
     │
     ▼
Long-Term Retention

Las políticas dependerán de:

regulación;
contratos;
auditoría;
necesidades analíticas.
65. Usage Privacy

El sistema debe evitar almacenar datos personales innecesarios.

Especialmente cuando el consumo contiene:

identificadores;
IPs;
device IDs;
ubicación;
información sensible.

Billing debería almacenar el mínimo necesario para:

medición;
trazabilidad;
auditoría.
66. Usage API
Event ingestion
POST /billing/usage/events
Batch ingestion
POST /billing/usage/batches
Query
GET /billing/usage
GET /billing/usage/{id}
Customer usage
GET /billing/customers/{id}/usage
Subscription usage
GET /billing/subscriptions/{id}/usage
Meter
GET /billing/meters
POST /billing/meters
PATCH /billing/meters/{id}
67. Usage Preview

Antes de facturar:

GET /billing/subscriptions/{id}/usage-preview

Puede devolver:

Included Usage
1,000,000


Current Usage
1,250,000


Billable Overage
250,000


Estimated Charge
$125

Esto proporciona transparencia al cliente.

68. Usage Billing Flow

El flujo completo de EVOXA:

Customer
   ↓
Subscription
   ↓
Product / Plan
   ↓
Meter Definition
   ↓
Usage Event
   ↓
Validation
   ↓
Deduplication
   ↓
Normalization
   ↓
Aggregation
   ↓
Included Usage
   ↓
Overage
   ↓
Pricing
   ↓
Rated Usage
   ↓
Charge
   ↓
Invoice
69. Usage + FinOps

El mismo consumo puede alimentar costos internos.

Usage
  │
  ├── Customer Billing
  │
  └── Internal Cost
          │
          ▼
       FinOps

Ejemplo:

AI Tokens
   │
   ├── Customer Charge
   │
   └── Model Provider Cost

Esto permite calcular:

Customer Revenue
-
Provider Cost
=
Gross Margin
70. Usage + Revenue

El usage puede alimentar revenue forecasting:

Usage Forecast
      │
      ▼
Pricing
      │
      ▼
Expected Charges
      │
      ▼
Revenue Forecast

Esto conecta directamente Usage con:

Revenue Management;
Financial Planning;
FinOps.
71. Usage + AI Intelligence

La plataforma puede analizar consumo:

Usage History
      │
      ▼
AI Analytics
      │
 ┌────┼──────────────┐
 ▼    ▼              ▼
Forecast Anomaly   Optimization

Puede recomendar:

upgrade;
plan optimization;
quota changes;
cost optimization;
usage reduction.
72. Enterprise Usage Model

Para un cliente Enterprise:

                     CUSTOMER
                        │
                        ▼
                 BILLING ACCOUNT
                        │
                        ▼
                   SUBSCRIPTION
                        │
       ┌────────────────┼────────────────┐
       ▼                ▼                ▼
    Tenant A         Tenant B         Tenant C
       │                │                │
       ▼                ▼                ▼
     Usage            Usage            Usage
       │                │                │
       └────────────────┼────────────────┘
                        ▼
                 Consolidated Usage
                        │
                        ▼
                      Rating
                        │
                        ▼
                    Invoice

Esto permite mantener detalle por tenant y al mismo tiempo facturación consolidada.

73. Core Principles
1. Usage ≠ Pricing

Medir consumo no significa determinar su precio.

2. Meter ≠ Usage

El Meter define cómo se mide; Usage representa lo ocurrido.

3. Idempotency

Un evento nunca debe facturarse dos veces.

4. Immutable Raw Events

El consumo original debe conservarse.

5. Corrections Through Adjustments

Las correcciones deben ser trazables.

6. Deterministic Aggregation

La misma entrada debe producir el mismo resultado.

7. Late Data Support

Los eventos atrasados deben poder procesarse correctamente.

8. Scalability

Debe soportar desde pequeños volúmenes hasta grandes volúmenes de eventos.

9. Multi-Tenant Attribution

Cada unidad de consumo debe poder atribuirse.

10. Billing Integrity

El consumo utilizado para facturación debe poder reconstruirse.

74. Resultado de 26/11

Billing Usage & Metering establece la infraestructura necesaria para convertir el consumo operativo de EVOXA en unidades de consumo medibles, auditables y posteriormente monetizables.

La cadena queda:

Usage Source
   ↓
Usage Event
   ↓
Validation
   ↓
Deduplication
   ↓
Normalization
   ↓
Meter
   ↓
Aggregation
   ↓
Billable Usage
   ↓
Pricing / Rating
   ↓
Charge
   ↓
Invoice
   ↓
Revenue

Y el principio fundamental es:

Billing Usage & Metering debe separar estrictamente el hecho de consumir de la valoración económica de ese consumo, proporcionando una infraestructura idempotente, escalable, versionada y auditable capaz de medir desde operaciones simples hasta grandes volúmenes de consumo de datos, movilidad, analytics e inteligencia artificial.

Con esto, el siguiente componente natural del capítulo es:

26/12 — Billing Rating & Charge Calculation
