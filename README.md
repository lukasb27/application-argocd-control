# application-argocd-control

App-of-apps root for **application-level** Argo CD Applications — one persistent
Application per service on the golden path (its `main` branch), plus one ephemeral
Application per open PR, created and torn down automatically by that service's own
CI. Not general homelab/platform infrastructure — see
[homelab-argocd-control](https://github.com/lukasb27/homelab-argocd-control) for
that, and see its
[ADR on why these are two separate repos](https://github.com/lukasb27/homelab-argocd-control/blob/main/docs/two-argocd-repos-adr.md)
for the reasoning.

For the full picture of how this repo fits into the rest of the golden-path system,
see
[backstage-templates' platform overview](https://github.com/lukasb27/backstage-templates/blob/main/docs/platform-overview.md).

## Bootstrap

One-time only, since ArgoCD has no way to discover a new repo except being told about
it directly:

```
kubectl apply -f main.yaml
```

`main.yaml` lives at the repo root, deliberately outside `apps/` — the root
Application it defines watches `apps/`, so if its own manifest lived inside that
folder it would try to manage (and endlessly re-sync) itself.

After that, never `kubectl apply` anything else in `apps/` by hand. What lands
here depends on a service's `template_version`
(`goldenpath.lukasb27/template-version` in its `catalog-info.yaml`):

- **The `http-service` template's `register-argo-app` scaffolder step**
  (`backstage-templates`) opens a PR here once, at scaffold time, for every
  service's persistent `main`-branch Application — and, for
  `template_version: v2`+ services, its previews `ApplicationSet`
  (`pullRequest` generator) alongside it. Both are static after that: neither
  changes again unless the service is re-scaffolded or the template itself
  changes.
- **`v1` services' own `ephemeral-env.yml`-based CI** (referenced from that
  service's `actions.yml`/`cleanup.yml`) pushes and removes that service's
  per-PR ephemeral Application manifests directly here, on every PR
  open/close. Only `lukas-test` still runs on this path — see
  [backstage-templates' ApplicationSet migration ADR](https://github.com/lukasb27/backstage-templates/blob/main/docs/argocd-applicationset-migration-adr.md).

Each previews `ApplicationSet`'s `pullRequest` generator authenticates via
`github-pr-generator-token`, a Secret held once, centrally, in the `argocd`
namespace — not a per-service secret, and not tracked in this repo. See
[homelab-argocd-control](https://github.com/lukasb27/homelab-argocd-control)
for where that's actually provisioned.

**`v2`+ services' per-PR ephemeral Applications are not git objects here at
all.** Their previews `ApplicationSet` (delivered once, as above) generates
and deletes them directly against the cluster as PRs open and close — the
root Application only ever sees the `ApplicationSet` resource itself; every
`Application` it generates exists purely as in-cluster state, same as any
other `ApplicationSet`-managed child. `git log` on this repo stops being a
complete audit trail of ephemeral-environment lifecycle for `v2`+ services —
accepted trade-off, see the ADR.

The root Application (automated sync, prune) picks up all of it on its own. Paths
and namespaces are prefixed by app name, so multiple services' entries coexist here
safely without colliding.

## History

This repo has been renamed twice:

1. `lukas-argocd-control` → `fermentation-station-argocd-control`, when it was still
   scoped to one service.
2. `fermentation-station-argocd-control` → `application-argocd-control` (current),
   once every golden-path service started sharing it — the earlier name no longer
   described its real role. GitHub's redirect means old clones and links from either
   previous name still resolve, but new clones should use the current name.
