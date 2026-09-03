# WinUtil Execution Domains — Gate 1

Baseline: `7ee5be3d2fbfe2374cbc6bf472be73ba002daa3b` (main) — 2026-09-02 18:03:22 -0400

Method: static inspection of all 135 `*.ps1` files, 216 `function` definitions. No code executed, no modules imported.

Pipeline per function: `FUNCTION → DOMAIN → SIDE_EFFECT_PATH → CAPABILITY → INTENT → PRIVILEGE → REVERSIBILITY → IDEMPOTENCY → OBSERVABILITY → EXTERNAL_DEP → RISK → MCP_CANDIDACY`

## Domain Summary

| Domain | Functions | Privilege split | Risk split | Dominant candidacy |
|---|---:|---|---|---|
| APPLICATION | 99 | {'USER_CONTEXT': 66, 'ELEVATED_ADMIN': 18, 'EXTERNAL_INSTALLER_PRIVILEGE': 10, 'SYSTEM_LEVEL': 5} | {'LOW': 85, 'MEDIUM': 7, 'HIGH': 7} | SAFE_FOR_READ_ONLY_MCP |
| AUTH | 1 | {'USER_CONTEXT': 1} | {'MEDIUM': 1} | REQUIRES_DEDICATED_SECURITY_GATE |
| BOOT_MEDIA | 10 | {'USER_CONTEXT': 8, 'ELEVATED_ADMIN': 2} | {'LOW': 10} | SAFE_FOR_READ_ONLY_MCP |
| DRIVER | 4 | {'USER_CONTEXT': 4} | {'LOW': 4} | SAFE_FOR_READ_ONLY_MCP |
| EXECUTION_ENGINE | 21 | {'USER_CONTEXT': 21} | {'LOW': 21} | SAFE_FOR_READ_ONLY_MCP |
| FIREWALL | 2 | {'USER_CONTEXT': 2} | {'LOW': 2} | SAFE_FOR_READ_ONLY_MCP |
| LOG | 16 | {'USER_CONTEXT': 16} | {'LOW': 16} | SAFE_FOR_READ_ONLY_MCP |
| NETWORK | 7 | {'USER_CONTEXT': 7} | {'LOW': 7} | SAFE_FOR_READ_ONLY_MCP |
| OTHER | 19 | {'ELEVATED_ADMIN': 2, 'USER_CONTEXT': 17} | {'LOW': 18, 'HIGH': 1} | SAFE_FOR_READ_ONLY_MCP |
| REGISTRY | 6 | {'USER_CONTEXT': 5, 'ELEVATED_ADMIN': 1} | {'LOW': 6} | SAFE_FOR_READ_ONLY_MCP |
| SCHEDULED_TASK | 6 | {'USER_CONTEXT': 6} | {'LOW': 6} | SAFE_FOR_READ_ONLY_MCP |
| SERVICE | 5 | {'ELEVATED_ADMIN': 5} | {'LOW': 5} | REQUIRES_DEDICATED_SECURITY_GATE |
| SYSTEM_REPAIR | 2 | {'ELEVATED_ADMIN': 2} | {'LOW': 2} | REQUIRES_DEDICATED_SECURITY_GATE |
| TWEAK | 9 | {'ELEVATED_ADMIN': 2, 'USER_CONTEXT': 7} | {'LOW': 9} | SAFE_FOR_READ_ONLY_MCP |
| UI | 9 | {'USER_CONTEXT': 9} | {'LOW': 9} | SAFE_FOR_READ_ONLY_MCP |

## Domain Details

### APPLICATION — 99 functions

| Function | File | Intent | Privilege → Runtime | Risk | Candidacy |
|---|---|---|---|---|---|
| `Add-AppxPackage` | `pester\appx.Tests.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Add-DnsClientDohServerAddress` | `pester\dns.Tests.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Add-PackageId` | `functions\private\Get-WinUtilSelectedPackages.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Add-SelectedAppsMenuItem` | `functions\private\Add-SelectedAppsMenuItem.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Add-WinUtilISOSetupCustomizations` | `functions\private\Invoke-WinUtilISOScript.ps1` | WRITE | SYSTEM_LEVEL → SYSTEM_LEVEL | HIGH | REQUIRES_DEDICATED_SECURITY_GATE |
| `Add-WinUtilISOStagedDrivers` | `functions\private\Invoke-WinUtilISOScript.ps1` | WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `Assert-WinUtilISOWimMetadata` | `functions\private\Invoke-WinUtilISOScript.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Complete-WinUtilInstallAppRendering` | `functions\private\Start-WinUtilInstallAppRendering.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Export-WinUtilTestDriverPackage` | `pester\win11creator.Tests.ps1` | WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `Find-AppsByNameOrDescription` | `functions\private\Find-AppsByNameOrDescription.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Find-TweaksByNameOrDescription` | `functions\private\Find-TweaksByNameOrDescription.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-AppxPackage` | `pester\appx.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-AppxProvisionedPackage` | `pester\appx.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-Package` | `pester\appx.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-WinUtilEntryToolTip` | `functions\private\Get-WinUtilEntryToolTip.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-WinUtilEnvironmentReport` | `functions\private\Get-WinUtilEnvironmentReport.ps1` | READ_MAYBE_WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `Get-WinUtilISODriverPackageVersion` | `functions\private\Invoke-WinUtilISOScript.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-WinUtilInstalledAPPX` | `functions\private\Get-WinUtilInstalledAPPX.ps1` | READ_MAYBE_WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `Get-WinUtilOSCDImgPath` | `functions\private\Invoke-WinUtilISO.ps1` | WRITE | EXTERNAL_INSTALLER_PRIVILEGE → EXTERNAL_INSTALLER_PRIVILEGE | MEDIUM | CANDIDATE_FOR_CONTROLLED_WRITE |
| `Get-WinUtilPackageLogSummary` | `functions\private\Get-WinUtilPackageLogSummary.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-WinUtilSelectedPackages` | `functions\private\Get-WinUtilSelectedPackages.ps1` | READ_MAYBE_WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `Get-WinUtilSelectedPackages` | `pester\install-workflow.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-WinUtilTweaksStateReport` | `functions\private\Get-WinUtilTweaksStateReport.ps1` | READ_MAYBE_WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `Get-WindowsCapability` | `pester\ssh-server.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Initialize-InstallAppArea` | `functions\private\Initialize-InstallAppArea.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Initialize-InstallAppEntry` | `functions\private\Initialize-InstallAppEntry.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Initialize-InstallCategoryAppList` | `functions\private\Initialize-InstallCategoryAppList.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Initialize-WPFUI` | `functions\public\Initialize-WPFUI.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Initialize-WinUtilTabContent` | `functions\private\Initialize-WinUtilTabContent.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Install-WinUtilAPPX` | `functions\private\Install-WinUtilAPPX.ps1` | WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | MEDIUM | REQUIRES_DEDICATED_SECURITY_GATE |
| `Install-WinUtilChoco` | `functions\private\Install-WinUtilChoco.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Install-WinUtilChoco` | `pester\install-workflow.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Install-WinUtilProgramChoco` | `functions\private\Install-WinUtilProgramChoco.ps1` | WRITE | EXTERNAL_INSTALLER_PRIVILEGE → EXTERNAL_INSTALLER_PRIVILEGE | LOW | CANDIDATE_FOR_CONTROLLED_WRITE |
| `Install-WinUtilProgramChoco` | `pester\install-workflow.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Install-WinUtilProgramWinget` | `functions\private\Install-WinUtilProgramWinget.ps1` | WRITE | EXTERNAL_INSTALLER_PRIVILEGE → EXTERNAL_INSTALLER_PRIVILEGE | LOW | CANDIDATE_FOR_CONTROLLED_WRITE |
| `Install-WinUtilProgramWinget` | `pester\appx.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Install-WinUtilProgramWinget` | `pester\install-workflow.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Install-WinUtilWinget` | `functions\private\Install-WinUtilWinget.ps1` | WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `Install-WinUtilWinget` | `pester\appx.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Install-WinUtilWinget` | `pester\install-workflow.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| … +59 more | | | | | see `WINUTIL_CAPABILITY_MATRIX.yaml` |

### AUTH — 1 functions

| Function | File | Intent | Privilege → Runtime | Risk | Candidacy |
|---|---|---|---|---|---|
| `Invoke-WPFPanelAutologin` | `functions\public\Invoke-WPFPanelAutologin.ps1` | READ | USER_CONTEXT → USER_CONTEXT | MEDIUM | REQUIRES_DEDICATED_SECURITY_GATE |

### BOOT_MEDIA — 10 functions

| Function | File | Intent | Privilege → Runtime | Risk | Candidacy |
|---|---|---|---|---|---|
| `Add-WinUtilISOSetupScriptFallback` | `functions\private\Invoke-WinUtilISOScript.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-WinUtilISOWimMetadata` | `functions\private\Invoke-WinUtilISOScript.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WinUtilISOBrowse` | `functions\private\Invoke-WinUtilISO.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WinUtilISOCheckExistingWork` | `functions\private\Invoke-WinUtilISO.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WinUtilISOCheckExistingWork` | `pester\lazy-tabs.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WinUtilISODism` | `functions\private\Invoke-WinUtilISOScript.ps1` | WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `Invoke-WinUtilISORefreshUSBDrives` | `functions\private\Invoke-WinUtilISOUSB.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WinUtilISOScript` | `functions\private\Invoke-WinUtilISO.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Test-WinUtilISOMountedImage` | `functions\private\Invoke-WinUtilISOScript.ps1` | WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `Write-WinUtilISOEditionConfig` | `functions\private\Invoke-WinUtilISOScript.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |

### DRIVER — 4 functions

| Function | File | Intent | Privilege → Runtime | Risk | Candidacy |
|---|---|---|---|---|---|
| `Copy-WinUtilISODriverFolder` | `functions\private\Invoke-WinUtilISOScript.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-WinUtilISODriverProvider` | `functions\private\Invoke-WinUtilISOScript.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Test-WinUtilISODriverExtensionClass` | `functions\private\Invoke-WinUtilISOScript.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Test-WinUtilISOStorageDriver` | `functions\private\Invoke-WinUtilISOScript.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |

### EXECUTION_ENGINE — 21 functions

| Function | File | Intent | Privilege → Runtime | Risk | Candidacy |
|---|---|---|---|---|---|
| `Close-WinUtilRunspacePool` | `functions\private\Close-WinUtilRunspacePool.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Initialize-WinUtilRunspacePool` | `functions\private\Initialize-WinUtilRunspacePool.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WPFPopup` | `functions\public\Invoke-WPFPopup.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WPFRunspace` | `pester\appx.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WPFRunspace` | `pester\install-workflow.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WPFRunspace` | `pester\oosu.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WPFRunspace` | `pester\tweaks.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WPFRunspace` | `pester\ui-state.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WPFUIElements` | `pester\lazy-tabs.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WPFUIThread` | `pester\appx.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WPFUIThread` | `pester\install-workflow.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WPFUIThread` | `pester\package.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WPFUIThread` | `pester\tweaks.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WPFUIThread` | `pester\ui-state.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WinUtilAssets` | `functions\private\Invoke-WinUtilAssets.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WinUtilCurrentSystem` | `pester\ui-state.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WinUtilExplorerUpdate` | `functions\private\Invoke-WinUtilExplorerUpdate.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WinUtilFontScaling` | `functions\private\Invoke-WinUtilFontScaling.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WinUtilScript` | `functions\private\Invoke-WinUtilScript.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WinUtilScript` | `pester\tweaks.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WinUtilSponsors` | `functions\private\Invoke-WinUtilSponsors.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |

### FIREWALL — 2 functions

| Function | File | Intent | Privilege → Runtime | Risk | Candidacy |
|---|---|---|---|---|---|
| `Get-NetFirewallRule` | `pester\ssh-server.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `New-NetFirewallRule` | `pester\ssh-server.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |

### LOG — 16 functions

| Function | File | Intent | Privilege → Runtime | Risk | Candidacy |
|---|---|---|---|---|---|
| `ConfigDialog` | `functions\public\Invoke-WPFImpex.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-WinUtilEnvironmentReportLogsPath` | `functions\private\Get-WinUtilEnvironmentReportLogsPath.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-WinUtilRecentLogs` | `functions\private\Get-WinUtilRecentLogs.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Write-WinUtilISOLog` | `functions\private\Invoke-WinUtilISO.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Write-WinUtilISOLog` | `functions\private\Invoke-WinUtilISO.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Write-WinUtilISOLog` | `functions\private\Invoke-WinUtilISO.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Write-WinUtilLog` | `functions\private\Write-WinUtilLog.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Write-WinUtilLog` | `pester\appx.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Write-WinUtilLog` | `pester\dns.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Write-WinUtilLog` | `pester\install-workflow.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Write-WinUtilLog` | `pester\oosu.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Write-WinUtilLog` | `pester\package.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Write-WinUtilLog` | `pester\system-helpers.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Write-WinUtilLog` | `pester\tweaks.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Write-WinUtilLog` | `pester\ui-state.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Write-WinUtilLog` | `pester\update-profiles.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |

### NETWORK — 7 functions

| Function | File | Intent | Privilege → Runtime | Risk | Candidacy |
|---|---|---|---|---|---|
| `Clear-DnsClientCache` | `pester\dns.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-DnsClientDohServerAddress` | `pester\dns.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-NetAdapter` | `pester\dns.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WPFFixesNetwork` | `functions\public\Invoke-WPFFixesNetwork.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Remove-DnsClientDohServerAddress` | `pester\dns.Tests.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Set-DnsClientServerAddress` | `pester\dns.Tests.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Set-WinUtilDNS` | `pester\tweaks.Tests.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |

### OTHER — 19 functions

| Function | File | Intent | Privilege → Runtime | Risk | Candidacy |
|---|---|---|---|---|---|
| `Add-LinkAttributeToJson` | `tools\devdocs-generator.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Add-SelectedAppsMenuItem` | `pester\ui-state.Tests.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Add-WindowsCapability` | `pester\ssh-server.Tests.ps1` | WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | HIGH | REQUIRES_DEDICATED_SECURITY_GATE |
| `BusyWait` | `functions\public\Invoke-WinUtilAutoRun.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-ButtonFunctionMapping` | `tools\devdocs-generator.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-FreeDriveLetter` | `functions\private\Invoke-WinUtilISOUSB.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-GeneratedFromNote` | `tools\devdocs-generator.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-RawJsonBlock` | `tools\devdocs-generator.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-WinUtilEditionIdFromName` | `functions\private\Invoke-WinUtilISO.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-WinUtilFunctionText` | `pester\win11creator.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-WinUtilToggleStatus` | `pester\preferences-theme.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-WinUtilVariables` | `functions\private\Get-WinUtilVariables.ps1` | READ_MAYBE_WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `New-WinUtilFossBadge` | `functions\private\New-WinUtilFossBadge.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Remove-WinUtilTempScript` | `scripts\main.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Reset-WPFCheckBoxes` | `pester\lazy-tabs.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Save-WinUtilFile` | `functions\private\Save-WinUtilFile.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Update-Progress` | `tools\devdocs-generator.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `netsh` | `pester\dns.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `secedit` | `pester\update-profiles.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |

### REGISTRY — 6 functions

| Function | File | Intent | Privilege → Runtime | Risk | Candidacy |
|---|---|---|---|---|---|
| `Get-WinUtilRegistryComboState` | `functions\private\Get-WinUtilRegistryComboState.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-WinUtilRegistryComboValue` | `functions\private\Get-WinUtilRegistryComboValue.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Set-WinUtilRegistry` | `functions\private\Set-WinUtilRegistry.ps1` | WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `Set-WinUtilRegistry` | `pester\multiplane-overlay.Tests.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Set-WinUtilRegistry` | `pester\tweaks.Tests.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Set-WinUtilRegistryComboState` | `functions\private\Set-WinUtilRegistryComboState.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |

### SCHEDULED_TASK — 6 functions

| Function | File | Intent | Privilege → Runtime | Risk | Candidacy |
|---|---|---|---|---|---|
| `Disable-ScheduledTask` | `pester\update-profiles.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Enable-ScheduledTask` | `pester\update-profiles.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Get-ScheduledTask` | `pester\update-profiles.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Initialize-WinUtilTaskbarOverlayAssets` | `functions\private\Initialize-WinUtilTaskbarOverlayAssets.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Set-WinUtilTaskbaritem` | `functions\private\Set-WinUtilTaskbarItem.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Set-WinUtilTaskbaritem` | `pester\ui-state.Tests.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |

### SERVICE — 5 functions

| Function | File | Intent | Privilege → Runtime | Risk | Candidacy |
|---|---|---|---|---|---|
| `Restart-Service` | `pester\ssh-server.Tests.ps1` | WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `Set-Service` | `pester\ssh-server.Tests.ps1` | WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `Set-WinUtilService` | `functions\private\Set-WinUtilService.ps1` | WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `Set-WinUtilService` | `pester\tweaks.Tests.ps1` | WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `Start-Service` | `pester\ssh-server.Tests.ps1` | WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |

### SYSTEM_REPAIR — 2 functions

| Function | File | Intent | Privilege → Runtime | Risk | Candidacy |
|---|---|---|---|---|---|
| `Invoke-WPFFixesNTPPool` | `functions\public\Invoke-WPFFixesNTPPool.ps1` | WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `Invoke-WPFSystemRepair` | `functions\public\Invoke-WPFSystemRepair.ps1` | WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |

### TWEAK — 9 functions

| Function | File | Intent | Privilege → Runtime | Risk | Candidacy |
|---|---|---|---|---|---|
| `Get-WinUtilToggleStatus` | `functions\private\Get-WinUtilToggleStatus.ps1` | READ_MAYBE_WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `Invoke-WPFExportEnvironmentReport` | `functions\public\Invoke-WPFExportEnvironmentReport.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WPFOOSU` | `functions\public\Invoke-WPFOOSU.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Invoke-WinUtilISOCleanAndReset` | `functions\private\Invoke-WinUtilISO.ps1` | WRITE | ELEVATED_ADMIN → ELEVATED_ADMIN | LOW | REQUIRES_DEDICATED_SECURITY_GATE |
| `Set-WinUtilTweaksProgressIndicator` | `pester\appx.Tests.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Set-WinUtilTweaksProgressIndicator` | `pester\install-workflow.Tests.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Set-WinUtilTweaksProgressIndicator` | `pester\oosu.Tests.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Set-WinUtilTweaksProgressIndicator` | `pester\tweaks.Tests.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Set-WinUtilTweaksProgressIndicator` | `pester\ui-state.Tests.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |

### UI — 9 functions

| Function | File | Intent | Privilege → Runtime | Risk | Candidacy |
|---|---|---|---|---|---|
| `Invoke-WinutilThemeChange` | `functions\private\Invoke-WinutilThemeChange.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Set-ThemeResourceProperty` | `functions\private\Invoke-WinutilThemeChange.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Set-WinutilTheme` | `functions\private\Invoke-WinutilThemeChange.ps1` | WRITE | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Show-WinUtilMessage` | `functions\private\Show-WinUtilMessage.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Show-WinUtilMessage` | `pester\appx.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Show-WinUtilMessage` | `pester\install-workflow.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Show-WinUtilMessage` | `pester\oosu.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Show-WinUtilMessage` | `pester\ui-state.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |
| `Show-WinUtilMessage` | `pester\update-profiles.Tests.ps1` | READ | USER_CONTEXT → USER_CONTEXT | LOW | SAFE_FOR_READ_ONLY_MCP |

