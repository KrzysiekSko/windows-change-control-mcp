# Product Evidence v0.1 - WCC-MCP Demonstrator

This demonstrator shows the flow of a request through the WCC-MCP without performing any privileged execution.

## Flow
1. **REQUEST** - AI agent submits a request (e.g., install 7zip via winget)
2. **CAPABILITY LOOKUP** - System maps request to a canonical operation
3. **CURRENT-STATE OBSERVATION** - Read-only check of current system state (mocked)
4. **RISK CLASSIFICATION** - Determine risk level and factors
5. **POLICY DECISION** - Engine evaluates request against policy (allow/deny)
6. **PLAN** - If allowed, generate a plan of steps that would be executed
7. **EXECUTION STATUS** - In v0.1, no execution is performed (all false)
8. **EVIDENCE BUNDLE** - All artifacts collected and hashed for tamper evidence

## Artifacts in C:\Users\Specjalista\Desktop\Hermes\hermes-agent-commons\evidence\product_evidence_v0_1
- request.json: The original request from the AI agent
- capability.json: Canonical operation details
- current_state.json: Mocked read-only observation
- risk_classification.json: Risk assessment
- policy_decision.json: Policy engine output (ALLOW/DENY)
- plan.json: Steps that would be executed if authorized
- execution_status.json: Shows no execution occurred in v0.1
- manifest.json: SHA-256 hashes of all artifacts for integrity verification

## How to Verify
1. Each artifact's hash is recorded in manifest.json
2. To verify integrity, recompute SHA-256 of each artifact and compare to manifest
3. The manifest itself is hashed (manifest_hash field) to prevent tampering

## Next Steps
- Product Evidence v0.2: Introduce a mock policy engine that can deny requests (e.g., attempt to install non-allowlisted app)
- Product Evidence v0.3: Add support for multiple request types (SYSTEM_INVENTORY, APPLICATION_INVENTORY)
- Gate 2: Define canonical operation model and begin building read-only MCP
- Later: Introduce controlled execution broker with strict authorization

## Notes
- No privileged Windows commands were executed in the creation of this demonstrator.
- All observations are mocked or based on public information.
- This is a demonstration of governance and control plane, not an execution platform.
