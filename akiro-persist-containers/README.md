# Persistent Containers

This directory contains the Docker Compose file for the four services that run persistently on the host machine — independent of any individual pipeline run.

---

## Services

| Container | Image | Verified version | Port | Purpose |
|-----------|-------|-----------------|------|---------|
| `influxdb3` | `influxdb:3.3.0` | 3.3.0 | `8181` | Time-series database storing all test execution metrics (build summaries, suite results, failed tests, performance data) |
| `grafana` | `grafana/grafana-oss:12.1.0` | 12.1.0 | `9090 → 3000` | Dashboard for visualizing test results, build trends, and quality metrics |
| `devpi` | `muccg/devpi` | devpi-server 4.5.0 (no versioned tags published by maintainer) | `3141` | Local Python package mirror — caches pip dependencies for the test framework container, reducing repeated downloads from PyPI |
| `verdaccio` | `verdaccio/verdaccio:6.2.0` | 6.2.0 | `4873` | Local npm package mirror — caches Node.js dependencies for the SUT containers |

All four containers are connected to the same Docker network as the per-pipeline containers, allowing the test framework to write results to InfluxDB by container name (`influxdb3`) without additional network configuration.

---

## Starting the Services

```bash
cd akiro-persist-containers
docker compose up -d
```

To stop without removing data volumes:

```bash
docker compose stop
```

To stop and remove all data (destructive — clears all InfluxDB metrics and Grafana configuration):

```bash
docker compose down -v
```

---

## Grafana Access

Once running, Grafana is available at [http://localhost:9090](http://localhost:9090).

Default credentials (as configured in `docker-compose.yml`):

| Field | Value |
|-------|-------|
| Username | `akiro` |
| Password | `akiro2026` |

> Change these credentials in `docker-compose.yml` before deploying in any shared or production environment.

After first login, follow the steps in `../grafana-source/README.md` to connect InfluxDB as a data source and import the dashboard pages.

---

## InfluxDB Access

InfluxDB is available at [http://localhost:8181](http://localhost:8181).

Data is persisted in `./influxdb/data` on the host. The node is configured with ID `devtestdb` and uses local file object storage.

### Creating the Operator Token

InfluxDB 3 Core uses token-based authorization — all CLI commands and HTTP requests require a token. After starting the containers for the first time, create the operator token (named `_admin`, grants access to all actions):

```bash
docker exec influxdb3 influxdb3 create token --admin
```

Copy the token from the output — **it will not be shown again.**

To use the token in subsequent CLI commands inside the container, set the environment variable:

```bash
export INFLUXDB3_AUTH_TOKEN=YOUR_TOKEN
```

Or pass it inline per command with `--token YOUR_TOKEN`.

Once you have the token, store it in:
- Jenkins Credentials as `influxdb-token` (see `../JenkinsPipelineConfiguration.md`)
- Grafana data source configuration (see `../grafana-source/README.md`)

---

## Notes

- Data for all services is persisted via bind mounts under `./influxdb`, `./grafana`, `./devpi`, and `./verdaccio` relative to this directory. These directories are created automatically on first run.
- `grafana` depends on `influxdb3` and will start after it is ready.
- All other services start independently.