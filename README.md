# fermentation-station-argocd-control

App-of-apps root for fermentation-station-agent's ephemeral-per-PR environments (not general homelab
infra — see [homelab-argocd-control](https://github.com/lukasb27/homelab-argocd-control) for that).

## Bootstrap

One-time only, since ArgoCD has no way to discover a new repo except being told about it directly:

```
kubectl apply -f main.yaml
```

`main.yaml` lives at the repo root, deliberately outside `apps/` — the root Application it defines
watches `apps/`, so if its own manifest lived inside that folder it would try to manage (and endlessly
re-sync) itself.

After that, never `kubectl apply` anything else in `apps/` by hand — `actions.yml` and `cleanup.yml` in
`fermentation-station-agent` push per-PR Application manifests here directly, and the root Application
(automated sync, prune) picks up changes on its own.

This repo was previously named `lukas-argocd-control`; old clones and links still resolve via GitHub's
redirect, but new clones should use the current name.