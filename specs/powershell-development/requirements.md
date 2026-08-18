# PowerShell Development Requirements

## Functional Requirements

### FR-1: Script Structure
- MUST begin with `#Requires` statements for module dependencies
- MUST include comment-based help inside the function (`.SYNOPSIS`, `.DESCRIPTION`, `.PARAMETER`, `.EXAMPLE`)
- MUST use `[CmdletBinding()]` and `param()` block
- MUST use `Set-StrictMode -Version Latest` in function body or script scope

### FR-2: Parameter Design
- MUST use PascalCase for parameter names
- MUST use full parameter names matching PowerShell conventions (`$ComputerName` not `$Server`)
- MUST use validation attributes (`[ValidateNotNullOrEmpty()]`, `[ValidateSet()]`, `[ValidateRange()]`, etc.)
- MUST use `[Alias()]` for convenience aliases (e.g., `[Alias('CN','Name')]`)
- MUST declare `[OutputType()]` when function returns objects
- SHOULD use `[Parameter(Mandatory)]` over manual null checks
- SHOULD support `[Parameter(ValueFromPipeline)]` where appropriate

### FR-3: State-Changing Operations
- MUST use `[CmdletBinding(SupportsShouldProcess)]` for any function that modifies state
- MUST wrap destructive operations in `if ($PSCmdlet.ShouldProcess(...))` blocks
- MUST capture before/after state for audit logging
- MUST use change log pattern: `[PSCustomObject]@{ Target; Property; Before; After; Timestamp; Status }`

### FR-4: Output & Collections
- MUST use `[System.Collections.Generic.List[PSCustomObject]]::new()` for building result collections (never `+=` on arrays)
- MUST NOT use `Format-*` cmdlets inside scripts that output to pipeline
- MUST export results with `Export-Csv -NoTypeInformation -Force` when CSV output requested

### FR-5: Defensive Coding
- MUST validate output directories exist before writing — create if missing
- MUST check module availability with `#Requires -Modules`
- MUST verify active connections before dependent cmdlets (e.g., vCenter, AD)
- MUST validate input files/paths exist before reading
- MUST fail fast with clear error messages when prerequisites not met

### FR-6: Documentation
- MUST include at minimum `.SYNOPSIS` and one `.EXAMPLE` per major use case
- MUST use RFC 5737 IPs (`192.0.2.x`) and `example.com` domains in examples
- MUST use Star Wars character names in examples where appropriate
- MUST explain **why** in comments, not what — code should be self-explanatory

## Non-Functional Requirements

### NFR-1: Code Layout
- MUST use One True Brace Style (OTBS)
- MUST indent with 4 spaces (not tabs)
- MUST limit lines to 115 characters — use splatting for long cmdlet calls
- MUST NOT use backtick line continuations — use splatting instead
- MUST surround function definitions with two blank lines
- MUST end files with single blank line

### NFR-2: Naming
- MUST use `Verb-Noun` PascalCase for functions with only approved verbs
- MUST use full command names — no aliases (`Get-Process` not `gps`)
- MUST use full parameter names — no positional
- PowerShell keywords lowercase (`foreach`, `if`, `param`)
- Operators lowercase (`-eq`, `-match`, `-gt`)

### NFR-3: Performance
- SHOULD prefer `foreach` construct over `ForEach-Object` for large datasets
- SHOULD use splatting for cmdlets with 3+ parameters
- SHOULD prefer `Get-CimInstance` over `Get-WmiObject`
- SHOULD measure with `Measure-Command` when performance matters

### NFR-4: Security
- MUST use `[PSCredential]` for credentials — never plain strings
- MUST accept credentials as parameters (don't call `Get-Credential` inside functions)
- MUST use `[System.Management.Automation.Credential()]` attribute on credential params
- MUST NOT store secrets in variables longer than needed

### NFR-5: Testing
- SHOULD include Pester tests for parameter validation and output structure
- SHOULD test with `-WhatIf` for state-changing functions
- SHOULD validate error paths produce meaningful messages
