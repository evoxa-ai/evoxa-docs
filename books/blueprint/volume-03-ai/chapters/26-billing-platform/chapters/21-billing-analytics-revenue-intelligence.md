1. Billing Analytics & Revenue Intelligence

Billing Analytics & Revenue Intelligence es la capa analítica que transforma todos los datos generados por Billing en información financiera, comercial, operacional y predictiva.

Hasta ahora EVOXA puede:

Catalog
   ↓
Pricing
   ↓
Subscriptions
   ↓
Usage
   ↓
Rating
   ↓
Invoicing
   ↓
Payments
   ↓
Revenue
   ↓
Accounting
   ↓
Collections
   ↓
Refunds / Adjustments / Disputes

Ahora necesitamos responder:

¿Qué está pasando con el Revenue, por qué está pasando y qué probablemente ocurrirá después?

2. Objetivo

Esta capa debe proporcionar:

Billing Analytics;
Revenue Analytics;
Revenue Intelligence;
Customer Analytics;
Subscription Analytics;
Payment Analytics;
AR Analytics;
Collections Analytics;
MRR;
ARR;
ACV;
ARPU;
LTV;
Churn;
Retention;
Expansion Revenue;
Contraction Revenue;
New Revenue;
Reactivation Revenue;
Revenue Forecasting;
Cash Collection Forecasting;
Cohort Analysis;
Revenue Waterfalls;
Variance Analysis;
Revenue Leakage Detection;
Billing Leakage Detection;
Pricing Intelligence;
Customer Segmentation;
Profitability Analytics;
AI Insights.
3. Posición dentro de Billing
26/08  Products & Catalog
26/09  Pricing & Plans
26/10  Subscriptions
26/11  Usage & Metering
26/12  Rating & Charge Calculation
26/13  Invoicing
26/14  Payments
26/15  Credits, Discounts & Adjustments
26/16  Taxes & Compliance
26/17  Revenue Management
26/18  Accounting & Financial Integration
26/19  Dunning & Collections
26/20  Refunds, Adjustments & Disputes
26/21  Analytics & Revenue Intelligence

La información de todos estos módulos converge aquí.

4. Billing Analytics vs Revenue Intelligence

Son conceptos relacionados pero diferentes.

Billing Analytics

Describe:

¿Qué ocurrió?

Ejemplo:

Invoices
$5M


Payments
$4.2M


Outstanding
$800K
Revenue Intelligence

Busca responder:

¿Por qué ocurrió y qué debería hacer EVOXA?

Ejemplo:

Revenue Growth
↓ 8%


Primary Driver:
Enterprise churn


Expected Impact:
-$350K next quarter
5. Analytics Architecture
                    BILLING PLATFORM
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
         Billing DB      Events       Accounting
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                     Data Platform
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                Data Warehouse   Data Lake
                    │
                    ▼
               Semantic Layer
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
      Dashboards  Reports    AI
          │         │         │
          └─────────┼─────────┘
                    ▼
             Revenue Intelligence
6. Analytics Layers

EVOXA debería separar:

Operational Data
       ↓
Analytical Data
       ↓
Metrics
       ↓
Insights
       ↓
Predictions
       ↓
Recommendations
7. Operational Data

Proviene directamente de:

Customers
Invoices
Payments
Subscriptions
Usage
Products
Plans
Credits
Refunds
Disputes
Collections
Revenue
Accounting
8. Analytical Data

No conviene ejecutar todos los dashboards directamente sobre las tablas transaccionales.

Debe existir una capa analítica:

OLTP
 ↓
CDC / Events
 ↓
ETL / ELT
 ↓
Data Warehouse

Esto protege el rendimiento del Billing Core.

9. Data Warehouse

Ejemplo:

EVOXA Billing Warehouse
│
├── Fact Invoices
├── Fact Payments
├── Fact Usage
├── Fact Revenue
├── Fact Collections
├── Fact Refunds
├── Fact Disputes
└── Fact Subscriptions
10. Dimensions
Dim Customer
Dim Product
Dim Plan
Dim Subscription
Dim Tenant
Dim Organization
Dim Legal Entity
Dim Date
Dim Currency
Dim Region
Dim Sales Channel
11. Star Schema
                   Dim Customer
                        │
                        │
Dim Product ─── Fact Revenue ─── Dim Date
                        │
                        │
                   Dim Tenant
                        │
                        │
                    Dim Plan

Esto facilita análisis rápidos.

12. Fact Invoice
FactInvoice
│
├── invoice_id
├── customer_id
├── tenant_id
├── product_id
├── plan_id
├── date_id
├── subtotal
├── discount
├── tax
├── total
├── currency
├── status
└── days_to_payment
13. Fact Payment
FactPayment
│
├── payment_id
├── customer_id
├── invoice_id
├── date_id
├── amount
├── currency
├── payment_method
├── provider
├── status
├── processing_fee
└── settlement_amount
14. Fact Subscription
FactSubscription
│
├── subscription_id
├── customer_id
├── plan_id
├── start_date
├── end_date
├── MRR
├── ARR
├── status
├── acquisition_channel
└── churn_date
15. Fact Revenue
FactRevenue
│
├── revenue_id
├── customer_id
├── contract_id
├── subscription_id
├── product_id
├── date_id
├── recognized_amount
├── deferred_amount
├── currency
└── revenue_type
16. Fact Collections
FactCollection
│
├── case_id
├── customer_id
├── invoice_id
├── amount_due
├── amount_recovered
├── days_past_due
├── recovery_date
├── collection_channel
└── outcome
17. Metric Layer

EVOXA debe tener una definición centralizada de métricas.

Por ejemplo:

MRR
ARR
Revenue
Net Revenue
Gross Revenue
AR
DSO
Churn
Retention
LTV
CAC

Cada métrica debe tener:

Definition
Formula
Dimensions
Currency
Time Grain
Owner
Version
18. Metric Governance

Esto evita tener:

Finance MRR = $10M
Sales MRR = $9.7M
Product MRR = $10.3M

La plataforma debe tener:

One Metric Definition

19. Revenue

Debe distinguir:

Gross Revenue
Net Revenue
Recognized Revenue
Billed Revenue
Collected Revenue
Deferred Revenue

No son intercambiables.

20. Gross Revenue

Conceptualmente:

Gross Revenue
=
Total Revenue Before Discounts / Adjustments

La definición exacta debe quedar gobernada por la política financiera de EVOXA.

21. Net Revenue

Conceptualmente:

Gross Revenue
-
Discounts
-
Credits
-
Refunds
-
Adjustments
=
Net Revenue

La definición debe ser consistente con Accounting.

22. Billed Revenue

Representa:

Amount Billed

No necesariamente:

Revenue Recognized

porque puede existir Revenue Deferred.

23. Collected Revenue

Una métrica operacional puede representar:

Cash Collected

pero:

Cash Collected
≠
Accounting Revenue

Esto es fundamental.

24. MRR

Monthly Recurring Revenue representa Revenue recurrente mensualizado.

Ejemplo:

Customer A
$100


Customer B
$200


Customer C
$700


MRR
$1,000
25. MRR Components

EVOXA debe separar:

New MRR
Expansion MRR
Contraction MRR
Churned MRR
Reactivation MRR
26. MRR Movement
Opening MRR
+
New
+
Expansion
+
Reactivation
-
Contraction
-
Churn
=
Ending MRR

Ejemplo:

Opening
$1M


New
+$100K


Expansion
+$50K


Contraction
-$20K


Churn
-$30K


Ending
$1.1M
27. ARR

Annual Recurring Revenue:

ARR ≈ MRR × 12

cuando esa definición sea apropiada para el modelo de negocio.

Ejemplo:

MRR
$1M


ARR
$12M

Debe existir una definición oficial en el Metric Catalog.

28. New MRR

Revenue recurrente generado por nuevos Customers:

Customer
New Subscription
$500 MRR

Entonces:

New MRR
+$500
29. Expansion MRR

Customer existente aumenta su consumo:

Old
$1,000


New
$1,500


Expansion
+$500
30. Contraction MRR

Customer reduce:

Old
$1,500


New
$1,000


Contraction
-$500
31. Churned MRR

Customer cancela:

MRR
$1,000


Cancellation
-$1,000
32. Reactivation MRR

Customer vuelve:

Previously Churned

y luego:

New Subscription
$700

Esto se clasifica como:

Reactivation MRR
+$700
33. Net New MRR
New
+
Expansion
+
Reactivation
-
Contraction
-
Churn
34. MRR Waterfall

Una visualización fundamental:

Opening MRR
     │
     ├── New
     ├── Expansion
     ├── Reactivation
     ├── Contraction
     └── Churn
     │
     ▼
Ending MRR
35. ARR Waterfall

Igualmente:

Opening ARR
   ↓
New ARR
   ↓
Expansion ARR
   ↓
Contraction ARR
   ↓
Churn ARR
   ↓
Ending ARR
36. ARPU

Average Revenue Per User/Account:

Revenue
÷
Average Customers

La definición puede adaptarse a:

User
Account
Customer
Subscription

según el negocio.

37. ACV

Annual Contract Value puede representar:

Contract Value
÷
Contract Years

Ejemplo:

3-year contract
$300K


ACV
$100K
38. LTV

Customer Lifetime Value puede estimarse utilizando:

ARPU
Retention
Gross Margin
Customer Lifetime

La fórmula exacta debe definirse según el modelo financiero.

39. CAC

Aunque CAC puede pertenecer a una plataforma comercial/marketing más amplia, Billing Analytics puede consumirlo para calcular:

LTV / CAC
40. LTV / CAC

Ejemplo:

LTV
$12,000


CAC
$3,000


LTV/CAC
4.0x
41. Churn

Debe distinguir:

Customer Churn
Revenue Churn
Logo Churn
Subscription Churn
42. Logo Churn
Customers Lost
÷
Customers at Start
43. Revenue Churn

Puede medir:

Churned MRR
÷
Opening MRR
44. Gross Revenue Retention

GRR mide cuánto Revenue recurrente se conserva sin contar Expansion.

Conceptualmente:

Opening MRR
-
Contraction
-
Churn

dividido por Opening MRR.

45. Net Revenue Retention

NRR incorpora Expansion:

Opening MRR
+
Expansion
+
Reactivation
-
Contraction
-
Churn

dividido por Opening MRR.

Ejemplo:

Opening
$1M


Ending from existing cohort
$1.1M


NRR
110%
46. Retention Analytics
Customer Cohort
       ↓
Month 0
Month 1
Month 2
Month 3
...

Esto permite medir retención real.

47. Cohort Analysis

Ejemplo:

2026-Q1 Cohort


Month 1
98%


Month 3
92%


Month 6
85%


Month 12
75%
48. Revenue Cohorts

También:

Customer Cohort
   ↓
Revenue over time

Esto permite descubrir si los Customers adquiridos en distintos períodos tienen comportamientos diferentes.

49. Product Revenue Analytics
Product A
$2M


Product B
$1.5M


Product C
$500K

Pero además:

Growth
Margin
Churn
Refund Rate
Expansion
50. Plan Analytics

Por plan:

Basic
MRR $500K
Churn 5%


Pro
MRR $2M
Churn 3%


Enterprise
MRR $5M
Churn 1%
51. Pricing Intelligence

La plataforma debe analizar:

Price
Discount
Conversion
Expansion
Churn
Refund

para identificar:

Price Sensitivity
52. Discount Analytics
Gross Billings
$10M


Discounts
-$1M


Discount Rate
10%

Analizar por:

Product
Plan
Customer Segment
Sales Channel
Region
Sales Rep
53. Discount Leakage

Puede existir:

Discount
10%


Expected
5%

Entonces:

Discount Leakage
5%
54. Billing Leakage

Revenue esperado:

$10M

Revenue facturado:

$9.5M

Potential leakage:

$500K
55. Billing Leakage Sources
Missing Usage
Incorrect Rating
Incorrect Pricing
Failed Invoice
Incorrect Discount
Unbilled Contract
Data Integration Failure
56. Revenue Leakage

Puede aparecer entre:

Contract
   ↓
Usage
   ↓
Charge
   ↓
Invoice
   ↓
Revenue

Ejemplo:

Usage
$100K


Billed
$90K

Potential leakage:

$10K
57. Leakage Detection Engine
Expected
   ↓
Actual
   ↓
Variance
   ↓
Threshold
   ↓
Anomaly
58. Revenue Variance

Comparar:

Forecast
vs
Actual

Ejemplo:

Forecast
$5M


Actual
$4.7M


Variance
-$300K
59. Variance Analysis

Debe explicar:

Variance
-$300K


Drivers:
Churn        -$120K
Contraction  -$80K
Discounts    -$40K
Collections  -$30K
Other        -$30K

Esto transforma un número en una explicación.

60. Revenue Forecasting

Puede proyectar:

Next Month
$5.2M


Next Quarter
$16M


Next Year
$68M

basándose en:

Subscriptions
Pipeline inputs
Usage
Pricing
Renewals
Churn
Collections
Seasonality
61. Forecast Models

EVOXA puede soportar:

Baseline Forecast
Driver-Based Forecast
Statistical Forecast
AI Forecast
Scenario Forecast
62. Scenario Forecasting
Base
Growth
10%
Upside
Growth
18%
Downside
Growth
3%
63. Revenue Scenario Model
                 Revenue
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
        Downside   Base    Upside
         $50M      $60M     $70M
64. Customer Health

Billing puede contribuir a un Customer Health Score:

Payment History
+
Usage
+
Expansion
+
Support Signals
+
Renewal

Resultado:

Healthy
At Risk
Critical
65. Revenue Risk

Customer:

MRR
$50K


Past Due
$20K


Usage
-30%


Support Issues
High

Revenue Risk:

HIGH
66. Churn Prediction

Modelo:

Customer
   │
   ├── Usage ↓
   ├── Payments ↓
   ├── Tickets ↑
   ├── Engagement ↓
   └── Renewal Soon
          │
          ▼
      Churn Risk
67. Expansion Opportunity

Lo mismo al contrario:

Customer
   │
   ├── Usage ↑
   ├── Seat Count ↑
   ├── Feature Adoption ↑
   └── Capacity Near Limit
          │
          ▼
   Expansion Opportunity
68. Revenue Intelligence Engine
                   BILLING DATA
                        │
                        ▼
                  Metric Engine
                        │
                        ▼
                  Analytics Engine
                        │
              ┌─────────┼─────────┐
              ▼         ▼         ▼
          Anomalies   Risks   Opportunities
              │         │         │
              └─────────┼─────────┘
                        ▼
                 AI Intelligence
                        │
                        ▼
                 Recommendations
69. AI Revenue Insights

Ejemplo:

Revenue increased 12% this month.


Primary drivers:
+8% Expansion
+5% New Customers
-1% Churn

Otro:

Collections decreased 7%.


Primary driver:
Increase in 60+ day AR.
70. AI Anomaly Detection

Puede detectar:

MRR suddenly drops
Refunds spike
Payment failures increase
Discounts increase
AR aging worsens
Usage decreases
Revenue diverges from forecast
71. AI Root Cause Analysis
Revenue ↓ 8%
       ↓
AI Analysis
       ↓
Enterprise Churn
       ↓
Top 5 Customers
       ↓
$600K ARR at Risk
72. AI Recommendation

Puede producir:

Recommendation


Prioritize renewal intervention
for 5 enterprise accounts.


Potential ARR protected:
$600K

Esto debe ser presentado como recomendación, no como una acción irreversible automática.

73. Revenue Alerts

Alertas:

MRR Drop > 5%
ARR Drop > 3%
Refund Rate > 10%
Payment Failure > 8%
AR 90+ > Threshold
Churn > Threshold
Revenue Forecast Variance > Threshold
74. Alert Engine
Metric
   ↓
Threshold
   ↓
Condition
   ↓
Alert
   ↓
Notification
   ↓
Action
75. Revenue Dashboard

Dashboard ejecutivo:

Revenue
$12.5M


MRR
$1.2M


ARR
$14.4M


NRR
108%


Gross Churn
3.2%


AR
$1.8M


DSO
41 days


Collection Rate
94%
76. Revenue Executive Dashboard

Debe permitir:

Current
vs
Previous Period
vs
Budget
vs
Forecast
vs
Prior Year
77. Revenue by Dimension
Revenue
│
├── Organization
├── Tenant
├── Customer
├── Product
├── Plan
├── Region
├── Currency
├── Industry
├── Sales Channel
└── Legal Entity
78. Revenue by Geography
Chile
$4M


USA
$5M


Mexico
$2M


Other
$1.5M
79. Revenue by Customer Segment
SMB
$2M


Mid-Market
$3M


Enterprise
$7.5M
80. Customer Concentration

Debe medir:

Top 1
Top 5
Top 10
Top 20

Ejemplo:

Top 10 Customers
45% of ARR

Esto representa riesgo de concentración.

81. Revenue Concentration Risk
Customer A
20%


Customer B
15%


Customer C
10%

Puede generar:

Concentration Risk
HIGH
82. Contract Analytics

Analizar:

Contract Value
Contract Duration
Renewal Date
Expansion
Discount
Payment Terms
Revenue Recognition
83. Renewal Analytics
Renewals Next 90 Days
$4M ARR


At Risk
$800K


Expected Renewal
$3.2M
84. Renewal Forecast
Contract
$100K ARR


Risk
Low


Expected Renewal
92%
85. Renewal Pipeline
Upcoming Renewal
        ↓
Health
        ↓
Risk
        ↓
Expansion
        ↓
Forecast
86. Revenue Retention Dashboard
GRR
92%


NRR
108%


Logo Retention
95%


Revenue Churn
3%
87. Usage-to-Revenue Analytics

Especialmente importante para modelos usage-based:

Usage
   ↓
Rated Usage
   ↓
Charges
   ↓
Invoice
   ↓
Revenue

Analytics debe comparar cada etapa.

88. Usage Revenue Efficiency

Ejemplo:

Usage
10M units


Revenue
$500K


Revenue / Unit
$0.05

Esto permite detectar cambios de monetización.

89. Usage Anomaly
Usage
+40%


Revenue
+5%

Puede indicar:

Pricing Issue
Rating Issue
Discount Issue
Billing Leakage
90. Invoice Analytics

Debe analizar:

Invoice Volume
Average Invoice
Invoice Size
Invoice Aging
Invoice Error Rate
Invoice Delivery
Time to Payment
91. Invoice Accuracy
Invoices
100,000


Correct
99,200


Error Rate
0.8%

Y clasificar:

Pricing
Tax
Usage
Customer
Product
92. Payment Analytics
Payment Success Rate
Retry Success
Decline Rate
Refund Rate
Chargeback Rate
Processing Fee
Settlement Time
93. Payment Success Funnel
Payment Attempt
100%


Approved
92%


Captured
91%


Settled
90%


Recovered Net
89%
94. Payment Method Analytics
Card
Success 94%


ACH / Bank
Success 97%


Wallet
Success 91%

La disponibilidad depende de los métodos soportados por cada mercado y proveedor.

95. Payment Failure Analytics
Insufficient Funds
35%


Expired Card
20%


Bank Decline
15%


Network
10%


Other
20%
96. Collection Analytics

Conecta con 26/19:

AR
$2M


Past Due
$300K


90+
$80K


Recovered
$220K
97. Refund Analytics

Conecta con 26/20:

Refunds
$250K


Refund Rate
2.1%


Top Reason
Billing Error
98. Dispute Analytics
Disputes
1,200


Win Rate
78%


Chargeback Rate
0.6%
99. Revenue Bridge

Una de las vistas más importantes:

Opening ARR
$10M


New
+$2M


Expansion
+$1M


Contraction
-$500K


Churn
-$700K


Ending ARR
$11.8M
100. Revenue Waterfall
              ARR
               │
        ┌──────┼──────┐
        ▼      ▼      ▼
       New   Expansion Reactivation
        │      │      │
        └──────┼──────┘
               │
        Contraction
               │
             Churn
               │
               ▼
          Ending ARR
101. Revenue Forecast Architecture
Historical Data
       +
Current Billing
       +
Subscriptions
       +
Usage
       +
Collections
       +
Churn
       +
Seasonality
       │
       ▼
Forecast Engine
       │
       ▼
Revenue Forecast
102. Forecast Confidence

Debe mostrar:

Forecast
$20M


Confidence
82%


Range
$19M – $21M

No presentar predicciones como certezas.

103. Forecast Accuracy

Comparar:

Forecast
$10M


Actual
$9.7M


Variance
-$300K

y medir:

MAPE
Bias
Forecast Error
104. Revenue Scenario Planning

Permitir modificar:

Churn
+2%


Expansion
-5%


New Business
+10%


Price
+3%

y calcular:

Projected ARR
105. Pricing Scenario

Ejemplo:

Current Price
$100


Scenario
+10%


Expected Churn
+1%


Expected Revenue
+8%

Esto permite evaluar cambios de pricing.

106. Revenue Optimization

La plataforma puede identificar:

Underpriced Customers
High Discount Customers
High Usage / Low Revenue
Expansion Opportunities
At-Risk Revenue
107. Customer Revenue Segmentation
High Value / Low Risk
High Value / High Risk
Low Value / Low Risk
Low Value / High Risk

Esto permite priorizar acciones.

108. Revenue Opportunity Matrix
                VALUE
                  ↑
                  │
      Expansion   │  Strategic
       Targets    │  Accounts
                  │
──────────────────┼──────────────→ Risk
                  │
       Monitor    │  Recovery
                  │  Required
109. Revenue Intelligence Actions

Las recomendaciones pueden ser:

Review Pricing
Contact Customer
Offer Upgrade
Investigate Billing Error
Review Payment Method
Prioritize Collection
Review Contract
Investigate Usage Anomaly
110. Analytics Permissions
analytics.view
analytics.export
analytics.manage
metrics.manage
forecast.view
forecast.manage
revenue_intelligence.view
revenue_intelligence.manage
111. Data Access Control

La analítica debe respetar:

Organization
Tenant
Legal Entity
Business Unit

Ejemplo:

Tenant A
→ only Tenant A analytics


Organization Admin
→ consolidated analytics
112. Row-Level Security

Puede existir:

User
   ↓
Role
   ↓
Scope
   ↓
Data Access

Esto es crítico en un Enterprise Multi-Tenant.

113. Analytics Export

Debe soportar:

CSV
Excel
PDF
API
Scheduled Reports
114. Scheduled Reports

Ejemplo:

Every Monday
Revenue Report


Every Month
Financial Billing Report


Every Quarter
Revenue Intelligence Report
115. Executive Reporting

Reporte:

Revenue
MRR
ARR
NRR
GRR
Churn
AR
DSO
Collections
Refunds
Disputes
Forecast
116. Real-Time vs Batch Analytics

No todas las métricas requieren tiempo real.

Real-Time
Payment Failure
AR Alerts
Fraud
Critical Revenue Anomaly
Near Real-Time
MRR
Collections
Revenue Dashboard
Batch
Financial Statements
Historical Cohorts
Monthly Close
Forecast
117. Event-Driven Analytics
InvoiceIssued
PaymentSucceeded
SubscriptionCanceled
UsageRecorded
RefundProcessed
ChargebackReceived

→ Analytics Event Stream.

118. Analytics Event Model
AnalyticsEvent
│
├── event_id
├── event_type
├── timestamp
├── tenant_id
├── customer_id
├── source_id
├── amount
├── currency
└── metadata
119. Data Quality

Revenue Intelligence solo es buena si los datos son confiables.

Debe existir:

Completeness
Accuracy
Consistency
Uniqueness
Timeliness
Validity
120. Data Quality Rules

Ejemplo:

Invoice
Total
=
Subtotal
+
Tax
-
Discount

y:

MRR
=
Sum Active Recurring Subscriptions
121. Metric Reconciliation
Billing Revenue
$10M


Analytics Revenue
$10M


Accounting Revenue
$10M

Debe existir:

Variance = $0

o una excepción explicada.

122. Revenue Data Lineage

Debe poder seguirse:

Dashboard
   ↓
Metric
   ↓
Data Model
   ↓
Fact Table
   ↓
Transaction
   ↓
Source System

Esto será esencial para auditoría y confianza.

123. Metric Versioning

Si cambia una fórmula:

MRR v1

a:

MRR v2

el sistema debe conservar ambas definiciones históricas.

Nunca cambiar silenciosamente una métrica histórica.

124. Revenue Intelligence Governance

Cada insight debe tener:

Metric Source
Calculation
Confidence
Timestamp
Model Version
Explanation
125. AI Explainability

En vez de:

Revenue Risk: HIGH

debe mostrar:

Revenue Risk: HIGH


Drivers:
- Usage down 28%
- Payment failures +15%
- Renewal in 45 days
- Past due balance $25K
126. AI Confidence
Churn Probability
72%


Confidence
84%

La confianza del modelo debe estar separada de la probabilidad estimada.

127. Revenue Intelligence Assistant

EVOXA puede incorporar un asistente:

"¿Por qué cayó el Revenue este mes?"

Respuesta esperada:

Revenue decreased 4.2%.


Main drivers:
1. Enterprise churn: -$180K
2. Contraction: -$90K
3. Billing adjustments: -$40K


Offset:
New Revenue +$120K.
128. Natural Language Analytics

Consultas:

"What is our MRR?"


"Why did collections decline?"


"Which customers are at risk?"


"Which products have the highest refund rate?"


"What is the ARR forecast?"


"Which customers expanded this quarter?"
129. AI Guardrails

El asistente debe:

respetar permisos;
no exponer datos de otros tenants;
explicar fuentes;
distinguir hechos de predicciones;
mostrar período analizado;
mostrar definición de métricas;
no ejecutar acciones financieras sin autorización.
130. Revenue Intelligence API
Metrics
GET /billing/analytics/metrics
GET /billing/analytics/metrics/{metric}
Revenue
GET /billing/analytics/revenue
GET /billing/analytics/revenue/waterfall
GET /billing/analytics/revenue/forecast
MRR
GET /billing/analytics/mrr
GET /billing/analytics/mrr/movements
Customers
GET /billing/analytics/customers
GET /billing/analytics/customers/cohorts
131. Forecast API
POST /billing/analytics/forecast
GET  /billing/analytics/forecast
POST /billing/analytics/scenarios
GET  /billing/analytics/scenarios
132. Insights API
GET /billing/intelligence/insights
GET /billing/intelligence/anomalies
GET /billing/intelligence/risks
GET /billing/intelligence/opportunities
133. Alert API
GET  /billing/analytics/alerts
POST /billing/analytics/alerts/rules
PATCH /billing/analytics/alerts/rules/{id}
134. Analytics Architecture
                   BILLING CORE
                        │
                        ▼
                 EVENT STREAM
                        │
                        ▼
                 DATA PIPELINE
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
         Data Warehouse       Data Lake
              │
              ▼
         Semantic Layer
              │
       ┌──────┼───────┐
       ▼      ▼       ▼
    Metrics Dashboards Forecast
       │      │       │
       └──────┼───────┘
              ▼
      REVENUE INTELLIGENCE
              │
       ┌──────┼──────┐
       ▼      ▼      ▼
     Risks  Trends Opportunities
              │
              ▼
             AI
135. Billing Analytics Domain Model
Analytics
│
├── Metrics
├── Dashboards
├── Reports
├── Cohorts
├── Revenue
├── MRR
├── ARR
├── Retention
├── Churn
├── Forecasts
├── Scenarios
├── Alerts
├── Anomalies
├── Risks
└── Opportunities
136. Revenue Intelligence Domain Model
RevenueIntelligence
│
├── Insight
├── Anomaly
├── Risk
├── Opportunity
├── Recommendation
├── Forecast
├── Scenario
└── ModelVersion
137. Insight Model
Insight
│
├── id
├── type
├── title
├── description
├── severity
├── confidence
├── metric
├── period
├── source
├── created_at
└── metadata
138. Anomaly Model
Anomaly
│
├── id
├── metric
├── expected_value
├── actual_value
├── variance
├── severity
├── detected_at
├── status
└── explanation
139. Opportunity Model
RevenueOpportunity
│
├── id
├── customer_id
├── opportunity_type
├── estimated_value
├── probability
├── confidence
├── reason
├── status
└── metadata
140. Risk Model
RevenueRisk
│
├── id
├── customer_id
├── risk_type
├── amount_at_risk
├── probability
├── severity
├── drivers
├── detected_at
└── status
141. Forecast Model
RevenueForecast
│
├── id
├── period
├── metric
├── forecast_value
├── lower_bound
├── upper_bound
├── confidence
├── model_version
└── generated_at
142. Scenario Model
RevenueScenario
│
├── id
├── name
├── assumptions
├── projected_revenue
├── projected_arr
├── projected_mrr
├── confidence
└── status
143. Core Metrics Catalog

EVOXA debería tener un catálogo oficial:

Revenue
MRR
ARR
New MRR
Expansion MRR
Contraction MRR
Churn MRR
Reactivation MRR
NRR
GRR
ARPU
ACV
LTV
CAC
DSO
AR
Past Due AR
Collection Rate
Refund Rate
Chargeback Rate
144. Executive KPI Tree
                    REVENUE
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
         MRR           ARR        Growth
          │
     ┌────┼─────┐
     ▼    ▼     ▼
    New Expansion Churn
          │
          ▼
         NRR
145. Financial KPI Tree
Revenue
   │
   ├── AR
   │    └── DSO
   │
   ├── Collections
   │
   ├── Refunds
   │
   └── Bad Debt
146. Revenue Risk Tree
ARR
 │
 ├── Churn Risk
 ├── Collection Risk
 ├── Payment Risk
 ├── Contract Risk
 ├── Pricing Risk
 └── Billing Risk
147. Revenue Opportunity Tree
ARR
 │
 ├── Expansion
 ├── Cross-Sell
 ├── Upsell
 ├── Price Optimization
 ├── Usage Monetization
 └── Reactivation
148. Revenue Intelligence Operating Loop
Measure
  ↓
Understand
  ↓
Predict
  ↓
Recommend
  ↓
Act
  ↓
Measure Again

Esta es la evolución natural de EVOXA hacia una plataforma verdaderamente inteligente.

149. Core Principles
1. One Metric Definition

Cada KPI debe tener una definición oficial.

2. Source Traceability

Todo número debe poder rastrearse hasta su fuente.

3. Operational vs Analytical Separation

El Billing Core no debe depender de consultas analíticas pesadas.

4. Historical Consistency

Los cambios de definición deben versionarse.

5. Multi-Tenant Security

Los datos analíticos deben respetar los límites de Tenant y Organization.

6. Explainable Intelligence

Las recomendaciones de IA deben explicar sus drivers.

7. Forecast ≠ Fact

Una predicción debe diferenciarse claramente de un dato histórico.

8. Revenue ≠ Cash

Revenue, Billing, Collections y Cash deben mantenerse separados.

9. Actionable Analytics

El objetivo no es solo mostrar dashboards, sino detectar acciones.

10. Continuous Intelligence

El sistema debe evolucionar desde reporting hacia inteligencia predictiva.

150. Resultado de 26/21

Billing Analytics & Revenue Intelligence transforma EVOXA desde un sistema que simplemente procesa Billing en un sistema capaz de entender, explicar y anticipar el comportamiento financiero y comercial.

La cadena completa:

Billing Transactions
        ↓
Data Platform
        ↓
Metrics
        ↓
Analytics
        ↓
Insights
        ↓
Forecasts
        ↓
Risks
        ↓
Opportunities
        ↓
Recommendations

Y la visión más avanzada:

                  EVOXA BILLING
                       │
                       ▼
                  DATA PLATFORM
                       │
                       ▼
                  ANALYTICS
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
        DESCRIPTIVE  PREDICTIVE  PRESCRIPTIVE
          │            │            │
       What?         What next?    What should
                                   we do?
          │            │            │
          └────────────┼────────────┘
                       ▼
              REVENUE INTELLIGENCE
                       │
                       ▼
                     AI
Arquitectura acumulada 26/08 → 26/21
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

Ahora la arquitectura ya tiene un circuito completo:

PRODUCT
   ↓
PRICE
   ↓
SUBSCRIPTION
   ↓
USAGE
   ↓
RATING
   ↓
CHARGE
   ↓
INVOICE
   ↓
TAX
   ↓
PAYMENT
   ↓
REVENUE
   ↓
ACCOUNTING
   ↓
COLLECTION
   ↓
REFUND / ADJUSTMENT / DISPUTE
   ↓
ANALYTICS
   ↓
INTELLIGENCE
   ↓
FORECAST
   ↓
RECOMMENDATION

Y esto abre la puerta al siguiente nivel:

26/22 — Billing FinOps & Cost Optimization

Donde ya no solo analizaremos cuánto factura EVOXA, sino cuánto cuesta generar, procesar, cobrar y operar ese Revenue, incluyendo costos de infraestructura, payment processing fees, cloud consumption, cost allocation, unit economics, gross margin, cost per customer, cost per transaction, FinOps, chargeback/showback, optimization y automatización de costos.
