# Performance Co-Pilot Helm Charts

Official Helm charts for deploying [Performance Co-Pilot (PCP)](https://pcp.io) on Kubernetes.

## Available Charts

| Chart | Description | Version |
|-------|-------------|---------|
| [pcp](./pcp/) | Performance Co-Pilot system monitoring | ![Version](https://img.shields.io/badge/version-0.1.0-blue) |
| [archive-analysis](./archive-analysis/) | PCP Archive Analysis with Grafana | ![Version](https://img.shields.io/badge/version-0.1.0-blue) |

## Quick Start

### Add Helm Repository

```bash
helm repo add performancecopilot https://pcp.io/helm-charts
helm repo update
```

### Install Charts

```bash
# Install PCP for system monitoring
helm install pcp performancecopilot/pcp

# Install Archive Analysis for historical data analysis
helm install archive-analysis performancecopilot/archive-analysis
```

## Installation Methods

### 1. From Helm Repository (Recommended)

```bash
helm repo add performancecopilot https://pcp.io/helm-charts
helm install pcp performancecopilot/pcp
helm install archive-analysis performancecopilot/archive-analysis
```

### 2. From OCI Registry

#### GitHub Container Registry
```bash
helm install pcp oci://ghcr.io/performancecopilot/pcp
helm install archive-analysis oci://ghcr.io/performancecopilot/archive-analysis
```

#### Quay.io
```bash
helm install pcp oci://quay.io/performancecopilot/pcp
helm install archive-analysis oci://quay.io/performancecopilot/archive-analysis
```

### 3. From Source

```bash
git clone https://github.com/performancecopilot/helm-charts.git
cd helm-charts

helm install pcp ./pcp
helm install archive-analysis ./archive-analysis
```

## Chart Documentation

- **[PCP Chart](./pcp/README.md)** - System performance monitoring with metrics collection, logging, and optional host monitoring
- **[Archive Analysis Chart](./archive-analysis/README.md)** - Grafana-based analysis of historical PCP data with pre-configured dashboards

## Requirements

- Kubernetes 1.19+
- Helm 3.0+

## Development

### Testing Charts Locally

```bash
# Lint charts
helm lint ./pcp
helm lint ./archive-analysis

# Test template generation
helm template pcp ./pcp
helm template archive-analysis ./archive-analysis

# Install in development
helm install --dry-run --debug pcp ./pcp
```
