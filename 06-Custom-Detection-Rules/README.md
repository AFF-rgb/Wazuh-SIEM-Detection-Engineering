# 06 — Custom Detection Rules

This chapter consolidates the custom Wazuh rules and decoder designs recorded in the supplied lab documentation. It covers Linux authentication and persistence, Windows logon correlation, RDP brute-force response, and network-scanning indicators.

> **Important:** These are lab-derived configurations, not a claim that every rule is production-ready. The notes contain configuration snippets and some test instructions, but do not include complete `wazuh-logtest` transcripts or a matching raw-event/alert pair for every rule. Each rule is therefore labelled according to the evidence actually present.

## Lab context

- Wazuh all-in-one deployment on Ubuntu; source notes identify Wazuh Manager **4.7.5**.
- Linux endpoint with `/var/log/auth.log` and syslog telemetry.
- Windows 10/11 endpoint collecting the Windows Security event channel and, where configured, Sysmon Operational events.
- Kali Linux used as the authorized test source in the isolated VMware lab.
- Centralized agent configuration was organized into `Linux` and `Windows` groups under `/var/ossec/etc/shared/`.

Version note: the documented Windows agent installer is also 4.7.5. Treat these as historical lab versions; confirm Manager/agent compatibility and rule syntax against the release actually deployed before reuse.

## Rule inventory and evidence status

| Custom ID | Use case | Source-note status | Main validation requirement |
|---|---|---|---|
| 100003 | Cron/crontab changes | Decoder and rule recorded; crontab syslog samples recorded | Capture complete `wazuh-logtest` Phase 1–3 output and a manager alert |
| 100010 | SSH brute force followed by successful login | Rule snippet recorded | Verify both parent rule IDs, event ordering, time window, and same-source correlation |
| 100011 | Sudo root-shell activity | Rule snippet recorded | Verify decoded command field and matching behavior |
| 100012 | Local account creation | Rule snippet recorded | Verify the base rule and account-creation event fields |
| 100013 | Authentication-log tampering | Rule snippet recorded | Verify FIM event group, file path field, and alert |
| 100020 | Failed RDP logon (Logon Type 10) | Rule snippet recorded | Verify Windows field names and an Event ID 4625 alert |
| 100021 | Successful RDP logon (Logon Type 10) | Rule snippet recorded | Verify Windows field names and an Event ID 4624 alert |
| 100030 | Possible network scanning | Rule snippet recorded | Verify the firewall base SID and actual source event |
| 100100 | Repeated Windows failed logons | Rule snippet and base SID 60112 recorded in one section | Confirm base SID from the exact 4625 alert and test threshold behavior |
| 100900 | RDP brute-force threshold / Active Response | Rule and response configuration recorded; base SID 60122 recorded in a separate RDP section | Verify source-IP grouping, firewall block, timeout, and safe rollback |

The source notes contain two different base rule IDs for Windows failed logons: **60112** in the general Windows correlation section and **60122** in the RDP/Active Response section. Do not treat them as interchangeable. Confirm the rule ID shown by the actual 4625 alert on the installed Wazuh version before enabling either correlation rule.

## Linux detection designs

### 100010 — SSH brute force followed by successful login

The notes record this rule design:

```xml
<group name="SSH,">
  <rule id="100010" level="12">
    <if_matched_sid>5712</if_matched_sid>
    <if_sid>5715</if_sid>
    <same_source_ip />
    <description>Attack Detected: SSH brute force followed by successful login.</description>
    <mitre>
      <id>T1110.001</id>
    </mitre>
  </rule>
</group>
```

**Version-specific ruleset check (Wazuh v4.7.5):** the upstream v4.7.5 SSH ruleset defines rule **5712** as an SSH brute-force threshold for a non-existent user (8 matches within 120 seconds, same source IP) and rule **5715** as SSH authentication success (child of 5700). This supports the intended parent-rule pairing in the recorded custom rule for that release. See the [Wazuh v4.7.5 SSH rules](https://github.com/wazuh/wazuh/blob/v4.7.5/ruleset/rules/0095-sshd_rules.xml).

**Remaining review notes:** This correlation therefore inherits the built-in 5712 threshold behavior; it does not represent every possible SSH brute-force pattern, such as attempts against an existing account that do not trigger 5712. Confirm the decoded source-IP field and event ordering with representative endpoint logs and `wazuh-logtest`. The custom rule still needs a captured raw-event sequence and generated alert before it can be marked validated. Re-check the parent IDs if the deployed Wazuh version differs from v4.7.5.

### 100011 — Sudo root-shell activity

```xml
<group name="Root,">
  <rule id="100011" level="9">
    <if_sid>5402</if_sid>
    <match>sudo su -|sudo -i</match>
    <description>Privilege escalation: user invoked a root shell.</description>
    <mitre>
      <id>T1548.003</id>
    </mitre>
  </rule>
</group>
```

**Version-specific parent-rule check (Wazuh v4.7.5):** upstream rule 5402 is a child of 5400 and describes a successful sudo operation to ROOT. It matches sudo log content containing `USER=root` and a `COMMAND=` or `TSID=... COMMAND=` field. See the [Wazuh v4.7.5 syslog rules](https://github.com/wazuh/wazuh/blob/v4.7.5/ruleset/rules/0020-syslog_rules.xml).

**Review notes:** The custom pattern is intended to narrow this event to `sudo su -` or `sudo -i`. Confirm what the endpoint actually records in the `COMMAND` field; it may be a resolved executable/argument string rather than the literal command typed. Validate the pipe alternation and matching behavior with representative raw logs and `wazuh-logtest`. Rule 5402 already identifies a successful sudo-to-root event, so the custom rule should be described as a narrower command-pattern alert, not as independent proof of malicious activity.

### 100012 — Local account creation

```xml
<group name=".SSH,">
  <rule id="100012" level="10">
    <if_sid>5902</if_sid>
    <description>Persistence: new user account created.</description>
    <mitre>
      <id>T1136.001</id>
    </mitre>
  </rule>
</group>
```

**Review notes:** Confirm that base rule 5902 corresponds to the intended account-creation event in the installed ruleset. The alert should identify the affected account and endpoint before being treated as useful detection coverage.

### 100013 — Authentication log tampering

```xml
<group name="Wipe_Evidence,">
  <rule id="100013" level="13">
    <if_group>syscheck</if_group>
    <match>/var/log/auth.log</match>
    <description>Critical: authentication log was cleared or deleted.</description>
    <mitre>
      <id>T1070.001</id>
    </mitre>
  </rule>
</group>
```

**Review notes:** The Linux agent configuration in the notes collects `/var/log/auth.log` and configures FIM for selected sensitive paths. Confirm that the generated FIM alert actually contains the path in a field matched by this rule. A generic `match` against the raw event may not match the decoded path field. Test truncation and deletion separately; they may produce different FIM event descriptions.

### 100003 — Cron/crontab persistence

The notes record a custom decoder and rule:

```xml
<decoder name="crontab-custom">
  <program_name>crontab</program_name>
</decoder>

<decoder name="crontab-fields">
  <parent>crontab-custom</parent>
  <regex>:\s+\((\w+)\)\s+(\w+)</regex>
  <order>user, action</order>
</decoder>
```

```xml
<group name="linux, persistence,">
  <rule id="100003" level="10">
    <if_sid>2830</if_sid>
    <decoded_as>crontab-fields</decoded_as>
    <description>Suspicious persistence: cron job $(action) by user $(user)</description>
    <mitre>
      <id>T1053.003</id>
    </mitre>
  </rule>
</group>
```

The source notes include sample syslog entries for crontab `BEGIN EDIT`, `REPLACE`, `END EDIT`, and `LIST` activity, plus the command used to launch `wazuh-logtest`. They describe an expected rule 100003 result, but do not include the actual Phase 1–3 output. Therefore, this is **documented with sample source events; end-to-end rule validation remains pending**.

**Decoder-chain correction:** the rule should identify the most specific decoder that extracts the `user` and `action` fields. In this parent/child decoder design, that is `crontab-fields`, not the parent `crontab-custom`. Confirm the final decoder name and extracted fields in Phase 2 of `wazuh-logtest` before relying on the rule.

The FIM configuration also monitors `/var/spool/cron`. This can provide a second telemetry path for cron persistence, but the custom decoder/rule and FIM alert are distinct detections and should be validated independently.

## Windows detection designs

### 100020 and 100021 — RDP logon events

The notes identify Windows Security Event ID 4625 as failed logon and 4624 as successful logon, with Logon Type 10 representing RemoteInteractive/RDP logon.

```xml
<group name="soc,rdp_tuning">
  <rule id="100020" level="6">
    <field name="win.system.eventID">4625</field>
    <field name="win.eventdata.LogonType">10</field>
    <description>RDP login failure observed.</description>
  </rule>
</group>

<group name="soc,rdp_success">
  <rule id="100021" level="4">
    <field name="win.system.eventID">4624</field>
    <field name="win.eventdata.LogonType">10</field>
    <description>RDP successful login observed.</description>
  </rule>
</group>
```

**Review notes:** Confirm the exact field names and values in the decoded event from the Windows agent. These rules identify individual RDP logon events; they do not, by themselves, detect brute force or prove that a successful login belongs to the same attack source.

### 100100 — Repeated Windows failed logons

The general Windows section records base rule ID 60112 and this threshold:

```xml
<group name="windows,bruteforce">
  <rule id="100100" level="10" frequency="5" timeframe="60">
    <if_matched_sid>60112</if_matched_sid>
    <description>Five Windows failed logons within 60 seconds — possible brute-force activity.</description>
  </rule>
</group>
```

**Review notes:** The supplied configuration does not group matches by source IP. Without a stable grouping key, failures from different sources may contribute to the same threshold. Add and test source-IP or another appropriate correlation key after confirming the decoded field. The notes state that 60112 was confirmed in one lab inspection, but another section records 60122 for the RDP scenario; verify the base SID for the specific event before use.

### 100900 — RDP brute-force threshold and Active Response

The RDP section records a threshold of three matches in 120 seconds using base rule 60122:

```xml
<group name="RDP,">
  <rule id="100900" level="10" frequency="3" timeframe="120">
    <if_matched_sid>60122</if_matched_sid>
    <description>Windows RDP brute force: three failed attempts within two minutes.</description>
  </rule>
</group>
```

The notes also record an Active Response command using `netsh.exe`, with local execution and a 300-second timeout. This is intended to block the attacking source temporarily on the Windows endpoint.

**Safety and validation requirements:**

- Confirm that the base SID represents a failed RDP logon (Event ID 4625, Logon Type 10).
- Scope the threshold to the same source IP; the recorded rule does not show a source-IP grouping condition.
- Confirm the response command is correctly defined for the Windows agent and supported by the installed Wazuh release.
- Test only with a controlled lab source and confirm the firewall rule blocks the expected address.
- Verify automatic removal after the 300-second timeout and document a manual rollback path.
- Ensure the response cannot block the administrator's management address or lock out the lab operator.

The source notes describe a three-attempt/two-minute threshold and a five-minute unblock interval, but do not include a complete alert record plus firewall-state evidence in the text. Treat the response as **configured in the lab notes; end-to-end validation must be evidenced**.

## Network scanning indicator — 100030

```xml
<group name="soc,network">
  <rule id="100030" level="7">
    <if_sid>18107</if_sid>
    <description>Possible network scanning activity.</description>
  </rule>
</group>
```

The source notes associate base SID 18107 with Windows Firewall-related scan telemetry. Verify the base rule against the actual alert and confirm the Windows Firewall log is being collected by the agent. A single firewall event or port connection is not sufficient to establish an Nmap scan; use event volume, destination-port diversity, time window, and source identity to support the conclusion. The rule as recorded is an indicator wrapper, not a complete scan-correlation rule.

## Telemetry and centralized agent configuration recorded in the notes

### Windows Security and Sysmon

Windows Security event collection:

```xml
<localfile>
  <location>Security</location>
  <log_format>eventchannel</log_format>
</localfile>
```

Sysmon Operational event collection:

```xml
<agent_config>
  <localfile>
    <location>Microsoft-Windows-Sysmon/Operational</location>
    <log_format>eventchannel</log_format>
  </localfile>
</agent_config>
```

The notes record centralized configuration under `/var/ossec/etc/shared/Windows/agent.conf`, validation using `/var/ossec/bin/verify-agent-conf`, and agent synchronization confirmation for agent 003. These are configuration-management steps; they do not independently prove that specific Sysmon events reached the manager.

### Linux authentication and FIM

The Linux shared configuration records:

```xml
<syscheck>
  <directories check_all="yes" report_changes="yes" realtime="yes">/home/*/.ssh/authorized_keys</directories>
  <directories check_all="yes" realtime="yes">/etc/passwd</directories>
  <directories check_all="yes" realtime="yes">/etc/shadow</directories>
  <directories realtime="yes">/var/spool/cron</directories>
</syscheck>

<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/auth.log</location>
</localfile>
```

The notes record successful `verify-agent-conf` checks for the Linux and Windows group configurations and a synchronization confirmation for Linux agent 002. Verify the effective merged configuration on each endpoint and confirm that the monitored paths exist and are accessible to the agent.

## Validation workflow

For every rule, retain sanitized evidence of the following:

1. **Raw telemetry:** representative original event from the endpoint (Windows Security, Sysmon, syslog, or FIM).
2. **Decoder output:** Phase 1 from `/var/ossec/bin/wazuh-logtest`.
3. **Decoded fields:** Phase 2, including the source IP, username, event ID, logon type, command, or file path used by the rule.
4. **Rule evaluation:** Phase 3 showing the built-in and custom rule IDs, level, and description.
5. **Live validation:** endpoint-generated event visible in `/var/ossec/logs/alerts/alerts.json` or the Wazuh dashboard.
6. **Response validation:** for Active Response, firewall state before, during, and after the timeout.

Run the utility on the manager:

```bash
sudo /var/ossec/bin/wazuh-logtest
```

Check manager alerts:

```bash
sudo tail -f /var/ossec/logs/alerts/alerts.json
```

Do not mark a rule **Validated** based only on an XML configuration being saved, the manager restarting, or an expected result written in notes.

## MITRE ATT&CK mapping recorded for the scenarios

| Scenario | Technique | Interpretation |
|---|---|---|
| SSH password guessing | T1110.001 — Password Guessing | Repeated password-authentication attempts |
| Sudo abuse / root shell | T1548.003 — Sudo and Sudo Caching | Privilege escalation through sudo |
| Local account creation | T1136.001 — Local Account | Creation of a local account for persistence |
| Authentication log clearing | T1070.001 — Clear Linux or Mac System Logs | Removal or alteration of Linux log evidence |
| Cron persistence | T1053.003 — Cron | Scheduled task persistence on Linux |

Mappings are scenario-level references from the lab notes. Keep them on individual alerts only when the observed behavior and event fields support the technique. RDP brute force maps to the broader T1110 family; successful use of valid credentials should not automatically be labelled T1078 without evidence of valid-account use.

## Secret handling

The source archive contains sensitive credential/key material. It is intentionally not reproduced here. Before publishing any further raw logs, screenshots, or configuration exports, replace **passwords, SSH keys, and hashes** with the literal `<REDACTED>`. Preserve the technical context needed to understand the lab, and rotate any credential that was exposed outside the lab.

## Current publication status

- **Documented:** custom rule and decoder designs, telemetry paths, attack scenarios, and agent-group configuration from the supplied archive.
- **Recorded in notes:** crontab syslog samples, configuration validation results, agent synchronization messages, and descriptions of expected alert/response behavior.
- **Needs validation evidence:** complete log-test output for each custom rule, raw-event-to-alert pairs, exact base rule IDs for the installed ruleset, and end-to-end Active Response/firewall timeout proof.

This chapter is an engineering record of the lab work, not a claim that all listed detections are production-ready or independently validated.
