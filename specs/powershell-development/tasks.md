# PowerShell Development Tasks

Checklist for developing a new PowerShell script, advanced function, or module.

## Phase 1: Design

- [ ] Define purpose — what problem does this solve?
- [ ] Choose type: standalone script | advanced function | module
- [ ] Identify inputs (parameters, pipeline, files)
- [ ] Identify outputs (pipeline objects, CSV, global variable)
- [ ] Identify dependencies (`#Requires -Modules`, connections)
- [ ] Determine if state-changing (needs `ShouldProcess`)

## Phase 2: Scaffold

- [ ] Create file with correct `Verb-Noun` naming
- [ ] Add `#Requires` statements at top
- [ ] Add `[CmdletBinding()]` with appropriate attributes
- [ ] Define `param()` block with validation attributes
- [ ] Add `[OutputType()]` declaration
- [ ] Add comment-based help skeleton (`.SYNOPSIS`, `.DESCRIPTION`, `.PARAMETER`, `.EXAMPLE`)
- [ ] Add `Set-StrictMode -Version Latest`

## Phase 3: Implement

- [ ] Add prerequisite checks (connection validation, path validation)
- [ ] Implement core logic in `begin/process/end` or sequential flow
- [ ] Use splatting for cmdlets with 3+ parameters
- [ ] Use `[System.Collections.Generic.List[PSCustomObject]]::new()` for collections
- [ ] Wrap state changes in `ShouldProcess` blocks
- [ ] Add before/after change logging if modifying state
- [ ] Add `Write-Verbose` for progress/status messages
- [ ] Add `Write-Warning` for non-fatal issues
- [ ] Handle errors with `try/catch` and `-ErrorAction Stop`
- [ ] Output `[PSCustomObject]` — never raw strings or Format data

## Phase 4: Export & Output

- [ ] Add `-ExportCsv` parameter if CSV output needed
- [ ] Validate output directory exists, create if missing
- [ ] Use `Export-Csv -NoTypeInformation -Force`
- [ ] If session storage needed: use `$global:VarName` as Generic.List, suppress pipeline output

## Phase 5: Documentation

- [ ] Complete all `.EXAMPLE` blocks with realistic scenarios
- [ ] Use RFC 5737 IPs and `example.com` in examples
- [ ] Use Star Wars names for sample data
- [ ] Add `.NOTES` section with prerequisites and caveats
- [ ] Create companion `docs/Verb-Noun.md` if complex

## Phase 6: Review & Test

- [ ] Run script with `-Verbose` to verify messaging
- [ ] Run state-changing functions with `-WhatIf`
- [ ] Verify output captures correctly: `$results = .\Script.ps1`
- [ ] Verify no `FormatStartData` in captured output
- [ ] Check `Get-Help .\Script.ps1 -Full` renders correctly
- [ ] Create Pester test file if reusable function (`tests/Verb-Noun.Tests.ps1`)
- [ ] Test parameter validation (empty, null, invalid input)
- [ ] Test error paths produce meaningful messages

## Phase 7: Finalize

- [ ] Remove any debugging `Write-Host` statements
- [ ] Ensure no hardcoded IPs/hostnames in function body
- [ ] Verify file ends with single blank line
- [ ] Verify no trailing whitespace
- [ ] Commit with descriptive message
