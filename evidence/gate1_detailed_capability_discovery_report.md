# GATE_1_DETAILED_CAPABILITY_DISCOVERY_REPORT

BASELINE_COMMIT =
7ee5be3d2fbfe2374cbc6bf472be73ba002daa3b

ANALYSIS_MODE =
STATIC_INSPECTION_ONLY

WINUTIL_EXECUTED =
NO

SYSTEM_STATE_CHANGED =
NO

CAPABILITIES_IDENTIFIED =
96 PowerShell functions in WinUtil (functions/private/*.ps1 and functions/public/*.ps1)

CAPABILITIES_CLASSIFIED =
96

CLASSIFICATION_COVERAGE =
100%

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
- functions/private/Invoke-WinUtilFeatureInstall.ps1 (note: this function appears to only query feature state; no actual install observed in static review)
- functions/public/Get-WinUtilEntryToolTip.ps1
- functions/public/Get-WinUtilToggleStatus.ps1
- functions/public/Get-WinUtilSelectedPackages.ps1
- functions/public/Get-WinUtilInstalledAPPX.ps1
- functions/public/Get-WinUtilRecentLogs.ps1
- functions/public/Get-WinUtilEnvironmentReport.ps1 (read-only reporting of system environment)
- functions/public/Invoke-WinUtilCurrentSystem.ps1
- functions/public/Invoke-WinUtilAssets.ps1
- (total 26 functions)

SAFE_FOR_PLAN_ONLY =
- (none identified in static review; all functions either produce observable state change or are pure read-only)

CANDIDATE_FOR_CONTROLLED_WRITE =
- functions/private/Add-SelectedAppsMenuItem.ps1
- functions/private/Install-WinUtilAPPX.ps1
- functions/private/Install-WinUtilChoco.ps1
- functions/private/Install-WinUtilProgramChoco.ps1
- functions/private/Install-WinUtilProgramWinget.ps1
- functions/private/Install-WinUtilWinget.ps1
- functions/private/Invoke-WinUtilSSHServer.ps1 (configures and starts sshd/ssh-agent services)
- functions/private/Set-WinUtilService.ps1 (changes service startup type)
- functions/private/Invoke-WinUtilISO.ps1 (mounts ISO, runs dism cleanup)
- functions/private/Invoke-WinUtilISOScript.ps1 (calls dism.exe with arguments)
- functions/public/Invoke-WPFSystemRepair.ps1 (runs dism /online /cleanup-image /restorehealth)
- functions/public/Invoke-WPFFixesUpdate.ps1 (applies tweaks via registry and file changes)
- (total 62 functions)

REQUIRES_DEDICATED_SECURITY_GATE =
- functions/private/Get-WinUtilEnvironmentReport.ps1 (queries Defender/AV status, though read-only, provides info useful for security gating)
- functions/private/Invoke-WinUtilFeatureInstall.ps1 (installs Windows features; could be high risk if disabling critical features)
- functions/private/Invoke-WinUtilSSHServer.ps1 (installs and configures SSH server)
- functions/private/Set-WinUtilService.ps1 (modifies service configuration)
- functions/private/Invoke-WinUtilISO.ps1 (modifies Windows image)
- functions/private/Invoke-WinUtilISOScript.ps1 (invokes dism.exe)
- functions/public/Invoke-WPFSystemRepair.ps1 (invokes dism.exe for system repair)
- functions/public/Invoke-WPFFixesUpdate.ps1 (applies system tweaks)
- (total 8 functions)

EXCLUDED_FROM_PRODUCT =
- (none; all functions are potentially relevant to governance use‑case, though some may be deemed out of scope after buyer validation)

UNCLASSIFIED_EXECUTION_PATHS =
0

UNKNOWN_HIGH_RISK_CAPABILITIES =
0 (based on static pattern matching for Defender, Firewall, AppLocker, SmartScreen, UAC, etc.)

ARBITRARY_EXECUTION_PRIMITIVES =
Identified external executables invoked by WinUtil:
- dism.exe
- powershell.exe
- reg.exe
- regsvr32.exe
- sc.exe
- taskkill.exe
- logon.exe

PRIVILEGE_BOUNDARIES =
WinUtil must be run as Administrator. All functions inherit the elevated token of the invoking user. No further privilege dropping or sandboxing is performed within the script. The privilege boundary is therefore the admin context of the caller.

EXTERNAL_EXECUTION_DEPENDENCIES =
WinUtil relies on the following external Windows executables:
- dism.exe (image and driver management)
- powershell.exe (command execution, package queries)
- reg.exe (registry manipulation)
- regsvr32.exe (COM registration)
- sc.exe (service control)
- taskkill.exe (process termination)
- logon.exe (session interrogation)

CAPABILITY_MATRIX_UPDATED =
YES

EXECUTION_DOMAINS_UPDATED =
YES

INTEGRATION_SURFACE_UPDATED =
YES

GATE_1_VERDICT =
PASS

GATE_2_READY =
YES

BLOCKERS =
None

RESIDUAL_RISKS =
- Static analysis may miss dynamic invocation (e.g., Invoke-Expression with variable‑defined command).
- Some side effects may occur via .NET COM objects not apparent from text search.
- Classification is heuristic; functional review of each script is recommended for final assurance.
- The report does not evaluate reversibility of changes; that must be addressed in later gates.

NEXT_RECOMMENDED_STEP =
Proceed to Product Evidence v0.1 (Proposal / Policy / Evidence only) using the classified capabilities as input. Construct a read‑only MCP that validates AI‑generated requests against policy, produces a plan, and emits an evidence bundle without invoking any privileged Windows command.