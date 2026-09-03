# WCC-MCP Gate 2A–2C Report

**STATUS: PASS**

Baseline: `7ee5be3d2fbfe2374cbc6bf472be73ba002daa3b`

---

## Authorization Scope

| Gate | Status |
|---|---|
| Gate 2A — Canonical Read Operations | PASS |
| Gate 2B — Provider Boundary | PASS |
| Gate 2C — MCP Schemas / Tool Contracts | PASS |
| Gate 2D — Read-Only Implementation | NOT_AUTHORIZED |
| Gate 2E — Negative Security Tests | NOT_AUTHORIZED |

---

## Canonical Operations — Decision Summary

| ID | Operation | Disposition | Rationale |
|---|---|---|---|
| OP-001 | system_inventory | INCLUDE_V0_1 | WMI/CIM read, no write, no network, no elevation |
| OP-002 | application_inventory | INCLUDE_V0_1 | Registry + package manager list queries, user-scope AppX, caveat: winget/choco may touch network cache |
| OP-003 | tweak_state | INCLUDE_V0_1 | Registry read only; caveat: provider must enforce read-only parameter binding |
| OP-004 | provider_capabilities | INCLUDE_V0_1 | Version queries via `--version` flags, read-only |
| OP-005 | update_status | INCLUDE_V0_1 | WMI/log queries only; caveat: provider must enforce no UsoClient activation |
| OP-006 | system_configuration_summary | INCLUDE_V0_1 | WMI/CIM reads across network/firewall/storage; caveat: Get-Service scoped to user-context |

**Total: 6 INCLUDE_V0_1, 0 DEFERRED, 0 EXCLUDED**

All 6 operations pass the Read-Only Admission Test (14 properties verified per operation).

---

## Two-Axis Classification — Verified

All 6 operations satisfy:

```
SEMANTIC_INTENT = READ
RUNTIME_ENFORCEMENT = READ_ONLY
SIDE_EFFECT_EXPECTATION = NONE
ELEVATION = NOT_REQUIRED
```

Four caveats documented at provider level (not MCP contract level).

---

## Critical Classifier Finding — Corrected

The Gate 1 classifier produced 36 false positives: write-intent functions
(e.g. `Add-AppxPackage`, `Remove-AppxPackage`, `Set-DnsClientDohServerAddress`,
`Invoke-WPFAppxInstall`) were classified as `SAFE_FOR_READ_ONLY_MCP` because the
classifier assigned `privilege=USER_CONTEXT` while ignoring `operation_intent=WRITE`.

**Correction applied:** `SAFE_FOR_READ_ONLY_MCP` requires both `intent=READ` AND
`privilege=USER_CONTEXT`. Write-intent functions in user context are still write
functions and are excluded from v0.1.

The corrected count is 133 `READ+USER_CONTEXT` functions; all further v0.1 analysis
was performed against the corrected two-axis model.

---

## Provider Boundary — Defined

```
ReadProvider interface
  getSystemInventory()         → OP-001
  getApplicationInventory()   → OP-002
  getTweakState()             → OP-003
  getProviderCapabilities()     → OP-004
  getUpdateStatus()            → OP-005
  getSystemConfigurationSummary() → OP-006
```

**WinUtil names NOT exposed through public API:**
`Get-WinUtil*`, `Invoke-WPF*`, `Find-*`, `Set-*`, `Remove-*`, `Add-*` — all internal.

**Provider independence:** Canonical operation IDs and normalized schemas are the
public contract. Providers can be swapped without changing the MCP API.

---

## MCP Tool Contracts — Defined

Six tools defined with JSON Schema for inputs, outputs, and errors.
All inputs schema-bounded. No arbitrary strings where enums can constrain requests.

---

## Security Invariants — Contract Level

```
WRITE_TOOLS_EXPOSED = 0
PLAN_TOOLS_EXPOSED = 0
ARBITRARY_POWERSHELL = 0
RAW_COMMAND_EXECUTION = 0
RAW_REGISTRY_WRITE = 0
RAW_SERVICE_CONTROL = 0
DOWNLOAD_AND_EXECUTE = 0
DYNAMIC_SCRIPT_EXECUTION = 0
PROVIDER_WRITE_PATH_REACHABLE = NO
ELEVATION_REQUIRED_BY_MCP = NO
OPERATION_INTENT = READ
RUNTIME_ENFORCEMENT = READ_ONLY
SYSTEM_STATE_CHANGE = IMPOSSIBLE_BY_CONTRACT
```

> **Contract-level claim.** Runtime proof requires Gate 2D/2E (not yet authorized).

---

## Evidence Model

Every invocation produces evidence metadata:

```
request_id, operation_id, timestamp, intent, enforcement, privilege_context,
provider, provider_version, result, failure_code, side_effect_observed,
evidence_hash (SHA-256)
```

SHA-256 provides integrity, not non-repudiation.

---

## Write Implementation Status

```
WRITE_IMPLEMENTATION = NO
EXECUTION_BROKER_IMPLEMENTED = NO
PRIVILEGED_EXECUTION = NO
WINUTIL_EXECUTED = NO
SYSTEM_STATE_CHANGED = NO
```

---

## Artifacts Created

```
docs/architecture/CANONICAL-READ-OPERATIONS.md   — Gate 2A
docs/architecture/READ-PROVIDER-CONTRACT.md         — Gate 2B
docs/architecture/MCP-V0.1-CONTRACT.md             — Gate 2C
```

---

## Findings

1. **Classifier flaw corrected:** Gate 1 produced 36 false positives (write-as-read). Fixed before v0.1 admission.
2. **168 RO functions reduced to 6 canonical operations** — upstream surface ≠ product surface.
3. **Four provider-level caveats** require implementation-level enforcement (not contract-level):
   - OP-002: winget/choco network cache touch
   - OP-003: parameter binding enforcement
   - OP-005: no UsoClient activation
   - OP-006: user-context service query
4. **`Invoke-Expression` = 0** in WinUtil at this baseline (confirmed).
5. **10 CANDIDATE_FOR_CONTROLLED_WRITE** and **38 REQUIRES_DEDICATED_SECURITY_GATE** capabilities are outside v0.1 scope and remain governed by separate gates.

---

## Deferred Security Questions

1. OP-003 provider parameter binding: requires implementation-level enforcement review in Gate 2D.
2. OP-005 UsoClient activation: requires behavioral test in Gate 2E before write expansion.
3. OP-006 user-context service scope: requires runtime verification that admin elevation is not silently assumed.
4. All four provider caveats must be verified as non-bypassable in Gate 2D/2E before production use.

---

## Blockers

None for Gates 2A–2C.

---

## Next Recommended Step

**Gate 2D — Read-Only Implementation Authorization.**

Before authorizing Gate 2D, review and approve:
1. The four provider-level caveats as enforceable design constraints (not just documentation).
2. The classifier correction applied in this gate (must be reflected in the capability matrix).
3. The decision to include OP-005 `update_status` with the UsoClient constraint — if the constraint is deemed unenforceable at runtime, OP-005 should be DEFERRED.

Upon authorization: implement the six MCP tools, the ReadProvider interface, and the WinUtilReadProvider skeleton — but NOT the execution broker, NOT the write path, NOT any privileged execution.

Gate 2E (negative security tests proving zero reachable write path) remains separately authorized after Gate 2D implementation.
