# SSH Brute-Force Detection and Successful-Login Correlation

## Objective

Document the Linux SSH authentication scenario from the Wazuh lab: repeated failed authentication attempts followed by a successful login. The investigation goal is to identify suspicious authentication sequences and correlate them with endpoint and source information.

## Lab telemetry

The source notes configure the Linux agent to collect `/var/log/auth.log`:

```xml
<localfile>
  <location>/var/log/auth.log</location>
  <log_format>syslog</log_format>
</localfile>
```

Confirm that the target Linux distribution writes SSH authentication events to this path. The location can differ by distribution and logging configuration.

## Controlled simulation

The notes describe using a dedicated lab account and a password list from the isolated Kali VM. Run authentication testing only against the lab endpoint and accounts you control.

```bash
hydra -l <LAB_TEST_USER> -P <LAB_PASSWORD_LIST> ssh://<LAB_AGENT_IP>
```

Do not use real user passwords or publish password lists, authentication secrets, or private keys.

## Detection design recorded in the notes

The custom rule ID recorded for this scenario is **100010**:

```xml
<group name="SSH,">
  <rule id="100010" level="12">
    <if_matched_sid>5712</if_matched_sid>
    <if_sid>5715</if_sid>
    <same_source_ip />
    <description>SSH brute force followed by successful login.</description>
    <mitre>
      <id>T1110.001</id>
    </mitre>
  </rule>
</group>
```

This is a source-note design, not a verified drop-in rule. Before enabling it, use the installed Wazuh ruleset and actual decoded events to confirm:
- Which built-in rule ID matches a failed SSH authentication.
- Which built-in rule ID matches a successful SSH authentication.
- Whether the rule structure correctly expresses the intended event sequence.
- Whether the source IP field is decoded and the correlation key is supported.
- Whether a bounded correlation window is required and correctly implemented.

Do not assume the numeric SIDs in older notes apply to a different Wazuh release.

## Validation procedure

Run on the Wazuh manager:

```bash
sudo /var/ossec/bin/wazuh-logtest
```

Record Phase 1 raw event recognition, Phase 2 decoder and extracted fields, and Phase 3 rule evaluation. Then generate the controlled event sequence on the endpoint and confirm the corresponding alert in the dashboard or:

```bash
sudo tail -f /var/ossec/logs/alerts/alerts.json
```

For a credible validation record, retain the failed-login event, successful-login event, custom alert, source address, affected account, agent, and timestamps. Sanitize passwords, SSH keys, and hashes using the literal `<REDACTED>`; preserve other technical details.

## Investigation guidance

A sequence of failed logins followed by a successful login from the same source can indicate password guessing, but it is not conclusive by itself. Check:
- Whether the successful login came from the same source.
- Whether the account owner recognizes the activity.
- Whether the login was followed by sudo activity, account creation, or SSH key changes.
- Whether the source is an approved administrator, scanner, or automation host.
- Whether the endpoint and source timestamps align.

## MITRE ATT&CK

- **T1110.001 — Password Guessing:** repeated password authentication attempts.

## Evidence status

The archive contains the scenario description, rule snippet, and test procedure. It does not include a complete `wazuh-logtest` Phase 1–3 transcript and matching live Wazuh alert for this rule. Therefore, the rule remains **documented; end-to-end validation pending**.

## Related documentation

- [Linux Threat Detection](../README.md)
- [Custom Detection Rules](../../06-Custom-Detection-Rules/README.md)
- [File Integrity Monitoring](../../04-File-Integrity-Monitoring/README.md)
- [Incident Investigation](../../08-Incident-Investigation/README.md)