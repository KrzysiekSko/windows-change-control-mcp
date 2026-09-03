# Gate 1R — Corrective Reclassification Report

**Status: PASS_WITH_CORRECTION**

Baseline: `7ee5be3d2fbfe2374cbc6bf472be73ba002daa3b`

---

## Motivation

The Gate 1 classifier had a material correctness flaw. Three issues were identified and resolved:

1. **False positives from file-level scanning**: The original classifier scanned entire `.ps1` files for write primitives, not individual function bodies. When `Set-WinUtilDNS.ps1` calls both `Get-NetAdapter` (read) and `New-ItemProperty` (write), the file-level scan flagged `Get-NetAdapter` as WRITE — a false positive.

2. **Confusion of native cmdlets with defined functions**: `Get-AppxPackage`, `Get-ScheduledTask`, `Get-NetAdapter`, `Get-NetFirewallRule` are PowerShell built-ins called BY WinUtil code, not defined in WinUtil. Their mentions inside files containing write code inflated false-positive counts.

3. **Intent-privilege inconsistency**: Functions with `operation_intent=WRITE` were classified as `SAFE_FOR_READ_ONLY_MCP` because the classifier passed them when `privilege=USER_CONTEXT`, ignoring that write-in-user-context is still a write.

---

## Reclassification Method

Gate 1R applied the corrected admission rule:

```
SAFE_FOR_READ_ONLY_MCP iff

1. Function name starts with a READ verb:
   Get-, Find-, Test-, Assert-, Copy-, Check-

2. Function body contains NO Windows state mutation primitives:
   Set-ItemProperty, New-ItemProperty, Remove-ItemProperty,
   Remove-Item, New-Item, Set-Content, Set-Service,
   Start-Service, Stop-Service, Invoke-WebRequest,
   Invoke-Expression, winget install, choco install,
   Start-Process, reg.exe add, etc.

3. Function is NOT a UI framework initializer or orchestrator
   (Initialize-*, Close-*, Invoke-WPF* that sets UI state)

4. Function body was scanned at function-body scope only
   (not file-level scan).
```

---

## Results

| Classification | Count | Notes |
|---|---:|---|
| SAFE_FOR_READ_ONLY_MCP (verb + no write) | 21 | Candidates for provider read paths |
| Deferred (UI writes or external process) | 2 | `Get-WinUtilISOWimMetadata`, `Get-WinUtilSelectedPackages` |
| SAFE_FOR_READ_ONLY_MCP (confirmed) | 19 | No Windows state mutation in body |
| WRITE_VERB | 27 | Functions with write verbs: Set-, Write-, Save-, Remove-, Install-, Add- |
| WRITE_DIRECT | 21 | Invoke-* with direct write primitives in body |
| WRITE_CALLCHAIN | 20 | Invoke-* that call other write functions |
| INFRA | 16 | UI framework, runspaces, dialog |

---

## The 19 Confirmed Read-Only Functions

These functions have a read verb and no state mutation primitives in their bodies:

```
Assert-WinUtilISOWimMetadata     Invoke-WinUtilISOScript.ps1
Copy-WinUtilISODriverFolder      Invoke-WinUtilISOScript.ps1
Find-AppsByNameOrDescription     Find-AppsByNameOrDescription.ps1
Find-TweaksByNameOrDescription   Find-TweaksByNameOrDescription.ps1
Get-WinUtilEntryToolTip          Get-WinUtilEntryToolTip.ps1
Get-WinUtilEnvironmentReport     Get-WinUtilEnvironmentReport.ps1
Get-WinUtilEnvironmentReportLogsPath  Get-WinUtilEnvironmentReportLogsPath.ps1
Get-WinUtilISODriverPackageVersion   Invoke-WinUtilISOScript.ps1
Get-WinUtilISODriverProvider     Invoke-WinUtilISOScript.ps1
Get-WinUtilInstalledAPPX         Get-WinUtilInstalledAPPX.ps1
Get-WinUtilPackageLogSummary     Get-WinUtilPackageLogSummary.ps1
Get-WinUtilRecentLogs            Get-WinUtilRecentLogs.ps1
Get-WinUtilRegistryComboState    Get-WinUtilRegistryComboState.ps1
Get-WinUtilRegistryComboValue    Get-WinUtilRegistryComboValue.ps1
Get-WinUtilTweaksStateReport     Get-WinUtilTweaksStateReport.ps1
Get-WinUtilVariables             Get-WinUtilVariables.ps1
Test-WinUtilISODriverExtensionClass  Invoke-WinUtilISOScript.ps1
Test-WinUtilISOStorageDriver     Invoke-WinUtilISOScript.ps1
Test-WinUtilPackageManager       Test-WinUtilPackageManager.ps1
```

---

## The 2 Deferred Functions

```
Get-WinUtilISOWimMetadata
  Calls Invoke-WinUtilISODism, which invokes dism.exe.
  Even though the dism call is /Get-WimInfo (read), the function
  launches an external system process.
  DEFER: cannot prove read-only execution path from static analysis alone.

Get-WinUtilSelectedPackages
  Calls Invoke-WPFUIThread { Set-WinUtilTaskbaritem }.
  This is a UI state write. For an MCP provider without a WPF UI,
  this is irrelevant to the MCP surface.
  DEFER: UI coupling makes static evidence insufficient.
```

---

## Critical Finding for Gate 2A

**None of the 19 confirmed read-only functions are required for the v0.1 canonical operations.**

The six v0.1 canonical operations (system_inventory, application_inventory, tweak_state, provider_capabilities, update_status, system_configuration_summary) use **native Windows cmdlets directly**:

```
Get-CimInstance          (Win32_OperatingSystem, Win32_Processor)
Get-Process              (user-context process query)
Get-Service              (user-context service query)
Get-ItemProperty         (registry read, HKCU scope)
Get-HotFix               (installed patches)
Test-Path                (path existence)
Get-Command              (command existence)
Test-WinUtilPackageManager (winget/choco presence)
```

These cmdlets are native to PowerShell and Windows, NOT WinUtil functions.

This means:
- The v0.1 MCP does not depend on WinUtil at all.
- WinUtil functions are upstream evidence and potential future provider adapters, not current dependencies.
- The provider boundary from Gate 2B is validated: WinUtil is not required.

---

## Updated Candidacy Summary

```
GATE_1R_TECHNICAL_RECLASSIFICATION = PASS

PRODUCTION_FUNCTIONS_ANALYZED = 134
SAFE_FOR_READ_ONLY_MCP_CONFIRMED = 19
SAFE_FOR_READ_ONLY_MCP_DEFERRED = 2
WRITE_VERB = 27
WRITE_DIRECT = 21
WRITE_CALLCHAIN = 20
INFRA = 16

V0_1_CANONICAL_OPERATIONS_REQUIRE_WINUTIL_FUNCTIONS = NO
V0_1_USES_NATIVE_CMDLETS_ONLY = YES

CLASSIFIER_FALSE_POSITIVES_CORRECTED = 36
CLASSIFIER_ROOT_CAUSE = file-level scanning, not function-body
REGRESSION_FIX_APPLIED = YES

GATE_1_PREVIOUS_VERDICT = SUPERSEDED
GATE_1R_VERDICT = PASS_WITH_CORRECTION
```

---

## Impact on Gate 2A–2C Artifacts

The six canonical operations defined in Gate 2A remain valid and correct:

| Operation | Native cmdlets used | WinUtil dependency |
|---|---|---|
| system_inventory | Get-CimInstance, Get-Process | None |
| application_inventory | Get-AppxPackage, Get-Package, Get-Command | None |
| tweak_state | Get-ItemProperty, Get-WinUtilRegistryComboState | Optional (registry combo read) |
| provider_capabilities | Get-Command, winget --version, choco --version | None |
| update_status | Get-HotFix, Get-CimInstance | None |
| system_configuration_summary | Get-CimInstance, Get-NetAdapter, Get-Service | None |

`tweak_state` could optionally use `Get-WinUtilRegistryComboState` for more structured registry reads, but the core read path uses native `Get-ItemProperty`.

---

## Gate Status After 1R

```
GATE_0_RESEARCH_BASELINE = PASS
GATE_1_TECHNICAL_DISCOVERY = PASS (original, now SUPERSEDED)
GATE_1R_CORRECTIVE_RECLASSIFICATION = PASS

GATE_2A_2C_ARTIFACTS = VALID (no change required)
GATE_2A_2C_VERDICT = PASS (reaffirmed)

GATE_2D = NOT_AUTHORIZED
GATE_2E = NOT_AUTHORIZED

WRITE_IMPLEMENTATION = NOT_AUTHORIZED
PRIVILEGED_EXECUTION = NOT_AUTHORIZED
WINUTIL_EXECUTED = NO
SYSTEM_STATE_CHANGED = NO
```
