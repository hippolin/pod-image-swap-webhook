# Repository Guidelines

## Project Structure & Module Organization

`main.go` wires configuration, controller-runtime, health checks, metrics, and the admission endpoint. Core packages live under `pkg/`: `pkg/admission` normalizes and patches Pod images, while `pkg/config` loads and validates replacement rules. Keep Go tests beside their implementation as `*_test.go`. The Helm chart is in `charts/pod-image-swap-webhook`, with defaults in `values.yaml` and resources under `templates/`. Use `config.sample.yaml` as the documented configuration reference; site metadata lives in `mkdocs.yml` and `docs/`.

## Build, Test, and Development Commands

- `make build` builds the `pod-image-swap-webhook` binary.
- `make test` runs all Go tests with the race detector by default.
- `make coverage` writes `coverage.txt` and prints package coverage.
- `make vet` runs Go's static analyzer.
- `make lint` runs golangci-lint; MegaLinter performs broader CI checks.
- `make docker-build IMAGE=pod-image-swap-webhook TAG=dev` builds a local image.
- `helm lint charts/pod-image-swap-webhook` validates the chart.
- `helm template test charts/pod-image-swap-webhook --namespace kube-system` renders manifests for review.

For direct execution, provide a configuration path with `PISW_CONFIG_PATH`; the webhook server also requires TLS files through `PISW_CERT_DIR`.

## Coding Style & Naming Conventions

Format Go code with `gofmt` and follow standard Go naming: short lowercase package names, exported identifiers in PascalCase, and descriptive table-test names. Keep admission behavior in `pkg/admission` and configuration concerns in `pkg/config`. YAML uses two-space indentation and a 120-character line limit; Helm templates are excluded from normal YAML linting because of template syntax.

## Testing Guidelines

Use Go's `testing` package with `testify/assert` and `require`. Prefer table-driven cases for image normalization, exclusion precedence, replacement patches, malformed AdmissionReviews, and configuration validation. Name tests `TestType_Action` or `TestFunction`. Run `make test`, `make vet`, and Helm lint/template checks before opening a PR.

## Commit & Pull Request Guidelines

History follows Conventional Commit-style subjects such as `chore(deps): bump k8s.io/api` and `fix(github): skip uploading existing chart versions`. Use a lowercase type (`feat`, `fix`, `test`, `docs`, `chore`) and an optional focused scope. Keep commits single-purpose. PRs should explain behavior changes, list validation commands, link related issues, and include rendered manifest excerpts when Helm output changes. Highlight changes to TLS, `failurePolicy`, selectors, or image-matching semantics because they affect cluster-wide admission behavior.

## Security & Configuration Tips

Never commit private keys, generated certificates, `config.yaml`, or local `values.yaml`. Preserve exclusion-before-replacement ordering and add regression tests for registry and repository boundaries. Verify webhook certificate SANs against the rendered Service DNS name for non-default Helm release names.
