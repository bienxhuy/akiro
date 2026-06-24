# Jenkins Pipeline Configuration

This document covers all Jenkins configuration required outside the `Jenkinsfile` itself — including credentials, webhook setup, Smee.io relay, and the cron schedule.

---

## 1. Required Jenkins Plugins

Install the following plugins via **Manage Jenkins → Plugins**:

| Plugin | Purpose |
|--------|---------|
| Generic Webhook Trigger | Parses incoming webhook payloads and populates pipeline parameters |
| Pipeline | Enables declarative `Jenkinsfile` support |
| Git | Allows Jenkins to clone repositories |

---

## 2. Credentials

All secrets are stored in Jenkins Credentials and referenced by ID in the `Jenkinsfile`. Navigate to **Manage Jenkins → Credentials → (global)** and add the following:

| Credential ID | Kind | Value |
|---------------|------|-------|
| `github-token` | Secret text | GitHub personal access token with `repo` scope — used to clone private repositories |
| `influxdb-token` | Secret text | InfluxDB API token with write access to the `devtest_metrics` bucket |

---

## 3. Webhook Setup (Push Trigger)

The push-mode pipeline is triggered automatically when a commit is pushed to GitHub. Because Jenkins runs locally without a public IP, a relay is required.

### 3.1 Smee.io Relay

1. Go to [https://smee.io](https://smee.io) and click **Start a new channel**. Copy the generated Smee URL (e.g., `https://smee.io/xxxxxxxxxxxxxxxx`).

2. Install the Smee client on the host machine:
   ```bash
   npm install --global smee-client
   ```

3. Start the Smee client, forwarding to your local Jenkins webhook endpoint:
   ```bash
   smee --url https://smee.io/xxxxxxxxxxxxxxxx --target http://localhost:8080/generic-webhook-trigger/invoke?token=your-token
   ```

   Run this as a background process or configure it as a system service to keep it alive persistently.

### 3.2 GitHub Webhook

1. In your SUT or source repository on GitHub, go to **Settings → Webhooks → Add webhook**.
2. Set **Payload URL** to your Smee channel URL (e.g., `https://smee.io/xxxxxxxxxxxxxxxx`).
3. Set **Content type** to `application/json`.
4. Select **Just the push event**.
5. Click **Add webhook**.

### 3.3 Generic Webhook Trigger Configuration

In the Jenkins pipeline job, under **Build Triggers → Generic Webhook Trigger**:

- Enable **Generic Webhook Trigger**.
- Add the following **Post content parameters** to extract values from the GitHub push payload:

| Variable | Expression | Expression type |
|----------|------------|-----------------|
| `BRANCH_NAME` | `$.ref` | JSONPath |
| `COMMIT_AUTHOR` | `$.head_commit.author.name` | JSONPath |

- Optionally set a **Token** to restrict which requests can trigger this pipeline.

---

## 4. Scheduled Trigger (Nightly Mode)

In the Jenkins pipeline job, under **Build Triggers → Build periodically**, enter a cron expression. Example for nightly at 2:00 AM:

```
H 2 * * *
```

The `Jenkinsfile` detects this trigger and sets `RUN_TYPE=nightly`, executing the full test suite.

---

## 5. Pipeline Job Setup

1. In Jenkins, create a new **Pipeline** job.
2. Under **Pipeline → Definition**, select **Pipeline script from SCM**.
3. Set **SCM** to Git and enter the URL of this repository (`akiro`).
4. Set **Script Path** to `Jenkinsfile`.
5. Save the job.

---

## 6. Configuration Summary

| Item | Location | Status to verify |
|------|----------|-----------------|
| Generic Webhook Trigger plugin | Manage Jenkins → Plugins | Installed |
| `github-token` credential | Manage Jenkins → Credentials | Added |
| `influxdb-token` credential | Manage Jenkins → Credentials | Added |
| Smee.io client | Host machine | Running |
| GitHub webhook | Repository Settings → Webhooks | Active, green checkmark |
| Generic Webhook Trigger | Pipeline job → Build Triggers | Enabled with parameters |
| Cron schedule | Pipeline job → Build Triggers | Set |
| Pipeline SCM | Pipeline job → Pipeline | Pointing to this repo |