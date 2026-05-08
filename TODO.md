# TODO

Things to explore next. Ordered roughly by how self-contained they are, not by priority.

---

## Cilium Service Mesh — mTLS

Envoy proxy (sidecarless) and SPIRE are deployed and running. mTLS is active on all application namespaces.

- ~~Enable Cilium Envoy proxy~~ ✓
- ~~Deploy SPIRE (certificate authority + node agents)~~ ✓
- ~~Configure mTLS per namespace via `CiliumNetworkPolicy` with authentication requirements~~ ✓
- ~~Verify in Hubble that connections show as mTLS-authenticated~~ ✓ (`Auth: SPIRE` visible on every ingress-nginx → app flow)
- Explore traffic splitting via `CiliumEnvoyConfig`

mTLS is applied to all app namespaces (recipit, wordpress, forgejo, headlamp, tuwunel, test-dev). Infrastructure namespaces (kube-system, argocd, monitoring, storage, etc.) are intentionally excluded: if SPIRE has an issue and mTLS blocks coredns, cilium, or ArgoCD, the cluster becomes unreachable or unrecoverable. The security value of mTLS is in authenticating application traffic — that part is done.

**Why:** No Istio complexity, no sidecars, and the CNI is already running. This is what eBPF-based networking actually looks like in practice.

---

## Tetragon — eBPF Security Observability

Also from Cilium. Tetragon hooks into the Linux kernel via eBPF and gives you real-time visibility into what containers are actually doing: syscalls, process executions, network connections, file access — at the kernel level, not the application level.

- Deploy [Tetragon](https://tetragon.io) into the cluster
- Write a `TracingPolicy` to observe a specific app (e.g. what files Recipit touches, what processes Wordpress spawns)
- Watch what chaoskube actually does when it kills a pod at the syscall level
- Optional: hook Tetragon alerts into Alertmanager

**Why:** This is genuinely eye-opening. Most people have never seen what a container does below the container runtime. eBPF makes the invisible visible.
