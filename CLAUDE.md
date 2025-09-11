# Claude Development Notes

This file contains development context and notes for the Performance Co-Pilot Helm chart.

## Project Overview

This is a Helm chart for deploying Performance Co-Pilot (PCP) on Kubernetes. PCP is a system performance analysis toolkit that provides real-time and historical metrics collection.

## References

- [PCP Website](https://pcp.io) - Official documentation and guides
- [PCP GitHub Repository](https://github.com/performancecopilot/pcp) - Source code and issues
- [PCP Package Repository](https://packagecloud.io/performancecopilot/pcp) - Pre-built deb/rpm packages
- [Container Documentation](./container/README.md) - PCP container usage and configuration

## Repository Context

This Helm chart will be hosted at the PCP package repository (packagecloud.io/performancecopilot/pcp) alongside the deb and rpm packages.

## Key Design Decisions

### Single Instance Design
- `replicaCount: 1` is fixed - PCP monitors system resources and multiple replicas would create duplicate data
- Autoscaling is disabled since horizontal scaling doesn't make sense for system monitoring

### Container Integration
- Based on `quay.io/performancecopilot/pcp:latest` container
- Uses systemd init (`/usr/sbin/init`) requiring special Kubernetes handling
- Privileged mode required for full system access

### Storage Strategy
- Persistent volumes for `/var/log/pcp/pmlogger` (1Gi) and `/var/log/pcp/pmproxy` (1Gi)
- ConfigMaps for OpenMetrics PMDA configuration
- Optional host filesystem mount for advanced monitoring

## Environment Variables

Key PCP configuration via environment variables:
- `PCP_SERVICES`: Controls which PCP services start (default: pmcd,pmie,pmlogger,pmproxy)
- `PCP_DOMAIN_AGENTS`: Comma-separated list of agents to enable (e.g., "bpf,bpftrace")
- `HOST_MOUNT`: Path to host root mount for host monitoring mode
- `KEY_SERVERS`: Redis/Valkey connection specs

## Host Monitoring Mode

For eBPF and full host monitoring:
```yaml
hostMonitoring:
  enabled: true
hostNetwork: true
env:
  HOST_MOUNT: "/host"
  PCP_DOMAIN_AGENTS: "bpf,bpftrace"
```

## Health Checks

- **Liveness**: `systemctl is-active pmcd` (systemd-aware)
- **Readiness**: TCP socket check on pmcd port (44321)
- Adjusted timing for systemd container startup

## Testing Commands

```bash
# Validate chart
helm lint ./pcp

# Template output
helm template pcp ./pcp

# Install with host monitoring
helm install pcp ./pcp --set hostMonitoring.enabled=true --set hostNetwork=true --set env.HOST_MOUNT="/host"
```