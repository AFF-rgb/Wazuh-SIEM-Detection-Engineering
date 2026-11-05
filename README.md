# Wazuh SIEM & Detection Engineering

Hands-on SIEM engineering portfolio using Wazuh for Linux and Windows security monitoring, detection engineering, endpoint telemetry, file integrity monitoring, automated response, network detection, and Active Directory security monitoring.

This repository documents a home lab through its architecture, configuration, attack simulations, detection logic, validation steps, and investigation notes. It distinguishes implemented work from detections that still require validation.

## Portfolio objectives

- Deploy and administer a Wazuh Manager, Indexer, Dashboard, and endpoint agents.
- Collect and analyze Linux authentication, Windows Security, and Sysmon telemetry.
- Build and validate custom detection and correlation rules.
- Simulate controlled attack techniques and investigate resulting alerts.
- Monitor sensitive files and persistence locations with File Integrity Monitoring (FIM).
- Test endpoint response actions with safeguards and rollback.
- Extend monitoring to network telemetry and Active Directory activity.

## Lab scope

The notes describe a VMware-based lab with an Ubuntu Wazuh server, Windows endpoints, and Kali Linux used for authorized simulations. Exact host addresses, usernames, and machine-specific values are intentionally represented as placeholders.

| Area | Coverage | Documentation status |
|---|---|---|
| Wazuh deployment | Ubuntu server, core services, dashboard, agent enrollment | Documented; version and OS compatibility caveats recorded |
| Linux detection | SSH authentication, privilege escalation, account/key persistence, log tampering, cron changes | Scenarios documented; custom rules require per-version validation |
| Windows detection | Security events, RDP logons, firewall logs, Sysmon | Configuration documented; telemetry and rule IDs must be verified on the target |
| Active response | Temporary blocking of a simulated RDP source | Lab configuration recorded; implementation requires correction and end-to-end revalidation |
| FIM | Linux account files, SSH keys, cron locations | Configuration documented; tune and validate on agents |
| Suricata | Wazuh integration and network alert analysis | Separate lab work; detailed configuration/evidence to be added |
| Active Directory | Domain and Windows event monitoring | Lab work in progress; detection coverage to be documented separately |

## Repository structure

```text
Wazuh-SIEM-Detection-Engineering/
├── README.md
├── 01-Wazuh-Deployment/
├── 02-Linux-Threat-Detection/
├── 03-Windows-Threat-Detection/
├── 04-File-Integrity-Monitoring/
├── 05-Wazuh-Suricata-Integration/
├── 06-Custom-Detection-Rules/
├── 07-Active-Directory-Threat-Detection/
├── 08-Incident-Investigation/
└── 09-Evidence-and-Documentation/
```

Each project area should contain its own README, configuration examples, validation procedure, and sanitized evidence where available.

## Documentation and validation standard

Every detection is documented using the following sequence:

1. **Objective** — what behavior is being monitored and why.
2. **Lab topology** — manager, agent, test source, and relevant network boundaries.
3. **Telemetry** — exact log channel or file and the collection configuration.
4. **Detection logic** — built-in rule dependency, custom rule, fields, and MITRE ATT&CK mapping.
5. **Simulation** — controlled steps performed only in the isolated lab.
6. **Validation** — raw event, Wazuh alert, rule ID, timestamps, and source/agent fields.
7. **Investigation** — interpretation, scope, and recommended analyst actions.
8. **Limitations** — false positives, missing telemetry, version dependencies, and remaining work.

Use these labels consistently:

- **Documented:** described in lab notes.
- **Configured:** configuration was added to the lab.
- **Validated:** the expected telemetry and alert were observed and recorded.
- **Needs validation:** the notes do not contain sufficient evidence to claim a successful test.

A configuration file passing XML validation does not, by itself, prove that a detection works.

## Safety and secret handling

All simulations are for systems and networks owned by the operator or explicitly authorized for testing. Keep attack simulations inside an isolated lab. Never publish passwords, API keys, private keys, real public IP addresses, personal data, or unredacted logs. Use placeholders in examples and sanitize screenshots before committing them.

## Version notes

The source notes refer to Wazuh 4.7 and a Windows agent package 4.7.5. These are historical lab versions, not a recommendation to deploy them today. Confirm Manager and agent compatibility and consult the documentation for the exact installed release before reusing commands.

## Project index

- [01 — Wazuh Deployment](01-Wazuh-Deployment/README.md)
- [02 — Linux Threat Detection](02-Linux-Threat-Detection/README.md)
- [03 — Windows Threat Detection](03-Windows-Threat-Detection/README.md)
- [04 — File Integrity Monitoring](04-File-Integrity-Monitoring/README.md)
- [05 — Wazuh and Suricata Integration](05-Wazuh-Suricata-Integration/README.md)
- [06 — Custom Detection Rules](06-Custom-Detection-Rules/README.md)
- [07 — Active Directory Threat Detection](07-Active-Directory-Threat-Detection/README.md)
- [08 — Incident Investigation](08-Incident-Investigation/README.md)
- [09 — Evidence and Documentation](09-Evidence-and-Documentation/README.md)
