# Grafana Source

This directory contains the exported JSON files for all ATI dashboard pages, along with instructions for connecting InfluxDB as a data source and importing the dashboards.

---

## Dashboard Pages

| File | Dashboard page | Purpose |
|------|---------------|---------|
| `home.json` | Home | Latest build health and historical pass rate / execution time trends |
| `build-detail.json` | Build detail | Single build overview with per-suite breakdown |
| `xml-suite-detail.json` | XML suite detail | Failed and unstable test list for a selected functional suite |
| `performance-detail.json` | Performance detail | Load test metrics: RPS, latency percentiles, error rate, per-endpoint breakdown |
| `test-detail.json` | Test detail | Failure message and execution logs for a single failing test |

Navigation between pages is handled automatically via Grafana data links — clicking a build ID, suite, or test row on one page passes the relevant context variable to the next.

---

## Step 1 — Configure InfluxDB Data Source

1. Open Grafana at [http://localhost:9090](http://localhost:9090) and log in.
2. Go to **Connections → Data sources → Add new data source**.
3. Select **InfluxDB**.
4. Configure the data source with the following values:

| Field | Value |
|-------|-------|
| Name | `InfluxDB` |
| Query language | `SQL` |
| URL | `http://influxdb3:8181` |
| Token | Your InfluxDB API token (same as `influxdb-token` in Jenkins Credentials) |
| Database | `devtest_metrics` |

5. Click **Save & Test**. You should see a confirmation that the data source is working.

> The URL `http://influxdb3:8181` works when Grafana and InfluxDB are on the same Docker network (as configured in `../persist-containers/docker-compose.yml`). If running Grafana outside Docker, use `http://localhost:8181` instead.

---

## Step 2 — Import Dashboard Pages

Import each JSON file individually:

1. In Grafana, go to **Dashboards → Import**.
2. Click **Upload dashboard JSON file** and select a file from this directory.
3. Select the `InfluxDB` data source configured in Step 1 when prompted.
4. Click **Import**.

Repeat for all five JSON files. The recommended import order follows the navigation hierarchy:

1. `home.json`
2. `build-detail.json`
3. `xml-suite-detail.json`
4. `performance-detail.json`
5. `test-detail.json`

---

## Step 3 — Verify Navigation Links

After importing all pages, open the **Home** dashboard and verify that:

- The build history chart is populated (requires at least one completed pipeline run).
- Clicking a build ID navigates to the **Build detail** page.
- Navigating from build detail to suite detail and test detail works as expected.

If data links appear broken after import, check that the dashboard UIDs match the ones referenced in each JSON file's data link configuration. Re-exporting and re-importing the affected pages typically resolves UID mismatches.

---

## Exporting Updated Dashboards

When modifying a dashboard, export the updated JSON to keep this directory in sync:

1. Open the dashboard in Grafana.
2. Go to **Dashboard settings (gear icon) → JSON Model**.
3. Copy the JSON and replace the corresponding file in this directory.

Commit the updated file to version-control the dashboard change.