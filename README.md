# Addon Flux

Upbound addon package for [Flux CD](https://fluxcd.io/).

Flux keeps Kubernetes clusters in sync with sources of configuration (like Git repositories and OCI artifacts) and automates updates to configuration when there is new code to deploy.

## Components

- **source-controller** — Fetches artifacts from Git, Helm, OCI, and S3-compatible repositories
- **kustomize-controller** — Reconciles Kustomization resources and applies manifests
- **helm-controller** — Reconciles HelmRelease resources and manages Helm chart releases
- **notification-controller** — Handles alerts, event forwarding, and webhook receivers
- **image-automation-controller** — Automates image updates in Git repositories
- **image-reflector-controller** — Scans container registries and reflects image metadata

## Install

### As AddOn (UXP v2)

```yaml
apiVersion: pkg.upbound.io/v1beta1
kind: AddOn
metadata:
  name: flux
spec:
  package: xpkg.upbound.io/upbound/addon-flux:2.18.4
```

### As Controller (Spaces)

```yaml
apiVersion: pkg.upbound.io/v1alpha1
kind: Controller
metadata:
  name: addon-flux
spec:
  package: xpkg.upbound.io/upbound/controller-flux:2.18.4
```

## Configuration

Override Helm values at install time using `AddOnRuntimeConfig`:

```yaml
apiVersion: pkg.upbound.io/v1beta1
kind: AddOnRuntimeConfig
metadata:
  name: flux-config
spec:
  helm:
    values:
      imageAutomationController:
        create: false
      imageReflectionController:
        create: false
      watchAllNamespaces: false
```

## Requirements

- Kubernetes >= 1.33.0 (enforced by the chart's pre-install `flux-check` job)
- UXP 2.x or Spaces with AddOn/Controller support

## RBAC

The addon requires the `upbound-addon:flux` ClusterRole to be applied before installation. This grants the `upbound-controller-manager` ServiceAccount the permissions needed to deploy Flux components (deployments, services, CRDs, RBAC, network policies, jobs).

```bash
kubectl apply -f manifests/addon-cluster-role.yaml
```

## Development

### Building locally

```bash
source .chart-attributes
helm pull $CHART_NAME --repo $REPO_URL --version $CHART_VERSION
# See .github/workflows/ci.yaml for the full build pipeline
```

### Generating with helm2addon

The initial scaffold was generated using [UpboundCare/helm2addon](https://github.com/UpboundCare/helm2addon):

```bash
helm pull fluxcd-community/flux2 --version 2.18.4 -d _work --untar
./helm2addon generate _work/flux2 -n flux-system
```

## Upstream

- Chart: `fluxcd-community/flux2` v2.18.4 (App v2.8.8)
- Source: https://github.com/fluxcd/flux2
- License: Apache-2.0
