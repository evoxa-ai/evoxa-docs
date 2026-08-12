Sí. Continuemos exactamente con la estructura que estamos utilizando para el Volumen 03 — AI Services, manteniendo el nivel de profundidad del capítulo 23 y la continuidad del Blueprint.

El capítulo siguiente es:

24 — AI Service Support & Customer Success
---
document_id: BP-0003-C23-24
chapter_id: CH-03-23-24
volume: Volume 03 — Artificial Intelligence Platform
parent_chapter: 23 — AI Services
title: AI Service Support & Customer Success
version: 1.0.0
status: Approved
owner: Enterprise AI Platform Office
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-10
---

# 24 — AI Service Support & Customer Success

> AI Service Support & Customer Success defines the operational framework through which EVOXA supports customers, resolves service issues, maintains customer health, drives adoption, protects business outcomes and ensures that AI Services continuously deliver measurable value throughout their lifecycle.

---

# Executive Summary

An AI Service does not end when the customer successfully activates the product.

Activation represents only the beginning of the customer relationship.

The service must subsequently provide:

- Reliable support.
- Technical assistance.
- Customer success management.
- Incident resolution.
- Service request management.
- Problem resolution.
- Knowledge management.
- Customer education.
- Adoption assistance.
- Outcome management.
- Customer health monitoring.
- Renewal support.
- Expansion support.
- Continuous improvement.

The fundamental lifecycle is:

```text
AI Service
      ↓
Customer
      ↓
Onboarding
      ↓
Activation
      ↓
Adoption
      ↓
Support
      ↓
Success
      ↓
Value Realization
      ↓
Retention
      ↓
Expansion

Customer Success is therefore not a separate activity from Service Operations.

It is the mechanism that connects operational service delivery with customer outcomes.

Purpose

The purpose of this chapter is to define how EVOXA:

Supports AI Service customers.
Manages customer requests.
Handles incidents.
Manages technical escalations.
Provides customer success.
Measures customer health.
Drives adoption.
Protects customer outcomes.
Identifies customer risk.
Prevents churn.
Supports renewals.
Enables expansion.
Captures customer feedback.
Improves AI Services continuously.
Strategic Objective

The strategic objective is:

Ensure that every customer receives the support, guidance, reliability and operational attention required to achieve sustained value from EVOXA AI Services.

Support Philosophy

EVOXA support should be:

Customer-Centric

Outcome-Oriented

Context-Aware

Proactive

Measurable

Transparent

Secure

Scalable
Customer Success Philosophy

Customer Success should focus on:

Customer Outcome

not

Customer Activity Alone

The objective is not simply:

More API Calls

The objective is:

More Successful Customer Outcomes
Support and Success Relationship

Support answers:

"How do we resolve this problem?"

Customer Success answers:

"How do we ensure the customer achieves the intended outcome?"

Both functions are required.

Support + Customer Success
                  CUSTOMER
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
       SUPPORT              CUSTOMER SUCCESS
          │                       │
          ▼                       ▼
      Resolve                  Adopt
       Issues                  Improve
          │                     Value
          └───────────┬───────────┘
                      ▼
                CUSTOMER OUTCOME
Customer Support Lifecycle
Issue / Request
      ↓
Detection
      ↓
Registration
      ↓
Classification
      ↓
Prioritization
      ↓
Assignment
      ↓
Investigation
      ↓
Resolution
      ↓
Validation
      ↓
Closure
      ↓
Feedback
Customer Success Lifecycle
Customer
   ↓
Onboarding
   ↓
Activation
   ↓
Adoption
   ↓
Outcome
   ↓
Health
   ↓
Optimization
   ↓
Renewal
   ↓
Expansion
Support Operating Model

EVOXA support should operate across multiple levels.

L0 — Self-Service

L1 — Service Desk

L2 — Technical Support

L3 — AI / Platform Engineering

L4 — Specialist / Vendor Support
Level 0 — Self-Service

Self-service should provide:

Documentation

Knowledge Base

FAQs

Troubleshooting

Status

Examples

Guides

Runbooks
Level 1 — Service Desk

L1 handles:

Basic Questions

Access

Authentication

Configuration

Known Issues

Service Requests

Initial Incident Classification
Level 2 — Technical Support

L2 handles:

Complex Configuration

Integration

API Problems

Performance

AI Service Behavior

Data Issues

Advanced Troubleshooting
Level 3 — Engineering

L3 handles:

Code Defects

Platform Failures

Model Problems

Architecture Issues

Critical Performance Problems

Systemic Defects
Level 4 — Specialist Support

L4 may involve:

Model Provider

Cloud Provider

Infrastructure Vendor

Security Specialist

External Technology Provider
Support Escalation
Customer
   ↓
L0
   ↓
L1
   ↓
L2
   ↓
L3
   ↓
L4

Escalation should occur based on:

Severity

Complexity

Business Impact

Technical Ownership

Risk
Support Channel Strategy

Possible channels:

Customer Portal

Email

Chat

Phone

API Support

Enterprise Support

Technical Account Management
Support Channel Selection

The appropriate channel depends on:

Urgency

Severity

Customer Plan

Contract

Issue Type
Customer Portal

The customer portal should provide:

Tickets

Service Requests

Incidents

Knowledge Base

Status

Usage

Support History

Notifications
Support Ticket
support_ticket:
  ticket_id:
  customer_id:
  tenant_id:
  user_id:
  service_id:
  type:
  category:
  priority:
  severity:
  status:
  created_at:
  updated_at:
  assigned_to:
  resolution:
Support Ticket Types
Incident

Service Request

Question

Problem

Change Request

Access Request

Billing Request
Incident

An incident is an unplanned interruption or degradation of an AI Service.

Service Request

A service request is a standard customer request that does not represent an incident.

Examples:

Access Request

Configuration Request

Quota Request

Information Request
Question

A question requests information without requiring operational intervention.

Examples:

How does this model work?

How do I configure the API?

Where can I see usage?
Problem

A problem represents the underlying cause of one or more incidents.

Incident

↓

Repeated Pattern

↓

Problem

↓

Root Cause
Change Request

A customer-requested change may include:

Configuration

Integration

Model

Policy

Quota

Workflow
Access Request

Access requests may include:

User

Role

Permission

API Key

Service Account
Billing Request

Billing requests may involve:

Invoice

Usage

Charge

Plan

Credit

Subscription
Ticket Classification

Every ticket should be classified by:

Type

Category

Service

Severity

Priority

Customer

Impact
Ticket Categories

Possible categories:

Authentication

Authorization

API

Model

Prompt

Knowledge

Integration

Performance

Availability

Billing

Security

Data

Configuration
Ticket Priority

Priority should consider:

Impact

Urgency

Business Criticality
Priority Model
P1 — Critical

P2 — High

P3 — Medium

P4 — Low
P1 — Critical

Examples:

Complete Service Outage

Critical Business Workflow Unavailable

Major Security Event

Large Customer Impact
P2 — High

Examples:

Major Degradation

Important Workflow Failure

High Customer Impact
P3 — Medium

Examples:

Limited Functionality

Non-Critical Integration Issue

Individual Customer Problem
P4 — Low

Examples:

Information Request

Minor UX Issue

Documentation Request
Severity

Severity measures the operational impact.

Severity ≠ Priority

Priority also incorporates:

Urgency

Customer

Contract

Business Importance
Customer Impact

Customer impact should consider:

Customers Affected

Users Affected

Workflows Affected

Business Criticality

Duration
Customer Impact Model
Customer Impact

=

Affected Customers
×

Affected Workflows
×

Business Criticality
×

Duration
Ticket Lifecycle
New
 ↓
Triaged
 ↓
Assigned
 ↓
Investigating
 ↓
Waiting
 ↓
Resolved
 ↓
Validated
 ↓
Closed
Ticket Status
New

Open

In Progress

Waiting Customer

Waiting Internal

Escalated

Resolved

Closed
Ticket Assignment

Tickets should be assigned according to:

Service Ownership

Technical Domain

Severity

Support Tier

Expertise
Ticket Ownership

Every active ticket must have an accountable owner.

Ticket

↓

Owner

↓

Action

↓

Resolution
Support Queue

Queues may be organized by:

Service

Priority

Customer

Technology

Support Tier
Support Workload

Support operations should monitor:

Open Tickets

New Tickets

Resolved Tickets

Backlog

Age

SLA Risk
Ticket Backlog
Backlog

=

Open Tickets

Backlog should be analyzed by:

Age

Priority

Customer

Service

Category
Ticket Aging
Ticket Age

=

Current Time
-

Ticket Creation Time
Aging Buckets
< 1 hour

1–4 hours

4–24 hours

1–3 days

3–7 days

> 7 days

Thresholds may vary by support plan.

Support SLA

Support commitments should define:

Response Time

Update Frequency

Resolution Target

Escalation

Coverage
First Response Time
FRT

=

First Human Response
-

Ticket Creation
Time to Resolution
TTR

=

Resolution Time
-

Ticket Creation
Time to Restore

For incidents:

TTRestore

=

Service Restoration
-

Incident Start
Support SLA Compliance
SLA Compliance

=

Tickets Meeting SLA
/
Tickets Subject to SLA
SLA Breach

A ticket is at risk of breach when:

Elapsed Time

≥

Configured Threshold
SLA Escalation
SLA Risk

↓

Escalation

↓

Priority Review

↓

Action
Support Coverage

Support coverage should define:

Business Hours

After Hours

24×7

Holiday Coverage

Emergency Coverage
Support Model by Customer Tier
Standard

Professional

Enterprise

Strategic

Each tier may define different:

Channels

Response

Coverage

Escalation

Customer Success
Enterprise Support

Enterprise support may include:

Dedicated Contacts

Priority Escalation

Technical Reviews

Customer Success

Architecture Assistance
Strategic Customer Support

Strategic customers may receive:

Executive Escalation

Technical Account Management

Architecture Reviews

Proactive Monitoring

Quarterly Business Reviews
Technical Account Management

A Technical Account Manager may coordinate:

Architecture

Adoption

Incidents

Roadmap

Risk

Optimization
Customer Success Manager

The Customer Success Manager focuses on:

Adoption

Outcomes

Customer Health

Training

Value

Renewal

Expansion
Customer Success Responsibilities

Customer Success should:

Define customer objectives.
Establish success criteria.
Build adoption plans.
Track customer health.
Identify risks.
Coordinate interventions.
Measure value.
Conduct reviews.
Support renewal.
Identify expansion opportunities.
Customer Success Plan
customer_success_plan:
  customer_id:
  tenant_id:
  services:
  business_objectives:
  use_cases:
  target_users:
  adoption_targets:
  success_metrics:
  milestones:
  risks:
  actions:
  owner:
  review_date:
Customer Objectives

Objectives should be expressed as outcomes.

Weak:

Use AI

Strong:

Reduce analysis time by 40%.
Success Criteria

Each customer should have measurable success criteria.

Objective

↓

Metric

↓

Baseline

↓

Target

↓

Measurement
Customer Baseline

Before measuring improvement:

Baseline

=

Current State
Customer Target
Target

=

Desired State
Customer Outcome
Outcome

=

Actual State
Customer Value Gap
Value Gap

=

Target Outcome
-

Actual Outcome
Customer Milestones

Typical milestones:

Contract

Onboarding

Integration

Activation

First Value

Production

Adoption

Outcome

Renewal
Milestone Tracking
customer_milestone:
  customer_id:
  service_id:
  milestone:
  target_date:
  completed_date:
  status:
  owner:
Customer Success Journey
Contract
   ↓
Kickoff
   ↓
Onboarding
   ↓
Integration
   ↓
Activation
   ↓
Adoption
   ↓
Value
   ↓
Optimization
   ↓
Renewal
   ↓
Expansion
Customer Kickoff

The kickoff should establish:

Objectives

Stakeholders

Timeline

Responsibilities

Success Criteria

Communication Model
Customer Stakeholders

Possible stakeholders:

Executive Sponsor

Business Owner

Technical Owner

Security

Operations

Users

Customer Success
Stakeholder Map
stakeholder:
  customer_id:
  name:
  role:
  responsibility:
  influence:
  engagement:
Executive Sponsor

The executive sponsor should understand:

Business Value

Strategic Importance

Risk

Outcome
Business Owner

The business owner manages:

Use Case

Process

Outcome

Adoption
Technical Owner

The technical owner manages:

Integration

Configuration

API

Security

Operations
Customer Success Cadence

Customer Success may use:

Weekly

Monthly

Quarterly

depending on customer tier and lifecycle stage.

Weekly Customer Review

May focus on:

Open Issues

Adoption

Blockers

Immediate Actions
Monthly Customer Review

May focus on:

Usage

Adoption

Health

Support

Outcomes
Quarterly Business Review

The QBR should review:

Business Objectives

Results

Value

Adoption

Challenges

Roadmap

Next Objectives
QBR Structure
Objectives
   ↓
Performance
   ↓
Outcomes
   ↓
Value
   ↓
Risks
   ↓
Roadmap
   ↓
Next Quarter
Customer Health

Customer health is a multidimensional representation of customer success.

Health

=

Usage

+

Adoption

+

Outcome

+

Satisfaction

+

Support

+

Relationship

-

Risk
Customer Health Dimensions
Usage

Adoption

Outcome

Satisfaction

Support

Relationship

Commercial

Risk
Usage Health

Measure:

Active Users

Requests

Workflows

Features

Frequency
Adoption Health

Measure:

Activated Users

Active Users

Feature Adoption

Workflow Adoption
Outcome Health

Measure:

Target Achievement

Business Improvement

Value Realization
Satisfaction Health

Measure:

CSAT

NPS

CES

Feedback
Support Health

Measure:

Open Tickets

Recurring Issues

Escalations

SLA Performance
Relationship Health

Measure:

Stakeholder Engagement

Executive Engagement

Meeting Participation

Feedback
Commercial Health

Measure:

Contract

Renewal

Expansion

Usage vs Entitlement
Customer Risk

Risk signals may include:

Usage Decline

Low Adoption

Poor Outcomes

Negative Feedback

High Ticket Volume

Repeated Incidents

Stakeholder Departure

Budget Pressure
Customer Risk Model
customer_risk:
  customer_id:
  service_id:
  risk_type:
  severity:
  probability:
  impact:
  detected_at:
  owner:
  mitigation:
  status:
Customer Risk Levels
Low

Medium

High

Critical
Customer Risk Score
Risk Score

=

Probability

×

Impact
Customer Health Score

Example conceptual model:

Health Score

=

Usage Score
+
Adoption Score
+
Outcome Score
+
Satisfaction Score
+
Relationship Score
-
Risk Score

Weights should be configured according to service characteristics.

Customer Health States
Healthy

Stable

Watch

At Risk

Critical
Healthy

Indicators:

Strong Adoption

Positive Outcomes

High Satisfaction

Low Support Friction
Stable

Indicators:

Consistent Usage

Expected Outcomes

Normal Support
Watch

Indicators:

Early Usage Decline

Feature Underutilization

Reduced Engagement
At Risk

Indicators:

Low Adoption

Poor Outcomes

Negative Feedback

Repeated Issues
Critical

Indicators:

Major Business Impact

Churn Risk

Executive Escalation

Service Failure
Customer Health Lifecycle
Healthy
  ↓
Watch
  ↓
At Risk
  ↓
Critical
  ↓
Recovery

OR

Healthy
  ↓
Expansion
Customer Risk Detection
Telemetry

+

Usage

+

Support

+

Feedback

+

Outcome

↓

Customer Health Engine
Customer Health Engine
Data
 ↓
Signals
 ↓
Scoring
 ↓
Health State
 ↓
Recommendation
 ↓
Action
Proactive Customer Success

Customer Success should not wait for customers to report every problem.

Proactive actions may include:

Usage Review

Training

Optimization

Configuration Review

Adoption Campaign

Risk Intervention
Proactive Support
Signal

↓

Potential Problem

↓

Customer Notification

↓

Mitigation

↓

Resolution
Proactive Adoption

If a customer has not adopted a valuable feature:

Underutilization

↓

Recommendation

↓

Training

↓

Activation

↓

Adoption
Customer Success Intervention
customer_intervention:
  customer_id:
  trigger:
  risk:
  action:
  owner:
  due_date:
  expected_outcome:
  status:
Intervention Types
Training

Technical Review

Executive Meeting

Configuration Review

Support Escalation

Product Recommendation

Integration Assistance
Intervention Priority
Customer Impact

+

Risk

+

Revenue

+

Strategic Importance
Customer Recovery Plan
Risk Detected
      ↓
Root Cause
      ↓
Recovery Plan
      ↓
Customer Agreement
      ↓
Execution
      ↓
Measurement
      ↓
Recovery
Recovery Success
Health Improved

+

Outcome Restored

+

Customer Confidence Restored
Customer Escalation

Escalation may be:

Technical

Operational

Commercial

Executive

Security
Technical Escalation
L2

↓

L3

↓

Engineering
Operational Escalation
Support

↓

Operations

↓

Incident Management
Commercial Escalation
Customer Success

↓

Account Management

↓

Commercial Leadership
Executive Escalation

Used for:

Critical Customer Impact

Strategic Customers

Major Contract Risk

Executive Concern
Security Escalation

Security-related issues should follow:

Support

↓

Security Team

↓

Incident Response

where appropriate.

Customer Incident Management

Customer-facing incident management should integrate with platform incident management.

Platform Incident
       ↓
Customer Impact
       ↓
Customer Communication
       ↓
Support Coordination
       ↓
Resolution
Incident Communication

Customer communication should explain:

What Happened

Who Is Affected

Current Impact

What Is Being Done

Next Update
Incident Update Cadence

Update frequency should depend on:

Severity

Customer Impact

Contract

Incident Duration
Major Incident Customer Management
Incident Commander
        │
        ├── Engineering
        ├── Operations
        ├── Support
        ├── Customer Success
        └── Communications
Customer Success During Major Incident

Customer Success should:

Identify Affected Customers

Coordinate Communication

Track Customer Impact

Coordinate Executive Updates

Track Recovery
Customer Impact Tracking
customer_impact:
  incident_id:
  customer_id:
  service_id:
  affected_users:
  affected_workflows:
  impact_level:
  start_time:
  recovery_time:
  business_impact:
Customer Incident Closure

An incident should not be considered fully closed from the customer perspective until:

Service Restored

+

Customer Impact Confirmed

+

Communication Completed
Post-Incident Customer Review

For major incidents:

Incident

↓

Resolution

↓

Customer Review

↓

Lessons Learned

↓

Corrective Actions
Customer-Facing RCA

Where appropriate provide:

Incident Summary

Impact

Timeline

Root Cause

Resolution

Corrective Actions
Problem Management

Repeated customer issues should generate problem records.

Incident

+

Pattern

↓

Problem
Customer Problem Record
customer_problem:
  problem_id:
  customer_id:
  service_id:
  related_incidents:
  symptoms:
  root_cause:
  impact:
  workaround:
  permanent_fix:
  owner:
  status:
Known Errors

Known errors should be documented when:

Root Cause Known

+

Permanent Fix Not Yet Available
Customer Workaround

Where possible provide:

Workaround

Expected Impact

Limitations

Permanent Fix Status
Problem Communication

Customers affected by systemic problems should receive appropriate communication.

Recurring Incident Detection
Incident 1
+
Incident 2
+
Incident 3

↓

Pattern

↓

Problem
Knowledge Management

Knowledge management supports both:

Customer

+

Support Team
Knowledge Base

The knowledge base should contain:

How-To

Troubleshooting

FAQs

API Documentation

Known Issues

Runbooks

Release Notes
Knowledge Article
knowledge_article:
  article_id:
  title:
  service_id:
  category:
  audience:
  content:
  version:
  status:
  owner:
  last_reviewed:
Knowledge Lifecycle
Draft
 ↓
Review
 ↓
Published
 ↓
Maintained
 ↓
Deprecated
Knowledge Quality

Knowledge should be:

Accurate

Current

Searchable

Actionable

Customer-Friendly
Knowledge Gap

A knowledge gap exists when:

Repeated Question

+

No Effective Documentation
Knowledge Gap Resolution
Question

↓

Identify Gap

↓

Create Article

↓

Publish

↓

Measure Reduction
Self-Service Resolution
Customer Problem

↓

Search

↓

Knowledge

↓

Resolution
Self-Service Rate
Self-Service Rate

=

Issues Resolved Without Human Support
/
Total Eligible Issues
Deflection

Support deflection should be measured carefully.

Deflection

=

Customer Resolves Issue
without Creating Support Ticket
AI-Powered Support

EVOXA may use AI to assist support.

Possible capabilities:

Ticket Classification

Suggested Response

Knowledge Retrieval

Troubleshooting

Ticket Summarization

Root Cause Assistance
AI Support Architecture
Customer
   ↓
AI Support Assistant
   ↓
Knowledge
   ↓
Service Context
   ↓
Recommendation
   ↓
Human Support
AI Ticket Classification
Ticket

↓

AI Classification

↓

Type

Category

Priority Suggestion

Routing
AI Support Response
Question

↓

Retrieve Knowledge

↓

Generate Draft

↓

Human Validation

↓

Customer
AI Support Guardrails

AI support must respect:

Authorization

Privacy

Security

Knowledge Freshness

Human Escalation
AI Support Confidence

Low-confidence support responses should trigger:

Human Review

rather than unsupported certainty.

AI Support Auditability

AI-assisted support should record:

Input

Context

Suggested Response

Human Action

Final Response

where required.

Customer Support Automation

Automate low-risk activities:

Ticket Routing

Acknowledgements

Status Notifications

Knowledge Suggestions

Ticket Summaries
Human Approval

Human approval should remain required for appropriate high-impact actions.

AI Recommendation

↓

Human Review

↓

Action
Support Automation Levels
Level 0 — Manual

Level 1 — Assisted

Level 2 — Recommended

Level 3 — Human Approved

Level 4 — Automated
Customer Success Automation

Possible automation:

Health Calculation

Usage Alerts

Renewal Reminders

Training Recommendations

Adoption Campaigns
Customer Health Alert
Health Decline

↓

Alert

↓

Customer Success

↓

Intervention
Customer Adoption Alert
Adoption Below Target

↓

Alert

↓

Investigation

↓

Intervention
Customer Outcome Alert
Outcome Below Target

↓

Alert

↓

Root Cause

↓

Customer Success Action
Renewal Management

Renewal should begin before contract expiration.

Contract

↓

Value Review

↓

Renewal Planning

↓

Decision

↓

Renew
Renewal Readiness

Indicators:

Value Realized

Strong Adoption

Healthy Relationship

Open Issues Controlled

Business Need Confirmed
Renewal Risk

Signals:

Low Adoption

Low Value

Negative Feedback

Open Critical Issues

Budget Pressure
Renewal Health
renewal_health:
  customer_id:
  contract_id:
  renewal_date:
  value_status:
  adoption_status:
  health_status:
  risk:
  renewal_probability:
Renewal Timeline
180 Days
    ↓
Value Review

120 Days
    ↓
Adoption Review

90 Days
    ↓
Renewal Strategy

60 Days
    ↓
Commercial Discussion

30 Days
    ↓
Finalization

Timelines may vary by contract.

Expansion Management

Expansion should follow demonstrated customer value.

Value

↓

Trust

↓

Additional Need

↓

Expansion
Expansion Signals
High Usage

Capacity Limit

Feature Demand

Additional Users

New Business Unit

New Use Case
Expansion Readiness
Healthy Customer

+

Demonstrated Value

+

Additional Need
Expansion Opportunity
expansion_opportunity:
  customer_id:
  current_service:
  opportunity_type:
  trigger:
  expected_value:
  customer_need:
  stage:
  owner:
Customer Advocacy

Advocacy may include:

Reference

Case Study

Testimonial

Design Partner

Advisory Board
Customer Reference Program

Customers may become references when:

Strong Outcome

+

High Satisfaction

+

Stable Relationship
Customer Feedback Integration

Support and Customer Success feedback should flow into Product Management.

Customer

↓

Support

↓

Customer Success

↓

Insights

↓

Product

↓

Roadmap
Customer Voice Pipeline
Tickets

+

Feedback

+

Meetings

+

Usage

+

Outcomes

↓

Customer Voice

↓

Product Insights
Support Analytics

Support analytics should measure:

Volume

Backlog

Response

Resolution

Escalation

SLA

Customer Satisfaction
Support KPI Tree
SUPPORT
 │
 ├── Volume
 │
 ├── Response
 │
 ├── Resolution
 │
 ├── SLA
 │
 ├── Escalation
 │
 ├── Quality
 │
 └── Satisfaction
Customer Success KPI Tree
CUSTOMER SUCCESS
 │
 ├── Activation
 │
 ├── Adoption
 │
 ├── Outcomes
 │
 ├── Health
 │
 ├── Retention
 │
 └── Expansion
Support KPIs

Core metrics:

First Response Time

Time to Resolution

Time to Restore

SLA Compliance

First Contact Resolution

Backlog

Ticket Aging

Escalation Rate

Reopen Rate

CSAT
First Contact Resolution
FCR

=

Tickets Resolved at First Contact
/
Eligible Tickets
Reopen Rate
Reopen Rate

=

Reopened Tickets
/
Resolved Tickets
Escalation Rate
Escalation Rate

=

Escalated Tickets
/
Total Tickets
Support Quality

Support quality should consider:

Correctness

Completeness

Clarity

Resolution

Customer Satisfaction
Customer Success KPIs

Core metrics:

Activation Rate

Adoption Rate

Time to Value

Value Realization

Customer Health

Retention

Churn

Renewal

Expansion

NPS
Customer Retention
Retention Rate

=

Retained Customers
/
Customers at Start of Period
Customer Churn
Churn Rate

=

Customers Lost
/
Customers at Start of Period
Net Revenue Retention

Where applicable:

NRR

=

Beginning Revenue
+
Expansion
-
Contraction
-
Churn

÷

Beginning Revenue
Gross Revenue Retention
GRR

=

Beginning Revenue
-
Contraction
-
Churn

÷

Beginning Revenue
Expansion Rate
Expansion Rate

=

Expansion Revenue
/
Beginning Revenue
Customer Lifetime Value

Customer lifetime value may incorporate:

Revenue

Margin

Retention

Expansion
Customer Success ROI
Customer Success ROI

=

Protected Value
+
Expansion Value
+
Retention Value

÷

Customer Success Cost
Support Cost
Support Cost

=

People

+

Infrastructure

+

Tools

+

Vendors
Cost per Ticket
Cost per Ticket

=

Support Cost
/
Resolved Tickets
Cost per Customer
Support Cost per Customer

=

Support Cost
/
Supported Customers
Support Efficiency
Support Efficiency

=

Resolved Customer Issues
/
Support Resources
Customer Success Efficiency
Customer Success Efficiency

=

Customers Achieving Outcomes
/
Customer Success Resources
Support Capacity

Support capacity should account for:

Customer Count

Ticket Volume

Complexity

Severity

Support Coverage
Support Forecasting
Historical Tickets

+

Customer Growth

+

Product Changes

+

Incidents

↓

Support Demand Forecast
Support Capacity Planning
Demand Forecast

↓

Required Capacity

↓

Staffing

↓

Coverage
Support Knowledge Optimization

Repeated tickets should trigger:

Knowledge Article

Product Improvement

UX Improvement

Automation
Ticket Reduction Loop
Repeated Ticket

↓

Root Cause

↓

Fix

↓

Documentation

↓

Automation

↓

Ticket Volume ↓
Customer Success Improvement Loop
Risk

↓

Intervention

↓

Outcome

↓

Measure

↓

Improve
Support Continuous Improvement
Measure

↓

Analyze

↓

Identify Problem

↓

Improve

↓

Validate
Support Problem Review

Regular reviews should identify:

Top Issues

Recurring Incidents

Knowledge Gaps

Automation Opportunities

Product Defects
Customer Success Review

Regular reviews should identify:

Health

Adoption

Outcomes

Risks

Opportunities
Customer Experience Review

Support and Customer Success should jointly review:

Friction

Satisfaction

Support

Adoption

Value
Cross-Functional Customer Operations
Customer Success
       │
       ├── Support
       ├── Product
       ├── Engineering
       ├── Operations
       ├── Security
       ├── Finance
       └── Sales
Customer Escalation Council

For strategic issues:

Customer Success

+

Support

+

Engineering

+

Product

+

Leadership
Customer Escalation Workflow
Issue

↓

Assess

↓

Classify

↓

Assign

↓

Resolve

↓

Communicate

↓

Validate

↓

Close

↓

Learn
Customer Success Governance

Governance should establish:

Roles

Responsibilities

Metrics

Cadence

Escalation

Accountability
RACI

Typical responsibilities:

Activity	Support	Customer Success	Engineering	Product
Ticket Intake	R	C	I	I
Incident Response	R	C	R	I
Customer Health	C	R	I	I
Adoption	C	R	C	C
Product Defect	C	C	R	R
Renewal	I	R	I	C
Expansion	I	R	C	C
Customer Feedback	R	R	C	R
Support Ownership

Every AI Service should define:

Service Owner

Support Owner

Engineering Owner

Customer Success Owner
Customer Success Ownership

Each strategic customer should have an accountable owner.

Customer

↓

Customer Success Manager

↓

Success Plan

↓

Outcome
Customer Support Ownership
Service

↓

Support Team

↓

Escalation Path

↓

Engineering
Support Runbooks

Runbooks should exist for common issues:

Authentication

API Failure

Model Failure

Quota

Integration

Performance

Data

Security
Customer Troubleshooting

Troubleshooting should follow:

Symptom

↓

Context

↓

Diagnosis

↓

Action

↓

Validation
Customer Troubleshooting Context

Relevant context may include:

Tenant

Service

Version

Model

Request

Error

Timestamp
Troubleshooting Safety

Support personnel must not expose:

Secrets

Credentials

Private Data

Unauthorized Customer Data
Customer Data Access

Support access should follow:

Least Privilege

Need to Know

Auditability
Support Access Audit

Support actions should be auditable where required:

Who

Accessed What

When

Why

Action
Customer Support Security

Support processes must protect:

Customer Data

Credentials

API Keys

Prompts

Responses

Knowledge
Secure Troubleshooting

Use:

Redacted Logs

Scoped Access

Temporary Access

Audited Access

where appropriate.

Customer Support and Privacy

Support should minimize the collection of unnecessary personal data.

Customer Support Data Retention

Support records should follow applicable:

Retention Policy

Contract

Privacy Requirements

Legal Requirements
Customer Success Data

Customer Success may use:

Usage

Outcomes

Support

Feedback

Health

Commercial Data

subject to authorization and governance.

Customer 360

EVOXA may provide a consolidated customer view:

Customer

├── Services
├── Users
├── Usage
├── Support
├── Health
├── Outcomes
├── Contracts
├── Risks
└── Opportunities
Customer 360 Architecture
                    CUSTOMER 360
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
     Product           Support         Success
        │                │                │
        └────────────────┼────────────────┘
                         ▼
                       Health
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
            Risk                 Value
              │                     │
              └──────────┬──────────┘
                         ▼
                    Action Plan
Customer 360 Data Model
customer_360:
  customer_id:
  tenant_id:
  services:
  users:
  usage:
  support:
  health:
  outcomes:
  contracts:
  risks:
  opportunities:
Customer Success Dashboard
Customer Health

Adoption

Usage

Outcomes

Support

Open Risks

Renewal

Expansion
Support Dashboard
Open Tickets

P1

P2

SLA Risk

Backlog

Aging

Resolution

CSAT
Customer Health Dashboard
Healthy

Watch

At Risk

Critical

Recovering

Expanding
Customer Success Forecast

Forecast:

Renewal

Churn

Expansion

Adoption
Predictive Customer Success
Historical Data

+

Current Signals

↓

Prediction

↓

Recommended Action
Predictive Churn
Risk Signals

↓

Churn Probability

↓

Customer Intervention
Predictive Expansion
Usage

+

Value

+

Capacity

↓

Expansion Probability
Predictive Support
Telemetry

↓

Potential Customer Issue

↓

Proactive Support
Predictive Incident Communication
Early Warning

↓

Affected Customers

↓

Notification

↓

Mitigation
Customer Success Automation
Health Monitoring

↓

Trigger

↓

Recommendation

↓

Customer Success Action
Customer Success AI Assistant

The Customer Success AI Assistant may help with:

Customer Summaries

Health Analysis

Risk Detection

Meeting Preparation

QBR Preparation

Action Recommendations
AI Customer Summary
customer_summary:
  customer:
  current_health:
  adoption:
  outcomes:
  support:
  risks:
  opportunities:
  recommended_actions:
AI Customer Success Guardrails

The AI assistant must:

Respect Authorization

Protect Customer Data

Cite Evidence Where Appropriate

Identify Uncertainty

Avoid Unsupported Claims
Customer Meeting Intelligence

AI may assist with:

Meeting Summary

Actions

Risks

Questions

Next Steps
QBR Intelligence

AI may generate:

Usage Summary

Outcome Summary

Support Summary

Health Summary

Risk Summary

Recommended Agenda
Customer Success Knowledge

Customer Success teams require access to:

Product Documentation

Support History

Architecture

Usage

Contracts

Roadmap

Known Issues

according to authorization.

Customer Success Training

Training should cover:

AI Services

Customer Journey

Support

Security

AI Capabilities

Limitations

Governance
Support Training

Support teams should understand:

AI Basics

Service Architecture

Common Failures

Troubleshooting

Escalation

Security
AI Literacy

Customer-facing teams should understand:

Models

Prompts

Hallucination

Confidence

Knowledge

Agents

Limitations
Customer Communication Training

Teams should communicate AI limitations without:

Overpromising

Creating False Certainty

Using Excessive Technical Language
Support Communication Standard

Use:

What Happened

What It Means

What We Are Doing

What the Customer Should Do

When We Will Update
Customer Success Communication Standard

Use:

Current State

Objective

Gap

Recommendation

Expected Outcome

Next Action
Customer Action Management

Every significant customer issue should have:

Owner

Action

Due Date

Status
Customer Action Record
customer_action:
  action_id:
  customer_id:
  source:
  description:
  owner:
  due_date:
  status:
  expected_outcome:
Customer Success Action Plan
Problem

↓

Action

↓

Owner

↓

Deadline

↓

Outcome
Customer Success Risk Register
customer_risk_register:
  customer_id:
  risk_id:
  description:
  probability:
  impact:
  severity:
  mitigation:
  owner:
  review_date:
  status:
Customer Success Opportunity Register
customer_opportunity:
  customer_id:
  opportunity_id:
  description:
  business_need:
  potential_value:
  probability:
  owner:
  next_action:
  status:
Customer Success Operating Rhythm
Daily
  ↓
Support / Critical Risks

Weekly
  ↓
Customer Actions

Monthly
  ↓
Health / Adoption

Quarterly
  ↓
Business Outcomes
Daily Customer Operations

Review:

P1/P2 Incidents

Critical Customers

SLA Risks

Critical Health Risks
Weekly Customer Operations

Review:

Open Actions

Adoption Risks

Support Trends

Customer Feedback
Monthly Customer Operations

Review:

Health

Usage

Adoption

Outcomes

Support

Risks
Quarterly Customer Operations

Review:

Value

Renewal

Expansion

Strategic Roadmap

Relationship
Customer Success Maturity
Level	Capability
Level 1	Reactive Support
Level 2	Structured Support
Level 3	Customer Success
Level 4	Proactive Success
Level 5	Predictive Customer Success
Level 1 — Reactive Support

The organization responds when customers report issues.

Level 2 — Structured Support

The organization has:

Tickets

SLAs

Queues

Escalation
Level 3 — Customer Success

The organization actively manages:

Adoption

Outcomes

Health
Level 4 — Proactive Success

The organization detects:

Risk

Underutilization

Opportunity

before the customer requests assistance.

Level 5 — Predictive Customer Success

The organization predicts:

Churn

Expansion

Support Needs

Adoption

and recommends actions.

Customer Success North Star

The North Star metric is:

Customers Achieving Sustained Value
Support North Star

The Support North Star is:

Customer Issues Resolved
with Minimum Customer Effort
Customer Success Value Chain
Support

↓

Stability

↓

Adoption

↓

Outcome

↓

Value

↓

Retention

↓

Expansion
Support Value Chain
Issue

↓

Resolution

↓

Restoration

↓

Trust

↓

Continued Usage
Customer Success Value Chain
Objective

↓

Adoption

↓

Usage

↓

Outcome

↓

Value
Support and Customer Success Flywheel
Customer

↓

Use

↓

Issue / Need

↓

Support

↓

Resolution

↓

Success

↓

Value

↓

Trust

↓

More Adoption

↓

Feedback

↓

Improvement
Customer Success Control Plane
Customer

↓

Identity

↓

Services

↓

Usage

↓

Support

↓

Health

↓

Outcomes

↓

Risks

↓

Actions

↓

Value
Customer Support Control Plane
Ticket

↓

Classification

↓

Priority

↓

Assignment

↓

Resolution

↓

Validation

↓

Closure

↓

Knowledge
Customer Health Control Plane
Signals

↓

Health Engine

↓

Health State

↓

Risk

↓

Intervention

↓

Outcome

↓

Updated Health
Customer Success Architecture
                    CUSTOMER SUCCESS

                         CUSTOMER
                            │
                            ▼
                     CUSTOMER 360
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
       Support           Adoption          Outcomes
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ▼
                     Health Engine
                            │
                 ┌──────────┴──────────┐
                 ▼                     ▼
               Risks             Opportunities
                 │                     │
                 └──────────┬──────────┘
                            ▼
                      Action Plans
                            │
                            ▼
                       Customer Value
Support Architecture
Customer
   │
   ▼
Customer Portal
   │
   ▼
Support Platform
   │
   ├── Knowledge
   ├── Ticketing
   ├── SLA
   ├── Incident
   └── Customer Context
   │
   ▼
Support Teams
   │
   ├── L1
   ├── L2
   ├── L3
   └── L4
Customer Success Architecture
Customer Data
      ↓
Customer 360
      ↓
Health Engine
      ↓
Risk / Opportunity
      ↓
Customer Success
      ↓
Intervention
      ↓
Outcome
Support Data Flow
Customer Request

↓

Ticket

↓

Classification

↓

Assignment

↓

Investigation

↓

Resolution

↓

Customer Validation

↓

Knowledge
Customer Success Data Flow
Customer Usage

+

Support

+

Feedback

+

Outcomes

↓

Customer Health

↓

Risk / Opportunity

↓

Action

↓

Customer Outcome
Integrated Customer Operations
                     CUSTOMER
                         │
                         ▼
              ┌────────────────────┐
              │ Customer Experience│
              └─────────┬──────────┘
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
         SUPPORT             CUSTOMER SUCCESS
             │                     │
             ▼                     ▼
        Resolution             Adoption
             │                     │
             └──────────┬──────────┘
                        ▼
                     Outcome
                        │
                        ▼
                       Value
Customer Success and Product

Customer Success provides Product Management with:

Customer Needs

Feature Requests

Adoption Barriers

Feedback

Outcome Gaps
Customer Success and Engineering

Engineering receives:

Recurring Defects

Technical Risks

Performance Problems

Integration Problems

Reliability Issues
Customer Success and Operations

Operations receives:

Customer Impact

Critical Customers

Business Criticality

Communication Requirements
Customer Success and Security

Security receives:

Security Incidents

Privacy Concerns

Access Issues

Customer Risk
Customer Success and Finance

Finance receives:

Usage

Billing Issues

Renewal

Expansion

Revenue Risk
Customer Success and Sales

Sales receives:

Adoption

Customer Health

Expansion Opportunities

Renewal Risk
Closed-Loop Customer Operations
Customer

↓

Support / Success

↓

Insight

↓

Product

↓

Engineering

↓

AI Service

↓

Customer
Customer Feedback Closed Loop

Customers should see where appropriate:

Feedback Submitted

↓

Acknowledged

↓

Investigated

↓

Action Taken

↓

Result
Customer Success Continuous Improvement
Observe

↓

Measure

↓

Understand

↓

Intervene

↓

Measure

↓

Improve
Customer Support Continuous Improvement
Ticket

↓

Root Cause

↓

Resolution

↓

Knowledge

↓

Automation

↓

Prevention
Support-to-Product Loop
Repeated Issue

↓

Support

↓

Problem

↓

Product / Engineering

↓

Fix

↓

Release

↓

Customer
Support-to-Knowledge Loop
Repeated Question

↓

Knowledge Gap

↓

Article

↓

Self-Service

↓

Ticket Reduction
Support-to-Automation Loop
Repeated Manual Task

↓

Identify Pattern

↓

Automate

↓

Measure

↓

Scale
Success-to-Product Loop
Customer Outcome Gap

↓

Customer Success

↓

Product Insight

↓

Product Change

↓

Improved Outcome
Customer Success-to-Roadmap
Customer Need

+

Business Value

+

Strategic Fit

↓

Roadmap Candidate
Customer Experience Continuity

Chapter 23 established:

Customer Experience

Activation

Adoption

Engagement

Value

Retention

Expansion

Chapter 24 operationalizes that experience through:

Support

Customer Success

Health

Incident Management

Knowledge

Intervention

Renewal

Expansion

The relationship is:

23 — Customer Experience & Adoption
                ↓
24 — Support & Customer Success
                ↓
25 — Customer Trust, Retention & Expansion
Relationship with Previous Chapters
21 — AI Service Business Value & Economics
                ↓
22 — AI Service Product Management & Monetization
                ↓
23 — AI Service Customer Experience & Adoption
                ↓
24 — AI Service Support & Customer Success
                ↓
25 — AI Service Trust, Retention & Expansion
Strategic Outcome

The objective of AI Service Support & Customer Success is to transform customer support from a reactive cost center into a strategic capability for maintaining:

Reliability

Adoption

Trust

Outcome

Value

Retention

Expansion

The target state is:

Every EVOXA AI Service customer should have a clear support path, an accountable success model, measurable health, proactive risk management and a continuous mechanism for achieving and expanding business value.

Key Takeaways
Support is a core component of AI Service delivery.
Customer Success extends beyond support.
Support resolves customer problems.
Customer Success drives customer outcomes.
Both capabilities must operate together.
Support should be multi-tiered.
Self-service should reduce unnecessary support demand.
L1 should handle standard customer requests.
L2 should handle advanced technical issues.
L3 should handle engineering-level problems.
L4 may involve external specialists and providers.
Every support ticket must have an owner.
Tickets should be classified.
Priority should reflect impact and urgency.
Severity should reflect operational impact.
Support SLAs should be measurable.
First Response Time should be measured.
Time to Resolution should be measured.
Time to Restore should be measured.
SLA compliance should be measured.
Ticket aging should be monitored.
Backlog should be actively managed.
Enterprise customers may require enhanced support.
Technical Account Management may support strategic customers.
Customer Success Managers should own customer outcomes.
Every strategic customer should have a success plan.
Customer objectives should be measurable.
Customer outcomes should have baselines and targets.
Customer milestones should be tracked.
Customer health should be multidimensional.
Usage alone does not define customer health.
Adoption should be measured.
Outcomes should be measured.
Satisfaction should be measured.
Support health should be measured.
Relationship health should be measured.
Commercial health should be measured.
Customer risk should be identified proactively.
Health states should be standardized.
At-risk customers require intervention.
Recovery plans should be measurable.
Customer incidents must integrate with Incident Management.
Customer communication is part of incident management.
Major incidents require coordinated customer communication.
Recurring incidents should become problems.
Problem Management should address root causes.
Knowledge Management should reduce repeated support demand.
Knowledge articles must remain current.
AI can assist support operations.
AI support must use appropriate guardrails.
Human escalation must remain available.
Customer Success activities can be automated where appropriate.
Automation should be risk-based.
Renewal management should begin before expiration.
Renewal should be based on demonstrated value.
Expansion should follow successful adoption.
Customer advocacy can create strategic value.
Support feedback should reach Product Management.
Customer Success feedback should influence the roadmap.
Customer 360 provides consolidated customer context.
Customer Health Engines can identify risks and opportunities.
Predictive Customer Success can identify churn risk.
Predictive Customer Success can identify expansion opportunities.
Support and Customer Success should share a common operating model.
Customer-facing teams must understand AI capabilities and limitations.
Customer data must be protected.
Support access must follow least privilege.
Support actions should be auditable where required.
Customer Success should operate proactively.
Customer support should minimize customer effort.
The ultimate goal is sustained customer value.
Acceptance Criteria

This chapter is complete when:

Support philosophy is defined.
Customer Success philosophy is defined.
Support and Customer Success relationship is documented.
Customer Support lifecycle is established.
Customer Success lifecycle is established.
Support operating model is defined.
L0 self-service model is defined.
L1 service desk model is defined.
L2 technical support model is defined.
L3 engineering escalation is defined.
L4 specialist support is defined.
Support channels are documented.
Customer portal requirements are established.
Support ticket model is defined.
Ticket types are established.
Incident classification is documented.
Service request classification is documented.
Problem classification is documented.
Change request classification is documented.
Access request classification is documented.
Billing request classification is documented.
Ticket priority model is defined.
Severity model is established.
Customer impact model is documented.
Ticket lifecycle is defined.
Ticket status model is established.
Ticket ownership requirements are documented.
Support queues are defined.
Ticket backlog management is established.
Ticket aging is defined.
Support SLA model is established.
First Response Time is defined.
Time to Resolution is defined.
Time to Restore is defined.
SLA compliance is established.
SLA escalation is documented.
Support coverage model is defined.
Customer support tiers are documented.
Enterprise support model is established.
Strategic customer support is defined.
Technical Account Management is documented.
Customer Success Manager responsibilities are established.
Customer Success responsibilities are defined.
Customer Success Plan is documented.
Customer objectives are defined.
Success criteria are established.
Customer baseline is defined.
Customer target is documented.
Customer outcome model is established.
Customer value gap is defined.
Customer milestones are documented.
Customer kickoff model is established.
Customer stakeholder model is defined.
Customer Success cadence is documented.
Weekly customer review is established.
Monthly customer review is defined.
Quarterly Business Review is documented.
Customer health model is established.
Customer health dimensions are defined.
Usage health is documented.
Adoption health is documented.
Outcome health is established.
Satisfaction health is defined.
Support health is documented.
Relationship health is established.
Commercial health is defined.
Customer risk model is documented.
Customer risk levels are established.
Customer health score is defined.
Customer health states are documented.
Customer risk detection is established.
Proactive Customer Success is defined.
Proactive support is documented.
Proactive adoption is established.
Customer intervention model is defined.
Customer recovery plan is documented.
Customer escalation model is established.
Technical escalation is defined.
Operational escalation is documented.
Commercial escalation is established.
Executive escalation is defined.
Security escalation is documented.
Customer incident management is established.
Incident communication requirements are defined.
Major incident customer management is documented.
Customer impact tracking is established.
Customer incident closure is defined.
Customer-facing RCA is documented.
Problem Management integration is established.
Customer problem records are defined.
Known error management is documented.
Customer workaround management is established.
Knowledge Management model is defined.
Knowledge base requirements are documented.
Knowledge article model is established.
Knowledge lifecycle is defined.
Knowledge quality requirements are documented.
Knowledge gap management is established.
Self-service resolution is defined.
Self-service rate is documented.
Support deflection is established.
AI-powered support model is defined.
AI ticket classification is documented.
AI support response model is established.
AI support guardrails are defined.
AI support confidence model is documented.
AI support auditability is established.
Support automation model is defined.
Human approval requirements are documented.
Customer Success automation is established.
Customer health alerts are defined.
Customer adoption alerts are documented.
Customer outcome alerts are established.
Renewal management is defined.
Renewal readiness is documented.
Renewal risk is established.
Renewal health is defined.
Expansion management is documented.
Expansion signals are established.
Expansion readiness is defined.
Expansion opportunity model is documented.
Customer advocacy is established.
Customer reference program is defined.
Customer feedback integration is documented.
Customer Voice pipeline is established.
Support analytics are defined.
Support KPI tree is documented.
Customer Success KPI tree is established.
First Contact Resolution is defined.
Reopen Rate is defined.
Escalation Rate is defined.
Support quality model is documented.
Customer Success KPIs are established.
Retention Rate is defined.
Churn Rate is defined.
NRR is defined.
GRR is defined.
Expansion Rate is documented.
Customer Lifetime Value relationship is established.
Customer Success ROI is defined.
Support cost model is documented.
Cost per ticket is established.
Support efficiency is defined.
Customer Success efficiency is documented.
Support capacity model is established.
Support forecasting is defined.
Support capacity planning is documented.
Knowledge optimization loop is established.
Ticket reduction loop is defined.
Customer Success improvement loop is documented.
Cross-functional Customer Operations model is established.
Customer escalation council is defined.
Customer escalation workflow is documented.
Customer Success governance is established.
Support ownership is defined.
Customer Success ownership is documented.
Support runbooks are established.
Customer troubleshooting model is defined.
Support access security is documented.
Customer data access controls are established.
Support audit requirements are defined.
Customer 360 model is documented.
Customer 360 architecture is established.
Customer Success dashboard is defined.
Support dashboard is documented.
Customer Health dashboard is established.
Customer Success forecasting is defined.
Predictive Customer Success is documented.
Predictive churn model is established.
Predictive expansion model is defined.
Predictive support model is documented.
Customer Success AI Assistant is established.
AI Customer Success guardrails are defined.
Customer meeting intelligence is documented.
QBR intelligence is established.
Customer Success knowledge requirements are defined.
Customer-facing team training is documented.
AI literacy requirements are established.
Customer communication standards are defined.
Customer action management is documented.
Customer risk register is established.
Customer opportunity register is defined.
Customer Success operating rhythm is documented.
Customer Success maturity model is established.
Customer Success North Star is defined.
Support North Star is documented.
Support value chain is established.
Customer Success value chain is defined.
Support and Customer Success flywheel is documented.
Customer Success control plane is established.
Support control plane is defined.
Customer Success architecture is documented.
Support architecture is established.
Support data flow is defined.
Customer Success data flow is documented.
Integrated Customer Operations model is established.
Customer Success and Product relationship is defined.
Customer Success and Engineering relationship is documented.
Customer Success and Operations relationship is established.
Customer Success and Security relationship is defined.
Customer Success and Finance relationship is documented.
Customer Success and Sales relationship is established.
Closed-loop Customer Operations is defined.
Support-to-Product loop is documented.
Support-to-Knowledge loop is established.
Support-to-Automation loop is defined.
Success-to-Product loop is documented.
Success-to-Roadmap process is established.
Customer Experience continuity with Chapter 23 is defined.
Relationship with Chapters 21–23 is documented.
Strategic outcome is established.
Continuity

Chapter 23 established the customer-facing experience:

Customer Experience

↓

Activation

↓

Adoption

↓

Engagement

↓

Value

↓

Retention

↓

Expansion

Chapter 24 establishes the operational mechanisms required to sustain that experience:

Support

↓

Resolution

↓

Customer Health

↓

Customer Success

↓

Outcome

↓

Retention

↓

Expansion

The resulting model is:

23 — AI Service Customer Experience & Adoption
                    ↓
24 — AI Service Support & Customer Success
                    ↓
25 — AI Service Trust, Retention & Expansion
Relationship with Volume 03
AI Services
    │
    ├── Product
    │
    ├── Customer Experience
    │
    ├── Support
    │
    ├── Customer Success
    │
    ├── Trust
    │
    ├── Retention
    │
    └── Expansion
Final Principle

An AI Service is not successful when the customer can merely use it. It is successful when the customer can rely on it, obtain support when needed, achieve the intended outcome, realize measurable value and continue expanding the relationship with confidence.


### Continuidad del capítulo 24

Con esto, la secuencia del **23 — AI Services** queda avanzando así:

```text
21 — AI Service Business Value & Economics
22 — AI Service Product Management & Monetization
23 — AI Service Customer Experience & Adoption
24 — AI Service Support & Customer Success
25 — AI Service Trust, Retention & Expansion
26 — ...
27 — ...
28 — ...
29 — ...
30 — ...

Y es importante que el 24 no repita simplemente el capítulo 23: el 23 define la experiencia y adopción del cliente, mientras que el 24 define la organización, procesos, soporte, Customer Success, health, escalamiento y operación necesaria para sostener esa experiencia
