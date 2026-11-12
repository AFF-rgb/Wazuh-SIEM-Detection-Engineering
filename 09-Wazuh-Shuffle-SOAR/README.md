# 09 — Wazuh and Shuffle SOAR

This chapter documents the SOAR deployment foundation recorded in the lab archive and defines a reproducible plan for integrating Wazuh with Shuffle. The source notes cover Docker-based Shuffle deployment and installation prerequisites for TheHive and Cassandra. They do **not** contain sufficient evidence to claim a completed Wazuh-to-Shuffle alert workflow, automated enrichment, case creation, or response execution.

## Objective and architecture

The intended workflow is:

1. Wazuh detects activity and generates an alert.
2. Wazuh forwards selected alert JSON to a Shuffle webhook.
3. Shuffle receives and parses the alert.
4. A workflow performs approved enrichment, notification, or case-management actions.
5. The workflow records execution outcome and any response action.

| Component | Role | Evidence in the supplied archive |
|---|---|---|
| Wazuh | SIEM and alert source | Separate Wazuh deployment and alerting notes |
| Shuffle | SOAR workflow platform | Docker Compose deployment instructions; no independent workflow execution evidence |
| TheHive | Security incident/case management | Installation prerequisites and setup notes |
| Cassandra | Database dependency in the recorded TheHive setup | Repository installation and service configuration notes |

## 1. Docker prerequisites

The archive records installation of Docker Engine and Docker Compose using Ubuntu packages:

```bash
sudo apt update
sudo apt install docker.io docker-compose
```

Confirm the installed Docker and Compose versions before deploying. The original notes use the legacy `docker-compose` command; newer Docker installations may use the Compose plugin (`docker compose`). Use the syntax supported by the installed version.

## 2. Deploy Shuffle

The recorded deployment procedure clones the Shuffle repository and starts the Compose project:

```bash
git clone https://github.com/Shuffle/Shuffle.git
cd Shuffle
sudo docker-compose up -d
sudo docker ps
```

Open the Shuffle web interface on the locally configured port (the notes record port 3001) and create a dedicated lab account.

**Data-loss warning:** the source notes also contain `docker-compose down -v`. The `-v` option removes named volumes declared by the Compose project and can delete persistent application data. Do not run it as a routine restart command. Back up required data and understand the Compose volume configuration before removing volumes.

For a deployment review, record:
- Docker Engine and Compose versions.
- The Compose file and image versions used.
- Container state and health checks.
- Published ports and network bindings.
- Persistent volumes and backup/restore approach.
- How the web interface is protected and who can access it.

Do not expose the Shuffle management interface or webhook endpoint publicly without appropriate network restrictions, TLS, and access controls.

## 3. TheHive and Cassandra prerequisites

The archive records installation of Java 11 (Amazon Corretto) and Cassandra as prerequisites for the TheHive deployment. It also includes a custom Cassandra systemd unit and heap-size adjustments.

These are historical setup notes, not a verified current installation recipe. Before reusing them:
- Check the supported Java and database versions for the exact TheHive release.
- Follow the current official TheHive installation documentation and supported database architecture.
- Verify repository signing keys and package sources.
- Validate the service account, permissions, systemd unit, startup order, and persistent data paths.
- Do not copy the custom service unit or heap settings into production without testing.

The archive does not include a complete TheHive configuration, successful service-health transcript, or a validated connection to Shuffle.

## 4. Wazuh-to-Shuffle integration design

Wazuh provides a built-in Shuffle integration through its Integrator module. The manager sends matching alert data in JSON format to a Shuffle webhook. The following is a **configuration template**, not a record of a configuration applied in this lab.

Add the integration block inside the existing `<ossec_config>` element in `/var/ossec/etc/ossec.conf` on the Wazuh manager:

```xml
<integration>
  <name>shuffle</name>
  <hook_url>http://SHUFFLE_HOST:3001/api/v1/hooks/WEBHOOK_ID</hook_url>
  <level>10</level>
  <alert_format>json</alert_format>
</integration>
```

Replace `SHUFFLE_HOST` and `WEBHOOK_ID` with the actual host and webhook identifier. The example uses a plain URL placeholder so the XML remains well-formed; use HTTPS when TLS is configured and trusted. Do not commit the real webhook URL or webhook ID: treat the complete URL as a secret because it can authorize workflow invocation.

Configuration notes:
- `name` selects Wazuh's built-in Shuffle integration.
- `hook_url` is the Shuffle webhook endpoint.
- `level` is an optional minimum alert-level filter. The example uses level 10 to limit forwarding; select a threshold appropriate to the lab objective.
- `alert_format` must be `json` for this integration.
- Wazuh also supports filters such as `rule_id` and `group`; prefer a narrow filter for a controlled workflow rather than forwarding every alert unnecessarily.

After editing the manager configuration, validate the XML/configuration and restart the Wazuh manager using the service command appropriate to the installed system:

```bash
sudo systemctl restart wazuh-manager
sudo systemctl status wazuh-manager --no-pager
```

The restart command above is an implementation step for a future test, not evidence that the integration has been configured or restarted in this lab.

## 5. Shuffle workflow design and safe testing

Create a workflow in Shuffle and add a Webhook trigger. Copy its generated webhook URL into the Wazuh manager configuration, then start the webhook trigger. For an initial connectivity test, use a workflow that only receives and displays or repeats the incoming alert payload.

Recommended staged validation:

1. **Webhook readiness:** confirm the Shuffle workflow is saved and the webhook trigger is running.
2. **Wazuh configuration:** confirm the integration block is in the manager's active `ossec.conf`, with the intended filter and JSON format.
3. **Controlled alert:** generate a known lab event that produces an alert meeting the configured filter.
4. **Delivery:** confirm the corresponding alert appears in Shuffle and compare its rule ID, timestamp, agent, and event fields with the Wazuh alert.
5. **Workflow execution:** capture a successful Shuffle execution and inspect its input/output.
6. **Enrichment:** add one non-destructive enrichment step and verify its result.
7. **Case management:** only after the previous stages work, test TheHive case creation using a dedicated integration identity.
8. **Response:** test response actions separately, with explicit approval, least privilege, and a documented rollback.

Do not use account disabling, host isolation, deletion, or other disruptive actions as the first integration test. Begin with receive-only workflow behavior.

## 6. Troubleshooting and operational controls

If an alert does not reach Shuffle, check the integration in layers:

- Confirm the Wazuh alert exists in `/var/ossec/logs/alerts/alerts.json` and meets the configured level/rule/group filter.
- Confirm the webhook URL is correct and the Shuffle trigger is running.
- Check Wazuh manager logs under `/var/ossec/logs/` for Integrator errors.
- Check Shuffle workflow execution history and webhook input.
- Verify name resolution, routing, firewall rules, TLS certificate trust, and connectivity from the Wazuh manager to the Shuffle endpoint, including any proxy or network policy that could block the request.

Operational safeguards:
- Use a dedicated integration identity and minimum required permissions.
- Store webhook URLs, API tokens, and passwords outside Git and workflow source text.
- Restrict inbound access to Shuffle and protect the webhook URL.
- Filter alerts to reduce noise and avoid unnecessary workflow executions.
- Add error handling, retries, duplicate-event handling, and audit logging before relying on automation.
- Require human approval for high-impact response actions.
- Record how to disable the integration and roll back a response.

## 7. Validation checklist

- [ ] Record Docker Engine and Compose versions.
- [ ] Confirm Shuffle containers are running and healthy.
- [ ] Confirm Shuffle persists data across a normal restart.
- [ ] Confirm the Shuffle webhook trigger is saved and running.
- [ ] Configure the Wazuh integration on the manager using a protected webhook URL.
- [ ] Confirm the Wazuh manager restarts successfully after configuration.
- [ ] Generate a controlled alert that matches the integration filter.
- [ ] Confirm the alert reaches Shuffle and compare its fields with the original Wazuh alert.
- [ ] Capture the Shuffle workflow execution result.
- [ ] If TheHive is integrated, confirm a test case is created with the correct alert context.
- [ ] Document error handling, duplicate-event handling, retries, and audit logs.
- [ ] Test any response action with explicit approval, least privilege, and a rollback procedure.

## 8. Credential and evidence handling

The original SOAR notes contain an example login credential in plaintext. It is deliberately omitted from this repository. If it was ever a real or reused credential, change it immediately and review whether it was stored in browser history, screenshots, shell history, or other files.

Before publication, protect passwords and SSH private keys by replacing them with the literal `<REDACTED>`. Hash values that are part of controlled demonstrations may remain visible, as agreed for this portfolio. Preserve IP addresses, MAC addresses, hostnames, usernames, timestamps, and other technical details. Do not commit webhook URLs, API tokens, session cookies, or other credentials; store integration secrets in an appropriate secret-management facility rather than workflow text or Git.

## Current documentation and evidence status

- **Documented:** Docker-based Shuffle deployment steps, TheHive/Cassandra installation prerequisites, and a Wazuh-to-Shuffle integration design.
- **Evidence available:** text instructions for Shuffle deployment and TheHive/Cassandra prerequisites. The screenshot folder labelled for Shuffle duplicates the Suricata screenshots; it does not provide independent SOAR evidence.
- **Pending validation:** current container health, configured webhook delivery, Wazuh alert ingestion, Shuffle workflow execution, TheHive case creation, and any automated response.

This chapter is a deployment and validation guide. It does not claim that the Wazuh–Shuffle integration or SOAR response has been completed in the lab.

## References

- [Wazuh — External API integration and Shuffle setup](https://documentation.wazuh.com/current/user-manual/manager/integration-with-external-apis.html)
- [Wazuh — Integration configuration reference](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/integration.html)
- [Shuffle — Wazuh webhook extension](https://shuffle.security/docs/extensions)
- [Shuffle source repository](https://github.com/Shuffle/Shuffle)
- [TheHive documentation](https://docs.strangebee.com/thehive/)
