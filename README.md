# 📊 Grafana Observability Pack

A production-grade, ready-to-run observability stack: **Grafana + Prometheus + Loki + Promtail + Alertmanager** — the same pattern I run in production to monitor 30+ microservices.

Spin up the entire stack locally with one command and get dashboards, log aggregation, and alerting out of the box.

```
┌─────────────┐     ┌────────────┐     ┌──────────────┐
│ node-exporter│────▶│ Prometheus │────▶│ Alertmanager │
└─────────────┘     └─────┬──────┘     └──────────────┘
                          │
┌─────────────┐     ┌─────▼──────┐
│  Promtail   │────▶│  Grafana   │◀─── dashboards auto-provisioned
└──────┬──────┘     └─────▲──────┘
       │            ┌─────┴──────┐
       └───────────▶│    Loki    │
                    └────────────┘
```

## 🚀 Quick Start

```bash
docker compose up -d
```

| Service | URL | Credentials |
|---|---|---|
| Grafana | http://localhost:3000 | admin / admin |
| Prometheus | http://localhost:9090 | — |
| Alertmanager | http://localhost:9093 | — |
| Loki | http://localhost:3100 | — |

Dashboards and datasources are **auto-provisioned** — open Grafana and everything is already wired up.

## 📁 What's Inside

```
├── docker-compose.yml            # Full stack definition
├── prometheus/
│   ├── prometheus.yml            # Scrape configs
│   └── alerts/
│       ├── node-alerts.yml       # Host-level alerts (CPU, memory, disk)
│       └── app-alerts.yml        # Application alerts (error rate, latency)
├── alertmanager/
│   └── alertmanager.yml          # Routing, grouping, inhibition rules
├── grafana/
│   ├── provisioning/             # Auto-provision datasources & dashboards
│   └── dashboards/
│       └── node-overview.json    # Host metrics dashboard
├── loki/
│   └── loki-config.yml           # Log storage config
└── promtail/
    └── promtail-config.yml       # Log shipping config
```

## 🔔 Alerting Philosophy

Every alert in this pack follows three rules I enforce in production:

1. **Every alert has a runbook** — the `runbook` annotation tells the on-call engineer what to do at 3 AM
2. **Alert on symptoms, not causes** — page on user-facing error rate, not on individual process restarts
3. **Severity drives routing** — `critical` pages immediately, `warning` goes to the team channel

Example from `node-alerts.yml`:

```yaml
- alert: HostOutOfDiskSpace
  expr: (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 < 10
  for: 5m
  labels:
    severity: critical
  annotations:
    summary: "Host {{ $labels.instance }} has less than 10% disk space"
    runbook: "1. Check /var/log growth  2. Verify log rotation  3. Clean docker images: docker system prune"
```

## 🧩 Extending

- **Add a scrape target:** drop a job into `prometheus/prometheus.yml`
- **Add a dashboard:** put the JSON in `grafana/dashboards/` — it auto-loads
- **Route alerts to Slack/Teams:** add a receiver in `alertmanager/alertmanager.yml`

## 🛠️ Built With Production Lessons

This mirrors the observability platform I built and operate in a PCI-DSS-aligned fintech environment:

- Grafana dashboards covering logs, error rates, response times, and service health
- Loki for centralized log aggregation (cheaper and simpler than ELK for most teams)
- Alert runbooks that made incident response measurably faster

## 📄 License

MIT — use it, fork it, ship it.

---
*Maintained by [Rohit Bhosale](https://github.com/rohitbhosale439) — DevOps Engineer*
