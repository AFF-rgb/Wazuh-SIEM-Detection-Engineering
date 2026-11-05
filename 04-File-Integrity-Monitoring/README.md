# 04 — File Integrity Monitoring

This section documents Wazuh FIM coverage for Linux files and directories associated with account management, SSH persistence, authentication logging, and scheduled tasks.

## Monitoring objectives

| Path | Security relevance | Recommended review |
|---|---|---|
| `/etc/passwd` | Local account metadata | Unexpected account creation or modification |
| `/etc/shadow` | Local password hashes and account state | Unexpected changes; do not publish file contents |
| `/home/*/.ssh/authorized_keys` | SSH key-based access | New, removed, or modified authorized keys |
| `/var/log/auth.log` | Authentication activity | Unexpected deletion, truncation, or modification |
| `/var/spool/cron` | Scheduled-task persistence | Unexpected cron entry changes |

## Centralized Linux agent configuration

The notes place the following configuration in the Linux agent group's `agent.conf` file on the Wazuh manager:

```xml
<agent_config os="Linux">
  <syscheck>
    <directories check_all="yes" report_changes="yes" realtime="yes">/home/*/.ssh/authorized_keys</directories>
    <directories check_all="yes" realtime="yes">/etc/passwd</directories>
    <directories check_all="yes" realtime="yes">/etc/shadow</directories>
    <directories realtime="yes">/var/spool/cron</directories>
    <directories check_all="yes">/var/log/auth.log</directories>
  </syscheck>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/auth.log</location>
  </localfile>
</agent_config>
```

This is a cleaned-up documentation example based on the notes, not a claim that every option/path has been tested on every Linux distribution.

### Sensitive-file caution

Do not enable content-diff reporting for `/etc/shadow`. It contains password hashes and sensitive account data. Monitor metadata and changes without copying its contents into alert payloads or public evidence. Review whether `report_changes` is appropriate for each monitored path.

Real-time monitoring can increase event volume. Tune paths and frequency to the needs of the lab.

## Validation workflow

1. Confirm the Linux agent is assigned to the intended group.
2. Validate the centralized configuration with the Wazuh configuration verification tool for the installed release.
3. Confirm the agent receives the shared configuration.
4. Record the initial FIM baseline.
5. Make one controlled change to a disposable test file or lab account/key.
6. Confirm the expected FIM alert appears.
7. Review the file path, operation, agent, timestamp, and relevant hashes.
8. Restore the lab state and record the result.

## Detection use cases

- **SSH persistence:** monitor changes to `authorized_keys` and investigate the account and key owner.
- **Account manipulation:** correlate `/etc/passwd` and `/etc/shadow` changes with authentication/audit logs.
- **Log tampering:** correlate changes to authentication logs with missing or unusual authentication activity.
- **Scheduled-task persistence:** monitor cron directories and correlate FIM events with cron/crontab system logs.

A FIM alert proves that a monitored file changed; it does not, by itself, prove malicious intent.

## Validation status

The uploaded notes record successful `verify-agent-conf` output for the Linux group configuration and agent synchronization. They do not provide a complete set of sanitized alert records for every monitored path. Those end-to-end tests should be added before marking each use case as validated.
