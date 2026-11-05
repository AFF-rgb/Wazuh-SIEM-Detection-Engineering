# 07 — Active Directory Threat Detection

This project extends endpoint monitoring to a Windows Active Directory lab.

## Lab environment

The broader lab notes describe the `stech.net` domain with a Windows Server 2022 domain controller named `DC01-LAB` and a Windows 10 client. The lab includes separate standard IT staff and privileged administrative identities, security groups, Group Policy, Fine-Grained Password Policies, and Windows LAPS.

## Detection objectives

The planned Wazuh coverage includes monitoring for:

| Event ID | Activity to investigate |
|---|---|
| 4624 / 4625 | Successful and failed logons |
| 4720 | User account created |
| 4728 / 4732 | Member added to a security-enabled group |
| 4725 / 4726 | Account disabled or deleted |
| 4688 | Process creation (requires suitable audit policy) |
| 4719 | System audit policy changed |
| PowerShell / Sysmon events | Suspicious script and process activity |

Event IDs are investigation starting points, not proof of malicious activity. Validate the event channel, audit policy, event fields, and Wazuh decoder/rule behavior on the domain controller and client.

## Implementation workflow

1. Confirm Windows Security auditing is enabled through the intended policy.
2. Confirm the Wazuh agent collects the Security event channel.
3. Configure Sysmon where required and collect its Operational channel.
4. Generate controlled test events in the domain lab.
5. Confirm the raw event reaches Wazuh.
6. Identify the built-in rule and extracted fields.
7. Create narrowly scoped custom rules where needed.
8. Investigate the account, actor, target object, source host, and timestamp.
9. Record sanitized alert evidence and false-positive considerations.

## Status

**AD lab foundation exists; this repository section is an index for the detection project.** The detailed Wazuh rules, event validation, and evidence should be added from the completed lab tests before the project is described as fully validated.
