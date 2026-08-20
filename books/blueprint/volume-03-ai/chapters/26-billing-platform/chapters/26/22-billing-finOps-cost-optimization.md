1. Billing FinOps & Cost Optimization

Billing FinOps & Cost Optimization es la capa que permite a EVOXA entender no solamente cuánto factura y recauda, sino cuánto cuesta generar, procesar, entregar y mantener ese Revenue.

Hasta ahora:

Product
   ↓
Pricing
   ↓
Subscription
   ↓
Usage
   ↓
Rating
   ↓
Invoice
   ↓
Payment
   ↓
Revenue
   ↓
Accounting
   ↓
Collections
   ↓
Refunds / Disputes
   ↓
Analytics & Revenue Intelligence

Ahora agregamos:

Revenue
   ↓
Cost
   ↓
Margin
   ↓
Optimization

La pregunta central de esta capa es:

¿Cuánto cuesta generar cada unidad de Revenue y cómo podemos maximizar el margen sin deteriorar la experiencia, el servicio o el crecimiento?

2. Propósito

Esta capa debe administrar:

Cost Management;
FinOps;
Cost Allocation;
Cost Attribution;
Cost Centers;
Profit Centers;
Unit Economics;
Gross Margin;
Contribution Margin;
Cost per Customer;
Cost per Subscription;
Cost per Invoice;
Cost per Payment;
Payment Processing Costs;
Cloud Costs;
Infrastructure Costs;
AI Costs;
Support Costs;
Collection Costs;
Refund Costs;
Chargeback Costs;
Partner Costs;
Vendor Costs;
Cost Forecasting;
Budgeting;
Variance Analysis;
Showback;
Chargeback;
Cost Optimization;
Cost Anomaly Detection;
AI Cost Intelligence.
3. FinOps vs Accounting

Es importante separar:

Accounting

Responde:

¿Cómo registramos financieramente el costo?

FinOps

Responde:

¿Por qué tenemos este costo, quién lo genera y cómo podemos optimizarlo?

Ejemplo:

Cloud Expense
$100,000

Accounting:

Expense
$100,000

FinOps:

Service A       $40K
Service B       $30K
Service C       $20K
Unused          $10K

Y posteriormente:

Optimization Opportunity
$10K
4. Posición dentro de Billing
26/17  Billing Revenue Management
26/18  Billing Accounting & Financial Integration
26/19  Billing Dunning & Collections
26/20  Billing Refunds, Adjustments & Disputes
26/21  Billing Analytics & Revenue Intelligence
26/22  Billing FinOps & Cost Optimization

La relación:

Revenue
   │
   ▼
Cost
   │
   ▼
Margin
   │
   ▼
Unit Economics
   │
   ▼
Optimization
5. FinOps Architecture
                         EVOXA
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
           REVENUE                    COST
              │                         │
              │              ┌──────────┼──────────┐
              │              ▼          ▼          ▼
              │           Cloud      Payments    Operations
              │           Cost        Cost         Cost
              │              │          │          │
              └──────────────┼──────────┼──────────┘
                             ▼
                       COST ALLOCATION
                             │
                             ▼
                       UNIT ECONOMICS
                             │
                             ▼
                          MARGIN
                             │
                             ▼
                       OPTIMIZATION
                             │
                             ▼
                            AI
6. Cost Categories

EVOXA debe clasificar costos.

Infrastructure
Cloud
Compute
Storage
Database
Network
CDN
Backup
Monitoring
Billing
Invoice Generation
Invoice Delivery
Tax Calculation
Payment Processing
Payment Gateway
Collections
Refunds
Chargebacks
Product
AI
API
Third-Party Services
Licenses
Data Providers
Operations
Support
Customer Success
Collections
Operations
7. Cost Taxonomy
Cost
│
├── Direct
│
├── Indirect
│
├── Fixed
│
├── Variable
│
├── Recurring
│
├── One-Time
│
└── Shared

Esta clasificación será fundamental para calcular márgenes correctamente.

8. Direct Cost

Costo directamente atribuible a un Customer o producto.

Ejemplo:

Customer A
API Usage
1M requests


Cost
$500
9. Indirect Cost

No puede asignarse directamente:

Corporate Infrastructure
Finance
HR
Security

Estos pueden distribuirse mediante reglas de allocation.

10. Fixed Cost
Monthly Infrastructure
$100K

aunque cambie el volumen de Customers.

11. Variable Cost
API Requests
1M
→ $100


10M
→ $1,000
12. Semi-Variable Cost

Ejemplo:

Database
Base Cost
$1,000


Usage
+$500

Total:

$1,500
13. Cost Driver

Un Cost Driver determina qué genera el costo.

Ejemplos:

API Requests
Storage GB
Compute Hours
Invoices
Payments
Customers
Users
Transactions
AI Tokens
Support Tickets
14. Cost Driver Model
CostDriver
│
├── id
├── name
├── type
├── unit
├── source
├── calculation_method
└── status

Ejemplo:

API Requests
Unit = request
15. Unit Economics

La capa debe responder:

¿Cuánto cuesta servir una unidad de negocio?

Ejemplos:

Cost per Customer
Cost per Invoice
Cost per Payment
Cost per API Request
Cost per Subscription
Cost per AI Interaction
Cost per GB
16. Cost per Customer

Ejemplo:

Infrastructure
$100K


Customers
10,000

Costo promedio:

$10 / Customer

Pero EVOXA debe intentar calcular el costo real por Customer, no solo el promedio.

17. Cost per Subscription
Total Subscription Costs
÷
Active Subscriptions

Ejemplo:

Cost
$200K


Subscriptions
20K


Cost / Subscription
$10
18. Cost per Invoice
Invoice Generation
$50K


Invoices
1M


Cost / Invoice
$0.05

Esto permite detectar invoices de bajo valor económico.

19. Cost per Payment
Payment Processing
$100K


Payments
500K


Average Cost
$0.20

Pero debe calcularse también por:

Provider
Country
Payment Method
Currency
Transaction Size
20. Cost per Customer Segment
SMB
$8 / Customer


Mid-Market
$25 / Customer


Enterprise
$150 / Customer

Esto permite comparar contra:

Revenue / Customer
21. Gross Margin

Una métrica fundamental:

Revenue
-
Cost of Revenue
=
Gross Profit

y:

Gross Profit
÷
Revenue
=
Gross Margin %

Ejemplo:

Revenue
$10M


Cost of Revenue
$3M


Gross Profit
$7M


Gross Margin
70%
22. Cost of Revenue

EVOXA debe poder definir qué costos forman parte del:

Cost of Revenue

Por ejemplo:

Cloud Infrastructure
Payment Fees
AI Inference
Third-Party APIs
Delivery
Direct Support

La clasificación debe ser configurable según el modelo financiero de la empresa.

23. Contribution Margin

Después del Gross Margin puede existir:

Gross Profit
-
Variable Operating Costs
=
Contribution Profit

Ejemplo:

Revenue
$1,000


COGS
$300


Support
$100


Contribution Profit
$600
24. Margin by Customer

Customer A:

Revenue
$100K


Cost
$20K


Gross Margin
80%

Customer B:

Revenue
$100K


Cost
$80K


Gross Margin
20%

Ambos generan el mismo Revenue, pero tienen economías completamente diferentes.

25. Margin by Product
Product A
Revenue $5M
Margin 80%


Product B
Revenue $3M
Margin 50%


Product C
Revenue $2M
Margin 30%

Esto permite identificar:

High Revenue / Low Margin
26. Margin by Plan
Basic
Revenue $1M
Margin 75%


Pro
Revenue $4M
Margin 82%


Enterprise
Revenue $10M
Margin 68%

Enterprise puede generar más Revenue pero también tener costos significativamente superiores.

27. Margin by Tenant
Tenant A
Revenue $500K
Cost $100K
Margin 80%


Tenant B
Revenue $500K
Cost $300K
Margin 40%

Esto es especialmente importante para EVOXA como plataforma Multi-Tenant.

28. Margin by Organization
Organization
   │
   ├── Tenant A
   ├── Tenant B
   ├── Tenant C
   └── Tenant D

Consolidación:

Revenue
-
Allocated Cost
=
Organization Margin
29. Cost Allocation

Los costos compartidos deben distribuirse.

Ejemplo:

Shared Infrastructure
$100K

Allocation:

Tenant A
40%


Tenant B
35%


Tenant C
25%

Resultado:

A
$40K


B
$35K


C
$25K
30. Allocation Rules
CostAllocationRule
│
├── id
├── cost_category
├── allocation_driver
├── allocation_method
├── scope
├── effective_from
└── status
31. Allocation Methods
USAGE_BASED
REVENUE_BASED
CUSTOMER_BASED
HEADCOUNT_BASED
EQUAL_SPLIT
WEIGHTED
ACTIVITY_BASED
32. Usage-Based Allocation
Tenant A
1M requests


Tenant B
3M requests

Total:

4M

Allocation:

A = 25%
B = 75%
33. Revenue-Based Allocation
Tenant A
Revenue $1M


Tenant B
Revenue $3M

Shared Cost:

$100K

Allocation:

A
$25K


B
$75K
34. Activity-Based Costing

Una arquitectura avanzada:

Customer
   ↓
Activities
   ├── API
   ├── Storage
   ├── Support
   ├── Payments
   └── AI
       ↓
Actual Cost

Esto proporciona un costo mucho más preciso.

35. Cost Attribution

Cada costo debe intentar responder:

Who?
What?
Where?
Why?
How much?

Ejemplo:

Cost
$1,000


Tenant
ACME


Service
AI


Driver
Tokens


Region
US


Provider
Cloud Provider
36. Cost Allocation Hierarchy
Cloud Provider
       ↓
Account
       ↓
Region
       ↓
Service
       ↓
Environment
       ↓
Application
       ↓
Tenant
       ↓
Customer
37. Cloud FinOps

EVOXA debe poder consumir:

Compute
Storage
Database
Network
Kubernetes
Serverless
AI
Monitoring
Security

costos provenientes de sus proveedores cloud.

38. Cloud Cost Model
CloudCost
│
├── provider
├── account
├── region
├── service
├── resource
├── usage
├── cost
├── currency
├── period
└── tags
39. Cloud Cost Tags

Ejemplo:

environment=production
service=billing
tenant=acme
product=ai
region=us-east
owner=platform

Las etiquetas permiten attribution.

40. Untagged Cost

Un problema frecuente:

Total Cloud Cost
$1M


Allocated
$900K


Unallocated
$100K

EVOXA debe detectar:

Unallocated Cost
10%

y generar una oportunidad de optimización.

41. Cost Allocation Coverage

KPI:

Allocated Cost
÷
Total Cost

Ejemplo:

$900K / $1M
=
90%

Objetivo:

Increase Allocation Coverage
42. Cost Centers
CostCenter
│
├── Engineering
├── Infrastructure
├── Billing
├── AI
├── Support
├── Security
└── Operations
43. Profit Centers
ProfitCenter
│
├── Billing Platform
├── AI Platform
├── Data Platform
├── Marketplace
└── Enterprise Services
44. Cost Center vs Profit Center
Cost Center

Administra:

Cost
Profit Center

Administra:

Revenue
-
Cost
=
Profit
45. Budget

Cada área puede tener:

Budget
Actual
Forecast
Variance

Ejemplo:

AI


Budget
$100K


Actual
$120K


Variance
+$20K
46. Budget Model
CostBudget
│
├── id
├── organization_id
├── cost_center_id
├── period
├── category
├── budget_amount
├── currency
└── status
47. Budget vs Actual
Budget
$1M


Actual
$900K


Remaining
$100K

o:

Budget
$1M


Actual
$1.2M


Overrun
$200K
48. Cost Variance
Actual Cost
-
Budget Cost
=
Variance

Pero EVOXA debe además explicar:

Why?
49. Cost Variance Drivers

Ejemplo:

Cloud Cost
+$100K


Drivers:


Compute
+$40K


AI
+$35K


Storage
+$15K


Network
+$10K
50. Cost Forecast
Current Cost
$500K


Forecast End of Month
$650K


Budget
$600K

Entonces:

Expected Overrun
$50K
51. FinOps Forecasting

Puede utilizar:

Historical Usage
+
Current Usage
+
Growth
+
Pricing
+
Contracts

para proyectar:

Next Month
Next Quarter
Next Year
52. Cost Anomaly Detection

Ejemplo:

Normal Daily Cost
$10K


Today
$18K

Anomaly:

+80%
53. Cost Anomaly Engine
Actual
   ↓
Expected
   ↓
Variance
   ↓
Threshold
   ↓
Anomaly
   ↓
Root Cause
54. Cost Root Cause
Cost Spike
$50K
   ↓
AI Service
   ↓
Model Inference
   ↓
Token Usage
   ↓
Tenant A
   ↓
Feature X

Esto permite pasar de:

"El cloud está caro"

a:

"El 65% del incremento proviene de AI inference del Tenant A."

55. AI Cost Analytics

EVOXA debe poder calcular:

Input Tokens
Output Tokens
Model Calls
Embedding Calls
Vector Searches
Inference Time
GPU Hours

y asociarlos a:

Customer
Tenant
Feature
Agent
Workflow
56. Cost per AI Request

Ejemplo:

AI Cost
$10,000


Requests
1M


Cost / Request
$0.01

Pero también:

Customer A
$0.02


Customer B
$0.005
57. AI Margin

Customer paga:

$100

AI cost:

$30

Gross contribution:

$70

EVOXA puede calcular:

AI Gross Margin
70%
58. Agent Cost

En el futuro EVOXA tendrá agentes.

Cada Agent puede tener:

Agent
   ↓
Tasks
   ↓
LLM Calls
   ↓
Tools
   ↓
APIs
   ↓
Compute

Costo total:

Agent Cost
59. Cost per Workflow

Ejemplo:

Workflow
Invoice Processing


Executions
100K


Total Cost
$5K


Cost / Execution
$0.05
60. Cost per Feature
Feature A
Revenue $1M
Cost $100K
Margin 90%


Feature B
Revenue $500K
Cost $300K
Margin 40%

Esto permite tomar decisiones de producto.

61. Cost-to-Serve

Una métrica especialmente importante:

Cost-to-Serve

representa cuánto cuesta atender y operar un Customer.

Puede incluir:

Infrastructure
Support
Payments
Collections
AI
Storage
Networking
Operations
62. Customer Profitability

Ejemplo:

Customer A


Revenue
$100K


Infrastructure
$10K


Support
$5K


Payments
$2K


AI
$3K


Collections
$1K


Total Cost
$21K


Contribution
$79K
63. Customer Profitability Segmentation
High Revenue
High Margin
→ Strategic


High Revenue
Low Margin
→ Optimize


Low Revenue
High Cost
→ Review


Low Revenue
Low Cost
→ Self-Service
64. Cost-to-Revenue Ratio
Cost
÷
Revenue

Ejemplo:

Cost
$3M


Revenue
$10M


Cost / Revenue
30%
65. Gross Margin by Customer
Customer A
90%


Customer B
75%


Customer C
40%


Customer D
15%

Esto permite identificar Customers que requieren intervención comercial o técnica.

66. Cost-Based Pricing Intelligence

Billing Analytics puede combinar:

Price
+
Revenue
+
Cost
+
Margin

para detectar:

Underpriced Customer

Ejemplo:

Customer pays
$100


Cost-to-Serve
$80


Margin
20%

Mientras el objetivo puede ser:

Target Margin
60%
67. Pricing Optimization

EVOXA puede recomendar:

Increase Price
Reduce Discount
Change Plan
Introduce Usage Charge
Introduce Minimum Commitment

La recomendación debe quedar separada de la ejecución.

68. Minimum Viable Margin

Cada producto puede tener:

Target Margin

Ejemplo:

AI Product
Target Margin
70%

Si:

Actual
45%

se genera:

Margin Risk
69. Margin Alert
Product
AI Agent


Target Margin
70%


Current
52%


Alert
HIGH
70. Cost Optimization Engine
Cost
  ↓
Analyze
  ↓
Detect Waste
  ↓
Identify Opportunity
  ↓
Estimate Savings
  ↓
Recommend Action
  ↓
Approval
  ↓
Execute
  ↓
Measure Savings
71. Optimization Opportunities

Ejemplos:

Unused Resources
Overprovisioned Compute
Low Utilization
Duplicate Services
High Payment Fees
Expensive AI Model
Excessive Storage
Unoptimized Network
72. Cloud Rightsizing

Ejemplo:

Instance
16 CPU


Actual Usage
20%

Recomendación:

Resize
16 CPU → 4 CPU

Potential saving:

$5,000/month

La acción debe estar sujeta a validación técnica.

73. Idle Resource Detection
Resource
Production DB


Utilization
2%


Cost
$2,000/month

Insight:

Potential Waste
74. Storage Optimization
Hot Storage
100TB


Old Data
60TB

Recomendación:

Move
Cold Data
→
Archive Storage
75. Payment Processing Optimization

EVOXA puede analizar:

Provider
Fee
Success Rate
Currency
Region
Payment Method

Ejemplo:

Provider A
Fee 2.9%


Provider B
Fee 2.4%


Provider B
Potential Saving
$50K/year

No implica que el proveedor más barato sea siempre mejor: debe considerarse también aprobación, fraude, cobertura, settlement y experiencia del Customer.

76. Payment Routing Optimization

Puede existir:

Customer
   ↓
Payment Routing Engine
   ↓
Provider A / B / C

según:

Cost
Success Rate
Region
Currency
Risk
77. Payment Cost Model
Payment Cost
=
Processing Fee
+
Gateway Fee
+
FX Fee
+
Chargeback Fee
+
Other Provider Costs
78. Collection Cost

No basta con saber:

Recovered
$1M

Debe saberse:

Collection Cost
$100K

Entonces:

Net Recovery
$900K
79. Collection ROI
Recovered Amount
-
Collection Cost

o:

Incremental Recovery
÷
Collection Cost

Ejemplo:

Incremental Recovery
$500K


Cost
$100K


ROI
5x
80. Refund Cost

Un refund puede tener:

Refund Amount
+
Payment Processing Loss
+
FX Cost
+
Operational Cost

Esto permite calcular:

True Refund Cost
81. Chargeback Cost
Chargeback Amount
+
Provider Fee
+
Operational Cost
+
Lost Product/Service Cost

Esto permite medir:

True Chargeback Cost
82. Support Cost Attribution

Customer Support puede ser asignado mediante:

Tickets
Interactions
Minutes
Agents
Priority

Ejemplo:

Customer A
10 tickets
$200 support cost


Customer B
100 tickets
$2,000 support cost
83. Cost per Support Ticket
Support Cost
÷
Tickets

Ejemplo:

$100K
÷
10K tickets
=
$10/ticket
84. Cost of Discounts

Un descuento:

List Price
$1,000


Discount
$200


Customer pays
$800

representa:

$200 Revenue Sacrifice

FinOps puede analizar:

Discount
vs
Customer Lifetime Value
vs
Margin
85. Discount ROI

Ejemplo:

Discount
$10K


Incremental ARR
$100K

Puede calcularse:

Discount Efficiency

para determinar si la concesión produjo suficiente valor.

86. Cost of Free Trials
Free Customers
10K


Infrastructure Cost
$20K


Support Cost
$10K


AI Cost
$15K


Total
$45K

Después:

Conversions
500

Cost per Converted Customer:

$90
87. Cost of Acquisition Integration

Si EVOXA integra CAC:

CAC
+
Cost-to-Serve
+
Payment Cost
+
Support

puede calcular:

True Customer Acquisition Economics
88. Customer Lifetime Economics
Customer
   │
   ├── Acquisition Cost
   ├── Revenue
   ├── Infrastructure
   ├── Support
   ├── Payments
   ├── AI
   ├── Collections
   └── Refunds
          │
          ▼
      Lifetime Profit
89. Lifetime Gross Profit

Ejemplo:

Lifetime Revenue
$50K


Lifetime Cost
$15K


Lifetime Gross Profit
$35K
90. Margin Cohorts

Los cohortes también pueden analizar:

2026-Q1 Customers
Gross Margin 72%


2026-Q2
Gross Margin 68%


2026-Q3
Gross Margin 60%

Esto puede revelar que los nuevos Customers son cada vez más caros de servir.

91. Cost Cohort Analysis
Customer Cohort
   ↓
Month 1 Cost
Month 3 Cost
Month 6 Cost
Month 12 Cost

Esto ayuda a entender cómo evoluciona el Cost-to-Serve.

92. Cost Forecast

Ejemplo:

Current
$1M/month


Growth
15%


Forecast
$1.15M/month

Pero el modelo debe considerar:

Usage Growth
Pricing Changes
Contracts
Optimization
Seasonality
93. Cost Scenario Planning
Base
Revenue Growth
10%


Cost Growth
8%
Upside
Revenue
+20%


Cost
+10%
Downside
Revenue
+3%


Cost
+12%

Resultado:

Margin Scenario
94. Margin Forecast
Revenue Forecast
$20M


Cost Forecast
$7M


Projected Gross Profit
$13M


Projected Margin
65%
95. Margin Waterfall
Opening Margin
   ↓
Revenue Growth
   ↓
Price Changes
   ↓
Cloud Cost
   ↓
AI Cost
   ↓
Payment Cost
   ↓
Support Cost
   ↓
Optimization
   ↓
Ending Margin
96. Cost Optimization Waterfall
Current Cost
$10M
   │
   ├── Rightsizing
   -$500K
   │
   ├── Payment Optimization
   -$200K
   │
   ├── Storage Optimization
   -$100K
   │
   └── AI Optimization
   -$300K
        │
        ▼
Optimized Cost
$8.9M
97. Savings Tracking

No basta con recomendar:

Potential Saving
$1M

Debe distinguir:

Potential
Committed
Implemented
Realized
98. Savings Lifecycle
Identified
   ↓
Validated
   ↓
Approved
   ↓
Implemented
   ↓
Measured
   ↓
Realized
99. Savings Model
OptimizationOpportunity
│
├── id
├── category
├── estimated_savings
├── committed_savings
├── realized_savings
├── implementation_cost
├── payback_period
├── owner
├── status
└── metadata
100. Payback Period

Ejemplo:

Implementation Cost
$20K


Monthly Savings
$10K

Payback:

2 months
101. Optimization Priority

Puede calcularse usando:

Savings
+
Confidence
+
Effort
+
Risk

Resultado:

QUICK_WIN
HIGH_VALUE
STRATEGIC
LOW_PRIORITY
102. Optimization Opportunity

Ejemplo:

Opportunity
AI Model Optimization


Current Cost
$100K/month


Potential Saving
$30K/month


Implementation
Medium


Risk
Low


Priority
HIGH
103. Cost Governance

Debe existir:

Cost Policies
Budgets
Limits
Approvals
Alerts
Owners
104. Cost Policy
CostPolicy
│
├── scope
├── category
├── threshold
├── action
├── approval_required
├── effective_from
└── status
105. Budget Alert
Budget
$100K


Actual
$85K


Forecast
$110K

El sistema puede alertar:

Projected Budget Overrun

antes de que ocurra.

106. Cost Guardrails

Ejemplos:

AI Cost > $50K/month
→ Alert


Tenant Cost > $10K/month
→ Review


Unallocated Cloud Cost > 5%
→ Alert


Payment Fee > Target
→ Optimization
107. Showback

Showback muestra a cada equipo cuánto consume.

Ejemplo:

Engineering
Cloud Cost
$200K


AI
$100K


Billing
$80K

No necesariamente se les cobra internamente.

108. Chargeback

Chargeback asigna el costo a un área o Tenant.

Ejemplo:

Shared Cloud
$1M


Tenant A
$400K


Tenant B
$350K


Tenant C
$250K

Esto puede alimentar la rentabilidad por Tenant.

109. Showback vs Chargeback
SHOWBACK
→ Visibility


CHARGEBACK
→ Financial Allocation

EVOXA debe soportar ambos.

110. Cost Allocation Ledger
Cost
$100K


Allocation
Tenant A $40K
Tenant B $35K
Tenant C $25K

Debe quedar registrado como un ledger analítico.

111. Cost Lineage

Cada costo debe poder rastrearse:

Invoice from Provider
   ↓
Cloud Cost
   ↓
Service
   ↓
Resource
   ↓
Tenant
   ↓
Customer
   ↓
Product
112. Cost Reconciliation

El total asignado debe coincidir:

Provider Cost
=
Allocated Cost
+
Unallocated Cost

Ejemplo:

$1M
=
$950K
+
$50K
113. FinOps Reconciliation

Debe reconciliar:

Cloud Provider
Payment Provider
Accounting
Billing
FinOps

Ejemplo:

Provider
$100K


FinOps
$100K


Accounting
$100K
114. Cost Data Model
CostRecord
│
├── id
├── provider
├── source
├── category
├── subcategory
├── amount
├── currency
├── date
├── organization_id
├── tenant_id
├── customer_id
├── product_id
├── cost_center_id
├── profit_center_id
├── allocation_status
└── metadata
115. Cost Driver Model
CostDriver
│
├── id
├── name
├── unit
├── quantity
├── unit_cost
├── total_cost
├── source
├── period
└── metadata

Ejemplo:

AI Tokens
1,000,000
$0.01 / 1K
$10
116. Cost Allocation Model
CostAllocation
│
├── id
├── source_cost_id
├── target_type
├── target_id
├── allocation_rule_id
├── allocated_amount
├── percentage
└── metadata
117. Margin Model
MarginSnapshot
│
├── period
├── revenue
├── cost_of_revenue
├── gross_profit
├── gross_margin
├── contribution_cost
├── contribution_profit
└── contribution_margin
118. Budget Model
CostBudget
│
├── id
├── period
├── organization_id
├── cost_center_id
├── category
├── budget
├── actual
├── forecast
├── variance
└── status
119. Optimization Model
OptimizationOpportunity
│
├── id
├── category
├── description
├── current_cost
├── estimated_savings
├── implementation_cost
├── payback_period
├── confidence
├── priority
├── owner
├── status
└── metadata
120. Cost API
Costs
GET /billing/finops/costs
GET /billing/finops/costs/{id}
Allocation
GET  /billing/finops/allocations
POST /billing/finops/allocations
Budgets
GET  /billing/finops/budgets
POST /billing/finops/budgets
PATCH /billing/finops/budgets/{id}
121. Margin API
GET /billing/finops/margins
GET /billing/finops/margins/customers
GET /billing/finops/margins/products
GET /billing/finops/margins/tenants
122. Optimization API
GET  /billing/finops/opportunities
POST /billing/finops/opportunities
POST /billing/finops/opportunities/{id}/approve
POST /billing/finops/opportunities/{id}/implement
POST /billing/finops/opportunities/{id}/close
123. FinOps Alerts API
GET /billing/finops/alerts
POST /billing/finops/policies
PATCH /billing/finops/policies/{id}
124. Events

Eventos principales:

CostRecorded
CostAllocated
BudgetCreated
BudgetExceeded
CostAnomalyDetected
OptimizationIdentified
OptimizationApproved
OptimizationImplemented
SavingsRealized
MarginThresholdBreached
125. Event Architecture
Cloud Provider
      │
Payment Provider
      │
AI Provider
      │
Internal Systems
      │
      ▼
Cost Events
      │
      ▼
Cost Normalization
      │
      ▼
Cost Allocation
      │
      ▼
FinOps Engine
      │
 ┌────┼─────┐
 ▼    ▼     ▼
Budget Margin Optimization
126. Cost Normalization

Diferentes proveedores pueden usar:

USD
EUR
CLP
Credits
Tokens
Units

EVOXA debe normalizar:

Source Amount
Source Currency
FX Rate
Normalized Amount
Functional Currency
127. Multi-Currency Cost

Ejemplo:

Cloud Cost
USD 10,000


FX
950


Functional Cost
CLP 9,500,000

Debe conservarse tanto:

Original Currency

como:

Functional Currency
128. Vendor Cost Management

Debe existir:

Vendor
Contract
Pricing
Commitment
Usage
Invoice
Actual Cost
129. Vendor Model
Vendor
│
├── id
├── name
├── category
├── contract
├── billing_account
├── currency
├── payment_terms
├── status
└── metadata
130. Vendor Contract

Debe almacenar:

Contract Start
Contract End
Committed Spend
Discount
Pricing
Minimum Commitment
Renewal Date
131. Cloud Commitments

FinOps puede analizar:

Committed Spend
vs
Actual Usage

Ejemplo:

Commitment
$1M


Actual Usage
$700K

Potential waste:

$300K
132. Reserved Capacity

Puede analizar:

Reserved Capacity
Utilization
Effective Rate
On-Demand Rate
Savings
133. Effective Cost

Un recurso puede tener:

List Price
$100


Discount
-$20


Actual
$80

FinOps debe utilizar el costo efectivo para los análisis apropiados.

134. Cost Optimization Categories
RIGHTSIZE
REMOVE
SCHEDULE
RESERVE
NEGOTIATE
REDESIGN
CACHE
ARCHIVE
AUTOMATE
CHANGE_PROVIDER
CHANGE_MODEL
135. AI Model Optimization

Ejemplo:

Current Model
Cost $100K


Alternative Model
Cost $40K


Quality Difference
2%

Opportunity:

Potential Saving
$60K

La decisión debe considerar calidad, latencia, seguridad y requisitos funcionales.

136. AI Routing

Puede existir:

Request
   ↓
Complexity
   ↓
Model Router
   ├── Small Model
   ├── Medium Model
   └── Large Model

Esto optimiza:

Cost
Latency
Quality
137. Cost-Aware AI

EVOXA puede imponer:

Cost Budget

por:

Tenant
Agent
Workflow
Feature
Model

Ejemplo:

Tenant A
AI Budget
$10K/month
138. Cost Guardrail for AI
AI Cost
$9K


Forecast
$12K

Alert:

Projected AI Budget Overrun
139. Cost per Agent
Agent
Billing Assistant


Executions
100K


LLM Cost
$5K


Tools
$2K


Compute
$1K


Total
$8K


Cost / Execution
$0.08
140. Agent Profitability

Si el Agent genera:

Revenue
$50K

y cuesta:

$8K

entonces:

Gross Contribution
$42K
141. Cost Optimization AI

La IA puede detectar:

"Tenant A generates 30% of AI cost
but only 8% of AI revenue."

y recomendar:

Review Pricing
or
Optimize AI Usage
142. Cost Intelligence

La arquitectura puede evolucionar:

Cost Analytics
      ↓
Cost Intelligence
      ↓
Predictive Cost
      ↓
Prescriptive FinOps
143. Predictive Cost
Current Cost
$1M


Projected
$1.3M


Expected Driver:
Usage +25%
144. Prescriptive FinOps

En lugar de:

Cost increased

EVOXA puede indicar:

Recommended Action:


Reduce idle compute
Estimated saving:
$25K/month


Confidence:
91%
145. Automated Optimization

En acciones de bajo riesgo puede existir:

Detect
   ↓
Validate
   ↓
Approve Policy
   ↓
Execute
   ↓
Measure

Ejemplo:

Non-production resource
Idle > 7 days

podría programarse para apagado automático si existe una política explícita.

146. Optimization Approval

Acciones críticas:

Change Architecture
Change Provider
Change AI Model
Resize Production

deben requerir aprobación.

147. Savings Verification

Después de optimizar:

Before
$100K


After
$80K

Savings:

$20K

pero debe comprobarse que:

Revenue
Performance
Reliability
Quality

no hayan sufrido efectos negativos inaceptables.

148. Optimization Impact

Cada iniciativa debe medir:

Cost Saving
Revenue Impact
Performance Impact
Reliability Impact
Customer Impact
Implementation Cost
149. FinOps Dashboard
Total Cost
$5M


Cost / Revenue
32%


Gross Margin
68%


Budget Variance
+$120K


Unallocated Cost
4%


Optimization Opportunities
37


Potential Savings
$800K


Realized Savings
$350K
150. Executive FinOps Dashboard
Revenue                 $15M
Cost                     $5M
Gross Profit            $10M
Gross Margin              67%


Cloud Cost               $2M
Payment Cost              $1M
AI Cost                  $800K
Support Cost              $500K
Other                     $700K
151. Cost by Revenue

Una vista fundamental:

Revenue
$15M


Cost
$5M


Gross Margin
66.7%

y por segmento:

Product A
Margin 80%


Product B
Margin 55%


Product C
Margin 35%
152. Cost Efficiency

Métricas:

Cost / Revenue
Cost / Customer
Cost / Transaction
Cost / Invoice
Cost / Payment
Cost / Usage Unit
Cost / AI Request
Cost / Agent Execution
153. Efficiency Trend
January
Cost / Revenue
35%


February
33%


March
31%


April
29%

Esto indica:

Improving Cost Efficiency
154. Revenue Efficiency

Puede medirse:

Revenue
÷
Operating Cost

o métricas específicas como:

Revenue / Cloud Dollar
Revenue / Support Dollar
Revenue / Employee

según el modelo financiero.

155. Rule of 40 Integration

Para SaaS, EVOXA puede mostrar:

Revenue Growth %
+
Profit Margin %

Ejemplo:

Growth
30%


Margin
20%


Rule of 40
50%

Debe considerarse una métrica de referencia, no una regla universal.

156. Unit Economics Dashboard
ARPU
$500


Cost-to-Serve
$120


Gross Profit
$380


Gross Margin
76%


LTV
$10K


CAC
$2K


LTV/CAC
5x
157. Cost Risk
Cost Risk
│
├── Cloud
├── AI
├── Payment Fees
├── Vendor Dependency
├── FX
├── Usage Volatility
└── Capacity
158. Vendor Concentration

Ejemplo:

Cloud Provider A
80%


Provider B
15%


Provider C
5%

Esto puede generar:

Vendor Concentration Risk
159. Cost Scenario — Cloud
Current
$2M


Usage +20%
→ $2.4M


Optimization -10%
→ $2.16M
160. Cost Scenario — Payment
Current Payment Cost
$1M


Optimization
-15%


Projected
$850K


Saving
$150K
161. Cost Scenario — AI
Current AI Cost
$500K


Model Optimization
-20%


Caching
-10%


Routing
-15%


Potential
$275K

Las reducciones no deben sumarse ciegamente si se superponen; el motor debe modelar dependencias.

162. FinOps Data Lineage
Provider Invoice
      ↓
Cost Record
      ↓
Cost Allocation
      ↓
Customer Cost
      ↓
Margin
      ↓
Dashboard
163. Auditability

Cada allocation debe registrar:

Source
Rule
Driver
Formula
Amount
Period
Version
Created By
Created At
164. Cost Rule Versioning

Si cambia:

Allocation Rule v1

a:

Allocation Rule v2

los períodos históricos deben conservar la versión utilizada originalmente.

165. Financial Period Integration

FinOps debe respetar:

Accounting Period
Fiscal Period
Billing Period
Cost Period

y permitir conciliación.

166. Cost Closing

Proceso:

Collect Provider Costs
      ↓
Normalize
      ↓
Validate
      ↓
Allocate
      ↓
Reconcile
      ↓
Calculate Margin
      ↓
Close Cost Period
167. FinOps Close Dashboard
Cloud Costs       ✓
Payment Costs     ✓
AI Costs          ✓
Vendor Costs      ✓
Allocations       ✓
Reconciliation    ✓
Budget Variance   ✓
Margin            ✓
168. FinOps Permissions
finops.view
finops.manage
finops.allocate
finops.budget.manage
finops.forecast
finops.optimize
finops.approve
finops.export
finops.admin
169. Separation of Duties
Engineering
→ Optimize Infrastructure


Finance
→ Validate Cost


FinOps
→ Analyze / Recommend


Management
→ Approve Strategic Optimization


Accounting
→ Record Financial Impact
170. FinOps and Revenue Intelligence

Los dos módulos deben conectarse:

Revenue Intelligence
        │
        ▼
Revenue
        │
        ├─────────────┐
        │             │
        ▼             ▼
      Cost         Margin
        │             │
        └──────┬──────┘
               ▼
        Revenue Economics
               │
               ▼
         Optimization
171. Revenue + Cost Intelligence

Ejemplo:

Revenue
+$500K


Cost
+$400K


Net Economic Impact
+$100K

Aunque Revenue haya crecido mucho:

Margin Increment
20%

puede ser insuficiente.

172. Growth vs Margin

Dashboard:

Product A
Growth +30%
Margin 80%


Product B
Growth +50%
Margin 30%


Product C
Growth +10%
Margin 90%

Esto ayuda a determinar dónde invertir.

173. Customer Economics Matrix
                MARGIN
                  ↑
                  │
     Strategic    │    Ideal
                  │
──────────────────┼──────────────→ Revenue
                  │
     Review       │    High Cost
                  │
174. Cost Optimization Loop
Measure
   ↓
Attribute
   ↓
Analyze
   ↓
Detect Waste
   ↓
Prioritize
   ↓
Optimize
   ↓
Measure Savings
   ↓
Repeat

Esto convierte FinOps en un proceso continuo.

175. AI FinOps Assistant

Un usuario podría preguntar:

"¿Dónde estamos gastando más?"

EVOXA:

Cloud represents 42% of total cost.


Top drivers:
Compute 45%
AI 30%
Database 15%
Network 10%

Otra consulta:

"¿Qué podemos optimizar?"

Respuesta:

1. AI model routing
   Potential saving $120K/year


2. Idle compute
   Potential saving $80K/year


3. Payment provider routing
   Potential saving $50K/year
176. AI Cost Explainability

Toda recomendación debe indicar:

Recommendation
Estimated Saving
Calculation
Confidence
Risk
Affected Customers
Affected Services
177. AI Cost Governance

La IA no debería poder:

Delete production resources
Change billing prices
Change accounting entries
Cancel contracts

por sí sola.

Puede:

Detect
Analyze
Recommend
Simulate

y ejecutar únicamente acciones autorizadas por políticas.

178. FinOps Automation

Ejemplo:

IF
Non-production resource
idle > 7 days


AND
owner confirmed


THEN
schedule shutdown

Otro:

IF
Payment Provider Cost
> threshold


THEN
generate routing optimization recommendation
179. Optimization Automation
Opportunity
   ↓
Policy Check
   ↓
Risk Check
   ↓
Approval
   ↓
Execution
   ↓
Verification
   ↓
Savings Recorded
180. Core Principles
1. Revenue Is Not Profit

Facturar más no significa necesariamente ganar más.

2. Every Cost Needs Attribution

Debe saberse quién o qué genera el costo.

3. Shared Costs Need Transparent Allocation

Las reglas deben ser explícitas y auditables.

4. Unit Economics Matter

El costo por Customer, Product, Payment y AI Request es fundamental.

5. FinOps Is Continuous

No es un reporte mensual; es un ciclo permanente.

6. Optimization Must Be Measured

Una recomendación no es un ahorro hasta que se implementa y verifica.

7. Cost Optimization Cannot Break Revenue

Reducir costos que destruyen Revenue o Customer Experience puede ser contraproducente.

8. Forecast Before Overrun

La plataforma debe detectar futuros sobrecostos antes de que ocurran.

9. AI Should Recommend Before It Acts

Las acciones críticas requieren políticas y aprobación.

10. Margin Is the Ultimate Connection
Revenue
-
Cost
=
Profit
181. Resultado de 26/22

Billing FinOps & Cost Optimization convierte EVOXA en una plataforma capaz de entender la economía completa de cada operación.

No solamente:

Customer
→
Revenue

sino:

Customer
   │
   ├── Revenue
   │
   ├── Infrastructure Cost
   ├── AI Cost
   ├── Payment Cost
   ├── Support Cost
   ├── Collection Cost
   ├── Refund Cost
   └── Other Costs
          │
          ▼
       Margin
          │
          ▼
     Profitability
          │
          ▼
     Optimization
182. Arquitectura completa 26/08 → 26/22
26/08  Billing Products & Catalog
          ↓
26/09  Billing Pricing & Plans
          ↓
26/10  Billing Subscriptions
          ↓
26/11  Billing Usage & Metering
          ↓
26/12  Billing Rating & Charge Calculation
          ↓
26/13  Billing Invoicing
          ↓
26/14  Billing Payments
          ↓
26/15  Billing Credits, Discounts & Adjustments
          ↓
26/16  Billing Taxes & Compliance
          ↓
26/17  Billing Revenue Management
          ↓
26/18  Billing Accounting & Financial Integration
          ↓
26/19  Billing Dunning & Collections
          ↓
26/20  Billing Refunds, Adjustments & Disputes
          ↓
26/21  Billing Analytics & Revenue Intelligence
          ↓
26/22  Billing FinOps & Cost Optimization

Ahora tenemos una arquitectura económica mucho más potente:

                         BILLING
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
           REVENUE                      COST
              │                           │
              ▼                           ▼
       Revenue Analytics             FinOps
              │                           │
              └─────────────┬─────────────┘
                            ▼
                          MARGIN
                            │
                   ┌────────┴────────┐
                   ▼                 ▼
              Profitability      Optimization
                   │                 │
                   └────────┬────────┘
                            ▼
                         AI
                            │
                            ▼
                    Recommendations

Y a nivel de Customer:

Customer
   │
   ├── Revenue
   ├── MRR / ARR
   ├── Payments
   ├── Collections
   ├── Refunds
   ├── Disputes
   │
   ├── Infrastructure Cost
   ├── AI Cost
   ├── Payment Cost
   ├── Support Cost
   └── Operational Cost
             │
             ▼
       Customer Margin
             │
             ▼
       Customer LTV
             │
             ▼
      Revenue Intelligence

26/22 establece la capa FinOps de EVOXA: atribuye costos a Customers, Tenants, Products, Plans, Features, AI Agents y servicios; calcula Cost-to-Serve, Gross Margin y Unit Economics; controla presupuestos y forecasts; detecta desperdicios y anomalías; identifica oportunidades de ahorro; y conecta Revenue con Cost para determinar la verdadera rentabilidad del ecosistema.

Siguiente capítulo
26/23 — Billing Ecosystem & Partner Management

Aquí entraremos en la capa de ecosistema externo de Billing, donde EVOXA podrá administrar:

Payment Providers;
Tax Providers;
ERP Providers;
Accounting Providers;
Cloud Providers;
AI Providers;
Data Providers;
Resellers;
Distributors;
Marketplaces;
Billing Partners;
Referral Partners;
Revenue Share;
Partner Commissions;
Partner Pricing;
Partner Contracts;
Partner Settlements;
Partner Billing;
Partner Reconciliation;
Partner APIs;
Partner Portal;
Ecosystem Governance.

La arquitectura empezará a pasar de un Billing Platform interno a un verdadero Billing Ecosystem Platform, preparado para múltiples proveedores, partners y modelos de Revenue Share.
