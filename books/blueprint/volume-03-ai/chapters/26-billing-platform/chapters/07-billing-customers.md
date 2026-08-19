1. Billing Customers

Billing Customers define la entidad comercial que representa a la persona, empresa u organización que mantiene una relación económica con la plataforma y que puede consumir productos, contratar servicios, mantener suscripciones, recibir facturas y realizar pagos.

Dentro de la arquitectura Billing, el Customer no debe confundirse con:

User;
Tenant;
Organization;
Billing Organization;
Billing Account;
Legal Entity.

Cada uno representa una dimensión diferente.

El Customer representa principalmente la contraparte comercial de la relación de monetización.

2. Customer dentro de Billing

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
   ▼
Customer
   │
   ▼
Billing Account
   │
   ├── Subscriptions
   ├── Contracts
   ├── Charges
   ├── Invoices
   ├── Payments
   ├── Credits
   └── Refunds

Sin embargo, la arquitectura debe permitir que un Customer pueda estar asociado a múltiples cuentas de Billing cuando el modelo empresarial lo requiera.

3. Propósito de Billing Customers

El dominio Customer debe permitir:

identificar al cliente;
administrar su perfil comercial;
administrar información de facturación;
asociar cuentas financieras;
gestionar contratos;
administrar subscriptions;
mantener payment methods;
gestionar tax information;
mantener historial financiero;
controlar lifecycle;
analizar revenue;
evaluar riesgo;
gestionar comunicación financiera.
4. Customer vs User

Una de las separaciones más importantes es:

User
  │
  │ utiliza la plataforma
  ▼
Application

Mientras:

Customer
  │
  │ compra / paga
  ▼
Billing

Por ejemplo:

Customer
   │
   ├── 500 Users
   │
   ├── 10 Subscriptions
   │
   └── 1 Billing Account

Un Customer puede tener cientos o miles de usuarios.

5. Customer vs Tenant

Tampoco son necesariamente la misma entidad.

Tenant
   │
   ├── Users
   ├── Data
   ├── Configuration
   └── Customer

En un SaaS sencillo:

Tenant = Customer

Pero en Enterprise:

Customer Corporation
        │
        ├── Tenant Chile
        ├── Tenant Mexico
        ├── Tenant USA
        └── Tenant Europe

Un Customer puede administrar múltiples tenants.

6. Customer vs Billing Account

El Customer representa la contraparte comercial.

El Billing Account representa el contexto financiero.

Customer
   │
   ├── Billing Account A
   ├── Billing Account B
   └── Billing Account C

Ejemplo:

ACME Corporation
│
├── Chile Billing Account
├── Mexico Billing Account
└── Global Corporate Billing Account

Esto permite estructuras multinacionales.

7. Customer Types

Billing puede soportar diferentes tipos de clientes.

Individual

Persona natural.

Business

Empresa.

Enterprise

Gran organización con estructura compleja.

Government

Entidad pública.

Non-Profit

Organización sin fines de lucro.

Partner

Socio comercial.

Marketplace Customer

Cliente de un marketplace.

Internal Customer

Unidad interna utilizada para operaciones internas.

8. Customer Classification

Además del tipo, puede existir una clasificación comercial:

Customer
│
├── Segment
├── Industry
├── Size
├── Region
├── Revenue Tier
├── Risk Level
└── Lifecycle Stage

Ejemplo:

Segment = Enterprise
Industry = Telecommunications
Region = LATAM
Size = Large

Estas dimensiones alimentan Pricing, Sales y Revenue Intelligence.

9. Customer Identity

Cada Customer debe poseer una identidad estable.

Customer
│
├── customer_id
├── external_customer_id
├── customer_number
├── type
├── status
├── legal_name
├── display_name
└── created_at

Debe existir separación entre:

internal ID;
external ID;
customer number.
10. Customer Lifecycle

El ciclo de vida puede ser:

PROSPECT
   │
   ▼
ONBOARDING
   │
   ▼
ACTIVE
   │
   ├── PAST_DUE
   │
   ├── SUSPENDED
   │
   └── INACTIVE
          │
          ▼
        CLOSED

El estado comercial y el estado financiero deben poder coexistir.

Por ejemplo:

Customer Status = ACTIVE
Billing Status  = PAST_DUE
11. Customer Onboarding

El proceso puede ser:

Customer Created
      │
      ▼
Identity Verification
      │
      ▼
Business Information
      │
      ▼
Billing Profile
      │
      ▼
Tax Profile
      │
      ▼
Billing Account
      │
      ▼
Payment Method
      │
      ▼
Customer Activated

Para clientes Enterprise pueden existir procesos de aprobación adicionales.

12. Customer Profile

El perfil comercial puede incluir:

Customer
│
├── Identity
├── Contact Information
├── Business Information
├── Billing Information
├── Tax Information
├── Payment Information
├── Contract Information
└── Preferences

Debe evitarse duplicar información innecesariamente entre Customer y Billing Account.

13. Customer Legal Identity

Para clientes empresariales:

Customer
   │
   ▼
Legal Entity
   │
   ├── Legal Name
   ├── Registration Number
   ├── Tax ID
   ├── Country
   └── Legal Address

Una empresa puede tener varias entidades legales.

ACME Group
│
├── ACME Chile SpA
├── ACME Mexico S.A.
└── ACME USA Inc.

Por lo tanto, Customer puede representar el grupo comercial mientras las entidades legales representan las estructuras jurídicas.

14. Customer Addresses

El Customer puede tener:

Customer
│
├── Legal Address
├── Billing Address
├── Tax Address
├── Service Address
└── Mailing Address

Las direcciones deben tener propósito explícito.

No se debe asumir:

legal_address = billing_address
15. Customer Contacts

Un cliente Enterprise puede tener múltiples contactos:

Customer
│
├── Executive Contact
├── Finance Contact
├── Billing Contact
├── Procurement Contact
├── Technical Contact
├── Legal Contact
└── Security Contact

Cada contacto puede tener permisos y canales de comunicación diferentes.

16. Customer Billing Profile

El Billing Profile contiene la información necesaria para facturar.

Billing Profile
│
├── Billing Name
├── Billing Address
├── Currency
├── Timezone
├── Payment Terms
├── Invoice Delivery
├── Tax Profile
└── Invoice Preferences

Puede existir un Billing Profile por cuenta.

17. Customer Tax Profile

La información fiscal puede incluir:

Tax Profile
│
├── Tax ID
├── Tax Country
├── Tax Jurisdiction
├── Tax Category
├── Tax Exemption
└── Tax Certificates

El Tax Engine utilizará esta información junto con:

ubicación;
producto;
tipo de transacción;
entidad legal;
jurisdicción.
18. Customer Payment Profile

El cliente puede tener múltiples métodos de pago.

Customer
│
└── Payment Profile
      │
      ├── Card
      ├── Bank Account
      ├── Wire Transfer
      └── Other Methods

Los datos sensibles deben permanecer tokenizados cuando sea posible.

Billing debería almacenar referencias al proveedor y no información sensible innecesaria.

19. Customer Payment Preferences

Puede configurarse:

Payment Preferences
│
├── Auto-Pay
├── Preferred Method
├── Backup Method
├── Payment Currency
├── Payment Terms
└── Retry Preferences

Ejemplo:

Auto-Pay = Enabled
Primary = Card
Backup = Bank Account
Terms = Net 30
20. Customer Billing Accounts

Un Customer puede tener múltiples Billing Accounts:

Customer
│
├── Billing Account A
│     └── Chile Operations
│
├── Billing Account B
│     └── Mexico Operations
│
└── Billing Account C
      └── Corporate

Esto permite separar:

monedas;
entidades legales;
presupuestos;
contratos;
departamentos;
regiones.
21. Customer Contracts

Los contratos definen condiciones comerciales específicas.

Customer
   │
   ▼
Contract
   │
   ├── Start Date
   ├── End Date
   ├── Commitment
   ├── Pricing
   ├── Discounts
   ├── Payment Terms
   └── Minimum Spend

Un cliente puede tener varios contratos activos para diferentes servicios.

22. Customer Subscriptions

La relación puede ser:

Customer
   │
   ├── Subscription A
   ├── Subscription B
   └── Subscription C

Cada subscription puede estar asociada a:

tenant;
billing account;
product;
plan;
price;
contract.
23. Customer Products

El cliente puede consumir:

Customer
│
├── Product A
├── Product B
├── Add-on C
└── AI Service D

La relación entre Customer y Product puede ocurrir mediante:

subscription;
order;
contract;
usage;
one-time purchase.
24. Customer Usage

El consumo puede estar asociado al cliente:

Customer
   │
   ▼
Usage
   │
   ├── API
   ├── Storage
   ├── AI
   ├── Data
   └── Transactions

Pero siempre debe conservarse también el contexto de:

Organization
Tenant
Billing Account
Subscription
Product
25. Customer Charges

Los cargos se generan a partir de:

Customer
   │
   ├── Subscription
   ├── Usage
   ├── Add-ons
   ├── Overage
   └── Adjustments
          │
          ▼
       Charges

Cada cargo debe mantener una referencia a su origen.

26. Customer Invoices

El historial de facturación:

Customer
   │
   ├── Invoice #1001
   ├── Invoice #1002
   ├── Invoice #1003
   └── Invoice #1004

Cada factura debe permitir conocer:

período;
cargos;
impuestos;
descuentos;
créditos;
total;
pagos;
saldo pendiente.
27. Customer Balance

La posición financiera del cliente puede calcularse:

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
Customer Balance

Pero el saldo agregado no debe reemplazar el ledger transaccional.

28. Customer Credit

Los clientes Enterprise pueden recibir líneas de crédito.

Customer
   │
   ▼
Credit Profile
   │
   ├── Credit Limit
   ├── Current Exposure
   ├── Available Credit
   ├── Risk Score
   └── Payment Terms

Ejemplo:

Credit Limit      $100,000
Exposure           $35,000
Available Credit   $65,000
29. Customer Risk

El perfil de riesgo puede utilizar:

Payment History
       +
Outstanding Balance
       +
Credit Utilization
       +
Failed Payments
       +
Collection History
       ↓
Customer Risk Profile

Categorías:

LOW
MEDIUM
HIGH
CRITICAL

Las políticas financieras pueden utilizar este perfil.

30. Customer Collections

El proceso:

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
   │
   ├── Recovered
   ├── Payment Plan
   └── Write-Off

El Customer debe mantener historial de cobranza.

31. Customer Payment History

Debe ser posible consultar:

Customer Payment History


Invoice    Amount    Date       Status
----------------------------------------
INV-001    $1,000    01/08      Paid
INV-002    $1,200    01/09      Paid
INV-003    $1,500    01/10      Failed
INV-004    $1,500    01/10      Pending

Este historial alimenta Revenue Intelligence y Risk Management.

32. Customer Refunds

Un refund debe mantener relación:

Customer
   │
   ▼
Invoice
   │
   ▼
Payment
   │
   ▼
Refund

Debe existir trazabilidad completa.

33. Customer Adjustments

Los ajustes pueden ser:

credit adjustment;
debit adjustment;
invoice correction;
service compensation;
goodwill credit.

Ejemplo:

Invoice = $1,000


Billing Error = $100


Adjustment = -$100


Corrected Amount = $900

La factura original debe permanecer históricamente disponible.

34. Customer Communication

Billing debe poder comunicarse con el Customer.

Eventos:

Invoice Generated
Payment Received
Payment Failed
Invoice Due
Invoice Overdue
Credit Applied
Refund Issued
Subscription Renewed
Subscription Cancelled

Canales:

Email
Portal
Webhook
API
Notification Center
35. Customer Notification Preferences

El cliente puede configurar:

Notifications
│
├── Invoice
├── Payment
├── Failed Payment
├── Subscription
├── Usage Alert
├── Credit
└── Collection

También puede definir:

idioma;
timezone;
canales;
frecuencia.
36. Customer Segmentation

Billing Customers puede integrarse con Customer Intelligence.

Segmentos:

Enterprise
Mid-Market
SMB
Startup
Government
Partner

También:

High Value
High Growth
At Risk
High Usage
Low Usage
High Margin
Low Margin
37. Customer Revenue Intelligence

El sistema puede calcular:

Customer Revenue
│
├── MRR
├── ARR
├── One-Time Revenue
├── Usage Revenue
├── Expansion Revenue
└── Net Revenue

Esto permite identificar los clientes más valiosos.

38. Customer Lifetime Value

El Customer puede tener métricas:

LTV
ARPU
CAC
Gross Margin
Retention
Churn
Expansion

Una vista conceptual:

Customer
   │
   ├── Revenue
   ├── Cost
   ├── Margin
   └── Lifetime Value
39. Customer Cost-to-Serve

La rentabilidad debe considerar costos.

Customer Revenue
       -
Cloud Cost
       -
AI Cost
       -
Support Cost
       -
Payment Fees
       -
Operational Cost
       =
Customer Contribution Margin

Esto evita evaluar clientes únicamente por revenue.

40. Customer Analytics

KPIs principales:

Commercial
Customer Revenue;
MRR;
ARR;
expansion;
contraction.
Billing
invoices;
average invoice;
collection rate;
overdue balance.
Payment
payment success;
payment failures;
retry rate.
Usage
usage volume;
utilization;
overage.
Profitability
gross margin;
contribution margin;
cost-to-serve.
41. Customer Data Model

Modelo conceptual:

BillingCustomer
│
├── id
├── customer_number
├── external_customer_id
├── organization_id
├── tenant_id
├── customer_type
├── status
├── legal_name
├── display_name
├── country
├── default_currency
├── timezone
├── tax_profile_id
├── billing_profile_id
├── payment_profile_id
├── risk_profile_id
├── created_at
└── updated_at

Relaciones:

BillingCustomer
│
├── LegalEntities
├── Contacts
├── Addresses
├── BillingAccounts
├── Contracts
├── Subscriptions
├── Charges
├── Invoices
├── Payments
├── Credits
├── Refunds
├── Adjustments
└── CustomerMetrics
42. Customer Contact Model

Conceptualmente:

CustomerContact
│
├── id
├── customer_id
├── contact_type
├── name
├── email
├── phone
├── language
├── timezone
├── is_primary
├── notification_preferences
└── status

Esto permite múltiples responsables.

43. Customer Address Model
CustomerAddress
│
├── id
├── customer_id
├── address_type
├── country
├── region
├── city
├── postal_code
├── address_line_1
├── address_line_2
├── is_primary
└── valid_from

Las direcciones deben poder tener vigencia temporal.

44. Customer APIs

APIs principales:

POST   /billing/customers
GET    /billing/customers
GET    /billing/customers/{id}
PATCH  /billing/customers/{id}

Billing:

GET /billing/customers/{id}/accounts
GET /billing/customers/{id}/subscriptions
GET /billing/customers/{id}/contracts
GET /billing/customers/{id}/invoices
GET /billing/customers/{id}/payments
GET /billing/customers/{id}/credits
GET /billing/customers/{id}/balance
GET /billing/customers/{id}/usage

Financial profile:

GET /billing/customers/{id}/revenue
GET /billing/customers/{id}/risk
GET /billing/customers/{id}/profitability
45. Customer Security

El acceso debe respetar:

User
 │
 ▼
Organization
 │
 ▼
Tenant
 │
 ▼
Customer
 │
 ▼
Billing Permission

No todos los usuarios del tenant necesariamente deben tener acceso a información financiera.

Por ejemplo:

Operational User
    → No Financial Access


Billing User
    → Customer Billing Access


Finance Manager
    → Full Financial Access
46. Customer Audit

Acciones críticas:

CustomerCreated
CustomerUpdated
BillingProfileChanged
TaxProfileChanged
PaymentMethodChanged
CreditLimitChanged
CreditGranted
RefundIssued
CustomerSuspended
CustomerClosed

Cada evento debe contener:

actor;
timestamp;
resource;
previous state;
new state;
reason;
correlation ID.

47. Customer AI

La IA puede construir un Customer Financial Intelligence Profile.

Customer Data
│
├── Billing
├── Payments
├── Usage
├── Contracts
├── Support
└── Behavior
       │
       ▼
      AI
       │
 ┌─────┼────────────┐
 ▼     ▼            ▼
Risk  Churn       Expansion

Puede generar:

churn probability;
payment risk;
expansion probability;
revenue forecast;
anomaly detection;
recommended plan.
48. Customer Billing Anomaly Detection

La plataforma puede detectar:

Normal Monthly Billing
$10,000

y posteriormente:

Current Billing
$35,000

El sistema puede generar:

Billing Anomaly
│
├── Customer
├── Expected Amount
├── Actual Amount
├── Variance
├── Possible Cause
└── Risk Level

Esto puede ayudar a detectar:

consumo anormal;
errores de pricing;
duplicación de eventos;
configuración incorrecta;
fraude;
errores de integración.
49. Customer Lifecycle Intelligence

El ciclo comercial puede analizarse:

Prospect
   ↓
Onboarding
   ↓
Activated
   ↓
Growing
   ↓
Mature
   ↓
At Risk
   ↓
Churned

Billing proporciona señales financieras para cada etapa.

50. Customer Architecture

La arquitectura completa:

                         BILLING CUSTOMER
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                 │
           Identity          Profile          Contacts
              │                 │                 │
              └─────────────────┼─────────────────┘
                                │
                    ┌───────────┼───────────┐
                    ▼           ▼           ▼
              Billing Accounts Contracts   Tax
                    │           │
                    └─────┬─────┘
                          ▼
                  Subscriptions
                          │
                          ▼
                        Usage
                          │
                          ▼
                       Charges
                          │
                          ▼
                       Invoices
                          │
                          ▼
                       Payments
                          │
                 ┌────────┴────────┐
                 ▼                 ▼
              Credits           Refunds
                 │                 │
                 └────────┬────────┘
                          ▼
                       Ledger
                          │
              ┌───────────┼───────────┐
              ▼           ▼           ▼
           Revenue      Risk       Profitability
                          │
                          ▼
                         AI
51. Enterprise Customer Model

Para EVOXA, un modelo Enterprise podría ser:

                         CUSTOMER
                            │
                     ACME CORPORATION
                            │
            ┌───────────────┼───────────────┐
            ▼               ▼               ▼
       Legal Entity      Contacts       Contracts
            │
     ┌──────┼───────┐
     ▼      ▼       ▼
  Chile   Mexico    USA
     │      │       │
     ▼      ▼       ▼
  Tenants Tenants Tenants
     │      │       │
     └──────┼───────┘
            ▼
     Billing Accounts
            │
     ┌──────┼─────────┐
     ▼      ▼         ▼
Subscriptions Usage  Payments
     │      │         │
     └──────┼─────────┘
            ▼
         Invoices
            │
            ▼
         Revenue

Este modelo permite que un único Customer corporativo tenga múltiples entidades, tenants y cuentas financieras.

52. Customer Governance

Las operaciones críticas deben estar gobernadas.

Ejemplo:

Customer Credit Limit Increase
             │
             ▼
Risk Evaluation
             │
             ▼
Approval
             │
             ▼
Credit Limit Updated

Otro:

Customer Refund
      │
      ▼
Amount Check
      │
      ├── Small → Automatic
      │
      └── Large → Approval
53. Core Principles
1. Customer Independence

Customer no debe confundirse con User o Tenant.

2. Financial Separation

Customer y Billing Account deben ser entidades independientes.

3. Multiple Accounts

Un Customer puede administrar múltiples Billing Accounts.

4. Enterprise Support

Debe soportar estructuras corporativas complejas.

5. Financial Identity

Cada Customer debe tener identidad comercial y fiscal consistente.

6. Lifecycle Management

El estado del cliente debe administrarse explícitamente.

7. Payment Intelligence

Debe existir historial y análisis de pagos.

8. Risk Management

El cliente debe poder evaluarse financieramente.

9. Profitability

Debe poder calcularse el costo de servir.

10. AI-Ready

Toda la estructura debe alimentar Customer Financial Intelligence.

54. Resultado de 26/07

Billing Customers establece al Customer como la contraparte comercial central de Billing, separándolo correctamente de usuarios, tenants, organizaciones y cuentas financieras.

La estructura queda:

Enterprise
   ↓
Organization
   ↓
Billing Organization
   ↓
Customer
   │
   ├── Legal Entities
   ├── Contacts
   ├── Billing Accounts
   ├── Contracts
   └── Subscriptions
          │
          ▼
        Usage
          │
          ▼
        Charges
          │
          ▼
       Invoices
          │
          ▼
       Payments
          │
          ▼
        Ledger
          │
          ▼
       Revenue
          │
          ▼
      Profitability

El principio fundamental es:

Billing Customer representa la contraparte comercial de una relación económica y proporciona el contexto necesario para administrar contratos, cuentas, suscripciones, facturación, pagos, riesgo, ingresos y rentabilidad a lo largo de todo el ciclo de vida del cliente.

El siguiente componente natural de la arquitectura es:

26/08 — Billing Products & Catalog
