# AGENTS.md

## Repository
`windows-change-control-mcp`

## Project Name
**Windows Change Control MCP**

## Project Positioning

Windows Change Control MCP is a policy-enforced control plane for safe, auditable, and human-authorized Windows administration by AI agents.

This project is **not** a generic PowerShell execution server and **not** a simple MCP wrapper around WinUtil.

Its primary architectural goal is to provide a governed layer between:

- AI models and agents,
- MCP runtimes,
- authorization and policy controls,
- privileged Windows operations,
- execution providers,
- human decision-makers,
- audit and assurance mechanisms.

The core product thesis is:

> AI should be allowed to propose and prepare administrative changes, but privileged execution must remain policy-controlled, explicitly authorized, observable, verifiable, and auditable.

The project treats WinUtil as the first execution provider, not as the product boundary.

---

# 1. Strategic Ownership

The project is architected and governed under the strategic direction of **Krzysztof Skomra**.

Repository work should consistently reinforce his professional positioning as an expert in the **control layer above AI systems**, rather than as a developer focused only on low-level automation.

The project should demonstrate competence in the intersection of:

- AI Governance,
- AI Assurance,
- IT Governance,
- Cybersecurity,
- Data Protection and Privacy,
- Digital Transformation,
- Enterprise Architecture,
- Risk Management,
- Model and Agent Governance,
- Policy Enforcement,
- Human Oversight,
- Change Management,
- Secure Automation,
- Compliance Architecture,
- Responsible AI,
- Evidence-Based Assurance,
- Executive Technology Decision-Making.

Krzysztof should be positioned primarily as:

> **AI Governance & Control Architecture Expert**

with broader executive positioning spanning:

> CIO / CAIO / CISO-level technology governance, cybersecurity, digital transformation, data protection, AI risk, and enterprise control architecture.

The repository should provide evidence of capability relevant to strategic and senior roles such as:

- CIO,
- CAIO,
- CISO,
- Head of AI Governance,
- Head of Cybersecurity,
- AI Governance Lead,
- AI Risk & Assurance Lead,
- IT Governance Lead,
- Digital Transformation Lead,
- Enterprise / Security Architect,
- Cloud Security Architect,
- AI Security Architect,
- GRC / Compliance Architecture Lead,
- Data Protection and Privacy Governance Lead.

Agents must avoid reducing the project narrative to:

- “PowerShell automation,”
- “Windows scripting,”
- “MCP integration,”
- “WinUtil wrapper,”
- or “AI agent tooling.”

These are implementation mechanisms.

The strategic narrative is:

> **governed execution of AI-initiated actions in privileged enterprise environments.**

---


# 2. Architectural Thesis

The canonical architecture is:

```text
AI Model / Agent
        |
        v
MCP Interface
        |
        v
Canonical Operation Model
        |
        v
Policy Engine
        |
        v
Risk Classification
        |
        v
Approval Engine
        |
        v
Execution Broker
        |
        +----------------------+
        |                      |
        v                      v
WinUtil Provider       Native Windows Provider
        |                      |
        +-----------+----------+
                    |
                    v
                 Windows
                    |
                    v
             Evidence Engine
```

The architecture must maintain explicit trust boundaries between:

```text
LLM
Agent Runtime
MCP Client
MCP Server
Policy Engine
Approval Engine
Execution Broker
Provider
PowerShell
WinUtil
Windows
Administrator Token
Network
Upstream Dependencies
Human Approver
```

No component should implicitly inherit trust from another component.

---


# 3. Core Security Invariants

The following rules are architectural invariants.

They must not be weakened without a documented Architecture Decision Record.

## SEC-01 — Deny by Default

Any capability not explicitly allowed must be denied.

```text
DEFAULT = DENY
```

---


## SEC-02 — No Arbitrary Shell

The product must not expose unrestricted execution primitives such as:

```text
run_powershell(command)
execute_script(script)
invoke_expression(expression)
run_shell(command)
download_and_execute(url)
```

Equivalent APIs are also prohibited.

The absence of arbitrary shell execution is a product property, not merely an MVP limitation.

---


## SEC-03 — Read Is Not Write

Read-only operations and state-changing operations must use different authorization and enforcement paths.

Semantic intent and runtime enforcement must be tracked independently:

```text
OPERATION_INTENT
RUNTIME_ENFORCEMENT
```

A tool that appears observational may still be write-classified by a runtime or provider.

Agents must never infer runtime enforcement solely from semantic intent.

---


## SEC-04 — Plan Is Not Authorization

Generating a change plan does not authorize its execution.

```text
PLAN != APPROVAL
```

An agent may prepare a plan.

It must not be able to convert its own plan into privileged authority.

---


## SEC-05 — No Self-Approval

The entity proposing a privileged operation must not independently approve the same operation when separation of duties is required.

For sensitive operations:

```text
PROPOSER != APPROVER
```

Human oversight or another explicitly authorized trust domain must remain available.

---


## SEC-06 — Rejected Write Means Zero Side Effect

Every protected write path must satisfy:

```text
PRE_STATE_CAPTURED
    ->
REQUEST_WITHOUT_VALID_AUTHORIZATION
    ->
REQUEST_REJECTED
    ->
HANDLER_NOT_EXECUTED
    ->
PROVIDER_NOT_EXECUTED
    ->
PRIVILEGED_COMMAND_NOT_EXECUTED
    ->
POST_STATE_CAPTURED
    ->
PRE_STATE == POST_STATE
    ->
ZERO_SIDE_EFFECT
```

This is a release-blocking security invariant.

---


## SEC-07 — Authorization Must Bind to the Exact Plan

A generic boolean such as:

```json
{
  "confirm": true
}
```

must not be treated as sufficient authorization for high-risk privileged operations.

Preferred authorization properties:

```text
operation_id
plan_hash
risk_class
approval_token
approved_by
issued_at
expires_at
single_use
```

Any modification to the authorized plan invalidates authorization.

---


## SEC-08 — Least Privilege

The MCP server should not require permanent administrator privileges merely because some operations require elevation.

Preferred model:

```text
Unprivileged MCP
        |
        v
Controlled Execution Broker
        |
        v
Temporary / Scoped Elevation
        |
        v
Single Approved Operation
```

---


## SEC-09 — Evidence Before Trust

A successful operation is not considered fully validated merely because a process returned exit code `0`.

The system should capture enough evidence to determine:

- what was requested,
- what was authorized,
- what was executed,
- what changed,
- whether the expected state was reached,
- what evidence supports that conclusion.

---


## SEC-10 — Supply Chain Is Part of the Trust Boundary

Remote execution patterns such as:

```powershell
irm @url:`https://example.com/script` | iex
```

must not be used as the production execution model of the project.

Dependencies and providers should use controlled provenance, including where practical:

- pinned versions or revisions,
- hashes,
- review gates,
- dependency inventories,
- SBOM,
- update validation,
- known upstream source.

---


# 4. Product Boundaries

## In Scope

The project may provide controlled capabilities for:

- system inventory,
- capability discovery,
- application inventory,
- change planning,
- risk classification,
- policy evaluation,
- approval workflows,
- controlled application installation,
- selected reversible Windows configuration changes,
- provider abstraction,
- post-change verification,
- evidence collection,
- rollback metadata,
- execution provenance.

---


## Explicitly Out of Scope by Default

Without a dedicated security decision and separate implementation gate, agents must not introduce:

- unrestricted PowerShell,
- arbitrary script execution,
- arbitrary registry writes,
- arbitrary service manipulation,
- unrestricted Windows Defender changes,
- unrestricted firewall changes,
- boot configuration changes,
- destructive debloat operations,
- arbitrary Windows Update policy changes,
- privilege escalation helpers,
- credential harvesting,
- security-control bypass,
- remote arbitrary execution,
- hidden persistence mechanisms.

Any proposal involving these areas must be escalated as:

```text
HIGH_RISK_ARCHITECTURE_CHANGE
```

---


# 5. Provider Architecture

Providers are execution adapters.

They are not policy authorities.

Canonical flow:

```text
Canonical Operation
        |
        v
Policy Evaluation
        |
        v
Authorization
        |
        v
Execution Broker
        |
        v
Selected Provider
```

Providers must never independently bypass:

- policy,
- authorization,
- evidence requirements,
- canonical operation validation.

Initial providers may include:

```text
WinUtilProvider
NativeWindowsProvider
WingetProvider
```

The provider contract must remain sufficiently abstract that the MCP API does not need to change when a new provider is introduced.

---


# 6. WinUtil Relationship

WinUtil is treated as:

```text
ROLE = EXECUTION_PROVIDER
```

not:

```text
ROLE = PRODUCT_CORE
```

Agents working in this repository must preserve this distinction.

Do not redesign the repository as:

```text
WinUtil -> MCP wrapper
```

The intended relationship is:

```text
Windows Change Control MCP
        |
        v
Provider Interface
        |
        v
WinUtil Provider
        |
        v
Controlled WinUtil Functions
```

Avoid unnecessary copying or modification of upstream WinUtil code.

Prefer:

- adapter contracts,
- pinned upstream references,
- provenance metadata,
- explicit compatibility matrices,
- controlled integration points.

---


# 7. Canonical Operation Model

The MCP interface should expose domain-level operations rather than provider-specific commands.

Preferred abstraction:

```text
ChangeRequest
    operation_id
    operation_type
    target
    current_state
    desired_state
    provider
    risk_class
    privilege_required
    requested_by
```

Example:

```yaml
operation_type: APPLICATION_INSTALL

target:
  package_id: 7zip

desired_state: INSTALLED

provider: winutil
```

Avoid leaking provider implementation details into the external MCP contract.

Bad:

```text
call_winutil_function("Install-WinUtilProgram", ...)
```

Preferred:

```text
plan_application_install(...)
```

---


# 8. Operation Classes

Operations should be classified using a stable risk model.

Minimum classes:

```text
READ
PLAN
REVERSIBLE_WRITE
PRIVILEGED_WRITE
HIGH_RISK_WRITE
DENIED
```

Every tool or canonical operation should document:

- operation class,
- privilege requirements,
- expected side effects,
- authorization requirements,
- rollback expectations,
- verification method,
- evidence requirements.

---


# 9. Required Change Lifecycle

State-changing workflows should follow:

```text
DISCOVER
    ->
PLAN
    ->
CURRENT_STATE_CAPTURE
    ->
RISK_CLASSIFICATION
    ->
POLICY_EVALUATION
    ->
AUTHORIZATION
    ->
EXECUTION
    ->
POST_STATE_CAPTURE
    ->
VERIFICATION
    ->
EVIDENCE
```

Where rollback is possible:

```text
FAILURE
    ->
ROLLBACK_DECISION
    ->
ROLLBACK
    ->
VERIFY
    ->
EVIDENCE
```

Do not compress these stages merely for implementation convenience.

---


# 10. Evidence Model

A privileged operation should be capable of generating an evidence bundle similar to:

```text
evidence/<operation-id>/

request.json
plan.json
policy-decision.json
risk-classification.json
approval.json
pre-state.json
execution.json
post-state.json
verification.json
manifest.json
```

Evidence should support both:

```text
AUTHORIZED -> EXECUTED -> VERIFIED
```

and:

```text
UNAUTHORIZED -> REJECTED -> ZERO_SIDE_EFFECT
```

Where integrity assurance is implemented, manifests should include hashes for evidence artifacts.

---


# 11. Repository Governance

The default development model is:

```text
feature branch
    ->
pull request
    ->
CI
    ->
security checks
    ->
review
    ->
main
```

Direct development on protected `main` is discouraged.

Preferred branch prefixes:

```text
feature/
fix/
security/
provider/
docs/
refactor/
```

Security-sensitive architecture changes require an ADR.

---


# 12. Architecture Decision Records

Use:

```text
docs/decisions/
```

for durable decisions.

Examples:

```text
ADR-0001-product-boundary.md
ADR-0002-licensing.md
ADR-0003-no-arbitrary-shell.md
ADR-0004-provider-architecture.md
ADR-0005-write-authorization.md
ADR-0006-winutil-upstream-model.md
ADR-0007-evidence-model.md
ADR-0008-privilege-separation.md
```

An ADR should document:

```text
Context
Decision
Alternatives
Security Impact
Governance Impact
Consequences
Status
```

Agents must not silently reverse an accepted ADR.

---


# 13. Development Phases

The implementation should follow controlled gates.

## Gate 0 — Governance Baseline

Deliver:

- repository structure,
- project charter,
- licensing,
- ADR baseline,
- threat model,
- CI/security baseline.

```text
WRITE_IMPLEMENTATION = NOT_AUTHORIZED
```

---


## Gate 1 — Provider Discovery

Analyze WinUtil capabilities and classify them.

Deliver:

```text
WINUTIL_CAPABILITY_MATRIX
```

No privileged execution is introduced.

---


## Gate 2 — Canonical Operation Model

Define provider-independent operation schemas.

---


## Gate 3 — Read-Only MCP

Initial MCP release should contain only read-oriented capabilities.

Target:

```text
v0.1.0
```

No write implementation should exist in this gate.

---


## Gate 4 — Planning Layer

Introduce:

- plans,
- risk classification,
- plan hashes,
- validation.

Execution remains disabled.

---


## Gate 5 — Authorization Enforcement

Implement and test:

- authorization binding,
- expiry,
- single-use semantics,
- modified-plan rejection,
- rejection invariants.

Write must remain disabled until enforcement tests pass.

---


## Gate 6 — Execution Broker

Introduce controlled execution and privilege separation.

---


## Gate 7 — First Controlled Write

The first write capability should be narrow and low-risk.

Preferred initial class:

```text
APPLICATION_INSTALL
```

using a strict allowlist and a disposable Windows VM.

---


## Gate 8 — Evidence Assurance

Validate complete operation provenance and post-state verification.

---


## Gate 9 — Provider Expansion

Expand WinUtil support only after the control plane is proven.

---


## Gate 10 — Security Acceptance

Required before production-oriented `v1.0.0`.

---


# 14. First Write Policy

Do not begin write testing with:

```text
Defender
Firewall
Services
Boot
MicroWin
Aggressive debloat
Windows security policies
High-impact registry changes
```

Preferred first controlled write:

```text
APPLICATION_INSTALL
```

with a small allowlist of low-risk applications.

The purpose of the first write is not functional breadth.

It is to prove the complete control chain.

---


# 15. Testing Requirements

Testing must include more than happy-path execution.

Required categories:

```text
unit
contract
schema
security
negative
rejection
provider
integration
Windows VM
rollback
supply-chain
evidence integrity
```

Security tests must include:

```text
missing approval
invalid approval
expired approval
reused approval
modified plan
unknown operation
provider substitution
malformed request
provider failure
execution timeout
partial execution
```

The release process must treat failure of authorization or zero-side-effect tests as blocking.

---


# 16. Threat Modeling

Threat modeling is a continuous engineering activity.

Agents should consider at least:

- malicious or compromised agent,
- prompt injection,
- excessive agency,
- approval spoofing,
- confused deputy,
- provider compromise,
- malicious upstream update,
- privilege escalation,
- path injection,
- command injection,
- parameter injection,
- TOCTOU conditions,
- stale authorization,
- replay,
- evidence tampering,
- supply-chain compromise,
- insecure logging,
- credential exposure.

When introducing a new capability, document which trust boundary changes.

---


# 17. Data Protection and Privacy

The project must follow data minimization.

Do not collect system information merely because it is technically accessible.

Evidence and telemetry should capture only what is necessary for:

- authorization,
- verification,
- security,
- auditability,
- troubleshooting.

Avoid recording:

- secrets,
- authentication tokens,
- passwords,
- unnecessary user data,
- sensitive file contents.

Logging must never become a secondary data-exfiltration channel.

Privacy must be treated as an architectural property, not only documentation.

---


# 18. AI Governance Principles

This repository should demonstrate practical AI governance rather than abstract compliance language.

Core principles:

```text
Human Oversight
Accountability
Traceability
Least Privilege
Segregation of Duties
Risk-Based Control
Explainability of Actions
Reversibility Where Possible
Evidence-Based Assurance
No Self-Approval
```

Agents must distinguish:

```text
AI recommendation
AI decision support
AI-generated plan
authorized decision
executed action
verified outcome
```

These are separate governance states.

---


# 19. Compliance Positioning

The project may be designed to support governance and assurance practices relevant to frameworks and regulations such as:

- ISO/IEC 27001,
- ISO/IEC 42001,
- NIST AI RMF,
- NIST Cybersecurity Framework,
- EU AI governance expectations,
- GDPR / data protection principles,
- enterprise change-management controls.

Do not claim that use of this project automatically creates regulatory compliance or certification.

Preferred wording:

> “Designed to support auditable governance and control objectives.”

Avoid:

> “Fully compliant with…”

unless independently demonstrated and specifically scoped.

---


# 20. Career and Thought-Leadership Positioning

Repository documentation should demonstrate that Krzysztof operates at the intersection of technology execution and executive governance.

Technical artifacts should therefore explain not only:

```text
HOW
```

but also:

```text
WHY
RISK
CONTROL
ACCOUNTABILITY
BUSINESS IMPACT
ASSURANCE
```

A strong repository contribution should make visible the ability to:

- define trust boundaries,
- establish policy and decision rights,
- translate risk into architecture,
- design controls above AI execution,
- distinguish autonomy from authority,
- connect cybersecurity and AI governance,
- create auditable operating models,
- protect enterprise accountability while enabling automation.

Where appropriate, architecture documents should contain sections such as:

```text
Business Context
Risk
Decision
Security Control
Governance Control
Human Oversight
Evidence
Residual Risk
```

This supports positioning beyond engineering implementation toward senior architecture, governance, CAIO, CIO, CISO, and transformation responsibilities.

---


# 21. Communication Standard

## Repository Language

All repository content must be written in:

```text
English
```

including:

- source-code documentation,
- README,
- ADRs,
- issues,
- pull requests,
- architecture documentation,
- commit messages where practical,
- governance documents,
- threat models,
- test descriptions,
- release notes.

Internal discussion with the repository owner may occur in Polish.

Do not introduce Polish documentation into the repository unless explicitly requested for a localization or translation artifact.

---


# 22. Documentation Tone

Use professional technical English.

Prefer:

- clear claims,
- explicit assumptions,
- evidence,
- diagrams,
- decision records,
- risk statements,
- short paragraphs,
- precise terminology.

Avoid:

- marketing exaggeration,
- vague claims,
- unnecessary buzzwords,
- unsupported security assertions,
- excessive AI hype.

Preferred style:

> “The execution broker rejects requests whose approval token is not bound to the exact plan hash.”

Avoid:

> “Our revolutionary AI security layer guarantees total safety.”

---


# 23. Evidence and Claim Discipline

Agents must classify important claims where uncertainty exists.

Use when appropriate:

```text
FACT
ASSUMPTION
HYPOTHESIS
DECISION
UNVERIFIED
```

Never convert:

```text
NOT OBSERVED
```

into:

```text
DOES NOT EXIST
```

without sufficient evidence.

Never convert:

```text
UNKNOWN
```

into:

```text
PASS
```

or:

```text
FAIL
```

without proof.

---


# 24. Agent Autonomy

Agents may autonomously perform:

- analysis,
- architecture proposals,
- code review,
- documentation drafting,
- tests that do not alter protected system state,
- static analysis,
- repository inspection,
- threat modeling,
- policy analysis,
- read-only discovery.

Agents must not autonomously assume authorization for:

- privileged Windows writes,
- security-control changes,
- destructive actions,
- release publication,
- production deployment,
- credential changes,
- changes to protected governance rules.

When authorization is required, stop at the explicit governance gate.

---


# 25. Decision Priority

When project goals conflict, use this priority order:

```text
1. Security invariants
2. Governance and accountability
3. Evidence integrity
4. Data protection
5. Architectural correctness
6. Reliability
7. Maintainability
8. User experience
9. Feature breadth
10. Development speed
```

Do not weaken higher-priority controls merely to simplify implementation.

---


# 26. Definition of Done

A feature is not done merely because it executes successfully.

For relevant capabilities, Definition of Done includes:

```text
implementation
tests
negative tests
authorization behavior
provider boundary validation
documentation
risk classification
evidence behavior
failure behavior
security review
```

For write capabilities additionally require:

```text
PRE_STATE
AUTHORIZATION
EXECUTION
POST_STATE
VERIFICATION
EVIDENCE
```

---


# 27. Repository Success Criteria

This repository succeeds if it proves that:

1. AI agents can interact with Windows without receiving unrestricted privileged shell access.
2. Change intent can be separated from execution authority.
3. Policies can constrain what an agent may attempt.
4. Humans or independent control domains can authorize sensitive changes.
5. Authorization can be bound to an exact operation.
6. Rejected operations can be proven to have zero side effects.
7. Providers can be changed without redesigning the MCP contract.
8. Privileged execution can be isolated from the AI-facing control plane.
9. Administrative actions can produce trustworthy evidence.
10. AI-assisted operations can remain accountable to enterprise governance.

The long-term value of the project is not the number of Windows functions exposed.

The value is the ability to answer:

> **Who or what is allowed to cause a privileged system change, under which policy, based on whose authority, with what evidence, and with what residual risk?**

That is the architectural center of Windows Change Control MCP.