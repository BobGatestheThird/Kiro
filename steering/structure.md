# Project Structure

## Layout

This is a flat-structured script repository. Most PowerShell scripts live in the root directory. Subdirectories group related tools or data.

```
/                           # Root — standalone PowerShell scripts (majority of codebase)
├── ansible-hw-upgrade/     # Ansible playbooks for VM hardware version upgrades
│   └── group_vars/         # Ansible variables (vCenter connection, policies)
├── Inventory/              # Server forensic scan exports (CSV per server)
│   └── <ServerName>/       # Per-server folder with standardized CSVs
├── infoblox-mcp-server/    # TypeScript MCP server for Infoblox NIOS
│   └── src/                # TypeScript source
├── harden/                 # Server hardening scripts
├── decoms/                 # Server decommission data
├── splunk/                 # Splunk-related configs
├── csv/                    # CSV data files
├── reports/                # Generated reports
└── .kiro/                  # Kiro IDE configuration
```

## Naming Conventions

- Scripts use `Verb-Noun` PowerShell convention (e.g., `Get-ServerForensics.ps1`, `Set-EsxSyslog.ps1`)
- Use a prefix on scripts to indicate frequently-used or priority scripts (e.g., `AAAget-accountLockOut.ps1`)
- Use an org-specific prefix for organization scripts
- Use a deprecation prefix for deprecated or experimental scripts (e.g., `xxx`)
- Function scripts wrap logic in named functions; utility scripts run directly
- Server inventory folders use the server hostname as the directory name

## Script Patterns

- Well-structured scripts include comment-based help (`.SYNOPSIS`, `.DESCRIPTION`, `.PARAMETER`, `.EXAMPLE`)
- Advanced functions use `[CmdletBinding()]`, `param()` blocks, and `SupportsShouldProcess`
- Quick utility scripts may be minimal with no formal function wrapper
- Remote execution uses `Invoke-Command` with WinRM
- Server lists are typically read from `.txt` files or passed as parameters

## Language Best Practices

### PowerShell

> Style and best practices derived from the [PoshCode PowerShell Practice and Style Guide](https://github.com/PoshCode/PowerShellPracticeAndStyle) (CC BY-SA 4.0, Don Jones, Matt Penny, Carlos Perez, Joel Bennett & community). Content rephrased for compliance with licensing restrictions.

#### Code Layout & Formatting

- Use **One True Brace Style (OTBS)**: opening brace on same line as statement, closing brace on own line
- Exception: small scriptblocks passed to parameters may be single-line (`Where-Object { $_.Length -gt 10mb }`)
- Indent with **4 spaces** (not tabs)
- Limit lines to **115 characters** — use splatting or natural line breaks (after commas, pipes, inside parens) instead of backticks
- Avoid backtick line continuations — use splatting (`@params`) instead
- Use splatting for cmdlet calls with 3+ parameters — easier to read, maintain, and avoids invisible whitespace issues after backticks
- Single space around operators and parameter names; no space inside parens/brackets
- No space before unary operators (`$i++`, `(Get-Date).AddDays(-1)`)
- Single space after commas and semicolons
- Subexpressions `$( ... )` and scriptblocks `{ ... }` get single space inside braces — distinguishes from variable delimiters `${...}`
- Surround function definitions with **two blank lines**; method definitions with one
- Additional blank lines may be used sparingly to separate groups of related functions or logical sections within functions
- No trailing whitespace; end files with single blank line
- Avoid semicolons as line terminators
- Always start with `[CmdletBinding()]` and `param()` — even for scripts
- Prefer block order: `param()`, `begin`, `process`, `end` — writing out of execution order makes code confusing
- Whitespace-only reformats go in their own commit — never mixed with logic changes
- Use PowerShell's implied line continuation inside parentheses, brackets, and braces in preference to backticks
- For long strings, use concatenation inside parentheses rather than backtick continuation

#### Capitalization

- **PascalCase** for all public identifiers: functions, parameters, modules, variables, classes, enums
- PowerShell keywords lowercase (`foreach`, `if`, `param`, `dynamicparam`)
- Operators lowercase (`-eq`, `-match`, `-gt`)
- Comment-based help keywords UPPERCASE (`.SYNOPSIS`, `.DESCRIPTION`)
- Two-letter acronyms both caps (`$PSBoundParameters`, `Get-PSDrive`) — does not extend to compound acronyms or common words like "OK" and "ID"
- Optional: camelCase for private/local variables to distinguish from parameters
- When using camelCase with a two-letter acronym prefix, both letters lowercase (`$adComputer`)
- Use scope prefix for shared variables (`$Script:VarName`, `$Global:DebugPreference`)
- For global variable assignments, use `Set-Variable -Name <Name> -Value <Value> -Scope Global` instead of `$global:VarName = ...` — explicit cmdlet form is clearer in scripts and avoids issues with scope resolution in nested calls

#### Naming

- Use full command names — no aliases (`Get-Process` not `gps`)
- Use full parameter names — no positional (`-Name Explorer` not just `Explorer`)
- Use `Verb-Noun` PascalCase for functions; only approved verbs (`Get-Verb`)
- Match standard PowerShell parameter names (`$ComputerName` not `$Param_Computer`)
- Avoid `~` for home folder — use `${Env:UserProfile}` instead
- Use `$PSScriptRoot`-based paths, not relative paths — .NET methods use `[Environment]::CurrentDirectory` which doesn't track PowerShell's `$PWD`

#### Function Structure

- Always use `[CmdletBinding()]` on advanced functions
- Avoid `return` keyword — place objects directly on pipeline in `process {}` block
- Return objects in `process {}`, not `begin {}` or `end {}` — preserves pipeline streaming
- Specify `[OutputType()]` attribute when function returns objects
- When using `ParameterSetName`, set `DefaultParameterSetName` in `CmdletBinding`
- Use parameter validation attributes over manual validation in function body:
  `[ValidateSet()]`, `[ValidateRange()]`, `[ValidatePattern()]`, `[ValidateScript()]`,
  `[ValidateNotNullOrEmpty()]`, `[ValidateCount()]`, `[ValidateLength()]`
- Use `[AllowNull()]`, `[AllowEmptyString()]`, `[AllowEmptyCollection()]` when needed on mandatory params

#### Documentation & Comments

- Comment-based help goes **inside** the function, at the top
- Include at minimum: `.SYNOPSIS`, `.EXAMPLE` for each major use case
- Document parameters inline above each param (preferred over `.PARAMETER` blocks)
- Comments explain **why**, not what — code should be self-explanatory
- Block comments for sections; inline comments separated by 2+ spaces, aligned
- Use `<# ... #>` for long documentation blocks
- Use RFC 5737 documentation IPs (`192.0.2.x`, `198.51.100.x`, `203.0.113.x`) in examples — never real internal IPs
- Use generic hostnames in examples (`esxi-example-01`, `vcenter.example.com`, `server01`) — never real environment names
- Use `example.com` / `example.local` for domain names in help text
- When creating or updating a companion `docs/Verb-Noun.md` file, it MUST include mermaid diagrams:
  - A **flowchart** showing the script's decision logic and execution path
  - A **sequence diagram** showing interactions between the script and external systems/APIs
  - Diagrams use standard mermaid syntax (```mermaid code blocks)

#### Output & Streams

- `Write-Verbose` for progress/status info
- `Write-Warning` for non-fatal issues
- `Write-Error` for failures
- `Write-Debug` for maintainer/debugging info
- `Write-Progress` for long-running operations
- Avoid `Write-Host` unless function uses `Show`/`Format` verb or interactive prompts
- Tools output raw data (bytes, not converted units); controllers may format
- Output single type per command; use `[OutputType()]` attribute
- Return structured output via `[PSCustomObject]@{}` — not raw strings
- For modules: use `.format.ps1xml` view definitions instead of `Format-*` inside functions — keeps raw data intact
- Never use `Format-*` cmdlets in tool functions — it destroys object data for downstream consumers

#### Error Handling

- Use `-ErrorAction Stop` on cmdlets to generate trappable exceptions
- Set `$ErrorActionPreference = 'Stop'` when calling non-cmdlet code
- Put entire transaction in `try` block — avoid flag variables for error state
- Avoid `$?` for error detection — unreliable
- Avoid testing null variables as error conditions when exceptions are available
- Copy `$_` or `$Error[0]` to own variable immediately in `catch` block
- Don't clear `$Error` — PowerShell manages the collection automatically

#### Performance

- Measure with `Measure-Command` when performance matters
- `foreach` construct faster than `ForEach-Object` cmdlet for large datasets
- Pipeline streaming better for memory with large files
- Use splatting and structured code over micro-optimizations
- Language features > .NET Framework > Script > Pipeline (rough speed order)
- Wrap .NET calls in PowerShell functions for readability when performance requires lower-level code

#### Security

- Always use `[PSCredential]` for credentials — never plain strings
- Accept credentials as parameters (don't call `Get-Credential` inside functions)
- Use `[System.Management.Automation.Credential()]` attribute on credential params
- Decrypt credentials inline at point of use, not into variables
- Use `SecureString` for sensitive non-credential strings
- Use `Export-CliXml` / `Import-CliXml` for persisting credentials (DPAPI-protected)

#### Reusability

- Distinguish **tools** (reusable functions, pipeline-friendly) from **controllers** (business-process scripts)
- Tools accept input via parameters, output raw data to pipeline — maximize reusability
- Controllers orchestrate tools for a specific business process — may format output for humans or log to files
- Controllers are not required to be reusable; tools are
- Modularize working code into functions in script modules
- Use native PowerShell commands before .NET/COM — document when going lower-level
- Wrap external tools/APIs in advanced functions — get shell benefits (pipeline, help, parameters) around non-native code
- Check if built-in cmdlet exists before writing custom (`Test-Connection` vs custom ping)
- Before writing a new function, verify PowerShell doesn't already have a native way to accomplish the task
- Tools should output raw data (bytes, not converted units); controllers may format for display
- For modules: use `.format.ps1xml` view definitions to format display without altering underlying data

#### Defensive Coding & Prerequisites

- Validate output directories exist before writing — create if missing: `if (-not (Test-Path $OutputDir)) { New-Item -ItemType Directory -Path $OutputDir -Force | Out-Null }`
- Check module availability with `#Requires -Modules <ModuleName>` at script top — fail at load time, not mid-execution
- Verify active connections before running dependent cmdlets (e.g., `if (-not $global:DefaultVIServer) { Write-Error "Not connected to vCenter. Run Connect-VIServer first."; return }`)
- Validate input files/paths exist before reading (`if (-not (Test-Path $InputFile)) { Write-Error "Input file '$InputFile' not found"; return }`)
- Fail fast with clear error messages when prerequisites not met — don't silently continue with null data
- Test network connectivity before remote operations when practical (`Test-Connection`, `Test-NetConnection`)

#### Change Logging & Before/After State

- For any `Set-*` or state-changing operation, capture the **current value before** making the change
- After the change completes, capture the **new value** to confirm it took effect
- Log both before and after states — at minimum via `Write-Verbose`, preferably to a structured log object or CSV
- Use a `[PSCustomObject]@{ Target; Property; Before; After; Timestamp; Status }` pattern for change records
- Store change records in a session variable (e.g., `$script:ChangeLog`) for post-run review
- For bulk changes, export the change log to CSV with timestamp in filename (e.g., `ChangeLog_2024-01-15_143022.csv`)
- **Append each record to CSV immediately** after processing — do not wait until script completion to export. This ensures progress is saved if the script terminates unexpectedly (crash, timeout, Ctrl+C). Use `Export-Csv -Append` for subsequent records after writing the header with the first record.
- Include `ShouldProcess` confirmation — the before/after log doubles as an audit trail
- Example pattern:
  ```powershell
  $LogFile = "ChangeLog_$(Get-Date -Format 'yyyy-MM-dd_HHmmss').csv"
  $HeaderWritten = $false

  $Before = $CurrentObject.PropertyValue
  if ($PSCmdlet.ShouldProcess($Target, "Set PropertyName from '$Before' to '$NewValue'")) {
      Set-Thing -Value $NewValue
      $After = (Get-Thing).PropertyValue
      $Record = [PSCustomObject]@{
          Target    = $Target
          Property  = 'PropertyName'
          Before    = $Before
          After     = $After
          Timestamp = Get-Date -Format 'yyyy-MM-dd HH:mm:ss'
          Status    = if ($After -eq $NewValue) { 'Success' } else { 'Failed' }
      }

      $script:ChangeLog.Add($Record)

      # Append to CSV immediately — preserves progress on crash
      if (-not $HeaderWritten) {
          $Record | Export-Csv -Path $LogFile -NoTypeInformation -Force
          $HeaderWritten = $true
      } else {
          $Record | Export-Csv -Path $LogFile -NoTypeInformation -Append
      }
  }
  ```

#### Project-Specific Rules

- Use `#Requires -Modules <ModuleName>` at top when script depends on external modules
- For PowerCLI scripts: use `#Requires -Modules VMware.VimAutomation.Core` — this is the actual module name installed (the umbrella `VMware.PowerCLI` module no longer ships as a single installable package in 13.4+). Do not use `VMware.PowerCLI` or `Broadcom.PowerCLI` in `#Requires` lines.
- Use `Set-StrictMode -Version Latest` in function bodies
- Prefer `[CmdletBinding(SupportsShouldProcess)]` for any function that changes state
- Use `[System.Collections.Generic.List[PSCustomObject]]::new()` for building result collections (not `+=` on arrays)
- Wrap destructive operations in `if ($PSCmdlet.ShouldProcess(...))` blocks
- Export results with `Export-Csv -NoTypeInformation -Force`
- Use `[Alias()]` on parameters for convenience (e.g., `[Alias('CN', 'Name')]`)
- Prefer `Get-CimInstance` over `Get-WmiObject` (deprecated)

#### Nested Hashtable Structures (PS7+ Best Practice)

- When storing objects for later use, prefer nested hashtables with `List[object]` values
- Hashtable keys = logical buckets (categories of data)
- Values = `[System.Collections.Generic.List[object]]::new()` — fast, mutable, no array-copy penalty
- Objects = `[PSCustomObject]` or typed classes
- Never use `+=` on arrays inside hashtable values — same O(n) copy penalty applies
- Example pattern:
  ```powershell
  $Data = @{
      Users   = [System.Collections.Generic.List[object]]::new()
      Servers = [System.Collections.Generic.List[object]]::new()
      Logs    = @{
          Errors   = [System.Collections.Generic.List[object]]::new()
          Warnings = [System.Collections.Generic.List[object]]::new()
      }
  }

  # Add objects
  $Data.Users.Add([PSCustomObject]@{ Name = 'Han Solo'; Role = 'Admin' })
  $Data.Users.Add([PSCustomObject]@{ Name = 'Leia Organa'; Role = 'Dev' })
  $Data.Logs.Errors.Add([PSCustomObject]@{ Code = 500; Message = 'Internal Error' })
  ```

### Ansible (YAML Playbooks)

- Use `---` document start marker
- Include a descriptive comment block at the top of each playbook with usage examples
- Use `community.vmware` collection modules for vSphere operations
- Store sensitive values (passwords) in Ansible Vault — reference via `{{ vault_* }}` variables
- Centralize shared variables in `group_vars/all.yml`
- Use `validate_certs: false` for lab/internal vCenter connections
- Use `register:` to capture task output, then `set_fact:` for filtering
- Use Jinja2 filters (`selectattr`, `rejectattr`, `map`) for list processing
- Add `when:` conditions to skip tasks when data is empty
- Use `loop_control: label:` to keep verbose output readable

### TypeScript (MCP Server)

- Use ES module syntax (`import`/`export`, `"type": "module"` in package.json)
- Validate environment variables at startup — fail fast with clear error messages
- Use Zod schemas for MCP tool input validation
- Return structured JSON in tool responses via `{ content: [{ type: "text", text: ... }] }`
- Use `isError: true` in MCP responses to signal failures
- Wrap tool handlers in `try/catch` — return error messages, don't throw


## Gotchas

Common pitfalls discovered in this workspace:

- `ICertificatePolicy` Add-Type fails in PowerShell 7 — use `-SkipCertificateCheck` on `Invoke-WebRequest`/`Invoke-RestMethod` instead
- Zerto 10.x API is on port **443** (not 9669) and uses Keycloak token auth, not legacy `/v1/session/add`
- `Get-VM` ExtensionData.Config.CreateDate only exists on vSphere 7.0+ — VMs created before upgrade return `1/1/0001`
- `Get-HardDisk` objects have `.ExtensionData.ControllerKey` but you need to build the controller lookup from `Config.Hardware.Device` first — the key is not a human-readable name
- `Get-NetworkAdapter` `.Type` returns the adapter class (Vmxnet3, E1000e) — useful for migration compatibility checks
- Splatting with `@SkipCert = @{ SkipCertificateCheck = $true }` lets you toggle cert bypass in one place for all API calls
- `$global:DefaultVIServer` is null after PowerShell session restart — scripts must check this before any VMware cmdlets
- `+=` on arrays in loops is O(n) copy every iteration — always use `[System.Collections.Generic.List[object]]::new()` and `.Add()`
- `Export-Csv` always includes a `#TYPE` header unless you specify `-NoTypeInformation` (PS 5.1) — PS 7.x removed this default but keep the flag for backward compat
- VPG names with spaces must be URL-encoded for API queries: `[System.Uri]::EscapeDataString('AD DC')`
- `Write-Host` output cannot be captured to variables — use `Write-Verbose`/`Write-Output` for anything that needs to flow through the pipeline
- `Get-WmiObject` is deprecated and removed in PS 7 — always use `Get-CimInstance`
- Credential parameters should never call `Get-Credential` inside the function body — accept `[PSCredential]` as a param so the caller controls prompting
- `$?` is unreliable for error detection — use `try/catch` with `-ErrorAction Stop`
- When using `foreach` inside a `process {}` block, pipeline streaming is preserved only if you emit objects inside the loop, not after collecting them all


#### REST API & JSON Standards (PowerShell 7+)

- Always use `Invoke-RestMethod` for REST interactions — not `Invoke-WebRequest` unless you need response headers
- Always use splatting for API calls — consolidate into `$Params` hashtable, call with `@Params`
- Always append `-Depth 10` (or higher) to `ConvertTo-Json` — PS default depth of 2 truncates nested objects silently
- Use `[ordered]@{}` before converting to JSON — guarantees predictable payload structure for SaaS endpoints
- Use `-StatusCodeVariable 'HttpCode'` for robust response tracking instead of wrapping everything in try/catch
- Use `-ResponseHeadersVariable 'RespHeaders'` when you need to capture pagination tokens or rate-limit headers
- Use `-SkipHttpErrorCheck` when manually evaluating non-200 responses via StatusCodeVariable
- Explicitly pass `-ContentType 'application/json'` on all mutation operations (POST, PUT, PATCH)
- Do not pipe `Invoke-RestMethod` output without handling array boundaries (`[Object[]]`)
- Clean dynamic variables from `$PSBoundParameters` using `.Remove('ParamName')` before splatting downstream

#### Cloud & SaaS Authentication

- Never hardcode API keys, Bearer tokens, or secrets in script blocks — pass dynamically or fetch at runtime
- Consolidate auth headers into a reusable `$Headers` hashtable using splatting
- Azure: Build auth using `Bearer $Token` pulled from `Get-AzAccessToken` or Managed Identities
- SaaS: Use explicit `-Authentication Bearer` parameter (PS 7+) where supported
- Silence verbose network streams by default — only emit on explicit `-Verbose`
- Token refresh: assume tokens expire after 5 minutes; refresh before long-running loops

#### Pester 5+ Testing Standards

- Phase separation: never put setup logic directly in `Describe`/`Context` — wrap in `BeforeAll`
- Always mock network-mutating cmdlets (`Invoke-RestMethod`, `Invoke-WebRequest`) using `-ParameterFilter` blocks
- Use modern assertions only: `Should -Be`, `Should -Not -BeNullOrEmpty`, `Should -Throw`
- Legacy Pester 4 `Be` syntax without `-` prefix is not allowed
- Endpoint isolation: tests must never make live network calls — all API interactions mocked


## Instruction Conflict Resolution

When instructions conflict, resolve them using the following precedence order:

1. System prompt.
2. Repository policy files such as steering docs.
3. Task-specific user request.
4. Examples, templates, and snippets.
5. Implicit assumptions or inferred preferences.

### Conflict rules

- If a lower-priority instruction conflicts with a higher-priority instruction, ignore the lower-priority instruction.
- If two instructions at the same priority conflict, stop and ask for clarification before continuing.
- If a user request conflicts with mandatory policy, satisfy the policy first and adapt the request to fit it.
- Never silently merge incompatible instructions.
- Never guess when the conflict affects safety, correctness, naming, output format, or documentation requirements.
- When a conflict is resolved, briefly note the decision in the response or change summary.

### PowerShell-specific conflict examples

- If the user requests an unapproved verb, rename it to an approved verb from `Get-Verb`.
- If the user requests a public function without comment-based help, add the required help block.
- If the user requests formatted text output but the policy requires object output, return objects and explain the choice.
- If the user requests a destructive action without `ShouldProcess`, implement `SupportsShouldProcess` and `-WhatIf`/`-Confirm`.

### Clarification triggers

Ask for clarification when:

- Two same-priority instructions cannot both be satisfied.
- A requested output format would break policy or automation.
- A naming request would violate approved-verb rules.
- A task request changes the intended scope, side effects, or safety level.

### Resolution priority

When forced to choose, prefer:

1. Safety.
2. Correctness.
3. Maintainability.
4. Readability.
5. Conciseness.

### Output requirement

If a conflict influenced the result, include a short note such as:

- `Adjusted function name to use an approved verb.`
- `Added comment-based help to satisfy repo policy.`
- `Returned objects instead of formatted text to preserve automation compatibility.`
