# 10 — Wazuh and Shuffle SOAR

This chapter documents the SOAR deployment foundation recorded in the lab archive. The notes cover Docker-based Shuffle deployment and installation prerequisites for TheHive and Cassandra. They do not contain sufficient evidence to claim a completed Wazuh-to-Shuffle alert workflow, automated enrichment, case creation, or response execution.

## Objective

Explore security orchestration by connecting Wazuh alerts to a workflow platform and, where implemented, case-management and analysis services.

## Lab components recorded

| Component | Purpose in the planned architecture | Evidence in the archive |
|---|---|---|
| Wazuh | SIEM and alert source | Separate Wazuh deployment and alerting notes |
| Shuffle | SOAR workflow platform | Docker Compose deployment instructions and screenshots of the environment |
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

## 3. TheHive and Cassandra prerequisites

The archive records installation of Java 11 (Amazon Corretto) and Cassandra as prerequisites for the TheHive deployment. It also includes a custom Cassandra systemd unit and heap-size adjustments.

These are historical setup notes, not a verified current installation recipe. Before reusing them:
- Check the supported Java and Cassandra versions for the exact TheHive release.
- Follow the current official TheHive installation documentation and supported database architecture.
- Verify repository signing keys and package sources.
- Validate the service account, permissions, systemd unit, startup order, and persistent data paths.
- Do not copy the custom service unit or heap settings into production without testing.

The archive does not include a complete TheHive configuration, successful service-health transcript, or a validated connection to Shuffle.

## 4. Wazuh integration — implementation still to be evidenced

The intended integration architecture is:

1. Wazuh generates a security alert.
2. The alert is delivered to a Shuffle trigger or integration endpoint.
3. A Shuffle workflow parses and enriches the alert.
4. The workflow optionally creates or updates a case in TheHive.
5. The workflow records execution status and any analyst notification or response action.

The supplied notes do not include the actual Wazuh integration configuration, webhook/API credential setup, a saved Shuffle workflow, a successful test execution, or a resulting TheHive case. Therefore, these steps are an architecture and validation plan, not claims of completed functionality.

## 5. Validation checklist

- [ ] Record Docker Engine and Compose versions.
- [ ] Confirm Shuffle containers are running and healthy.
- [ ] Confirm Shuffle persists data across a normal restart.
- [ ] Confirm TheHive and its required database services are supported and healthy for the selected versions.
- [ ] Configure a dedicated integration identity with least privilege.
- [ ] Configure the Wazuh alert trigger in Shuffle.
- [ ] Send a controlled test alert and confirm it reaches the workflow.
- [ ] Capture the workflow execution result and relevant event fields.
- [ ] If TheHive is integrated, confirm a test case is created with the correct alert context.
- [ ] Document error handling, duplicate-event handling, retries, and audit logs.
- [ ] Test any response action with an explicit approval/safety control and a rollback procedure.

## 6. Credential and evidence handling

The original SOAR notes contain an example login credential in plaintext. It is deliberately omitted from this repository. If it was ever a real or reused credential, change it immediately and review whether it was stored in browser history, screenshots, shell history, or other files.

Before publication, replace only passwords, SSH private keys, and hashes with the literal `<REDACTED>`. Preserve IP addresses, MAC addresses, hostnames, usernames, timestamps, and other technical details. Do not commit API tokens, session cookies, or other credentials; store integration secrets in the platform's secret-management facility rather than workflow text or Git.

## Current documentation status

- **Documented:** Docker-based Shuffle deployment steps and TheHive/Cassandra installation prerequisites.
- **Evidence available:** text instructions for Shuffle deployment and TheHive/Cassandra prerequisites. The screenshot folder labelled for Shuffle duplicates the Suricata screenshots; it does not provide independent SOAR evidence.
- **Needs validation:** current container health, working Wazuh alert ingestion, Shuffle workflow execution, TheHive case creation, and any automated response.

## References

- [Shuffle source repository](https://github.com/Shuffle/Shuffle)
- [TheHive documentation](https://docs.strangebee.com/thehive/)
- [Wazuh documentation](https://documentation.wazuh.com/current/)