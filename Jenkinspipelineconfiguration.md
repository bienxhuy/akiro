# Jenkins Pipeline Configuration

This document covers all Jenkins configuration required outside the `Jenkinsfile` itself — including credentials, webhook setup, Smee.io relay.

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
| `influxdb-token` | Secret text | InfluxDB API token with write access to the `devtest_metrics` bucket |

---

## 3. Webhook Setup (Push Trigger)

The push-mode pipeline is triggered automatically when a commit is pushed to GitHub. Because Jenkins runs locally without a public IP, a relay is required.

### 3.1 Generic Webhook Trigger Configuration

In the Jenkins pipeline job, under **Build Triggers → Generic Webhook Trigger**:

1. Enable **Generic Webhook Trigger**.

2. Add the following **Post content parameters**:

| Variable | Expression | Expression type | Value filter |
|----------|------------|-----------------|--------------|
| `BRANCH` | `$.ref` | JSONPath | `refs/heads/(.*)` |
| `AUTHOR` | `$.pusher.name` | JSONPath | |

3. Set a **Token** to restrict which requests can trigger this pipeline. The trigger URL becomes:
   ```
   http://localhost:8080/generic-webhook-trigger/invoke?token=your-token
   ```

4. Set **Cause** to:
   ```
   $AUTHOR committed to $BRANCH
   ```

5. Enable **Print contributed variables** for easier debugging.

### 3.2 Smee.io Relay

1. Go to [https://smee.io](https://smee.io) and click **Start a new channel**. Copy the generated Smee URL (e.g., `https://smee.io/xxxxxxxxxxxxxxxx`).

2. Install the Smee client on the host (if not already installed):
   ```bash
   npm install --global smee-client
   ```

3. Start the Smee client, forwarding to the Jenkins trigger URL:
   ```bash
   smee --url https://smee.io/xxxxxxxxxxxxxxxx --target http://localhost:8080/generic-webhook-trigger/invoke?token=your-token
   ```

   Run this as a background process or configure it as a system service to keep it alive persistently.

### 3.3 GitHub Webhook

1. In your repository on GitHub, go to **Settings → Webhooks → Add webhook**.
2. Set **Payload URL** to your Smee channel URL (e.g., `https://smee.io/xxxxxxxxxxxxxxxx`).
3. Set **Content type** to `application/json`.
4. Select **Just the push event**.
5. Click **Add webhook**.

---

## 4. Pipeline Job Setup

1. In Jenkins, create a new **Pipeline** job.
2. Under **Pipeline → Definition**, select **Pipeline script from SCM**.
3. Set **SCM** to Git and enter the URL of this repository (`akiro`).
4. Set **Script Path** to `Jenkinsfile`.
5. Save the job.
