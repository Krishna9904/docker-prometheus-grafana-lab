# Prometheus + Grafana Monitoring Lab

A small but realistic infrastructure monitoring stack built with Docker Compose. It shows how metrics are collected, stored, visualized, and alerted on in a Linux environment, running on Windows through Docker Desktop.

## Architecture

```
Node Exporter (:9100)
        │  scraped every 15s
        ▼
Prometheus (:9090)
        │
        ├──► Grafana (:3000)          dashboards via PromQL
        └──► Alertmanager (:9093)     receives FIRING alerts
```

| Service | Port | Role |
|---|---|---|
| Node Exporter | 9100 | Exposes CPU, memory, filesystem, network and load metrics at `/metrics` |
| Prometheus | 9090 | Scrapes and stores time-series data, runs PromQL, evaluates alert rules |
| Grafana | 3000 | Dashboards and visualization on top of Prometheus |
| Alertmanager | 9093 | Receives alerts from Prometheus and groups them |

All containers share a Docker network called `monitoring`, so they reach each other by service name (for example `prometheus → node-exporter:9100`). `localhost` would not work inside the Prometheus container because it points back at Prometheus itself.

## Quick start

**Requirements:** Docker Desktop (Windows, macOS or Linux).

```bash
git clone <your-repo-url>
cd <your-repo-folder>
docker compose up -d
```

| Tool | URL | Login |
|---|---|---|
| Prometheus | http://localhost:9090 | none |
| Grafana | http://localhost:3000 | `admin` / `admin` (lab only, change it for anything real) |
| Alertmanager | http://localhost:9093 | none |

Check that both targets show **UP** at http://localhost:9090/targets.

## Prometheus configuration

- `scrape_interval: 15s` and `evaluation_interval: 15s`
- Two scrape jobs: `prometheus` (`prometheus:9090`) and `node` (`node-exporter:9100`)
- Alerts are sent to `alertmanager:9093`

## Example PromQL queries

```promql
# Is the target reachable? (1 = up, 0 = down)
up{job="node"}

# CPU usage %
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage %
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100

# Network traffic (bytes/s)
rate(node_network_receive_bytes_total[5m])
rate(node_network_transmit_bytes_total[5m])
```

**Counters vs. gauges:** `node_cpu_seconds_total` and the network metrics are counters that only ever increase, so we use `rate()` to get their rate of change. Memory metrics are gauges that move up and down, so they are used directly.

## Grafana dashboard

The provisioned Linux infrastructure dashboard has panels for node status, CPU, memory, network and system load. Grafana uses Prometheus (`http://prometheus:9090`) as its datasource.

## Alert rules

| Alert | Condition | For | Severity |
|---|---|---|---|
| `NodeExporterDown` | Node Exporter unreachable (`up{job="node"} == 0`) | 1m | critical |
| `HighCPUUsage` | CPU > 80% | 5m | warning |
| `HighMemoryUsage` | Memory > 85% | 5m | |
| `FilesystemAlmostFull` | Filesystem > 80% | 10m | |

The `for:` duration means a condition must stay true continuously before the alert moves from **PENDING** to **FIRING**. This avoids alerts from short, transient blips.

### Failure simulation

```bash
docker compose stop node-exporter
```

1. Prometheus marks the `node` target DOWN and `up{job="node"}` becomes `0`.
2. At http://localhost:9090/alerts, `NodeExporterDown` becomes **PENDING**.
3. After one continuous minute it becomes **FIRING** and is sent to Alertmanager.

Restore the service with `docker compose start node-exporter`.

## Alertmanager

Alertmanager is connected and groups alerts by `alertname` and `severity`. **No notification channel (email, Slack, Teams, etc.) is configured yet.**

## Known limitation (Windows / Docker Desktop)

I first tried the usual Linux-host Node Exporter setup, mounting the host root (`/:/host:ro,rslave`) with `--path.rootfs`, `--path.procfs` and `--path.sysfs`. Docker Desktop on Windows rejected it:

```
path / is mounted on / but it is not a shared or slave mount
```

I removed the host-root mount, so Node Exporter now runs as an ordinary Linux container.

> **This stack monitors the Linux container environment inside Docker Desktop, not the physical Windows host.**

On a real Linux host the host mount can be used to monitor the machine itself.

## What I learned

- How the scrape → store → query → alert pipeline fits together
- Labels and time-series data in Prometheus
- PromQL basics: `up`, `rate()`, aggregation with `avg by(...)`
- Building Grafana dashboards from PromQL
- Writing alert rules and the PENDING → FIRING lifecycle
- Docker networking with service-name DNS
- Troubleshooting a real platform limitation

## Roadmap

- [ ] Finish the end-to-end test: `NodeExporterDown` PENDING → FIRING → Alertmanager
- [ ] Configure a real notification channel (email or Slack)
- [ ] Monitor a real Linux host
- [ ] Separate follow-up project: automate deployment with Ansible
