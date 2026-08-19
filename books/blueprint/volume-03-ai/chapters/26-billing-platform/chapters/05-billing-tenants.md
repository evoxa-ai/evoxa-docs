1. Billing Tenants

Billing Tenants define cómo la plataforma administra la dimensión financiera de cada tenant dentro de una arquitectura Enterprise multi-tenant.

El tenant representa una unidad lógica de aislamiento y operación dentro de la plataforma. En Billing, además, debe convertirse en un contexto financiero explícito, capaz de determinar:

qué productos utiliza;
qué servicios consume;
qué contratos mantiene;
qué pricing se aplica;
qué consumo genera;
qué cargos produce;
qué cuenta de facturación utiliza;
cómo se factura;
quién paga;
qué moneda utiliza;
qué impuestos corresponden;
qué créditos posee;
qué saldo mantiene;
qué ingresos genera.

La arquitectura debe permitir que un mismo Enterprise administre cientos, miles o millones de tenants sin perder aislamiento ni trazabilidad financiera.

2. Tenant dentro del Billing Domain

La relación puede representarse:

Enterprise
    │
    ▼
Organization
    │
    ▼
Billing Organization
    │
    ▼
Tenant
    │
    ├── Billing Configuration
    ├── Billing Account
    ├── Customers
    ├── Subscriptions
    ├── Usage
    ├── Charges
    ├── Invoices
    ├── Payments
    ├── Credits
    └── Financial Analytics

El tenant se convierte así en una de las principales dimensiones de aislamiento financiero.

3. Billing Tenant vs Operational Tenant

No siempre deben considerarse idénticos.

Un tenant operativo representa:

Dónde y cómo funciona un servicio.

Un Billing Tenant representa:

Cómo se administra económicamente ese servicio.

Puede existir una relación:

Operational Tenant
        │
        ▼
Billing Tenant

Pero también pueden existir escenarios donde:

Operational Tenant A ─┐
Operational Tenant B ─┼──→ Billing Account
Operational Tenant C ─┘

Esto es fundamental para soportar consolidated billing.

4. Objetivos

Billing Tenants debe proporcionar:

aislamiento financiero;
configuración específica;
trazabilidad por tenant;
billing independiente;
pricing específico;
usage tracking;
invoice allocation;
payment association;
credit management;
financial reporting;
cost allocation;
governance.
5. Tenant Financial Context

Cada operación de Billing debe poder determinar el tenant al que pertenece.

Billing Event
     │
     ▼
Tenant Context
     │
     ├── Organization
     ├── Billing Account
     ├── Currency
     ├── Pricing
     ├── Tax
     └── Policies

Esto significa que un evento financiero no debería existir sin contexto suficiente.

6. Tenant Billing Identity

Cada tenant debe tener una identidad financiera estable.

Conceptualmente:

BillingTenant
│
├── tenant_id
├── organization_id
├── billing_account_id
├── legal_entity_id
├── currency
├── timezone
├── billing_status
├── pricing_profile_id
├── tax_profile_id
├── credit_policy_id
└── configuration_id

El tenant_id debe formar parte de las relaciones relevantes del dominio.

7. Tenant Isolation

La regla fundamental es:

Un tenant nunca debe poder acceder accidentalmente a información financiera de otro tenant.

Arquitectura:

Tenant A
│
├── Customers
├── Subscriptions
├── Usage
├── Charges
├── Invoices
└── Payments


Tenant B
│
├── Customers
├── Subscriptions
├── Usage
├── Charges
├── Invoices
└── Payments

El aislamiento debe existir en:

API;
aplicación;
base de datos;
cache;
eventos;
almacenamiento;
analytics;
logs;
exports.
8. Tenant Isolation Model

Existen diferentes estrategias.

Shared Database / Shared Schema
billing_transactions
│
├── tenant_id = A
├── tenant_id = B
└── tenant_id = C

Ventajas:

menor costo;
operación sencilla;
buena escalabilidad inicial.

Requiere controles estrictos de aislamiento.

Shared Database / Separate Schema
database
│
├── tenant_a
├── tenant_b
└── tenant_c

Ofrece mayor aislamiento lógico.

Database per Tenant
Tenant A → DB A
Tenant B → DB B
Tenant C → DB C

Proporciona mayor aislamiento, pero aumenta:

complejidad;
costo;
operación;
backups;
migrations.
9. Recommended Enterprise Model

Para una plataforma como EVOXA puede utilizarse una estrategia híbrida:

Standard Tenants
       │
       ▼
Shared Infrastructure
       │
       ▼
Logical Isolation


Enterprise / Regulated Tenants
       │
       ▼
Dedicated Resources
       │
       ▼
Enhanced Isolation

Esto permite adaptar la arquitectura al nivel de riesgo y requisitos del cliente.

10. Tenant Billing Configuration

Cada tenant puede tener configuración específica:

Tenant
│
├── Billing Cycle
├── Currency
├── Timezone
├── Payment Terms
├── Tax Profile
├── Pricing Profile
├── Credit Policy
├── Invoice Settings
├── Collection Policy
└── Notification Settings

Debe existir una jerarquía de configuración.

11. Configuration Inheritance

Las configuraciones pueden heredarse:

Enterprise
   │
   ▼
Organization
   │
   ▼
Billing Organization
   │
   ▼
Tenant
   │
   ▼
Billing Account

Ejemplo:

Enterprise Currency = USD
        ↓
Organization Currency = USD
        ↓
Tenant Currency = CLP

El tenant puede utilizar CLP si la política Enterprise lo permite.

12. Tenant Billing Account

El tenant normalmente estará asociado a uno o más Billing Accounts.

Tenant
   │
   ├── Primary Billing Account
   │
   ├── Secondary Billing Account
   │
   └── Consolidated Billing Account

Esto permite escenarios como:

Tenant A
   │
   ├── Production Account
   └── Development Account
13. Tenant Customers

Un tenant puede administrar múltiples clientes.

Tenant
│
├── Customer A
├── Customer B
├── Customer C
└── Customer D

Cada cliente puede tener:

subscriptions;
invoices;
payment methods;
credits;
balances.
14. Tenant Subscription Model
Tenant
   │
   ├── Subscription A
   ├── Subscription B
   └── Subscription C

Cada subscription debe estar asociada a:

tenant;
customer;
product;
plan;
price;
billing account;
billing cycle.
15. Tenant Usage

El tenant puede producir grandes cantidades de eventos.

Tenant
   │
   ▼
Usage Events
   │
   ├── API Calls
   ├── Storage
   ├── AI Tokens
   ├── Transactions
   └── Data Processing

Cada evento debe mantener:

tenant_id
event_id
metric
quantity
timestamp
source
16. Tenant Usage Isolation

El sistema debe evitar errores como:

Tenant A Usage
      +
Tenant B Usage
      =
Tenant A Charge

El pipeline correcto es:

Usage Event
    │
    ▼
Tenant Context
    │
    ▼
Meter
    │
    ▼
Rating
    │
    ▼
Tenant Charge

El tenant_id debe mantenerse durante todo el procesamiento.

17. Tenant Pricing

Un tenant puede utilizar pricing:

Global
Enterprise Price
       ↓
Tenant
Organization-specific
Organization Price
       ↓
Tenant
Tenant-specific
Custom Price
       ↓
Tenant
Contract-based
Contract
       ↓
Tenant

La precedencia debe estar definida.

18. Pricing Precedence

Una posible jerarquía:

Global Price
      ↓
Organization Price
      ↓
Tenant Price
      ↓
Contract Price
      ↓
Subscription Override

La regla más específica tiene precedencia cuando la gobernanza lo permite.

Esto evita que una modificación general destruya acuerdos personalizados.

19. Tenant Billing Lifecycle

El tenant financiero puede seguir:

PROVISIONING
     │
     ▼
PENDING_BILLING
     │
     ▼
ACTIVE
     │
     ├── PAST_DUE
     │      │
     │      ▼
     │   RECOVERY
     │      │
     │      ▼
     │    ACTIVE
     │
     ├── SUSPENDED
     │
     └── TERMINATING
             │
             ▼
          CLOSED

Los estados deben estar relacionados con el estado operativo, pero no necesariamente ser idénticos.

20. Operational vs Billing Status

Un tenant puede estar:

Operational Status = ACTIVE
Billing Status     = PAST_DUE

o:

Operational Status = SUSPENDED
Billing Status     = ACTIVE

Por lo tanto, deben mantenerse como dimensiones separadas.

21. Tenant Billing Events

Eventos importantes:

BillingTenantCreated
BillingTenantActivated
BillingConfigurationChanged
BillingAccountAssigned
PricingAssigned
UsageRecorded
ChargeCreated
InvoiceGenerated
PaymentReceived
PaymentFailed
CreditApplied
TenantPastDue
TenantSuspended
BillingTenantClosed

Todos deben contener contexto suficiente para auditoría.

22. Tenant Financial State

El estado financiero puede representarse:

Tenant Financial State
│
├── Current Balance
├── Credit Balance
├── Outstanding Balance
├── Overdue Balance
├── Monthly Recurring Revenue
├── Annual Recurring Revenue
├── Usage Revenue
└── Lifetime Revenue

Esto permite construir un Tenant Financial Profile.

23. Tenant Financial Profile

Conceptualmente:

Tenant
│
├── Commercial Profile
│
├── Billing Profile
│
├── Payment Profile
│
├── Credit Profile
│
├── Usage Profile
│
└── Revenue Profile

Puede utilizarse para decisiones comerciales y financieras.

24. Tenant Credit Management

Cada tenant puede tener una política de crédito.

Tenant
   │
   ▼
Credit Policy
   │
   ├── Credit Limit
   ├── Payment Terms
   ├── Grace Period
   ├── Risk Level
   └── Collection Policy

Ejemplo:

Credit Limit = $25,000
Terms = Net 30
Risk = Medium
25. Tenant Collections

El estado de cobranza debe poder determinarse individualmente.

Tenant
   │
   ▼
Invoice
   │
   ▼
Due Date
   │
   ▼
Past Due
   │
   ▼
Dunning
   │
   ▼
Collections

Esto permite que un tenant sea tratado de manera diferente a otro.

26. Tenant Invoicing

El modelo puede ser:

Tenant-specific invoice
Tenant A
   ↓
Invoice A
Customer-specific
Tenant
   ↓
Customer
   ↓
Invoice
Consolidated
Tenant A ─┐
Tenant B ─┼──→ Consolidated Invoice
Tenant C ─┘

La arquitectura debe soportar las tres modalidades.

27. Tenant Payment Configuration

Puede definirse:

Tenant
│
├── Default Payment Method
├── Payment Provider
├── Payment Terms
├── Currency
├── Auto-Pay
└── Retry Policy

En Enterprise, estas configuraciones pueden ser heredadas desde la organización.

28. Tenant Credits

Los créditos pueden asignarse a un tenant:

Credit Grant
      │
      ▼
Tenant Credit Wallet
      │
      ▼
Credit Usage

Ejemplo:

Granted = $5,000
Used    = $2,500
Remaining = $2,500

Debe mantenerse el historial completo.

29. Tenant Cost Allocation

Billing también debe permitir conocer cuánto cuesta operar cada tenant.

Tenant Revenue
       │
       -
       │
Tenant Cost
       │
       ▼
Tenant Margin

Costos:

Cloud
AI
Storage
Data
Payments
Support
Infrastructure

Esto permite conocer el cost-to-serve.

30. Tenant Profitability

El resultado puede calcularse:

Revenue
   -
Direct Cost
   -
Payment Cost
   -
AI Cost
   -
Infrastructure Cost
   =
Contribution Margin

Ejemplo conceptual:

Revenue              $10,000
Cloud Cost             $900
AI Cost                $600
Payment Cost           $300
Support Cost           $400
────────────────────────────
Contribution Margin   $7,800
31. Tenant Analytics

Los principales KPIs:

Revenue
MRR;
ARR;
total revenue;
net revenue.
Usage
usage volume;
billable usage;
overage;
utilization.
Payments
payment success rate;
failed payments;
overdue balance.
Customer
ARPU;
LTV;
churn;
expansion.
Profitability
cost-to-serve;
gross margin;
contribution margin.
32. Tenant Data Warehouse Model

El tenant debe ser una dimensión central:

FactCharges
      │
      ├── tenant_id
      ├── customer_id
      ├── product_id
      ├── subscription_id
      ├── date_id
      └── amount

También:

FactUsage
FactInvoices
FactPayments
FactRevenue
FactCosts

Esto permite análisis financieros multidimensionales.

33. Tenant Billing APIs

APIs conceptuales:

POST   /billing/tenants
GET    /billing/tenants
GET    /billing/tenants/{id}
PATCH  /billing/tenants/{id}

Configuration:

GET   /billing/tenants/{id}/configuration
PATCH /billing/tenants/{id}/configuration

Financial:

GET /billing/tenants/{id}/accounts
GET /billing/tenants/{id}/subscriptions
GET /billing/tenants/{id}/invoices
GET /billing/tenants/{id}/payments
GET /billing/tenants/{id}/credits
GET /billing/tenants/{id}/balance
GET /billing/tenants/{id}/usage
GET /billing/tenants/{id}/revenue
34. Tenant Security Model

El acceso debe aplicar:

User
 │
 ▼
Organization
 │
 ▼
Tenant
 │
 ▼
Billing Permission
 │
 ▼
Resource

Un usuario puede tener:

Tenant A → Full Access
Tenant B → Read Only
Tenant C → No Access

Esto permite administración granular.

35. Tenant-Level Audit

Las acciones deben registrarse:

Tenant
 │
 ▼
Billing Action
 │
 ▼
Audit Event

Ejemplos:

PriceOverrideCreated
CreditGranted
CreditRevoked
PaymentMethodChanged
BillingAccountChanged
InvoiceAdjusted
RefundIssued
SubscriptionChanged
36. Tenant Billing Governance

Las acciones de mayor impacto pueden requerir aprobación.

Ejemplo:

Credit > $5,000
       │
       ▼
Approval Required
       │
       ▼
Finance Manager

Otro ejemplo:

Price Override
       │
       ▼
Commercial Approval
       │
       ▼
Finance Approval

Esto permite controlar descuentos y excepciones.

37. Tenant-Level AI

La IA puede crear un perfil inteligente del tenant.

Tenant Data
   │
   ├── Usage
   ├── Billing
   ├── Payments
   ├── Contracts
   └── Behavior
          │
          ▼
       AI Layer
          │
    ┌─────┼─────────────┐
    ▼     ▼             ▼
Revenue  Risk        Expansion
Forecast Prediction Opportunity

Puede detectar:

riesgo de churn;
riesgo de morosidad;
oportunidades de upselling;
anomalías de consumo;
anomalías de facturación;
incremento inesperado de costos.
38. Tenant Billing Anomaly Detection

Ejemplo:

Normal Usage
     │
     ▼
10,000 events/day

De repente:

Detected
250,000 events/day

La plataforma puede generar:

Usage Anomaly Detected
       │
       ├── Tenant
       ├── Metric
       ├── Expected Value
       ├── Actual Value
       └── Risk Level

Esto puede prevenir:

errores de integración;
abuso;
sobrecostos;
facturación inesperada.
39. Tenant Architecture

La arquitectura conceptual:

                     BILLING ORGANIZATION
                             │
                             ▼
                           TENANT
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
      Identity          Configuration        Policies
         │                   │                   │
         └───────────────────┼───────────────────┘
                             │
             ┌───────────────┼───────────────┐
             ▼               ▼               ▼
         Customers     Subscriptions       Usage
             │               │               │
             └───────────────┼───────────────┘
                             ▼
                          Charges
                             │
                             ▼
                          Invoices
                             │
                             ▼
                          Payments
                             │
                  ┌──────────┴──────────┐
                  ▼                     ▼
               Revenue               Costs
                  │                     │
                  └──────────┬──────────┘
                             ▼
                       Profitability
40. Tenant Data Model

Modelo conceptual:

BillingTenant
│
├── id
├── organization_id
├── billing_organization_id
├── legal_entity_id
├── billing_account_id
├── status
├── currency
├── timezone
├── pricing_profile_id
├── tax_profile_id
├── credit_policy_id
├── billing_cycle
├── payment_terms
├── created_at
└── updated_at

Relaciones:

BillingTenant
│
├── BillingAccounts
├── Customers
├── Subscriptions
├── Contracts
├── UsageEvents
├── Charges
├── Invoices
├── Payments
├── Credits
├── Refunds
└── FinancialMetrics
41. Enterprise Tenant Model

Para EVOXA:

                         EVOXA
                           │
                    Billing Organization
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
     Tenant A           Tenant B           Tenant C
        │                  │                  │
        ├── Customers      ├── Customers      ├── Customers
        ├── Usage          ├── Usage          ├── Usage
        ├── Subscriptions  ├── Subscriptions  ├── Subscriptions
        ├── Charges        ├── Charges        ├── Charges
        └── Invoice        └── Invoice        └── Invoice

Y puede existir:

Tenant A ─┐
Tenant B ─┼──→ Corporate Billing Account
Tenant C ─┘

para clientes Enterprise con facturación consolidada.

42. Tenant Billing Principles
1. Tenant Isolation

Cada tenant debe permanecer financieramente aislado.

2. Financial Context

Toda transacción debe mantener contexto del tenant.

3. Configuration Inheritance

Las configuraciones pueden heredarse desde niveles superiores.

4. Tenant Overrides

Deben existir excepciones controladas.

5. Billing Independence

El estado financiero debe poder administrarse independientemente del estado operativo.

6. Scalability

El modelo debe funcionar desde pocos hasta millones de tenants.

7. Auditability

Toda operación debe ser trazable.

8. Consolidation

Debe ser posible agrupar múltiples tenants.

9. Cost Attribution

Los costos deben poder asignarse al tenant.

10. AI Intelligence

El tenant debe ser una dimensión fundamental para revenue, risk y profitability intelligence.

43. Resultado de 26/05

Billing Tenants establece al tenant como una unidad financiera y de aislamiento dentro de Billing, sin perder su relación con la estructura Enterprise superior.

La jerarquía queda:

Enterprise
   ↓
Organization
   ↓
Billing Organization
   ↓
Tenant
   ↓
Billing Account
   ↓
Customer
   ↓
Subscription / Contract
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
   ↓
Profitability

El principio fundamental es:

Cada tenant debe poseer un contexto financiero claramente identificable, aislado, configurable, auditable y escalable, permitiendo billing independiente o consolidado según las necesidades de la organización.

Con esto, la estructura organizacional y multi-tenant de Billing queda establecida. El siguiente nivel es:

26/06 — Billing Accounts
