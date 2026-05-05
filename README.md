# k3s-gitops

Personal homelab GitOps setup running on k3s. This is an experiment to learn GitOps patterns — not a production setup. The goal is to be able to tear down the cluster and have everything back up within half an hour.

## What's in here

This repo manages the full cluster state via ArgoCD using an app-of-apps pattern. Everything except the initial ArgoCD bootstrap and secrets is managed through Git.

**Infrastructure**
- [cert-manager](https://cert-manager.io) — TLS certificates via Let's Encrypt + Cloudflare DNS
- [ingress-nginx](https://kubernetes.github.io/ingress-nginx) — ingress controller
- [MetalLB](https://metallb.universe.tf) — bare metal load balancer
- [democratic-csi](https://github.com/democratic-csi/democratic-csi) — iSCSI and NFS storage via TrueNAS
- [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) — encrypted secrets safe to store in Git
- [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) — monitoring
- [snapshot-controller](https://github.com/kubernetes-csi/external-snapshotter) — volume snapshots

**Apps**
- [Forgejo](https://forgejo.org) — self-hosted Git
- [Wordpress](https://wordpress.org) — dev and prod environments
- [Recipit](https://github.com/yoramvandevelde/recipit) — personal recipe manager, includes a full CI/CD pipeline that automatically updates the image tag in this repo on push

## Structure

```
bootstrap/        # ArgoCD root app and app-of-apps entrypoints
infrastructure/   # Cluster infrastructure (networking, storage, monitoring, etc.)
apps/             # Application deployments with kustomize base/overlay structure
config/           # Public sealed secrets cert and encrypted master key
```

## Secrets

Secrets are encrypted with Sealed Secrets. The public cert is in `config/sealed-secret-pub.crt` and can be used to encrypt new secrets without cluster access:

```bash
kubeseal --cert config/sealed-secret-pub.crt --format yaml < secret.yaml > sealed-secret.yaml
```

The master key is stored as a GPG-encrypted file in `config/sealed-secrets-master-key.yaml.gpg`. Without it, sealed secrets cannot be decrypted on a new cluster.

## Bootstrap

ArgoCD is the only thing installed manually:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

After that, applying the root app picks up everything else:

```bash
kubectl apply -f bootstrap/root.yaml
```

The sealed secrets master key needs to be restored before ArgoCD syncs, otherwise secrets will fail to decrypt.
