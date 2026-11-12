# 07 — Active Directory Threat Detection

This chapter documents the Active Directory security baseline and the planned Wazuh detection work for the `stech.net` home lab. It separates domain hardening controls from SIEM telemetry and detection validation.

## Lab environment

| Component | Recorded configuration |
|---|---|
| Domain | `stech.net` |
| Domain controller | `DC01-LAB` — Windows Server 2022 Standard Evaluation, build 20348 |
| Domain functional level | `Windows2016Domain` |
| Forest functional level | `Windows2016Forest` |
| Client | Windows 10 Pro |
| Standard IT identity | `a.fadous` |
| Privileged identity | `IT.sysadmin` |
| Standard IT group | `GG_IT_Staff` |
| Privileged group | `GG-Privileged-Accounts` |

The intended access model separates day-to-day IT work from domain administration: `a.fadous` is a standard IT user, while `IT.sysadmin` is the dedicated privileged identity. Membership and effective permissions must be checked on the target objects before claiming that access is restricted.

## Active Directory security baseline

### Domain password and lockout policy

The lab notes record these domain-level settings:

| Policy | Recorded value |
|---|---:|
| Minimum password length | 14 characters |
| Maximum password age | 0 (passwords do not expire) |
| Account lockout threshold | 5 invalid attempts |
| Account lockout duration | 15 minutes |
| Reset account lockout counter after | 15 minutes |

A maximum password age of 0 disables password expiration; it is not a password-strength control. Review this choice against the organization's password and identity policy before applying it outside the isolated lab.

### Fine-Grained Password Policy

The recorded privileged-account policy is `FGPP-Privileged-Accounts`:

| Setting | Recorded value |
|---|---:|
| Precedence | 1 |
| Minimum password length | 16 characters |
| Maximum password age | 0 |
| Password complexity | Enabled |
| Lockout threshold | 5 attempts |

The policy is assigned to `GG-Privileged-Accounts`. Confirm the resultant password settings for each privileged user; creating a PSO or adding a group membership alone does not prove that the PSO applies.

### Windows LAPS

The lab includes the Windows LAPS schema update and the `msLAPS-*` attributes. The recorded permissions include the computer's self-permission to update its managed password and read access for Domain Admins. Validate password backup and retrieval on a test computer, and confirm that ordinary IT users cannot read the managed local administrator password.

## Wazuh telemetry and detection scope

The Windows Security channel is the primary source for domain authentication, account, group-membership, and audit-policy events. Wazuh's Windows agent monitors the Security channel by default in its standard event-channel configuration; verify the actual deployed `ossec.conf` and agent version rather than adding duplicate collection blocks. Wazuh supports additional channels, including Sysmon and PowerShell Operational, when explicitly configured. See the [Wazuh Windows event-channel collection guide](https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/configuration.html).

| Windows Event ID | Activity | Investigation focus |
|---|---|---|
| 4624 | Successful logon | Account, logon type, source address, workstation |
| 4625 | Failed logon | Target account, status/substatus, source address, logon type |
| 4720 | User account created | Subject (actor), target account, timestamp |
| 4728 / 4732 | Member added to a security-enabled group | Actor, target group, added member |
| 4725 / 4726 | Account disabled / deleted | Actor and affected account |
| 4688 | Process created | Process, parent process, account, command line (if configured) |
| 4719 | System audit policy changed | Changed subcategory and actor |

These event IDs are investigation starting points, not proof of malicious activity. Event availability depends on the effective audit policy and the system where the action occurs. Domain-controller account and group changes should be validated on the domain controller's Security log.

### Optional Sysmon and PowerShell telemetry

If Sysmon is installed and configured, its events are written to `Microsoft-Windows-Sysmon/Operational`. The Wazuh agent must collect that channel explicitly if it is not already configured:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

PowerShell Operational logging is a separate channel and should be enabled and collected only when the lab requires that telemetry. Do not describe Sysmon or PowerShell detections as implemented until the channel is configured and matching events are visible in Wazuh.

## Validation workflow

1. Record the effective domain audit policy and confirm the required subcategories are enabled.
2. Confirm the Wazuh agent is enrolled and active on the domain controller and Windows client.
3. Confirm Security events are being collected from each relevant endpoint.
4. Generate controlled lab actions: create a test user, change a test user's group membership, disable a test account, and perform successful and failed logons.
5. Locate each raw Windows event and record its Event ID, timestamp, computer, actor, target, and relevant fields.
6. Locate the corresponding Wazuh alert and record its `rule.id`, `rule.description`, `agent.name`, and decoded event fields.
7. Create custom rules only where the built-in rules do not provide the required detection; test each rule with representative raw events and `wazuh-logtest`.
8. Retain screenshots or sanitized JSON evidence for both the source event and resulting alert.

## Evidence and current status

- **AD foundation:** Domain, Windows Server 2022 domain controller, Windows 10 client, standard and privileged identities, security groups, domain password/lockout policy, FGPP, and Windows LAPS configuration are recorded in the lab history.
- **Wazuh AD detection:** Not yet documented as validated in this repository chapter. The supplied `wazuh-documentation(1).zip` archive contains the Wazuh deployment, Windows brute-force, Linux FIM, Suricata, and Shuffle notes, but does not contain a dedicated AD detection evidence set.
- **Next evidence required:** Screenshots or raw event/alert pairs from the domain controller and client. Do not infer successful Wazuh detection solely from the existence of an AD policy, a user/group, or an audit event in Event Viewer.

## Security and operational notes

- Keep the standard IT identity out of privileged groups unless a documented task requires it.
- Use a dedicated administrative identity for domain administration and avoid routine browsing or email with that identity.
- Review the effective Fine-Grained Password Policy for each privileged account.
- Treat LAPS read permissions as sensitive; test access with both an authorized administrator and an ordinary IT user.
- Preserve hostnames, usernames, timestamps, IP addresses, and technical details in portfolio evidence. Protect passwords and SSH private keys with `<REDACTED>`. Hash values that are part of controlled demonstrations may remain visible, as agreed for this portfolio.
