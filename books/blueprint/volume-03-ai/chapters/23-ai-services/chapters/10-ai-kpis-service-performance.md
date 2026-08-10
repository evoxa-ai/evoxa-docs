---
document_id: BP-0003-C23-10
chapter_id: CH-03-23-10
volume: Volume 03 — Artificial Intelligence Platform
parent_chapter: 23 — AI Services
title: AI KPIs & Service Performance
version: 1.0.0
status: Approved
owner: Enterprise AI Platform Office
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-09
---

# 10 — AI KPIs & Service Performance

> *The AI KPIs & Service Performance framework establishes the enterprise measurement system used to evaluate the reliability, performance, quality, security, cost efficiency, operational health and business value of AI Services across the EVOXA ecosystem.*

---

# Executive Summary

AI Services require a performance model that goes beyond traditional infrastructure and application metrics.

A service may have:

- High availability
- Low infrastructure latency
- Low error rates

and still provide poor business value because:

- The model produces inaccurate responses.
- Retrieval returns irrelevant information.
- Agents fail to complete tasks.
- Token consumption is excessive.
- Costs exceed the business case.
- Users do not trust the results.
- Security controls block legitimate workloads.
- Model quality degrades over time.

Therefore EVOXA defines AI Service Performance as a multidimensional framework.

```text
Reliability

+

Performance

+

AI Quality

+

Security

+

Cost Efficiency

+

User Experience

+

Business Value

=

AI Service Performance
Performance Vision

The vision is:

Create a measurable AI ecosystem where every production AI Service has clearly defined performance expectations, measurable outcomes, accountable owners and continuous improvement mechanisms.

Performance Mission

The mission is to ensure that AI Services:

Meet their SLOs
Deliver expected quality
Operate within approved cost
Maintain security
Provide acceptable user experience
Deliver business value
Continuously improve
KPI Objectives

The KPI framework must provide:

Operational visibility
Performance measurement
Quality measurement
Cost transparency
Security measurement
Business outcome measurement
Executive reporting
Service comparison
Trend analysis
Continuous improvement
KPI Principles

The EVOXA AI KPI framework follows:

Measurable
Actionable
Consistent
Comparable
Traceable
Business-Aligned
Risk-Aware
Automated
Continuous
Evidence-Based
KPI Hierarchy

KPIs are organized into multiple levels.

Enterprise AI KPIs

        ↓

Portfolio KPIs

        ↓

Service KPIs

        ↓

Model KPIs

        ↓

Component KPIs

        ↓

Request-Level Metrics
AI KPI Domains

The enterprise framework includes:

1. Availability
2. Reliability
3. Performance
4. API
5. Model
6. Quality
7. RAG
8. Agent
9. Security
10. Cost
11. Capacity
12. User Experience
13. Business Value
14. Governance
15. Provider
16. Sustainability
AI Performance Model
                  AI SERVICE PERFORMANCE
                           │
       ┌───────────────────┼───────────────────┐
       │                   │                   │
   Reliability          Quality              Cost
       │                   │                   │
       ├── Availability    ├── Accuracy        ├── Cost/Request
       ├── Errors          ├── Relevance       ├── Cost/User
       └── SLO             ├── Groundedness    └── Budget
                           └── Safety
       │                   │
       └───────────────────┼───────────────────┘
                           │
                     Business Value
KPI vs Metric

A metric is a measurable value.

A KPI is a metric that has strategic or operational significance.

Example:

Metric:
Total Tokens = 10,000,000

KPI:
Cost per Successful AI Task

Not every metric should become a KPI.

KPI Classification

KPIs should be classified as:

Strategic
Tactical
Operational
Technical
Financial
Risk
Quality
Strategic KPIs

Strategic KPIs measure:

AI Adoption
Business Value
ROI
Portfolio Health
Risk
Cost
Tactical KPIs

Tactical KPIs measure:

Service Performance
Model Performance
Provider Performance
Quality
Cost Efficiency
Operational KPIs

Operational KPIs measure:

Availability
Latency
Errors
Tokens
Incidents
Capacity
KPI Ownership

Every KPI requires:

KPI Owner
Data Owner
Calculation Definition
Data Source
Target
Threshold
Review Frequency
KPI Definition Standard

Every KPI should have:

KPI ID

Name

Description

Purpose

Formula

Unit

Data Source

Owner

Target

Warning Threshold

Critical Threshold

Frequency

Scope

Data Classification
KPI Example
kpi:
  id: AI-KPI-001
  name: AI Service Availability
  description: Percentage of time the service is available.
  unit: percentage
  target: ">=99.95%"
  frequency: monthly
  owner: service-owner
KPI Lifecycle
Define

↓

Approve

↓

Implement

↓

Measure

↓

Analyze

↓

Report

↓

Improve

↓

Review
Availability KPIs

Availability is one of the foundational AI Service KPIs.

Primary metrics:

Availability
Uptime
Successful Requests
Failed Requests
Dependency Availability
Availability KPI
Availability =
Successful Service Time
/
Total Service Time
× 100
Availability Target

Typical critical AI services may target:

≥99.95%

The actual target must be defined by the service SLA/SLO.

Request Success Rate
Success Rate =
Successful Requests
/
Total Requests
× 100

Target:

≥99.9%

for critical services where technically and commercially appropriate.

Error Rate
Error Rate =
Failed Requests
/
Total Requests
× 100
Error Budget

AI services should use error budgets.

Error Budget =
Allowed Failure
-
Actual Failure
AI Error Budget

Error budgets may include:

Availability
Latency
Quality

This expands traditional SRE practices into AI reliability.

Reliability KPI

Reliability measures the ability of the AI Service to perform successfully over time.

Key indicators:

Availability
Failure Rate
MTBF
MTTR
SLO Compliance
MTBF

Mean Time Between Failures measures the average operational period between significant failures.

MTBF =
Operating Time
/
Number of Failures
MTTR

Mean Time to Recovery measures how quickly service is restored.

MTTR =
Total Recovery Time
/
Number of Incidents
Reliability Target

Critical AI Services should establish:

Availability SLO
Error Rate SLO
MTTR Target
Incident Response Target
Performance KPIs

Core performance metrics include:

Latency
P50
P95
P99
TTFT
TTLT
Throughput
Concurrent Requests
Latency

Latency measures the time required to process an AI request.

Latency =
Response Completion Time
-
Request Start Time
Latency Percentiles

AI Services should measure:

P50
P75
P90
P95
P99

P95 and P99 are particularly important for identifying poor tail performance.

P95 Latency KPI

Example target:

P95 < Service-Specific Threshold

The threshold must be defined according to the AI Service's workload and user experience requirements.

Time to First Token

For streaming AI:

TTFT =
First Token Timestamp
-
Request Timestamp

TTFT is a critical conversational UX KPI.

Time to Last Token
TTLT =
Final Token Timestamp
-
Request Timestamp
Tokens Per Second
Tokens / Second =
Generated Tokens
/
Generation Time

Useful for model and provider performance comparison.

Throughput

Throughput may be measured as:

Requests per Second
Requests per Minute
Tokens per Second
Completed Tasks per Hour
Concurrency

Monitor:

Active Requests
Concurrent Users
Concurrent Model Calls
Concurrent Agent Sessions
Capacity KPIs

Capacity KPIs include:

Maximum Concurrent Requests
Token Throughput
Provider Quota Utilization
Infrastructure Utilization
Queue Depth
Capacity Utilization
Capacity Utilization =
Current Usage
/
Available Capacity
× 100
Capacity Thresholds

Example:

70% → Normal

80% → Warning

90% → High

95% → Critical

Thresholds must be configurable.

API KPIs

API performance includes:

Requests
Success Rate
Error Rate
Latency
Rate Limit Events
Authentication Failures
Throttling
API Success Rate
API Success Rate =
Successful API Requests
/
Total API Requests
× 100
API Availability

API availability should be monitored independently from model availability.

API

↓

Gateway

↓

AI Service

↓

Model

↓

Provider

A provider outage does not necessarily mean the API infrastructure itself is unavailable.

API Throttling

Monitor:

Rate Limit Hits
Quota Exceeded
Retry Rate
Throttle Duration
Retry Rate
Retry Rate =
Retried Requests
/
Total Requests
× 100

Excessive retries can indicate dependency instability.

Model KPIs

Model KPIs measure:

Usage
Quality
Latency
Cost
Availability
Error Rate
Model Utilization
Model Utilization =
Model Requests
/
Total AI Requests
× 100
Model Quality Score

A model quality score may combine:

Accuracy
Relevance
Groundedness
Safety
Task Success

The scoring methodology must be defined for each use case.

Model Error Rate
Model Error Rate =
Model Failures
/
Model Calls
× 100
Model Cost Efficiency
Model Cost Efficiency =
Successful High-Quality Tasks
/
Model Cost
Model Comparison

Models should be compared using a common evaluation framework.

                    Model A    Model B    Model C

Quality              92%        95%        89%

P95 Latency          1.2s       1.8s       0.9s

Cost/Request         $0.010     $0.020     $0.006

Safety                98%        99%        96%
Provider KPIs

Provider KPIs include:

Availability
Latency
Error Rate
Quota Utilization
Rate Limits
Cost
Failover Frequency
Provider Availability
Provider Availability =
Available Provider Time
/
Total Provider Time
× 100
Provider Error Rate
Provider Error Rate =
Provider Errors
/
Provider Requests
× 100
Provider Failover Rate
Failover Rate =
Failover Requests
/
Total Requests
× 100

A high failover rate may indicate provider instability.

Provider Concentration KPI
Provider Concentration =
Requests to Largest Provider
/
Total Requests
× 100

This KPI supports resilience and concentration-risk management.

Token KPIs

Token metrics are fundamental AI operational KPIs.

Measure:

Input Tokens
Output Tokens
Total Tokens
Tokens per Request
Tokens per User
Tokens per Service
Total Token Consumption
Total Tokens =
Input Tokens
+
Output Tokens
Average Tokens per Request
Average Tokens =
Total Tokens
/
Total Requests
Token Efficiency
Token Efficiency =
Successful Tasks
/
Total Tokens
Token Growth
Token Growth =
(Current Tokens - Previous Tokens)
/
Previous Tokens
× 100

Sudden increases should trigger investigation.

Context Efficiency
Context Efficiency =
Useful Context
/
Total Context

Where measurable.

AI Quality KPIs

Quality KPIs are among the most important AI-specific measures.

Key indicators include:

Accuracy
Relevance
Groundedness
Completeness
Helpfulness
Safety
Task Success
Hallucination Rate
Accuracy

Accuracy measures whether the output is correct for the defined task.

Relevance

Relevance measures whether the response addresses the requested task.

Groundedness

Groundedness measures whether generated content is supported by available evidence.

Hallucination Rate

Where a reliable evaluation methodology exists:

Hallucination Rate =
Unsupported Outputs
/
Evaluated Outputs
× 100

This metric is use-case dependent.

Task Success Rate
Task Success Rate =
Successfully Completed Tasks
/
Total Tasks
× 100

This is one of the most valuable operational AI KPIs.

AI Quality Score

A composite score may be constructed:

AI Quality Score

=

Accuracy
+

Relevance
+

Groundedness
+

Safety
+

Task Success

Weighting must be defined by the service.

Quality Thresholds

Example:

≥95% → Excellent

90–94% → Good

80–89% → Warning

<80% → Critical

Thresholds must be calibrated using real evaluation data.

Quality Regression

A quality regression occurs when:

Current Quality
<
Approved Baseline
Quality Regression KPI
Quality Regression Rate =
Regression Events
/
Evaluated Releases
× 100
Evaluation Coverage
Evaluation Coverage =
Evaluated AI Interactions
/
Total AI Interactions
× 100

Critical services should target high evaluation coverage.

RAG KPIs

RAG systems require specialized KPIs.

Measure:

Retrieval Success
Retrieval Latency
Precision
Recall
Relevance
Groundedness
Citation Accuracy
Source Freshness
Retrieval Success Rate
Retrieval Success =
Successful Retrievals
/
Total Retrieval Requests
× 100
Retrieval Latency
Retrieval Latency =
Retrieval Completion
-
Retrieval Start
Retrieval Precision
Precision =
Relevant Retrieved Documents
/
Total Retrieved Documents
Retrieval Recall
Recall =
Relevant Retrieved Documents
/
Total Relevant Documents

Where a suitable ground truth exists.

Grounded Response Rate
Grounded Response Rate =
Grounded Responses
/
Evaluated Responses
× 100
Citation Accuracy

Where citations are generated:

Citation Accuracy =
Correct Citations
/
Total Evaluated Citations
× 100
Knowledge Freshness KPI
Knowledge Freshness =
Current Knowledge Date
-
Source Update Date

The acceptable freshness window depends on the business use case.

Agent KPIs

Agent systems require specialized measurements.

Measure:

Task Completion
Tool Success
Tool Failure
Execution Time
Number of Steps
Number of Tool Calls
Human Intervention
Cost
Loop Detection
Agent Task Success
Agent Task Success =
Successful Agent Tasks
/
Total Agent Tasks
× 100
Agent Completion Rate

Measures the percentage of tasks completed without failure or escalation.

Agent Escalation Rate
Escalation Rate =
Human Escalations
/
Total Agent Tasks
× 100
Agent Tool Success Rate
Tool Success Rate =
Successful Tool Calls
/
Total Tool Calls
× 100
Agent Steps

Track:

Average Steps / Task
Maximum Steps / Task
P95 Steps / Task
Agent Execution Time
Execution Time =
Agent Completion
-
Agent Start
Agent Loop Rate
Loop Rate =
Detected Loops
/
Total Agent Executions
× 100
Agent Human Intervention
Human Intervention Rate =
Human Interventions
/
Total Agent Executions
× 100
Security KPIs

AI security performance includes:

Security Events
Prompt Injection Attempts
Jailbreak Attempts
DLP Events
Unauthorized Requests
Tool Violations
Credential Events
Prompt Injection Rate
Prompt Injection Rate =
Detected Injection Attempts
/
Total Requests
× 100
Policy Violation Rate
Policy Violation Rate =
Policy Violations
/
Total Requests
× 100
DLP Block Rate
DLP Block Rate =
Blocked DLP Requests
/
DLP Evaluated Requests
× 100
Security Incident Rate
Security Incident Rate =
AI Security Incidents
/
AI Requests
× 100
Security Response KPIs

Measure:

MTTD
MTTR
Containment Time
Resolution Time
MTTD

Mean Time to Detect:

MTTD =
Detection Time
-
Incident Start
Security MTTR
Security MTTR =
Recovery Time
-
Incident Detection
Cost KPIs

AI cost management requires granular measurement.

Measure:

Total Cost
Cost per Request
Cost per User
Cost per Service
Cost per Model
Cost per Provider
Cost per Successful Task
Total AI Cost
Total AI Cost =
Model Costs
+
Provider Costs
+
Infrastructure
+
Additional AI Services
Cost per Request
Cost / Request =
Total AI Cost
/
Total Requests
Cost per User
Cost / User =
Total AI Cost
/
Active AI Users
Cost per Successful Task
Cost / Successful Task =
Total AI Cost
/
Successful Tasks

This is generally more meaningful than cost per request.

Cost Efficiency
Cost Efficiency =
Business Value
/
AI Cost
Budget Variance
Budget Variance =
Actual Cost
-
Approved Budget
Budget Variance Percentage
Budget Variance % =
(Actual Cost - Budget)
/
Budget
× 100
Cost Forecast Accuracy
Forecast Accuracy =
1 -
|Forecast - Actual|
/
Actual
Cost Anomaly Rate
Cost Anomaly Rate =
Detected Cost Anomalies
/
Measurement Periods
User Experience KPIs

AI Services must measure user experience.

Possible KPIs:

User Satisfaction
Response Time
Completion Rate
Abandonment Rate
Escalation Rate
Feedback Score
User Satisfaction

Possible measurement:

CSAT =
Positive Responses
/
Total Responses
× 100
AI Feedback Score

Feedback can be normalized into a score.

Positive Feedback
-
Negative Feedback

The methodology should be standardized per product.

Abandonment Rate
Abandonment Rate =
Abandoned Sessions
/
Total Sessions
× 100
Business KPIs

AI performance must ultimately connect to business outcomes.

Examples:

Revenue
Conversion
Cost Savings
Productivity
Automation
Resolution
Retention
Customer Satisfaction
AI Business Value
AI Business Value =
Incremental Benefit
-
AI Operating Cost
AI ROI
ROI =
(Net AI Benefit)
/
AI Investment
× 100
AI Productivity
Productivity Gain =
Baseline Processing Time
-
AI Processing Time
Automation Rate
Automation Rate =
AI-Completed Tasks
/
Total Eligible Tasks
× 100
Human Deflection Rate

For support or service use cases:

Deflection Rate =
Tasks Resolved by AI
/
Total Eligible Tasks
× 100
Business Outcome Rate
Business Outcome Rate =
Successful Business Outcomes
/
Eligible AI Interactions
× 100
KPI Correlation

Technical KPIs should be connected to business KPIs.

Latency

↓

User Satisfaction

↓

Task Completion

↓

Business Outcome
KPI Dependency Model
Infrastructure

↓

API

↓

AI Gateway

↓

Model

↓

AI Quality

↓

User Experience

↓

Business Outcome
KPI Scorecards

Each AI Service should maintain a scorecard.

AI SERVICE SCORECARD

Availability        99.97%    🟢
P95 Latency          1.8s     🟢
Quality              94.2%    🟢
Groundedness         96.1%    🟢
Error Rate            0.4%    🟢
Cost/Request        $0.012    🟡
Security Events          2    🟢
Task Success         91.4%    🟢
KPI Status

Recommended status:

🟢 Green  = Target Met

🟡 Yellow = Warning

🔴 Red    = Critical

⚪ Gray   = No Data
KPI Thresholds

Every KPI should define:

Target

Warning Threshold

Critical Threshold

Example:

Availability

Target:   ≥99.95%

Warning:  <99.90%

Critical: <99.50%
KPI Trend

KPIs should be evaluated over time.

Current

↓

7 Days

↓

30 Days

↓

90 Days

↓

Year over Year
KPI Trend Analysis

Analyze:

Improving
Stable
Degrading
Volatile
Unknown
KPI Forecasting

Predictive analytics can forecast:

Cost
Capacity
Quality
Demand
Reliability
Performance Baseline

Every production service must establish a baseline.

Baseline dimensions:

Availability
Latency
Quality
Cost
Tokens
Volume
Baseline Establishment
Production Launch

↓

Observe

↓

Collect Baseline

↓

Validate

↓

Approve

↓

Monitor
Performance Regression

A regression occurs when performance deviates materially from the approved baseline.

Potential areas:

Latency
Cost
Quality
Availability
Throughput
Regression Detection
Baseline

↓

Current Performance

↓

Comparison

↓

Threshold

↓

Alert
Release Performance

Every AI release should evaluate:

Availability
Latency
Quality
Cost
Security
Release Comparison
                 Before     After

Quality           93%        95%
P95 Latency       1.8s       1.6s
Cost/Request     $0.014      $0.012
Error Rate        0.6%        0.4%
Canary Performance

During canary deployments:

Version A

vs

Version B

Compare:

Quality
Latency
Cost
Error Rate
Business Outcomes
Performance Gates

Production promotion requires:

Quality ≥ Threshold

AND

Latency ≤ Threshold

AND

Error Rate ≤ Threshold

AND

Security = Passed

AND

Cost ≤ Approved Limit
SLI Framework

Service Level Indicators measure actual service behavior.

Core AI SLIs:

Availability
Latency
Error Rate
TTFT
Quality
Groundedness
Task Success
SLO Framework

Service Level Objectives define desired performance.

Example:

Availability ≥99.95%

P95 Latency <2 seconds

Task Success ≥90%

Groundedness ≥95%

Actual targets are service-specific.

SLA Framework

Service Level Agreements define contractual commitments.

SLA

↓

Availability

+

Performance

+

Support

+

Incident Response
SLI → SLO → SLA
SLI

What happened?

↓

SLO

What do we target?

↓

SLA

What do we commit to?
AI Error Budget

The difference between the SLO and actual performance creates the available error budget.

SLO = 99.95%

Actual = 99.97%

↓

Healthy Error Budget
Quality Error Budget

AI Services may also establish quality budgets.

Quality SLO = 95%

Actual Quality = 97%

↓

Quality Budget Available
Performance Governance

Performance must be reviewed regularly.

Review:

KPI Trends
SLO Compliance
Cost
Quality
Incidents
Capacity
KPI Review Frequency
Level	Frequency
Operational	Real Time
Service	Daily
Management	Weekly
Executive	Monthly
Strategic	Quarterly
KPI Reporting

Reporting should exist at:

Request

↓

Service

↓

Portfolio

↓

Enterprise
Service-Level Dashboard

Each service should display:

Availability
Latency
Errors
Quality
Tokens
Cost
Security
Task Success
Portfolio Dashboard

The AI portfolio dashboard should display:

Service Count
Service Health
Model Count
Provider Count
Cost
Quality
Risk
Adoption
Executive Dashboard

Executive reporting should focus on:

AI Value

AI Cost

AI Risk

AI Quality

AI Reliability

AI Adoption
AI Performance Index

EVOXA may define a composite performance index.

AI Performance Index

=

Reliability Score

+

Quality Score

+

Performance Score

+

Security Score

+

Cost Efficiency Score

+

Business Value Score

Weights must be defined by governance and validated before executive use.

Example AI Performance Index
Reliability        95
Quality            92
Performance        90
Security           98
Cost Efficiency    87
Business Value     94

Weighted Score     93.0

The score must never replace the underlying KPIs.

KPI Anti-Patterns

Avoid:

Too Many KPIs
Unclear Definitions
No Owner
No Target
No Data Source
Manual Measurement
Metrics Without Action
Vanity Metrics
Vanity KPI

Example:

1,000,000 AI Requests

This does not necessarily mean success.

A better KPI:

Successful High-Quality Tasks
KPI Actionability

Every important KPI should answer:

What happened?

Why?

What is the impact?

Who owns it?

What should happen next?
KPI Automation

KPIs should be generated automatically whenever possible.

Telemetry

↓

Data Pipeline

↓

KPI Calculation

↓

Dashboard

↓

Alert

↓

Action
KPI Data Sources

Potential sources include:

AI Gateway
API Management
AI Services
Model Registry
Provider APIs
Observability Platform
Evaluation Platform
Security Platform
FinOps Platform
Business Systems
KPI Data Quality

KPI calculations depend on data quality.

Monitor:

Completeness
Accuracy
Timeliness
Consistency
KPI Data Lineage

Every KPI should be traceable to its source.

KPI

↓

Calculation

↓

Dataset

↓

Telemetry

↓

Source System
KPI Reproducibility

A KPI must be reproducible from the underlying data and documented calculation.

KPI Versioning

If a KPI formula changes:

Version the KPI
Document the change
Preserve historical values
Communicate impact
KPI Governance

Every KPI requires governance.

Governance determines:

Definition
Owner
Target
Threshold
Review
Exception
KPI Exceptions

Exceptions require:

Reason
Owner
Approval
Expiration
Compensating Action
Performance Improvement

KPI results must feed continuous improvement.

Measure

↓

Analyze

↓

Identify Gap

↓

Improve

↓

Measure Again
AI Performance Optimization

Optimization areas include:

Model Selection
Prompt Optimization
Context Optimization
Retrieval Optimization
Caching
Routing
Provider Selection
Infrastructure
Latency Optimization

Potential strategies:

Model Selection
Streaming
Caching
Prompt Reduction
Retrieval Optimization
Provider Routing
Cost Optimization

Potential strategies:

Model Routing
Token Reduction
Caching
Batch Processing
Smaller Models
Prompt Optimization
Quality Optimization

Potential strategies:

Better Models
Better Prompts
Better Retrieval
Better Evaluation
Better Context
Human Feedback
Multi-Objective Optimization

AI performance must balance competing objectives.

Quality

vs

Latency

vs

Cost

vs

Safety

Optimizing one dimension may negatively affect another.

AI Optimization Model
                    Quality
                       ▲
                       │
                       │
Cost ◄─────────────────┼─────────────────► Latency
                       │
                       │
                       ▼
                     Safety
AI Performance Trade-Offs

Examples:

Larger Model
→ Higher Quality
→ Higher Cost
→ Higher Latency

Smaller Model
→ Lower Cost
→ Lower Latency
→ Potentially Lower Quality

The routing system should optimize according to service policy.

Model Routing KPI
Routing Efficiency =
Successful Tasks
/
Total Model Cost
Routing Quality

Measure:

Correct Model Selection
Quality
Latency
Cost
Failover
Routing Decision Success
Routing Success =
Requests Meeting Policy Objectives
/
Total Routed Requests
× 100
AI FinOps KPI Model
Usage

↓

Cost

↓

Allocation

↓

Efficiency

↓

Optimization

↓

Savings
AI Cost Savings
Cost Savings =
Baseline Cost
-
Optimized Cost
Cost Avoidance

Measure avoided cost resulting from:

Better Routing
Caching
Smaller Models
Token Optimization
AI Sustainability KPIs

Where infrastructure data is available, AI services may track:

Compute Consumption
Energy Consumption
Carbon Estimate
Energy Efficiency
Energy / Successful Task
Carbon Efficiency
Estimated Carbon /
Successful AI Task

Sustainability measurements require reliable infrastructure and provider data.

Governance KPIs

Governance-related KPIs include:

Governance Coverage
Risk Assessment Coverage
Model Approval Coverage
Provider Assessment Coverage
Policy Compliance
Exception Rate
Governance Coverage
Governed Services
/
Total AI Services
× 100
Model Approval Coverage
Approved Models
/
Production Models
× 100

Target:

100%
Provider Governance Coverage
Approved Providers
/
Production Providers
× 100
Security KPI Coverage
Monitored Security Controls
/
Required Security Controls
× 100
Observability KPI Coverage
Observable Services
/
Total Production AI Services
× 100

Target:

100% Critical Services
Business KPI Framework

Business KPIs must be selected according to the purpose of each AI Service.

Examples:

AI Use Case	KPI
Customer Support	Resolution Rate
Sales	Conversion
Knowledge Assistant	Task Success
Developer Assistant	Productivity
Automation	Automation Rate
Recommendation	Conversion
Analytics	Decision Accuracy
KPI-to-Outcome Mapping
Technical KPI

↓

AI KPI

↓

User KPI

↓

Business KPI

Example:

Latency

↓

Response Completion

↓

User Satisfaction

↓

Customer Retention
AI Service Performance Scorecard

Each critical AI Service should have:

SERVICE PERFORMANCE

Reliability
├── Availability
├── Error Rate
└── MTTR

Performance
├── P95
├── P99
└── TTFT

Quality
├── Accuracy
├── Groundedness
└── Task Success

Security
├── Violations
├── DLP
└── Incidents

Cost
├── Cost/Request
├── Cost/User
└── Budget

Business
├── Outcome
├── Productivity
└── ROI
KPI Benchmarking

AI Services may be benchmarked by:

Model
Provider
Business Unit
Service
Use Case

Benchmarking must compare equivalent workloads.

Internal Benchmarking
Service A

vs

Service B

vs

Enterprise Baseline
External Benchmarking

Where reliable comparable data exists, services may be compared against:

Industry Benchmarks
Provider Benchmarks
Internal Targets

External benchmarks should not replace enterprise-specific targets.

KPI Maturity Model
Level	Description
Level 1	Basic Metrics
Level 2	Standard KPIs
Level 3	Integrated Scorecards
Level 4	Predictive KPIs
Level 5	Adaptive Performance
Level 1 — Basic Metrics

Services measure:

Requests
Errors
Latency
Level 2 — Standard KPIs

Services measure:

Availability
Cost
Tokens
Quality
Level 3 — Integrated Scorecards

Technical, AI and business KPIs are combined.

Level 4 — Predictive KPIs

The platform forecasts:

Cost
Capacity
Quality
Reliability
Level 5 — Adaptive Performance

The platform dynamically optimizes:

Models
Providers
Routing
Cost
Performance

according to approved policies.

Target State
Level 5

Adaptive AI Performance
Adaptive Performance

The platform may dynamically select the optimal configuration.

Request

↓

Context

+

Risk

+

Quality

+

Latency

+

Cost

↓

Optimization

↓

Model / Provider

↓

Result
Performance Intelligence

Future AI capabilities may identify:

Performance Bottlenecks
Quality Regression
Cost Optimization
Capacity Risk
Model Selection Opportunities
Predictive Performance
Historical Data

↓

Performance Model

↓

Forecast

↓

Risk

↓

Preventive Action
Autonomous Optimization

The long-term target is a controlled optimization loop.

Measure

↓

Analyze

↓

Optimize

↓

Deploy

↓

Validate

All automated changes must remain within governance and security boundaries.

AI KPI Architecture
                         AI Services
                              │
                              ▼
                         Telemetry
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
        Metrics             Events             Evaluations
          │                   │                   │
          └───────────────────┼───────────────────┘
                              │
                              ▼
                       KPI Data Platform
                              │
                              ▼
                       KPI Calculation
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
       Operational          AI Quality          Business
          │                   │                   │
          └───────────────────┼───────────────────┘
                              │
                              ▼
                       Scorecards
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
      Operations          Management          Executive
KPI Data Pipeline
Source

↓

Collection

↓

Validation

↓

Transformation

↓

Aggregation

↓

KPI Calculation

↓

Storage

↓

Visualization

↓

Decision
KPI Data Warehouse

The KPI platform should support historical analysis.

Dimensions may include:

Time
Service
Model
Provider
Application
Tenant
Business Unit
Environment
KPI Fact Model

Example:

AI_PERFORMANCE_FACT

date
service_id
model_id
provider_id
tenant_id
requests
successful_requests
errors
input_tokens
output_tokens
latency_p95
ttft_p95
quality_score
task_success
cost
security_events
KPI Aggregation

KPIs should be available at:

Request

Hourly

Daily

Weekly

Monthly

Quarterly

Yearly
Real-Time KPIs

Real-time dashboards should focus on:

Availability
Errors
Latency
Provider Health
Security
Cost Anomalies
Historical KPIs

Historical analytics should focus on:

Trends
Seasonality
Cost
Quality
Reliability
Business Value
KPI Alerting

KPIs should trigger alerts when:

Actual

crosses

Warning / Critical Threshold
KPI Alert Example
KPI:
Cost / Successful Task

Target:
<$0.05

Current:
$0.083

Status:
CRITICAL
KPI Review Board

Critical AI Services should have regular performance reviews.

Participants may include:

Product
Engineering
Operations
Security
FinOps
AI Governance
Monthly AI Performance Review

Review:

Reliability

Quality

Security

Cost

Business Value

Incidents

Changes

Forecast
Quarterly AI Portfolio Review

Evaluate:

Portfolio Value
Service Health
Model Strategy
Provider Strategy
Cost
Risk
Adoption
Roadmap
AI Performance Governance

Performance governance establishes:

KPI Definitions
Targets
Thresholds
Owners
Review Frequency
Escalation
Performance Exceptions

A service exceeding a threshold may require:

Investigation
Corrective Action
Optimization
Governance Review
Performance Escalation
Warning

↓

Service Owner

↓

Critical

↓

Operations / SRE

↓

Major Impact

↓

Governance / Executive
Performance Improvement Plan

When a service repeatedly misses KPIs:

Problem

↓

Root Cause

↓

Improvement Plan

↓

Owner

↓

Deadline

↓

Validation
KPI Documentation

Every production AI Service must document:

KPIs

SLIs

SLOs

SLAs

Targets

Thresholds

Data Sources

Owners

Review Schedule
KPI Repository Structure
23-ai-services/

├── kpis/
│   ├── definitions/
│   ├── formulas/
│   ├── targets/
│   ├── scorecards/
│   ├── dashboards/
│   ├── benchmarks/
│   └── reports/
│
├── chapters/
│   └── 10-ai-kpis-service-performance.md
│
└── diagrams/
KPI Control Matrix
Domain	KPI
Availability	Uptime
Reliability	MTTR
Performance	P95 Latency
Streaming	TTFT
API	Success Rate
Model	Quality
RAG	Groundedness
Agent	Task Success
Security	Policy Violations
Cost	Cost / Task
User	Satisfaction
Business	Outcome Rate
Governance	Compliance
Provider	Availability
Sustainability	Energy / Task
Enterprise AI KPI Set

The minimum enterprise KPI set should include:

Availability

Success Rate

Error Rate

P95 Latency

TTFT

Token Consumption

Cost / Request

Cost / Successful Task

Quality Score

Task Success Rate

Groundedness

Security Events

Provider Availability

SLO Compliance

Business Outcome
Critical AI Service KPI Set

Critical services should additionally monitor:

P99 Latency

MTTR

MTTD

Quality Regression

Model Drift

Provider Failover

Agent Loop Rate

DLP Events

Budget Variance

Human Intervention

Business Impact
AI Executive Scorecard
┌──────────────────────────────────────────────┐
│              AI EXECUTIVE SCORECARD          │
├──────────────────────────────────────────────┤
│ Services                    42                │
│ Healthy                     38                │
│ Degraded                     3                │
│ Critical                     1                │
│                                              │
│ Availability               99.96%             │
│ Quality                     94.2%             │
│ Task Success                91.8%             │
│ Monthly Cost               $XX,XXX            │
│ Cost Efficiency             +12%              │
│ Security Incidents             0             │
│ SLO Compliance              98.7%             │
│ Business Value             $XXX,XXX           │
└──────────────────────────────────────────────┘
AI Portfolio Performance

Portfolio-level reporting should answer:

How many AI Services exist?

How many are healthy?

Which services are expensive?

Which models perform best?

Which providers are most reliable?

Which services generate the most value?

Where is AI risk increasing?

Where should investment increase?
AI Service Ranking

Services may be ranked by:

Reliability
Quality
Cost Efficiency
Business Value

Ranking must not be used without considering service criticality and workload differences.

AI Performance Optimization Priorities

Priority should generally follow:

1. Safety
2. Security
3. Reliability
4. Quality
5. User Experience
6. Cost
7. Optimization

Cost optimization must never compromise mandatory security or safety requirements.

AI Performance Decision Framework
Is the service safe?

        ↓ Yes

Is it secure?

        ↓ Yes

Is it reliable?

        ↓ Yes

Is quality acceptable?

        ↓ Yes

Is user experience acceptable?

        ↓ Yes

Is cost efficient?

        ↓ Yes

Is business value positive?
Strategic Outcome

The KPI framework transforms AI performance from an abstract concept into an observable and governable enterprise capability.

Measure

↓

Understand

↓

Compare

↓

Improve

↓

Optimize

↓

Deliver Value
Future Evolution

AI KPI management evolves through:

Basic Metrics

↓

Standard KPIs

↓

Integrated Scorecards

↓

Predictive KPIs

↓

Adaptive AI Performance
Autonomous Performance Management

Future AI Services may automatically:

Detect performance degradation
Select alternative models
Change routing
Reduce unnecessary context
Optimize token consumption
Trigger failover
Recommend capacity changes

All automated actions must operate within approved governance policies.

AI Performance Closed Loop
                    ┌──────────────┐
                    │    Measure   │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │    Analyze   │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │    Decide    │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │    Improve   │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │   Validate   │
                    └──────┬───────┘
                           │
                           └──────────────► Measure
AI Performance Maturity Model
Level	Description
Level 1	Basic Metrics
Level 2	Standard KPIs
Level 3	Integrated Performance
Level 4	Predictive Performance
Level 5	Adaptive Performance
Level 1 — Basic Metrics

Basic telemetry exists.

Level 2 — Standard KPIs

Services have defined:

Availability
Latency
Errors
Cost
Quality
Level 3 — Integrated Performance

Technical, AI and business metrics are connected.

Level 4 — Predictive Performance

The platform predicts:

Cost
Capacity
Quality
Reliability
Level 5 — Adaptive Performance

The platform dynamically optimizes AI Services based on:

Context
Quality
Cost
Latency
Risk
Business objectives
Target State
Level 5

Adaptive AI Performance
AI Performance Acceptance Criteria

This chapter is complete when:

The AI KPI framework is defined.
AI performance dimensions are established.
KPI hierarchy is documented.
KPI ownership is defined.
KPI definition standards are established.
KPI lifecycle is documented.
Availability KPIs are defined.
Reliability KPIs are defined.
MTBF is defined.
MTTR is defined.
Error budgets are established.
Latency KPIs are defined.
P50/P95/P99 are defined.
TTFT is defined.
TTLT is defined.
Throughput KPIs are defined.
Concurrency is defined.
Capacity KPIs are defined.
API KPIs are established.
API throttling is monitored.
Retry rate is defined.
Model KPIs are defined.
Model quality is defined.
Model cost efficiency is established.
Provider KPIs are defined.
Provider concentration is monitored.
Provider failover is measured.
Token KPIs are established.
Token efficiency is defined.
Context efficiency is addressed.
AI quality KPIs are established.
Accuracy is defined.
Relevance is defined.
Groundedness is defined.
Hallucination measurement is addressed.
Task success is defined.
Quality regression is defined.
Evaluation coverage is established.
RAG KPIs are defined.
Retrieval success is measured.
Retrieval latency is measured.
Retrieval precision and recall are defined where applicable.
Grounded response rate is defined.
Citation accuracy is addressed.
Knowledge freshness is defined.
Agent KPIs are established.
Agent task success is defined.
Agent escalation is measured.
Agent tool success is measured.
Agent execution time is defined.
Agent loop rate is defined.
Human intervention is measured.
Security KPIs are defined.
Prompt injection rate is defined.
Policy violation rate is defined.
DLP metrics are established.
Security incident metrics are defined.
MTTD is defined.
Security MTTR is defined.
Cost KPIs are established.
Cost per request is defined.
Cost per user is defined.
Cost per successful task is defined.
Cost efficiency is defined.
Budget variance is established.
Cost forecasting is addressed.
User experience KPIs are established.
User satisfaction is defined.
Abandonment is measured.
Business KPIs are established.
AI business value is defined.
AI ROI is defined.
Productivity gains are measured.
Automation rate is defined.
Human deflection is defined.
Business outcome rate is established.
KPI correlation is defined.
KPI scorecards are established.
KPI thresholds are defined.
KPI trends are monitored.
Performance baselines are established.
Performance regression is defined.
Release performance is measured.
Canary performance is measured.
Performance gates are established.
SLI framework is defined.
SLO framework is defined.
SLA framework is defined.
SLI/SLO/SLA relationships are documented.
AI error budgets are defined.
Quality error budgets are addressed.
KPI reporting is established.
KPI automation is defined.
KPI data sources are documented.
KPI data quality is established.
KPI data lineage is defined.
KPI reproducibility is established.
KPI versioning is defined.
KPI governance is established.
KPI exceptions are governed.
Performance improvement processes are defined.
Performance optimization strategies are documented.
Multi-objective optimization is established.
Routing KPIs are defined.
AI FinOps KPIs are established.
AI sustainability KPIs are addressed.
Governance KPIs are defined.
Observability coverage KPIs are defined.
Business KPI mapping is established.
KPI benchmarking is defined.
KPI maturity is established.
Adaptive AI Performance target state is defined.
Autonomous performance management is addressed.
AI performance closed-loop management is documented.
Key Takeaways
AI Service Performance must measure much more than infrastructure availability.
Reliability, latency, quality, security, cost and business value must be evaluated together.
Every production AI Service must have clearly defined KPIs, SLIs, SLOs and, where applicable, SLAs.
AI quality metrics such as accuracy, groundedness and task success are first-class operational measurements.
RAG systems require dedicated retrieval and grounding KPIs.
Agents require specialized measurements for task completion, tool usage, execution time and human intervention.
Token consumption must be measured because it directly influences both performance and cost.
Cost per successful task is often more meaningful than cost per request.
Technical KPIs should ultimately connect to user experience and business outcomes.
KPI definitions must be standardized, reproducible, traceable and governed.
Performance must be evaluated before, during and after AI releases.
AI error budgets extend traditional SRE concepts to include quality where appropriate.
Multi-objective optimization requires balancing quality, latency, cost and safety.
The KPI framework enables executive visibility while preserving detailed operational measurements.
The long-term target is Adaptive AI Performance capable of dynamically optimizing AI services within governance boundaries.
Next Section

11 — AI Service Lifecycle & Operations

The next chapter will define the complete lifecycle of an AI Service from ideation and registration through architecture, development, evaluation, approval, deployment, operation, monitoring, change, optimization, deprecation and retirement, integrating the AI Service lifecycle with EVOXA governance, DevSecOps, SRE, FinOps and Product Management.
