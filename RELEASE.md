# Release Process

This document describes the process for creating and publishing new releases of the Performance Co-Pilot Helm charts.

## Overview

The PCP Helm charts are published to multiple distribution channels:
- **OCI Registries**: GitHub Container Registry (ghcr.io) and Quay.io
- **GitHub Releases**: Packaged chart files attached to GitHub releases
- **ArtifactHub**: Automatic discovery via artifacthub-repo.yml

## Versioning

We follow [Semantic Versioning 2.0.0](https://semver.org/):

- **MAJOR** version: Breaking changes to chart structure or behavior
- **MINOR** version: New features, backward-compatible changes
- **PATCH** version: Bug fixes, documentation updates

Examples:
- `1.0.0` - Initial stable release
- `1.1.0` - Added new configuration options
- `1.1.1` - Fixed bug in service configuration
- `2.0.0` - Changed default values (breaking change)

## Automated Release Process

The release workflow is automated via GitHub Actions. Here's how to create a new release:

### 1. Update Chart Versions

Before creating a release, ensure the Chart.yaml files have the correct version:

```bash
# Edit version in Chart.yaml files
vim pcp/Chart.yaml
vim archive-analysis/Chart.yaml

# Commit the changes
git add pcp/Chart.yaml archive-analysis/Chart.yaml
git commit -m "Bump chart versions to 1.0.0"
git push origin main
```

**Note**: The GitHub Actions workflow will automatically update these versions during the release process, but having them match helps with tracking.

### 2. Create and Push a Git Tag

```bash
# Create an annotated tag with semver format (v prefix required)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Push the tag to GitHub
git push origin v1.0.0
```

### 3. Automated Publishing

Once the tag is pushed, the GitHub Actions workflow (`.github/workflows/release.yml`) automatically:

1. ✅ Checks out the repository
2. ✅ Extracts version from the tag (removes 'v' prefix)
3. ✅ Updates Chart.yaml versions to match the tag
4. ✅ Packages both charts (`helm package`)
5. ✅ Logs into GitHub Container Registry and Quay.io
6. ✅ Pushes charts to both OCI registries
7. ✅ Creates a GitHub Release with:
   - Packaged chart files (`.tgz`)
   - Auto-generated release notes
   - Installation instructions

### 4. Verify the Release

After the workflow completes, verify:

```bash
# Check GitHub releases
gh release list --repo performancecopilot/helm-charts

# Test installation from GitHub Container Registry
helm install pcp-test oci://ghcr.io/performancecopilot/helm-charts/pcp --version 1.0.0 -n test --create-namespace

# Test installation from Quay.io
helm install pcp-test oci://quay.io/performancecopilot-helm-charts/pcp --version 1.0.0 -n test --create-namespace

# Clean up test installation
helm uninstall pcp-test -n test
kubectl delete namespace test
```

## Development Builds

Every commit to the `main` branch triggers the publish workflow (`.github/workflows/publish.yml`), which:

1. Generates a development version: `0.1.0-dev.<commit-hash>`
2. Publishes to both OCI registries with the dev version

This allows testing of unreleased changes:

```bash
# Install latest development build
helm install pcp oci://ghcr.io/performancecopilot/helm-charts/pcp --version 0.1.0-dev.abc1234
```

## Manual Release Process (Fallback)

If automated releases fail, you can publish manually:

### 1. Package Charts

```bash
# Update versions in Chart.yaml files
VERSION="1.0.0"
sed -i "s/^version:.*/version: $VERSION/" pcp/Chart.yaml
sed -i "s/^version:.*/version: $VERSION/" archive-analysis/Chart.yaml

# Package charts
helm package pcp/
helm package archive-analysis/
```

### 2. Push to GitHub Container Registry

```bash
# Login to ghcr.io
echo $GITHUB_TOKEN | helm registry login ghcr.io -u $GITHUB_USER --password-stdin

# Push charts
helm push pcp-${VERSION}.tgz oci://ghcr.io/performancecopilot/helm-charts
helm push archive-analysis-${VERSION}.tgz oci://ghcr.io/performancecopilot/helm-charts
```

### 3. Push to Quay.io

```bash
# Login to Quay.io
helm registry login quay.io -u $QUAY_USERNAME -p $QUAY_PASSWORD

# Push charts
helm push pcp-${VERSION}.tgz oci://quay.io/performancecopilot-helm-charts
helm push archive-analysis-${VERSION}.tgz oci://quay.io/performancecopilot-helm-charts
```

### 4. Create GitHub Release

```bash
# Create release with gh CLI
gh release create v${VERSION} \
  --title "Release v${VERSION}" \
  --notes "See CHANGELOG.md for details" \
  pcp-${VERSION}.tgz \
  archive-analysis-${VERSION}.tgz
```

## Secrets Configuration

The following secrets must be configured in the GitHub repository settings:

| Secret | Description | Required For |
|--------|-------------|--------------|
| `GITHUB_TOKEN` | Automatic GitHub token | GHCR publishing, releases |
| `QUAY_USERNAME` | Quay.io username | Quay.io publishing |
| `QUAY_PASSWORD` | Quay.io password or robot token | Quay.io publishing |

To configure Quay.io credentials:

1. Go to GitHub repository Settings → Secrets and variables → Actions
2. Add `QUAY_USERNAME` and `QUAY_PASSWORD` repository secrets
3. For Quay.io, it's recommended to use a [Robot Account](https://docs.quay.io/glossary/robot-accounts.html)

## Rollback Process

To rollback a bad release:

### 1. Delete the Git Tag

```bash
# Delete local tag
git tag -d v1.0.0

# Delete remote tag
git push --delete origin v1.0.0
```

### 2. Delete GitHub Release

```bash
gh release delete v1.0.0 --yes
```

### 3. Remove from OCI Registries

**Note**: OCI registry tags cannot be easily deleted. Instead:
- Create a new patch version (e.g., v1.0.1) with the fix
- Update documentation to skip the problematic version
- Consider adding a warning in the GitHub release notes

## Testing Checklist

Before releasing, ensure:

- [ ] Charts pass linting: `helm lint ./pcp ./archive-analysis`
- [ ] Charts template correctly: `helm template pcp ./pcp && helm template archive-analysis ./archive-analysis`
- [ ] Charts install successfully in test cluster
- [ ] CHANGELOG.md is updated
- [ ] README.md reflects any new features or changes
- [ ] Chart.yaml versions are bumped appropriately
- [ ] All tests pass: `.github/workflows/test.yml`

## Release Cadence

- **Patch releases**: As needed for bug fixes
- **Minor releases**: Monthly or as features are completed
- **Major releases**: As needed for breaking changes (rare)

## ArtifactHub Integration

Charts are automatically discovered by ArtifactHub via:
1. The `artifacthub-repo.yml` file in the repository root
2. Publishing to GitHub Container Registry with proper metadata

No manual action required - ArtifactHub will scan and index releases automatically.

## Troubleshooting

### Workflow Fails on Push

**Error**: `Error: failed to authorize: failed to fetch oauth token`

**Solution**: Verify Quay.io credentials in repository secrets.

### Chart Not Appearing in Registry

**Error**: `repository not found`

**Solution**:
1. Check workflow logs for errors
2. Verify registry permissions
3. Ensure the tag follows the `v*` pattern

### Version Conflicts

**Error**: `chart version already exists`

**Solution**: You cannot overwrite existing chart versions. Create a new version instead.

## Resources

- [Helm OCI Documentation](https://helm.sh/docs/topics/registries/)
- [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [Quay.io Documentation](https://docs.quay.io/)
- [ArtifactHub](https://artifacthub.io)
- [Semantic Versioning](https://semver.org/)
