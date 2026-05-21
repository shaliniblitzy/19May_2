# Contributing to 19May_2

Welcome — and thank you for considering a contribution to the 19May_2
DevOps Platform Reference Architecture. This repository is an
opinionated, cloud-native reference covering the full software-delivery
lifecycle: plan → code → build → test → secure → release → operate →
observe → cost-account. Downstream adopters consume it as a starting
point for their own platforms, so quality, clarity, and version
discipline matter as much as the architectural intent. Please read
this guide end-to-end and the
[`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md) before participating.

Because this repository is a **reference architecture**, contributions
that improve generality, documentation, version pinning, security
posture, or test coverage are the most welcome. Bespoke deployment
overlays, customer-specific tweaks, and proprietary extensions belong
in a fork — the upstream stays intentionally vendor-neutral and
adopter-agnostic.

## Code of Conduct

This project and everyone participating in it is governed by the
[Contributor Covenant 2.1](CODE_OF_CONDUCT.md). By participating you
agree to uphold the code. Please report unacceptable behavior to the
maintainers using the contact channels listed in
[`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md). Harassment, discrimination,
or any conduct that makes the community unsafe will be met with the
enforcement responses documented there.

## Ground Rules

- **Discuss substantial changes in an issue before opening a PR.**
  "Substantial" means anything that touches the architecture, adds a
  component, changes a pinned Helm chart or Terraform provider version,
  alters a reusable workflow's contract, or modifies the supply-chain
  policy.
- **Keep PRs focused — one concern per PR.** A new Kyverno policy, a
  Loki dashboard refresh, and a Backstage template tweak are three
  PRs, not one. Smaller PRs review faster and bisect cleaner.
- **Update the documentation alongside the change.** Code or manifest
  changes that are not reflected in `docs/` will be sent back for
  revision; out-of-date docs are worse than missing docs.
- **Record architectural decisions in an ADR** (see the
  [ADR Process](#adr-process) section) — any change that selects a
  tool, drops a tool, alters the security posture, or shifts a default
  needs a new ADR under `docs/adr/`.
- **Add or update runbooks when operational behavior changes.** If a
  PR changes how something is rolled back, scaled, rotated, or
  recovered, update the corresponding page under `docs/runbooks/` in
  the same PR.
- **Be kind, be precise, be patient.** Reference architectures are
  consumed by people who weren't part of the original conversation;
  write for them.

## Development Environment

You do not need any commercial tooling to contribute. Everything below
is open-source and runs locally on Linux, macOS, or Windows (WSL2).

1. **Install the pinned tooling** by running
   [`scripts/install-tooling.sh`](scripts/install-tooling.sh). It
   installs the CLI versions this repository expects: `kubectl`,
   `helm`, `kustomize`, `terraform` (or `tofu`), `cosign`, `syft`,
   `grype`, `trivy`, `tfsec`, `checkov`, `opa`, `conftest`, `argocd`,
   `flux`, `kyverno`, and `dagger`.
2. **Install `pre-commit`** so hooks fire on every commit:

   ```bash
   pip install --user pre-commit
   pre-commit install
   pre-commit install --hook-type commit-msg
   ```

3. **Run the hooks before every commit** — the single most useful
   habit a contributor can build:

   ```bash
   pre-commit run --all-files
   ```

   CI will reject any commit that fails a hook, so failing early on
   your own machine is strictly faster than failing in CI.
4. **Use the project `make` targets where they exist** — `make lint`,
   `make scan`, `make docs-serve`, `make policy-test` — to wrap the
   long CLI invocations. The `Makefile` is forward-looking guidance;
   not every target may be implemented in early milestones — file an
   issue if one is missing, or add the target in your PR.

The pre-commit hook set is defined in
[`.pre-commit-config.yaml`](.pre-commit-config.yaml) and includes
`yamllint`, `markdownlint-cli2`, `shellcheck`, `tflint`, `checkov`,
`gitleaks`, `conventional-pre-commit`, and the standard whitespace/EOL
hooks.

## Branching Model

This project follows **trunk-based development**.

- The trunk branch is `main`. It must always be green: every commit
  on `main` is releasable.
- Feature work happens on short-lived branches named
  `<type>/<short-slug>` where `<type>` matches a Conventional Commit
  type. Examples: `feat/add-kyverno-policy`,
  `fix/argo-rollouts-canary-step`, `docs/runbook-secret-rotation`,
  `chore/bump-prometheus-stack`.
- Open pull requests **early as drafts**; mark **ready for review**
  only when CI is green and `pre-commit run --all-files` passes
  locally.
- **Never push directly to `main`.** Branch protection enforces this,
  and so does CODEOWNERS — every PR needs the explicit approval of
  the owners listed in [`CODEOWNERS`](CODEOWNERS) for the paths it
  touches.
- Rebase, do not merge, when keeping your branch current with `main`.
  Maintainers squash-merge on the way in, so a clean message on the
  squash matters more than per-commit history on the branch.

## Commit Messages — Conventional Commits

This repository adopts
[**Conventional Commits 1.0.0**](https://www.conventionalcommits.org/en/v1.0.0/).
The format is:

```text
<type>(<scope>): <short, imperative summary>

<optional body explaining the why, not the how>

<optional footer: BREAKING CHANGE, Refs #123, Signed-off-by: ...>
```

The **allowed commit types** are:

- `feat` — adding a user-facing capability or platform component
- `fix` — correcting incorrect behavior in an existing capability
- `chore` — maintenance with no functional change (dep bumps, etc.)
- `docs` — documentation-only changes under `docs/` or top-level `.md`
- `refactor` — restructuring without changing external behavior
- `test` — adding or fixing tests, fixtures, or policy-test cases
- `ci` — CI / pipeline changes under `.github/` or `.gitlab-ci/`
- `build` — build-system, container-image, or release-tooling changes
- `perf` — performance improvements without changing semantics
- `revert` — reverts a prior commit (cite the reverted SHA in body)

Realistic examples drawn from this platform's domains:

- `feat(observability): add Loki dashboard for ingress traffic`
- `chore(deps): bump argo-cd Helm chart 7.4.0 → 7.5.1`
- `docs(adr): add ADR 0009 selecting Istio for service mesh`
- `fix(policy): correct image-signature verification policy regex`
- `ci(workflows): pin actions/checkout to commit SHA`

The `conventional-pre-commit` hook in
[`.pre-commit-config.yaml`](.pre-commit-config.yaml) enforces this
format locally; the same check runs in CI on every PR. Conventional
Commit history is what allows
[`.github/workflows/release.yml`](.github/workflows/release.yml) to
generate the changelog automatically and to derive the correct
semantic-version bump from the merged commits — so this convention is
load-bearing, not a nicety.

## Signing Commits (DCO / GPG)

Every commit to this repository must be signed off under the
[**Developer Certificate of Origin (DCO)**](https://developercertificate.org/),
a lightweight, repository-friendly alternative to a Contributor
License Agreement (CLA): you attest, in each commit, that you have
the right to contribute the change under the project's license.

Sign off automatically by adding `--signoff` (or the shorter `-s`) to
every `git commit` invocation:

```bash
git commit -s -m "feat(observability): add Loki dashboard for ingress"
```

`git` will append the required trailer to your commit message:

```text
Signed-off-by: Your Name <you@example.com>
```

The trailer **must** match the name and email configured in your
local Git identity (`git config user.name` and `git config
user.email`). The DCO check in CI validates this on every PR; commits
without a valid `Signed-off-by` trailer will block the merge.

If you would also like your commits to show as **Verified** in the
GitHub UI, additionally sign with GPG or SSH (`git commit -S -s ...`)
and configure signing keys per the GitHub documentation on
[managing commit signature verification][gh-signed-commits].

**No CLA is required.** The contribution contract for this repository
is the combination of the Apache-2.0 license (see [`LICENSE`](LICENSE))
and the DCO sign-off — nothing more.

[gh-signed-commits]: https://docs.github.com/en/authentication/managing-commit-signature-verification

## Pull Request Workflow

1. **Fork the repository** (external contributors) or create a feature
   branch within the org (maintainers).
2. **Create a branch from `main`** following the `<type>/<short-slug>`
   convention from the [Branching Model](#branching-model) section.
3. **Make your changes** as small, logical, atomic commits. Each
   commit message must follow
   [Conventional Commits](#commit-messages--conventional-commits) and
   carry a DCO sign-off.
4. **Run the hooks** — `pre-commit run --all-files` must pass before
   you push. Treat hook failures as bugs in the change, not noise.
5. **Push and open a pull request** against `main`. Open as a draft
   while iterating; mark **ready for review** once CI is green.
6. **Fill out the PR template completely.** Link the issue this PR
   resolves, summarize the risk, describe the rollout plan (especially
   for changes touching GitOps, policy, or the supply chain), and
   attach the relevant runbook if operational behavior changes.
7. **Respond to review feedback.** Push fixups as additional commits
   so reviewers can see what changed; CODEOWNERS approval is required.
8. **Wait for CI to be green.** Every PR must pass the gates defined
   in [`.github/workflows/ci.yml`](.github/workflows/ci.yml),
   [`iac.yml`](.github/workflows/iac.yml), and
   [`policy.yml`](.github/workflows/policy.yml) where applicable.
9. **Maintainer squash-merge.** Maintainers squash-merge with a
   Conventional Commit subject line that summarizes the PR; the
   per-commit history is preserved in the PR itself.

A reviewer from [`CODEOWNERS`](CODEOWNERS) for every touched path must
approve before merge. The CODEOWNERS file is the authoritative routing
table — if no owner is listed for a path, ask in the issue or PR who
should review.

## Documentation Contributions

Documentation lives in [`docs/`](docs/) and is rendered as a static
site via [MkDocs](https://www.mkdocs.org/) with the
[Material theme](https://squidfunk.github.io/mkdocs-material/); the
site config is [`mkdocs.yml`](mkdocs.yml). Preview your changes
locally:

```bash
pip install -r docs/requirements.txt
mkdocs serve
```

The site is then available at `http://127.0.0.1:8000/` with hot
reload. Style guidelines:

- Use ATX-style headings (`#`, `##`) — never setext underlines.
- Use the Material **admonition** syntax (`!!! note`, `!!! warning`,
  `!!! danger`, `!!! tip`) rather than ad-hoc emphasis when calling
  out important information.
- Wrap prose at 80 characters (the markdownlint MD013 default).
- Always specify a language on fenced code blocks (`bash`, `yaml`,
  `hcl`, `dockerfile`, `text`) so syntax highlighting works.
- Cross-link liberally — adopters navigate from component pages into
  ADRs, runbooks, and the glossary.

Adding a **new platform component** requires three documentation
artifacts in the same PR:

1. A component page under `docs/components/<name>.md` describing what
   it does, how it integrates, and how to scale it.
2. An [ADR](#adr-process) under `docs/adr/00NN-choose-<name>.md`
   recording the decision to adopt it (or retire something it
   replaces).
3. A glossary entry in [`docs/glossary.md`](docs/glossary.md) if the
   component introduces new terminology.

## ADR Process

Architecture Decision Records (ADRs) capture **why** the platform is
the way it is. They are short, immutable historical records — once
accepted, an ADR is not edited; subsequent decisions supersede it via
a new ADR. This project uses the
[Michael Nygard format][nygard-adr] with these sections:

- **Title** — `ADR-00NN: <short statement>`
- **Status** — `Proposed`, `Accepted`, `Deprecated`, or
  `Superseded by ADR-00MM`
- **Context** — what is the issue motivating the decision?
- **Decision** — what is the change that we are proposing or doing?
- **Consequences** — what becomes easier or harder as a result?

Rules of the road:

- File naming: `docs/adr/00NN-<kebab-title>.md`, where `NN` is the
  next monotonically increasing number. Never reuse a number, never
  rename an accepted ADR.
- Status lifecycle: **Proposed** → **Accepted** → **Superseded by
  ADR-00MM**. A deprecated decision retains its ADR; the superseding
  ADR links back to it.
- Keep ADRs short — one page is the target. Long context belongs in
  the supporting documentation, not in the ADR itself.
- Cite the ADR by number from any documentation or code comment that
  depends on the decision so future readers can trace the rationale.

The seed ADRs in [`docs/adr/`](docs/adr/) demonstrate the format and
voice; mirror them when authoring new ones.

[nygard-adr]: https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions

## Testing & CI Expectations

Every pull request runs the following gates via
[`.github/workflows/ci.yml`](.github/workflows/ci.yml) and its
companion workflows. A PR cannot merge until every applicable gate is
green.

- **Lint** — `yamllint`, `markdownlint-cli2`, `shellcheck`, `tflint`.
- **Infrastructure-as-Code scan** — `tfsec` and `checkov`, wired via
  [`.github/workflows/iac.yml`](.github/workflows/iac.yml).
- **Policy-as-Code test** — `kyverno test`, `gator test`, and
  `conftest verify`, wired via
  [`.github/workflows/policy.yml`](.github/workflows/policy.yml).
- **Container build + scan** — `docker buildx` for the build,
  followed by `trivy` and `grype` scans of the resulting image.
- **SBOM generation** — `syft` emits CycloneDX and SPDX SBOMs for
  every built artifact.
- **Commit hygiene** — DCO sign-off check and Conventional Commits
  check on every commit in the PR.

Signing with `cosign` happens only on
[`.github/workflows/cd.yml`](.github/workflows/cd.yml) runs from
`main` and from the release workflow — not on pull-request runs — to
avoid signing artifacts that have not yet been reviewed. If your PR
cannot pass a gate for an unrelated reason (for example, a flaky
upstream registry), open a separate issue describing the flakiness;
do **not** disable the gate or pin around it without maintainer
approval.

## Releases

This project uses [**Semantic Versioning**](https://semver.org/)
(`vMAJOR.MINOR.PATCH`).

- **MAJOR** — backward-incompatible changes to the reference
  architecture (renamed top-level directories, removed components,
  breaking changes to the reusable-workflow contracts).
- **MINOR** — new components, new modules, new reusable workflows,
  or any backward-compatible feature addition.
- **PATCH** — bug fixes, documentation improvements, dependency-pin
  refreshes that do not change behavior.

Releases are cut by maintainers from `main` via
[`.github/workflows/release.yml`](.github/workflows/release.yml),
which derives the next version from the Conventional Commits merged
since the previous tag, generates the changelog, creates a Git tag
and a GitHub Release, generates **SLSA provenance** attestations via
[`slsa-framework/slsa-github-generator`][slsa-gen], and pushes
released container images signed keylessly with `cosign` and attested
with `syft` SBOMs. Contributors do not tag releases directly; merge
through the normal PR workflow and a maintainer will include the work
in the next release.

[slsa-gen]: https://github.com/slsa-framework/slsa-github-generator

## Security

Security is a first-class concern for this reference architecture
because vulnerabilities here propagate to every downstream adopter.

- **Vulnerability disclosure.** Do not file public issues for
  security vulnerabilities. Follow the private disclosure process in
  [`SECURITY.md`](SECURITY.md).
- **Never commit secrets.** The `gitleaks` pre-commit hook blocks
  obvious leaks locally; CI re-runs the scan on every push as
  defense-in-depth. If a secret is committed accidentally, **rotate
  it immediately** and then open a private issue per
  [`SECURITY.md`](SECURITY.md) — do not try to scrub history alone.
- **No production secret values in Git.** Only `ExternalSecret` and
  `SecretStore` references belong in the repository. Actual secret
  material lives in Vault or the cloud's native secret store and is
  projected into the cluster by the External Secrets Operator.
- **Pin third-party GitHub Actions by commit SHA**, not by tag.
  Floating tags can be re-pointed by upstream and silently change
  behavior; SHAs cannot.
- **Image signatures, SBOMs, and SLSA provenance are mandatory** for
  any artifact published to the registry. The reusable workflows
  enforce this — do not bypass them.

## Questions

If you are unsure where to ask:

- **Design questions** — open a GitHub Discussion (preferred) or an
  issue with the `question` label. Larger design questions often
  graduate into ADRs.
- **Terminology** — check [`docs/glossary.md`](docs/glossary.md)
  first; if a term is missing, open a `docs` PR adding it.
- **Onboarding** — read
  [`docs/getting-started/onboarding.md`](docs/getting-started/onboarding.md)
  and walk the
  [`quickstart`](docs/getting-started/quickstart.md).
- **Operational issues with a specific component** — start with the
  matching page under [`docs/components/`](docs/components/) and the
  runbooks under [`docs/runbooks/`](docs/runbooks/).

Thank you again for contributing — your work makes this reference
architecture better for everyone who builds on top of it.
