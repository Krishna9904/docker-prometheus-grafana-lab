# Troubleshooting Runbook

## Target DOWN

Open `http://localhost:9090/targets`.

Query:
```promql
up{job="node"}
```

Check:
```powershell
docker compose ps
docker compose logs node-exporter
```

Test the exporter from Prometheus:
```powershell
docker compose exec prometheus wget -qO- http://node-exporter:9100/metrics
```

## Grafana: No data

1. Test the same PromQL in Prometheus.
2. Check the Grafana Prometheus datasource.
3. Check the dashboard time range.
4. Check metric names and labels.

## Prometheus problems

```powershell
docker compose logs prometheus
docker compose exec prometheus promtool check config /etc/prometheus/prometheus.yml
```

## Alert not firing

Open:
`http://localhost:9090/alerts`

Test the alert expression directly in Prometheus.

Remember:
```yaml
for: 5m
```
means the expression must stay true for five minutes.

## Windows clarification

This lab runs on Windows but monitors the Linux environment exposed through Docker Desktop's Linux backend. It does not collect native Windows performance counters.

For native Windows monitoring, use `windows_exporter` as a separate project.
