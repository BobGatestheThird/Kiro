# Tech Stack

## Primary Language

- PowerShell 5.1+ (vast majority of the codebase)
- Scripts run on Windows; target both local and remote servers via WinRM

## Secondary Languages

- Ansible (YAML playbooks) for multi-step vSphere workflows (`ansible-hw-upgrade/`)
- TypeScript/Node.js for the Infoblox MCP server (`infoblox-mcp-server/`)

## Key PowerShell Modules

- `VMware.PowerCLI` / `Broadcom.PowerCLI` — vSphere automation (VMs, hosts, networking, snapshots)
- `ActiveDirectory` — AD user/group/computer management
- `Microsoft.PowerShell.SecretManagement` — credential storage
- `ImportExcel` — Excel report generation (used in some scripts)

## Ansible Dependencies

- `community.vmware` collection
- `pyvmomi` Python package

## Node.js Dependencies (infoblox-mcp-server)

- `@modelcontextprotocol/sdk` — MCP server framework
- `node-fetch` — HTTP client for Infoblox WAPI
- TypeScript 5.4+, Node 20+

## Common Commands

```powershell
# Connect to vCenter (required before most VMware scripts)
Connect-VIServer -Server vcenter.example.com

# Run a script
.\ScriptName.ps1

# Dot-source a function script to load it into the session
. .\ScriptName.ps1
```

```bash
# Ansible playbooks
ansible-playbook ansible-hw-upgrade/01-gather-inventory.yml --ask-vault-pass

# Build the Infoblox MCP server
cd infoblox-mcp-server && npm install && npm run build
```

## Output Formats

- CSV is the primary export format (via `Export-Csv`)
- Excel via `Export-Excel` in some reporting scripts
- `Out-GridView` for interactive viewing
- Console output with color-coded `Write-Host` for status


## Gotchas

- Zerto ZVM API runs on port 443 (not 9669) in 10.x — auth is Keycloak at `/auth/realms/zerto/protocol/openid-connect/token` with `client_id=zerto-client`
- VMware.PowerCLI no longer ships as a single umbrella module in 13.4+ — use `#Requires -Modules VMware.VimAutomation.Core` for the core cmdlets (`Connect-VIServer`, `Get-VM`, `Get-VMHost`, etc.). The `VMware.PowerCLI` and `Broadcom.PowerCLI` names are not resolvable on current installs.
- `Connect-VIServer` without `-Force` will prompt on cert issues — use `-Force` in automation
- vCenter session tokens timeout after ~30 minutes of inactivity — long-running loops should reconnect or catch `NotAuthenticated` errors
- Ansible `community.vmware` requires `pyvmomi` matching your vCenter version — mismatched versions cause silent API failures
- Node.js MCP server `node-fetch` v3+ is ESM-only — if you need CommonJS, pin to `node-fetch@2`
- `ImportExcel` module isn't pre-installed on most servers — always gate with `#Requires -Modules ImportExcel`
- Infoblox WAPI returns paginated results (1000 max) — use `_max_results` and `_paging=1` for large zones
