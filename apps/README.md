One Argo CD `Application` manifest per entry: one persistent Application per
golden-path service (named `<service>-main`). `v1` services (only
`lukas-test`) also get one ephemeral Application per open PR here (named
`<service>-<branch>` or similar), pushed and removed automatically by that
service's own CI. `v2`+ services instead get one `ApplicationSet` here
(named `<service>-previews`) — its generated ephemeral Applications never
land as files in this directory; they exist only as in-cluster state. Never
edit any of this by hand — see the root `README.md`.
