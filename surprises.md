# Surprises

Append-only, dated log of what didn't go as expected, across every repo in the portfolio.
Organized by repo. Entries are never edited away — wrong calls get recorded, not erased.

Format per entry:

- **Expected:**
- **Actually:** (with the number or the error)
- **Changed:**

## portfolio-site (this repo)

### 2026-09-11 — `surprises.md` location

- **Expected:** every repo would carry its own `surprises.md`, per the original build-spec
  standing rule.
- **Actually:** written content only ever lived on the site anyway, so a per-repo file meant
  either duplication or an empty stub in `chamberlain` / `chamberlain-state` / `usurper`.
- **Changed:** `build-spec.md` amended — one central `surprises.md` here, organized by
  repo/stage; other repos link to the relevant section instead.

## chamberlain

### 2026-09-11 — `just up` works, ArgoCD sync doesn't (yet)

- **Expected:** `just up` would bring up a kind cluster with ArgoCD synced against
  `chamberlain-state`, fully green, since both repos exist and the `Application` manifest is
  correct.
- **Actually:** cluster creation and ArgoCD install succeed cleanly, but the `Application`
  errors with `authentication required` — `chamberlain-state` only exists as a local repo, not
  pushed to GitHub, so ArgoCD can't clone it.
- **Changed:** nothing yet — deliberately left unpushed for now. `just up`/`just down` are
  verified as far as they can be without a remote; the Stage 0 checklist item isn't fully clear
  until `chamberlain-state` (and `chamberlain`) are pushed to GitHub.

### 2026-09-11 — repos pushed, sync green after a flaky first refresh

- **Expected:** once `chamberlain`, `chamberlain-state`, and `usurper` were pushed to GitHub,
  `just up` would sync clean on the first try.
- **Actually:** the `Application` first reported `ComparisonError: connection refused` to
  `argocd-repo-server:8081` — the application-controller raced the repo-server's own startup.
  A hard refresh (`kubectl annotate ... argocd.argoproj.io/refresh=hard`) immediately resolved
  it; `chamberlain-placeholder` namespace confirmed live in-cluster afterward.
- **Changed:** nothing in the justfile for now — a one-off startup race on a fresh install, not
  a real sync issue. Worth adding a short wait/retry in `just up` if it recurs.

### 2026-09-11 — `+kubebuilder:rbac` markers on a struct generate nothing, silently

- **Expected:** `+kubebuilder:rbac:...` marker comments placed directly above the
  `TenantReconciler`/`WorkloadReconciler` struct types (the pattern shown in every kubebuilder
  tutorial) would generate `config/rbac/role.yaml` via `controller-gen rbac:roleName=...`.
- **Actually:** `controller-gen` exited 0 with zero bytes of output, no error, no file, on both
  v0.16.5 and v0.22.0. Isolated in a minimal throwaway module: the marker's registered `target`
  is `"package"` (confirmed via `controller-gen rbac -wwww`), and in practice it's only collected
  when it's on a **package-level doc comment**, not when attached to an arbitrary type — despite
  `object`/`crd` generators happily reading markers off types in the same files.
- **Changed:** moved all `+kubebuilder:rbac` markers into a dedicated `internal/controller/doc.go`
  package comment instead of decorating the reconciler structs. No other generator was affected.

### 2026-09-11 — a controller can't grant a Role more than it itself holds

- **Expected:** `TenantReconciler` creating a namespace-scoped `Role` with
  `{APIGroups:["*"],Resources:["*"],Verbs:["*"]}` for each tenant would just work — the manager's
  own ClusterRole is separate from what it hands out to tenants.
- **Actually:** `roles.rbac.authorization.k8s.io "tenant-owner" is forbidden: ... attempting to
  grant RBAC permissions not currently held`. Kubernetes' RBAC escalation-prevention check blocks
  any principal from creating a Role/ClusterRole/Binding that grants more than it itself holds —
  a wildcard tenant Role would need a wildcard manager ClusterRole, defeating least-privilege.
- **Changed:** scoped `tenant-owner` down to read-only on `workloads`/`deployments`/`services`/
  `pods` — a real subset of `manager-role` — instead of a wildcard. This is arguably the correct
  design anyway (Stage 1 doesn't need tenants to have full namespace admin), but it wasn't a
  deliberate choice until the escalation check forced it.

### 2026-09-12 — ArgoCD's `timeout.reconciliation` doesn't bound git-commit detection latency

- **Expected:** setting `argocd-cm`'s `timeout.reconciliation` to `10s` (and confirming via logs
  that `appResyncPeriod=10s` and the controller genuinely reconciles every ~6-10s) would mean a
  fresh commit to `chamberlain-state` gets picked up within ~10s.
- **Actually:** it didn't. A commit at `14:40:15Z` wasn't reflected in the `Application`'s
  resource list until `14:42:29Z` — ~134s later — despite the controller visibly reconciling the
  whole time. The resync loop re-compares against the *last known remote revision*, and
  discovering a *new* remote revision is gated by the repo-server's own git-fetch/cache path,
  which `timeout.reconciliation` doesn't control. Never isolated the exact repo-server setting;
  didn't chase it further given the "leave it pure GitOps" decision already accepted this
  tradeoff.
- **Changed:** bumped the API's poll timeout from 90s to 240s so `chamberlain workload create`
  fails on genuine breakage rather than on normal GitOps propagation latency. Also caught (and
  fixed) two other real bugs this exposed: ArgoCD's plain-directory source doesn't recurse into
  subdirectories by default (`tenants/<name>/tenant.yaml` was silently ignored until
  `directory.recurse: true` was added to the `Application`), and CRDs must be applied *before*
  the application-controller's first start or its API-discovery cache goes stale for every
  `Application` until the controller is restarted — `just up` now applies CRDs and restarts the
  controller before creating the `Application`.

**Evidence — Stage 1's "time from create to serving" metric:** `chamberlain workload create
--tenant acme --name hello --image nginx:alpine` on a fresh `just up` cluster: **98.5s**
(server-measured: 98521ms), commit `chamberlain@520371b`, harness: `cmd/chamberlain` +
`cmd/api` as built in this repo. This number is almost entirely ArgoCD GitOps propagation
latency, not chamberlain's own code — worth revisiting if a later stage needs faster
tenant-facing turnaround.

## chamberlain-state

_No entries yet._

## usurper

_No entries yet._
