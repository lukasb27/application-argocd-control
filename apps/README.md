One Argo CD `Application` manifest per entry: one persistent Application per
golden-path service (named `<service>-main`), plus one ephemeral Application per
open PR (named `<service>-<branch>` or similar), pushed and removed automatically by
each service's own CI. Never edit these by hand — see the root `README.md`.
