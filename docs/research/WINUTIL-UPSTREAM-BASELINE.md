# WinUtil Upstream Research Baseline

## Provenance

Repository: https://github.com/ChrisTitusTech/winutil.git
Remote: origin
Commit SHA: 7ee5be3d2fbfe2374cbc6bf472be73ba002daa3b
Branch: main
Commit date: 2026-09-02 18:03:22 -0400
Clone date: 2026-09-03T05:54:42.088665+00:00
License: MIT License
Worktree status: CLEAN

## Research Scope

What was inspected: git metadata, directory structure, file names, static patterns for PowerShell entrypoints, external download mechanisms, privilege indicators, license.
What was explicitly not executed: No PowerShell scripts were sourced, imported, or executed. No functions invoked. No configuration applied. No installation or tweak functions called.

## Repository Architecture Overview

Top-level directories: config, docs, functions, lint, pester, scripts, tools, xaml
Top-level files: AGENTS.md, CLAUDE.md, Compile.ps1, GEMINI.md, LICENSE, README.md, sign.bat, SPEC.md, windev.ps1

## PowerShell Structure

PowerShell scripts (.ps1) found: Compile.ps1, windev.ps1, functions\private\Add-SelectedAppsMenuItem.ps1, functions\private\Close-WinUtilRunspacePool.ps1, functions\private\Find-AppsByNameOrDescription.ps1, functions\private\Find-TweaksByNameOrDescription.ps1, functions\private\Get-WinUtilEntryToolTip.ps1, functions\private\Get-WinUtilEnvironmentReport.ps1, functions\private\Get-WinUtilEnvironmentReportLogsPath.ps1, functions\private\Get-WinUtilInstalledAPPX.ps1, functions\private\Get-WinUtilPackageLogSummary.ps1, functions\private\Get-WinUtilRecentLogs.ps1, functions\private\Get-WinUtilRegistryComboState.ps1, functions\private\Get-WinUtilRegistryComboValue.ps1, functions\private\Get-WinUtilSelectedPackages.ps1, functions\private\Get-WinUtilToggleStatus.ps1, functions\private\Get-WinUtilTweaksStateReport.ps1, functions\private\Get-WinUtilVariables.ps1, functions\private\Initialize-InstallAppArea.ps1, functions\private\Initialize-InstallAppEntry.ps1, functions\private\Initialize-InstallCategoryAppList.ps1, functions\private\Initialize-WinUtilRunspacePool.ps1, functions\private\Initialize-WinUtilTabContent.ps1, functions\private\Initialize-WinUtilTaskbarOverlayAssets.ps1, functions\private\Install-WinUtilAPPX.ps1, functions\private\Install-WinUtilChoco.ps1, functions\private\Install-WinUtilProgramChoco.ps1, functions\private\Install-WinUtilProgramWinget.ps1, functions\private\Install-WinUtilWinget.ps1, functions\private\Invoke-WinUtilAppCategoryChip.ps1, functions\private\Invoke-WinUtilAssets.ps1, functions\private\Invoke-WinUtilCurrentSystem.ps1, functions\private\Invoke-WinUtilExplorerUpdate.ps1, functions\private\Invoke-WinUtilFeatureInstall.ps1, functions\private\Invoke-WinUtilFontScaling.ps1, functions\private\Invoke-WinUtilInstallPSProfile.ps1, functions\private\Invoke-WinUtilISO.ps1, functions\private\Invoke-WinUtilISOScript.ps1, functions\private\Invoke-WinUtilISOUSB.ps1, functions\private\Invoke-WinUtilScript.ps1, functions\private\Invoke-WinUtilSponsors.ps1, functions\private\Invoke-WinUtilSSHServer.ps1, functions\private\Invoke-WinutilThemeChange.ps1, functions\private\Invoke-WinUtilTweaks.ps1, functions\private\Invoke-WinUtilUninstallPSProfile.ps1, functions\private\New-WinUtilFossBadge.ps1, functions\private\Remove-WinUtilAPPX.ps1, functions\private\Remove-WinUtilProvisionedAPPX.ps1, functions\private\Reset-WPFCheckBoxes.ps1, functions\private\Save-WinUtilFile.ps1, functions\private\Set-WinUtilAppCategoryFilter.ps1, functions\private\Set-WinUtilDNS.ps1, functions\private\Set-WinUtilRegistry.ps1, functions\private\Set-WinUtilRegistryComboState.ps1, functions\private\Set-WinUtilService.ps1, functions\private\Set-WinUtilTaskbarItem.ps1, functions\private\Set-WinUtilTweaksProgressIndicator.ps1, functions\private\Show-CustomDialog.ps1, functions\private\Show-WinUtilMessage.ps1, functions\private\Start-WinUtilInstallAppRendering.ps1, functions\private\Test-WinUtilPackageManager.ps1, functions\private\Update-WinUtilAppCategoryChip.ps1, functions\private\Update-WinUtilSelections.ps1, functions\private\Write-WinUtilLog.ps1, functions\public\Initialize-WPFUI.ps1, functions\public\Invoke-WinUtilAutoRun.ps1, functions\public\Invoke-WPFAppxInstall.ps1, functions\public\Invoke-WPFAppxRemoval.ps1, functions\public\Invoke-WPFButton.ps1, functions\public\Invoke-WPFExportEnvironmentReport.ps1, functions\public\Invoke-WPFFeatureInstall.ps1, functions\public\Invoke-WPFFixesNetwork.ps1, functions\public\Invoke-WPFFixesNTPPool.ps1, functions\public\Invoke-WPFFixesUpdate.ps1, functions\public\Invoke-WPFFixesWinget.ps1, functions\public\Invoke-WPFGetInstalled.ps1, functions\public\Invoke-WPFImpex.ps1, functions\public\Invoke-WPFInstall.ps1, functions\public\Invoke-WPFInstallUpgrade.ps1, functions\public\Invoke-WPFOOSU.ps1, functions\public\Invoke-WPFPanelAutologin.ps1, functions\public\Invoke-WPFPopup.ps1, functions\public\Invoke-WPFPresets.ps1, functions\public\Invoke-WPFRunspace.ps1, functions\public\Invoke-WPFSelectedCheckboxesUpdate.ps1, functions\public\Invoke-WPFSSHServer.ps1, functions\public\Invoke-WPFSystemRepair.ps1, functions\public\Invoke-WPFTab.ps1, functions\public\Invoke-WPFToggleAllCategories.ps1, functions\public\Invoke-WPFtweaksbutton.ps1, functions\public\Invoke-WPFUIElements.ps1, functions\public\Invoke-WPFUIThread.ps1, functions\public\Invoke-WPFUltimatePerformance.ps1, functions\public\Invoke-WPFundoall.ps1, functions\public\Invoke-WPFUnInstall.ps1, functions\public\Invoke-WPFUpdatesdefault.ps1, functions\public\Invoke-WPFUpdatesdisable.ps1, functions\public\Invoke-WPFUpdatessecurity.ps1, lint\PSScriptAnalyser.ps1, pester\activity-history.Tests.ps1, pester\appx.Tests.ps1, pester\assets.Tests.ps1, pester\configs.Tests.ps1, pester\dns.Tests.ps1, pester\environment-report-logs.Tests.ps1, pester\environment-report.Tests.ps1, pester\functions.Tests.ps1, pester\headless-ui.Tests.ps1, pester\install-rendering.Tests.ps1, pester\install-workflow.Tests.ps1, pester\lazy-tabs.Tests.ps1, pester\logging.Tests.ps1, pester\multiplane-overlay.Tests.ps1, pester\oosu.Tests.ps1, pester\package.Tests.ps1, pester\preferences-theme.Tests.ps1, pester\runspace-lifecycle.Tests.ps1, pester\runspace.Tests.ps1, pester\sanity.Tests.ps1, pester\search-filter.Tests.ps1, pester\ssh-server.Tests.ps1, pester\system-helpers.Tests.ps1, pester\toggle-status.Tests.ps1, pester\tooltip-key.Tests.ps1, pester\triage-tools.Tests.ps1, pester\tweaks.Tests.ps1, pester\ui-state.Tests.ps1, pester\update-profiles.Tests.ps1, pester\variables.Tests.ps1, pester\win11creator.Tests.ps1, pester\winoneshot-compat.Tests.ps1, pester\xaml.Tests.ps1, scripts\main.ps1, scripts\start.ps1, tools\devdocs-generator.ps1
Modules/functions directories: functions

## Privilege Assumptions

Indicators found (counts): {
  "IsInRole": 1,
  "RunAsAdministrator": 0,
  "Require-Admin": 0,
  "#Requires": 0
}

## External Download and Execution Mechanisms

Pattern search results (number of matches): {
  "Invoke-WebRequest": 6,
  "Invoke-RestMethod": 4,
  "irm": 58,
  "iwr": 0,
  "Invoke-Expression": 0,
  "iex": 20,
  "Start-Process": 67,
  "DownloadString": 0,
  "WebClient": 0,
  "curl": 1,
  "wget": 0,
  "winget": 522,
  "choco": 397,
  "scoop": 0,
  "git clone": 1
}

## Configuration Model

Configuration directories: config

## Test Model

Test directories: pester

## Potential Integration Surfaces

Based on static inspection, candidate areas suitable for a future provider adapter include:
- Application inventory functions (if discovered)
- Application installation functions (if discovered)
- Configuration export/import functions
- Tweaks functions (selected low-risk only)
- Fixes functions (selected reversible only)
- Windows Update query functions (read-only)
- MicroWin (if applicable, likely high-risk)
- Defender/query functions (read-only)
- Service query functions (read-only)
- Registry query functions (read-only)

## Known High-Risk Areas

Based on typical WinUtil functionality and patterns observed, high-risk areas likely include:
- Defender modifications (if any Invoke-Expression or Set-MpPreference etc)
- Service modifications (Start-Service, Stop-Service, Set-Service)
- Registry writes (especially HKLM)
- Windows Update policy changes
- Boot configuration (bcdedit)
- MicroWin image creation
- Aggressive debloat (removing Windows features)
- Credential harvesting patterns (not expected but searched)

Only include areas supported by evidence. Current evidence based on pattern search and directory names.

## Supply-Chain Notes

The repository relies on external downloads via patterns such as Invoke-WebRequest/Invoke-RestMethod for fetching scripts, configurations, or binaries. The presence of `irm` (alias for Invoke-WebRequest) in the typical installation method indicates a supply-chain risk that must be controlled via pinned versions and hash verification.

## Research Conclusions

Facts:
- The upstream repository is licensed under MIT.
- The worktree is clean at commit 7ee5be3d2fbfe2374cbc6bf472be73ba002daa3b.
- The repository contains PowerShell scripts and modules.
- External download mechanisms are present (Invoke-WebRequest etc).
- Privilege checks (IsInRole) are present, indicating awareness of admin requirements.

Assumptions:
- The script entrypoints are intended to be run with administrative privileges for system changes.
- External downloads are used to fetch additional components or presets.

Hypotheses:
- Application inventory and installation capabilities exist and may be suitable for MCP with low risk.
- Tweaks and fixes may contain both reversible and irreversible operations; further classification needed.
- Defender, service, registry, and boot modifications are likely high-risk and should be excluded from initial MVP.

## Open Questions for Gate 1

- Which specific functions provide application inventory and are they reversible/idempotent?
- Which tweaks are reversible and low-risk?
- Which fixes are safe and reversible?
- How does WinUtil handle configuration import/export?
- What is the exact privilege model for each function (user vs admin)?
- Are there any signed binaries or checksums for downloaded resources?

## Baseline Freeze Confirmation (Gate 0.5)

- Current HEAD: 7ee5be3d2fbfe2374cbc6bf472be73ba002daa3b
- Expected HEAD: 7ee5be3d2fbfe2374cbc6bf472be73ba002daa3b
- HEAD matches expected: True
- Branch: main
- Origin: https://github.com/ChrisTitusTech/winutil.git
- Expected origin: https://github.com/ChrisTitusTech/winutil.git
- Origin matches expected: True
- Worktree status: CLEAN (porcelain lines: 0)
- Baseline frozen: YES
- Freeze timestamp: czw. 03.09.2026 07:57
