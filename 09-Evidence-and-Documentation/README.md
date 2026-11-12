# 09 — Evidence and Documentation

This section contains sanitized evidence and supporting documentation for the Wazuh lab projects.

## Evidence standard

For each validated scenario, aim to include:

1. Lab topology or a concise architecture description.
2. Relevant configuration excerpt with secrets and environment-specific values removed.
3. The controlled simulation description.
4. Raw event or event fields.
5. Wazuh alert showing the rule ID and severity.
6. Validation output, such as `wazuh-logtest` or agent synchronization status.
7. A short analyst interpretation and limitations.

## Evidence naming convention

Use consistent, descriptive filenames, for example:

```text
linux-ssh-bruteforce-01-raw-event.txt
linux-ssh-bruteforce-02-wazuh-alert.json
linux-ssh-bruteforce-03-validation.txt
windows-rdp-01-security-event.png
windows-rdp-02-wazuh-alert.png
```

## Redaction checklist

Before committing evidence, replace only the following values with the literal `<REDACTED>`:
- Passwords and other plaintext password values.
- SSH private keys.
- Password hashes and other authentication hashes.

Preserve IP addresses, MAC addresses, hostnames, usernames, timestamps, and other technical details. Do not broaden redaction to ordinary lab metadata. If a screenshot or log contains one of the three secret categories above, cover only the secret value with an opaque redaction and use the literal `<REDACTED>` where text replacement is possible.

## Evidence integrity

Do not fabricate screenshots, alerts, terminal output, timestamps, or validation results. If a screenshot is illustrative rather than captured from the lab, label it clearly as an illustration. If a test has not been run, mark it as pending instead of presenting expected output as observed output.

The archive supplied for this project is the primary source for the documented lab work. Preserve the original technical context when preparing evidence for publication; apply the narrow redaction policy above.
