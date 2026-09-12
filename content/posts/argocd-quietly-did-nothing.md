---
title: "Three ways ArgoCD quietly did nothing"
date: 2026-09-12
summary: >
  GitOps sounds simple until you watch it silently ignore your commits. Three real bugs
  from getting chamberlain's first tenant-to-pod flow working, and the 98.5-second number
  that came out the other side.
---

`chamberlain`'s first real feature is small on purpose: a tenant asks for a workload, and it
starts running, with no human in the loop. The write path is deliberately "pure GitOps" — the
API never touches the cluster directly, it only commits a manifest to `chamberlain-state` and
waits for ArgoCD to do the rest. That's one sentence to write and most of a day to make actually
work, because ArgoCD failed silently in three different ways before it worked at all.

**It doesn't recurse into subdirectories.** Tenant and workload manifests live at
`tenants/<name>/tenant.yaml` and `workloads/<tenant>/<name>.yaml` — a natural layout for anything
that writes one file per resource. ArgoCD's plain-directory source doesn't look inside
subdirectories unless you tell it to. It applied the one YAML file sitting at the repo root and
silently ignored everything nested below it — no error, no warning, `Synced` and `Healthy` the
whole time, just... not there. Fix: `directory.recurse: true`.

**It caches what CRDs exist at boot.** The `Application` was created before the
`Tenant`/`Workload` CRDs existed in the cluster (a reasonable-looking order: install ArgoCD, then
apply your CRDs). The `application-controller` builds its API-discovery cache once, early, and
never rechecks it — so every `Tenant`/`Workload` manifest was invisible to it until I manually
restarted the controller pod. Now `just up` applies CRDs and restarts the controller *before* the
`Application` ever gets created.

**A resync-interval config doesn't mean what it sounds like.** I set
`timeout.reconciliation: 10s`, confirmed via logs that the controller really was reconciling
every 6-10 seconds — and a commit still took over two minutes to show up. Turns out that setting
bounds how often the controller re-compares against the *last known* git revision; noticing a
*new* revision exists is gated by the repo-server's own fetch/cache path, which isn't the same
knob. I never ran that one all the way to ground — I just stopped fighting it and quadrupled my
timeout instead.

The honest number, after all three fixes: **98.5 seconds** from `chamberlain workload create` to
a running pod, on a fresh cluster, and nearly all of it is ArgoCD noticing a commit exists. Not
fast. Real, though, and now it's a baseline instead of a mystery.
