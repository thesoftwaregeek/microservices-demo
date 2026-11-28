# GitHub Actions workflows

This repo is acme's fork of Online Boutique. Google's GKE-based workflows (deploy tests per pull request, cleanup,
release builder, Terraform and Kubevious checks) were removed: they need Google Cloud credentials and a test cluster
that this repo does not have.

| Workflow | Runs on | What it does |
|---|---|---|
| [ci.yaml](ci.yaml) | pull request to `main`, push to `main`, and called by the release | Go unit tests (`shippingservice`, `productcatalogservice`, `frontend/validator`) and C# tests (`cartservice`) |
| [release.yaml](release.yaml) | push of a tag `<service>-<major>.<minor>.<patch>`; `workflow_dispatch` with a `tag` input | Tests, then builds `src/<service>` for `linux/amd64` and `linux/arm64`, pushes `<registry>/boutique/<service>:<version>`, scans it with Trivy (fixable CRITICAL fails) and signs it by digest with keyless cosign |
| [helm-chart-ci.yaml](helm-chart-ci.yaml) | changes under `helm-chart/` | `helm lint` and `helm template` combinations |
| [kustomize-build-ci.yaml](kustomize-build-ci.yaml) | changes under `kustomize/` | builds the base and every combination in `kustomize/tests` |

## Releasing an image

```bash
git tag cartservice-0.1.0 && git push origin cartservice-0.1.0
```

- Services: adservice, cartservice, checkoutservice, currencyservice, emailservice, frontend, loadgenerator,
  paymentservice, productcatalogservice, recommendationservice, shippingservice. Any other service name is rejected.
- The registry is `vars.PACKAGE_REGISTRY`, default `ghcr.io/<owner>`. An existing version is never overwritten.
- For `workflow_dispatch`, choose the tag itself under "Use workflow from": the signature records the ref the run
  started from.
- The signature identity is `.../microservices-demo/.github/workflows/release.yaml@refs/tags/<service>-<version>`.
  The helm-charts mirror gate (`VENDOR_REPOS`) currently accepts only `refs/tags/v<major>.<minor>.<patch>`, so it will
  not accept these images until its identity pattern is widened to the `<service>-<version>` form.
