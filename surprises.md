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

## chamberlain-state

_No entries yet._

## usurper

_No entries yet._
