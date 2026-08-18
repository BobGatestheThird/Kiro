---
inclusion: manual
---

# Skill: New-ServerReport

Scaffold a PowerShell script that collects data from remote servers and exports a structured report (CSV/Excel).

## Procedure

1. **Gather requirements** — ask the user for:
   - What data to collect (services, software, disk, network, IIS, SQL, etc.)
   - Server list source (txt file, AD query, parameter, clipboard)
   - Output format (CSV default, Excel if ImportExcel available)
   - Remote method (Invoke-Command/WinRM, CIM session, direct cmdlet)
   - Credential handling (current user, parameter, SecretManagement)
   - Error tolerance (skip failures and continue, or halt?)

2. **Generate the report script** with this structure:

```powershell
#Requires -Modules <ModuleName>  # ImportExcel if Excel output needed

<#
.SYNOPSIS
    Collects <DataType> from remote servers and exports to CSV.

.DESCRIPTION
    Queries a list of servers via WinRM, collects <DataType> information,
    and exports results to a timestamped CSV file. Failures are logged
    separately without halting execution.

.PARAMETER ComputerName
    One or more server names to query. Accepts pipeline input.

.PARAMETER InputFile
    Path to a text file containing one server name per line.

.PARAMETER OutputDir
    Directory for report output. Defaults to script directory.

.PARAMETER Credential
    PSCredential for remote connections. Uses current user if omitted.

.EXAMPLE
    .\Get-ServerReport.ps1 -InputFile .\servers.txt
    Collects data from all servers in the file.

.EXAMPLE
    Get-Content .\servers.txt | .\Get-ServerReport.ps1 -OutputDir C:\Reports
    Pipeline input with custom output directory.
#>
[CmdletBinding()]
param(
    [Parameter(Mandatory, ValueFromPipeline, ParameterSetName = 'Direct')]
    [ValidateNotNullOrEmpty()]
    [string[]]$ComputerName,

    [Parameter(Mandatory, ParameterSetName = 'File')]
    [ValidateScript({ Test-Path $_ -PathType Leaf })]
    [string]$InputFile,

    [Parameter()]
    [string]$OutputDir = $PSScriptRoot,

    [Parameter()]
    [System.Management.Automation.PSCredential]
    [System.Management.Automation.Credential()]
    $Credential
)

begin {
    Set-StrictMode -Version Latest

    # Validate output directory
    if (-not (Test-Path $OutputDir)) {
        New-Item -ItemType Directory -Path $OutputDir -Force | Out-Null
    }

    # Initialize collections
    $Results = [System.Collections.Generic.List[PSCustomObject]]::new()
    $Errors = [System.Collections.Generic.List[PSCustomObject]]::new()

    # Build file paths
    $Timestamp = Get-Date -Format 'yyyy-MM-dd_HHmmss'
    $ReportFile = Join-Path $OutputDir "ServerReport_$Timestamp.csv"
    $ErrorFile = Join-Path $OutputDir "ServerReport_Errors_$Timestamp.csv"
    $HeaderWritten = $false

    # Resolve server list from file if needed
    if ($PSCmdlet.ParameterSetName -eq 'File') {
        $ComputerName = Get-Content -Path $InputFile |
            Where-Object { $_ -and $_.Trim() -ne '' }
        Write-Verbose "Loaded $($ComputerName.Count) servers from '$InputFile'"
    }

    # Build credential splat
    $CredSplat = @{}
    if ($Credential) {
        $CredSplat['Credential'] = $Credential
    }

    $TotalCount = 0
    $SuccessCount = 0
    $FailCount = 0
}

process {
    foreach ($Computer in $ComputerName) {
        $TotalCount++
        Write-Progress -Activity 'Collecting server data' -Status $Computer -PercentComplete (
            ($TotalCount / $ComputerName.Count) * 100
        )

        try {
            # Test connectivity first
            if (-not (Test-Connection -ComputerName $Computer -Count 1 -Quiet)) {
                throw "Server unreachable (ping failed)"
            }

            # Remote data collection
            $Data = Invoke-Command -ComputerName $Computer @CredSplat -ErrorAction Stop -ScriptBlock {
                # Collect data inside remote session
                [PSCustomObject]@{
                    Hostname     = $env:COMPUTERNAME
                    # Add collected properties here
                }
            }

            $Result = [PSCustomObject]@{
                ComputerName = $Computer
                Status       = 'Success'
                # Map remote data to report columns
                CollectedAt  = Get-Date -Format 'yyyy-MM-dd HH:mm:ss'
            }

            $Results.Add($Result)
            $SuccessCount++

            # Append to CSV immediately — preserves progress on crash
            if (-not $HeaderWritten) {
                $Result | Export-Csv -Path $ReportFile -NoTypeInformation -Force
                $HeaderWritten = $true
            } else {
                $Result | Export-Csv -Path $ReportFile -NoTypeInformation -Append
            }

            $Result  # emit to pipeline

        }
        catch {
            $ErrorRecord = $_
            $FailCount++

            $ErrObj = [PSCustomObject]@{
                ComputerName = $Computer
                Error        = $ErrorRecord.Exception.Message
                Timestamp    = Get-Date -Format 'yyyy-MM-dd HH:mm:ss'
            }
            $Errors.Add($ErrObj)

            Write-Warning "$Computer : $($ErrorRecord.Exception.Message)"
        }
    }
}

end {
    Write-Progress -Activity 'Collecting server data' -Completed

    # Export error log if any failures
    if ($Errors.Count -gt 0) {
        $Errors | Export-Csv -Path $ErrorFile -NoTypeInformation -Force
        Write-Warning "$FailCount of $TotalCount servers failed. Error log: $ErrorFile"
    }

    Write-Verbose "Report complete: $SuccessCount success, $FailCount failed, $TotalCount total"
    Write-Verbose "Report saved: $ReportFile"
}
```

3. **Customize the remote scriptblock** based on data type requested:

   | Data Type | Cmdlets to use |
   |-----------|---------------|
   | System info | `Get-CimInstance Win32_OperatingSystem`, `Win32_ComputerSystem` |
   | Disk | `Get-CimInstance Win32_LogicalDisk` |
   | Services | `Get-Service`, `Get-CimInstance Win32_Service` |
   | Software | `Get-CimInstance Win32_Product` or registry query |
   | Network | `Get-NetIPConfiguration`, `Get-DnsClientServerAddress` |
   | IIS | `Get-IISSite`, `Get-WebBinding` (requires WebAdministration) |
   | SQL | `Get-Service *SQL*`, `Invoke-Sqlcmd` |
   | Certificates | `Get-ChildItem Cert:\LocalMachine\My` |
   | Uptime | `Get-CimInstance Win32_OperatingSystem | Select LastBootUpTime` |

4. **Output format options:**
   - CSV (default): `Export-Csv -NoTypeInformation`
   - Excel: `Export-Excel` with `-AutoSize -FreezeTopRow -BoldTopRow` (gate with `#Requires -Modules ImportExcel`)
   - HTML: `ConvertTo-Html` with CSS styling for email-ready reports
   - Console: `Out-GridView` for interactive review (mention in example)

5. **Final checks:**
   - `Test-Connection` before remote calls (or catch and log)
   - CSV append pattern for crash-safety
   - Separate error log file
   - `Write-Progress` for long-running collection
   - No `Get-WmiObject` — use `Get-CimInstance`
   - Credential parameter never calls `Get-Credential` internally
   - `$PSScriptRoot`-based paths, not relative
   - Pipeline support for server names
