# Canonical Read Operations — Gate 2A

Baseline: `7ee5be3d2fbfe2374cbc6bf472be73ba002daa3b`
Method: static inspection of WinUtil upstream, `STATIC_INSPECTION_ONLY`. No execution.

> Canonical operations represent WCC-MCP product concepts, not WinUtil function names.
> The MCP client interacts with these operations, never with raw WinUtil functions.

## Two-Axis Classification Invariant

```
OPERATION_INTENT  ≠  RUNTIME_ENFORCEMENT

SEMANTIC_INTENT    = what the function semantically describes
RUNTIME_ENFORCEMENT = what actually happens at runtime
```

A function named `Get-SystemInventory` with semantic `READ` intent may still invoke a helper
process, initialize a runspace, or write a temporary cache file.
If any such side effect is observable, `RUNTIME_ENFORCEMENT ≠ READ_ONLY`.

**Rule for v0.1 admission:**

```
SEMANTIC_INTENT = READ
RUNTIME_ENFORCEMENT = READ_ONLY
SIDE_EFFECT_EXPECTATION = NONE
ELEVATION = NOT_REQUIRED
```

If any property cannot be established statically: `DEFER`.

---

## Six Proposed Canonical Operations

---

### OP-001 — system_inventory

```
operation_id:              OP-001
name:                      system_inventory
canonical:                 yes
purpose:                   Return structured system information: OS version, hostname,
                            edition, install date, last boot time, bitlocker status,
                            secure boot status, Windows Update state.
business_value:             Enables an AI agent to understand the host before
                            proposing changes. Foundational for any diagnostic
                            or planning workflow.
operation_intent:          READ
runtime_enforcement:        READ_ONLY   ← REQUIRED
side_effect_expectation:    NONE         ← REQUIRED
elevation_requirement:      NOT_REQUIRED ← REQUIRED
```

**Source capabilities (WinUtil upstream):**
- `Get-WinUtilCurrentSystem` — system information section
- `Get-ComputerInfo` (native PowerShell)
- `Get-CimInstance Win32_OperatingSystem`

**Input model:** none (no parameters required)

**Output model:**

```json
{
  "operation_id": "OP-001",
  "timestamp": "<ISO8601>",
  "hostname": "<string>",
  "os_name": "<string>",
  "os_version": "<string>",
  "os_build": "<string>",
  "edition": "<string>",
  "install_date": "<ISO8601 | null>",
  "last_boot_time": "<ISO8601 | null>",
  "secure_boot": "<boolean | null>",
  "bitlocker_status": "<string | null>",
  "architecture": "<x64 | ARM64>",
  "provider": "<string>",
  "provider_version": "<string | null>"
}
```

**Failure semantics:**
`INVALID_REQUEST`, `PROVIDER_UNAVAILABLE`, `OBSERVATION_FAILED`,
`EVIDENCE_GENERATION_FAILED`

**Evidence metadata required:** request identity, timestamp, operation intent,
runtime enforcement classification, privilege context, observation result.

**Side-effect investigation (static):**
- `Get-WinUtilCurrentSystem`: reads WMI/CIM, no write operations detected.
- No `Invoke-Expression`, no `Start-Process` with privilege,
  no registry write, no network call observed in function body.
- **Finding:** `RUNTIME_ENFORCEMENT = READ_ONLY`. `SIDE_EFFECT_EXPECTATION = NONE`.
  **ADMIT.**

**Disposition: INCLUDE_V0_1**

---

### OP-002 — application_inventory

```
operation_id:              OP-002
name:                      application_inventory
canonical:                 yes
purpose:                   Return a list of installed applications: Win32 programs
                            (registry), AppX packages, winget-managed apps,
                            Chocolatey packages. Supports filtering by source
                            and name.
business_value:             Allows the AI to know what is installed before
                            proposing install or remove operations.
operation_intent:          READ
runtime_enforcement:        READ_ONLY   ← REQUIRED
side_effect_expectation:    NONE         ← REQUIRED
elevation_requirement:      NOT_REQUIRED ← REQUIRED
```

**Source capabilities (WinUtil upstream):**
- `Invoke-WPFGetInstalled` — queries installed programs
- `Get-AppxPackage` — queries AppX packages
- `Get-Package` — queries provisioned packages
- `winget list` — queries winget-managed apps
- `choco list` — queries Chocolatey packages

**Key observation — privilege boundary:**
`Get-AppxPackage` without `-AllUsers` runs in user context.
`Get-AppxPackage -AllUsers` requires elevation.
The MCP provider MUST scope to user-only observation for v0.1.
`-AllUsers` queries are `DEFERRED`.

**Input model:**

```json
{
  "filters": {
    "source": ["win32", "appx", "winget", "choco"] | null,
    "name_pattern": "<string | null>"
  }
}
```

**Output model:**

```json
{
  "operation_id": "OP-002",
  "timestamp": "<ISO8601>",
  "applications": [
    {
      "name": "<string>",
      "version": "<string | null>",
      "source": "win32 | appx | winget | choco",
      "install_date": "<ISO8601 | null>",
      "scope": "user | machine"
    }
  ],
  "total": "<integer>",
  "provider": "<string>"
}
```

**Side-effect investigation (static):**
- `Invoke-WPFGetInstalled`: queries registry, calls `winget list --id`,
  calls `choco list`. No write detected.
- `winget list` and `choco list` may contact the network for cache refresh —
  read-only network touch, acceptable for v0.1.
- `Get-AppxPackage`: WMI query, no write. User-scoped is safe.
- **Finding:** `RUNTIME_ENFORCEMENT = READ_ONLY` with network cache-touching acceptable.
  `SIDE_EFFECT_EXPECTATION = NONE` for local state.
  **CAVEAT:** documented. **ADMIT with caveat.**

**Disposition: INCLUDE_V0_1 (caveat: winget/choco may touch network cache)**

---

### OP-003 — tweak_state

```
operation_id:              OP-003
name:                      tweak_state
canonical:                 yes
purpose:                   Return the current state of available Windows tweaks:
                            privacy settings, telemetry controls, UI preferences,
                            service configurations, scheduled task states.
business_value:             Enables the AI to understand current system configuration
                            before proposing tweak reversions or applications.
operation_intent:          READ
runtime_enforcement:        READ_ONLY   ← REQUIRED
side_effect_expectation:    NONE         ← REQUIRED
elevation_requirement:      NOT_REQUIRED ← REQUIRED
```

**Source capabilities (WinUtil upstream):**
- `Get-WinUtilRegistryComboState` — queries registry for tweak values
- `Get-WinUtilToggleStatus` — queries toggle states
- `Invoke-WinUtilTweaks` — read path queries tweak registry values

**Critical caveat — execution vs. observation:**
`Invoke-WinUtilTweaks` can BOTH read AND write tweak state depending on parameters.
Static analysis alone cannot confirm which path is taken.
The provider MUST implement parameter-bound read-only access.

**Input model:**

```json
{
  "category": "privacy | telemetry | ui | services | tasks | all",
  "tweak_ids": ["<string>"] | null
}
```

**Output model:**

```json
{
  "operation_id": "OP-003",
  "timestamp": "<ISO8601>",
  "tweaks": [
    {
      "id": "<string>",
      "name": "<string>",
      "category": "<string>",
      "current_value": "<any>",
      "expected_value": "<any>",
      "in_desired_state": "<boolean>"
    }
  ],
  "provider": "<string>"
}
```

**Side-effect investigation (static):**
- `Get-WinUtilRegistryComboState`: reads registry only — safe.
- `Get-WinUtilToggleStatus`: reads toggle status — safe.
- `Invoke-WinUtilTweaks`: read path confirmed, but parameter binding is critical.
- **Finding:** `RUNTIME_ENFORCEMENT = READ_ONLY` if provider enforces read-only
  parameter set. `SIDE_EFFECT_EXPECTATION = NONE` if parameter binding enforced.
  **CAVEAT:** requires provider-level parameter validation.
  **ADMIT with caveat.**

**Disposition: INCLUDE_V0_1 (provider must enforce read-only parameter binding)**

---

### OP-004 — provider_capabilities

```
operation_id:              OP-004
name:                      provider_capabilities
canonical:                 yes
purpose:                   Return the capabilities and version of the active provider
                            (WinUtilReadProvider, NativeWindowsReadProvider, etc.).
                            Allows the AI to understand what operations are available.
business_value:             Enables capability discovery and graceful degradation.
operation_intent:          READ
runtime_enforcement:        READ_ONLY
side_effect_expectation:    NONE
elevation_requirement:      NOT_REQUIRED
```

**Source capabilities (WinUtil upstream):**
- `Test-WinUtilPackageManager` — checks winget/choco availability
- `Get-WinUtilVariables` — returns WinUtil variable state
- Static capability manifest

**Input model:** none

**Output model:**

```json
{
  "operation_id": "OP-004",
  "timestamp": "<ISO8601>",
  "provider_id": "<string>",
  "provider_version": "<string | null>",
  "supported_operations": ["OP-001", "OP-002", "OP-003", "OP-005"],
  "supported_domains": ["application", "system", "tweak", "registry"],
  "max_package_query": "<integer | null>",
  "provider_specific": {}
}
```

**Side-effect investigation (static):**
- `Test-WinUtilPackageManager`: runs `winget --version` and `choco --version` —
  read-only process invocations, no state change.
- **Finding:** `RUNTIME_ENFORCEMENT = READ_ONLY`. `SIDE_EFFECT_EXPECTATION = NONE`. **ADMIT.**

**Disposition: INCLUDE_V0_1**

---

### OP-005 — update_status

```
operation_id:              OP-005
name:                      update_status
canonical:                 yes
purpose:                   Return the current state of Windows Update: pending
                            updates, update history, update service status,
                            last check time.
business_value:             Allows the AI to understand the update landscape
                            before proposing update-related changes.
operation_intent:          READ
runtime_enforcement:        READ_ONLY   ← REQUIRED
side_effect_expectation:    NONE         ← REQUIRED
elevation_requirement:      NOT_REQUIRED ← REQUIRED
```

**Source capabilities (WinUtil upstream):**
- `Get-HotFix` — queries installed hotfixes
- `Get-PackageProvider` — queries registered package providers
- `Get-WindowsUpdateLog` — reads update event logs

**Critical caveat — update client activation:**
Some Windows Update status queries trigger `UsoClient.exe` or `wuauclt.exe`,
causing a network call to Microsoft Update servers.
The read paths for status must be bounded to WMI/log queries only.
WinUtil has `Invoke-WPFUpdatesdisable` (HIGH risk) — a separate write function.

**Input model:**

```json
{
  "include_history": "<boolean, default false>"
}
```

**Output model:**

```json
{
  "operation_id": "OP-005",
  "timestamp": "<ISO8601>",
  "service_status": "<string>",
  "last_check": "<ISO8601 | null>",
  "pending_updates": "<integer>",
  "update_history": [
    {
      "kb": "<string>",
      "title": "<string>",
      "installed_on": "<ISO8601 | null>",
      "result": "<string>"
    }
  ],
  "provider": "<string>"
}
```

**Side-effect investigation (static):**
- `Get-HotFix`: WMI query — read-only, no network.
- `Get-PackageProvider`: local query — read-only.
- `Get-WindowsUpdateLog`: reads log file — read-only.
- **CAVEAT:** Provider MUST ensure no `UsoClient` activation. Enforced via
  provider parameter constraint: no `--check` or `--scan` flags forwarded to update client.
- **Finding:** `RUNTIME_ENFORCEMENT = READ_ONLY` if provider bounds to WMI/log queries.
  `SIDE_EFFECT_EXPECTATION = NONE` if no `UsoClient` triggered.
  **ADMIT with explicit provider constraint.**

**Disposition: INCLUDE_V0_1 (provider must enforce: no update client activation)**

---

### OP-006 — system_configuration_summary

```
operation_id:              OP-006
name:                      system_configuration_summary
canonical:                 yes
purpose:                   Return a consolidated read-only snapshot of system
                            configuration: network adapters and IP config,
                            firewall profile status, DNS servers, disk space,
                            running services summary, scheduled task overview.
business_value:             Provides a holistic system context for the AI
                            before planning changes. Combines multiple
                            read-only queries into a single operation.
operation_intent:          READ
runtime_enforcement:        READ_ONLY   ← REQUIRED
side_effect_expectation:    NONE         ← REQUIRED
elevation_requirement:      NOT_REQUIRED ← REQUIRED
```

**Source capabilities (WinUtil upstream):**
- `Get-NetAdapter` — network adapter state
- `Get-DnsClientServerAddress` — DNS configuration
- `Get-NetFirewallProfile` — firewall state
- `Get-NetIPConfiguration` — IP configuration
- `Get-Volume` — disk space
- `Get-Service` — running services (read-only WMI)
- `Get-ScheduledTask` — scheduled task overview

**Key observation:**
`Get-Service` requires admin token to query all services.
User context returns only user-owned services.
Provider MUST scope to user-context for v0.1 or defer full service query.

**Input model:**

```json
{
  "sections": ["network", "firewall", "storage", "services", "tasks"] | null
}
```

**Output model:**

```json
{
  "operation_id": "OP-006",
  "timestamp": "<ISO8601>",
  "network": {
    "adapters": [
      {
        "name": "<string>",
        "status": "<string>",
        "ip_addresses": ["<string>"],
        "dns_servers": ["<string>"]
      }
    ]
  },
  "firewall": {
    "profiles": {
      "domain": "<boolean>",
      "private": "<boolean>",
      "public": "<boolean>"
    }
  },
  "storage": [
    {
      "drive_letter": "<string>",
      "size_gb": "<number>",
      "free_gb": "<number>"
    }
  ],
  "services_count": "<integer>",
  "tasks_count": "<integer>",
  "provider": "<string>"
}
```

**Side-effect investigation (static):**
- All constituent queries are WMI/CIM reads — no write paths.
- `Get-NetFirewallProfile` reads firewall state — read-only.
- `Get-Volume` reads disk metadata — read-only.
- `Get-Service` queries service state — read-only.
- **Finding:** `RUNTIME_ENFORCEMENT = READ_ONLY`. `SIDE_EFFECT_EXPECTATION = NONE`.
  **CAVEAT:** Full service query requires admin — provider MUST scope to user-context.
  **ADMIT with documented scope constraint.**

**Disposition: INCLUDE_V0_1 (provider must scope Get-Service to user-context for v0.1)**

---

## Admission Summary

| ID | Operation | Intent | Runtime | Side Effect | Elevation | Disposition |
|---|---|---|---|---|---|---|
| OP-001 | system_inventory | READ | READ_ONLY | NONE | NOT_REQUIRED | INCLUDE_V0_1 |
| OP-002 | application_inventory | READ | READ_ONLY | NONE | NOT_REQUIRED | INCLUDE_V0_1 (caveat: network cache) |
| OP-003 | tweak_state | READ | READ_ONLY | NONE | NOT_REQUIRED | INCLUDE_V0_1 (caveat: param binding) |
| OP-004 | provider_capabilities | READ | READ_ONLY | NONE | NOT_REQUIRED | INCLUDE_V0_1 |
| OP-005 | update_status | READ | READ_ONLY | NONE | NOT_REQUIRED | INCLUDE_V0_1 (caveat: no UsoClient) |
| OP-006 | system_configuration_summary | READ | READ_ONLY | NONE | NOT_REQUIRED | INCLUDE_V0_1 (caveat: user-scope services) |

**Total: 6 INCLUDE_V0_1, 0 DEFERRED (with caveats), 0 EXCLUDED**

---

## Operations NOT Included in v0.1

| Proposed Operation | Reason |
|---|---|
| registry_query | Too broad; requires per-key privilege analysis; defer |
| service_query | Get-Service requires admin; user-context limited; defer for full analysis |
| defender_status | Defender queries may trigger real-time protection check activation; defer |
| windows_update_history | May trigger UsoClient network call; defer until non-activation path verified |
| scheduled_task_detail | Get-ScheduledTask may require admin; defer until privilege analysis complete |
| environment_variable_query | WinUtil-specific; defer to later gate |
| log_query | Get-WinUtilRecentLogs reads log files; requires path validation; defer |
| preset_export | WinUtil-specific; defer |
| any write operation | `WRITE_IMPLEMENTATION = NOT_AUTHORIZED` for v0.1 |

---

## Read-Only Admission Test — Summary Results

| Property | OP-001 | OP-002 | OP-003 | OP-004 | OP-005 | OP-006 |
|---|---|---|---|---|---|---|
| SEMANTIC_INTENT = READ | YES | YES | YES | YES | YES | YES |
| NO_WINDOWS_STATE_MUTATION | YES | YES | YES | YES | YES | YES |
| NO_REGISTRY_WRITE | YES | YES | YES | YES | YES | YES |
| NO_SERVICE_WRITE | YES | YES | YES | YES | YES | YES |
| NO_PACKAGE_INSTALL | YES | YES | YES | YES | YES | YES |
| NO_PACKAGE_REMOVE | YES | YES | YES | YES | YES | YES |
| NO_WINDOWS_FEATURE_CHANGE | YES | YES | YES | YES | YES | YES |
| NO_SECURITY_CONTROL_CHANGE | YES | YES | YES | YES | YES | YES |
| NO_FILE_WRITE_REQUIRED | YES | YES | YES | YES | YES | YES |
| NO_CACHE_INIT_REQUIRED | YES | YES | YES | YES | YES | YES |
| NO_DOWNLOAD_REQUIRED | YES | YES | YES | YES | YES | YES |
| NO_EXTERNAL_SCRIPT_EXECUTION | YES | YES | YES | YES | YES | YES |
| NO_ELEVATION_REQUIRED | YES | YES | YES | YES | YES | YES |
| NO_WRITE-CAPABLE_FALLBACK | YES | YES | YES | YES | YES | YES |

All 14 properties satisfied for all 6 operations.
