# PowerShell Development Spec

## Overview

Standard specification template for developing PowerShell scripts, advanced functions, and modules following PowerShell best practices and the PoshCode Style Guide.

## Scope

Applies to all new PowerShell development in this workspace:
- Standalone `.ps1` scripts (flat execution)
- Advanced functions (dot-sourced or imported)
- Script modules (`.psm1` + manifest `.psd1`)

## Design Principles

1. **Verb-Noun naming** — Only approved verbs (`Get-Verb`)
2. **Pipeline-first** — Tools output objects to pipeline; controllers orchestrate
3. **Fail fast** — Validate prerequisites before doing work
4. **Idempotent where possible** — Running twice should not break state
5. **Audit trail** — State-changing operations log before/after values

## Architecture Decision

| Type | When to Use |
|------|-------------|
| Standalone script | One-off automation, pipeline output captured via `$results = .\Script.ps1` |
| Advanced function | Reusable tool, accepts pipeline input, dot-sourced into session |
| Module | Multiple related functions, shared state, formal distribution |

## Output Contract

- Scripts that store mutable session data: use `$global:VarName` as `Generic.List`, do NOT output to pipeline
- Scripts that produce results for capture: output to pipeline directly, no `Format-*` cmdlets
- All output as `[PSCustomObject]` — never raw strings

## Error Strategy

- `[CmdletBinding()]` on everything
- `-ErrorAction Stop` on cmdlets where failure must halt
- `try/catch` for transaction blocks
- `Write-Error` for failures, `Write-Warning` for non-fatal, `Write-Verbose` for progress

## Security

- Credentials via `[PSCredential]` parameter — never plaintext
- Sensitive values decrypted inline at point-of-use
- No hardcoded IPs/hostnames in function bodies — parameterize or config-file
