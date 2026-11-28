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
(`boutique-adservice` ... `boutique-shippingservice`). Each renders this repo's `helm-chart/` at the same pinned commit,
as release `boutique` into namespace `boutique`, with every other service's `<service>.create` flag set to `false`, so a
service syncs, rolls back and shows its health on its own. Together they render exactly the resources the single
`boutique` Application did (same kinds and names, each once).

- `boutique-cartservice` also owns `redis-cart`: the chart defines its Deployment, Service and ServiceAccount in the
  cartservice template, behind the same flag, so there is no separate Redis Application.
- The chart's shared objects (the `deny-all` NetworkPolicy and AuthorizationPolicy in `common.yaml`, the OpenTelemetry
  collector) are all off by default and are not enabled here. If one is switched on, set it in exactly one of the
  files (the shared `deny-all` in `boutique-frontend.yaml`, say), or two Applications will own the same object.
- Resource overrides (more memory for cartservice, currencyservice, paymentservice, emailservice, recommendationservice
  and adservice) live in the file of the service they raise; `externalService: false` is in `boutique-frontend.yaml`.
- The files were generated from the former single Application's values; to change the chart commit, change it in all
  eleven (`sed` over `uk-dev/boutique-*.yaml`).
