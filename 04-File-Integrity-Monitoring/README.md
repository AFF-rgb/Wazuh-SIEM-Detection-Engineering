# 04 — File Integrity Monitoring

This chapter documents two Wazuh File Integrity Monitoring (FIM) experiments from the lab notes:

1. Linux monitoring for account files, SSH authorized keys, authentication logs, and cron locations.
2. Windows monitoring of the `C:\\Users\\Public` directory, progressing from basic change detection to user attribution and text-change reporting.

FIM identifies changes to monitored files and directories. An alert is an investigation lead; it does not, by itself, establish malicious intent.

## 1. Linux FIM — monitored paths

| Path | Security relevance | Investigation focus |
|---|---|---|
| `/etc/passwd` | Local account metadata | Unexpected account creation or modification |
| `/etc/shadow` | Local password hashes and account state | Unexpected changes; never publish file contents |
| `/home/*/.ssh/authorized_keys` | SSH key-based access | Added, removed, or modified authorized keys |
| `/var/log/auth.log` | Authentication activity | Unexpected deletion, truncation, or modification |
| `/var/spool/cron` | Scheduled-task persistence | Unexpected cron entry changes |

The notes place Linux monitoring in the Wazuh manager's Linux agent-group configuration under:

```text
/var/ossec/etc/shared/Linux/agent.conf
```

Example based on the recorded configuration:

```xml
<agent_config os="Linux">
  <syscheck>
    <directories check_all="yes" report_changes="yes">/home/*/.ssh/authorized_keys</directories>
    <directories check_all="yes">/etc/passwd</directories>
    <directories check_all="yes">/etc/shadow</directories>
    <directories realtime="yes">/var/spool/cron</directories>
    <directories check_all="yes">/var/log/auth.log</directories>
  </syscheck>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/auth.log</location>
  </localfile>
</agent_config>
```

**Sensitive-file handling:** Do not enable content-diff reporting for `/etc/shadow`. It contains password hashes. Monitor its metadata and changes without copying its contents into alerts or public evidence. Review whether `report_changes` is appropriate for each path.

## 2. Windows FIM — Public folder experiment

The Windows FIM notes and screenshots document a staged experiment using:

```text
C:\Users\Public
```

The objective was to observe a file change in the Wazuh Dashboard, then enrich the event with the user responsible and the changed text content.

### Stage 1 — Basic directory monitoring

The notes changed the Windows agent's `<syscheck>` configuration and set the scan frequency to 10 seconds:

```xml
<syscheck>
  <disabled>no</disabled>
  <frequency>10</frequency>
  <directories>C:\Users\Public</directories>
</syscheck>
```

A test text file was created under the monitored Public folder, and the resulting FIM event was checked in the Wazuh Dashboard.

**Operational note:** 10 seconds is the interval recorded for this lab experiment, not a general production recommendation. Frequent scheduled scans can increase endpoint and manager workload. Choose a frequency based on the number and size of monitored paths and the required detection latency.

### Stage 2 — Identify the user who changed a file

The configuration was extended with `whodata="yes"`:

```xml
<directories whodata="yes">C:\Users\Public</directories>
```

The purpose of this stage was to collect who-data associated with a monitored change, where supported by the endpoint's audit configuration and Wazuh requirements.

When validating who-data, inspect the generated event for the actor/account fields. Do not infer attribution solely from the fact that a FIM alert was generated.

### Stage 3 — Report text changes

The final recorded configuration added `report_changes="yes"`:

```xml
<directories whodata="yes" report_changes="yes">C:\Users\Public</directories>
```

This stage was intended to show both the user associated with the modification and the content changes for supported text files.

Use a disposable text file for this test. Do not enable change reporting on files that may contain passwords, SSH keys, hashes, or other sensitive content. Content-diff reporting is not a substitute for secure evidence handling.

### Applying Windows configuration changes

The source notes instruct restarting the Wazuh agent after editing its configuration. The service can be restarted from elevated PowerShell:

```powershell
Restart-Service WazuhSvc
Get-Service WazuhSvc
```

Confirm that the service returns to the running state and that the agent reconnects to the manager before testing.

## 3. Validation workflow

### Linux

1. Confirm the Linux agent is assigned to the intended group.
2. Validate the centralized configuration with the Wazuh verification utility for the installed release.
3. Confirm the group configuration is synchronized to the endpoint.
4. Record the initial FIM baseline.
5. Make one controlled change to a disposable test file or lab account/key.
6. Confirm the expected FIM alert appears.
7. Review the file path, operation, agent, timestamp, and relevant metadata.
8. Restore the lab state and record the result.

### Windows

1. Confirm the Windows agent is connected and collecting FIM events.
2. Start with basic monitoring of `C:\Users\Public`.
3. Create or modify a disposable text file in that directory.
4. Confirm a FIM event appears in the Wazuh Dashboard.
5. Enable `whodata="yes"` and repeat the test; inspect the actor information.
6. Enable `report_changes="yes"` and repeat the text-file test; inspect the recorded change.
7. Restart the Wazuh agent after configuration changes and confirm it reconnects.
8. Remove the test file and document the cleanup event if it is generated.

## 4. Evidence and validation status

The supplied archive contains 13 screenshots associated with the Windows FIM experiment. They show the Wazuh dashboard, Windows agent configuration, service management, creation of a test file in the Public folder, and dashboard event views.

**Evidence publication status:** the 13 source screenshots have been reviewed. Nine do not visibly expose password, SSH-key, or hash values. Four detailed event screenshots (source filenames `6.png`, `8.png`, `10.png`, and `13.png`) contain hash values and must have those values replaced with the literal `<REDACTED>` before publication. They are not linked here yet. Preserve all other technical details, including IP addresses, MAC addresses, hostnames, usernames, and timestamps.

| Use case | What the source notes/screenshots show | Publication status |
|---|---|---|
| Linux account and SSH paths | Linux FIM paths are recorded in the centralized configuration notes | Configuration documented; per-path alert evidence still required |
| Linux authentication log and cron | Paths and intended monitoring are recorded | Configuration documented; per-path alert evidence still required |
| Windows basic FIM | Public-folder monitoring and test-file activity are shown | Experiment documented; retain the matching event evidence |
| Windows who-data | Configuration stage using `whodata="yes"` is shown | Attribution should be confirmed from the event fields |
| Windows change reporting | Configuration stage using `report_changes="yes"` is shown | Confirm the event contains the expected text diff |

Do not mark a use case as fully validated unless the evidence includes the relevant configuration, the controlled file change, and the corresponding Wazuh event with the expected fields.

## 5. Investigation guidance

For each FIM alert, review:

- Agent and operating system.
- Full file path and file name.
- Event timestamp.
- Change type (created, modified, or deleted).
- User/actor fields when who-data is enabled.
- Hash and metadata changes, without publishing sensitive values.
- Related authentication, process, account, or scheduled-task events.

A file change can be expected maintenance, software activity, or user work. Establish context before escalating it as an incident.

## 6. Related chapters

- [02 — Linux Threat Detection](../02-Linux-Threat-Detection/README.md)
- [06 — Custom Detection Rules](../06-Custom-Detection-Rules/README.md)
- [08 — Incident Investigation](../08-Incident-Investigation/README.md)
- [09 — Evidence and Documentation](../09-Evidence-and-Documentation/README.md)
