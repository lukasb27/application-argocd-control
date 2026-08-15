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

After that, never `kubectl apply` anything else in `apps/` by hand. Two kinds of
writers push here automatically:

- **The `http-service` template's `register-argo-app` scaffolder step**
  (`backstage-templates`) opens a PR here for every newly scaffolded service's
  persistent `main`-branch Application, once, at scaffold time.
- **Each service's own `ephemeral-env.yml`-based CI** (referenced from that
  service's `actions.yml`/`cleanup.yml`) pushes and removes that service's per-PR
  ephemeral Application manifests directly, on every PR open/close.

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
