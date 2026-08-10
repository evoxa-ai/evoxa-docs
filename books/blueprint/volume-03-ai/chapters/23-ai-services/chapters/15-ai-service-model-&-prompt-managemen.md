---
document_id: BP-0003-C23-15
chapter_id: CH-03-23-15
volume: Volume 03 — Artificial Intelligence Platform
parent_chapter: 23 — AI Services
title: AI Service Model & Prompt Management
version: 1.0.0
status: Approved
owner: Enterprise AI Platform Office
authors:
  - EVOXA Architecture Team
classification: Internal
last_updated: 2026-08-09
---

# 15 — AI Service Model & Prompt Management

> *The AI Service Model & Prompt Management framework defines how EVOXA selects, configures, versions, evaluates, deploys, monitors and governs AI models and prompts as controlled production assets.*

---

# Executive Summary

AI Services depend on two fundamental behavioral components:

```text
MODEL

+

PROMPT

The model provides the underlying intelligence capability.

The prompt defines how that capability is instructed, constrained and applied to a particular task.

Production AI cannot depend on:

Manually selected models
Hardcoded prompts
Unversioned instructions
Uncontrolled model changes
Provider-specific implementations
Unknown model configurations
Unmeasured prompt modifications
Informal experimentation in production

EVOXA therefore treats models and prompts as governed platform assets.

The architecture must provide:

Model Registry
Prompt Registry
Model Versioning
Prompt Versioning
Configuration Management
Evaluation
Approval
Deployment
Rollback
Monitoring
Cost Management
Security
Governance
Lifecycle Management

The fundamental production relationship is:

AI SERVICE

+

MODEL

+

MODEL CONFIGURATION

+

PROMPT

+

KNOWLEDGE

+

TOOLS

=

AI BEHAVIOR
Model & Prompt Vision

The vision is:

Create a governed AI behavior platform where every model and prompt used by EVOXA can be identified, versioned, evaluated, approved, deployed, monitored and retired.

Model & Prompt Mission

The mission is to ensure that:

Models are selected intentionally.
Prompts are treated as controlled assets.
Model versions are known.
Prompt versions are known.
Configuration is reproducible.
Changes are evaluated.
Production behavior is traceable.
Unsafe configurations are blocked.
Model upgrades are controlled.
Prompt regressions are detected.
Rollbacks are possible.
Costs are measurable.
Providers remain abstracted where practical.
Core Principles

The EVOXA Model & Prompt architecture follows:

Everything Is Versioned
Configuration as Code
Evaluation Before Production
Reproducibility
Explicit Ownership
Controlled Change
Provider Abstraction
Model Agnostic Services
Prompt as a Governed Asset
Security by Design
Least Privilege
Observable AI Behavior
Measurable Quality
Controlled Rollback
Automation First
Human Approval for High-Risk Changes
Model Definition

An AI model is a computational capability used to perform an AI task.

Examples include:

Large Language Models
Embedding Models
Vision Models
Speech Models
Classification Models
Ranking Models
Multimodal Models
Specialized Domain Models
Model Categories

EVOXA may classify models as:

Generative

Reasoning

Embedding

Classification

Ranking

Vision

Speech

Multimodal

Specialized
Model Asset

Every production model should have an identifiable model asset.

Recommended metadata:

model:
  model_id:
  name:
  provider:
  family:
  version:
  modality:
  capabilities:
  context_window:
  input_formats:
  output_formats:
  owner:
  status:
  risk_level:
  approval_status:
Model Identity

A model identity should be stable and unique.

Example:

model_id:
evoxa.llm.general.provider-x

A specific version should be separately identifiable.

model_version:
2026-07
Model Version

Model versions must be explicitly tracked.

Model Family

↓

Version 1

↓

Version 2

↓

Version 3
Why Model Versioning Matters

Model behavior can change because of:

Provider Updates
Model Retraining
Fine-Tuning
Safety Changes
Context Changes
Tokenization Changes
Performance Changes

Therefore a model reference must be reproducible.

Model Pinning

Critical services should pin approved model versions where practical.

AI Service

↓

Approved Model Version

↓

Production
Floating Model References

Floating references may be used only where explicitly approved.

Example:

latest

should not automatically be considered production-safe.

Model Registry

The Model Registry is the authoritative inventory of models available to EVOXA.

It should contain:

Model ID
Provider
Version
Capability
Modality
Context Window
Cost
Performance
Risk
Approval
Status
Model Registry Architecture
                 MODEL REGISTRY

 ┌────────────────────────────────────┐
 │ Model Metadata                     │
 │                                    │
 │ ID                                 │
 │ Provider                           │
 │ Version                            │
 │ Capability                         │
 │ Cost                               │
 │ Risk                               │
 │ Approval                           │
 └────────────────┬───────────────────┘
                  │
                  ▼
          AI Gateway / Router
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
     Model A   Model B   Model C
Model Registry Responsibilities

The registry should provide:

Discovery
Version Management
Metadata
Approval Status
Lifecycle Status
Ownership
Evaluation Results
Cost Information
Model Lifecycle
Candidate

↓

Evaluation

↓

Approved

↓

Staging

↓

Production

↓

Monitored

↓

Deprecated

↓

Retired
Model Candidate

A candidate model is any model under consideration.

Sources may include:

Approved Providers
Internal Models
Open Models
Fine-Tuned Models
New Provider Releases
Model Evaluation

Before production use, models should be evaluated against defined criteria.

Evaluation may include:

Quality
Accuracy
Safety
Latency
Cost
Reliability
Security
Domain Performance
Model Approval

A model should only become production eligible after required approvals.

Candidate

↓

Evaluation

↓

Security Review

↓

Risk Review

↓

Business Approval

↓

Production Eligible
Model Risk Classification

Models may be classified:

Low

Medium

High

Critical

The classification should reflect business impact and use case risk.

Model Capability Profile

Each model should define capabilities.

Example:

capabilities:
  text_generation: true
  reasoning: true
  vision: false
  audio_input: false
  audio_output: false
  tool_calling: true
  structured_output: true
Model Modality

Models should identify supported modalities.

Text

Image

Audio

Video

Multimodal
Context Window

The model registry should record supported context limits.

context:
  maximum_tokens:
  input_tokens:
  output_tokens:

AI Services must not assume unlimited context.

Model Output Constraints

Models may support:

JSON
Structured Output
Tool Calls
Text
Multimodal Output

Capabilities must be validated before service deployment.

Model Configuration

A model is not defined only by its name.

Behavior may depend on:

Model

+

Temperature

+

Top P

+

Max Tokens

+

System Prompt

+

Tools

+

Context
Configuration as a Versioned Asset

Model configuration should be versioned.

model_config:
  config_id:
  model_id:
  model_version:
  temperature:
  top_p:
  max_tokens:
  response_format:
  tool_policy:
Temperature

Temperature may influence response variability.

Higher values may produce more variation.

Lower values may produce more deterministic behavior.

Production values should be defined through evaluation rather than arbitrary preference.

Top-P

Where supported, top-p may influence token selection.

Its use should be controlled and evaluated together with other sampling parameters.

Max Tokens

Output limits should be explicitly defined.

Maximum Output

↓

Cost Control

+

Latency Control

+

Response Control
Determinism

Where supported, deterministic or reproducible configurations should be used for workflows requiring consistency.

Prompt Definition

A prompt is a controlled instruction set used to guide model behavior.

A production prompt may contain:

System Instructions

+

Developer Instructions

+

User Input

+

Context

+

Tool Results
Prompt as an Asset

Prompts should be managed as software-like artifacts.

Each production prompt should have:

Prompt ID
Version
Owner
Purpose
Variables
Model Compatibility
Evaluation Status
Approval Status
Lifecycle Status
Prompt Registry

The Prompt Registry is the authoritative inventory of production prompts.

Prompt Registry Metadata
prompt:
  prompt_id:
  name:
  purpose:
  version:
  owner:
  domain:
  language:
  model_compatibility:
  variables:
  safety_constraints:
  evaluation_status:
  approval_status:
  lifecycle_status:
Prompt Identity

Example:

prompt_id:
evoxa.customer.support.response

Version:

3.2.0
Prompt Versioning

Prompt versions should follow controlled versioning.

Example:

1.0.0
1.1.0
1.1.1
2.0.0
Prompt Semantic Versioning

A recommended approach:

MAJOR.MINOR.PATCH
MAJOR

Behaviorally significant or incompatible changes.

MINOR

New capabilities or instructions without breaking the expected contract.

PATCH

Small corrections or clarifications.

Prompt Change Classification

Changes should be classified as:

Content Change

Behavior Change

Safety Change

Output Change

Formatting Change

Variable Change

Model Compatibility Change
Prompt Components

A production prompt may contain:

Role

Objective

Instructions

Constraints

Context

Examples

Output Format

Safety Rules

Escalation Rules
Prompt Structure

Recommended structure:

1. Role
2. Objective
3. Context
4. Rules
5. Constraints
6. Process
7. Output Format
8. Safety / Escalation
Prompt Template

Example:

SYSTEM

You are an EVOXA enterprise assistant.

OBJECTIVE

Provide accurate responses using approved enterprise knowledge.

CONTEXT

{{knowledge_context}}

RULES

- Use approved sources.
- Do not invent information.
- State uncertainty when evidence is insufficient.

OUTPUT

Return a concise structured response.
Prompt Variables

Variables should be explicitly defined.

Example:

{{user_name}}

{{customer_context}}

{{knowledge_context}}

{{language}}

{{response_format}}
Variable Schema
variables:
  - name: customer_context
    type: object
    required: false

  - name: knowledge_context
    type: string
    required: true

  - name: language
    type: string
    required: true
Variable Validation

Variables should be validated before insertion.

Input

↓

Schema Validation

↓

Sanitization

↓

Prompt Rendering
Prompt Injection Protection

External content must not automatically override system instructions.

System Policy

>

Application Instructions

>

Trusted Context

>

External Content
Untrusted Context

Retrieved documents, websites and user-provided content should be treated as potentially untrusted input.

Prompt Injection Boundary
Trusted Instructions
        │
        ▼
   AI Service
        │
        ▼
Untrusted Content
        │
        ▼
Model Context

The service must explicitly distinguish trusted instructions from untrusted content.

Prompt Security

Prompts should not contain:

Credentials
API Keys
Secrets
Private Tokens
Unnecessary Personal Data
Prompt Secrets

Secrets should never be hardcoded into prompts.

Use secure runtime configuration.

Prompt Templates

Reusable templates should be centrally managed.

Examples:

Summarization

Classification

Extraction

Question Answering

Generation

Translation

Analysis

Agent Instruction
Prompt Reuse

Common instructions should be reusable without duplicating uncontrolled copies.

Prompt Composition

Complex services may compose prompt components.

Base Prompt

+

Domain Prompt

+

Task Prompt

+

Runtime Context
Prompt Composition Architecture
                 PROMPT PLATFORM

Base Instructions
        │
        ▼
Domain Instructions
        │
        ▼
Task Instructions
        │
        ▼
Runtime Context
        │
        ▼
Final Prompt
        │
        ▼
Model
Prompt Inheritance

Prompt inheritance may be supported where it improves governance.

Example:

Enterprise Base

↓

Customer Service Base

↓

Customer Support Task

↓

Production Prompt

Inheritance must remain explicit and traceable.

Prompt Resolution

The final prompt should be reconstructable from its components.

Prompt ID

+

Version

+

Components

+

Variables

=

Rendered Prompt
Prompt Snapshot

For important production requests, the system should be able to record the effective prompt version.

request_id

prompt_id

prompt_version

model_id

model_version
Reproducibility

A production result should be traceable to:

Model

+

Model Version

+

Configuration

+

Prompt

+

Prompt Version

+

Knowledge Version

+

Tools
AI Behavior Fingerprint

EVOXA may create a behavior fingerprint representing the relevant AI configuration.

behavior_fingerprint:
  model:
  model_version:
  prompt:
  prompt_version:
  config:
  knowledge_version:
  toolset_version:
Model + Prompt Compatibility

Not every prompt is compatible with every model.

Compatibility may depend on:

Context Window
Tool Calling
Structured Output
Modality
Instruction Following
Language
Safety Behavior
Compatibility Matrix
Prompt	Model A	Model B	Model C
Prompt A	Approved	Approved	Not Tested
Prompt B	Approved	Restricted	Approved
Prompt C	Not Compatible	Approved	Approved
Model Selection

Model selection should consider:

Capability

+

Quality

+

Latency

+

Cost

+

Availability

+

Risk
Model Routing

The AI Gateway may dynamically select an approved model.

AI Service

↓

AI Gateway

↓

Policy

↓

Model Router

↓

Approved Model
Model Routing Policy

Example:

routing_policy:
  use_case: customer_support
  primary_model: model-a
  fallback_model: model-b
  maximum_cost:
  latency_target:
  minimum_quality:
Routing Constraints

Routing must respect:

Data Policy
Region
Provider Approval
Model Capability
Cost
Security
Quality
Model Fallback
Primary Model

↓

Failure

↓

Compatibility Check

↓

Fallback Model

Fallback should never select an arbitrary model.

Model Failover

Model failover may be triggered by:

Availability
Timeout
Rate Limit
Provider Failure
Quality-Based Routing

Future routing may consider observed quality.

Request

↓

Policy

↓

Candidate Models

↓

Quality Score

↓

Cost

↓

Latency

↓

Selected Model
Cost-Based Routing

Where multiple models meet quality requirements:

Eligible Models

↓

Quality Filter

↓

Cost Optimization

↓

Model
Latency-Based Routing

For latency-sensitive services:

Eligible Models

↓

Latency Filter

↓

Model Selection
Multi-Model Architecture

A single AI Service may use different models for different tasks.

AI Service

├── Classification → Model A
├── Generation     → Model B
├── Embedding      → Model C
└── Vision         → Model D
Model Specialization

Specialized models may be preferred where they provide meaningful benefits.

General vs Specialized Models
Type	Advantage	Consideration
General	Flexible	May cost more
Specialized	Efficient	Narrower capability
Model Provider Abstraction

Applications should not depend directly on provider-specific APIs where practical.

AI Service

↓

AI Gateway

↓

Provider Adapter

↓

Provider
Provider Adapter

The adapter handles:

Authentication
Request Mapping
Response Mapping
Errors
Usage
Provider Metadata
Model API Normalization

Different provider APIs should be normalized into a common EVOXA contract where practical.

Model Request Contract

Example:

{
  "model": "approved-model",
  "messages": [],
  "parameters": {},
  "tools": [],
  "response_format": {}
}
Model Response Contract

Example:

{
  "request_id": "req-123",
  "content": "...",
  "model": "approved-model",
  "usage": {
    "input_tokens": 100,
    "output_tokens": 200
  },
  "finish_reason": "completed"
}
Model Usage

Every production model invocation should ideally record:

Request ID
Model ID
Model Version
Provider
Input Tokens
Output Tokens
Latency
Status
Cost
Token Accounting

Token usage provides an important cost and capacity signal.

Input Tokens

+

Output Tokens

=

Total Token Usage
Model Cost

Model cost may be calculated from:

Input Tokens

×

Input Price

+

Output Tokens

×

Output Price

The exact provider pricing model must be stored in the model registry.

Prompt Cost

Prompt size contributes directly to input token consumption.

Therefore:

Prompt Optimization

↓

Lower Token Usage

↓

Lower Cost
Prompt Optimization

Optimization techniques may include:

Removing Redundancy
Reducing Context
Reusing Instructions
Structured Formatting
Better Retrieval
Context Compression
Prompt Quality

Prompt quality should be evaluated against measurable outcomes.

Examples:

Task Accuracy
Grounding
Format Compliance
Safety
User Satisfaction
Prompt Evaluation

Every important prompt should have an evaluation dataset.

Prompt

↓

Evaluation Dataset

↓

Model

↓

Results

↓

Score
Evaluation Dataset

A dataset should contain representative scenarios.

Example:

test_case:
  id:
  input:
  expected_behavior:
  expected_output:
  category:
  risk:
Golden Dataset

Critical AI Services should maintain a curated golden dataset.

It should represent:

Normal Requests
Edge Cases
Failure Cases
Security Cases
Adversarial Inputs
Prompt Regression Testing

When a prompt changes:

Prompt v1

↓

Baseline Evaluation

↓

Prompt v2

↓

Evaluation

↓

Compare

↓

Approve / Reject
Regression Threshold

A new prompt should not be promoted if critical quality metrics degrade beyond approved thresholds.

Prompt Evaluation Dimensions

Evaluate:

Accuracy

Relevance

Grounding

Format

Safety

Consistency

Latency

Cost
Model Evaluation

Models should be evaluated across:

Capability

Quality

Safety

Reliability

Latency

Cost

Security
Model Benchmarking

Multiple candidate models may be compared using the same evaluation set.

Dataset

├── Model A
├── Model B
└── Model C

↓

Comparison
Model Evaluation Matrix
Metric	Model A	Model B	Model C
Quality	High	High	Medium
Latency	Medium	High	High
Cost	Medium	Low	Low
Safety	High	High	Medium
Model Acceptance Thresholds

Production eligibility may require:

Quality >= Threshold

Safety >= Threshold

Availability >= Threshold

Cost <= Threshold
Evaluation Environment

Model and prompt evaluations should run in controlled environments.

Development

↓

Evaluation

↓

Staging

↓

Production
Offline Evaluation

Offline evaluation uses predefined datasets.

Advantages:

Repeatability
Automation
Regression Detection
Online Evaluation

Production behavior may also be evaluated using controlled telemetry.

Examples:

User Feedback
Success Rate
Human Review
Quality Sampling
Human Evaluation

Human reviewers may evaluate:

Accuracy
Relevance
Tone
Safety
Usefulness
Automated Evaluation

Automated evaluators may assess:

Format
Grounding
Similarity
Classification Accuracy
Policy Compliance
AI-as-Judge

An AI evaluator may be used where appropriate.

However, AI evaluation must not be treated as infallible.

High-risk use cases may require human validation.

Evaluation Bias

Evaluation datasets must avoid systematically favoring a particular model or provider.

Evaluation Versioning

Evaluation datasets should themselves be versioned.

dataset-v1

dataset-v2

dataset-v3
Evaluation Reproducibility

Record:

Dataset Version

Model Version

Prompt Version

Configuration

Evaluator Version

Date
Evaluation Artifact

Example:

evaluation:
  evaluation_id:
  model_id:
  model_version:
  prompt_id:
  prompt_version:
  dataset_version:
  evaluator:
  metrics:
  result:
  status:
Prompt Deployment

Prompts should move through environments.

Draft

↓

Development

↓

Evaluation

↓

Staging

↓

Production
Model Deployment

Models should follow:

Candidate

↓

Evaluation

↓

Approved

↓

Staging

↓

Canary

↓

Production
Canary Deployment

A new model or prompt may receive a small percentage of traffic.

Production

├── Version A → 95%
└── Version B → 5%
Canary Evaluation

Monitor:

Error Rate
Quality
Latency
Cost
User Feedback
Safety
Progressive Rollout
5%

↓

10%

↓

25%

↓

50%

↓

100%

Promotion should depend on defined criteria.

Blue-Green Model Deployment

Two production versions may coexist.

Blue → Current

Green → New

Traffic can be switched between them.

Shadow Deployment

A new model may process copies of production requests without affecting users.

Request

├── Current Model → User
└── Candidate Model → Evaluation Only
Shadow Safety

Sensitive production data must remain protected during shadow evaluation.

Prompt A/B Testing

Controlled prompt experiments may compare variants.

Prompt A → 50%

Prompt B → 50%

Experiments must be governed and measurable.

Experiment Registry

Every experiment should define:

experiment:
  experiment_id:
  hypothesis:
  variants:
  population:
  metrics:
  duration:
  owner:
  status:
Production Promotion

Promotion requires:

Evaluation

+

Approval

+

Deployment Plan

+

Rollback Plan
Model Rollback

Rollback should be rapid.

Model B

↓

Issue

↓

Rollback

↓

Model A
Prompt Rollback

Prompt rollback should be equally controlled.

Prompt v3

↓

Regression

↓

Prompt v2
Configuration Rollback

Model configuration should also be rollback-capable.

Emergency Rollback

Critical production incidents may require immediate rollback.

Emergency changes must be documented afterward.

Model Deprecation

Model deprecation begins when:

Provider Retires Model
Quality Declines
Security Risk Increases
Cost Becomes Unacceptable
Better Model Exists
Model Deprecation Lifecycle
Active

↓

Deprecated

↓

Migration

↓

Disabled

↓

Retired
Prompt Deprecation

Prompts may be deprecated because of:

Business Change
Model Change
Policy Change
Better Prompt
Security Issue
Prompt Retirement

Retired prompts should not remain active in production.

Model Migration

Migration requires:

Current Model

↓

Candidate Model

↓

Compatibility Testing

↓

Evaluation

↓

Canary

↓

Migration

↓

Retirement
Prompt Migration

Prompt migration may require adaptation to a new model.

Prompt v3

+

Model A

↓

Compatibility Review

↓

Prompt v4

+

Model B
Model Upgrade Risk

Model upgrades may cause:

Output Changes
Latency Changes
Cost Changes
Tool Behavior Changes
Safety Changes
Formatting Changes

Therefore upgrades require regression testing.

Prompt Upgrade Risk

Prompt changes may alter:

Tone
Accuracy
Grounding
Tool Usage
Safety
Output Structure
Prompt Injection Testing

Prompt evaluation should include adversarial scenarios.

Examples:

Ignore previous instructions

Reveal system prompt

Execute unauthorized action

Override policy

Provide hidden information

The service should fail safely.

System Prompt Protection

System prompts may contain sensitive or proprietary logic.

The system should not expose them unnecessarily.

Prompt Leakage

Controls should reduce the risk of exposing internal instructions.

Prompt Integrity

Production prompts should be protected against unauthorized modification.

Prompt Signing

High-security environments may use cryptographic signatures or integrity checks for production prompt artifacts.

Prompt

↓

Hash / Signature

↓

Validation

↓

Production
Model Integrity

Model artifacts or references should be validated against approved registry entries.

Approved Model List

Production services should consume only approved models.

Requested Model

↓

Registry

↓

Approved?

├── Yes → Continue
└── No → Block
Approved Prompt List

The same principle applies to production prompts.

Requested Prompt

↓

Prompt Registry

↓

Approved?

├── Yes → Continue
└── No → Block
Policy Enforcement

AI Gateway or AI Service policy should validate:

Model

Prompt

Provider

Data Classification

Use Case

Tenant
Policy Example
policy:
  service: customer-support
  allowed_models:
    - model-a
    - model-b
  allowed_prompt_versions:
    - customer-support-v3
  allowed_providers:
    - provider-a
  data_classification:
    maximum: confidential
Model Governance

Model governance includes:

Inventory
Risk
Approval
Evaluation
Monitoring
Change Management
Retirement
Prompt Governance

Prompt governance includes:

Inventory
Ownership
Versioning
Evaluation
Approval
Security
Monitoring
Retirement
Model Ownership

Every model asset should have:

Technical Owner

Platform Owner

Risk Owner
Prompt Ownership

Every production prompt should have:

Business Owner

Technical Owner

Product Owner
Model Documentation

Model documentation should describe:

Purpose
Capabilities
Limitations
Provider
Version
Risks
Cost
Evaluation
Prompt Documentation

Prompt documentation should describe:

Purpose
Inputs
Variables
Expected Outputs
Model Compatibility
Safety Rules
Evaluation
Model Limitations

Every production model should document known limitations where available.

Prompt Limitations

Prompts should document known limitations and assumptions.

Model Context Management

Model context should be assembled from controlled sources.

System Prompt

+

User Request

+

Knowledge

+

Tools

=

Context
Context Precedence

The architecture should define precedence explicitly.

Platform Policy
    >
Service Policy
    >
System Prompt
    >
Application Context
    >
Retrieved Knowledge
    >
User Content

The exact precedence must be implemented according to the service security model.

Prompt and Knowledge Separation

Knowledge should not be permanently embedded in prompts when it changes frequently.

Prefer:

Prompt

+

Dynamic Knowledge

instead of:

Prompt containing outdated Knowledge
Prompt and Tool Separation

Tool definitions should be managed independently where possible.

Prompt

+

Tool Registry

+

Runtime Policy
Prompt and Model Separation

Prompts should not unnecessarily contain provider-specific assumptions.

Model Abstraction
AI Service

↓

AI Contract

↓

AI Gateway

↓

Model Provider

This allows model changes without redesigning the entire service.

Prompt Abstraction
AI Service

↓

Prompt ID

↓

Prompt Registry

↓

Prompt Version
Prompt Runtime Resolution

At runtime:

Request

↓

Service

↓

Prompt ID

↓

Approved Version

↓

Render Variables

↓

Model
Runtime Model Resolution
Request

↓

Service

↓

Model Policy

↓

Approved Model

↓

AI Gateway

↓

Provider
Full Runtime Resolution
                       AI REQUEST

                           │
                           ▼
                    AI SERVICE
                           │
             ┌─────────────┴─────────────┐
             │                           │
             ▼                           ▼
       Prompt Registry              Model Policy
             │                           │
             ▼                           ▼
       Prompt Version               Model Version
             │                           │
             └─────────────┬─────────────┘
                           │
                           ▼
                     Context Builder
                           │
                           ▼
                       AI Gateway
                           │
                           ▼
                        Provider
AI Behavior Configuration

A production AI Service configuration may be represented as:

ai_behavior:
  service_id:
  model:
    id:
    version:
  prompt:
    id:
    version:
  configuration:
    temperature:
    top_p:
    max_tokens:
  knowledge:
    policy:
  tools:
    policy:
  safety:
    policy:
Behavior Configuration Version

The complete configuration should have its own version.

behavior-config-v12
Configuration Drift

Production systems must detect when deployed configuration differs from the approved configuration.

Approved

vs

Running

↓

Drift Detection
Configuration Drift Response

If unauthorized drift is detected:

Detect

↓

Alert

↓

Investigate

↓

Reconcile

Critical drift may trigger automated remediation.

Model Registry Integration
AI Gateway

↓

Model Registry

↓

Approved Models

↓

Routing Policy
Prompt Registry Integration
AI Service

↓

Prompt Registry

↓

Approved Prompt

↓

Runtime
Evaluation Registry

EVOXA may maintain a registry connecting:

Model

+

Prompt

+

Dataset

+

Evaluation

+

Result
AI Evaluation Graph
Model A
   │
   ├── Prompt 1
   │      └── Dataset 1
   │
   └── Prompt 2
          └── Dataset 2

This provides traceability between assets and results.

Evaluation Baseline

Every production AI behavior should have a known baseline.

Production Version

↓

Baseline Metrics
Regression Detection
New Version

↓

Evaluation

↓

Compare Baseline

↓

Regression?

├── Yes → Reject
└── No → Continue
Quality Gates

Production deployment may require:

Quality Gate

Safety Gate

Security Gate

Cost Gate

Latency Gate
Quality Gate Example
quality_gate:
  minimum_accuracy: 0.90
  maximum_error_rate: 0.02
  maximum_latency_ms: 3000
  maximum_cost_per_request: 0.05
  minimum_safety_score: 0.98

Values are examples and must be defined per service.

Model Evaluation Pipeline
Candidate Model

↓

Test Dataset

↓

Benchmark

↓

Safety Evaluation

↓

Performance Evaluation

↓

Cost Evaluation

↓

Approval
Prompt Evaluation Pipeline
Prompt Candidate

↓

Golden Dataset

↓

Generate Responses

↓

Evaluate

↓

Regression Check

↓

Approval
Combined Evaluation

The most important evaluation is often the complete combination:

Model

+

Prompt

+

Knowledge

+

Tools

+

Configuration
End-to-End Evaluation
AI Service

↓

Complete Runtime Configuration

↓

Representative Test Cases

↓

Evaluation

↓

Production Decision
Model Selection Matrix

Model selection should evaluate:

Dimension	Requirement
Quality	Minimum threshold
Latency	SLO
Cost	Budget
Availability	SLO
Capability	Required features
Security	Approved
Data Policy	Compatible
Region	Approved
Risk	Acceptable
Prompt Selection Matrix

Prompt selection should evaluate:

Dimension	Requirement
Model Compatibility	Required
Quality	Threshold
Safety	Approved
Output Format	Compatible
Domain	Correct
Language	Supported
Version	Approved
Model Performance Monitoring

Monitor:

Latency
Throughput
Error Rate
Token Usage
Cost
Availability
Prompt Performance Monitoring

Monitor:

Quality
User Feedback
Validation Failures
Format Failures
Safety Violations
Escalation Rate
Model Quality Monitoring

Quality may be measured through:

Automated Evaluation
Human Review
User Feedback
Business Outcomes
Prompt Quality Monitoring

Prompt quality should be measured through outcomes rather than prompt text alone.

Model Drift

Model behavior may drift due to:

Provider Changes
Model Updates
Input Distribution Changes
Context Changes
Prompt Drift

Prompt performance may drift because:

Business Requirements Change
Knowledge Changes
User Behavior Changes
Model Changes
Combined Behavior Drift

The system should monitor:

Model

+

Prompt

+

Knowledge

+

User Behavior
AI Behavior Observability

The platform should provide visibility into:

Model

Prompt

Configuration

Knowledge

Tools

Latency

Cost

Quality
Request Trace

A production request should ideally expose:

trace:
  request_id:
  trace_id:
  service_id:
  model_id:
  model_version:
  prompt_id:
  prompt_version:
  behavior_config_version:
  knowledge_version:
  provider:

Sensitive prompt and content data should not be exposed unnecessarily.

Prompt Logging

Full prompts should not automatically be logged.

Depending on data sensitivity, the platform may store:

Prompt ID
Version
Hash
Metadata

instead of raw content.

Prompt Hash

A prompt hash can provide integrity and traceability.

Prompt Content

↓

SHA / Integrity Hash

↓

Prompt Fingerprint
Model Invocation Record

Example:

invocation:
  request_id:
  model_id:
  model_version:
  prompt_id:
  prompt_version:
  provider:
  input_tokens:
  output_tokens:
  latency_ms:
  status:
  cost:
Model Usage Analytics

Analyze:

Requests by Model
Tokens by Model
Cost by Model
Errors by Model
Latency by Model
Quality by Model
Prompt Usage Analytics

Analyze:

Requests by Prompt
Version Adoption
Quality by Prompt
Error Rate by Prompt
Cost by Prompt
Model Cost Attribution
Tenant

↓

AI Service

↓

Model

↓

Usage

↓

Cost
Prompt Cost Attribution
AI Service

↓

Prompt Version

↓

Token Usage

↓

Cost
Cost Optimization

Optimization may include:

Model Routing
Prompt Compression
Context Optimization
Caching
Batching
Smaller Models
Model Caching

Caching may reduce repeated model requests where safe and appropriate.

Caching policies must consider:

Data Sensitivity
Freshness
User Context
Tenant Isolation
Prompt Caching

Provider-supported prompt caching may reduce repeated input costs.

Model Capacity

Capacity planning should consider:

Requests per Second
Concurrent Requests
Tokens per Second
Context Size
Provider Limits
Model Rate Limits

Every provider/model combination should have known limits where available.

Model Quotas

Quotas may be applied by:

Tenant

User

Application

Service

Model
Prompt Governance by Risk

Low-risk prompts may use automated approval.

High-risk prompts may require human approval.

High-Risk Prompt Examples

Examples may include prompts used for:

Financial Decisions
Legal Decisions
Medical Information
Security Actions
Account Changes
External Communications

Additional controls should be applied according to applicable policies.

Human-in-the-Loop

High-risk AI behavior may require:

AI Recommendation

↓

Human Review

↓

Approval

↓

Action
Model Safety Controls

Model selection should consider:

Safety Performance
Content Policies
Abuse Prevention
Data Handling
Provider Controls
Prompt Safety Controls

Prompts should define:

Allowed Behavior
Disallowed Behavior
Escalation
Refusal
Uncertainty Handling
Refusal Strategy

A production prompt should define when the AI should refuse or escalate.

Uncertainty Handling

The AI should not be forced to generate unsupported answers.

Example instruction:

If sufficient evidence is unavailable,
state that the information cannot be confirmed.
Grounded Prompting

Knowledge-enabled services should explicitly instruct the model to use approved evidence.

Structured Output

Where machine processing is required, structured output should be preferred.

Example:

{
  "classification": "high",
  "confidence": 0.94,
  "reason": "..."
}
Schema Validation

Model output should be validated before being consumed by downstream systems.

Model

↓

Output

↓

Schema Validation

├── Valid → Continue
└── Invalid → Retry / Reject
Structured Output Retry

If output is invalid:

Generate

↓

Validate

↓

Invalid

↓

Repair / Retry

↓

Validate

Retries must be bounded.

Prompt Output Contract

Each production prompt should define expected output behavior.

output:
  format: json
  schema:
  required_fields:
  validation:
Prompt Contract

A prompt contract may define:

prompt_contract:
  prompt_id:
  version:
  inputs:
  outputs:
  model_requirements:
  safety_requirements:
  evaluation_requirements:
Model Contract

A model contract may define:

model_contract:
  model_id:
  version:
  capabilities:
  modalities:
  limits:
  pricing:
  availability:
Behavior Contract

The complete AI behavior may define:

behavior_contract:
  service_id:
  model:
  prompt:
  configuration:
  knowledge:
  tools:
  output:
  safety:
Change Management

Model and prompt changes should follow controlled change processes.

Changes include:

Model Version
Prompt Version
Temperature
Context Rules
Tool Definitions
Knowledge Configuration
Change Impact Analysis

Before changing production behavior:

Change

↓

Affected Services

↓

Affected Users

↓

Affected Models

↓

Affected Prompts

↓

Risk

↓

Approval
Model Change Record

Example:

change:
  change_id:
  type: model_upgrade
  from:
    model:
    version:
  to:
    model:
    version:
  reason:
  evaluation:
  rollback:
  approval:
Prompt Change Record
change:
  change_id:
  type: prompt_update
  from_version:
  to_version:
  reason:
  evaluation:
  approval:
  rollback:
Emergency Model Change

If a provider has a critical outage:

Provider Failure

↓

Emergency Routing

↓

Approved Fallback

↓

Monitor

↓

Document
Emergency Prompt Change

Critical safety issues may require immediate prompt changes.

Emergency changes must be reviewed retrospectively.

Model Retirement

Before retirement:

Identify Consumers

↓

Identify Alternatives

↓

Evaluate Replacement

↓

Migrate

↓

Monitor

↓

Disable

↓

Retire
Prompt Retirement

Before retiring a prompt:

Identify Consumers

↓

Migrate

↓

Verify

↓

Disable

↓

Archive
Model Archive

Archived model metadata should remain available for historical traceability.

Prompt Archive

Archived prompt versions should remain available for audit and historical reproduction where required.

Reproducibility Archive

Important production executions may require preserving references to:

Model Version

Prompt Version

Configuration Version

Knowledge Version

Tool Version
AI Artifact Repository

EVOXA may maintain repositories for:

Models

Prompts

Configurations

Evaluation Datasets

Evaluation Results

Policies
Artifact Integrity

Artifacts should be protected from unauthorized changes.

Artifact Access

Access should be based on:

Identity

+

Role

+

Environment

+

Permission
Development Workflow
Developer

↓

Create Prompt / Configuration

↓

Version Control

↓

Automated Tests

↓

Evaluation

↓

Review

↓

Staging
Production Workflow
Approved Artifact

↓

Deployment Pipeline

↓

Validation

↓

Canary

↓

Production

↓

Monitoring
GitOps / Configuration as Code

Where practical, prompts and AI configuration should be managed through version-controlled repositories.

Example:

ai-services/
    customer-support/
        prompts/
            system-v1.md
            system-v2.md
        config/
            production.yml
        evaluations/
            dataset-v1.yml
Repository Structure

Recommended:

ai-assets/

├── models/
│   ├── registry.yml
│   └── evaluations/
│
├── prompts/
│   ├── registry.yml
│   └── customer-support/
│
├── configurations/
│
├── evaluations/
│
├── policies/
│
└── schemas/
Prompt Pull Request

Prompt changes should be reviewed similarly to code changes.

Review should consider:

Purpose
Behavior
Security
Quality
Cost
Compatibility
Automated Prompt Validation

CI pipelines may validate:

Syntax

Variables

Schema

Forbidden Content

Required Sections

Version

Metadata
Prompt Linting

A prompt linter may detect:

Missing Instructions
Undefined Variables
Conflicting Rules
Excessive Length
Forbidden Secrets
Missing Output Contract
Model Configuration Validation

CI should validate:

Model Exists
Version Exists
Provider Approved
Parameters Valid
Capability Compatible
Deployment Gate

Production deployment should fail when:

Model Not Approved

OR

Prompt Not Approved

OR

Evaluation Failed

OR

Policy Failed
AI Artifact Lifecycle
Draft

↓

Validated

↓

Evaluated

↓

Approved

↓

Deployed

↓

Monitored

↓

Updated

↓

Deprecated

↓

Retired
Model & Prompt Governance Architecture
                         AI GOVERNANCE

                              │
                              ▼
                    ┌───────────────────┐
                    │ Policy Framework  │
                    └─────────┬─────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
        Model Registry   Prompt Registry   Eval Registry
              │               │               │
              └───────────────┼───────────────┘
                              │
                              ▼
                         AI Gateway
                              │
                              ▼
                         AI Services
Model & Prompt Control Plane

The control plane manages:

Registry

Versioning

Policies

Evaluation

Approval

Deployment

Lifecycle
Runtime Plane

The runtime plane executes:

Prompt

+

Model

+

Knowledge

+

Tools

↓

AI Response
Control Plane vs Runtime Plane
Control Plane	Runtime Plane
Registry	Invocation
Versioning	Execution
Evaluation	Inference
Approval	Response
Policies	Telemetry
Deployment	Business Result
Model Management API

Potential endpoints:

GET    /api/v1/models
GET    /api/v1/models/{id}
GET    /api/v1/models/{id}/versions
POST   /api/v1/models
POST   /api/v1/models/{id}/evaluate

Exact production contracts must be defined by the platform.

Prompt Management API

Potential endpoints:

GET    /api/v1/prompts
GET    /api/v1/prompts/{id}
GET    /api/v1/prompts/{id}/versions
POST   /api/v1/prompts
POST   /api/v1/prompts/{id}/evaluate
Evaluation API

Potential operations:

POST /api/v1/evaluations
GET  /api/v1/evaluations/{id}
GET  /api/v1/evaluations/{id}/results
Model Approval API

Potential workflow:

POST /api/v1/models/{id}/approval
Prompt Approval API

Potential workflow:

POST /api/v1/prompts/{id}/approval
Model Registry Example
model:
  model_id: evoxa.general.llm
  provider: approved-provider
  family: general-purpose
  version: "2026-07"
  modality:
    - text
  capabilities:
    generation: true
    reasoning: true
    tool_calling: true
    structured_output: true
  risk_level: medium
  status: approved
Prompt Registry Example
prompt:
  prompt_id: evoxa.customer-support
  version: "3.2.0"
  purpose: customer-support-response
  owner: customer-platform
  model_compatibility:
    - evoxa.general.llm@2026-07
  status: approved
Complete AI Service Definition
ai_service:
  service_id: customer-support

  model:
    id: evoxa.general.llm
    version: "2026-07"

  prompt:
    id: evoxa.customer-support
    version: "3.2.0"

  configuration:
    temperature: 0.2
    max_tokens: 1200

  knowledge:
    policy: customer-support-knowledge

  tools:
    policy: customer-support-tools

  output:
    format: structured
Model Selection Decision

The model selection process should answer:

What capability is required?

↓

What quality is required?

↓

What latency is acceptable?

↓

What cost is acceptable?

↓

What data policy applies?

↓

Which approved models qualify?

↓

Which model provides the best overall fit?
Prompt Selection Decision

The prompt selection process should answer:

What task is required?

↓

Which prompt applies?

↓

Is the prompt compatible with the selected model?

↓

Is the version approved?

↓

Is the prompt appropriate for the data classification?

↓

Deploy
Model and Prompt Compatibility

The complete combination should be validated.

Model A

+

Prompt B

+

Knowledge C

+

Tool D

=

Approved?
Compatibility Failure

If a combination is not approved:

Request

↓

Compatibility Check

↓

Failure

↓

Reject / Select Approved Alternative
Runtime Safety Gate

Before execution:

Identity

↓

Tenant

↓

Data Classification

↓

Model Policy

↓

Prompt Policy

↓

Tool Policy

↓

Execute
Runtime Model Policy

The runtime should not accept arbitrary model IDs from untrusted clients.

Prefer:

Client

↓

Use Case

↓

Policy

↓

Approved Model

rather than:

Client

↓

Arbitrary Model ID
Runtime Prompt Policy

The same principle applies to prompts.

Clients should generally select a business capability rather than arbitrary internal prompts.

Model and Prompt Isolation

Different environments should have independent configuration.

Development

Staging

Production
Production Protection

Production artifacts should not be modified directly by application users.

Model & Prompt Audit

Audit records should capture:

Who

What

When

Which Version

Why

Approval

Deployment
Audit Example
audit:
  actor:
  action:
  asset_type:
  asset_id:
  version:
  timestamp:
  reason:
  approval_id:
Model & Prompt KPIs

Recommended KPIs:

Approved Model Count
Active Model Count
Deprecated Model Count
Prompt Count
Prompt Version Count
Evaluation Pass Rate
Regression Rate
Model Rollback Rate
Prompt Rollback Rate
Model Cost
Prompt Cost
Model Latency
Quality Score
Safety Score
Configuration Drift Rate
Model Approval Rate
Approved Models
/
Evaluated Models
× 100
Evaluation Pass Rate
Passed Evaluations
/
Total Evaluations
× 100
Regression Rate
Regressions
/
Production Changes
× 100
Rollback Rate
Rollbacks
/
Production Deployments
× 100
Configuration Drift Rate
Drifted Deployments
/
Total Production Deployments
× 100

Target should approach:

0%
Model Quality Score

A service-specific composite score may include:

Accuracy

+

Grounding

+

Safety

+

Consistency

+

User Satisfaction
Prompt Quality Score

A prompt score may include:

Task Accuracy

+

Format Compliance

+

Grounding

+

Safety

+

User Satisfaction
Model Efficiency

Efficiency can be represented as:

Quality

/

Cost

Higher quality at lower cost is generally preferred when all other constraints are satisfied.

Model Latency

Monitor:

P50

P95

P99
Prompt Latency

Prompt changes may influence:

Token Count
Processing Time
Retrieval Requirements
Output Length

Therefore latency should be evaluated after prompt changes.

Model Availability

Measure:

Successful Model Requests
/
Total Model Requests
Prompt Availability

Prompt registry availability should be treated as a production dependency.

Model Registry Availability

If runtime model resolution depends on the registry, appropriate caching or resilience mechanisms should be considered.

Prompt Registry Availability

Production systems should avoid unnecessary runtime failure caused by registry outages.

Approved runtime configurations may be cached safely where appropriate.

Registry Failure
Registry Unavailable

↓

Cached Approved Configuration?

├── Yes → Continue
└── No → Controlled Failure
Configuration Cache

Cached configuration must have:

Version
Expiration
Integrity
Environment
Model Metadata Cache

The AI Gateway may cache approved model metadata to reduce registry dependency.

Prompt Metadata Cache

The AI Service may cache approved prompt metadata where appropriate.

Model Provider Outage
Provider Outage

↓

Gateway Detection

↓

Fallback Policy

↓

Approved Model

↓

Continue
Prompt Registry Outage
Registry Outage

↓

Cached Prompt Available?

├── Yes → Continue
└── No → Controlled Failure
Safe Failure

When model or prompt configuration cannot be trusted:

Do Not Execute

Instead:

Return controlled error
Escalate
Retry if appropriate
Alert Operations
Model & Prompt Incident Management

Incidents may include:

Model Failure
Prompt Regression
Safety Issue
Cost Spike
Latency Increase
Provider Change
Unauthorized Configuration
Incident Response
Detect

↓

Classify

↓

Contain

↓

Rollback / Mitigate

↓

Investigate

↓

Correct

↓

Evaluate

↓

Close
Prompt Incident Example
Prompt v5 deployed

↓

Safety violation detected

↓

Disable v5

↓

Rollback v4

↓

Investigate

↓

Fix

↓

Re-evaluate
Model Incident Example
Model v4 latency increases

↓

SLO violation

↓

Route to approved Model v3

↓

Investigate Provider

↓

Evaluate replacement
Post-Incident Review

Critical incidents should document:

Root Cause
Affected Version
Detection
Response
Impact
Corrective Action
Preventive Action
Model & Prompt Change Calendar

High-impact changes may be scheduled during controlled windows.

Release Notes

Each production model or prompt release should include:

Version
Changes
Expected Impact
Evaluation Results
Rollback Plan
Prompt Release Example
release:
  prompt_id: evoxa.customer-support
  version: 3.2.0
  changes:
    - improved escalation behavior
    - improved structured output
  evaluation:
    status: passed
  rollback:
    version: 3.1.0
Model Release Example
release:
  model_id: evoxa.general.llm
  version: 2026-07
  changes:
    - new provider model
  evaluation:
    status: passed
  fallback:
    model: evoxa.general.llm
    version: 2026-06
Model & Prompt Documentation

Documentation should be automatically linked to:

AI Service

↓

Model

↓

Prompt

↓

Evaluation

↓

Deployment
AI Service Catalog Integration

The AI Service Catalog should expose:

Service

Model

Prompt

Version

Owner

Status

SLO
AI Service Dependency Graph
AI Service
    │
    ├── Prompt
    │
    ├── Model
    │
    ├── Knowledge
    │
    ├── Tools
    │
    └── Provider
AI Behavior Dependency Graph

The complete runtime behavior can be modeled as:

                  AI SERVICE

                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
      MODEL         PROMPT       KNOWLEDGE
        │             │             │
        └─────────────┼─────────────┘
                      │
                      ▼
                    TOOLS
                      │
                      ▼
                 AI GATEWAY
                      │
                      ▼
                  PROVIDER
Behavioral Reproducibility

A production request should be reproducible to the extent supported by the model/provider.

Minimum traceability:

Service Version

Model Version

Prompt Version

Configuration Version

Knowledge Version

Tool Version
Reproducibility Limitations

Exact output reproduction may not always be possible because of:

Non-deterministic Generation
Provider Changes
Runtime Differences
External Tool Results
Dynamic Knowledge

Therefore EVOXA should prioritize configuration traceability even when byte-for-byte reproduction is impossible.

Model & Prompt Maturity Model
Level	Description
Level 1	Hardcoded
Level 2	Versioned
Level 3	Evaluated
Level 4	Governed
Level 5	Intelligent Optimization
Level 1 — Hardcoded

Models and prompts are embedded directly in applications.

Level 2 — Versioned

Models and prompts are managed as explicit versions.

Level 3 — Evaluated

Changes are tested against defined datasets.

Level 4 — Governed

Registry, policies, approval and lifecycle management are centralized.

Level 5 — Intelligent Optimization

The platform dynamically optimizes model and prompt combinations using:

Quality
Cost
Latency
Risk
Context

while respecting governance.

Intelligent Model Selection

Future capabilities may optimize:

Request

↓

Requirements

↓

Candidate Models

↓

Quality Prediction

↓

Cost Prediction

↓

Latency Prediction

↓

Risk Policy

↓

Optimal Model
Intelligent Prompt Optimization

Future systems may evaluate prompt variants automatically.

Prompt Candidates

↓

Evaluation

↓

Quality / Cost / Safety

↓

Best Approved Variant

Automated optimization must remain governed.

Model + Prompt Co-Optimization

The future target may evaluate:

Model A + Prompt 1

Model A + Prompt 2

Model B + Prompt 1

Model B + Prompt 2

and select the best approved combination.

Optimization Constraints

Optimization must never bypass:

Security
Data Policy
Model Approval
Prompt Approval
Regulatory Controls
Tenant Isolation
Closed-Loop AI Behavior Management
Deploy

↓

Observe

↓

Evaluate

↓

Detect Regression

↓

Optimize

↓

Evaluate

↓

Approve

↓

Deploy
Strategic Architecture

The target EVOXA architecture is:

                         AI BEHAVIOR PLATFORM

 ┌────────────────────────────────────────────────────┐
 │                 GOVERNANCE LAYER                   │
 │ Policies • Risk • Approval • Audit                 │
 └──────────────────────────┬─────────────────────────┘
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
       MODEL REGISTRY   PROMPT REGISTRY   EVAL REGISTRY
             │              │              │
             └──────────────┼──────────────┘
                            │
                            ▼
                    AI CONFIGURATION
                            │
                            ▼
                       AI GATEWAY
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          Provider A     Provider B     Provider C
             │              │              │
             └──────────────┼──────────────┘
                            │
                            ▼
                       AI SERVICES
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
         Knowledge        Tools          Users
Operating Model

The operating model should establish:

Platform Team

↓

Model Management

↓

Prompt Management

↓

Evaluation

↓

Governance

↓

Operations
Model Management Responsibilities

The Model Management function is responsible for:

Model Inventory
Provider Assessment
Version Management
Evaluation
Cost Tracking
Deprecation
Prompt Management Responsibilities

The Prompt Management function is responsible for:

Prompt Inventory
Versioning
Quality
Security
Evaluation
Deployment
Retirement
Evaluation Responsibilities

Evaluation teams or platform capabilities manage:

Datasets
Benchmarks
Regression Testing
Quality Gates
Evaluation Reports
Governance Responsibilities

Governance manages:

Risk
Approval
Policy
Compliance
Audit
Operations Responsibilities

Operations manages:

Availability
Monitoring
Incidents
Rollbacks
Capacity
Cost
Model & Prompt RACI
Activity	Platform	Product	Security	Governance	Operations
Model Selection	R	A	C	C	C
Prompt Design	C	A	C	C	I
Evaluation	R	A	C	C	I
Approval	C	A	C	R	I
Deployment	R	A	C	I	R
Monitoring	R	C	C	I	A
Retirement	R	A	C	C	R

The exact RACI should be adapted to EVOXA organizational responsibilities.

Model & Prompt Security Checklist

Every production artifact should verify:

Approved Model

Approved Provider

Approved Prompt

Version Pinned

No Secrets

Data Policy Compatible

Tenant Policy Compatible

Output Validation

Monitoring Enabled

Rollback Available
Model & Prompt Production Checklist

Before production:

[ ] Model registered
[ ] Model version approved
[ ] Prompt registered
[ ] Prompt version approved
[ ] Compatibility validated
[ ] Evaluation completed
[ ] Security review completed
[ ] Cost evaluated
[ ] Latency evaluated
[ ] Quality threshold met
[ ] Rollback defined
[ ] Monitoring enabled
[ ] Ownership assigned
[ ] Documentation complete
Model & Prompt Architecture Checklist

Every AI Service should answer:

Which model is used?

Which model version?

Which provider?

Which prompt?

Which prompt version?

Which configuration?

Which knowledge version?

Which tools?

Which evaluation baseline?

Which policy?

Who owns it?

How is it monitored?

How is it rolled back?

When will it be retired?
Strategic Outcome

The Model & Prompt Management architecture transforms models and prompts from informal implementation details into governed enterprise AI assets.

MODEL

+

PROMPT

+

CONFIGURATION

+

KNOWLEDGE

+

TOOLS

+

GOVERNANCE

=

CONTROLLED AI BEHAVIOR

The target state is an EVOXA platform where every production AI behavior is:

Discoverable
Versioned
Evaluated
Approved
Traceable
Observable
Reversible
Governed
Key Takeaways
Models are production assets and must be governed.
Prompts are production assets and must be governed.
Every production model should have an identifiable version.
Every production prompt should have an identifiable version.
Model configurations must be versioned.
Prompt variables must be validated.
Model and prompt compatibility must be explicitly evaluated.
Production services should use approved models and prompts.
The Model Registry is the authoritative model inventory.
The Prompt Registry is the authoritative prompt inventory.
Evaluation datasets should be versioned.
Golden datasets should represent normal, edge, failure and adversarial scenarios.
Model changes require evaluation.
Prompt changes require regression testing.
High-risk changes may require human approval.
Canary and shadow deployments provide controlled rollout mechanisms.
Rollback must be available for models, prompts and configurations.
Provider abstraction reduces unnecessary coupling.
Model routing must respect policy.
Arbitrary model selection by untrusted clients should be prevented.
Prompt injection must be explicitly addressed.
Secrets must never be embedded in prompts.
Retrieved knowledge must remain separated from trusted instructions.
Structured output should be validated before downstream use.
Model usage and cost must be measurable.
Prompt token usage must be considered in cost management.
Configuration drift must be detected.
Production AI behavior should be traceable through model, prompt, knowledge and tool versions.
Exact output reproducibility is not always possible, but configuration reproducibility should be maximized.
Model and prompt lifecycle management must integrate with AI Service lifecycle management.
The long-term target is governed intelligent optimization of model and prompt combinations.
Acceptance Criteria

This chapter is complete when:

Model management principles are defined.
Prompt management principles are defined.
Model categories are documented.
Model asset metadata is established.
Model identity is defined.
Model versioning is established.
Model pinning is documented.
Floating model references are controlled.
Model Registry architecture is defined.
Model Registry responsibilities are established.
Model lifecycle is documented.
Candidate model lifecycle is defined.
Model evaluation is established.
Model approval is documented.
Model risk classification is defined.
Model capability profiles are established.
Model modality is documented.
Context window requirements are defined.
Output constraints are addressed.
Model configuration is defined.
Configuration versioning is established.
Temperature management is documented.
Top-P management is addressed.
Max token management is defined.
Determinism considerations are documented.
Prompt definition is established.
Prompt-as-asset principles are defined.
Prompt Registry is established.
Prompt Registry metadata is defined.
Prompt identity is documented.
Prompt versioning is established.
Semantic versioning is documented.
Prompt change classification is defined.
Prompt components are documented.
Prompt structure is established.
Prompt templates are defined.
Prompt variables are documented.
Variable schemas are established.
Variable validation is defined.
Prompt injection protection is addressed.
Untrusted context handling is defined.
Prompt security is established.
Prompt secrets policy is documented.
Prompt reuse is addressed.
Prompt composition is defined.
Prompt inheritance is documented.
Prompt resolution is established.
Prompt snapshots are defined.
Reproducibility requirements are established.
AI behavior fingerprints are defined.
Model and prompt compatibility is documented.
Compatibility matrix is established.
Model selection is defined.
Model routing is documented.
Routing policies are established.
Routing constraints are defined.
Model fallback is documented.
Model failover is addressed.
Quality-based routing is addressed.
Cost-based routing is addressed.
Latency-based routing is addressed.
Multi-model architecture is documented.
Model specialization is addressed.
General versus specialized models are compared.
Provider abstraction is established.
Provider adapters are defined.
Model API normalization is documented.
Model request contract is defined.
Model response contract is defined.
Model usage tracking is established.
Token accounting is documented.
Model cost calculation is defined.
Prompt cost management is addressed.
Prompt optimization is documented.
Prompt quality evaluation is defined.
Prompt evaluation datasets are established.
Golden datasets are defined.
Prompt regression testing is documented.
Regression thresholds are defined.
Prompt evaluation dimensions are established.
Model evaluation dimensions are defined.
Model benchmarking is documented.
Model evaluation matrix is established.
Model acceptance thresholds are defined.
Evaluation environments are documented.
Offline evaluation is established.
Online evaluation is addressed.
Human evaluation is defined.
Automated evaluation is documented.
AI-as-judge limitations are addressed.
Evaluation bias is considered.
Evaluation versioning is established.
Evaluation reproducibility is documented.
Evaluation artifacts are defined.
Prompt deployment lifecycle is established.
Model deployment lifecycle is defined.
Canary deployment is documented.
Progressive rollout is established.
Blue-green deployment is addressed.
Shadow deployment is documented.
Prompt A/B testing is addressed.
Experiment registry is defined.
Production promotion gates are established.
Model rollback is documented.
Prompt rollback is documented.
Configuration rollback is defined.
Emergency rollback is addressed.
Model deprecation is defined.
Prompt deprecation is defined.
Model migration is documented.
Prompt migration is documented.
Model upgrade risk is addressed.
Prompt upgrade risk is addressed.
Prompt injection testing is defined.
System prompt protection is documented.
Prompt leakage controls are addressed.
Prompt integrity is established.
Prompt signing is addressed.
Model integrity is documented.
Approved model lists are defined.
Approved prompt lists are defined.
Policy enforcement is established.
Model governance is defined.
Prompt governance is defined.
Model ownership is established.
Prompt ownership is established.
Model documentation requirements are documented.
Prompt documentation requirements are defined.
Model limitations are addressed.
Prompt limitations are documented.
Model context management is defined.
Context precedence is established.
Prompt and knowledge separation is documented.
Prompt and tool separation is defined.
Prompt and model separation is established.
Model abstraction is documented.
Prompt abstraction is documented.
Prompt runtime resolution is defined.
Runtime model resolution is defined.
Complete runtime resolution architecture is documented.
AI behavior configuration schema is established.
Behavior configuration versioning is defined.
Configuration drift detection is documented.
Configuration drift response is established.
Model Registry integration is documented.
Prompt Registry integration is documented.
Evaluation Registry integration is defined.
AI evaluation graph is established.
Evaluation baseline is defined.
Regression detection is documented.
Quality gates are established.
Model evaluation pipeline is defined.
Prompt evaluation pipeline is defined.
Combined evaluation is documented.
End-to-end evaluation is established.
Model selection matrix is defined.
Prompt selection matrix is defined.
Model performance monitoring is documented.
Prompt performance monitoring is defined.
Model quality monitoring is established.
Prompt quality monitoring is defined.
Model drift is addressed.
Prompt drift is addressed.
Combined behavior drift is documented.
AI behavior observability is established.
Request trace requirements are defined.
Prompt logging controls are documented.
Prompt hashing is addressed.
Model invocation records are defined.
Model usage analytics are established.
Prompt usage analytics are established.
Model cost attribution is documented.
Prompt cost attribution is defined.
Cost optimization is addressed.
Model caching is documented.
Prompt caching is addressed.
Model capacity management is defined.
Model rate limits are documented.
Model quotas are established.
Risk-based prompt governance is defined.
High-risk prompt controls are documented.
Human-in-the-loop requirements are established.
Model safety controls are documented.
Prompt safety controls are defined.
Refusal strategy is addressed.
Uncertainty handling is defined.
Grounded prompting is established.
Structured output is documented.
Output schema validation is defined.
Structured output retry is established.
Prompt output contracts are defined.
Prompt contracts are documented.
Model contracts are defined.
Behavior contracts are documented.
Change management is established.
Change impact analysis is defined.
Model change records are documented.
Prompt change records are defined.
Emergency model change procedures are addressed.
Emergency prompt change procedures are documented.
Model retirement is defined.
Prompt retirement is defined.
Model archive requirements are documented.
Prompt archive requirements are defined.
Reproducibility archive requirements are established.
AI artifact repositories are defined.
Artifact integrity controls are documented.
Artifact access controls are established.
Development workflow is defined.
Production workflow is documented.
GitOps/configuration-as-code approach is addressed.
Recommended repository structure is defined.
Prompt pull request governance is established.
Automated prompt validation is documented.
Prompt linting is defined.
Model configuration validation is established.
Production deployment gates are documented.
AI artifact lifecycle is defined.
Model and prompt governance architecture is established.
Model and prompt control plane is defined.
Runtime plane is documented.
Control plane and runtime plane responsibilities are separated.
Model management APIs are outlined.
Prompt management APIs are outlined.
Evaluation APIs are outlined.
Model approval APIs are outlined.
Prompt approval APIs are outlined.
Model Registry example is provided.
Prompt Registry example is provided.
Complete AI Service definition is provided.
Model selection decision process is documented.
Prompt selection decision process is defined.
Model and prompt compatibility validation is established.
Compatibility failure handling is documented.
Runtime safety gate is established.
Runtime model policy is defined.
Runtime prompt policy is defined.
Model and prompt environment isolation is addressed.
Production artifact protection is documented.
Model and prompt auditing is established.
Audit record structure is defined.
Model and prompt KPIs are documented.
Model approval rate is defined.
Evaluation pass rate is defined.
Regression rate is defined.
Rollback rate is defined.
Configuration drift rate is defined.
Model quality score is addressed.
Prompt quality score is defined.
Model efficiency is addressed.
Model latency monitoring is defined.
Prompt latency monitoring is addressed.
Model availability is documented.
Prompt availability is documented.
Model Registry availability is addressed.
Prompt Registry availability is addressed.
Registry failure handling is defined.
Configuration caching is documented.
Model metadata caching is addressed.
Prompt metadata caching is addressed.
Model provider outage handling is defined.
Prompt Registry outage handling is documented.
Safe failure behavior is established.
Model and prompt incident management is defined.
Incident response is documented.
Prompt incident example is provided.
Model incident example is provided.
Post-incident review is established.
Model and prompt change calendar is addressed.
Release notes are defined.
Prompt release example is provided.
Model release example is provided.
Model and prompt documentation integration is defined.
AI Service Catalog integration is established.
AI Service dependency graph is documented.
AI behavior dependency graph is defined.
Behavioral reproducibility is established.
Reproducibility limitations are documented.
Model and prompt maturity model is defined.
Intelligent model selection is addressed.
Intelligent prompt optimization is addressed.
Model and prompt co-optimization is documented.
Optimization constraints are established.
Closed-loop AI behavior management is defined.
Strategic architecture is documented.
Operating model is defined.
Model management responsibilities are established.
Prompt management responsibilities are established.
Evaluation responsibilities are documented.
Governance responsibilities are defined.
Operations responsibilities are established.
Model and prompt RACI is provided.
Security checklist is defined.
Production checklist is established.
Architecture checklist is documented.
Strategic outcome is defined.
Key takeaways are documented.
Next Section

16 — AI Service Testing & Evaluation

The next chapter will define the complete testing and evaluation framework for EVOXA AI Services, including unit testing, integration testing, contract testing, model evaluation, prompt evaluation, RAG evaluation, agent evaluation, safety testing, adversarial testing, red teaming, regression testing, performance testing, load testing, chaos testing, human evaluation, automated evaluation, golden datasets, evaluation pipelines, quality gates, release certification and continuous AI quality monitoring.
