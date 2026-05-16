---
name: Script Templates
description: This skill should be used when generating PowerShell scripts to ensure they follow project standards, include proper headers, logging, error handling, and Windows best practices.
version: 1.0.0
---

# Script Templates Skill

## Overview

Standard templates and patterns for PowerShell script generation in the Central Agent system.

## PowerShell Script Template

### Complete Production Template

```powershell
#Requires -Version 5.1
<#
.SYNOPSIS
    {SYNOPSIS - One line description}

.DESCRIPTION
    {DESCRIPTION - Detailed explanation of what the script does}

.PARAMETER {ParamName}
    {Description of the parameter}

.EXAMPLE
    .\{ScriptName}.ps1
    {Description of what this example does}

.EXAMPLE
    .\{ScriptName}.ps1 -Verbose
    {Description of verbose execution}

.NOTES
    Author: Central Agent
    Created: {YYYY-MM-DD}
    Modified: {YYYY-MM-DD}
    Version: 1.0.0

    Change Log:
    1.0.0 - Initial release
#>

[CmdletBinding(SupportsShouldProcess, ConfirmImpact = 'Medium')]
param(
    [Parameter(Mandatory = $false, HelpMessage = "Path for log files")]
    [ValidateScript({ Test-Path $_ -PathType Container })]
    [string]$LogPath = "%HOME%\Projects\scripts\maintenance\logs",

    [Parameter(Mandatory = $false, HelpMessage = "Enable verbose logging")]
    [switch]$VerboseLog
)

#region Configuration
$ErrorActionPreference = "Stop"
$ProgressPreference = "SilentlyContinue"

$Script:Config = @{
    ScriptName = $MyInvocation.MyCommand.Name
    ScriptPath = $PSScriptRoot
    LogPath    = $LogPath
    LogFile    = Join-Path $LogPath "$(Get-Date -Format 'yyyy-MM-dd')-$($MyInvocation.MyCommand.Name -replace '\.ps1$','').log"
    StartTime  = Get-Date
}
#endregion

#region Logging Functions
function Write-Log {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory, Position = 0)]
        [string]$Message,

        [Parameter()]
        [ValidateSet('INFO', 'WARN', 'ERROR', 'DEBUG', 'SUCCESS')]
        [string]$Level = 'INFO',

        [Parameter()]
        [switch]$NoConsole
    )

    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss.fff"
    $logEntry = "[$timestamp] [$Level] $Message"

    # Console output with colors
    if (-not $NoConsole) {
        $color = switch ($Level) {
            'ERROR'   { 'Red' }
            'WARN'    { 'Yellow' }
            'SUCCESS' { 'Green' }
            'DEBUG'   { 'Gray' }
            default   { 'White' }
        }
        Write-Host $logEntry -ForegroundColor $color
    }

    # File output
    if (-not (Test-Path (Split-Path $Script:Config.LogFile -Parent))) {
        New-Item -ItemType Directory -Path (Split-Path $Script:Config.LogFile -Parent) -Force | Out-Null
    }
    Add-Content -Path $Script:Config.LogFile -Value $logEntry -Encoding UTF8
}

function Write-ErrorLog {
    param([string]$Message, [System.Management.Automation.ErrorRecord]$ErrorRecord)

    Write-Log $Message -Level ERROR
    if ($ErrorRecord) {
        Write-Log "Exception: $($ErrorRecord.Exception.Message)" -Level ERROR
        Write-Log "Stack Trace: $($ErrorRecord.ScriptStackTrace)" -Level DEBUG
    }
}
#endregion

#region Lifecycle Functions
function Initialize-Script {
    Write-Log "=" * 60 -NoConsole
    Write-Log "Script: $($Script:Config.ScriptName)" -Level INFO
    Write-Log "Started: $($Script:Config.StartTime)" -Level INFO
    Write-Log "User: $env:USERNAME" -Level INFO
    Write-Log "Computer: $env:COMPUTERNAME" -Level INFO
    Write-Log "PowerShell: $($PSVersionTable.PSVersion)" -Level INFO
    Write-Log "=" * 60 -NoConsole
}

function Complete-Script {
    param(
        [int]$ExitCode = 0,
        [string]$Message = "Script completed"
    )

    $duration = (Get-Date) - $Script:Config.StartTime
    $level = if ($ExitCode -eq 0) { 'SUCCESS' } else { 'ERROR' }

    Write-Log "=" * 60 -NoConsole
    Write-Log "$Message" -Level $level
    Write-Log "Duration: $($duration.ToString('hh\:mm\:ss\.fff'))" -Level INFO
    Write-Log "Exit Code: $ExitCode" -Level INFO
    Write-Log "=" * 60 -NoConsole

    exit $ExitCode
}
#endregion

#region Helper Functions
function Test-AdminRights {
    $currentPrincipal = New-Object Security.Principal.WindowsPrincipal([Security.Principal.WindowsIdentity]::GetCurrent())
    return $currentPrincipal.IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
}

function Get-FormattedSize {
    param([long]$Bytes)

    switch ($Bytes) {
        { $_ -ge 1TB } { return "{0:N2} TB" -f ($_ / 1TB) }
        { $_ -ge 1GB } { return "{0:N2} GB" -f ($_ / 1GB) }
        { $_ -ge 1MB } { return "{0:N2} MB" -f ($_ / 1MB) }
        { $_ -ge 1KB } { return "{0:N2} KB" -f ($_ / 1KB) }
        default { return "$_ Bytes" }
    }
}

function Confirm-Action {
    param(
        [string]$Action,
        [string]$Target
    )

    if ($PSCmdlet.ShouldProcess($Target, $Action)) {
        return $true
    }
    return $false
}
#endregion

#region Main Logic
function Invoke-MainLogic {
    [CmdletBinding()]
    param()

    # ============================================
    # YOUR MAIN SCRIPT LOGIC GOES HERE
    # ============================================

    {MAIN_LOGIC}

    # ============================================
}
#endregion

#region Execution
try {
    Initialize-Script
    Invoke-MainLogic
    Complete-Script -ExitCode 0 -Message "Script completed successfully"
}
catch {
    Write-ErrorLog -Message "Fatal error occurred" -ErrorRecord $_
    Complete-Script -ExitCode 1 -Message "Script failed with errors"
}
#endregion
```

## Common Script Patterns

### File Cleanup Pattern

```powershell
function Remove-OldFiles {
    [CmdletBinding(SupportsShouldProcess)]
    param(
        [string]$Path,
        [int]$RetentionDays = 30,
        [string[]]$Extensions = @('*')
    )

    $cutoffDate = (Get-Date).AddDays(-$RetentionDays)
    $removedCount = 0
    $removedSize = 0

    foreach ($ext in $Extensions) {
        Get-ChildItem -Path $Path -Filter $ext -Recurse -File -ErrorAction SilentlyContinue |
            Where-Object { $_.LastWriteTime -lt $cutoffDate } |
            ForEach-Object {
                if ($PSCmdlet.ShouldProcess($_.FullName, "Remove")) {
                    $removedSize += $_.Length
                    Remove-Item $_.FullName -Force
                    $removedCount++
                    Write-Log "Removed: $($_.FullName)"
                }
            }
    }

    Write-Log "Removed $removedCount files ($(Get-FormattedSize $removedSize))" -Level SUCCESS
}
```

### File Organization Pattern

```powershell
function Move-FilesByType {
    [CmdletBinding(SupportsShouldProcess)]
    param(
        [string]$SourcePath,
        [string]$DestinationBase,
        [hashtable]$Rules
    )

    $movedCount = 0

    foreach ($pattern in $Rules.Keys) {
        $destination = Join-Path $DestinationBase $Rules[$pattern]

        if (-not (Test-Path $destination)) {
            New-Item -ItemType Directory -Path $destination -Force | Out-Null
        }

        Get-ChildItem -Path $SourcePath -Filter $pattern -File |
            ForEach-Object {
                $targetPath = Join-Path $destination $_.Name

                if (Test-Path $targetPath) {
                    $baseName = [System.IO.Path]::GetFileNameWithoutExtension($_.Name)
                    $extension = [System.IO.Path]::GetExtension($_.Name)
                    $counter = 1
                    do {
                        $targetPath = Join-Path $destination "${baseName}_${counter}${extension}"
                        $counter++
                    } while (Test-Path $targetPath)
                }

                if ($PSCmdlet.ShouldProcess($_.FullName, "Move to $targetPath")) {
                    Move-Item $_.FullName -Destination $targetPath
                    $movedCount++
                    Write-Log "Moved: $($_.Name) -> $destination"
                }
            }
    }

    Write-Log "Moved $movedCount files" -Level SUCCESS
}
```

### Directory Status Pattern

```powershell
function Get-DirectoryStatus {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$Path
    )

    if (-not (Test-Path $Path)) {
        return [PSCustomObject]@{
            Path = $Path
            Exists = $false
        }
    }

    $items = Get-ChildItem -Path $Path -Recurse -File -ErrorAction SilentlyContinue

    return [PSCustomObject]@{
        Path = $Path
        Exists = $true
        FileCount = $items.Count
        TotalSize = ($items | Measure-Object -Property Length -Sum).Sum
        TotalSizeFormatted = Get-FormattedSize (($items | Measure-Object -Property Length -Sum).Sum)
        LastModified = ($items | Sort-Object LastWriteTime -Descending | Select-Object -First 1).LastWriteTime
        NewToday = ($items | Where-Object { $_.CreationTime.Date -eq (Get-Date).Date }).Count
        ModifiedToday = ($items | Where-Object { $_.LastWriteTime.Date -eq (Get-Date).Date }).Count
    }
}
```

### Backup Pattern

```powershell
function Backup-Directory {
    [CmdletBinding(SupportsShouldProcess)]
    param(
        [Parameter(Mandatory)]
        [string]$SourcePath,

        [Parameter(Mandatory)]
        [string]$BackupPath,

        [switch]$Compress
    )

    $timestamp = Get-Date -Format "yyyy-MM-dd_HHmmss"
    $backupName = "$(Split-Path $SourcePath -Leaf)_$timestamp"

    if ($Compress) {
        $archivePath = Join-Path $BackupPath "$backupName.zip"

        if ($PSCmdlet.ShouldProcess($SourcePath, "Compress to $archivePath")) {
            Compress-Archive -Path $SourcePath -DestinationPath $archivePath -CompressionLevel Optimal
            Write-Log "Created backup archive: $archivePath" -Level SUCCESS
        }
    }
    else {
        $targetPath = Join-Path $BackupPath $backupName

        if ($PSCmdlet.ShouldProcess($SourcePath, "Copy to $targetPath")) {
            Copy-Item -Path $SourcePath -Destination $targetPath -Recurse
            Write-Log "Created backup directory: $targetPath" -Level SUCCESS
        }
    }
}
```

## Naming Conventions

### Script Names
Use **Verb-Noun** format with approved PowerShell verbs:

| Verb | Usage |
|------|-------|
| Get | Retrieve information |
| Set | Modify existing data |
| New | Create new items |
| Remove | Delete items |
| Move | Relocate items |
| Copy | Duplicate items |
| Clear | Empty containers |
| Backup | Create backups |
| Restore | Recover from backup |
| Sync | Synchronize data |
| Test | Validate conditions |
| Invoke | Execute actions |

### Examples
- `Clean-OldLogs.ps1`
- `Organize-Downloads.ps1`
- `Backup-ConfigFiles.ps1`
- `Sync-ProjectFolders.ps1`
- `Get-DirectoryStatus.ps1`

## Quality Checklist

Before finalizing any script, verify:

- [ ] Uses standard template structure
- [ ] Has complete comment-based help (.SYNOPSIS, .DESCRIPTION, etc.)
- [ ] All parameters have validation attributes
- [ ] Uses Write-Log for all operations
- [ ] Implements proper error handling with try/catch
- [ ] Uses SupportsShouldProcess for destructive operations
- [ ] Includes Initialize-Script and Complete-Script lifecycle
- [ ] Has usage examples in help
- [ ] Follows Verb-Noun naming convention
- [ ] No hardcoded paths (use parameters)
- [ ] Logs to standard location
