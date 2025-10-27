## Grafana + Loki + Prometheus + Alloy


What is Grafana Alloy?
Alloy is a unified agent that can:

Collect logs (replaces Promtail)
Collect metrics (can replace Node Exporter + cAdvisor)
Collect traces (for distributed tracing)
Transform and process data before sending
All in one lightweight agent

```
                ┌─────────────────────────────────────────────┐
                │           Centralized Monitoring VM         │
                │                                             │
                │  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
                │  │ Grafana  │  │   Loki   │  │Prometheus│   │
                │  │  (UI)    │  │ (Logs)   │  │ (Metrics)│   │
                │  └──────────┘  └──────────┘  └──────────┘   │
                │       ↑             ↑              ↑        │
                └───────┼─────────────┼──────────────┼────────┘
                        │             │              │
                        │             │              │
                    ┌───┴─────────────┴──────────────┴───┐
                    │         Network / Internet         │
                    └───┬─────────────┬──────────────┬───┘
                        │             │              │
                        │             │              │
                ┌───────▼─────┐ ┌─────▼──────┐ ┌────▼───────┐
                │  DB VM      │ │ Backend VM │ │Frontend VM │
                │             │ │            │ │            │
                │   ┊Alloy    │ │  ┊Alloy    │ │  ┊Alloy    │
                │ (All-in-1)│ │ │(All-in-1)  │ │ (All-in-1) │
                └─────────────┘ └────────────┘ └────────────┘
                        │             │              │
                        │             │              │
                ┌───────▼─────┐ ┌─────▼──────┐ ┌────▼───────┐
                │  DB VM      │ │ Backend VM │ │Frontend VM │
                │   QA        │ │    QA      │ │    QA      │
                │             │ │            │ │            │
                │   ┊Alloy    │ │  ┊Alloy    │ │  ┊Alloy    │
                │ (All-in-1)│ │ │(All-in-1)  │ │ (All-in-1) │
                └─────────────┘ └────────────┘ └────────────┘
                        │             │              │
                        │             │              │
                ┌───────▼─────┐ ┌─────▼──────┐ ┌────▼───────┐
                │  DB VM      │ │ Backend VM │ │Frontend VM │
                │   sandbox   │ │  sandbox   │ │  sandbox   │
                │             │ │            │ │            │
                │   ┊Alloy    │ │  ┊Alloy    │ │  ┊Alloy    │
                │ (All-in-1)│ │ │(All-in-1)  │ │ (All-in-1) │
                └─────────────┘ └────────────┘ └────────────┘

```

*Key change:* One Alloy agent replaces Promtail + Node Exporter + cAdvisor!


