# Portfolio Website — Fleet Deployment

GitOps manifests for deploying the portfolio site (`hoffeloffe/my-website`) to a
Raspberry Pi (ARM) Kubernetes homelab cluster using
[Rancher Fleet](https://fleet.rancher.io/).

Fleet watches this repo and reconciles the cluster to match the manifests under
[`fleet/`](./fleet) — no manual `kubectl apply`.

## What gets deployed

Namespace: `my-website`

| Manifest | Resource | Notes |
| --- | --- | --- |
| [`fleet/deployment.yaml`](./fleet/deployment.yaml) | `Deployment` my-website | 1 replica, ARM node affinity, probes, requests + limits |
| [`fleet/service.yaml`](./fleet/service.yaml) | `Service` my-website-service | ClusterIP, `80 -> 3000` |
| [`fleet/ingress.yaml`](./fleet/ingress.yaml) | `Ingress` my-website-ingress | external host |
| [`fleet/fleet.yaml`](./fleet/fleet.yaml) | Fleet bundle config | `defaultNamespace: my-website` |

The app listens on port **3000** (`PORT=3000`). The image is **pinned by digest**
(not `:latest`) so deploys are reproducible and rollback-able.

## Raspberry Pi specifics

- `nodeAffinity` prefers `arm`/`arm64` nodes and broad `tolerations` let it land
  on tainted Pi nodes.
- `strategy: Recreate` + `progressDeadlineSeconds: 600` suit limited hardware.

## Deploying a new version

1. Build & push a new `hoffeloffe/my-website` image.
2. Update the image digest in `fleet/deployment.yaml` and commit.
3. Fleet detects the change and rolls it out.

## Local validation

```bash
kubeconform -strict -ignore-missing-schemas \
  fleet/deployment.yaml fleet/service.yaml fleet/ingress.yaml
```
