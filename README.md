# k3s-gitops

Personal homelab GitOps setup running on Talos Linux (Kubernetes 1.36). This is an experiment to learn GitOps patterns — not a production setup. The goal is to be able to tear down the cluster and have everything back up within half an hour.

This repo manages the full cluster state via ArgoCD using an app-of-apps pattern. Everything except the initial bootstrap (automated via `tofu apply` in the [k8s-infra](https://github.com/yoramvandevelde/k8s-infra) repo) is managed through Git.

**Infrastructure**
- [cert-manager](https://cert-manager.io) — TLS certificates via Let's Encrypt + Cloudflare DNS challenge
- [ingress-nginx](https://kubernetes.github.io/ingress-nginx) — ingress controller
- [MetalLB](https://metallb.universe.tf) — bare metal load balancer (L2, pool `10.10.99.1–10.10.99.250`)
- [Cilium](https://cilium.io) — CNI with Hubble, Envoy proxy and SPIRE for service mesh and mTLS
- [democratic-csi](https://github.com/democratic-csi/democratic-csi) — iSCSI and NFS storage via TrueNAS
- [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) — encrypted secrets safe to store in Git
- [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) — monitoring (Prometheus + Grafana + Alertmanager)
- [snapshot-controller](https://github.com/kubernetes-csi/external-snapshotter) — volume snapshots
- [Kyverno](https://kyverno.io) — policy engine, all policies in Audit mode (see below)
- [Argo Rollouts](https://argoproj.github.io/rollouts/) — progressive delivery (canary deployments)
- [chaoskube](https://github.com/linki/chaoskube) — chaos engineering, random pod termination every 10 minutes

**Apps**
- [Forgejo](https://forgejo.org) — self-hosted Git (SQLite, iSCSI storage, SSH via MetalLB)
- [Headlamp](https://headlamp.dev) — Kubernetes web UI
- [Wordpress](https://wordpress.org) — dev and prod environments (MySQL + Redis + custom image with Redis plugin)
- [Recipit](https://github.com/yoramvandevelde/recipit) — personal recipe manager with Home Assistant integration; full CI/CD pipeline that automatically updates the image tag in this repo on push
- [Tuwunel](https://github.com/matrix-construct/tuwunel) — Matrix homeserver (Rust-based, RocksDB, no federation)

## Structure
```bash
bootstrap/        # ArgoCD root app and app-of-apps entrypoints
infrastructure/   # Cluster infrastructure (networking, storage, monitoring, policy, etc.)
apps/             # Application deployments with kustomize base/overlay structure
config/           # Public sealed secrets cert and encrypted master key
```

## Security

All workloads are hardened and comply with the following Kyverno policies (all in Audit mode):

- `disallow-latest-tag` — image tags must be pinned
- `require-resource-requests-limits` — all containers must define CPU/memory requests and limits
- `block-privileged-containers` — privileged containers are blocked (exceptions: Cilium and democratic-csi node drivers)
- `require-non-root` — containers must run with `runAsNonRoot: true`
- `require-readonly-rootfs` — containers must set `readOnlyRootFilesystem: true` (exceptions: `ingress-nginx`, `kube-system`, `storage`, `metallb-system`)

All application namespaces have CiliumNetworkPolicy with default-deny and explicit allow rules per workload.

## Secrets

Secrets are encrypted with Sealed Secrets. The public cert is in `config/sealed-secret-pub.crt` and can be used to encrypt new secrets without cluster access:

```bash
kubeseal --cert config/sealed-secret-pub.crt --format yaml < secret.yaml > sealed-secret.yaml
```

The master key is stored as a GPG-encrypted file in `config/sealed-secrets-master-key.yaml.gpg`. Without it, sealed secrets cannot be decrypted on a new cluster.

## Bootstrap

The full bootstrap is automated via `tofu apply` in the [k8s-infra](https://github.com/yoramvandevelde/k8s-infra) repo. It runs a Docker container that installs Cilium, ArgoCD, and Sealed Secrets in order, restores the sealed secrets master key, and then applies the root app:

```bash
kubectl apply -f bootstrap/root.yaml
```

ArgoCD takes over from there and syncs all infrastructure and apps from Git.

The sealed secrets master key must be restored before ArgoCD syncs, otherwise secrets will fail to decrypt.
