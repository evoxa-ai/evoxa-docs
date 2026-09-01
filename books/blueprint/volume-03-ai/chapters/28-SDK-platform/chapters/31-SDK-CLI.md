# 28/31 — SDK CLI

## 1. Document Purpose

SDK CLI defines the command-line interface, developer tooling, project management, authentication, configuration, application lifecycle, SDK operations, testing, sandbox execution, deployment, observability and automation capabilities required to manage the EVOXA SDK Platform directly from the terminal.

The SDK CLI provides a programmatic developer interface to EVOXA.

The fundamental model is:

```text
Developer
    ↓
Terminal
    ↓
EVOXA CLI
    ↓
SDK Platform
    ↓
Application / API / AI / Agent / Tools / Sandbox
```

SDK CLI builds directly on:

```text
28/01 — SDK Overview
28/02 — SDK Business Model
28/03 — SDK Strategy
28/04 — SDK Architecture
28/05 — SDK Core
28/06 — SDK Runtime
28/07 — SDK Lifecycle
28/08 — SDK Versioning
28/09 — SDK Languages
28/10 — SDK Generation
28/11 — SDK API Clients
28/12 — SDK Authentication
28/13 — SDK Authorization
28/14 — SDK Identity
28/15 — SDK Multi-Tenancy
28/16 — SDK Organizations
28/17 — SDK Data Access
28/18 — SDK Events
28/19 — SDK Webhooks
28/20 — SDK Workflows
28/21 — SDK Automation
28/22 — SDK AI
28/23 — SDK Agents
28/24 — SDK Tools
28/25 — SDK Memory
28/26 — SDK Observability
28/27 — SDK Testing
28/28 — SDK Sandbox
28/29 — SDK Developer Portal
28/30 — SDK Documentation
```

SDK CLI therefore becomes the **command-line control and automation layer** of the EVOXA SDK Platform.

---

# 2. SDK CLI Definition

The SDK CLI is a command-line tool that allows developers and platform teams to interact with EVOXA without requiring direct interaction with the web-based Developer Portal.

Conceptually:

```text
EVOXA CLI
 ├── Authentication
 ├── Projects
 ├── Applications
 ├── SDKs
 ├── APIs
 ├── Environments
 ├── Credentials
 ├── Testing
 ├── Sandbox
 ├── AI
 ├── Agents
 ├── Tools
 ├── Memory
 ├── Workflows
 ├── Automation
 ├── Observability
 └── Deployment
```

---

# 3. CLI Mission

The mission is:

> **Provide developers and technical teams with a fast, scriptable and automation-friendly interface for building, testing, configuring, deploying and operating applications on the EVOXA SDK Platform.**

---

# 4. CLI Principles

SDK CLI follows:

```text
Simplicity
Consistency
Discoverability
Scriptability
Automation
Security
Portability
Predictability
Extensibility
Developer Productivity
```

---

# 5. CLI Architecture

```text
                              DEVELOPER
                                  │
                                  ▼
                               TERMINAL
                                  │
                                  ▼
                              EVOXA CLI
                                  │
          ┌───────────────────────┼───────────────────────┐
          ▼                       ▼                       ▼
      COMMANDS                CONFIGURATION           AUTH
          │                       │                       │
          └───────────────────────┼───────────────────────┘
                                  ▼
                              SDK / API
                                  │
              ┌───────────────────┼───────────────────┐
              ▼                   ▼                   ▼
           PLATFORM            SANDBOX             TESTING
              │                   │                   │
              └───────────────────┼───────────────────┘
                                  ▼
                              OBSERVABILITY
                                  │
                                  ▼
                                RESULT
```

---

# 6. CLI Installation

The CLI should provide straightforward installation for supported operating systems.

Potential platforms:

```text
Linux
macOS
Windows
```

---

# 7. CLI Distribution

Potential distribution mechanisms include:

```text
Package Manager
Binary
Installer
Container
```

depending on the final EVOXA implementation.

---

# 8. CLI Version

The CLI should expose its version.

Conceptually:

```text
evoxa --version
```

---

# 9. CLI Help

Every command should provide contextual help.

```text
evoxa --help
evoxa <command> --help
```

---

# 10. Command Discovery

The CLI should make available commands discoverable.

Conceptually:

```text
evoxa
```

may display major command groups.

---

# 11. Command Structure

The general model is:

```text
evoxa <resource> <action>
```

Examples:

```text
evoxa app create
evoxa app list
evoxa app deploy
evoxa sandbox run
evoxa test run
```

---

# 12. Command Groups

Potential top-level groups:

```text
auth
config
project
app
sdk
api
env
credential
test
sandbox
ai
agent
tool
memory
workflow
automation
logs
metrics
trace
deploy
org
tenant
```

---

# 13. CLI Authentication

Authentication should integrate with EVOXA Identity.

Conceptually:

```text
evoxa auth login
```

---

# 14. Login Flow

```text
CLI
 ↓
Authentication
 ↓
Browser / Device Flow
 ↓
EVOXA Identity
 ↓
Token
 ↓
CLI Session
```

---

# 15. Authentication Methods

Depending on EVOXA capabilities, the CLI may support:

```text
Interactive Login
Device Authorization
API Credentials
Service Credentials
CI/CD Credentials
```

---

# 16. Session Management

The CLI should manage authenticated sessions securely.

---

# 17. Logout

Conceptually:

```text
evoxa auth logout
```

---

# 18. Authentication Status

Developers should be able to inspect authentication status.

```text
evoxa auth status
```

---

# 19. Credential Security

CLI credentials should be stored using secure operating-system mechanisms where possible.

---

# 20. Credential Isolation

Different organizations and environments should not accidentally share credentials.

---

# 21. Credential Profiles

The CLI may support named profiles.

```text
default
development
staging
production
```

---

# 22. Profile Selection

Conceptually:

```text
evoxa --profile production ...
```

---

# 23. Organization Context

CLI commands may operate against a selected organization.

```text
evoxa org list
evoxa org use <organization>
```

---

# 24. Tenant Context

Where applicable, commands may specify tenant context.

---

# 25. Environment Context

Commands should clearly identify the target environment.

```text
development
testing
staging
production
```

---

# 26. Project Management

The CLI should provide project management.

```text
evoxa project create
evoxa project list
evoxa project init
```

---

# 27. Project Initialization

The CLI may initialize a new EVOXA project.

Conceptually:

```text
evoxa project init
```

---

# 28. Project Structure

A generated project may contain:

```text
project/
├── src/
├── tests/
├── config/
├── evoxa/
├── .env.example
├── package configuration
└── project metadata
```

Actual structure depends on the selected language and framework.

---

# 29. Project Configuration

Projects should maintain EVOXA configuration.

---

# 30. Configuration File

Conceptually:

```text
evoxa.yaml
```

or an equivalent configuration format.

---

# 31. Configuration Management

Configuration may define:

```text
Project
Application
Environment
SDK
Runtime
Region
```

---

# 32. Environment Variables

The CLI should support environment-variable configuration for automation and CI/CD.

---

# 33. Configuration Precedence

A predictable configuration precedence should be established.

Conceptually:

```text
Command Arguments
        ↓
Environment Variables
        ↓
Project Configuration
        ↓
User Configuration
        ↓
Defaults
```

---

# 34. Application Management

The CLI should manage EVOXA applications.

```text
evoxa app create
evoxa app list
evoxa app get
evoxa app update
evoxa app delete
```

---

# 35. Application Initialization

The CLI may connect a local project to an EVOXA application.

```text
evoxa app init
```

---

# 36. Application Linking

A local project should be linkable to:

```text
Application
Organization
Environment
```

---

# 37. Application Status

Developers should be able to inspect application status.

---

# 38. Application Configuration

CLI commands should support application configuration.

---

# 39. SDK Management

The CLI may help developers install and configure EVOXA SDKs.

---

# 40. SDK Discovery

Conceptually:

```text
evoxa sdk list
```

---

# 41. SDK Information

```text
evoxa sdk info
```

may expose:

```text
Version
Language
Compatibility
Status
```

---

# 42. SDK Generation

The CLI may invoke SDK generation.

```text
evoxa sdk generate
```

---

# 43. SDK Update

The CLI may help update SDK dependencies.

---

# 44. SDK Validation

The CLI may validate SDK compatibility.

---

# 45. API Discovery

The CLI may provide API discovery.

```text
evoxa api list
evoxa api get <api>
```

---

# 46. API Invocation

Authorized developers may invoke APIs directly from the CLI.

Conceptually:

```text
evoxa api call <endpoint>
```

---

# 47. API Request Configuration

The CLI may support:

```text
Headers
Parameters
Body
Authentication
Environment
```

---

# 48. API Output

CLI API responses should support human-readable output.

---

# 49. Machine-Readable Output

The CLI should support formats such as:

```text
JSON
YAML
Table
```

where appropriate.

---

# 50. JSON Output

Conceptually:

```text
evoxa app list --output json
```

This is important for automation.

---

# 51. Table Output

Human-friendly terminal output may use tables.

---

# 52. Quiet Mode

Automation should be able to suppress unnecessary output.

Conceptually:

```text
--quiet
```

---

# 53. Verbose Mode

Debugging may support:

```text
--verbose
```

---

# 54. Debug Mode

A debug mode may expose additional diagnostic information without exposing secrets.

---

# 55. Error Handling

CLI errors should be:

```text
Clear
Actionable
Machine-Readable
Consistent
```

---

# 56. Exit Codes

The CLI should return meaningful process exit codes.

Example model:

```text
0 → Success
1 → General Failure
2 → Invalid Usage
3 → Authentication Failure
4 → Authorization Failure
5 → Resource Failure
6 → Execution Failure
```

Exact codes should be standardized during implementation.

---

# 57. Automation Compatibility

CLI output and exit codes must be reliable for scripts.

```text
Script
 ↓
EVOXA CLI
 ↓
Exit Code
 ↓
Automation
```

---

# 58. Testing

The CLI should integrate with SDK Testing.

```text
evoxa test run
```

---

# 59. Test Selection

Developers should be able to select:

```text
Unit
Integration
Contract
E2E
Sandbox
```

tests.

---

# 60. Test Output

The CLI should provide:

```text
Passed
Failed
Skipped
Duration
Errors
```

---

# 61. Test Watch Mode

Future capabilities may support continuous test execution.

```text
evoxa test watch
```

---

# 62. Test Coverage

Where supported:

```text
evoxa test coverage
```

---

# 63. Sandbox

The CLI should provide direct access to SDK Sandbox.

```text
evoxa sandbox create
evoxa sandbox run
evoxa sandbox list
evoxa sandbox delete
```

---

# 64. Sandbox Execution

Conceptually:

```text
evoxa sandbox run script.py
```

---

# 65. Sandbox Configuration

CLI users may configure:

```text
Runtime
CPU
Memory
Timeout
Network
Storage
Permissions
```

---

# 66. Sandbox Result

The CLI should display:

```text
Status
Output
Errors
Duration
Usage
```

---

# 67. Sandbox Logs

Sandbox logs should be accessible through the CLI.

---

# 68. AI Commands

The CLI may expose EVOXA AI capabilities.

Potential structure:

```text
evoxa ai models
evoxa ai run
evoxa ai inspect
```

---

# 69. AI Execution

Conceptually:

```text
evoxa ai run
```

may execute a controlled AI request.

---

# 70. AI Configuration

CLI configuration may include:

```text
Model
Parameters
Tools
Memory
Environment
```

---

# 71. Agent Commands

The CLI may manage agents.

```text
evoxa agent create
evoxa agent list
evoxa agent get
evoxa agent run
evoxa agent test
```

---

# 72. Agent Development

The CLI may initialize agent projects.

```text
evoxa agent init
```

---

# 73. Agent Testing

Agents should be testable locally or in controlled EVOXA environments.

---

# 74. Agent Execution

Agent execution should respect:

```text
Identity
Permissions
Tools
Memory
Sandbox
Policies
```

---

# 75. Tool Commands

The CLI may manage tools.

```text
evoxa tool list
evoxa tool create
evoxa tool test
evoxa tool invoke
```

---

# 76. Tool Invocation

Tool execution must respect authorization.

---

# 77. Memory Commands

The CLI may provide memory management.

```text
evoxa memory list
evoxa memory get
evoxa memory search
evoxa memory delete
```

---

# 78. Memory Security

Memory operations must remain tenant, organization and permission scoped.

---

# 79. Workflow Commands

The CLI may manage workflows.

```text
evoxa workflow create
evoxa workflow list
evoxa workflow validate
evoxa workflow run
```

---

# 80. Workflow Validation

The CLI may validate workflow definitions before deployment.

---

# 81. Automation Commands

The CLI may manage automations.

```text
evoxa automation list
evoxa automation create
evoxa automation enable
evoxa automation disable
```

---

# 82. Event Commands

The CLI may provide event management.

```text
evoxa event publish
evoxa event list
```

---

# 83. Webhook Commands

The CLI may manage webhooks.

```text
evoxa webhook list
evoxa webhook create
evoxa webhook delete
evoxa webhook test
```

---

# 84. Webhook Testing

The CLI may generate controlled webhook test events.

---

# 85. Observability Commands

The CLI should integrate with SDK Observability.

Potential commands:

```text
evoxa logs
evoxa metrics
evoxa traces
evoxa events
```

---

# 86. Log Retrieval

Developers should be able to retrieve authorized logs.

---

# 87. Log Filtering

Filtering may include:

```text
Application
Environment
Time
Severity
Execution
Trace
```

---

# 88. Trace Retrieval

The CLI may retrieve distributed traces.

---

# 89. Metrics

Developers may query application metrics.

---

# 90. Error Diagnostics

The CLI may expose recent application errors.

---

# 91. Deployment

The CLI should provide deployment capabilities where supported.

```text
evoxa deploy
```

---

# 92. Deployment Flow

```text
Local Project
 ↓
Validate
 ↓
Test
 ↓
Build
 ↓
Package
 ↓
Deploy
 ↓
Verify
```

---

# 93. Deployment Validation

Before deployment:

```text
Configuration
Dependencies
Tests
Security
Environment
```

may be validated.

---

# 94. Deployment Targets

Possible targets:

```text
Development
Testing
Staging
Production
```

---

# 95. Deployment Approval

Production deployment may require explicit authorization or approval.

---

# 96. Deployment Status

```text
evoxa deploy status
```

may expose deployment progress and result.

---

# 97. Deployment History

Developers should be able to inspect deployment history.

---

# 98. Rollback

Where supported:

```text
evoxa deploy rollback
```

may restore a previous version.

---

# 99. Environment Management

The CLI should manage environments.

```text
evoxa env list
evoxa env get
evoxa env create
evoxa env use
```

---

# 100. Environment Variables

The CLI may manage environment configuration while protecting secrets.

---

# 101. Credential Management

The CLI may manage application credentials.

```text
evoxa credential list
evoxa credential create
evoxa credential revoke
evoxa credential rotate
```

---

# 102. Secret Protection

Credential values must not appear in command history or standard output unnecessarily.

---

# 103. Organization Commands

Enterprise users may manage organization context.

```text
evoxa org list
evoxa org get
```

---

# 104. Team Commands

Future capabilities may include:

```text
evoxa team list
evoxa team members
```

---

# 105. Tenant Commands

Where applicable:

```text
evoxa tenant list
evoxa tenant use
```

---

# 106. Project Templates

The CLI may create projects from templates.

```text
evoxa init --template ai-agent
```

---

# 107. Application Templates

Potential templates:

```text
AI Application
Agent
API Service
Workflow
Automation
Integration
```

---

# 108. Code Generation

The CLI may generate:

```text
Client Code
Models
API Wrappers
Configuration
Tests
```

---

# 109. Code Scaffolding

A generated project should provide a functional starting point rather than an empty directory.

---

# 110. Local Development

The CLI may support local development workflows.

```text
evoxa dev
```

---

# 111. Development Server

Where applicable, local development may provide:

```text
Hot Reload
Local Configuration
Logs
Testing
Sandbox
```

---

# 112. Local Sandbox

The CLI may connect local development directly to EVOXA Sandbox.

---

# 113. Local Testing

Developers should be able to run tests without manually configuring every platform dependency.

---

# 114. Local Configuration Validation

The CLI may validate project configuration before execution.

---

# 115. Dependency Validation

The CLI may detect incompatible dependencies.

---

# 116. SDK Diagnostics

The CLI may provide diagnostics:

```text
evoxa doctor
```

Potential checks:

```text
CLI Version
Authentication
SDK
Project
Environment
Network
Credentials
Permissions
```

---

# 117. Developer Diagnostics

```text
evoxa doctor
```

should produce actionable recommendations.

---

# 118. Documentation Integration

The CLI should integrate with SDK Documentation.

Potential behavior:

```text
evoxa docs
evoxa docs search <query>
```

---

# 119. Contextual Documentation

Errors may provide links or references to relevant documentation.

---

# 120. Command Documentation

Each command should have:

```text
Description
Syntax
Arguments
Options
Examples
Errors
```

---

# 121. CLI Completion

The CLI should support shell completion where practical.

Potential shells:

```text
Bash
Zsh
PowerShell
Fish
```

---

# 122. Completion Generation

Conceptually:

```text
evoxa completion
```

---

# 123. Interactive Mode

Future versions may provide an interactive CLI mode.

```text
evoxa shell
```

---

# 124. Interactive Developer Environment

The interactive mode may expose:

```text
Projects
Applications
APIs
Sandbox
Testing
Logs
```

---

# 125. CI/CD Integration

CLI is a primary automation interface for CI/CD.

```text
Repository
 ↓
CI
 ↓
EVOXA CLI
 ↓
Test
 ↓
Build
 ↓
Deploy
```

---

# 126. CI Authentication

CI environments should use non-interactive authentication.

---

# 127. Service Credentials

Service credentials should be scoped to the minimum required permissions.

---

# 128. CI Security

CI credentials should not provide unrestricted developer privileges.

---

# 129. Deployment Pipelines

The CLI should support repeatable deployment pipelines.

---

# 130. Infrastructure Automation

CLI commands should be deterministic enough to support infrastructure automation.

---

# 131. Scripting

Every major operation should be scriptable.

```text
Script
 ↓
EVOXA CLI
 ↓
Platform
```

---

# 132. Idempotency

Where possible, configuration commands should be idempotent.

---

# 133. Declarative Configuration

Future versions may support declarative project configuration.

```text
evoxa.yaml
      ↓
evoxa apply
      ↓
EVOXA Platform
```

---

# 134. Apply Model

Conceptually:

```text
Desired State
 ↓
Compare
 ↓
Plan
 ↓
Apply
```

---

# 135. Plan

Future infrastructure-style workflows may support:

```text
evoxa plan
```

before applying changes.

---

# 136. Diff

The CLI may display configuration differences.

```text
evoxa diff
```

---

# 137. Configuration Drift

The CLI may detect drift between local configuration and platform state.

---

# 138. Export

Authorized configuration may be exported.

```text
evoxa app export
```

---

# 139. Import

Compatible configuration may be imported.

```text
evoxa app import
```

---

# 140. Backup

Future CLI functionality may support controlled application configuration backups.

---

# 141. CLI Plugins

The CLI may support extensions.

```text
evoxa plugin list
evoxa plugin install
evoxa plugin remove
```

---

# 142. Plugin Security

Plugins must operate under defined security and trust policies.

---

# 143. Plugin Registry

Future EVOXA capabilities may provide an official CLI plugin registry.

---

# 144. CLI Extensibility

Extensions may add:

```text
Commands
Integrations
Templates
Diagnostics
Deployment Targets
```

---

# 145. Output Standardization

All commands should follow consistent output conventions.

---

# 146. JSON Schema

Machine-readable output should remain stable enough for automation.

---

# 147. API Compatibility

CLI commands should use stable EVOXA APIs rather than hidden internal interfaces.

---

# 148. CLI Telemetry

Where permitted, the CLI may collect operational telemetry.

---

# 149. Privacy

Telemetry should respect EVOXA privacy policies and enterprise configuration.

---

# 150. CLI Logging

CLI diagnostic logs should be separate from application logs where appropriate.

---

# 151. Debug Bundles

Future capabilities may generate sanitized diagnostic bundles.

---

# 152. CLI Security

The CLI must protect:

```text
Credentials
Tokens
Secrets
Application Data
Tenant Data
```

---

# 153. Secret Redaction

Secrets should be redacted from:

```text
Output
Logs
Errors
Diagnostics
```

---

# 154. Command Confirmation

Destructive operations may require confirmation.

Examples:

```text
app delete
credential revoke
environment delete
```

---

# 155. Force Mode

Automation may explicitly bypass interactive confirmation.

```text
--force
```

should be used carefully and only where appropriate.

---

# 156. Destructive Operation Safety

The CLI should clearly identify irreversible operations.

---

# 157. Permission Enforcement

CLI authorization must be enforced server-side.

The CLI cannot be considered a security boundary by itself.

---

# 158. Tenant Isolation

CLI requests must remain tenant-scoped according to EVOXA authorization.

---

# 159. Organization Isolation

Organization resources must remain isolated according to permissions.

---

# 160. Audit

Sensitive CLI operations should generate audit events.

---

# 161. CLI Audit Context

Audit records may identify:

```text
Developer
Application
Organization
Tenant
Command
Resource
Timestamp
Result
```

---

# 162. CLI Observability

CLI operations may integrate with EVOXA tracing and diagnostics.

```text
CLI
 ↓
API
 ↓
Platform
 ↓
Trace
```

---

# 163. Request IDs

CLI errors should expose request or correlation identifiers where useful.

---

# 164. Incident Troubleshooting

The CLI should help developers diagnose:

```text
Authentication
Authorization
Network
API
SDK
Sandbox
Deployment
```

problems.

---

# 165. Offline Behavior

The CLI should clearly distinguish:

```text
Local Operation
Network Operation
```

and fail predictably when network connectivity is unavailable.

---

# 166. Network Configuration

Enterprise environments may require:

```text
Proxy
Custom Endpoint
Private Network
Certificate Configuration
```

where supported.

---

# 167. Enterprise Proxy Support

The CLI should support enterprise network environments where practical.

---

# 168. Regional Endpoints

The CLI may allow supported EVOXA regional endpoints to be selected.

---

# 169. CLI Performance

The CLI should optimize:

```text
Startup Time
Authentication
API Requests
Output Rendering
```

---

# 170. CLI Reliability

Network failures should support appropriate retry behavior without duplicating non-idempotent operations.

---

# 171. Retry Policies

Retry behavior should respect:

```text
Timeout
Backoff
Idempotency
Server Signals
```

---

# 172. CLI Testing

The CLI itself should be tested through:

```text
Unit Tests
Integration Tests
End-to-End Tests
Compatibility Tests
Security Tests
```

---

# 173. Cross-Platform Testing

The CLI should be validated across supported operating systems.

---

# 174. Shell Testing

Command behavior should be validated across supported shells.

---

# 175. Backward Compatibility

CLI command behavior should maintain compatibility across minor releases where possible.

---

# 176. CLI Versioning

CLI releases should use explicit versioning.

---

# 177. CLI Upgrade

Developers should have a straightforward upgrade path.

---

# 178. CLI Update Notification

The CLI may notify developers when a newer version is available.

---

# 179. Deprecation

Deprecated CLI commands should provide migration guidance.

---

# 180. CLI Release Notes

Every release should document:

```text
Features
Changes
Fixes
Breaking Changes
Deprecations
```

---

# 181. CLI Documentation

CLI documentation belongs in SDK Documentation and Developer Portal.

---

# 182. CLI Examples

Each command should have practical examples.

---

# 183. CLI Learning Path

The recommended learning path:

```text
Install
 ↓
Login
 ↓
Init
 ↓
Configure
 ↓
Develop
 ↓
Test
 ↓
Sandbox
 ↓
Deploy
 ↓
Observe
```

---

# 184. First CLI Success

A developer should be able to reach a first successful platform operation quickly.

---

# 185. CLI Developer Experience

Ideal experience:

```text
$ evoxa login

$ evoxa init

$ evoxa test

$ evoxa sandbox run

$ evoxa deploy

$ evoxa logs
```

---

# 186. CLI + Developer Portal

The portal and CLI should provide equivalent core capabilities where practical.

```text
Developer Portal
       ↕
      EVOXA
       ↕
    EVOXA CLI
```

---

# 187. CLI + SDK

The CLI should complement rather than replace application SDKs.

```text
CLI
 ↓
Development / Operations

SDK
 ↓
Application Runtime
```

---

# 188. CLI + Documentation

Documentation should provide copyable CLI commands.

---

# 189. CLI + Sandbox

The CLI becomes the fastest interface for sandbox experimentation.

```text
Code
 ↓
evoxa sandbox run
 ↓
Result
```

---

# 190. CLI + Testing

The CLI becomes the primary automation interface for SDK Testing.

---

# 191. CLI + Observability

Developers should be able to diagnose applications without leaving the terminal.

```text
evoxa logs
evoxa metrics
evoxa traces
```

---

# 192. CLI + AI

AI-assisted CLI capabilities may eventually provide natural-language developer workflows.

Conceptually:

```text
Developer
 ↓
EVOXA AI
 ↓
CLI Operations
 ↓
Sandbox
 ↓
Tests
```

---

# 193. AI-Assisted CLI

Future commands may allow:

```text
evoxa ai explain-error
evoxa ai generate
evoxa ai troubleshoot
```

subject to final platform design.

---

# 194. Agent-Assisted CLI

Future agents may operate the CLI under controlled authorization.

```text
Developer Agent
 ↓
EVOXA CLI
 ↓
Platform
```

---

# 195. Autonomous CLI Operations

Long-term automation may allow agents to:

```text
Inspect
Test
Diagnose
Deploy
Monitor
Optimize
```

within explicitly authorized boundaries.

---

# 196. Human Approval

High-risk CLI operations may require explicit human approval.

---

# 197. Policy Enforcement

Autonomous CLI operations remain governed by:

```text
Identity
Authorization
Policy
Environment
Risk
```

---

# 198. Final CLI Architecture

```text
                         EVOXA CLI
                             │
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                    ▼
      AUTH                 CONFIG              PROJECT
        │                    │                    │
        └────────────────────┼────────────────────┘
                             ▼
                         APPLICATION
                             │
            ┌────────────────┼────────────────┐
            ▼                ▼                ▼
           SDK              API              ENV
            │                │                │
            └────────────────┼────────────────┘
                             ▼
                   ┌─────────┼─────────┐
                   ▼         ▼         ▼
                TEST      SANDBOX     AI
                   │         │         │
                   └─────────┼─────────┘
                             ▼
                       AGENTS / TOOLS
                             │
                             ▼
                    MEMORY / WORKFLOWS
                             │
                             ▼
                       AUTOMATION
                             │
                             ▼
                         DEPLOY
                             │
                             ▼
                      OBSERVABILITY
                             │
                             ▼
                          GOVERNANCE
```

---

# 199. Final SDK CLI Definition

SDK CLI is the **command-line control and automation layer of the EVOXA SDK Platform**, providing developers, DevOps teams, platform engineers and enterprise operators with a scriptable interface for authentication, project management, application configuration, SDK management, API interaction, testing, sandbox execution, AI and agent development, deployment, observability and operational automation.

The fundamental model is:

```text
INSTALL
   ↓
LOGIN
   ↓
INIT
   ↓
CONFIGURE
   ↓
BUILD
   ↓
TEST
   ↓
SANDBOX
   ↓
DEPLOY
   ↓
OBSERVE
   ↓
AUTOMATE
```

The complete CLI lifecycle becomes:

```text
Developer
    ↓
EVOXA CLI
    ↓
Identity
    ↓
Organization / Tenant
    ↓
Project
    ↓
Application
    ↓
Environment
    ↓
SDK / API
    ↓
Testing
    ↓
Sandbox
    ↓
Deployment
    ↓
Observability
    ↓
Governance
```

The CLI evolution is:

```text
Command Line Tool
       ↓
Developer CLI
       ↓
SDK Management CLI
       ↓
Application Platform CLI
       ↓
DevOps / CI-CD CLI
       ↓
AI-Assisted CLI
       ↓
Agent-Driven CLI
       ↓
Autonomous Developer Operations
```

The ultimate developer model becomes:

```text
                           DEVELOPER
                               │
                               ▼
                           EVOXA CLI
                               │
        ┌──────────────────────┼──────────────────────┐
        ▼                      ▼                      ▼
      PROJECT               APPLICATION             SDK
        │                      │                      │
        └──────────────────────┼──────────────────────┘
                               ▼
                           ENVIRONMENT
                               │
            ┌──────────────────┼──────────────────┐
            ▼                  ▼                  ▼
         TESTING            SANDBOX               AI
            │                  │                  │
            └──────────────────┼──────────────────┘
                               ▼
                           AGENTS / TOOLS
                               │
                               ▼
                         MEMORY / WORKFLOW
                               │
                               ▼
                            DEPLOY
                               │
                               ▼
                         PRODUCTION
                               │
                               ▼
                        OBSERVABILITY
                               │
                               ▼
                           GOVERNANCE
```

The ultimate automation model becomes:

```text
                         CI / CD / AGENT
                                │
                                ▼
                           EVOXA CLI
                                │
                                ▼
                             VALIDATE
                                │
                                ▼
                              TEST
                                │
                                ▼
                             SANDBOX
                                │
                                ▼
                              BUILD
                                │
                                ▼
                             DEPLOY
                                │
                                ▼
                             VERIFY
                                │
                                ▼
                            OBSERVE
                                │
                                ▼
                            OPTIMIZE
```

Therefore:

> **SDK CLI transforms the EVOXA SDK Platform into a programmable developer and operations environment, allowing developers and automated systems to perform the complete application lifecycle directly from the terminal.**

Its strategic objective is to make EVOXA:

```text
Scriptable
Automatable
Fast
Consistent
Secure
Portable
Observable
CI/CD Ready
AI-Assisted
Enterprise-Ready
```

while ensuring that every CLI operation remains governed by EVOXA Identity, Authorization, Tenant Isolation, Security, Policy and Audit controls.

The long-term EVOXA CLI architecture becomes:

```text
                         EVOXA COMMAND FABRIC
                                  │
                                  ▼
                              EVOXA CLI
                                  │
          ┌───────────────────────┼───────────────────────┐
          ▼                       ▼                       ▼
       PROJECT                 APPLICATION              SDK
          │                       │                       │
          └───────────────────────┼───────────────────────┘
                                  ▼
                              PLATFORM
                                  │
          ┌───────────────────────┼───────────────────────┐
          ▼                       ▼                       ▼
        TESTING                 SANDBOX                  AI
          │                       │                       │
          └───────────────────────┼───────────────────────┘
                                  ▼
                            AGENTS / TOOLS
                                  │
                                  ▼
                           MEMORY / WORKFLOW
                                  │
                                  ▼
                              AUTOMATION
                                  │
                                  ▼
                              DEPLOYMENT
                                  │
                                  ▼
                           OBSERVABILITY
                                  │
                                  ▼
                             GOVERNANCE
                                  │
                                  ▼
                       AUTONOMOUS OPERATIONS
```

**SDK CLI therefore becomes the terminal interface to the EVOXA Platform, connecting human developers, CI/CD systems and future AI agents to the complete EVOXA application lifecycle through a secure, consistent and automation-first command-line experience.**
