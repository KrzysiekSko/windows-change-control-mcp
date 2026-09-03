# Read Provider Contract — Gate 2B

Baseline: `7ee5be3d2fbfe2374cbc6bf472be73ba002daa3b`

> The WCC-MCP core must not depend directly on WinUtil function names.
> This contract defines the abstract read provider interface that any
> provider implementation (WinUtil, Native Windows, etc.) must satisfy.

## Provider Interface Definition

```text
ReadProvider
  getSystemInventory(input)        → SystemInventoryOutput
  getApplicationInventory(input)   → ApplicationInventoryOutput
  getTweakState(input)             → TweakStateOutput
  getProviderCapabilities(input)   → ProviderCapabilitiesOutput
  getUpdateStatus(input)           → UpdateStatusOutput
  getSystemConfigurationSummary(input) → SystemConfigurationSummaryOutput
```

All methods return typed results or normalized failures.

---

## Method Contracts

### getSystemInventory

```text
canonical_operation: OP-001
provider_method:     getSystemInventory
input:               {}  (no parameters)
normalized_output:
  operation_id: "OP-001"
  timestamp: "<ISO8601>"
  hostname: "<string>"
  os_name: "<string>"
  os_version: "<string>"
  os_build: "<string>"
  edition: "<string>"
  install_date: "<ISO8601 | null>"
  last_boot_time: "<ISO8601 | null>"
  secure_boot: "<boolean | null>"
  bitlocker_status: "<string | null>"
  architecture: "<x64 | ARM64>"
  provider: "<string>"
  provider_version: "<string | null>"
possible_native_sources:
  - Get-WinUtilCurrentSystem (WinUtil)
  - Get-ComputerInfo (PowerShell 7+)
  - Get-CimInstance Win32_OperatingSystem
required_privilege: USER_CONTEXT
side_effect_contract: NONE — pure WMI/CIM query
failure_contract:
  INVALID_REQUEST
  PROVIDER_UNAVAILABLE
  OBSERVATION_FAILED
  EVIDENCE_GENERATION_FAILED
evidence_contract:
  request_id: "<uuid>"
  timestamp: "<ISO8601>"
  operation: "OP-001"
  provider: "<provider_id>"
  provider_version: "<string>"
  intent: "READ"
  enforcement: "READ_ONLY"
  privilege: "USER_CONTEXT"
  result: "success | failure"
  failure_code: "<failure_code | null>"
```

---

### getApplicationInventory

```text
canonical_operation: OP-002
provider_method:     getApplicationInventory
input:
  filters:
    source: ["win32", "appx", "winget", "choco"] | null
    name_pattern: "<string | null>"
normalized_output:
  operation_id: "OP-002"
  timestamp: "<ISO8601>"
  applications:
    - name: "<string>"
      version: "<string | null>"
      source: "win32 | appx | winget | choco"
      install_date: "<ISO8601 | null>"
      scope: "user | machine"
  total: "<integer>"
  provider: "<string>"
possible_native_sources:
  - Invoke-WPFGetInstalled (WinUtil)
  - Get-AppxPackage (PowerShell) — user-scoped only
  - Get-Package (PowerShell)
  - winget list (read-only CLI)
  - choco list (read-only CLI)
required_privilege: USER_CONTEXT
side_effect_contract:
  - Registry read (HKLM/HKCU) — no write
  - winget list / choco list — may touch network cache (read-only)
  - No package install/remove/upgrade
failure_contract:
  INVALID_REQUEST
  PROVIDER_UNAVAILABLE
  OBSERVATION_FAILED
  INSUFFICIENT_PRIVILEGE (if all-users scope requested)
  UNSAFE_PROVIDER_PATH
  PARTIAL_OBSERVATION
  EVIDENCE_GENERATION_FAILED
evidence_contract:
  request_id: "<uuid>"
  timestamp: "<ISO8601>"
  operation: "OP-002"
  provider: "<provider_id>"
  intent: "READ"
  enforcement: "READ_ONLY"
  privilege: "USER_CONTEXT"
  result: "success | failure"
  failure_code: "<failure_code | null>"
  partial: "<boolean>"
```

---

### getTweakState

```text
canonical_operation: OP-003
provider_method:     getTweakState
input:
  category: "privacy | telemetry | ui | services | tasks | all"
  tweak_ids: ["<string>"] | null
normalized_output:
  operation_id: "OP-003"
  timestamp: "<ISO8601>"
  tweaks:
    - id: "<string>"
      name: "<string>"
      category: "<string>"
      current_value: "<any>"
      expected_value: "<any>"
      in_desired_state: "<boolean>"
  provider: "<string>"
possible_native_sources:
  - Get-WinUtilRegistryComboState (WinUtil)
  - Get-WinUtilToggleStatus (WinUtil)
  - Invoke-WinUtilTweaks (WinUtil) — ONLY read path
required_privilege: USER_CONTEXT
side_effect_contract:
  - Registry read only (HKLM/HKCU for tweak keys)
  - NO registry write, NO service modification
  - Parameter binding MUST enforce read-only path
failure_contract:
  INVALID_REQUEST
  PROVIDER_UNAVAILABLE
  OBSERVATION_FAILED
  UNSAFE_PROVIDER_PATH (if write path detected)
  PARTIAL_OBSERVATION
  EVIDENCE_GENERATION_FAILED
evidence_contract:
  request_id: "<uuid>"
  timestamp: "<ISO8601>"
  operation: "OP-003"
  provider: "<provider_id>"
  intent: "READ"
  enforcement: "READ_ONLY"
  privilege: "USER_CONTEXT"
  result: "success | failure"
  failure_code: "<failure_code | null>"
  parameters_enforced: "<boolean>"
```

---

### getProviderCapabilities

```text
canonical_operation: OP-004
provider_method:     getProviderCapabilities
input:               {}
normalized_output:
  operation_id: "OP-004"
  timestamp: "<ISO8601>"
  provider_id: "<string>"
  provider_version: "<string | null>"
  supported_operations: ["OP-001", "OP-002", "OP-003", "OP-005", "OP-006"]
  supported_domains: ["application", "system", "tweak", "registry", "network", "firewall", "storage", "services"]
  max_package_query: "<integer | null>"
  provider_specific: {}
possible_native_sources:
  - Test-WinUtilPackageManager (WinUtil)
  - Get-WinUtilVariables (WinUtil)
  - Static capability manifest
required_privilege: USER_CONTEXT
side_effect_contract:
  - winget --version (read-only process)
  - choco --version (read-only process)
  - No install/upgrade
failure_contract:
  INVALID_REQUEST
  PROVIDER_UNAVAILABLE
  OBSERVATION_FAILED
  EVIDENCE_GENERATION_FAILED
evidence_contract:
  request_id: "<uuid>"
  timestamp: "<ISO8601>"
  operation: "OP-004"
  provider: "<provider_id>"
  intent: "READ"
  enforcement: "READ_ONLY"
  privilege: "USER_CONTEXT"
  result: "success | failure"
  failure_code: "<failure_code | null>"
```

---

### getUpdateStatus

```text
canonical_operation: OP-005
provider_method:     getUpdateStatus
input:
  include_history: "<boolean, default false>"
normalized_output:
  operation_id: "OP-005"
  timestamp: "<ISO8601>"
  service_status: "<string>"
  last_check: "<ISO8601 | null>"
  pending_updates: "<integer>"
  update_history:
    - kb: "<string>"
      title: "<string>"
      installed_on: "<ISO8601 | null>"
      result: "<string>"
  provider: "<string>"
possible_native_sources:
  - Get-HotFix (WMI)
  - Get-PackageProvider (PowerShell)
  - Get-WindowsUpdateLog (event log)
  - NO UsoClient.exe / wuauclt.exe / USO client activation
required_privilege: USER_CONTEXT
side_effect_contract:
  - WMI query for hotfixes
  - Local package provider query
  - Log file read
  - NO update client invocation
  - NO network contact for freshness check
failure_contract:
  INVALID_REQUEST
  PROVIDER_UNAVAILABLE
  OBSERVATION_FAILED
  UNSAFE_PROVIDER_PATH (if update client activation detected)
  PARTIAL_OBSERVATION
  EVIDENCE_GENERATION_FAILED
evidence_contract:
  request_id: "<uuid>"
  timestamp: "<ISO8601>"
  operation: "OP-005"
  provider: "<provider_id>"
  intent: "READ"
  enforcement: "READ_ONLY"
  privilege: "USER_CONTEXT"
  result: "success | failure"
  failure_code: "<failure_code | null>"
  client_activated: "<boolean>"
```

---

### getSystemConfigurationSummary

```text
canonical_operation: OP-006
provider_method:     getSystemConfigurationSummary
input:
  sections: ["network", "firewall", "storage", "services", "tasks"] | null
normalized_output:
  operation_id: "OP-006"
  timestamp: "<ISO8601>"
  network:
    adapters:
      - name: "<string>"
        status: "<string>"
        ip_addresses: ["<string>"]
        dns_servers: ["<string>"]
  firewall:
    profiles:
      domain: "<boolean>"
      private: "<boolean>"
      public: "<boolean>"
  storage:
    - drive_letter: "<string>"
      size_gb: "<number>"
      free_gb: "<number>"
  services_count: "<integer>"
  tasks_count: "<integer>"
  provider: "<string>"
possible_native_sources:
  - Get-NetAdapter
  - Get-DnsClientServerAddress
  - Get-NetFirewallProfile
  - Get-NetIPConfiguration
  - Get-Volume
  - Get-Service — user-scoped only
  - Get-ScheduledTask — user-scoped only
required_privilege: USER_CONTEXT
side_effect_contract:
  - All queries are WMI/CIM reads
  - NO service start/stop, NO firewall rule modification
  - Get-Service scoped to user-context (no admin token)
failure_contract:
  INVALID_REQUEST
  PROVIDER_UNAVAILABLE
  OBSERVATION_FAILED
  INSUFFICIENT_PRIVILEGE (if admin-only query requested)
  PARTIAL_OBSERVATION
  EVIDENCE_GENERATION_FAILED
evidence_contract:
  request_id: "<uuid>"
  timestamp: "<ISO8601>"
  operation: "OP-006"
  provider: "<provider_id>"
  intent: "READ"
  enforcement: "READ_ONLY"
  privilege: "USER_CONTEXT"
  result: "success | failure"
  failure_code: "<failure_code | null>"
  partial: "<boolean>"
```

---

## Provider Security Invariants

The following are **mandatory** for all read providers in v0.1:

```
PROVIDER_WRITE_METHODS = PROHIBITED_IN_V0_1
ARBITRARY_POWERSHELL = PROHIBITED
ARBITRARY_COMMAND_EXECUTION = PROHIBITED
RAW_REGISTRY_WRITE = PROHIBITED
RAW_SERVICE_CONTROL = PROHIBITED
DOWNLOAD_AND_EXECUTE = PROHIBITED
DYNAMIC_SCRIPT_EXECUTION = PROHIBITED
UNBOUNDED_ARGUMENT_FORWARDING = PROHIBITED
PROVIDER_ELEVATION = PROHIBITED
PROVIDER_STATE_MUTATION = PROHIBITED
```

### Forbidden Provider Interface Patterns

Even as internal implementation details, providers MUST NOT expose:

```
run(command)
execute(script)
runPowerShell(command)
invoke(expression)
executeNative(binary, args)
runScript(path)
runAsAdmin(command)
```

Any such pattern in provider code is a **gate violation** and must be removed before v0.1 release.

---

## Provider Independence Guarantee

The WCC-MCP public MCP contract must remain unchanged when:

```
WinUtilReadProvider → NativeWindowsReadProvider
```

or when a new provider is introduced.

This is achieved by:

1. **Canonical operation IDs** (OP-001..OP-006) are the public API surface
2. **Normalized output schemas** are the contract, not provider-native structures
3. **Provider identity** is returned as metadata, not as structural difference
4. **Provider-specific fields** are allowed only under `provider_specific` object
5. **Capability negotiation** via `getProviderCapabilities` enables graceful degradation

---

## Future Provider Registration

When a new provider is registered:

```text
provider_id: "<unique_id>"
provider_type: "winutil | native_windows | custom"
version: "<semver>"
supported_operations: ["OP-001", "OP-002", ...]
constraints: []
```

The core does not need modification. The MCP tool dispatcher routes to the
registered provider based on capability and availability.

---

## Implementation Status

| Provider | OP-001 | OP-002 | OP-003 | OP-004 | OP-005 | OP-006 |
|---|---|---|---|---|---|---|
| WinUtilReadProvider | PLANNED | PLANNED | PLANNED | PLANNED | PLANNED | PLANNED |
| NativeWindowsReadProvider | PLANNED | PLANNED | PLANNED | PLANNED | PLANNED | PLANNED |

**No provider implementation exists at Gate 2B.** This contract defines the
boundary; implementation is Gate 2D (not yet authorized).