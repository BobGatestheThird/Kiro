---
inclusion: manual
---

# Skill: New-PesterTest

Generate a Pester 5+ test file for an existing PowerShell function, following workspace testing standards.

## Procedure

1. **Identify the target** — ask the user for:
   - Path to the function/script to test
   - Specific scenarios to cover (happy path, error cases, edge cases)
   - Any external dependencies that need mocking (APIs, AD, VMware, file system)

2. **Read the target function** — analyze:
   - Parameter names, types, and validation attributes
   - External cmdlets called (candidates for mocking)
   - Output type and structure
   - Error handling paths
   - Whether it uses `ShouldProcess`

3. **Generate test file** at `tests/Verb-Noun.Tests.ps1` with this structure:

```powershell
#Requires -Modules Pester

BeforeAll {
    # Dot-source the function under test
    . $PSScriptRoot\..\Verb-Noun.ps1
}

Describe 'Verb-Noun' {
    BeforeAll {
        # Shared test fixtures — complex objects, reusable data
        $MockServerList = @('server01', 'server02', 'server03')
    }

    Context 'Parameter validation' {
        It 'Should have mandatory ComputerName parameter' {
            (Get-Command Verb-Noun).Parameters['ComputerName'].Attributes |
                Where-Object { $_ -is [System.Management.Automation.ParameterAttribute] } |
                Select-Object -ExpandProperty Mandatory |
                Should -Be $true
        }

        It 'Should reject empty ComputerName' {
            { Verb-Noun -ComputerName '' } | Should -Throw
        }

        It 'Should reject null ComputerName' {
            { Verb-Noun -ComputerName $null } | Should -Throw
        }
    }

    Context 'Successful execution' {
        BeforeAll {
            # Mock external dependencies — NEVER make live network calls
            Mock Invoke-Command {
                [PSCustomObject]@{
                    ComputerName = $ComputerName
                    Status       = 'Online'
                }
            }

            Mock Get-CimInstance {
                [PSCustomObject]@{
                    CSName          = 'server01'
                    LastBootUpTime  = (Get-Date).AddDays(-5)
                }
            } -ParameterFilter { $ClassName -eq 'Win32_OperatingSystem' }

            $Result = Verb-Noun -ComputerName 'server01'
        }

        It 'Should return PSCustomObject' {
            $Result | Should -BeOfType [PSCustomObject]
        }

        It 'Should include ComputerName property' {
            $Result.ComputerName | Should -Be 'server01'
        }

        It 'Should call Invoke-Command exactly once' {
            Should -Invoke Invoke-Command -Times 1 -Exactly
        }
    }

    Context 'Pipeline input' {
        BeforeAll {
            Mock Invoke-Command {
                [PSCustomObject]@{ ComputerName = $ComputerName; Status = 'Online' }
            }

            $Results = $MockServerList | Verb-Noun
        }

        It 'Should process all pipeline items' {
            $Results.Count | Should -Be 3
        }
    }

    Context 'Error handling' {
        BeforeAll {
            Mock Invoke-Command { throw 'Connection refused' }
        }

        It 'Should write error for unreachable server' {
            { Verb-Noun -ComputerName 'badserver' -ErrorAction Stop } |
                Should -Throw
        }
    }

    Context 'WhatIf support' -Skip:(-not (Get-Command Verb-Noun).Parameters.ContainsKey('WhatIf')) {
        BeforeAll {
            Mock Set-Thing {}  # mock the state-changing cmdlet
        }

        It 'Should not execute changes with -WhatIf' {
            Verb-Noun -ComputerName 'server01' -WhatIf
            Should -Invoke Set-Thing -Times 0 -Exactly
        }
    }
}
```

4. **Mock strategy** — apply these rules:
   - **Always mock**: `Invoke-Command`, `Invoke-RestMethod`, `Invoke-WebRequest`, `Connect-VIServer`, `Get-VM`, `Get-ADUser`, `Get-ADComputer`, `Send-MailMessage`
   - **Use `-ParameterFilter`** to scope mocks to specific parameter combinations
   - **Return realistic objects** — match the shape of real cmdlet output
   - **Mock failures** in separate Context blocks — don't mix success/failure mocks
   - **Never** let a test make a live network call, AD query, or vCenter connection

5. **Assertions to include per scenario:**
   - Output type matches `[OutputType()]`
   - Required properties exist and have expected values
   - Mock invocation counts (`Should -Invoke -Times`)
   - Error cases throw or write errors appropriately
   - Pipeline input processes all items

6. **File organization:**
   - One `Describe` per function
   - Separate `Context` for: parameter validation, success path, pipeline, errors, WhatIf
   - `BeforeAll` for setup, never bare code in Describe/Context
   - No `AfterAll` cleanup unless test creates temp files

7. **Final checks:**
   - No Pester 4 syntax (`Should Be` without `-`)
   - No live network calls anywhere
   - All mocks use `-ParameterFilter` when cmdlet is called multiple ways
   - Test file name matches `Verb-Noun.Tests.ps1` convention
   - Relative dot-source path is correct
