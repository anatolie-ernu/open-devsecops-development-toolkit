# Consolidated original recommendation

Source-code testing, code compliance and automated review require a stack rather
than one product:

- **SonarQube Community Build** covers bugs, code smells, duplication,
  complexity, coverage and a Quality Gate. Verify branch and pull-request
  capabilities for the installed edition.
- **Semgrep CE** provides SAST and custom organizational rules.
- **MegaLinter** orchestrates linters for C sharp, TypeScript, JavaScript,
  Python, PHP, Dart, YAML, Markdown, Docker and shell.
- **Trivy** scans CVEs, images, filesystems, secrets and IaC configuration.
- **Gitleaks** specializes in secret detection and Git history.
- **OSS Review Toolkit** produces SBOM, license and copyright evidence and
  evaluates policy as code.
- **Reviewdog** places deterministic findings on changed lines.
- **PR-Agent with Ollama** provides optional self-hosted advisory AI review.

A mature Quality Gate combines build, tests, coverage, SAST, secrets,
dependencies, licenses and human approval. Pull requests run fast controls.
After merge or before deployment, run the complete analysis, final-image scan,
SBOM generation, authorized DAST and staging validation.

The pilot keeps checks manual and non-blocking until thresholds are calibrated
and formally approved.

