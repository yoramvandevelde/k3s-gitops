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

- ~~Deploy [Tetragon](https://tetragon.io) into the cluster~~ ✓
- ~~Write `TracingPolicy` to observe specific apps~~ ✓ (three policies: shell exec detection cluster-wide, sensitive file access in recipit-dev, TCP connect tracking cluster-wide)
- Watch what chaoskube actually does when it kills a pod at the syscall level
- Build a web UI for Tetragon events (JSON stream from `kubectl exec … tetra getevents`)
- Optional: hook Tetragon alerts into Alertmanager

---

## Matrix / Tuwunel — WebRTC calls

- ~~Deploy coturn TURN server for voice/video calls~~ ✓ (MetalLB IP `10.10.99.81`, relay ports `49152-49161`)
- ~~Configure Tuwunel with TURN URIs and shared HMAC secret~~ ✓
- Re-enable federation when ready (`CONDUWUIT_ALLOW_FEDERATION=true`)

---

## Certificates

- ~~Switch `*.local.sifft.io` issuer from letsencrypt-staging to letsencrypt-prod~~ ✓
- ~~Add `*.sifft.io` wildcard cert and propagate via Reflector~~ ✓ (manually imported, valid until 2026-07-21)
- Re-add `Certificate` resource for `*.sifft.io` once Let's Encrypt status page shows no issues — must be done before 2026-07-21

---

## Observability

- ~~Deploy kube-prometheus-stack~~ ✓
- ~~Configure Grafana dashboards via GitOps (ArgoCD, Cilium, ingress-nginx, Tetragon)~~ ✓
- ~~Deploy metrics-server~~ ✓
- ~~Deploy VPA recommender + Goldilocks~~ ✓
- ~~Deploy Trivy Operator for vulnerability scanning~~ ✓
- ~~Deploy Reloader~~ ✓

**Why:** This is genuinely eye-opening. Most people have never seen what a container does below the container runtime. eBPF makes the invisible visible.
