# Archive Analysis Helm Chart

A Helm chart for deploying Performance Co-Pilot Archive Analysis with Grafana on Kubernetes.

## Overview

This chart deploys the PCP Archive Analysis container which includes Grafana with the grafana-pcp plugin for analyzing PCP archive data. It's designed for analysis of retrospective system performance data collected by PCP.

## Features

- **Grafana with PCP Plugin**: Pre-configured Grafana instance with grafana-pcp plugin
- **Importing**: User-supplied volumes for Grafana dashboards and PCP archives
- **Security**: Non-privileged container with configurable security contexts
- **Ingress Support**: Optional ingress for external web access

## Prerequisites

- Kubernetes 1.19+
- Helm 3.8+ (required for OCI registry support)

## Installing the Chart

### From OCI Registry (Recommended)

```bash
# Install specific version from GitHub Container Registry
helm install archive-analysis oci://ghcr.io/performancecopilot/helm-charts/archive-analysis --version 1.0.0 -n monitoring --create-namespace

# Or from Quay.io
helm install archive-analysis oci://quay.io/performancecopilot-helm-charts/archive-analysis --version 1.0.0 -n monitoring --create-namespace
```

### From Source

```bash
# Clone repository
git clone https://github.com/performancecopilot/helm-charts.git
cd helm-charts

# Install from local chart
helm install archive-analysis ./archive-analysis -n monitoring --create-namespace
```

### From GitHub Release

```bash
# Download packaged chart
wget https://github.com/performancecopilot/helm-charts/releases/download/v1.0.0/archive-analysis-1.0.0.tgz

# Install from package
helm install archive-analysis archive-analysis-1.0.0.tgz -n monitoring --create-namespace
```

## Uninstalling the Chart

```bash
helm uninstall archive-analysis -n monitoring
```

## Configuration

The following table lists the configurable parameters and their default values:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `1` |
| `image.repository` | Container image repository | `quay.io/performancecopilot/archive-analysis` |
| `image.tag` | Container image tag | `""` (uses appVersion) |
| `image.pullPolicy` | Container image pull policy | `IfNotPresent` |
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.ports.grafana` | Grafana service port | `3000` |
| `service.ports.pcp` | PCP REST API service port | `44323` |
| `ingress.enabled` | Enable ingress | `false` |
| `persistence.dashboards.enabled` | Enable Grafana dashboards persistence | `true` |
| `persistence.dashboards.size` | Grafana dashboards storage size | `16Mi` |
| `persistence.archives.enabled` | Enable archives persistence | `true` |
| `persistence.archives.size` | Archives storage size | `1Gi` |
| `env.GF_SECURITY_ADMIN_PASSWORD` | Grafana admin password | `admin` |

## Usage Examples

### Basic Installation

```bash
helm install archive-analysis oci://ghcr.io/performancecopilot/helm-charts/archive-analysis --version 1.0.0 -n monitoring --create-namespace
```

### With Custom Grafana Password

```bash
helm install archive-analysis oci://ghcr.io/performancecopilot/helm-charts/archive-analysis --version 1.0.0 \
  --set env.GF_SECURITY_ADMIN_PASSWORD=mypassword \
  -n monitoring --create-namespace
```

### With Ingress Enabled

```bash
helm install archive-analysis oci://ghcr.io/performancecopilot/helm-charts/archive-analysis --version 1.0.0 \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=grafana.example.com \
  --set ingress.hosts[0].paths[0].path=/ \
  -n monitoring --create-namespace
```

### With Custom Storage

```bash
helm install archive-analysis oci://ghcr.io/performancecopilot/helm-charts/archive-analysis --version 1.0.0 \
  --set persistence.dashboards.size=32Mi \
  --set persistence.archives.size=5Gi \
  --set persistence.dashboards.storageClass=fast-ssd \
  -n monitoring --create-namespace
```

### From Source (Development)

```bash
git clone https://github.com/performancecopilot/helm-charts.git
cd helm-charts
helm install archive-analysis ./archive-analysis -n monitoring --create-namespace
```

## Accessing Grafana

After installation, access Grafana using:

1. **Port Forward** (default):
   ```bash
   kubectl port-forward -n monitoring svc/archive-analysis 3000:3000
   ```
   Then open http://localhost:3000

2. **NodePort Service**:
   ```bash
   helm upgrade archive-analysis oci://ghcr.io/performancecopilot/helm-charts/archive-analysis --version 1.0.0 \
     --set service.type=NodePort \
     -n monitoring
   ```

3. **Ingress** (if enabled):
   Access via configured hostname

Default login: `admin` / `admin` (or custom password if set)

## Data Import

To analyze PCP archives:

1. Copy archive files to the archives persistent volume
2. Use Grafana's PCP datasource to query the archive data
3. Create dashboards for visualization

## Persistence

The chart creates two persistent volume claims:

- **Grafana Dashboards** (`/var/lib/grafana`): Stores Grafana dashboards, configuration, and user data
- **Archives** (`/var/log/pcp`): Stores PCP archive files for analysis

## Security

This chart runs as a non-privileged container by default:
- Runs as user 1000
- No privilege escalation
- RuntimeDefault seccomp profile
- No special capabilities required

## Troubleshooting

### Pod Won't Start
Check the logs:
```bash
kubectl logs -f deployment/archive-analysis
```

### Storage Issues
Check PVC status:
```bash
kubectl get pvc
```

### Grafana Access Issues
Verify service and ingress configuration:
```bash
kubectl get svc,ingress
```
