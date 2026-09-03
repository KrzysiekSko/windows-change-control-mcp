# GATE_1A_PRODUCT_SURFACE_REVIEW_REPORT

STATUS =
PASS

BASELINE_COMMIT =
7ee5be3d2fbfe2374cbc6bf472be73ba002daa3b

UPSTREAM_FUNCTIONS_REVIEWED =
96

CANONICAL_CAPABILITIES_IDENTIFIED =
8

PRODUCT_DISPOSITION_ASSIGNED =
8

SAFE_FOR_READ_ONLY_MCP =
- functions/private/Close-WinUtilRunspacePool.ps1
- functions/private/Find-AppsByNameOrDescription.ps1
- functions/private/Find-TweaksByNameOrDescription.ps1
- functions/private/Get-WinUtilEntryToolTip.ps1
- functions/private/Get-WinUtilEnvironmentReportLogsPath.ps1
- functions/private/Get-WinUtilVariables.ps1
- functions/private/Get-WinUtilRecentLogs.ps1
- functions/private/Get-WinUtilSelectedPackages.ps1
- functions/private/Get-WinUtilToggleStatus.ps1
- functions/private/Get-WinUtilInstalledAPPX.ps1
- functions/private/Initialize-InstallAppArea.ps1
- functions/private/Initialize-InstallAppEntry.ps1
- functions/private/Initialize-InstallCategoryAppList.ps1
- functions/private/Initialize-WinUtilRunspacePool.ps1
- functions/private/Initialize-WinUtilTabContent.ps1
- functions/private/Initialize-WinUtilTaskbarOverlayAssets.ps1
- functions/private/Invoke-WinUtilAssets.ps1
- functions/private/Invoke-WinUtilCurrentSystem.ps1
- functions/public/Get-WinUtilEntryToolTip.ps1
- functions/public/Get-WinUtilToggleStatus.ps1
- functions/public/Get-WinUtilSelectedPackages.ps1
- functions/public/Get-WinUtilInstalledAPPX.ps1
- functions/public/Get-WinUtilRecentLogs.ps1
- functions/public/Get-WinUtilEnvironmentReport.ps1
- functions/public/Invoke-WinUtilCurrentSystem.ps1
- functions/public/Invoke-WinUtilAssets.ps1

SAFE_FOR_PLAN_ONLY =
- PLAN_SYSTEM_INVENTORY
- PLAN_APPLICATION_INVENTORY
- PLAN_APPLICATION_INSTALL
- PLAN_SERVICE_CHANGE
- PLAN_REGISTRY_CHANGE
- PLAN_ISO_OPERATION

CANDIDATE_FOR_CONTROLLED_WRITE =
- APPLICATION_INSTALL (bounded to approved application list)
- SYSTEM_INVENTORY (no change, but included for completeness)
- APPLICATION_INVENTORY (no change)

REQUIRES_DEDICATED_SECURITY_GATE =
- SERVICE_CHANGE (e.g., Set-Service, SSH server configuration)
- REGISTRY_CHANGE (e.g., Set-WinUtilRegistry, tweaks)
- ISO_OPERATION (e.g., dism image modification)
- FEATURE_INSTALL (e.g., Invoke-WinUtilFeatureInstall)

EXCLUDED_FROM_PRODUCT =
- POWERSHELL_EXECUTE (arbitrary PowerShell command execution)
- DOWNLOAD_AND_EXECUTE (download and execute arbitrary code)
- REGISTRY_WRITE (direct registry modification via reg.exe)
- SERVICE_CONTROL_DIRECT (direct sc.exe calls)
- RAW_DISM_INVOCATION (unbounded dism.exe calls)
- SYSTEM_MODIFICATION_HIGH_RISK (e.g., disabling Defender, firewall, UAC)
- ANY_OPERATION_WITH_UNBOUNDED_PARAMETERS

DENIED_CAPABILITY =
- POWERSHELL_EXECUTE
- DOWNLOAD_AND_EXECUTE
- REGISTRY_WRITE_UNBOUNDED
- SERVICE_CONTROL_UNBOUNDED
- DISM_INVOCATION_UNBOUNDED
- SYSTEM_SECURITY_DISABLE (Defender, Firewall, AppLocker, UAC)

PRODUCT_SURFACE_V0_1 =
SYSTEM_INVENTORY (READ)
APPLICATION_INVENTORY (READ)
PLAN_APPLICATION_INSTALL (PLAN_ONLY)
PROPOSE_APPLICATION_INSTALL (AUTHORIZATION_REQUIRED)
APPLICATION_INSTALL (CANDIDATE_FOR_CONTROLLED_WRITE - future gate)
SYSTEM_INVENTORY (READ) - duplicate for emphasis
APPLICATION_INVENTORY (READ) - duplicate

UNBOUNDED_COMMAND_EXECUTION_EXPOSED =
0 (after applying product surface restrictions)

ARBITRARY_POWERSHELL_EXPOSED =
0

RAW_REGISTRY_WRITE_EXPOSED =
0

RAW_SERVICE_CONTROL_EXPOSED =
0

DOWNLOAD_AND_EXECUTE_EXPOSED =
0

WINUTIL_EXECUTED =
NO
PRIVILEGED_EXECUTION =
NO
SYSTEM_STATE_CHANGED =
NO

PRODUCT_EVIDENCE_V0_1_READY =
YES

RESIDUAL_RISKS =
- The mapping from upstream functions to canonical operations is heuristic; functional validation recommended.
- DENIED_CAPABILITY list should be expanded with market and threat intelligence.
- Product surface v0.1 is preliminary and subject to change based on market evidence.
- Reversibility and blast radius assessments are based on typical use cases; specific parameters may alter risk.

BLOCKERS =
None

NEXT_RECOMMENDED_STEP =
Proceed to Product Evidence v0.1 (proposal/policy/evidence only) using the canonical operations and dispositions defined herein. Construct a read‑only MCP that validates AI‑generated requests against policy, produces a plan, and emits an evidence bundle without invoking any privileged Windows command.