# TODO

Things to explore next. Ordered roughly by how self-contained they are, not by priority.

---

## Cilium Service Mesh — mTLS

Envoy proxy (sidecarless) and SPIRE are deployed and running. The foundation for mTLS is in place.

- ~~Enable Cilium Envoy proxy~~ ✓
- ~~Deploy SPIRE (certificate authority + node agents)~~ ✓
- Configure mTLS per namespace via `CiliumNetworkPolicy` with authentication requirements
- Verify in Hubble that connections show as mTLS-authenticated
- Explore traffic splitting via `CiliumEnvoyConfig`

**Why:** No Istio complexity, no sidecars, and the CNI is already running. This is what eBPF-based networking actually looks like in practice.

---

## External Secrets Operator

Sealed Secrets works, but secrets still live in Git (encrypted). External Secrets Operator flips the model: secrets live in a real secret store and are synced into Kubernetes at runtime. Nothing secret ever touches the repo.

- Deploy [External Secrets Operator](https://external-secrets.io)
- Set up a backend — HashiCorp Vault is the most educational, but a simple option is running [Infisical](https://infisical.com) or using an existing cloud secret manager
- Migrate one app (Recipit or Forgejo) off Sealed Secrets as a proof of concept
- Compare the operational model: rotation, auditing, access control vs. the Sealed Secrets approach

**Why:** Different philosophy entirely. Understanding both models is valuable for real-world work.

---

## TrueNAS via OpenTofu

Proxmox VMs and Talos bootstrap are fully automated (`tofu apply` builds the full stack). The remaining manual gap is TrueNAS — datasets, shares, and storage pools are still configured by hand.

- Add TrueNAS to the OpenTofu setup using the [TrueNAS provider](https://registry.terraform.io/providers/dariusbakunas/truenas)
- Manage iSCSI and NFS targets as code alongside the rest of the infrastructure

**Why:** Closes the last manual step in the stack.

---

## Tetragon — eBPF Security Observability

Also from Cilium. Tetragon hooks into the Linux kernel via eBPF and gives you real-time visibility into what containers are actually doing: syscalls, process executions, network connections, file access — at the kernel level, not the application level.

- Deploy [Tetragon](https://tetragon.io) into the cluster
- Write a `TracingPolicy` to observe a specific app (e.g. what files Recipit touches, what processes Wordpress spawns)
- Watch what chaoskube actually does when it kills a pod at the syscall level
- Optional: hook Tetragon alerts into Alertmanager

**Why:** This is genuinely eye-opening. Most people have never seen what a container does below the container runtime. eBPF makes the invisible visible.
