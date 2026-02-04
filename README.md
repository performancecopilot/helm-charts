# Performance Co-Pilot Helm Charts

Official Helm charts for deploying [Performance Co-Pilot (PCP)](https://pcp.io) on Kubernetes.

## Available Charts

| Chart | Description | Version |
|-------|-------------|---------|
| [pcp](./pcp/) | Performance Co-Pilot system monitoring | ![Version](https://img.shields.io/badge/version-1.0.0-blue) |
| [archive-analysis](./archive-analysis/) | PCP Archive Analysis with Grafana | ![Version](https://img.shields.io/badge/version-1.0.0-blue) |

## Installation

### Prerequisites

- Kubernetes 1.19+
- Helm 3.8+ (required for OCI registry support)

### From OCI Registry (Recommended)

After a versioned release has been published, you can install charts from OCI registries:

#### GitHub Container Registry
```bash
# Install specific version
helm install pcp oci://ghcr.io/performancecopilot/helm-charts/pcp --version 1.0.0
helm install archive-analysis oci://ghcr.io/performancecopilot/helm-charts/archive-analysis --version 1.0.0

# Or use version constraints
helm install pcp oci://ghcr.io/performancecopilot/helm-charts/pcp --version ">=1.0.0 <2.0.0"
```

#### Quay.io
```bash
# Install specific version
helm install pcp oci://quay.io/performancecopilot-helm-charts/pcp --version 1.0.0
helm install archive-analysis oci://quay.io/performancecopilot-helm-charts/archive-analysis --version 1.0.0
```

### From Helm Repository (Traditional Method)

```bash
# Add the PCP Helm repository
helm repo add pcp https://performancecopilot.github.io/helm-charts/
helm repo update

# Install charts
helm install pcp pcp/pcp -n monitoring --create-namespace
helm install archive-analysis pcp/archive-analysis -n monitoring --create-namespace

# Or install specific version
helm install pcp pcp/pcp --version 1.0.0 -n monitoring --create-namespace
```

### From Source (Development/Testing)

If you need the latest development version or the OCI registries are not yet available:

```bash
# Clone repository
git clone https://github.com/performancecopilot/helm-charts.git
cd helm-charts

# Install from local charts
helm install pcp ./pcp -n monitoring --create-namespace
helm install archive-analysis ./archive-analysis -n monitoring
```

### From GitHub Releases

Download packaged charts from [GitHub Releases](https://github.com/performancecopilot/helm-charts/releases):

```bash
# Download chart package
wget https://github.com/performancecopilot/helm-charts/releases/download/v1.0.0/pcp-1.0.0.tgz

# Install from package
helm install pcp pcp-1.0.0.tgz -n monitoring --create-namespace
```

## Chart Documentation

- **[PCP Chart](./pcp/README.md)** - System performance monitoring with metrics collection, logging, and optional host monitoring
- **[Archive Analysis Chart](./archive-analysis/README.md)** - Grafana-based analysis of historical PCP data with pre-configured dashboards

## Quick Start Examples

### Basic PCP Deployment
```bash
# Install PCP with default settings
helm install pcp oci://ghcr.io/performancecopilot/helm-charts/pcp --version 1.0.0 -n monitoring --create-namespace
```

### PCP with Host Monitoring (eBPF)
```bash
# Enable host network and eBPF agents
helm install pcp oci://ghcr.io/performancecopilot/helm-charts/pcp --version 1.0.0 \
  --set hostMonitoring.enabled=true \
  --set hostNetwork=true \
  --set env.HOST_MOUNT="/host" \
  --set env.PCP_DOMAIN_AGENTS="bpf,bpftrace" \
  -n monitoring --create-namespace
```

### Archive Analysis with Grafana
```bash
# Install archive-analysis with ingress enabled
helm install archive-analysis oci://ghcr.io/performancecopilot/helm-charts/archive-analysis --version 1.0.0 \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=pcp-grafana.example.com \
  -n monitoring --create-namespace
```

## Troubleshooting

### OCI Registry Not Found

If you encounter errors like "repository not found" or "no versions available":

**Problem**: The chart may not yet be published to the OCI registry.

**Solution**: Use the source installation method:
```bash
git clone https://github.com/performancecopilot/helm-charts.git
cd helm-charts
helm install pcp ./pcp -n monitoring --create-namespace
```

### Invalid Version Constraint

If you see errors like `invalid version constraint: "latest"`:

**Problem**: Helm OCI registries require semantic version constraints, not "latest".

**Solution**: Specify a version number or range:
```bash
# Specific version
helm install pcp oci://ghcr.io/performancecopilot/helm-charts/pcp --version 1.0.0

# Version range
helm install pcp oci://ghcr.io/performancecopilot/helm-charts/pcp --version ">=1.0.0"
```

### List Available Versions

To see available chart versions:
```bash
# Check GitHub releases
gh release list --repo performancecopilot/helm-charts

# Or visit the releases page
# https://github.com/performancecopilot/helm-charts/releases
```

## Development

### Testing Charts Locally

```bash
# Lint charts
helm lint ./pcp
helm lint ./archive-analysis

# Test template generation
helm template pcp ./pcp
helm template archive-analysis ./archive-analysis

# Dry-run installation
helm install --dry-run --debug pcp ./pcp -n monitoring
```

### Packaging Charts

```bash
# Package charts
helm package pcp/
helm package archive-analysis/

# Test packaged chart
helm install pcp pcp-1.0.0.tgz -n monitoring --create-namespace
```

## Contributing

See [RELEASE.md](./RELEASE.md) for information about creating releases and publishing charts.

## Resources

- [PCP Website](https://pcp.io) - Official documentation
- [PCP GitHub](https://github.com/performancecopilot/pcp) - Source code
- [Container Documentation](https://github.com/performancecopilot/pcp/blob/main/build/containers/README.md) - PCP containers guide
- [ArtifactHub](https://artifacthub.io) - Chart discovery platform
