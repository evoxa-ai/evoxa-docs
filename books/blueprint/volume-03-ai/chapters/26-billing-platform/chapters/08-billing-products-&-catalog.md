1. Billing Products & Catalog

Billing Products & Catalog define la estructura mediante la cual la plataforma administra todo aquello que puede ser ofrecido, vendido, contratado, consumido y monetizado.

El catálogo constituye el punto de conexión entre el mundo comercial y Billing.

Mientras:

Product define qué se ofrece;
Plan define cómo se empaqueta;
Price define cuánto cuesta;
Subscription define qué contrató el cliente;
Usage define cuánto consumió;
Billing determina cuánto debe pagar.

La relación fundamental es:

Product
   │
   ▼
Catalog
   │
   ▼
Offer / Plan
   │
   ▼
Price
   │
   ▼
Subscription / Order
   │
   ▼
Usage / Charges
   │
   ▼
Invoice

El catálogo debe ser versionado, gobernado, extensible y desacoplado del motor financiero.

2. Propósito del Catalog

El Billing Catalog debe permitir:

definir productos;
organizar productos;
crear planes;
definir features;
administrar add-ons;
crear bundles;
definir SKUs;
asociar precios;
administrar versiones;
controlar disponibilidad;
definir reglas comerciales;
manejar productos digitales;
manejar servicios;
administrar productos Enterprise;
soportar productos basados en consumo.

El catálogo se convierte en la fuente de verdad comercial para los objetos monetizables.

3. Product vs Catalog

No son exactamente lo mismo.

Product

Representa una cosa comercial.

Product
= EVOXA Analytics
Catalog

Representa el conjunto estructurado de productos disponibles.

Catalog
│
├── Analytics
├── Mobility
├── AI
├── OOH
├── Data
└── Enterprise Services

Por lo tanto:

Catalog
   └── Products
4. Product Model

Un producto puede representar:

software;
servicio;
API;
módulo;
feature;
datos;
capacidad de IA;
almacenamiento;
procesamiento;
soporte;
servicio profesional.

Ejemplo:

EVOXA Analytics Platform

Puede contener:

EVOXA Analytics
│
├── Dashboard
├── Mobility Analytics
├── OOH Analytics
├── Audience Analytics
├── AI Insights
└── Reporting
5. Product Hierarchy

Los productos pueden organizarse jerárquicamente:

EVOXA Platform
│
├── Analytics
│   ├── Audience
│   ├── Mobility
│   └── OOH
│
├── AI
│   ├── AI Insights
│   ├── AI Forecasting
│   └── AI Agents
│
└── Data
    ├── Mobility Data
    ├── Audience Data
    └── Location Data

Esto facilita navegación y administración.

6. Product Types

El catálogo puede soportar:

Subscription Product

Producto vendido mediante suscripción.

Usage Product

Producto cobrado por consumo.

One-Time Product

Compra única.

Add-On

Funcionalidad adicional.

Bundle

Conjunto de productos.

Service

Servicio profesional.

Data Product

Producto de datos.

AI Product

Producto basado en capacidades de IA.

Platform Product

Producto principal de la plataforma.

7. Product Lifecycle

Un producto debe tener lifecycle explícito:

DRAFT
  │
  ▼
PREVIEW
  │
  ▼
ACTIVE
  │
  ├── DEPRECATED
  │
  ▼
RETIRED

Un producto retirado no debería desaparecer del historial financiero.

Debe seguir siendo consultable para:

subscriptions históricas;
invoices;
revenue;
reporting;
auditoría.
8. Product Status

Los estados pueden ser:

DRAFT
ACTIVE
PAUSED
DEPRECATED
RETIRED
ARCHIVED

La diferencia entre DEPRECATED y RETIRED es importante.

Deprecated

No se recomienda venderlo a nuevos clientes.

Retired

Ya no puede utilizarse comercialmente.

9. Product Versioning

Los productos deben ser versionables.

EVOXA Analytics


v1
v2
v3

Esto permite evolucionar características sin romper referencias históricas.

Subscription
      │
      ▼
Product Version 2

Una subscription histórica debe seguir apuntando a la versión correspondiente.

10. Product Features

Un producto puede estar compuesto por features.

Product
│
├── Feature A
├── Feature B
├── Feature C
└── Feature D

Ejemplo:

Analytics Platform
│
├── Dashboard
├── Maps
├── Audience
├── Reports
└── AI Insights

Las features pueden utilizarse para:

entitlement;
pricing;
usage;
permissions;
packaging.
11. Product Entitlements

El catálogo puede definir qué tiene derecho a utilizar el cliente.

Plan Professional
      │
      ▼
Entitlements
      │
      ├── 10 Users
      ├── 100 Reports
      ├── 1M Events
      └── AI Insights

Billing puede generar cargos mientras otro sistema controla el acceso.

Esto permite separar:

Billing
   ≠
Entitlement
   ≠
Authorization
12. Product SKU

Cada producto monetizable puede tener un SKU.

SKU
│
├── Internal Code
├── External Code
├── Product
├── Version
└── Status

Ejemplo:

EVOXA-ANALYTICS-PRO

Los SKU facilitan:

ERP integration;
accounting;
reporting;
inventory conceptual;
external systems.
13. Product Categories

Los productos pueden clasificarse:

Catalog
│
├── SaaS
├── Data
├── AI
├── Analytics
├── Services
└── Infrastructure

Las categorías pueden tener:

metadata;
tax category;
pricing rules;
visibility rules.
14. Product Bundles

Un bundle agrupa productos.

Enterprise Bundle
│
├── Analytics
├── Mobility
├── OOH
├── AI
└── Support

Puede tener un precio único o componentes individualmente valorizados.

15. Bundle Pricing

Dos modelos comunes:

Bundle Price
Analytics     $500
Mobility      $300
AI            $200
────────────────
Bundle        $800
Component Pricing
Bundle
│
├── Analytics → $500
├── Mobility  → $300
└── AI        → $200

Billing debe poder manejar ambos.

16. Add-Ons

Los Add-ons amplían un producto principal.

Base Plan
   │
   ├── AI Add-on
   ├── Storage Add-on
   ├── Users Add-on
   └── Support Add-on

Pueden ser:

recurrentes;
one-time;
usage-based.
17. Product Packaging

El producto puede empaquetarse en diferentes ofertas:

Product
   │
   ├── Starter
   ├── Professional
   ├── Business
   └── Enterprise

Cada plan puede tener:

features diferentes;
límites diferentes;
precios diferentes;
billing cycles diferentes.
18. Plans

El Plan representa un paquete comercial.

Product
   │
   └── Plan
        │
        ├── Features
        ├── Limits
        ├── Entitlements
        └── Price

Ejemplo:

Professional
$499 / month
19. Plan Versioning

Los planes deben poder versionarse.

Professional v1
Professional v2
Professional v3

Esto permite:

Existing Customers
       ↓
v1


New Customers
       ↓
v3

sin modificar retroactivamente los contratos existentes.

20. Pricing Separation

Una regla arquitectónica fundamental:

Product ≠ Price

Por ejemplo:

Product
"EVOXA Analytics"


Prices
├── $500 USD/month
├── $5,000 USD/year
├── $450 EUR/month
└── Custom Enterprise Price

Esto permite múltiples modelos comerciales sobre un mismo producto.

21. Price Objects

El Price Object puede contener:

Price
│
├── Amount
├── Currency
├── Billing Interval
├── Pricing Model
├── Meter
├── Tax Behavior
├── Effective Date
└── Version

Ejemplo:

Amount = 500
Currency = USD
Interval = MONTH
Model = FLAT
22. Pricing Models

El catálogo debe soportar:

Flat
Per Seat
Per Unit
Usage
Tiered
Volume
Graduated
Hybrid
Commitment
Custom Contract

Esto conecta directamente con 26/02 — Billing Business Model.

23. Product Pricing Matrix

Ejemplo:

Product	Model	Unit	Currency	Interval
Analytics	Flat	Plan	USD	Monthly
Mobility	Usage	Event	USD	Monthly
AI	Usage	Token	USD	Monthly
Storage	Usage	GB	USD	Monthly
Enterprise	Contract	Contract	USD	Annual
24. Usage-Based Products

Un producto puede depender de un Meter.

Product
   │
   ▼
Usage Meter
   │
   ▼
Pricing
   │
   ▼
Charge

Ejemplo:

AI Product
   │
   ├── Input Tokens
   ├── Output Tokens
   └── Requests

Cada métrica puede tener pricing diferente.

25. Billing Meters

El catálogo puede registrar los meters que utiliza un producto.

Product
│
└── Meters
    ├── API_CALL
    ├── STORAGE_GB
    ├── AI_TOKEN
    └── REPORT_GENERATED

Un meter define:

metric;
unit;
aggregation;
reset period;
pricing relationship.
26. Product Limits

Los planes pueden incluir límites:

Professional
│
├── Users = 25
├── Storage = 100 GB
├── API Calls = 1M
├── Reports = 500
└── AI Tokens = 10M

Estos límites pueden generar:

entitlement;
warning;
overage;
blocking;
additional charges.
27. Overage

Cuando el consumo supera un límite:

Included Usage
     +
Additional Usage
     ↓
Overage
     ↓
Charge

Ejemplo:

Included = 1,000,000 events


Actual = 1,250,000


Overage = 250,000

El catálogo debe definir cómo se monetiza el exceso.

28. Product Eligibility

No todos los productos están disponibles para todos los clientes.

Product
   │
   ▼
Eligibility Rules
   │
   ├── Region
   ├── Customer Segment
   ├── Tenant Type
   ├── Contract
   └── Subscription

Ejemplo:

AI Enterprise
→ Only Enterprise Customers
29. Regional Catalog

El catálogo puede variar por región.

Product
│
├── Chile
│   └── CLP Pricing
│
├── Mexico
│   └── MXN Pricing
│
└── USA
    └── USD Pricing

La lógica regional debe estar versionada.

30. Multi-Currency Pricing

Un producto puede tener diferentes precios:

Product
│
├── USD → $500
├── EUR → €470
├── CLP → $450,000
└── MXN → $9,000

No siempre debe calcularse mediante una simple conversión automática.

Puede existir pricing comercial específico por moneda.

31. Price Effective Dates

Cada precio debe tener vigencia.

Price v1
01/01/2026 → 30/06/2026


Price v2
01/07/2026 → Present

Esto permite reconstruir qué precio debía aplicarse en cualquier fecha.

32. Catalog Publishing

El catálogo puede tener workflow:

Draft
  │
  ▼
Review
  │
  ▼
Approved
  │
  ▼
Published
  │
  ▼
Active

Los cambios financieros importantes no deberían publicarse sin control.

33. Catalog Governance

Debe existir control sobre:

creación de productos;
modificación de precios;
eliminación;
publicación;
descuentos;
bundles;
tax categories;
pricing rules.

Ejemplo:

Price Change
    │
    ▼
Commercial Review
    │
    ▼
Finance Review
    │
    ▼
Approval
    │
    ▼
Publish
34. Product Ownership

Cada producto debe tener propietario.

Product
│
├── Product Owner
├── Business Owner
├── Finance Owner
├── Technical Owner
└── Compliance Owner

Esto facilita governance Enterprise.

35. Product Metadata

El catálogo debe soportar metadata extensible:

Product
│
├── Standard Attributes
└── Metadata
      │
      ├── Industry
      ├── Region
      ├── Business Unit
      ├── Tax Category
      └── External IDs

Esto permite integraciones sin alterar continuamente el modelo principal.

36. Product External Integrations

El catálogo puede integrarse con:

Billing Catalog
      │
 ┌────┼───────────┐
 ▼    ▼           ▼
CRM  ERP      Marketplace
      │
      ▼
Accounting

También:

CRM;
CPQ;
ERP;
e-commerce;
payment systems;
entitlement systems.
37. Catalog API

APIs principales:

POST   /billing/catalog/products
GET    /billing/catalog/products
GET    /billing/catalog/products/{id}
PATCH  /billing/catalog/products/{id}

Plans:

POST /billing/catalog/products/{id}/plans
GET  /billing/catalog/products/{id}/plans

Prices:

POST /billing/catalog/prices
GET  /billing/catalog/prices
GET  /billing/catalog/prices/{id}
38. Catalog Search

El catálogo debe permitir búsquedas:

Search
│
├── Product
├── SKU
├── Category
├── Plan
├── Feature
├── Price
└── Status

Ejemplo:

GET /billing/catalog/products?
category=AI&
status=ACTIVE
39. Catalog Data Model

Modelo conceptual:

Catalog
│
├── id
├── organization_id
├── name
├── status
├── currency_policy
├── region_policy
└── created_at

Products:

Product
│
├── id
├── catalog_id
├── product_code
├── sku
├── name
├── description
├── product_type
├── category_id
├── status
├── version
├── effective_from
├── effective_to
└── metadata
40. Product Relationships
Catalog
   │
   └── Product
         │
         ├── Features
         ├── Plans
         ├── Add-ons
         ├── Bundles
         ├── Meters
         └── Prices

Esto proporciona una estructura flexible.

41. Plan Data Model
Plan
│
├── id
├── product_id
├── plan_code
├── name
├── version
├── status
├── billing_interval
├── effective_from
├── effective_to
└── metadata

Relaciones:

Plan
│
├── Features
├── Entitlements
├── Limits
├── Prices
├── Meters
└── Add-ons
42. Price Data Model
Price
│
├── id
├── product_id
├── plan_id
├── pricing_model
├── amount
├── currency
├── interval
├── unit
├── meter_id
├── effective_from
├── effective_to
├── version
└── metadata

Esto permite pricing flexible sin acoplarlo directamente a Product.

43. Catalog Versioning

Todo el catálogo debe poder reconstruirse históricamente.

Catalog Version 1
      │
      ▼
Catalog Version 2
      │
      ▼
Catalog Version 3

La plataforma debe poder responder:

¿Qué producto, plan y precio estaba vigente cuando se creó esta subscription?

La respuesta debe ser determinística.

44. Catalog Snapshot

Al crear una subscription, puede conservarse un snapshot de los elementos relevantes:

Subscription
│
├── Product Version
├── Plan Version
├── Price Version
├── Pricing Rules
└── Entitlements

Esto evita depender exclusivamente del catálogo actual.

45. Product-to-Billing Flow

El flujo completo:

Product Created
      │
      ▼
Plan Created
      │
      ▼
Price Created
      │
      ▼
Catalog Published
      │
      ▼
Customer Selects Product
      │
      ▼
Subscription Created
      │
      ▼
Usage / Recurring Charge
      │
      ▼
Invoice
      │
      ▼
Payment
46. Product-to-Revenue Traceability

La arquitectura debe permitir:

Revenue
   ↑
Invoice
   ↑
Charge
   ↑
Subscription / Usage
   ↑
Price
   ↑
Plan
   ↑
Product

Así puede responderse:

¿Cuánto revenue genera cada producto?

Y también:

¿Qué clientes están generando revenue a partir de determinado producto?

47. Product Analytics

El catálogo debe alimentar:

Product Revenue;
Product MRR;
Product ARR;
Product Adoption;
Product Usage;
Product Churn;
Product Expansion;
Product Margin.

Ejemplo:

Product
   │
   ├── Customers
   ├── Revenue
   ├── Usage
   ├── Cost
   └── Margin
48. Product Profitability

El análisis puede ser:

Product Revenue
      -
Infrastructure Cost
      -
AI Cost
      -
Support Cost
      -
Payment Cost
      =
Product Margin

Esto conecta Catalog con FinOps.

49. AI Products

Para productos de IA, el catálogo debe soportar componentes específicos:

AI Product
│
├── Model
├── Input Tokens
├── Output Tokens
├── Requests
├── Compute
├── Storage
└── Agent Executions

Puede existir pricing:

Input Token     → $X / 1M
Output Token    → $Y / 1M
Request         → $Z / 1K
Agent Execution → $N
50. AI-Powered Catalog

La IA también puede ayudar a administrar el catálogo.

Catalog Data
     │
     ▼
AI
     │
 ┌───┼──────────────┐
 ▼   ▼              ▼
Pricing Analysis  Demand  Product Insights

Casos:

recomendación de precios;
detección de productos poco utilizados;
análisis de packaging;
oportunidades de bundling;
forecast de demanda;
análisis de rentabilidad.

Las modificaciones críticas deben continuar sujetas a governance.

51. Catalog Architecture

La arquitectura completa:

                         BILLING CATALOG
                                │
          ┌─────────────────────┼─────────────────────┐
          │                     │                     │
       Products               Plans                Categories
          │                     │
          ├──────────┬──────────┤
          ▼          ▼          ▼
       Features   Add-ons     Bundles
          │          │          │
          └──────────┼──────────┘
                     ▼
                  Pricing
                     │
              ┌──────┼───────┐
              ▼      ▼       ▼
            Flat   Usage    Hybrid
                     │
                     ▼
                   Meters
                     │
                     ▼
                Subscriptions
                     │
                     ▼
                  Charges
                     │
                     ▼
                  Invoices
                     │
                     ▼
                  Revenue
52. Enterprise Catalog Model

Para EVOXA:

                         EVOXA CATALOG
                              │
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
       Platform              Data                 AI
          │                   │                   │
     ┌────┼────┐         ┌────┼────┐         ┌───┼────┐
     ▼    ▼    ▼         ▼    ▼    ▼         ▼   ▼    ▼
 Analytics OOH Mobility  Audience Mobility  AI  Agents Models
     │
     ├── Starter
     ├── Professional
     ├── Business
     └── Enterprise
             │
             ▼
           Prices
             │
             ▼
        Subscriptions
             │
             ▼
           Billing

Esto permite que EVOXA tenga un catálogo modular y escalable.

53. Catalog Governance

Los principios de gobierno son:

Product Governance

Cada producto debe tener propietario.

Pricing Governance

Los precios deben tener aprobación.

Version Governance

Los cambios deben ser versionados.

Lifecycle Governance

Los productos no deben desaparecer arbitrariamente.

Tax Governance

Cada producto debe poder tener clasificación fiscal.

Access Governance

No todos los usuarios deben poder modificar precios.

Audit Governance

Todos los cambios importantes deben quedar registrados.

54. Core Principles
1. Product Separation

Product y Price son entidades independientes.

2. Versioning

Productos, planes y precios deben poder versionarse.

3. Reusability

Un producto puede tener múltiples ofertas.

4. Flexibility

Debe soportar diferentes modelos de monetización.

5. Historical Integrity

Los cambios futuros no deben modificar el pasado.

6. Multi-Tenant

El catálogo puede ser global o específico por organización/tenant.

7. Extensibility

Debe soportar nuevos productos y pricing models.

8. Governance

Los cambios comerciales deben estar controlados.

9. Traceability

Product → Plan → Price → Subscription → Charge → Invoice → Revenue.

10. AI-Ready

El catálogo debe alimentar pricing intelligence y product intelligence.

55. Resultado de 26/08

Billing Products & Catalog establece la capa comercial que define qué se vende y cómo puede ser empaquetado y monetizado.

La estructura principal queda:

Catalog
   ↓
Product
   ↓
Feature
   ↓
Plan
   ↓
Price
   ↓
Meter
   ↓
Subscription
   ↓
Usage
   ↓
Charge
   ↓
Invoice
   ↓
Payment
   ↓
Revenue

Y la regla fundamental es:

El Billing Catalog debe actuar como una fuente de verdad comercial versionada y gobernada, separando productos, planes, features, entitlements, meters y precios, permitiendo que la plataforma evolucione sus modelos de monetización sin comprometer la integridad de las operaciones financieras históricas.

Con esto queda preparada la siguiente capa:

26/09 — Billing Pricing & Plans
