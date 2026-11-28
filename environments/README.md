# Environments

acme's deployments, one folder per placement (`<jurisdiction>-<environment>/`). The ArgoCD on each of acme's
clusters syncs only its own folder, into namespace `applications`, as AppProject `owner`
(`tenants/acme/project.yaml` in cluster-management names this repo). Put ordinary ArgoCD Applications here, with
`namespace: applications` and `project: owner`; anything else in the folder is applied too, within what `owner`
allows: any namespace on the cluster except `argocd`, `external-secrets`, `kyverno` and `crossplane-system`, and only
`Namespace` at cluster scope.

The cluster reads this repo with a read-only deploy key that acme writes to its store path (OpenBao namespace
`management-nonprod/uk/acme`, key `argocd/repository`: `type: git`, `url`, `sshPrivateKey`).

## uk-dev: the boutique, one Application per service

`uk-dev/boutique-<service>.yaml` are eleven ordinary ArgoCD Applications, one per Online Boutique service
(`boutique-adservice` ... `boutique-shippingservice`). Each installs that service's own chart `boutique-<service>` from
`oci://ghcr.io/thesoftwaregeek/charts` (source `repoURL: ghcr.io/thesoftwaregeek/charts`, `chart`, `targetRevision`)
as release `boutique` into namespace `boutique`, so object names are unchanged and a service syncs, rolls back and shows
its health on its own. The image is `ghcr.io/thesoftwaregeek/boutique/<service>`, tagged with the chart version.

- Release flow: a Git tag `<service>-<version>` in this repo builds and signs the image; a pull request in
  `helm-charts` bumps the chart (`boutique-<service>`, same version); a pull request here bumps that file's
  `targetRevision`. Rollback is moving `targetRevision` back to the previous version.
- `boutique-cartservice` also owns `redis-cart`: the chart defines its Deployment, Service and ServiceAccount, so there
  is no separate Redis Application.
- The charts have no shared objects (no `deny-all` policy, no OpenTelemetry collector); the old `helm-chart/` in this
  repo is no longer used by uk-dev.
- `valuesObject` holds only the overrides, in the chart's own shape: more memory for cartservice, currencyservice,
  paymentservice, emailservice, recommendationservice and adservice (the release limits OOM-kill on a busy laptop), and
  `externalService: false` in `boutique-frontend.yaml`.

