1. Billing Accounts

Billing Accounts representa la cuenta financiera central utilizada para agrupar, controlar y administrar las obligaciones económicas de una relación comercial.

Mientras el Tenant representa una unidad lógica y operativa dentro de la plataforma, el Billing Account representa la unidad financiera contra la cual se registran cargos, facturas, pagos, créditos, ajustes, saldos y obligaciones de pago.

La cuenta de Billing constituye uno de los elementos fundamentales del modelo financiero:

Organization
      │
      ▼
    Tenant
      │
      ▼
Billing Account
      │
 ┌────┼───────────────┐
 ▼    ▼               ▼
Charges Invoices    Payments
      │
      ├── Credits
      ├── Refunds
      ├── Adjustments
      └── Balance
2. Propósito del Billing Account

El Billing Account debe responder:

¿Quién es responsable financieramente?
¿Qué cliente debe pagar?
¿Qué tenant genera los cargos?
¿Dónde se acumulan las obligaciones?
¿Qué facturas pertenecen a esta relación?
¿Qué pagos se han realizado?
¿Qué saldo existe?
¿Qué créditos están disponibles?
¿Qué condiciones de pago aplican?
¿Qué moneda se utiliza?
¿Qué entidad legal está involucrada?

Por lo tanto, el Billing Account funciona como el financial container de una relación comercial.

3. Billing Account dentro de la jerarquía

La jerarquía completa:

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
   │
   ├── Customers
   ├── Subscriptions
   ├── Contracts
   ├── Usage
   ├── Charges
   ├── Invoices
   ├── Payments
   ├── Credits
   ├── Refunds
   └── Adjustments

El Billing Account es el punto donde estas operaciones convergen financieramente.

4. Billing Account vs Customer

No deben considerarse necesariamente la misma entidad.

Customer
   │
   ▼
Billing Account
   │
   ├── Subscription
   ├── Invoice
   └── Payment

Un Customer representa una entidad comercial o persona.

El Billing Account representa cómo se administra financieramente esa relación.

5. Un Customer, múltiples Billing Accounts

Un cliente Enterprise puede tener varias cuentas:

Customer
│
├── Chile Billing Account
├── Mexico Billing Account
├── USA Billing Account
└── Corporate Billing Account

Cada cuenta puede tener:

moneda;
condiciones de pago;
contratos;
invoices;
payment methods;
créditos.
6. Múltiples Customers, un Billing Account

También puede ocurrir lo contrario.

Corporate Billing Account
│
├── Customer A
├── Customer B
└── Customer C

Esto permite administrar estructuras corporativas donde diferentes unidades utilizan una cuenta financiera consolidada.

7. Tenant vs Billing Account

Un tenant puede tener:

Tenant
│
├── Billing Account A
├── Billing Account B
└── Billing Account C

Por ejemplo:

Tenant
│
├── Production Account
├── Sandbox Account
└── Enterprise Account

Esto permite separar diferentes contextos financieros.

8. Billing Account Types

La plataforma puede soportar diferentes tipos.

Customer Billing Account

Cuenta asociada directamente a un cliente.

Tenant Billing Account

Cuenta utilizada por un tenant.

Corporate Billing Account

Cuenta consolidada para toda una organización.

Project Billing Account

Cuenta asociada a un proyecto específico.

Department Billing Account

Cuenta asociada a una unidad interna.

Marketplace Billing Account

Cuenta utilizada para operaciones de marketplace.

Prepaid Account

Cuenta con saldo prepagado.

Postpaid Account

Cuenta con facturación posterior al consumo.

9. Account Lifecycle

Una cuenta puede seguir:

CREATED
   │
   ▼
PENDING
   │
   ▼
ACTIVE
   │
   ├── SUSPENDED
   │       │
   │       ▼
   │     ACTIVE
   │
   ├── CLOSED
   │
   └── ARCHIVED

Cada estado controla qué operaciones pueden ejecutarse.

10. Account Status vs Payment Status

No deben confundirse.

Una cuenta puede estar:

Account Status = ACTIVE
Payment Status = PAST_DUE

O:

Account Status = ACTIVE
Payment Status = CURRENT

También:

Account Status = SUSPENDED
Payment Status = CURRENT

La arquitectura debe manejar estas dimensiones independientemente.

11. Billing Account Identity

Cada cuenta debe poseer una identidad estable.

BillingAccount
│
├── id
├── account_number
├── organization_id
├── tenant_id
├── customer_id
├── legal_entity_id
├── status
├── account_type
├── currency
├── timezone
└── created_at

El account_number puede ser un identificador amigable para usuarios y documentos.

12. Financial Responsibility

El Billing Account debe definir quién es financieramente responsable.

Customer
   │
   ▼
Billing Account
   │
   ▼
Financial Responsibility
   │
   ├── Charges
   ├── Invoices
   └── Payments

Esto permite diferenciar:

Service User
      ≠
Billing Responsible Party

Por ejemplo, una empresa puede tener:

Users → consume service
Corporate Finance → pays invoice
13. Bill-To / Service-To

Una estructura Enterprise debe distinguir:

Bill-To

Quién recibe y paga la factura.

Service-To

Quién utiliza el servicio.

Service-To
   │
   ▼
Tenant
   │
   ▼
Usage
   │
   ▼
Billing Account
   │
   ▼
Bill-To

Esto es esencial para consolidated billing.

14. Billing Account Configuration

Cada cuenta puede tener:

Billing Account
│
├── Currency
├── Timezone
├── Billing Cycle
├── Invoice Frequency
├── Payment Terms
├── Payment Method
├── Tax Profile
├── Credit Policy
├── Collection Policy
└── Notification Policy

Las configuraciones pueden heredarse desde la organización o tenant.

15. Billing Cycle

La cuenta define cómo se agrupan los cargos.

Puede utilizar:

monthly;
quarterly;
annual;
custom;
anniversary;
usage period.

Ejemplo:

Billing Account
       │
       ▼
Monthly Cycle
       │
       ├── January
       ├── February
       ├── March
       └── ...
16. Billing Anchor

El Billing Anchor determina el día de referencia del ciclo.

Ejemplo:

Billing Anchor = 15


Cycle:
15 Jan → 14 Feb
15 Feb → 14 Mar
15 Mar → 14 Apr

Esto es especialmente importante para subscriptions.

17. Currency

El Billing Account puede establecer una moneda principal:

Billing Account
   │
   ├── Transaction Currency
   ├── Invoice Currency
   ├── Settlement Currency
   └── Reporting Currency

Debe definirse claramente qué moneda se utiliza en cada operación.

18. Payment Terms

La cuenta puede definir condiciones:

Payment Terms
│
├── Due Immediately
├── Net 7
├── Net 15
├── Net 30
├── Net 45
└── Net 60

Ejemplo:

Invoice Date: 01/08
Terms: Net 30
Due Date: 31/08
19. Credit Limit

El Billing Account puede tener un límite de crédito.

Credit Limit
     │
     ▼
$50,000

El sistema debe controlar:

Current Exposure
      +
Pending Charges
      >
Credit Limit

y activar las políticas correspondientes.

20. Account Balance

El saldo representa la posición financiera actual.

Conceptualmente:

Opening Balance
     +
Charges
     +
Taxes
     +
Adjustments
     -
Payments
     -
Credits
     -
Refunds
     =
Current Balance

Debe existir una clara diferencia entre:

amount due;
outstanding balance;
available credit;
prepaid balance;
overdue balance.
21. Balance Types

Una cuenta puede tener:

Outstanding Balance

Total pendiente de pago.

Overdue Balance

Parte vencida.

Credit Balance

Crédito disponible.

Prepaid Balance

Saldo disponible antes del consumo.

Available Credit

Capacidad de crédito restante.

Ejemplo:

Credit Limit       $50,000
Outstanding        $15,000
Available Credit   $35,000
22. Charges

Todos los cargos relacionados con la cuenta deben asociarse al Billing Account.

Billing Account
      │
      ▼
Charge
│
├── Subscription
├── Usage
├── Add-on
├── Overage
└── Adjustment

Cada charge debe poder rastrearse hasta su origen.

23. Invoice Ownership

Las facturas deben pertenecer a una cuenta de Billing.

Billing Account
      │
      ├── Invoice #1001
      ├── Invoice #1002
      └── Invoice #1003

La factura puede incluir cargos de:

múltiples subscriptions;
múltiples productos;
múltiples tenants;
múltiples períodos.
24. Consolidated Billing Account

Un Billing Account puede consolidar múltiples fuentes.

Tenant A ─┐
Tenant B ─┼──→ Corporate Billing Account
Tenant C ─┘

La factura puede mostrar:

Corporate Invoice
│
├── Tenant A
│    └── $2,000
│
├── Tenant B
│    └── $3,500
│
└── Tenant C
     └── $1,500


Total = $7,000

Esto es muy importante para clientes Enterprise.

25. Payment Methods

El Billing Account puede administrar múltiples métodos:

Billing Account
│
├── Credit Card
├── Debit Card
├── Bank Account
├── Wire Transfer
└── ACH / Local Transfer

Debe existir:

método principal;
métodos secundarios;
método de backup;
token del proveedor.
26. Payment Routing

La cuenta puede determinar qué proveedor utilizar.

Billing Account
       │
       ▼
Payment Routing
       │
 ┌─────┼─────┐
 ▼     ▼     ▼
PSP A  PSP B  Bank

La selección puede depender de:

país;
moneda;
tipo de pago;
riesgo;
costo;
disponibilidad.
27. Auto-Pay

Puede habilitarse:

Auto-Pay = Enabled

Workflow:

Invoice Finalized
       │
       ▼
Auto-Pay
       │
       ▼
Payment Attempt
       │
 ┌─────┴─────┐
 ▼           ▼
Success     Failure
             │
             ▼
           Retry

Debe manejarse con idempotencia.

28. Invoice Delivery

El Billing Account puede definir:

email;
portal;
API;
EDI;
electronic invoicing;
SFTP;
ERP integration.
Invoice
   │
   ├── Email
   ├── Portal
   ├── API
   └── ERP
29. Tax Profile

La cuenta puede estar asociada a un perfil fiscal.

Billing Account
      │
      ▼
Tax Profile
      │
      ├── Jurisdiction
      ├── Tax ID
      ├── Tax Category
      ├── Exemption
      └── Tax Rules

El Tax Engine utilizará este contexto para determinar impuestos.

30. Credits

El Billing Account puede mantener créditos.

Billing Account
      │
      ▼
Credit Balance
      │
      ├── Promotional Credit
      ├── Service Credit
      ├── Contract Credit
      └── Refund Credit

Los créditos deben tener:

origen;
monto;
moneda;
fecha de creación;
expiración;
condiciones de uso.
31. Refunds

Los refunds deben estar asociados a:

Payment
   │
   ▼
Refund
   │
   ▼
Billing Account

Debe existir trazabilidad:

Invoice
   ↓
Payment
   ↓
Refund
   ↓
Balance Adjustment
32. Adjustments

Los ajustes permiten corregir operaciones sin modificar el historial original.

Original Charge
      │
      ▼
Adjustment
      │
      ▼
New Financial State

Tipos:

debit adjustment;
credit adjustment;
invoice adjustment;
balance adjustment.

Esto mantiene la auditabilidad.

33. Account Ledger

Cada Billing Account debe tener un historial financiero.

Billing Account Ledger


Date       Type        Debit    Credit   Balance
------------------------------------------------
01/08      Invoice     1,000       -     1,000
05/08      Payment         -     500       500
10/08      Credit          -     100       400
15/08      Charge        200       -       600

El ledger permite reconstruir la posición financiera.

34. Account Statement

El sistema debe poder generar un estado de cuenta:

Billing Account Statement
│
├── Opening Balance
├── Charges
├── Invoices
├── Payments
├── Credits
├── Refunds
├── Adjustments
└── Closing Balance

Esto es fundamental para clientes Enterprise.

35. Account Reconciliation

El Billing Account debe poder conciliar:

Billing Account
      │
      ▼
Internal Ledger
      │
      ▼
Payment Provider
      │
      ▼
Bank / Settlement

Las diferencias deben generar excepciones.

36. Account Collections

Cuando existe deuda:

Billing Account
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
 ┌─────┴─────┐
 ▼           ▼
Recovered   Written Off

Las políticas pueden variar por cuenta.

37. Account Risk Profile

Billing puede mantener un perfil de riesgo:

Billing Account
      │
      ▼
Risk Profile
      │
      ├── Payment History
      ├── Outstanding Balance
      ├── Credit Utilization
      ├── Failed Payments
      └── Collection History

Esto puede alimentar modelos de riesgo.

38. Account-Level AI

La IA puede analizar la cuenta:

Billing Account Data
        │
        ▼
       AI
        │
 ┌──────┼─────────┐
 ▼      ▼         ▼
Risk   Revenue   Anomaly

Casos:

payment failure prediction;
credit risk;
churn;
unusual usage;
invoice anomaly;
collection probability.
39. Billing Account Security

Debe existir control granular.

User
 │
 ▼
Organization
 │
 ▼
Tenant
 │
 ▼
Billing Account
 │
 ▼
Permission

Ejemplo:

Finance User
│
├── View Account
├── View Invoices
├── View Payments
└── No Refund Permission
40. Approval Controls

Operaciones sensibles pueden requerir aprobación.

Refund > $10,000
       │
       ▼
Approval Workflow
       │
       ├── Finance
       └── Management

También:

credit increases;
large discounts;
write-offs;
manual adjustments;
account closure.
41. Billing Account APIs

APIs conceptuales:

POST   /billing/accounts
GET    /billing/accounts
GET    /billing/accounts/{id}
PATCH  /billing/accounts/{id}

Financial:

GET /billing/accounts/{id}/balance
GET /billing/accounts/{id}/statement
GET /billing/accounts/{id}/charges
GET /billing/accounts/{id}/invoices
GET /billing/accounts/{id}/payments
GET /billing/accounts/{id}/credits
GET /billing/accounts/{id}/refunds
GET /billing/accounts/{id}/ledger

Actions:

POST /billing/accounts/{id}/credits
POST /billing/accounts/{id}/adjustments
POST /billing/accounts/{id}/payment-methods
POST /billing/accounts/{id}/close
42. Billing Account Data Model

Modelo conceptual:

BillingAccount
│
├── id
├── account_number
├── organization_id
├── tenant_id
├── customer_id
├── legal_entity_id
├── account_type
├── status
├── currency
├── timezone
├── billing_cycle
├── billing_anchor
├── payment_terms
├── credit_limit
├── tax_profile_id
├── payment_configuration_id
├── collection_policy_id
├── created_at
└── updated_at

Relaciones:

BillingAccount
│
├── Subscriptions
├── Contracts
├── Charges
├── Invoices
├── Payments
├── Credits
├── Refunds
├── Adjustments
├── Ledger Entries
└── Collections
43. Billing Account Ledger Model

Un modelo conceptual:

LedgerEntry
│
├── id
├── billing_account_id
├── transaction_id
├── transaction_type
├── debit
├── credit
├── amount
├── currency
├── balance_after
├── source_type
├── source_id
├── occurred_at
└── metadata

La entrada debe ser inmutable una vez confirmada, salvo mecanismos controlados de corrección mediante nuevas entradas.

44. Billing Account State Machine
CREATED
   │
   ▼
PENDING
   │
   ▼
ACTIVE
   │
   ├── PAST_DUE
   │      │
   │      ├── RECOVERING
   │      └── COLLECTIONS
   │
   ├── SUSPENDED
   │
   └── CLOSED
          │
          ▼
       ARCHIVED

El estado financiero puede coexistir con el estado operativo del tenant.

45. Billing Account Architecture

A arquitetura completa:

                    BILLING TENANT
                           │
                           ▼
                    BILLING ACCOUNT
                           │
       ┌───────────────────┼───────────────────┐
       │                   │                   │
    Customer          Configuration          Policies
       │                   │                   │
       └───────────────────┼───────────────────┘
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
       Subscriptions    Contracts       Usage
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                        Charges
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Credits         Taxes       Discounts
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                        Invoice
                           │
                           ▼
                        Payment
                           │
                  ┌────────┴────────┐
                  ▼                 ▼
               Refund           Settlement
                  │
                  ▼
                 Ledger
                  │
                  ▼
             Balance Engine
                  │
                  ▼
             Financial State
46. Billing Account + Revenue

El Billing Account debe alimentar Revenue Management.

Charges
   │
   ▼
Invoices
   │
   ▼
Payments
   │
   ▼
Revenue Events
   │
   ▼
Revenue Recognition

Esto permite calcular revenue por:

account;
tenant;
organization;
product;
customer;
geography.

47. Billing Account + FinOps

El account también puede ser una unidad de cost allocation.

Billing Account
       │
       ├── Revenue
       │
       └── Cost
              │
              ▼
         Unit Economics

Puede calcularse:

Account Revenue
-
Account Cost
=
Account Margin
48. Enterprise Billing Account Model

Para EVOXA:

                        ENTERPRISE
                            │
                       Organization
                            │
                      Billing Organization
                            │
                ┌───────────┼───────────┐
                ▼           ▼           ▼
             Tenant A    Tenant B    Tenant C
                │           │           │
                ▼           ▼           ▼
            Account A   Account B   Account C
                │           │           │
                └───────────┼───────────┘
                            │
                  Corporate Account
                            │
                ┌───────────┼───────────┐
                ▼           ▼           ▼
             Invoices    Payments     Credits
                            │
                            ▼
                         Revenue

Esto soporta tanto:

billing independiente;
billing consolidado;
múltiples tenants;
múltiples cuentas;
estructuras corporativas complejas.
49. Core Principles

Los principios de Billing Accounts son:

1. Financial Ownership

La cuenta debe representar claramente la responsabilidad financiera.

2. Balance Integrity

El saldo debe poder reconstruirse y conciliarse.

3. Immutable History

Las operaciones históricas deben preservarse.

4. Multi-Tenant Support

Una cuenta puede pertenecer a un tenant o consolidar varios.

5. Customer Separation

Customer y Billing Account deben poder evolucionar independientemente.

6. Payment Independence

La cuenta puede soportar múltiples métodos y proveedores.

7. Credit Management

Debe administrar límites, créditos y exposición.

8. Consolidation

Debe soportar billing consolidado.

9. Auditability

Toda modificación debe ser rastreable.

10. Automation

Invoices, payments, collections y reconciliation deben poder automatizarse.

50. Resultado de 26/06

Billing Accounts establece la cuenta financiera como el núcleo transaccional de la relación económica.

La estructura fundamental queda:

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
Ledger
   ↓
Balance
   ↓
Revenue
   ↓
Profitability

El principio central es:

El Billing Account es la unidad financiera responsable de consolidar cargos, facturas, pagos, créditos, ajustes, saldos y obligaciones de una relación comercial, proporcionando una representación consistente, auditable y escalable de la posición financiera del cliente.

Con esto queda establecida la cuenta financiera sobre la cual se apoyarán los siguientes dominios:

26/07 — Billing Customers
