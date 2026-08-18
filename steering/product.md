# Product Overview

This is a Windows infrastructure automation toolkit — a collection of PowerShell scripts, Ansible playbooks, and supporting tools used by a systems/infrastructure team to manage a VMware vSphere environment, Active Directory, DNS, SCCM, and Windows Server fleet.

## Key Domains

- VMware vSphere management (VM inventory, hardware upgrades, migrations, networking, ESXi host config)
- Active Directory administration (account lockouts, group membership, password management, replication)
- Windows Server forensics and inventory (system info, services, software, network config, IIS, SQL)
- Server hardening and compliance (baseline validation, TLS, registry, firewall)
- DNS management and testing (Infoblox NIOS integration via MCP server)
- Monitoring integrations (Splunk, Zerto, Tanium)
- Server decommissioning workflows
- Certificate management and validation
- SCCM/ConfigMgr client remediation

## Target Environment

- Windows Server fleet (2012 R2, 2016, 2019+)
- VMware vSphere / vCenter (ESXi 7.0+)
- Active Directory domain environment
- Infoblox NIOS for DNS/IPAM
- Splunk for log aggregation
- Zerto for disaster recovery
- Tanium for endpoint management


## Gotchas

- Use a consistent server naming convention that encodes site and role — misreading the prefix leads to running scripts against the wrong environment
- AD replication latency means changes (group adds, password resets) may not be visible on all DCs for up to 15 minutes
- ESXi 7.0 hosts in lockdown mode block WinRM and SSH — must use vCenter API or DCUI for management
- Zerto VRA VMs show as `z-vra-*` in vCenter — do NOT snapshot, migrate, or power-off these without coordination with the DR team
- SCCM client remediation scripts require local admin — verify `Invoke-Command` credentials have elevation
- Tanium endpoint data has a 15-minute staleness window — don't use it for real-time verification
- Splunk HEC (HTTP Event Collector) tokens are per-index — wrong token = silent data loss
- Decommission workflows must complete in your ITSM tool before removing AD computer objects — jumping the gun breaks CMDB integrity
