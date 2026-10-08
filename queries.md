# PromQL Cheat Sheet

Target health:
```promql
up
```

CPU:
```promql
100 - (
  avg by(instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  ) * 100
)
```

Memory:
```promql
(
  1 -
  node_memory_MemAvailable_bytes /
  node_memory_MemTotal_bytes
) * 100
```

Filesystem:
```promql
(
  1 -
  node_filesystem_avail_bytes{fstype!~"tmpfs|overlay"}
  /
  node_filesystem_size_bytes{fstype!~"tmpfs|overlay"}
) * 100
```

Network receive:
```promql
rate(node_network_receive_bytes_total[5m])
```

Network transmit:
```promql
rate(node_network_transmit_bytes_total[5m])
```

Load:
```promql
node_load1
node_load5
node_load15
```

Remember:
- Counters generally increase: use `rate()` or `increase()`.
- Gauges can go up and down: query them directly.
- Labels define dimensions of a time series.
