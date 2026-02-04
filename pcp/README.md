# Performance Co-Pilot Helm Chart

A Helm chart for deploying Performance Co-Pilot (PCP) on Kubernetes.

Performance Co-Pilot is a system performance analysis toolkit that provides comprehensive real-time and historical metrics collection and analysis capabilities. PCP includes:

- **Real-time monitoring**: Live system metrics via pmcd daemon
- **Historical analysis**: Archive logging via pmlogger 
- **Web interface**: REST API via pmproxy for integration with Grafana, Prometheus
- **Extensive metrics**: CPU, memory, disk, network, containers, and custom application metrics
- **eBPF support**: Advanced kernel tracing with bpftrace and BPF agents
- **Multi-host monitoring**: Central collection from distributed systems

For more information, visit [pcp.io](https://pcp.io) or the [GitHub repository](https://github.com/performancecopilot/pcp).

Pre-built packages are available at [packagecloud.io/performancecopilot/pcp](https://packagecloud.io/performancecopilot/pcp).

## Prerequisites

- Kubernetes 1.19+
- Helm 3.8+ (required for OCI registry support)
- PV provisioner support in the underlying infrastructure (for persistent storage)

## Installing the Chart

### From OCI Registry (Recommended)

```bash
# Install specific version from GitHub Container Registry
helm install pcp oci://ghcr.io/performancecopilot/helm-charts/pcp --version 1.0.0 -n monitoring --create-namespace

# Or from Quay.io
helm install pcp oci://quay.io/performancecopilot-helm-charts/pcp --version 1.0.0 -n monitoring --create-namespace
```

### From Source

```bash
# Clone repository
git clone https://github.com/performancecopilot/helm-charts.git
cd helm-charts

# Install from local chart
helm install pcp ./pcp -n monitoring --create-namespace
```

### From GitHub Release

```bash
# Download packaged chart
wget https://github.com/performancecopilot/helm-charts/releases/download/v1.0.0/pcp-1.0.0.tgz

# Install from package
helm install pcp pcp-1.0.0.tgz -n monitoring --create-namespace
```

## Uninstalling the Chart

```bash
helm uninstall pcp -n monitoring
```

## Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas (fixed at 1) | `1` |
| `image.repository` | PCP container image repository | `quay.io/performancecopilot/pcp` |
| `image.tag` | PCP container image tag | `latest` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `hostNetwork` | Use host networking | `false` |
| `securityContext.privileged` | Run in privileged mode | `true` |
| `persistence.pmlogger.enabled` | Enable persistent storage for pmlogger | `true` |
| `persistence.pmlogger.size` | Size of pmlogger volume | `1Gi` |
| `persistence.pmproxy.enabled` | Enable persistent storage for pmproxy | `true` |
| `persistence.pmproxy.size` | Size of pmproxy volume | `1Gi` |
| `hostMonitoring.enabled` | Enable host monitoring mode | `false` |
| `env.PCP_SERVICES` | PCP services to start | `pmcd,pmie,pmlogger,pmproxy` |
| `env.PCP_DOMAIN_AGENTS` | PCP agents to enable | `""` |
| `env.HOST_MOUNT` | Host mount path for monitoring | `""` |
| `env.KEY_SERVERS` | Redis/Valkey connection specs | `localhost:6379` |

## Host Monitoring

For comprehensive system monitoring including eBPF capabilities:

```bash
helm install pcp ./pcp \
  --set hostMonitoring.enabled=true \
  --set hostNetwork=true \
  --set env.HOST_MOUNT="/host" \
  --set env.PCP_DOMAIN_AGENTS="bpf,bpftrace"
```

## Accessing PCP

- **PMCD**: Port 44321 (native PCP tools)
- **PMPROXY**: Port 44322 (HTTP/REST API)

## Examples

### Basic deployment:
```bash
helm install pcp oci://ghcr.io/performancecopilot/helm-charts/pcp --version 1.0.0 -n monitoring --create-namespace
```

### With custom agents:
```bash
helm install pcp oci://ghcr.io/performancecopilot/helm-charts/pcp --version 1.0.0 \
  --set env.PCP_DOMAIN_AGENTS="postgresql,apache" \
  -n monitoring --create-namespace
```

### Host monitoring with eBPF:
```bash
helm install pcp oci://ghcr.io/performancecopilot/helm-charts/pcp --version 1.0.0 \
  --set hostMonitoring.enabled=true \
  --set hostNetwork=true \
  --set env.HOST_MOUNT="/host" \
  --set env.PCP_DOMAIN_AGENTS="bpf,bpftrace" \
  -n monitoring --create-namespace
```

### From source (development):
```bash
git clone https://github.com/performancecopilot/helm-charts.git
cd helm-charts
helm install pcp ./pcp -n monitoring --create-namespace
```

## Notes

- PCP runs as a single instance (replicaCount=1) since multiple replicas would duplicate system monitoring data
- Privileged mode is required for full system access
- Host monitoring mode requires additional cluster permissions