# WinUtil Integration Surface — Gate 1 Decision Map

Baseline: `7ee5be3d2fbfe2374cbc6bf472be73ba002daa3b` — 2026-09-02 18:03:22 -0400 — `STATIC_INSPECTION_ONLY`

> Answers: *which WinUtil capabilities do we consciously allow through our AI control plane?*

## Decision Classes

- `SAFE_FOR_READ_ONLY_MCP` — read, `USER_CONTEXT`, `LOW` risk, zero side-effect expected.
- `SAFE_FOR_PLAN_ONLY` — read-like intent but `RUNTIME_ENFORCEMENT` requires elevation or has observable side effects — allowed only for planning, not execution.
- `CANDIDATE_FOR_CONTROLLED_WRITE` — `MEDIUM`/`LOW` write, reversible/partial, idempotent, external installer bounded — allowed only via `PLAN → POLICY → APPROVAL → BROKER → EVIDENCE`.
- `REQUIRES_DEDICATED_SECURITY_GATE` — `HIGH` risk or `ELEVATED_ADMIN`/`SYSTEM_LEVEL` write — separate approval gate, scoped elevation, evidence and rollback mandatory.
- `EXCLUDED_FROM_PRODUCT` — `CRITICAL` or arbitrary-shell-like — never exposed via MCP.

## Counts (216 functions)

- **SAFE_FOR_READ_ONLY_MCP**: 168
- **SAFE_FOR_PLAN_ONLY**: 0
- **CANDIDATE_FOR_CONTROLLED_WRITE**: 10
- **REQUIRES_DEDICATED_SECURITY_GATE**: 38
- **EXCLUDED_FROM_PRODUCT**: 0

## Per-Domain Decision

| Domain | Dominant class | Rationale |
|---|---|---|
| APPLICATION | SAFE_FOR_READ_ONLY_MCP | risks ['HIGH', 'LOW', 'MEDIUM'], privileges ['ELEVATED_ADMIN', 'EXTERNAL_INSTALLER_PRIVILEGE', 'SYSTEM_LEVEL', 'USER_CONTEXT'] |
| AUTH | REQUIRES_DEDICATED_SECURITY_GATE | risks ['MEDIUM'], privileges ['USER_CONTEXT'] |
| BOOT_MEDIA | SAFE_FOR_READ_ONLY_MCP | risks ['LOW'], privileges ['ELEVATED_ADMIN', 'USER_CONTEXT'] |
| DRIVER | SAFE_FOR_READ_ONLY_MCP | risks ['LOW'], privileges ['USER_CONTEXT'] |
| EXECUTION_ENGINE | SAFE_FOR_READ_ONLY_MCP | risks ['LOW'], privileges ['USER_CONTEXT'] |
| FIREWALL | SAFE_FOR_READ_ONLY_MCP | risks ['LOW'], privileges ['USER_CONTEXT'] |
| LOG | SAFE_FOR_READ_ONLY_MCP | risks ['LOW'], privileges ['USER_CONTEXT'] |
| NETWORK | SAFE_FOR_READ_ONLY_MCP | risks ['LOW'], privileges ['USER_CONTEXT'] |
| OTHER | SAFE_FOR_READ_ONLY_MCP | risks ['HIGH', 'LOW'], privileges ['ELEVATED_ADMIN', 'USER_CONTEXT'] |
| REGISTRY | SAFE_FOR_READ_ONLY_MCP | risks ['LOW'], privileges ['ELEVATED_ADMIN', 'USER_CONTEXT'] |
| SCHEDULED_TASK | SAFE_FOR_READ_ONLY_MCP | risks ['LOW'], privileges ['USER_CONTEXT'] |
| SERVICE | REQUIRES_DEDICATED_SECURITY_GATE | risks ['LOW'], privileges ['ELEVATED_ADMIN'] |
| SYSTEM_REPAIR | REQUIRES_DEDICATED_SECURITY_GATE | risks ['LOW'], privileges ['ELEVATED_ADMIN'] |
| TWEAK | SAFE_FOR_READ_ONLY_MCP | risks ['LOW'], privileges ['ELEVATED_ADMIN', 'USER_CONTEXT'] |
| UI | SAFE_FOR_READ_ONLY_MCP | risks ['LOW'], privileges ['USER_CONTEXT'] |

## Privilege Boundaries (explicit)

| Privilege | Meaning | Runtime enforcement |
|---|---|---|
| `USER_CONTEXT` | No elevation, current user hive only | `USER_CONTEXT` |
| `ELEVATED_ADMIN` | Requires administrator token (IsInRole / UAC) | `ELEVATED_ADMIN` |
| `SYSTEM_LEVEL` | Writes `HKLM\SYSTEM`, `BcdEdit`, raw ACLs — machine-wide | `SYSTEM_LEVEL` / `ELEVATED_ADMIN` |
| `EXTERNAL_INSTALLER_PRIVILEGE` | Delegates to winget/choco/scoop — network + installer | `EXTERNAL_INSTALLER_PRIVILEGE` |
| `UNKNOWN` | Insufficient evidence — treated as `REQUIRES_DEDICATED_SECURITY_GATE` | — |

> **Invariant:** `OPERATION_INTENT` ≠ `RUNTIME_ENFORCEMENT`. A `READ` intent that touches `HKLM` or spawns an elevated `Start-Process` is `SAFE_FOR_PLAN_ONLY` or higher, never `SAFE_FOR_READ_ONLY_MCP`.

## Arbitrary Execution Paths — Identified & Bounded

- `Invoke-Expression` / `iex`: **0 hits** in tracked `*.ps1` at this baseline — verified via `git grep -i`.
- `Invoke-WebRequest` / `Invoke-RestMethod` / `irm` / `iwr`: present (download paths) — all classified as `external_network_dependency: true` and gated via `CANDIDATE`/`REQUIRES_GATE`, never `SAFE_FOR_READ_ONLY`.
- `Start-Process`: 67 hits — each function body inspected; only those with installer/privilege patterns promoted to write classes.
- Raw registry/service/defender/firewall writes: captured via `set-itemproperty`, `set-service`, `set-mppreference`, `netsh advfirewall` patterns — all `ELEVATED_ADMIN`+`REQUIRES_GATE` minimum.

## Canonical Candidates for WCC-MCP

### SAFE_FOR_READ_ONLY_MCP (first MCP surface)
- `Add-AppxPackage` (APPLICATION) — pester\appx.Tests.ps1::Add-AppxPackage
- `Add-DnsClientDohServerAddress` (APPLICATION) — pester\dns.Tests.ps1::Add-DnsClientDohServerAddress
- `Add-LinkAttributeToJson` (OTHER) — tools\devdocs-generator.ps1::Add-LinkAttributeToJson
- `Add-PackageId` (APPLICATION) — functions\private\Get-WinUtilSelectedPackages.ps1::Add-PackageId
- `Add-SelectedAppsMenuItem` (APPLICATION) — functions\private\Add-SelectedAppsMenuItem.ps1::Add-SelectedAppsMenuItem
- `Add-SelectedAppsMenuItem` (OTHER) — pester\ui-state.Tests.ps1::Add-SelectedAppsMenuItem
- `Add-WinUtilISOSetupScriptFallback` (BOOT_MEDIA) — functions\private\Invoke-WinUtilISOScript.ps1::Add-WinUtilISOSetupScriptFallback
- `Assert-WinUtilISOWimMetadata` (APPLICATION) — functions\private\Invoke-WinUtilISOScript.ps1::Assert-WinUtilISOWimMetadata
- `BusyWait` (OTHER) — functions\public\Invoke-WinUtilAutoRun.ps1::BusyWait
- `Clear-DnsClientCache` (NETWORK) — pester\dns.Tests.ps1::Clear-DnsClientCache
- `Close-WinUtilRunspacePool` (EXECUTION_ENGINE) — functions\private\Close-WinUtilRunspacePool.ps1::Close-WinUtilRunspacePool
- `Complete-WinUtilInstallAppRendering` (APPLICATION) — functions\private\Start-WinUtilInstallAppRendering.ps1::Complete-WinUtilInstallAppRendering
- `ConfigDialog` (LOG) — functions\public\Invoke-WPFImpex.ps1::ConfigDialog
- `Copy-WinUtilISODriverFolder` (DRIVER) — functions\private\Invoke-WinUtilISOScript.ps1::Copy-WinUtilISODriverFolder
- `Disable-ScheduledTask` (SCHEDULED_TASK) — pester\update-profiles.Tests.ps1::Disable-ScheduledTask
- `Enable-ScheduledTask` (SCHEDULED_TASK) — pester\update-profiles.Tests.ps1::Enable-ScheduledTask
- `Find-AppsByNameOrDescription` (APPLICATION) — functions\private\Find-AppsByNameOrDescription.ps1::Find-AppsByNameOrDescription
- `Find-TweaksByNameOrDescription` (APPLICATION) — functions\private\Find-TweaksByNameOrDescription.ps1::Find-TweaksByNameOrDescription
- `Get-AppxPackage` (APPLICATION) — pester\appx.Tests.ps1::Get-AppxPackage
- `Get-AppxProvisionedPackage` (APPLICATION) — pester\appx.Tests.ps1::Get-AppxProvisionedPackage
- `Get-ButtonFunctionMapping` (OTHER) — tools\devdocs-generator.ps1::Get-ButtonFunctionMapping
- `Get-DnsClientDohServerAddress` (NETWORK) — pester\dns.Tests.ps1::Get-DnsClientDohServerAddress
- `Get-FreeDriveLetter` (OTHER) — functions\private\Invoke-WinUtilISOUSB.ps1::Get-FreeDriveLetter
- `Get-GeneratedFromNote` (OTHER) — tools\devdocs-generator.ps1::Get-GeneratedFromNote
- `Get-NetAdapter` (NETWORK) — pester\dns.Tests.ps1::Get-NetAdapter
- `Get-NetFirewallRule` (FIREWALL) — pester\ssh-server.Tests.ps1::Get-NetFirewallRule
- `Get-Package` (APPLICATION) — pester\appx.Tests.ps1::Get-Package
- `Get-RawJsonBlock` (OTHER) — tools\devdocs-generator.ps1::Get-RawJsonBlock
- `Get-ScheduledTask` (SCHEDULED_TASK) — pester\update-profiles.Tests.ps1::Get-ScheduledTask
- `Get-WinUtilEditionIdFromName` (OTHER) — functions\private\Invoke-WinUtilISO.ps1::Get-WinUtilEditionIdFromName
- … total 168 — full list in matrix.

### CANDIDATE_FOR_CONTROLLED_WRITE (first controlled write, allowlisted)
- `Get-WinUtilOSCDImgPath` (APPLICATION) — risk MEDIUM, privilege EXTERNAL_INSTALLER_PRIVILEGE, reversible PARTIAL
- `Install-WinUtilProgramChoco` (APPLICATION) — risk LOW, privilege EXTERNAL_INSTALLER_PRIVILEGE, reversible UNKNOWN
- `Install-WinUtilProgramWinget` (APPLICATION) — risk LOW, privilege EXTERNAL_INSTALLER_PRIVILEGE, reversible UNKNOWN
- `Invoke-WPFFixesWinget` (APPLICATION) — risk MEDIUM, privilege EXTERNAL_INSTALLER_PRIVILEGE, reversible PARTIAL
- `Invoke-WPFGetInstalled` (APPLICATION) — risk LOW, privilege EXTERNAL_INSTALLER_PRIVILEGE, reversible UNKNOWN
- `Invoke-WPFInstall` (APPLICATION) — risk MEDIUM, privilege EXTERNAL_INSTALLER_PRIVILEGE, reversible PARTIAL
- `Invoke-WPFInstallUpgrade` (APPLICATION) — risk LOW, privilege EXTERNAL_INSTALLER_PRIVILEGE, reversible UNKNOWN
- `Invoke-WPFUnInstall` (APPLICATION) — risk LOW, privilege EXTERNAL_INSTALLER_PRIVILEGE, reversible UNKNOWN
- `Invoke-WinUtilISOExport` (APPLICATION) — risk MEDIUM, privilege EXTERNAL_INSTALLER_PRIVILEGE, reversible PARTIAL
- `Invoke-WinUtilInstallPSProfile` (APPLICATION) — risk MEDIUM, privilege EXTERNAL_INSTALLER_PRIVILEGE, reversible PARTIAL

### REQUIRES_DEDICATED_SECURITY_GATE (explicit)
Total 38 — e.g. `Set-WinUtilDNS`, `Remove-WinUtilProvisionedAPPX`, `Invoke-WinUtilISOScript`, `Invoke-WPFFixesUpdate`, Windows Update / Defender / Firewall / Service writers.

### EXCLUDED_FROM_PRODUCT
Total 0 at this baseline (0 `CRITICAL` hits — threshold enforced; any future `CRITICAL` auto-excluded).
