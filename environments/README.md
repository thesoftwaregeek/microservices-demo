# Environments

acme's deployments, one folder per placement (`<jurisdiction>-<environment>/`). The ArgoCD on each of acme's
clusters syncs only its own folder, into namespace `applications`, as AppProject `owner`
(`tenants/acme/project.yaml` in cluster-management names this repo). Put ordinary ArgoCD Applications here, with
`namespace: applications` and `project: owner`; anything else in the folder is applied too, within what `owner`
allows: any namespace on the cluster except `argocd`, `external-secrets`, `kyverno` and `crossplane-system`, and only
`Namespace` at cluster scope.

The cluster reads this repo with a read-only deploy key that acme writes to its store path (OpenBao namespace
`management-nonprod/uk/acme`, key `argocd/repository`: `type: git`, `url`, `sshPrivateKey`).
