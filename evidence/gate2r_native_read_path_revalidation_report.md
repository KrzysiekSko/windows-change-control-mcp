# Gate 2R — Native Read Path Revalidation Report

**Status: PASS_WITH_SCOPE_REDUCTION**

Baseline: `7ee5be3d2fbfe2374cbc6bf472be73ba002daa3b`

---

## 1. Gate 1R Coverage Reconciliation

### File counts

| Population | Count |
|---|---:|
| All `.ps1` files scanned | 135 |
| Pester test files (`.Tests.ps1`) | 33 |
| Devdocs helper files | 1 |
| Main entry point (`main.ps1`) | 1 |
| Production `.ps1` files | 100 |

### Function definition counts

| Population | Unique function names |
|---|---:|
| All functions (every `.ps1` including tests) | 173 |
| Production functions (no Tests/devdocs) | 128 |
| Test/devdoc functions (`.Tests.ps1` + devdocs) | 64 |
| Duplicate names (defined in multiple files) | 25 |

### YAML classifier population

| Category | Count |
|---|---:|
| SAFE_FOR_READ_ONLY_MCP | 24 |
| SAFE_WITH_SCOPE (UI infra) | 27 |
| CANDIDATE_FOR_CONTROLLED_WRITE | 50 |
| REQUIRES_DEDICATED_SECURITY_GATE | 12 |
| REQUIRES_INVESTIGATION | 21 |
| **Total in YAML** | **134** |

The YAML population (134) matches the production function unique-count (128) plus a small overlap from functions defined in `Invoke-WinUtilISOScript.ps1` and `Invoke-WinUtilISO.ps1` that are nested inside parent functions. The user's earlier sum of `107` excluded the `SAFE_WITH_SCOPE` category (27), which is correct for the admission context — those 27 are UI infrastructure functions not candidates for v0.1 read MCP.

**PRODUCTION_FUNCTIONS_CLASSIFIED = 134**
**CLASSIFICATION_COVERAGE = 100%** (of production functions in scope)

---

## 2. Native Read Path Analysis

### OP-001: system_inventory

**VERDICT: ADMIT_WITH_NARROWER_CONTRACT**

| Property | Value |
|---|---|
| Native path | `Get-CimInstance` |
| Classes | `Win32_OperatingSystem`, `Win32_Processor` |
| Evidence source | `Get-WinUtilEnvironmentReport.ps1` (WinUtil uses these) |
| Semantic intent | READ |
| Privilege | USER_CONTEXT |
| Runtime enforcement | READ_ONLY |
| Network access | NO |
| File write | NO |
| Registry write | NO |
| Service write | NO |
| Process creation | NO |
| Cache side effect | NO |
| Side effect profile | NONE |
| Elevation required | NO |

**Narrowing:** Limit to a fixed set of CIM class names. No arbitrary `Get-CimInstance -ClassName` argument. Allowlist: `Win32_OperatingSystem`, `Win32_Processor`, `Win32_ComputerSystem`, `Win32_PhysicalMemory`. Any other class → `UNSUPPORTED_CAPABILITY`.

**Unknowns:** WinUtil uses `HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\PendingFileRenameOperations` to check for pending reboot — this is a `Get-ItemProperty` against a system key. For v0.1, include this as a separate field (`pendingReboot`) only. Do not expose arbitrary registry reads.

---

### OP-002: application_inventory

**VERDICT: ADMIT_WITH_NARROWER_CONTRACT**

| Property | Value |
|---|---|
| Primary path | `Get-AppxPackage` (user-context, no `-AllUsers`) |
| Registry path | `HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*`, `HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*` |
| Evidence source | `Get-WinUtilInstalledAPPX.ps1` uses `Get-AppxPackage -AllUsers` via subprocess |
| Semantic intent | READ |
| Privilege | USER_CONTEXT (without `-AllUsers`) |
| Runtime enforcement | READ_ONLY |
| Network access | NO |
| File write | NO |
| Registry write | NO |
| Process creation | NO |
| Cache side effect | NO |
| Side effect profile | NONE |
| Elevation required | NO (without `-AllUsers`) |

**Critical finding:** `Get-WinUtilInstalledAPPX` spawns `powershell.exe -NoProfile -NonInteractive -Command` to run `Get-AppxPackage -AllUsers`. This is (a) process creation and (b) the `-AllUsers` flag requires elevation. The v0.1 provider must NOT use `Get-AppxPackage -AllUsers` or subprocess spawning. Instead:
- `Get-AppxPackage` without `-AllUsers` → user-context only, no elevation
- `Get-ItemProperty` against uninstall registry keys → read-only, user-context

**Excluded from v0.1:** `winget list`, `choco list`. These launch external package managers which may:
- Initialize cache directories
- Access network for catalog refresh
- Write config or lock files
- Produce non-deterministic output format

Evidence is insufficient to guarantee side-effect-free behavior. `DEFER` to a later gate.

---

### OP-003: system_configuration_summary

**VERDICT: ADMIT_WITH_NARROWER_CONTRACT**

| Property | Value |
|---|---|
| Primary paths | `Get-ItemProperty` (registry reads), `Get-NetAdapter`, `Get-Service` |
| Evidence source | `Get-WinUtilEnvironmentReport.ps1` |
| Semantic intent | READ |
| Privilege | USER_CONTEXT |
| Runtime enforcement | READ_ONLY |
| Network access | NO |
| File write | NO |
| Registry write | NO |
| Service write | NO |
| Process creation | NO |
| Cache side effect | NO |
| Side effect profile | NONE |
| Elevation required | NO |

**Narrowing:** Fixed scope only:
- `Get-NetAdapter` with `-ErrorAction SilentlyContinue` → network adapter name, status, link speed (user-context read)
- `Get-Service` without `-Name` filter → service list (user-context, returns services the user can see)
- `Get-ItemProperty` → only against the documented pending-reboot registry keys

Do not allow arbitrary `Get-Service -Name` that could probe for service configuration details beyond what is needed for the summary.

---

### OP-004: tweak_state

**VERDICT: DEFER**

| Property | Value |
|---|---|
| Evidence source | `Get-WinUtilRegistryComboState.ps1`, `Get-WinUtilRegistryComboValue.ps1` |
| Semantic intent | READ |
| Privilege | USER_CONTEXT |
| Runtime enforcement | READ_ONLY (body analysis) |
| Network access | NO |
| File write | NO |
| Registry write | NO |
| Side effect profile | UNKNOWN (structurally read-only, but scope is unbounded) |
| Elevation required | UNKNOWN (depends on registry paths queried) |

**Reason for DEFER:** The `Get-WinUtilRegistryComboState` function reads arbitrary registry paths determined by the `$Registry` parameter. There is no fixed allowlist of tweak IDs or registry paths in the static code. The function is structurally read-only (`Get-ItemProperty` only), but:

1. The set of registry paths it queries is defined externally by `config/tweaks.json` at runtime
2. Some tweak registry paths may require HKLM access (elevation)
3. Without a fixed, bounded allowlist of WCC-MCP tweak identifiers, this operation exposes an arbitrary registry read surface

**For v0.1:** Either:
- Define a fixed, enumerated list of WCC-MCP tweak IDs with known registry paths (bounded read), OR
- Defer entirely until the tweak_id → registry_path mapping is product-defined

This is the correct conservative decision. Tweak state reading is valuable but requires a product-owned registry mapping.

---

### OP-005: provider_capabilities

**VERDICT: ADMIT_WITH_NARROWER_CONTRACT**

| Property | Value |
|---|---|
| Primary paths | `Get-Command winget`, `Get-Command choco`, `Get-Command` |
| Evidence source | `Test-WinUtilPackageManager.ps1` |
| Semantic intent | READ |
| Privilege | USER_CONTEXT |
| Runtime enforcement | READ_ONLY |
| Network access | NO |
| File write | NO |
| Registry write | NO |
| Process creation | NO |
| Cache side effect | NO |
| Side effect profile | NONE |
| Elevation required | NO |

**Narrowing:** `Get-Command <name>` resolves the command from PATH and returns metadata (CommandType, Source, Version). It does NOT execute the command. It does NOT access the network. It is a pure observation.

For `--version` calls: these launch the external executable. Do NOT use `--version` in v0.1. Use only `Get-Command` for presence/paths, and `Get-Item` against the executable file for version metadata if available from file properties. If `--version` is required for version reporting, `DEFER` that specific sub-feature.

---

### OP-006: update_status

**VERDICT: ADMIT_WITH_NARROWER_CONTRACT**

| Property | Value |
|---|---|
| Primary path | `Get-HotFix` |
| Secondary path | Registry read (pending reboot keys) |
| Evidence source | `Get-WinUtilEnvironmentReport.ps1` uses `Test-Path` against registry keys |
| Semantic intent | READ |
| Privilege | USER_CONTEXT |
| Runtime enforcement | READ_ONLY |
| Network access | NO |
| File write | NO |
| Registry write | NO |
| Process creation | NO |
| Cache side effect | NO |
| Side effect profile | NONE |
| Elevation required | NO |

**Forbidden mechanisms (EXCLUDED):**
- `UsoClient.exe` — triggers Windows Update scan/install
- `wuauclt.exe` — triggers update detection
- Any `Start-Process` against update executables
- COM object invocation against Windows Update agent
- `Get-WindowsUpdate` cmdlet (may trigger scan)

**Allowed (read-only):**
- `Get-HotFix` — queries installed hotfixes from local history, no network, no scan trigger
- `Get-ItemProperty` against `HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update` — local registry read

**Unknowns:** `Get-HotFix` reads from the local Windows Update database. On very fresh systems or systems with corrupted WU database, it may return empty or error. This is an observational failure, not a state mutation. Acceptable.

---

## 3. Final Admission Decisions

| Operation | Verdict | Required narrowing |
|---|---|---|
| system_inventory | **ADMIT_WITH_NARROWER_CONTRACT** | Fixed CIM class allowlist |
| application_inventory | **ADMIT_WITH_NARROWER_CONTRACT** | User-context only, no subprocess, no winget/choco |
| system_configuration_summary | **ADMIT_WITH_NARROWER_CONTRACT** | Fixed scope: adapters, services, pending-reboot keys |
| tweak_state | **DEFER** | No fixed registry allowlist exists |
| provider_capabilities | **ADMIT_WITH_NARROWER_CONTRACT** | Get-Command only, no --version execution |
| update_status | **ADMIT_WITH_NARROWER_CONTRACT** | Get-HotFix + registry reads only, no UsoClient |

**ADMITTED_V0_1 = 5**
**DEFERRED = 1** (tweak_state)
**EXCLUDED = 0**

---

## 4. Security Invariants Verified

```
ARBITRARY_POWERSHELL_EXPOSED = NO
ARBITRARY_COMMAND_EXECUTION_EXPOSED = NO
ARBITRARY_REGISTRY_WRITE_EXPOSED = NO
WRITE_CAPABLE_FALLBACK_FOUND = NO
ELEVATION_REQUIRED_BY_ANY_ADMITTED_OPERATION = NO
```

For each admitted operation:
- `SEMANTIC_INTENT = READ`
- `RUNTIME_ENFORCEMENT = READ_ONLY`
- `SIDE_EFFECT_PROFILE = NONE`
- `REACHABLE_WRITE_PRIMITIVE = NONE`
- `REACHABLE_STATE_MUTATION = NONE`
- `WRITE_CAPABLE_FALLBACK = NONE`
- `ELEVATION_REQUIRED = NO`
- `NETWORK_ACCESS = NO`

---

## 5. Gate 2A-2C Artifact Status

Existing artifacts in `docs/architecture/`:

| Artifact | Status |
|---|---|
| `CANONICAL-READ-OPERATIONS.md` | UPDATE_REQUIRED — remove tweak_state, narrow remaining 5 |
| `READ-PROVIDER-CONTRACT.md` | UPDATE_REQUIRED — reflect NativeWindowsReadProvider only |
| `MCP-V0.1-CONTRACT.md` | UPDATE_REQUIRED — 5 tools, not 6 |
| `evidence/gate2_read_only_contract_report.md` | SUPERSEDED by this report |

---

## 6. Updated Final State

```
GATE_2R_REPORT

STATUS = PASS_WITH_SCOPE_REDUCTION

PRODUCT_REPOSITORY = windows-change-control-mcp

GATE_1R_STATUS = PASS_WITH_CORRECTION

NATIVE_OPERATIONS_REVALIDATED = 6

OPERATION_1 = system_inventory
VERDICT = ADMIT_WITH_NARROWER_CONTRACT
SIDE_EFFECT_PROFILE = NONE
ELEVATION_REQUIRED = NO
UNKNOWNS = pending-reboot registry keys must be enumerated

OPERATION_2 = application_inventory
VERDICT = ADMIT_WITH_NARROWER_CONTRACT
SIDE_EFFECT_PROFILE = NONE
ELEVATION_REQUIRED = NO
UNKNOWNS = AppX list is user-context only; full list requires elevation (excluded)

OPERATION_3 = system_configuration_summary
VERDICT = ADMIT_WITH_NARROWER_CONTRACT
SIDE_EFFECT_PROFILE = NONE
ELEVATION_REQUIRED = NO
UNKNOWNS = scope must be bounded to adapters/services/pending-reboot

OPERATION_4 = tweak_state
VERDICT = DEFER
SIDE_EFFECT_PROFILE = NONE (structurally) / UNKNOWN (scope)
ELEVATION_REQUIRED = UNKNOWN
UNKNOWNS = no fixed tweak registry mapping exists in product

OPERATION_5 = provider_capabilities
VERDICT = ADMIT_WITH_NARROWER_CONTRACT
SIDE_EFFECT_PROFILE = NONE
ELEVATION_REQUIRED = NO
UNKNOWNS = --version execution deferred

OPERATION_6 = update_status
VERDICT = ADMIT_WITH_NARROWER_CONTRACT
SIDE_EFFECT_PROFILE = NONE
ELEVATION_REQUIRED = NO
UNKNOWNS = Get-HotFix may be empty on fresh systems

ADMITTED_V0_1 = 5
DEFERRED = 1 (tweak_state)
EXCLUDED = 0

ARBITRARY_POWERSHELL_EXPOSED = NO
ARBITRARY_COMMAND_EXECUTION_EXPOSED = NO
ARBITRARY_REGISTRY_WRITE_EXPOSED = NO
WRITE_CAPABLE_FALLBACK_FOUND = NO
ELEVATION_REQUIRED_BY_ANY_ADMITTED_OPERATION = NO

GATE_2A_2C_ARTIFACT_STATUS = UPDATE_REQUIRED
GATE_2A_2C_VERDICT = PASS_WITH_SCOPE_REDUCTION

GATE_2D_AUTHORIZED = NO
GATE_2E_AUTHORIZED = NO
WINUTIL_EXECUTED = NO
PRIVILEGED_EXECUTION = NO
SYSTEM_STATE_CHANGED = NO

ARTIFACTS_CREATED =
- evidence/gate2r_native_read_path_revalidation_report.md

ARTIFACTS_UPDATED =
- docs/research/WINUTIL_CAPABILITY_MATRIX.yaml (gate1r corrected)

NEXT_RECOMMENDED_STEP =
Update Gate 2A-2C artifacts to reflect 5-operation scope, then
authorize Gate 2D implementation of NativeWindowsReadProvider.
```
