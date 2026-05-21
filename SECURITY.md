# Security Policy

The 19May_2 DevOps Platform Reference Architecture takes the security of the
platform — and the supply chain through which it is delivered — seriously.
This document explains how to report vulnerabilities responsibly.

The maintainers operate this repository as a public reference architecture.
Many adopters embed these manifests, modules, and reusable workflows directly
into their own platforms, which means a vulnerability here may propagate
quickly across many downstream consumers. Coordinated disclosure protects
those consumers while the fix is being prepared.

## Supported Versions

The table below lists which release lines currently receive security fixes:

| Version  | Supported          |
|----------|--------------------|
| 1.x      | :white_check_mark: |
| < 1.0    | :x:                |

> **Note:** Pre-1.0 releases are reference snapshots and may not receive
> backports; adopters are encouraged to track the latest minor release.

Each minor line (for example `1.1.x`, `1.2.x`) is supported for security
fixes until the next minor release ships plus a 30-day grace window so
adopters have time to upgrade.

## Reporting a Vulnerability

If you believe you have discovered a security vulnerability in this
reference architecture — whether in a manifest, a workflow, a Terraform
module, a policy bundle, a script, or the documentation itself — please
follow the steps below.

1. **Do not open a public issue.** Use one of the private disclosure
   channels described below. A public issue exposes the vulnerability to
   anyone watching the repository before a fix is available.
2. Use GitHub's **Private Vulnerability Reporting** workflow — click the
   **Report a vulnerability** button on the **Security** tab of this
   repository. This is the **preferred** channel because it creates a
   private advisory automatically, lets the maintainers collaborate with
   you inside GitHub, and produces a publishable Security Advisory once
   the fix is released.
3. Alternatively, email `security@example.org` (placeholder — adopters of
   this reference architecture **MUST** replace this address with their own
   security mailbox before publishing). Include all of the following:
   - Affected file paths or platform components
   - Steps to reproduce the issue
   - Impact assessment (what an attacker can achieve, blast radius,
     prerequisites, and any known exploit chain)
   - Suggested remediation or mitigation, if known
   - Whether you would like to be credited in the public advisory (used
     for CVE filing and the published GitHub Security Advisory)
4. Encrypt sensitive details with the maintainers' GPG key referenced at
   `keys/security.gpg`. The actual key material is **not** committed to
   this repository — the path is a placeholder that adopters wire up to
   their own key-distribution mechanism (see the **Encryption Key**
   section below for retrieval instructions).

## Response SLA

The maintainers commit to the following service levels for every report
received through the channels above:

- **Acknowledgement:** within **3 business days** of receipt.
- **Initial assessment:** within **7 business days**, including a triaged
  severity score using the **CVSS v3.1** scoring standard.
- **Fix or mitigation:** within **30 days** for **High** or **Critical**
  findings; within **90 days** for **Medium** or **Low** findings.
- **Disclosure:** coordinated **90-day** disclosure by default; expedited
  if the issue is being actively exploited in the wild, embargoed only
  while the fix is being developed and tested.

If a report does not receive an acknowledgement within the 3-business-day
window, please re-send it; messages can be lost to spam filters. Repeated
non-response is itself reportable through the project's governance
escalation path documented in `CONTRIBUTING.md`.

## Scope

### In scope

The following artifacts are in scope for vulnerability reports filed against
this repository:

- All files in this repository, including:
  - Kubernetes manifests under `kubernetes/`
  - Helm charts under `helm/`
  - Terraform and OpenTofu modules under `infrastructure/terraform/`
  - Crossplane providers, XRDs, and compositions under
    `infrastructure/crossplane/`
  - Policy bundles under `policy/` (Kyverno, Gatekeeper, OPA, Conftest,
    Checkov, tfsec configurations)
  - Observability configurations under `observability/` (Prometheus,
    Grafana, Loki, Tempo, OpenTelemetry Collector)
  - Progressive-delivery manifests under `progressive-delivery/`
  - Supply-chain configuration under `security/supply-chain/`
  - External Secrets and Vault wiring under `security/secrets/`
  - Bootstrap and helper scripts under `scripts/`
  - The Backstage software catalog and templates under `backstage/`
- The reusable GitHub Actions workflows under
  `.github/workflows/reusable-*.yml` (these are consumed by downstream
  pipelines and therefore have a wide blast radius).
- The GitLab CI templates under `.gitlab-ci/`.
- The example workload under `examples/sample-service/`.
- The documentation site under `docs/`, where misleading guidance could
  cause adopters to ship insecure configurations.

### Out of scope

The following are explicitly out of scope. Please direct reports for these
items to the appropriate upstream project or adopter:

- Third-party upstream projects — report directly to the project. Examples
  include Argo CD, Flux, Kyverno, Gatekeeper, Cosign, Syft, Grype, Trivy,
  External Secrets Operator, Vault, Prometheus, Grafana, Loki, Tempo, the
  OpenTelemetry Collector, Argo Rollouts, Flagger, Backstage, OpenCost,
  Crossplane, Terraform, and OpenTofu.
- Adopters' downstream deployments built **on top of** this reference
  architecture — those are the adopter's responsibility, including their
  own cluster configuration, IAM bindings, secret values, and image
  registry contents.
- Issues already documented in
  [`docs/runbooks/image-vuln-response.md`](docs/runbooks/image-vuln-response.md)
  as accepted risks. The runbook is the operational counterpart of this
  policy and lists known-and-accepted findings together with their
  compensating controls.
- Findings that require physical access to the developer workstation, an
  already-compromised CI runner, or other prerequisites outside the
  platform's threat model.
- Social-engineering attacks against individual maintainers.

## Supply Chain Security Posture

The platform defends its own supply chain end-to-end. Every guarantee below
is implemented in the reusable workflows under `.github/workflows/` so
adopters inherit the posture by referencing the workflows from their own
repositories.

- Container images produced by `.github/workflows/ci.yml` and
  `.github/workflows/reusable-build.yml` are signed with **Cosign** using
  **keyless OIDC** (Sigstore Fulcio for certificates, Rekor for the
  transparency log). No long-lived signing keys are stored anywhere.
- Each release ships **SLSA Level 3** provenance attestations generated by
  the `slsa-framework/slsa-github-generator` reusable workflow. The
  provenance binds every artifact to its source commit, builder identity,
  and build parameters.
- **SBOMs** in **CycloneDX** format are generated by **Syft** during the
  build and attached as Cosign attestations to every published image. A
  parallel SPDX SBOM is produced for adopters whose downstream tooling
  prefers that format.
- **Vulnerability scans** (**Trivy** for broad coverage including IaC and
  secrets, **Grype** for SBOM-driven analysis) run on every pull request
  and every release. Findings are uploaded to **GitHub Code Scanning** as
  SARIF so they appear in the repository's Security view.
- Kyverno policies in `policy/kyverno/require-signed-images.yaml` enforce
  signature verification at admission time, so unsigned or unverifiable
  images cannot be deployed to a cluster running the platform.
- Pre-commit **gitleaks** runs on every commit (via
  `.pre-commit-config.yaml`) to catch credentials, tokens, and other
  secrets before they ever reach a remote branch.
- Dependencies are kept current by **Dependabot**, configured in
  `.github/dependabot.yml` to file pull requests for GitHub Actions,
  Terraform providers, container base images, and language ecosystems
  used by the documentation site and example workload.

## Verifying Releases

Adopters should verify the signature and provenance of every image before
deploying it. Verification relies on the GitHub Actions OIDC issuer and the
platform's certificate identity pattern; no keys need to be pre-shared.

Verify the signature of a release image:

```bash
# Verify the signature of a release image
cosign verify <registry>/19May_2/sample-service:v1.2.3 \
  --certificate-identity-regexp "https://github.com/19May_2/.*" \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com"
```

Retrieve the CycloneDX SBOM attestation attached to a specific image digest:

```bash
cosign download attestation <image>@sha256:<digest> --predicate-type cyclonedx
```

Verify the SLSA provenance attestation:

```bash
cosign verify-attestation <registry>/19May_2/sample-service:v1.2.3 \
  --type slsaprovenance \
  --certificate-identity-regexp "https://github.com/19May_2/.*" \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com"
```

Both signing and verification require `cosign` v2.x or later. The platform
is pinned to the Cosign release line documented in `scripts/install-tooling.sh`
and the matching ADR under `docs/adr/0007-supply-chain-slsa.md`. The
canonical verification snippet is also embedded in the per-environment
deployment pipelines under `.github/workflows/reusable-deploy.yml`.

## Hall of Fame

Researchers who follow this policy will be acknowledged here unless they
prefer to remain anonymous. Submissions that lead to a CVE assignment will
be linked to their corresponding **GitHub Security Advisory** so the
public record reflects the responsible disclosure.

_No entries yet — be the first._

## Encryption Key

For especially sensitive disclosures (for example, reports that include a
working exploit, customer data, or credentials), encrypt the report body
with the maintainers' GPG public key. The placeholder fingerprint below is
what adopters of this reference architecture replace with their real key
before publishing this policy.

Placeholder fingerprint (40 hex characters, space-separated in the standard
GPG block format):

```text
0000 0000 0000 0000 0000  0000 0000 0000 0000 0000
```

Fetch the public key from a public keyserver such as `keys.openpgp.org`:

```bash
# Pull the key by its full fingerprint
gpg --keyserver hkps://keys.openpgp.org \
    --recv-keys 0000000000000000000000000000000000000000

# Export the armored public key for review
gpg --export --armor 0000000000000000000000000000000000000000
```

Once retrieved, verify the fingerprint **out of band** (for example via
the maintainers' personal websites or a signed commit on this repository)
before relying on the key for encryption.

> **Adopters MUST replace** both the placeholder fingerprint and the
> keyserver URL with their own key material before publishing this
> policy. The placeholder values are intentionally non-functional so a
> misconfiguration never silently encrypts to a key the adopter does not
> control.

---

_This policy is reviewed in conjunction with the operational runbook at_
[`docs/runbooks/image-vuln-response.md`](docs/runbooks/image-vuln-response.md)
_and the supply-chain Architecture Decision Record at_
[`docs/adr/0007-supply-chain-slsa.md`](docs/adr/0007-supply-chain-slsa.md).
_Changes to this policy follow the standard contribution workflow defined
in_ [`CONTRIBUTING.md`](CONTRIBUTING.md)
_and require review by at least one CODEOWNER._
