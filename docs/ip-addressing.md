# IP addressing & monitoring design

## PT topology — management LAN

| Device      | IP            | Role                      |
|-------------|---------------|---------------------------|
| CORE-RTR    | 10.10.10.1    | Core router               |
| DIST-RTR    | 10.10.10.2    | Distribution router       |
| ACCESS-RTR  | 10.10.10.3    | Access router             |
| SYSLOG-SRV  | 10.10.10.100  | Syslog + SNMP trap receiver |

All devices share a flat management VLAN. In production, monitoring traffic would be on a dedicated management VLAN with ACLs restricting access.

## SNMP configuration

| Setting                | Value            |
|------------------------|-------------------|
| Read-only community    | MONITOR-RO        |
| Read-write community   | MONITOR-RW        |
| Trap destination       | 10.10.10.100      |
| SNMP version           | 2c                |
| Traps enabled          | linkdown, linkup, config |

## Syslog configuration

| Setting              | Value              |
|----------------------|---------------------|
| Syslog host          | 10.10.10.100        |
| Severity level       | informational (6)   |
| Timestamps           | datetime with msec  |
| Source interface      | GigabitEthernet0/0/0 |

## Docker monitoring stack

| Service       | Port  | Purpose                                |
|---------------|-------|----------------------------------------|
| Prometheus    | 9090  | Metrics collection and storage         |
| SNMP Exporter | 9116  | Translates SNMP polls to Prometheus format |
| Grafana       | 3000  | Dashboard visualization (admin/admin)  |

## How the pieces connect

**In this lab (two separate verifications):**
- PT proves the device-side config: SNMP communities, trap hosts, syslog forwarding
- Docker proves the monitoring stack: Prometheus scrapes SNMP metrics from a local demo target, Grafana visualizes them

**In production (single integrated flow):**
- Cisco devices send SNMP traps and syslog to the monitoring server
- Prometheus + SNMP Exporter polls the devices every 60 seconds via SNMP
- Grafana dashboards show interface status, traffic rates, error counts, device availability
- Alerting rules fire when an interface goes down or error rates spike
- All from the same SNMP communities and syslog config proven in the PT half of this lab
