---
inclusion: manual
---

# Skill: Convert-LegacyScript

Modernize an existing PowerShell script to comply with all workspace steering standards.

## Procedure

1. **Read the target script** — analyze current state and identify violations.

2. **Audit against standards** — check each category and note issues:

### Code Structure
| Check | Fix |
|-------|-----|
| Missing `[CmdletBinding()]` | Add with appropriate attributes |
| No `param()` block | Extract hardcoded values to parameters |
| No `begin/process/end` | Restructure (process for pipeline items) |
| Logic in bare script body | Wrap in named function |
| Missing `Set-StrictMode` | Add `-Version Latest` in begin/function body |

### Naming & Aliases
| Check | Fix |
|-------|-----|
| Aliases used (`gps`, `?`, `%`, `foreach`, `select`) | Replace with full cmdlet names |
| Positional parameters | Add explicit parameter names |
| Non-approved verb in function name | Rename to approved verb |
| Non-PascalCase variables/params | Rename to PascalCase (public) or camelCase (local) |
| `$Param_Computer` style | Rename to `$ComputerName` standard names |

### Deprecated Patterns
| Check | Fix |
|-------|-----|
| `Get-WmiObject` | Replace with `Get-CimInstance` |
| `$?` for error checking | Replace with try/catch + `-ErrorAction Stop` |
| `Write-Host` for data | Replace with `Write-Verbose` or pipeline output |
| `+=` on arrays in loops | Replace with `List[PSCustomObject]::new()` + `.Add()` |
| `return $variable` | Remove `return`, place on pipeline |
| Backtick line continuation | Replace with splatting or natural continuation |
| `ICertificatePolicy` Add-Type | Replace with `-SkipCertificateCheck` |
| `[System.Net.ServicePointManager]` | Remove — use per-call `-SkipCertificateCheck` |
| Semicolons as line terminators | Remove semicolons |

### Error Handling
| Check | Fix |
|-------|-----|
| No error handling | Add try/catch around external calls |
| `$ErrorActionPreference` not set | Add where needed for non-cmdlet code |
| Errors silently swallowed | Add `Write-Error` or re-throw |
| Flag variables for error state | Replace with exception flow |

### Documentation
| Check | Fix |
|-------|-----|
| No comment-based help | Add `.SYNOPSIS`, `.EXAMPLE` minimum |
| Real IPs in examples | Replace with RFC 5737 (`192.0.2.x`) |
| Real server names in help | Replace with `server01.example.com` |
| Comments describe "what" not "why" | Rewrite to explain intent |

### Output & Collections
| Check | Fix |
|-------|-----|
| Raw string output | Replace with `[PSCustomObject]@{}` |
| `Format-*` in tool functions | Remove — output raw objects |
| Missing `[OutputType()]` | Add attribute matching actual output |
| Array concatenation in loops | Replace with Generic List |

### Security
| Check | Fix |
|-------|-----|
| Plaintext passwords | Convert to `[PSCredential]` parameter |
| `Get-Credential` inside function | Move to parameter, caller provides |
| Hardcoded tokens/keys | Extract to parameter or SecretManagement |

### Performance
| Check | Fix |
|-------|-----|
| `ForEach-Object` on large static collections | Consider `foreach` construct |
| `+=` array building | `List[object]::new()` |
| Repeated remote calls in loop | Batch with `-ComputerName` array |

3. **Apply fixes** in this order (to minimize conflicts):
   1. Structural changes (add function wrapper, CmdletBinding, param block)
   2. Rename aliases to full cmdlet names
   3. Replace deprecated cmdlets
   4. Fix error handling
   5. Fix output patterns (objects not strings)
   6. Fix collections (List not +=)
   7. Add documentation
   8. Clean formatting (OTBS braces, 4-space indent, 115 char lines)

4. **Preserve behavior** — the modernized script must produce identical functional output. If a behavioral change is unavoidable (e.g., switching from `Write-Host` to pipeline output changes capture behavior), document it in the commit message.

5. **Add missing infrastructure:**
   - `#Requires -Modules` for any external modules used
   - `SupportsShouldProcess` if script changes state
   - Parameter validation attributes
   - Change logging pattern (if state-changing and missing)

6. **Generate summary** — after conversion, provide:
   - Count of changes by category
   - Any behavioral differences from original
   - Recommendations for further improvement (e.g., "consider adding Pester tests")

7. **Final checks:**
   - Run a mental/static analysis pass for remaining aliases
   - Confirm no `Get-WmiObject`, no `+=` in loops, no backtick continuation
   - Verify comment-based help is present and uses safe example data
   - Confirm output type matches `[OutputType()]` attribute
   - Ensure whitespace changes are clean (no mixed tabs/spaces)
