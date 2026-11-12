# 03 — Windows Threat Detection

This project documents Windows Security event collection, RDP authentication monitoring, firewall log collection, Sysmon telemetry, and a lab Active Response experiment.

## Lab environment recorded

- Windows 10/11 endpoint VM with Wazuh agent.
- Kali Linux VM used as the authorized test source.
- Ubuntu VM running the Wazuh server components.
- VMs connected to the same isolated lab network.

The original notes contain private lab IP addresses and a machine-specific agent name. These values are intentionally omitted.

## 1. Baseline Windows Security telemetry

Check recent Security events from an elevated PowerShell session:

```powershell
Get-WinEvent -LogName Security -MaxEvents 10 |
  Format-List -Property TimeCreated, Id, Message
```

Enable logon auditing where required by the lab policy:

```powershell
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /get /subcategory:"Logon"
```

The Wazuh agent must collect the Security event channel. The notes include an Application channel example, but that does not replace Security collection. The relevant configuration is:

```xml
<localfile>
  <location>Security</location>
  <log_format>eventchannel</log_format>
</localfile>
```

Confirm that the configuration is present in the Windows agent's `ossec.conf` or is delivered through the intended centralized agent group.

## 2. RDP authentication monitoring

Relevant Windows Security events:

| Event ID | Meaning | Investigation use |
|---|---|---|
| 4625 | Failed logon | Review logon type, account, source address, and failure reason |
| 4624 | Successful logon | Review successful access after preceding failures |

For RDP-focused monitoring, inspect Logon Type 10 in the event data. Do not classify every 4625 as an RDP failure; other logon types represent different authentication paths.

The notes propose:
- Custom rule 100020 for a failed RDP logon.
- Custom rule 100021 for a successful RDP logon.
- Correlation rule 100900 for repeated failures.

The separate SIEM-WAZUH notes provide a more concrete correlation example: custom rule **100100**, using base SID **60112**, with `frequency="5"` and `timeframe="60"`. This is a generic failed-Windows-logon threshold in the notes; it does not itself prove that all five events are RDP Logon Type 10. The example also does not group by source IP, so events from different sources may be counted together. Validate the event fields and correlation scope, and add a supported source grouping key if the detection objective requires per-source counting. The identifiers 100020, 100021, and 100900 remain draft entries until their XML and event behavior are confirmed.

## 3. RDP brute-force correlation

The notes describe a threshold of three failed RDP attempts within 120 seconds. A reliable correlation must verify:
- The base rule matches the intended failed RDP event.
- The event is from the intended Windows agent.
- The source IP is present and decoded.
- Correlation is scoped to the same source IP when that is the intended behavior.
- The alert includes enough fields for an analyst to identify the target account and source.

The primary SIEM-WAZUH notes explicitly state that **rule ID 60112** was confirmed by inspecting alerts for a single failed Windows logon in this lab. A separate later note mentions 60122, so the source material is inconsistent. Treat **60112 as the better-supported historical value**, but re-check `rule.id` in the actual alert JSON before reusing it with another Wazuh release or event pipeline.

## 4. Sysmon telemetry

The notes describe installing Microsoft Sysmon and applying an XML configuration from the OTRF Blacksmith project.

After installation, validate events in Event Viewer under:

```text
Applications and Services Logs
  Microsoft
    Windows
      Sysmon
        Operational
```

Collect the Sysmon Operational channel through Wazuh:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

The notes used centralized configuration through a Windows agent group. Confirm that the endpoint belongs to that group and that the shared configuration is synchronized.

## 5. Windows Firewall log collection and network scanning

The notes propose enabling Windows Firewall logging and collecting `pfirewall.log`. Before treating this as a working Nmap detection:

- Confirm the firewall profile logging settings.
- Confirm the log file path and that the file is being written.
- Confirm the Wazuh agent can read the file.
- Inspect the actual decoded events and the base rule IDs.
- Validate whether the proposed base SID 18107 matches the event source in this lab.

A single firewall log line is not necessarily proof of a port scan. A scan detection should correlate multiple relevant connection attempts over a defined period and retain the source and destination context.

**MITRE ATT&CK:** T1046 — Network Service Discovery.

## 6. Windows OpenSSH (optional)

The notes propose collecting `C:\ProgramData\ssh\logs\sshd.log`. This path is not guaranteed to exist or be enabled on every Windows installation. Confirm the OpenSSH Server feature, logging configuration, and actual log location before adding a Wazuh `localfile` entry.

Do not assume Linux SSH rule IDs will correctly parse Windows OpenSSH logs without testing the decoder and event format.

## 7. Active Response — RDP source blocking

The lab notes describe blocking a test source after three failed attempts in 120 seconds and restoring access after 300 seconds.

**Correction to the initial review:** Wazuh provides a built-in Windows `netsh.exe` Active Response executable for blocking an IP address. The notes' use of a `netsh` command and `local` execution is consistent with the documented Windows response pattern, provided the executable is present in the agent's Active Response directory and the alert contains a usable source IP. A custom script is only needed if the built-in action does not meet the lab requirements. See the [Wazuh Windows Active Response example](https://documentation.wazuh.com/current/proof-of-concept-guide/block-malicious-actor-ip-reputation.html).

Before enabling automatic blocking:
- Use a disposable test endpoint and source.
- Exclude trusted management and monitoring addresses.
- Ensure the response handles both block and timeout/reversal behavior.
- Confirm the firewall rule is created on the intended Windows endpoint.
- Confirm it is removed after the configured timeout.
- Record the Wazuh Active Response log and Windows Firewall state.
- Keep a manual recovery path to avoid locking yourself out.

The original notes record a 300-second timeout. This is a lab setting, not a universal recommendation.

## Validation checklist

- [ ] Security channel is collected by the Wazuh agent.
- [ ] Event IDs 4625 and 4624 are visible in Wazuh.
- [ ] Logon Type 10 is parsed correctly for RDP.
- [ ] Custom rule base SIDs are verified from actual alerts.
- [ ] Three failures from one source produce the intended correlation alert.
- [ ] Sysmon Operational events arrive.
- [ ] Firewall log collection and scan correlation are validated.
- [ ] Active Response blocks and unblocks only the intended lab source.

## Current documentation status

- **Documented:** Windows auditing, Security event collection, RDP event IDs, Sysmon collection, and the intended Active Response scenario.
- **Needs validation:** exact rule IDs, correlation scope, firewall scan logic, OpenSSH log path, and the Active Response executable/rollback implementation.
