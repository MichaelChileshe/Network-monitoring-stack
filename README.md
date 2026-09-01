# Full Network Monitoring Stack (SNMP / Syslog / Grafana)

**Author:** Michael Chileshe

A complete network monitoring solution in two parts: Cisco IOS device-side configuration (SNMP traps, syslog forwarding, verified in Packet Tracer) and a production-style monitoring stack (Prometheus + SNMP Exporter + Grafana, running in Docker on WSL) with a pre-built network dashboard. Both parts are independently verified.

## Objectives

- Configure SNMP communities, trap hosts, and syslog forwarding on Cisco IOS devices in Packet Tracer
- Verify syslog collection on PT's built-in syslog server (interface up/down events, config changes)
- Deploy a Docker-based monitoring stack: Prometheus scrapes SNMP metrics, Grafana visualizes them
- Pre-build a Grafana network monitoring dashboard (interface status, traffic, device availability)
- Document how the two halves connect in production (device configs point at the monitoring server)

## Architecture

```
┌─────────────────────────────────────────────────────┐
│  Packet Tracer (device-side config)                 │
│                                                     │
│  CORE-RTR ──┐                                       │
│  DIST-RTR ──┼── SNMP traps + syslog ──► SYSLOG-SRV │
│  ACCESS-RTR ┘                                       │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│  Docker on WSL (monitoring stack)                   │
│                                                     │
│  SNMP Target ──► SNMP Exporter ──► Prometheus       │
│  (snmpd on       :9116              :9090           │
│   localhost)         │                              │
│                      └──────────► Grafana :3000     │
│                                   (pre-built        │
│                                    dashboard)       │
└─────────────────────────────────────────────────────┘
```

## Tech stack

| Component       | Purpose                                    | Runs on    |
|----------------|--------------------------------------------|------------|
| Cisco IOS      | SNMP agent + syslog source                 | Packet Tracer |
| PT Syslog Server| Collects and displays syslog messages     | Packet Tracer |
| Prometheus     | Time-series metrics database               | Docker/WSL |
| SNMP Exporter  | Translates SNMP polls into Prometheus metrics | Docker/WSL |
| Grafana        | Dashboard visualization                    | Docker/WSL |
| snmpd          | Demo SNMP target (WSL localhost)           | WSL native |

## Repo structure

```
network-monitoring-stack/
├── README.md
├── .gitignore
├── pt-configs/
│   ├── CORE-RTR.txt
│   ├── DIST-RTR.txt
│   ├── ACCESS-RTR.txt
│   └── MONITOR-SW.txt
├── docker/
│   ├── docker-compose.yml
│   ├── prometheus/
│   │   └── prometheus.yml
│   └── grafana/
│       └── provisioning/
│           ├── datasources/
│           │   └── prometheus.yml
│           └── dashboards/
│               ├── dashboard.yml
│               └── network-monitoring.json
├── docs/
│   ├── ip-addressing.md
└── topology/
    └── (PT screenshots + Docker screenshots)
```

## Quick start

### Part 1 — Packet Tracer (device-side)
```
Build topology → paste configs → generate syslog events → screenshot the syslog server
```

### Part 2 — Docker (monitoring stack)
```bash
cd docker
docker compose up -d
# Open http://localhost:3000 in your browser (admin / admin)
# Dashboard auto-loads — screenshot the live metrics
```

## Verification checklist

### Packet Tracer
- [ ] SNMP community configured on all 3 routers (`show snmp community`)
- [ ] Syslog host configured pointing to SYSLOG-SRV (`show logging`)
- [ ] Syslog server shows collected messages after interface up/down events
- [ ] SNMP trap host configured (`show snmp host`)

### Docker / Grafana
- [ ] `docker compose up -d` starts all 3 containers without errors
- [ ] Prometheus UI (localhost:9090) shows SNMP target as UP
- [ ] Grafana (localhost:3000) loads with the pre-built network dashboard
- [ ] Dashboard shows live SNMP metrics from the demo target
