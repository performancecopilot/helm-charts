# Performance Co-Pilot (PCP) Helm Chart Improvements

## Date: 2026-02-04

---

## Issues Encountered During Installation

### 1. OCI Registry URLs Not Functional
- **Problem**: Documentation references OCI registry at `ghcr.io/performancecopilot/helm-charts` but this registry is not accessible
- **Impact**: Cannot install using standard `helm install oci://` commands
- **Error**: Repository not found or no versions available

### 2. Traditional Helm Repository Deprecated
- **Problem**: Legacy helm repo URL `https://performancecopilot.github.io/helm-charts` returns 404 Not Found
- **Impact**: `helm repo add` approach no longer works
- **Error**: HTTP 404

### 3. Version Tag Issues
- **Problem**: No version tags available in OCI registry; "latest" tag not a valid semver constraint
- **Impact**: Cannot specify versions like `--version v1.0.0` or use `--version latest`
- **Error**: `invalid version constraint: "latest"`

### 4. Documentation Outdated
- **Problem**: Installation instructions don't reflect current state of chart distribution
- **Impact**: Users cannot follow documented installation procedures
- **Workaround Required**: Manual clone from GitHub repository

---

## Current Workaround

```bash
# Clone the helm charts repository
cd /tmp
git clone https://github.com/performancecopilot/helm-charts.git pcp-charts

# Install from local chart
helm install pcp /tmp/pcp-charts/pcp -n <namespace>
```

---

## Recommended Improvements

### 1. Publish Versioned Charts to OCI Registries
- **Action**: Push versioned helm charts to OCI-compatible registries (ghcr.io, quay.io, or registry.hub.docker.com)
- **Implementation**:
  ```bash
  helm package pcp/
  helm push pcp-<version>.tgz oci://ghcr.io/performancecopilot/helm-charts
  ```
- **Benefit**: Standard helm installation workflow with version control

### 2. Use Proper Semantic Versioning
- **Action**: Tag chart releases with semver-compliant versions (e.g., v1.0.0, v1.1.0, v2.0.0)
- **Implementation**: Update Chart.yaml with version field and create matching git tags
- **Benefit**: Allows `helm install --version v1.0.0` syntax

### 3. Update Documentation
- **Action**: Revise README and installation guides to reflect current distribution method
- **Include**:
  - Primary installation method (OCI registry once available)
  - Fallback method (local clone from GitHub)
  - Version compatibility matrix
  - Troubleshooting section for common issues
- **Benefit**: Users can successfully install without trial and error

### 4. Publish to ArtifactHub
- **Action**: Register PCP helm charts on https://artifacthub.io
- **Implementation**: Add artifacthub-repo.yml to repository
- **Benefit**: Better discoverability, standardized documentation, security scanning

### 5. Automate Chart Releases
- **Action**: Set up GitHub Actions workflow to automatically package and publish charts on tag/release
- **Implementation**:
  ```yaml
  # .github/workflows/release-charts.yaml
  - helm package charts/pcp
  - helm push pcp-${{ VERSION }}.tgz oci://ghcr.io/performancecopilot/helm-charts
  ```
- **Benefit**: Consistent, automated releases reduce maintenance burden

### 6. Provide Multiple Distribution Channels
- **Action**: Support both OCI registries and traditional helm repositories
- **Options**:
  - OCI: `oci://ghcr.io/performancecopilot/helm-charts/pcp`
  - Traditional: `helm repo add pcp https://performancecopilot.github.io/helm-charts`
  - GitHub Releases: Attach packaged charts to releases
- **Benefit**: Users can choose their preferred installation method

---

## Testing Recommendations

Before publishing, test installation methods:

```bash
# Test OCI installation
helm install pcp oci://ghcr.io/performancecopilot/helm-charts/pcp --version v1.0.0

# Test with specific version constraint
helm install pcp oci://ghcr.io/performancecopilot/helm-charts/pcp --version ">=1.0.0 <2.0.0"

# Test repository method (if traditional repo is restored)
helm repo add pcp https://performancecopilot.github.io/helm-charts
helm repo update
helm install pcp pcp/pcp --version v1.0.0
```

---

## Reference Links

- **PCP Helm Charts Repository**: https://github.com/performancecopilot/helm-charts
- **Helm OCI Documentation**: https://helm.sh/docs/topics/registries/
- **ArtifactHub**: https://artifacthub.io
- **Semantic Versioning**: https://semver.org

---

## Impact Assessment

**Current State**: Installation requires manual workaround (git clone)
**Desired State**: One-command installation with version control
**User Experience Impact**: High - significantly simplifies deployment workflow
**Priority**: High - affects all new users attempting to deploy PCP via Helm

---

_Document created during llm-d cluster installation on nathans-llm-d-6_
