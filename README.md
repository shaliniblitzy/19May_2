# 19May_2

An opinionated, cloud-native DevOps platform reference architecture
covering the full software-delivery lifecycle: **plan → code → build →
test → secure → release → operate → observe → cost-account**. Every
artifact in this repository is declarative, versioned, signed, and
reconciled by a GitOps controller so that downstream adopters can fork
the platform, replace the cloud and registry endpoints, and have an
end-to-end delivery pipeline running on day one.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![CI](https://img.shields.io/badge/CI-GitHub%20Actions-2088FF.svg?logo=githubactions&logoColor=white)](.github/workflows/ci.yml)
[![Docs](https://img.shields.io/badge/Docs-MkDocs%20Material-526CFE.svg?logo=materialformkdocs&logoColor=white)](mkdocs.yml)
[![SLSA Level 3](https://img.shields.io/badge/SLSA-Level%203-7B42BC.svg)](https://slsa.dev/spec/v1.0/levels#build-l3)
[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-FE5196.svg?logo=conventionalcommits&logoColor=white)](https://www.conventionalcommits.org/en/v1.0.0/)

## What's Inside

The platform is composed from best-of-breed cloud-native projects, each
slotted into a single, well-defined concern. Defaults below are
opinionated; ADRs under [`docs/adr/`](docs/adr/) explain every choice
and a documented alternative.

- **CI/CD primary** — [GitHub Actions](docs/components/github-actions.md)
  with reusable workflows under
  [`.github/workflows/reusable-*.yml`](.github/workflows/); GitLab CI
  component templates also provided under
  [`.gitlab-ci/`](.gitlab-ci/).
- **GitOps** — [Argo CD](docs/components/argocd.md) (primary) under
  [`gitops/argocd/`](gitops/argocd/); Flux alternative under
  [`gitops/flux/`](gitops/flux/).
- **Infrastructure as Code** —
  [Terraform / OpenTofu](docs/components/terraform.md) under
  [`infrastructure/terraform/`](infrastructure/terraform/);
  [Crossplane](docs/components/crossplane.md) under
  [`infrastructure/crossplane/`](infrastructure/crossplane/).
- **Policy as Code** — [Kyverno](docs/components/kyverno.md) plus
  [OPA Gatekeeper](docs/components/gatekeeper.md) under
  [`policy/`](policy/); Conftest, Checkov, and tfsec for pre-merge
  IaC scanning.
- **Supply chain** — [Cosign](docs/components/cosign.md) signing,
  [Syft / Grype / Trivy](docs/components/syft-grype-trivy.md) for
  SBOM + vulnerability scanning, and SLSA provenance under
  [`security/supply-chain/`](security/supply-chain/).
- **Observability** —
  [OpenTelemetry](docs/components/opentelemetry.md) feeding
  [Prometheus + Grafana](docs/components/prometheus-grafana.md),
  [Loki + Tempo](docs/components/loki-tempo.md) under
  [`observability/`](observability/).
- **Progressive delivery** —
  [Argo Rollouts](docs/components/argo-rollouts.md) and
  [Flagger](docs/components/flagger.md) under
  [`progressive-delivery/`](progressive-delivery/).
- **Secrets** —
  [External Secrets Operator](docs/components/external-secrets.md) +
  HashiCorp Vault under [`security/secrets/`](security/secrets/).
- **Internal Developer Platform** —
  [Backstage](docs/components/backstage.md) with software templates
  under [`backstage/`](backstage/).
- **FinOps** — [OpenCost](docs/components/opencost.md) under
  [`finops/`](finops/).

## Repository Layout

```text
19May_2/
├── .github/                   # GitHub Actions workflows + dependabot
├── .gitlab-ci/                # GitLab CI component includes
├── backstage/                 # Internal Developer Portal + templates
├── docs/                      # MkDocs-rendered documentation site
├── examples/                  # Sample service consuming the platform
├── finops/                    # Cost allocation (OpenCost) + budgets
├── gitops/                    # Argo CD + Flux desired-state manifests
├── helm/                      # Umbrella Helm chart aggregating sub-charts
├── infrastructure/            # Terraform modules/envs + Crossplane
├── kubernetes/                # Base + overlays (Kustomize)
├── observability/             # Prometheus, Grafana, Loki, Tempo, OTel
├── policy/                    # Kyverno, Gatekeeper, OPA, IaC policy
├── progressive-delivery/      # Argo Rollouts + Flagger
├── scripts/                   # Bootstrap and helper scripts
├── security/                  # Supply-chain + secrets management
├── CODEOWNERS                 # Reviewer assignments
├── CODE_OF_CONDUCT.md         # Contributor Covenant 2.1
├── CONTRIBUTING.md            # Contribution workflow
├── LICENSE                    # Apache-2.0
├── README.md                  # You are here
├── SECURITY.md                # Vulnerability disclosure
├── .editorconfig              # Editor conventions
├── .gitattributes             # Git attributes
├── .gitignore                 # Git ignore
├── .gitlab-ci.yml             # Top-level GitLab pipeline
├── .pre-commit-config.yaml    # Pre-commit hooks
└── mkdocs.yml                 # MkDocs site config
```

Each top-level directory is a single, non-overlapping concern. New
contributors should be able to locate the file they need by reading
this tree alone — see
[`docs/architecture/component-map.md`](docs/architecture/component-map.md)
for the full responsibility matrix.

## Prerequisites

The platform is designed to be operable from any POSIX-compliant
workstation. The list below names the **minimum** versions verified
against the reference; newer patch releases of the same minor line are
always preferred.

- [`git`](https://git-scm.com/), `make`, and `bash` (≥ 4)
- [`kubectl`](https://kubernetes.io/docs/reference/kubectl/) (≥ 1.30.x)
- [`helm`](https://helm.sh/) (≥ 3.15.x)
- [`kustomize`](https://kustomize.io/) (≥ 5.4.x)
- [`terraform`](https://developer.hashicorp.com/terraform) (≥ 1.9.x)
  **or** [`opentofu`](https://opentofu.org/) (≥ 1.8.x)
- [`cosign`](https://github.com/sigstore/cosign) (≥ 2.4.x),
  [`syft`](https://github.com/anchore/syft) (≥ 1.x),
  [`grype`](https://github.com/anchore/grype) (≥ 0.x),
  [`trivy`](https://github.com/aquasecurity/trivy) (≥ 0.55.x)
- [`pre-commit`](https://pre-commit.com/) (≥ 3.x)
- [Docker](https://www.docker.com/) (or a compatible OCI builder such
  as [Podman](https://podman.io/) or
  [BuildKit](https://github.com/moby/buildkit)) for local image builds
- Access to a Kubernetes cluster —
  [kind](https://kind.sigs.k8s.io/) or
  [minikube](https://minikube.sigs.k8s.io/) for local; any managed
  service for cloud. Cluster provisioning is documented in
  [`docs/getting-started/prerequisites.md`](docs/getting-started/prerequisites.md).

Run [`scripts/install-tooling.sh`](scripts/install-tooling.sh) to
install every CLI above at the pinned version into `/usr/local/bin`.

## Quickstart

The complete walkthrough lives in
[`docs/getting-started/quickstart.md`](docs/getting-started/quickstart.md);
the seven steps below are the executive summary that gets you from a
clean clone to a running platform.

1. **Clone the repository.**

   ```bash
   git clone <your-fork-url> 19May_2 && cd 19May_2
   ```

2. **Install the pinned CLI tooling.**

   ```bash
   ./scripts/install-tooling.sh
   ```

3. **Create (or select) a Kubernetes cluster.**

   ```bash
   ./scripts/bootstrap-cluster.sh   # creates a local kind cluster
   ```

4. **Install Argo CD and apply the platform `AppProject` and
   `ApplicationSet`.**

   ```bash
   ./scripts/bootstrap-argocd.sh
   ```

5. **Wait for the GitOps controller to reconcile.** Argo CD will pull
   the manifests under [`gitops/argocd/`](gitops/argocd/) and roll out
   Kyverno, Gatekeeper, the kube-prometheus-stack, Loki, Tempo, the
   OpenTelemetry Collector, External Secrets Operator, OpenCost, and
   Backstage. Watch progress with:

   ```bash
   argocd app list && kubectl get applications -n argocd
   ```

6. **Deploy the working example.** The
   [`examples/sample-service/`](examples/sample-service/) directory
   ships a minimal hello-world wired through every platform component
   — CI, signing, GitOps, progressive rollout, and observability.

7. **Open the UIs.** Argo CD, Grafana, and the Backstage portal each
   expose a web UI; their initial credentials are surfaced as
   Kubernetes `Secret` objects in their respective namespaces:

   ```bash
   kubectl -n argocd   get secret argocd-initial-admin-secret -o yaml
   kubectl -n monitoring get secret grafana-admin-credentials  -o yaml
   ```

## Documentation

The full documentation set is rendered with
[MkDocs Material](https://squidfunk.github.io/mkdocs-material/) from
[`mkdocs.yml`](mkdocs.yml). Run `mkdocs serve` after installing the
docs dependencies (`pip install mkdocs-material`) to preview the site
locally at <http://127.0.0.1:8000/>.

- [`docs/index.md`](docs/index.md) — documentation site landing page
- [`docs/getting-started/`](docs/getting-started/) — prerequisites,
  quickstart, and team onboarding
- [`docs/architecture/`](docs/architecture/) — platform overview,
  component map, data flow, network, security model, and supply-chain
  posture
- [`docs/components/`](docs/components/) — one page per platform
  component (CI runners, GitOps controllers, IaC engines, policy
  engines, signing + scanning, observability, IDP, secrets, cost)
- [`docs/adr/`](docs/adr/) — Architecture Decision Records (Nygard
  format) explaining every major technology choice
- [`docs/runbooks/`](docs/runbooks/) — incident response, on-call,
  rollback, secret rotation, and image-vulnerability response
- [`docs/glossary.md`](docs/glossary.md) — term definitions

## Design Principles

The architecture is shaped by a small set of non-negotiable rules.
They are restated in [`CONTRIBUTING.md`](CONTRIBUTING.md) and enforced
in CI:

- **Declarative-first.** Every cluster, infrastructure, and policy
  change is a Git commit reconciled by a GitOps controller — no
  imperative `kubectl apply` from pipelines or workstations.
- **No production secrets in Git.** Only `SecretStore` and
  `ExternalSecret` references are committed; secret values live in
  Vault or the cloud's native secret store and are projected into the
  cluster by the External Secrets Operator.
- **Every image is signed, scanned, and inventoried.** Cosign keyless
  signing, Trivy and Grype vulnerability scanning, and a Syft SBOM
  are attached to every OCI artifact produced by the platform.
- **Every pull request runs the same gates.** Lint + test + SAST +
  IaC-scan + policy-test — wired into
  [`.github/workflows/ci.yml`](.github/workflows/ci.yml) via the
  reusable workflows.
- **Pin third-party actions by commit SHA**, never by floating tag —
  enforced by review and Dependabot.
- **Pin Helm chart versions exactly** in
  [`helm/platform/Chart.yaml`](helm/platform/) — no `>=`, no `~`.
- **Trunk-based development** with branch protection,
  [`CODEOWNERS`](CODEOWNERS)-driven review, and signed commits.

## Contributing

Contributions are welcome — see [`CONTRIBUTING.md`](CONTRIBUTING.md)
for the full workflow. In short: open an issue, fork, branch off
`main`, sign your commits ([Developer Certificate of
Origin](https://developercertificate.org/)), use
[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/),
run `pre-commit run --all-files` locally, and open a pull request.
All participants are bound by the
[`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md) (Contributor Covenant 2.1).

## Security

Found a vulnerability? Please follow the coordinated disclosure
process documented in [`SECURITY.md`](SECURITY.md). Do **not** open a
public GitHub issue for security reports.

## License

This project is licensed under the [Apache License 2.0](LICENSE).
SPDX-License-Identifier: `Apache-2.0`.

## Status

- **Reference architecture, not a turnkey deployment.** This
  repository is intended to be forked, parameterized for your
  organization, and progressively adopted — not deployed verbatim.
- **Live cloud provisioning is out of scope.** The Terraform code
  under [`infrastructure/terraform/`](infrastructure/terraform/) is
  authored, formatted, validated, and policy-scanned in CI, but
  `terraform apply` against a real cloud account is the adopter's
  responsibility. `backend.tf.example` is provided as a template.
- **Cloud-provider choice is deliberately deferred.** The reference
  is cloud-agnostic; per-cloud bindings (AWS, GCP, Azure) are
  discussed in
  [`docs/adr/0004-choose-terraform-and-crossplane.md`](docs/adr/0004-choose-terraform-and-crossplane.md)
  and surfaced as Crossplane provider packages under
  [`infrastructure/crossplane/providers/`](infrastructure/crossplane/).
- **Production sizing is your job.** Default
  [`observability/`](observability/) and
  [`helm/platform/values.yaml`](helm/platform/) values target a
  small-to-medium development cluster (≤ 20 nodes); each component
  page in [`docs/components/`](docs/components/) documents the
  scale-up path (Thanos / Mimir for Prometheus, microservices mode
  for Loki and Tempo, and so on).
