# Reproducible repository onboarding standard

This document is the authoritative source for a future request to apply the
same DevSecOps validation to another repository.

The recommended target is recommended-enterprise-v1. The manual profile is the
onboarding and calibration stage before mandatory controls.

## Standard profile manual nonblocking v1

1. Perform a read-only audit of repository rules, README, roadmap, languages,
   lockfiles, containers, existing CI, default branch and permissions.
2. Do not modify existing build or deployment workflows.
3. Add .github/workflows/manual-devsecops-audit.yml.
4. Permit only the workflow_dispatch trigger.
5. Grant contents read by default.
6. Run Semgrep CE, Trivy filesystem, Gitleaks and MegaLinter.
7. Generate a CycloneDX SBOM with Trivy.
8. Upload artifacts even when a scanner identifies findings.
9. Do not configure Required Status Checks or change Branch Protection.
10. Add operational documentation covering purpose, use, interpretation,
    optional secrets and rollback.
11. validate YAML and confirm the absence of automatic triggers.
12. Publish through a separate branch and pull request unless direct commit was
    explicitly requested.

## Result contract

| Artifact | Content |
|---|---|
| semgrep-report | SAST findings in SARIF |
| trivy-report | CVEs, secrets and misconfiguration in SARIF |
| gitleaks-report | Git secret findings in SARIF |
| megalinter-reports | multi-language linter reports |
| sbom-cyclonedx | CycloneDX JSON component inventory |

## Permitted adaptations

Native build and test jobs may be added for the application stack, but the base
controls remain. Generated directories such as vendor, node_modules, build and
dist may be excluded with justification. Automatic or blocking execution is a
new profile and requires explicit approval.

After calibration, the target includes SonarQube Community, Semgrep CE,
MegaLinter, Trivy, Gitleaks and Reviewdog. Enable ORT for compliance and
release. PR-Agent with Ollama may provide advisory self-hosted AI review.

## Conventions

- Workflow name: Manual DevSecOps Audit.
- File name: .github/workflows/manual-devsecops-audit.yml.
- Job timeout: no more than 30 minutes in the standard profile.
- Artifact retention: 14 days.
- Unidentified or unauthorized repositories are excluded.
- Do not include secrets or internal addresses in files or artifacts.

## Controlled evolution

Increment the profile version only when the trigger, minimum scanner set,
permissions or artifact contract changes. Keep only fictional examples in the
public repository. Maintain the registry of real implementations in a separate
authorized private location.

