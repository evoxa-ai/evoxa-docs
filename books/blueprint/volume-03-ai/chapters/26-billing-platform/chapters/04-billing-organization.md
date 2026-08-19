1. Billing Organizations

Billing Organizations define cómo la plataforma administra la estructura empresarial responsable de las relaciones comerciales y financieras.

En una plataforma Enterprise, Billing no puede asumir que existe una única empresa que compra y paga por todos los servicios. Una misma plataforma puede contener:

múltiples grupos empresariales;
múltiples organizaciones;
múltiples entidades legales;
múltiples unidades de negocio;
múltiples tenants;
múltiples clientes;
múltiples cuentas de facturación;
diferentes responsables financieros.

Por ello, Billing debe incorporar una jerarquía organizacional financiera capaz de representar quién vende, quién compra, quién recibe la factura, quién paga y a qué entidad financiera pertenece cada operación.

2. Organización dentro de Billing

La organización financiera puede representarse:

Enterprise
   │
   ├── Organization
   │      │
   │      ├── Business Unit
   │      │       │
   │      │       └── Tenant
   │      │
   │      └── Legal Entity
   │
   └── Billing Configuration

La estructura comercial puede ser diferente de la estructura financiera.

Por ejemplo:

Enterprise
│
├── Chile
│   ├── Sales
│   └── Operations
│
├── Mexico
│   ├── Sales
│   └── Operations
│
└── USA
    ├── Sales
    └── Operations

Mientras Billing podría requerir:

Enterprise
│
├── Chile Legal Entity
│      └── Billing Accounts
│
├── Mexico Legal Entity
│      └── Billing Accounts
│
└── USA Legal Entity
       └── Billing Accounts
3. Billing Organization vs Enterprise Organization

No necesariamente son exactamente lo mismo.

La Enterprise Organization representa la estructura operativa.

La Billing Organization representa la estructura necesaria para administrar las relaciones económicas.

Enterprise Organization
        │
        ▼
Billing Organization
        │
        ├── Legal Entity
        ├── Tax Configuration
        ├── Billing Accounts
        ├── Payment Configuration
        ├── Currency
        └── Revenue Configuration

Una organización puede tener múltiples configuraciones financieras.

4. Objetivos

Billing Organizations debe permitir:

representar entidades financieras;
separar responsabilidades;
administrar múltiples unidades de negocio;
soportar múltiples monedas;
administrar jurisdicciones fiscales;
separar ingresos por organización;
controlar acceso financiero;
consolidar información;
realizar reporting financiero;
soportar estructuras Enterprise complejas.
5. Organización financiera

Una Billing Organization puede representar una entidad responsable de:

contratar servicios;
recibir facturas;
realizar pagos;
administrar créditos;
administrar contratos;
controlar presupuesto;
recibir información financiera.

Conceptualmente:

Billing Organization
│
├── Identity
├── Legal Information
├── Tax Information
├── Billing Configuration
├── Payment Configuration
├── Revenue Configuration
└── Financial Policies
6. Legal Entity

Una de las entidades más importantes es la Legal Entity.

Representa jurídicamente a la empresa que:

vende;
compra;
factura;
paga;
reconoce ingresos;
declara impuestos.

Ejemplo:

Enterprise
│
├── EVOXA Chile SpA
├── EVOXA México S.A. de C.V.
└── EVOXA USA Inc.

Cada entidad puede tener:

tax ID;
dirección legal;
moneda;
jurisdicción;
configuración fiscal;
cuentas bancarias;
configuración contable.
7. Billing Organization Hierarchy

Una estructura Enterprise completa puede ser:

Enterprise
   │
   ├── Organization
   │      │
   │      ├── Legal Entity
   │      │
   │      ├── Business Unit
   │      │       │
   │      │       └── Tenant
   │      │
   │      └── Billing Accounts
   │
   └── Finance

Esto permite separar:

estructura empresarial → estructura financiera → estructura de facturación.

8. Billing Organization Types

No todas las organizaciones tienen necesariamente el mismo propósito.

Podemos distinguir:

Corporate Organization

Representa la organización corporativa principal.

Business Organization

Representa una unidad comercial.

Legal Organization

Representa una entidad jurídica.

Billing Organization

Representa una unidad responsable de facturación.

Customer Organization

Representa la organización cliente.

Partner Organization

Representa un socio comercial.

Marketplace Organization

Representa un participante del marketplace.

9. Buyer vs Seller

Billing debe distinguir claramente quién compra y quién vende.

Seller Organization
        │
        │ Invoice
        ▼
Buyer Organization
        │
        │ Payment
        ▼
Seller Organization

Ejemplo:

EVOXA
   │
   │ $10,000 Invoice
   ▼
Customer Corporation
   │
   │ $10,000 Payment
   ▼
EVOXA

Esto permite modelar correctamente las relaciones comerciales.

10. Billing Relationship

La relación entre organizaciones debe ser explícita.

Seller
  │
  ▼
Commercial Relationship
  │
  ▼
Buyer

La relación puede contener:

contract;
pricing;
payment terms;
currency;
tax configuration;
billing frequency;
credit limits;
discounts.
11. Organization-Level Billing Configuration

Cada Billing Organization puede tener configuración propia.

Billing Organization
│
├── Currency
├── Timezone
├── Billing Cycle
├── Invoice Rules
├── Payment Terms
├── Tax Rules
├── Credit Policy
├── Collection Policy
└── Revenue Policy

Esto evita tener reglas financieras globales rígidas.

12. Multi-Currency Organizations

Una organización puede operar con múltiples monedas.

Billing Organization
│
├── Primary Currency: USD
│
├── Supported:
│   ├── EUR
│   ├── CLP
│   ├── MXN
│   └── BRL
│
└── Reporting Currency: USD

Debe diferenciarse:

transaction currency;
invoice currency;
settlement currency;
reporting currency;
accounting currency.
13. Tax Jurisdiction

La organización también define contexto tributario.

Billing Organization
        │
        ▼
Tax Profile
        │
        ├── Country
        ├── Region
        ├── Tax ID
        ├── Tax Category
        └── Tax Exemptions

Esto permite que el Tax Engine determine las reglas aplicables.

14. Billing Addresses

Una organización puede tener diferentes direcciones:

Organization
│
├── Legal Address
├── Billing Address
├── Tax Address
├── Shipping Address
└── Operational Address

No deben asumirse como una única dirección.

15. Billing Contacts

Las organizaciones pueden tener múltiples contactos.

Billing Organization
│
├── Finance Contact
├── Accounts Payable
├── Accounts Receivable
├── Procurement
├── Legal
└── Billing Administrator

Cada contacto puede tener diferentes permisos.

16. Organization Billing Roles

Los roles financieros pueden incluir:

Billing Administrator

Administra configuración.

Finance Manager

Supervisa operaciones financieras.

Accounts Payable

Gestiona facturas y pagos.

Accounts Receivable

Gestiona cobros.

Accountant

Accede a información contable.

Auditor

Accede a registros históricos.

Billing Viewer

Puede consultar información sin modificarla.

17. Organization Access Control

El acceso debe seguir:

User
  │
  ▼
Organization
  │
  ▼
Billing Role
  │
  ▼
Permission
  │
  ▼
Billing Resource

Por ejemplo:

Finance Manager
    │
    ├── View Invoices
    ├── View Payments
    ├── Issue Credits
    └── View Revenue

Mientras:

Billing Viewer
    │
    ├── View Invoices
    └── View Payments
18. Organization-Level Credit Policy

La organización puede tener políticas de crédito.

Credit Policy
│
├── Credit Limit
├── Payment Terms
├── Grace Period
├── Overdue Threshold
├── Collection Strategy
└── Suspension Policy

Ejemplo:

Credit Limit = $50,000
Payment Terms = Net 30
Grace Period = 5 days
19. Organization-Level Payment Configuration

Puede existir configuración específica por organización.

Billing Organization
        │
        ▼
Payment Configuration
        │
        ├── Payment Methods
        ├── Default Provider
        ├── Currency
        ├── Settlement Account
        └── Payment Terms

Esto es importante para organizaciones Enterprise.

20. Billing Organization and Tenants

Una organización puede administrar múltiples tenants.

Billing Organization
│
├── Tenant A
├── Tenant B
├── Tenant C
└── Tenant D

Pero el billing puede configurarse de distintas formas.

Individual Billing
Tenant A → Invoice A
Tenant B → Invoice B
Tenant C → Invoice C
Consolidated Billing
Tenant A ─┐
Tenant B ─┼──→ Consolidated Invoice
Tenant C ─┘
21. Consolidated Billing

Este es un requisito importante para Enterprise.

Una organización puede recibir una única factura:

Organization
│
├── Tenant A → $1,000
├── Tenant B → $2,000
├── Tenant C → $1,500
│
└── Total → $4,500

La factura debe conservar el detalle de cada tenant.

Consolidated Invoice
│
├── Tenant A
│    ├── Subscription
│    └── Usage
│
├── Tenant B
│    ├── Subscription
│    └── Usage
│
└── Tenant C
     ├── Subscription
     └── Usage
22. Organization Billing Account Structure

Una organización puede tener múltiples Billing Accounts.

Billing Organization
│
├── Corporate Account
│
├── Chile Account
│
├── Mexico Account
│
└── Enterprise Account

Cada cuenta puede tener:

currency;
payment terms;
payment methods;
invoices;
subscriptions;
credits;
balance.
23. Parent-Child Organizations

Enterprise puede requerir relaciones jerárquicas.

Global Corporation
       │
 ┌─────┼─────┐
 ▼     ▼     ▼
Chile Mexico USA
 │      │      │
 ▼      ▼      ▼
Tenants Tenants Tenants

Esto permite:

reporting consolidado;
billing consolidado;
administración centralizada;
políticas globales.
24. Organization Billing Policies

Las políticas pueden heredarse.

Global Policy
     │
     ▼
Organization Policy
     │
     ▼
Tenant Policy
     │
     ▼
Billing Account Policy

Pero debe existir una regla de precedencia clara.

Por ejemplo:

Global
  ↓
Organization
  ↓
Tenant
  ↓
Billing Account

La configuración más específica puede sobrescribir la general cuando la política lo permita.

25. Billing Organization Lifecycle

Una organización puede seguir:

PROSPECT
   │
   ▼
ONBOARDING
   │
   ▼
ACTIVE
   │
   ├── SUSPENDED
   │       │
   │       ▼
   │     ACTIVE
   │
   └── CLOSED

Los estados deben controlar qué operaciones financieras pueden realizarse.

26. Organization Onboarding

El onboarding financiero puede incluir:

Organization Created
       │
       ▼
Legal Information
       │
       ▼
Tax Configuration
       │
       ▼
Billing Configuration
       │
       ▼
Payment Configuration
       │
       ▼
Credit Policy
       │
       ▼
Billing Activated

Una organización no debería comenzar a facturar hasta que los requisitos obligatorios estén completos.

27. Organization Financial Identity

Cada organización debe tener una identidad financiera estable.

Conceptualmente:

Billing Organization
│
├── organization_id
├── legal_entity_id
├── tax_profile_id
├── default_currency
├── billing_timezone
├── billing_account
└── financial_status

Esta identidad debe permanecer estable aunque cambien otras configuraciones.

28. Organization Data Isolation

La información financiera debe respetar aislamiento.

Organization A
   │
   ├── Customers
   ├── Invoices
   ├── Payments
   └── Revenue


Organization B
   │
   ├── Customers
   ├── Invoices
   ├── Payments
   └── Revenue

Un usuario autorizado para A no debe poder consultar B salvo que tenga permisos explícitos a nivel superior.

29. Organization Financial Boundaries

La organización define límites importantes:

Organization
│
├── Financial Boundary
│
├── Data Boundary
│
├── Access Boundary
│
├── Tax Boundary
│
└── Reporting Boundary

Esto es fundamental en plataformas Enterprise.

30. Organization Consolidation

Para organizaciones multinacionales:

                    Global Enterprise
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Chile          Mexico         USA
             │             │             │
             ▼             ▼             ▼
          Revenue       Revenue        Revenue
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                    Consolidated Revenue

El sistema debe permitir:

consolidación;
reporting;
conversiones de moneda;
eliminación de duplicidades;
segmentación.
31. Organization-Level Analytics

Billing Analytics debe poder segmentar:

Enterprise
   │
   ├── Organization Revenue
   ├── Organization MRR
   ├── Organization ARR
   ├── Organization Churn
   ├── Organization ARPU
   ├── Outstanding Balance
   └── Collection Rate

Esto permite analizar la rentabilidad por unidad empresarial.

32. Organization Cost & Profitability

Billing Organizations también debe conectarse con FinOps.

Organization Revenue
       -
Organization Costs
       =
Organization Margin

Puede descomponerse:

Revenue
│
├── Subscription Revenue
├── Usage Revenue
└── Services Revenue


Costs
│
├── Infrastructure
├── AI
├── Payment Fees
├── Support
└── Operations
33. Organization Events

Los cambios organizacionales importantes deben generar eventos.

OrganizationCreated
OrganizationActivated
OrganizationSuspended
OrganizationUpdated
LegalEntityChanged
TaxProfileChanged
BillingConfigurationChanged
CreditLimitChanged
PaymentConfigurationChanged
OrganizationClosed

Estos eventos permiten:

auditoría;
integración;
automatización;
analytics.
34. Organization APIs

El dominio debe exponer APIs como:

POST   /billing/organizations
GET    /billing/organizations
GET    /billing/organizations/{id}
PATCH  /billing/organizations/{id}
DELETE /billing/organizations/{id}

Configuraciones:

GET   /billing/organizations/{id}/configuration
PATCH /billing/organizations/{id}/configuration

Financial structure:

GET /billing/organizations/{id}/accounts
GET /billing/organizations/{id}/invoices
GET /billing/organizations/{id}/payments
GET /billing/organizations/{id}/balances
35. Organization Data Model

Modelo conceptual:

BillingOrganization
│
├── id
├── enterprise_id
├── organization_id
├── legal_entity_id
├── name
├── status
├── default_currency
├── timezone
├── tax_profile_id
├── billing_configuration_id
├── payment_configuration_id
├── credit_policy_id
├── created_at
└── updated_at

Relaciones:

BillingOrganization
       │
       ├── LegalEntity
       ├── BillingAccounts
       ├── Customers
       ├── Tenants
       ├── Contracts
       ├── Invoices
       ├── Payments
       └── Policies
36. Organization Security

Las operaciones críticas deben requerir permisos específicos.

Por ejemplo:

Create Organization
       │
       ▼
Organization Admin

Mientras:

Change Credit Limit
       │
       ▼
Finance Administrator

Y:

Close Organization
       │
       ▼
Financial Administrator
       +
Approval Workflow

Las operaciones de alto impacto deberían soportar aprobación dual.

37. Organization Governance

Billing Organizations debe integrarse con Governance.

Debe poder definir:

quién puede modificar precios;
quién puede otorgar créditos;
quién puede aprobar refunds;
quién puede modificar payment terms;
quién puede cambiar credit limits;
quién puede cerrar cuentas.

Ejemplo:

Refund > $10,000
       │
       ▼
Approval Required
       │
 ┌─────┴─────┐
 ▼           ▼
Finance     Executive
Approval    Approval
38. Organization + AI

La IA puede aportar inteligencia a nivel organizacional.

Revenue Forecasting
Historical Billing
       ↓
AI Model
       ↓
Revenue Forecast
Credit Risk
Payment History
       ↓
Risk Model
       ↓
Credit Risk
Customer Expansion
Usage
  +
Billing
  +
Behavior
  ↓
AI
  ↓
Expansion Opportunity

La IA debe actuar como capa de inteligencia, no reemplazar los controles financieros.

39. Organization Architecture

La arquitectura puede resumirse:

                    ENTERPRISE
                        │
                        ▼
                ORGANIZATION DOMAIN
                        │
        ┌───────────────┼────────────────┐
        ▼               ▼                ▼
   Organization      Legal Entity      Policies
        │
        ├───────────────┬───────────────┐
        ▼               ▼               ▼
     Tenants       Billing Accounts   Contracts
        │               │
        │               ├── Invoices
        │               ├── Payments
        │               ├── Credits
        │               └── Balances
        │
        └───────────────────────────────┐
                                        ▼
                                Financial Analytics
40. Enterprise Billing Organization Model

Para EVOXA, el modelo puede ser:

                         EVOXA ENTERPRISE
                                │
                 ┌──────────────┴──────────────┐
                 │                             │
          Corporate Organization        Legal Entities
                 │                             │
        ┌────────┼────────┐          ┌─────────┼─────────┐
        ▼        ▼        ▼          ▼         ▼         ▼
      Chile    Mexico    USA      Chile SpA  Mexico    USA Inc
        │        │        │
        ▼        ▼        ▼
     Tenants  Tenants  Tenants
        │        │        │
        └────────┼────────┘
                 ▼
         Billing Accounts
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
     Invoices Payments Credits
                 │
                 ▼
             Revenue

Este modelo permite que EVOXA evolucione desde un SaaS simple hacia una plataforma Enterprise multinacional.

41. Core Principles

Los principios de Billing Organizations son:

1. Organizational Independence

La estructura financiera debe poder evolucionar independientemente de la estructura operativa.

2. Legal Separation

Las entidades legales deben poder separarse correctamente.

3. Financial Isolation

Los datos y operaciones financieras deben respetar límites organizacionales.

4. Hierarchical Management

Debe soportarse estructura padre-hijo.

5. Consolidation

Debe ser posible consolidar información financiera.

6. Multi-Currency

Las organizaciones pueden operar con múltiples monedas.

7. Multi-Jurisdiction

Cada entidad puede tener diferentes reglas fiscales.

8. Policy Inheritance

Las políticas pueden heredarse con reglas de precedencia.

9. Controlled Access

El acceso financiero debe depender del rol y contexto organizacional.

10. Auditability

Los cambios organizacionales críticos deben quedar registrados.

42. Resultado de 26/04

Billing Organizations establece la estructura organizacional necesaria para que Billing pueda operar a escala Enterprise.

El modelo central es:

Enterprise
   ↓
Organization
   ↓
Legal Entity
   ↓
Business Unit
   ↓
Tenant
   ↓
Billing Account
   ↓
Customer
   ↓
Subscription / Contract
   ↓
Invoice
   ↓
Payment
   ↓
Revenue

La idea fundamental es:

Billing Organization representa la estructura empresarial y financiera que determina quién participa en una relación económica, bajo qué entidad legal, con qué políticas, moneda, jurisdicción, cuentas y responsabilidades financieras.

Esto prepara el siguiente nivel de la arquitectura:

26/05 — Billing Tenants
