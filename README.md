# Kiro Configuration — Infrastructure Automation Toolkit

A shareable `.kiro` configuration for Windows infrastructure teams using PowerShell, VMware, Active Directory, and related technologies with the [Kiro IDE](https://kiro.dev).

## What's Included

| Folder | Contents |
|--------|----------|
| `settings/` | MCP server configuration templates (Infoblox, Microsoft Learn) |
| `skills/` | Kiro skill files for scaffolding scripts, tests, reviews, and conversions |
| `steering/` | Always-on rules for coding standards, project structure, and communication style |
| `specs/` | Structured requirements and design docs (PowerShell standards, asset inventory app) |

## Quick Start

1. Copy this entire folder to your project root as `.kiro/`
2. Open the project in Kiro IDE
3. Steering files load automatically — Kiro will follow the coding standards on every interaction
4. Activate skills in chat using `#skill-name`

## Skills

| Skill | Trigger | Purpose |
|-------|---------|---------|
| `new-advanced-function` | `#new-advanced-function` | Scaffold a compliant PowerShell advanced function with CmdletBinding, help, validation |
| `new-server-report` | `#new-server-report` | Scaffold a remote data collection script with CSV export and error logging |
| `new-pester-test` | `#new-pester-test` | Generate Pester 5+ tests with proper mocking for any function |
| `new-rest-api-function` | `#new-rest-api-function` | Scaffold a REST API wrapper with auth, pagination, and splatting (PS 7+) |
| `convert-legacy-script` | `#convert-legacy-script` | Modernize a legacy script — fix aliases, deprecated cmdlets, error handling |
| `performance-review` | `#performance-review` | Generate a performance review accomplishment mapped to organizational core values |

## Steering Files

| File | Purpose |
|------|---------|
| `caveman.md` | Concise communication mode — drops filler, keeps technical precision |
| `tech.md` | Tech stack definition — PowerShell, Ansible, TypeScript, module versions, gotchas |
| `structure.md` | Comprehensive PowerShell coding standards (formatting, naming, error handling, security, REST, Pester) |
| `product.md` | Product context — target environment, key domains, operational gotchas |

## Customization

### Adapt to your organization

- **`steering/product.md`** — Update the target environment, key domains, and gotchas to match your infrastructure
- **`steering/structure.md`** — Adjust naming conventions and prefixes to your team's standards
- **`skills/performance-review.md`** — Replace the example core values with your organization's values
- **`settings/mcp.json`** — Update server paths, hosts, and credentials for your environment

### MCP Servers

The included `mcp.json` has template entries:

| Server | Purpose | Action Required |
|--------|---------|----------------|
| `microsoft-learn` | MS docs search (works out of the box) | None — enabled by default |
| `infoblox-nios` | DNS/IPAM via Infoblox WAPI | Set `INFOBLOX_HOST`, `INFOBLOX_USER`, `INFOBLOX_PASSWORD` |
| `infra-tools` | Custom PowerShell MCP server | Update file path to your server script |

## What These Standards Cover

- PowerShell formatting (OTBS, 4-space indent, 115-char lines, splatting)
- Naming (PascalCase, approved verbs, full cmdlet names)
- Function structure (CmdletBinding, param validation, OutputType, pipeline support)
- Error handling (try/catch, -ErrorAction Stop, no $? usage)
- Security (PSCredential, no hardcoded secrets, SecureString)
- Collections (Generic List, never += on arrays)
- REST API patterns (splatting, ConvertTo-Json -Depth 10, StatusCodeVariable)
- Change logging (before/after state, immediate CSV append)
- Pester 5+ testing (BeforeAll, mocked network calls, modern assertions)
- Ansible playbook conventions
- TypeScript MCP server patterns

## Requirements

- [Kiro IDE](https://kiro.dev)
- PowerShell 5.1+ or PowerShell 7+
- Windows environment (scripts target WinRM/Windows Server)

## License

These configuration files are provided as-is for educational and productivity purposes. Adapt freely to your environment.
