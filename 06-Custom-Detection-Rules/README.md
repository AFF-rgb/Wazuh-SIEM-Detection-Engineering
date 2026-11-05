# 06 — Custom Detection Rules

This section is the central index for custom Wazuh rules, decoders, correlation logic, and rule validation.

## Rule engineering principles

- Confirm the raw event is being collected before writing a rule.
- Identify the actual decoder output and built-in rule ID for the installed Wazuh version.
- Use unique custom rule IDs and check for collisions.
- Scope correlation to the correct agent, source IP, user, or other stable event field.
- Use a time window that reflects the behavior being detected.
- Add MITRE ATT&CK metadata only when the behavior is genuinely represented by the technique.
- Test with `wazuh-logtest` and then validate with a real endpoint-generated event.
- Review false positives and document known limitations.

## Custom rules recorded in the lab notes

| Custom ID | Intended behavior | Notes status |
|---|---|---|
| 100003 | Cron/crontab persistence activity | Draft decoder/rule; validate event semantics |
| 100010 | SSH brute force followed by successful login | Draft; verify parent/base rule IDs and correlation |
| 100011 | Sudo-based root shell activity | Draft; validate decoded fields and matching logic |
| 100012 | Local account creation | Draft; verify the base rule ID |
| 100013 | Authentication log tampering | Draft; verify FIM group and event fields |
| 100020 | Failed RDP logon (Logon Type 10) | Draft; validate Windows field names |
| 100021 | Successful RDP logon (Logon Type 10) | Draft; validate Windows field names |
| 100030 | Possible network scanning | Draft; verify the source event and correlation |
| 100100 | Repeated Windows failed logons | Draft; verify base rule ID and correlation scope |
| 100900 | RDP brute-force threshold for Active Response | Draft; verify source-IP grouping and response safety |

These IDs are taken from the user's lab notes. They are not asserted to be valid or active in the current Wazuh deployment.

## Example correlation design

The following is a conceptual pattern only. Replace the base SID with the exact rule ID observed in the lab and confirm that the source IP is decoded. Validate the syntax against the installed Wazuh release.

```xml
<group name="windows,bruteforce,">
  <rule id="100100" level="10" frequency="5" timeframe="60">
    <if_matched_sid>REPLACE_WITH_VERIFIED_BASE_SID</if_matched_sid>
    <same_source_ip />
    <description>Repeated Windows logon failures from the same source within 60 seconds</description>
  </rule>
</group>
```

A threshold without source-IP or equivalent grouping can combine events from different sources and produce misleading results. If the event does not contain a usable source IP, select another reliable correlation key and document the limitation.

## Test procedure

Run the log test utility on the Wazuh manager:

```bash
sudo /var/ossec/bin/wazuh-logtest
```

Paste a sanitized representative raw event and record:
- Decoder name and extracted fields.
- Built-in rule ID and description.
- Custom rule ID and description, if it fires.
- Any missing fields or unexpected matching behavior.

Then generate the event from the lab endpoint and verify the alert in the dashboard or `/var/ossec/logs/alerts/alerts.json`.

## Rule publication status

A rule is **validated** only after both log-test validation and an endpoint-generated alert are recorded. XML syntax validation alone is insufficient.
