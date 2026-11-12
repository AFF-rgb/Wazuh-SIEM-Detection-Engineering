# 09 — Evidence and Documentation

This chapter defines how evidence from the Wazuh lab is reviewed, organized, validated, and prepared for publication. The goal is a portfolio that is technically credible, traceable, and reproducible without inventing results or removing useful technical context.

## 1. Evidence standard

For each validated scenario, aim to retain the following evidence chain:

1. **Context:** lab topology, affected host, software/version, and scenario objective.
2. **Configuration:** relevant agent, manager, decoder, rule, or integration configuration.
3. **Simulation:** the controlled action performed, including the test command or steps.
4. **Source event:** endpoint-native log or original event fields.
5. **Wazuh result:** matching alert with timestamp, rule ID, level, agent, and relevant decoded fields.
6. **Validation:** supporting output such as `wazuh-logtest`, service status, agent synchronization, or verified response result.
7. **Analysis:** concise interpretation, expected versus observed behavior, limitations, and conclusion.

A configuration screenshot alone proves that a setting was displayed; it does not prove that the setting was applied or that the expected event was generated. A `wazuh-logtest` result proves how the supplied sample was processed; it does not independently prove live endpoint collection. State exactly what each item demonstrates.

## 2. Evidence lifecycle

### Capture

- Capture evidence directly from the lab system or application being documented.
- Include enough surrounding context to identify the host, relevant configuration section, event, or command result.
- Keep the original source file unchanged in the working evidence set.
- Record the scenario, source host, capture date, and original filename.
- Avoid cropping out fields needed to understand the result. If a secret is visible, redact only that value.

### Review

- Confirm that the evidence belongs to the stated scenario and chapter.
- Check that configuration, test activity, and resulting event are consistent with one another.
- Compare screenshots with source notes and raw event data when available.
- Identify duplicate, unrelated, incomplete, or illustrative material.
- Mark missing evidence as pending; do not fill gaps with expected output.

### Prepare for publication

- Create a publication copy of the original evidence.
- Apply the narrow redaction policy in Section 4.
- Re-open every edited image and inspect the redacted area at readable zoom.
- Confirm that the redaction is opaque and that the underlying value cannot be recovered from the published image.
- Use descriptive filenames and link evidence from the relevant chapter.
- Keep the original unredacted source private and do not commit it.

### Maintain

- When evidence is replaced or corrected, update its filename/reference and the chapter's evidence status.
- Keep documentation consistent with the actual repository contents.
- Do not label a scenario fully validated unless the evidence chain supports that claim.

## 3. Evidence naming convention

Use consistent, descriptive filenames:

```text
linux-ssh-bruteforce-01-configuration.png
linux-ssh-bruteforce-02-raw-event.txt
linux-ssh-bruteforce-03-wazuh-alert.json
linux-ssh-bruteforce-04-validation.txt
windows-rdp-01-security-event.png
windows-rdp-02-wazuh-alert.png
windows-rdp-03-active-response-result.txt
windows-fim-01-agent-configuration.png
windows-fim-02-dashboard-alert.png
suricata-01-eve-alert.json
suricata-02-wazuh-alert.json
```

Use a numbered prefix to preserve the intended sequence. Keep the original filename in the evidence notes when renaming a supplied screenshot.

## 4. Redaction checklist

Before committing evidence, replace only these secret values with the literal `<REDACTED>`:

- Passwords and other plaintext password values.
- SSH private keys.
- Password hashes and other authentication hashes.

Preserve IP addresses, MAC addresses, hostnames, usernames, timestamps, rule IDs, file paths, and other technical details. Do not broaden redaction to ordinary lab metadata.

For screenshots, cover only the secret value with an opaque redaction. Where text can be edited, replace the value with the literal `<REDACTED>`. After editing, inspect the published copy to ensure the secret is not visible beneath a translucent overlay, in an adjacent panel, or elsewhere in the image.

Do not publish credentials, private-key material, or password hashes. If a secret appears in a configuration file, redact its value while retaining the surrounding setting name and structure where safe.

## 5. Evidence integrity and validation labels

Use these labels consistently in chapter status tables:

| Status | Meaning |
|---|---|
| **Documented** | The procedure or configuration is described in notes or documentation. |
| **Evidence captured** | Relevant source screenshots, logs, or outputs are available. |
| **Partially validated** | Some expected behavior is shown, but one or more parts of the evidence chain are missing. |
| **Validated** | The controlled activity, collected event, matching Wazuh result, and relevant expected fields are supported by evidence. |
| **Pending** | Required evidence or a validation step has not yet been supplied or completed. |

Do not fabricate screenshots, alerts, terminal output, timestamps, or validation results. If a graphic is illustrative rather than captured from the lab, label it clearly as an illustration. If a test has not been run, mark it as pending rather than presenting expected output as observed output.

## 6. Wazuh log evidence notes

Wazuh alert files and event archives serve different purposes:

- `/var/ossec/logs/alerts/alerts.json` contains events that matched rules and met the configured alert threshold.
- `/var/ossec/logs/alerts/alerts.log` contains human-readable alert records.
- `/var/ossec/logs/archives/archives.json` can retain all received events, including events that did not trigger alerts, when JSON archive logging is enabled.

Archive logging is disabled by default in the documented Wazuh manager configuration. Enabling `<logall_json>yes</logall_json>` and restarting the manager writes received events to `archives.json`. Forwarding those events for dashboard search also requires the relevant Filebeat archive configuration. Archive logging can substantially increase storage use; enable it deliberately, monitor disk consumption, and disable it when it is no longer needed for the investigation.

Official references:
- [Wazuh server event logging and archives](https://documentation.wazuh.com/current/user-manual/manager/event-logging.html)
- [Wazuh global configuration: logall and logall_json](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/global.html)

## 7. Supplied archive inventory and publication status

The supplied `wazuh-documentation.zip` archive is the source material for this review. It includes Wazuh deployment and detection notes, Linux and Windows FIM material, Suricata notes and screenshots, and Shuffle/SOAR notes.

| Evidence set | Source material reviewed | Publication status |
|---|---|---|
| Windows FIM | 13 screenshots showing configuration, service activity, Public-folder testing, and Wazuh Dashboard views | Nine screenshots do not visibly expose the specified secret categories. Four screenshots — `6.png`, `8.png`, `10.png`, and `13.png` — contain hash values and require redaction before publication. The screenshots have not yet been added to the repository. |
| Suricata | Suricata notes and 19 screenshots | Screenshots are present in Chapter 05. Raw EVE JSON and matching Wazuh alert JSON should be added when available to strengthen the end-to-end evidence chain. |
| Shuffle/SOAR | Deployment and integration notes | The supplied screenshot folder duplicates the Suricata screenshot set; it is not evidence of a completed Shuffle workflow. Do not present workflow execution or automated response as validated without actual workflow evidence. |
| Active Directory | Domain and policy foundation is documented in Chapter 07 | The supplied archive does not contain a dedicated AD alert evidence set. Wazuh AD detection remains pending until corresponding Windows events and Wazuh alerts are captured. |

This inventory describes what was present in the supplied archive at review time. Update it when new evidence is safely prepared and committed.

## 8. Evidence checklist for each chapter

Before considering a chapter complete, verify:

- [ ] The lab environment and software versions are identified.
- [ ] Configuration excerpts match the documented lab setup.
- [ ] The controlled test is described accurately.
- [ ] Source event evidence is available where applicable.
- [ ] The corresponding Wazuh alert and rule metadata are available.
- [ ] Validation evidence supports the stated outcome.
- [ ] Limitations and unverified behavior are disclosed.
- [ ] Screenshots and logs have been reviewed under the redaction policy.
- [ ] Filenames, links, and status labels match the repository contents.
- [ ] No credentials, private keys, or authentication hashes remain in published material.

## 9. Related chapters

- [01 — Wazuh Deployment](../01-Wazuh-Deployment/README.md)
- [02 — Linux Threat Detection](../02-Linux-Threat-Detection/README.md)
- [03 — Windows Threat Detection](../03-Windows-Threat-Detection/README.md)
- [04 — File Integrity Monitoring](../04-File-Integrity-Monitoring/README.md)
- [05 — Wazuh and Suricata Integration](../05-Wazuh-Suricata-Integration/README.md)
- [06 — Custom Detection Rules](../06-Custom-Detection-Rules/README.md)
- [07 — Active Directory Threat Detection](../07-Active-Directory-Threat-Detection/README.md)
- [08 — Incident Investigation](../08-Incident-Investigation/README.md)
