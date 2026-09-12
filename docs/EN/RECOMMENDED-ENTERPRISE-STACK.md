# Recommended replication version recommended enterprise v1

## Decision

**SonarQube Community, Semgrep CE, MegaLinter, Trivy, Gitleaks and Reviewdog**
form the standard base. Add **OSS Review Toolkit ORT** for compliance, licenses
and SBOM, and **PR-Agent with Ollama** for fully self-hosted AI review.
Distribute the stack through a reusable workflow and make deterministic
controls mandatory after calibration.

## Components

| Component | Status | Responsibility |
|---|---|---|
| SonarQube Community | mandatory | quality dashboard, maintainability, bugs, coverage |
| Semgrep CE | mandatory | SAST and custom organizational rules |
| MegaLinter | mandatory | multi-language style and compliance |
| Trivy | mandatory | CVEs, images, repository, secrets and IaC |
| Gitleaks | mandatory | secrets in files and Git history |
| Reviewdog | mandatory in PR phase | inline deterministic findings |
| ORT | mandatory for release and compliance | licenses, copyright, policy as code and SBOM |
| PR-Agent with Ollama | recommended | self-hosted advisory AI review |

## Replication model

1. The central repository maintains reusable workflows, versions and policies.
2. Every consuming repository pins an approved tag rather than main.
3. Start with manual-nonblocking-v1 for two to four weeks.
4. After triage and baseline creation, enable informative pull-request runs.
5. Promote critical deterministic controls to Required Status Checks.
6. Run ORT at release and periodically under a legally approved license policy.
7. Run PR-Agent with self-hosted Ollama; never let it decide merge by itself.

## Blocking activation order

1. confirmed secrets;
2. build and tests;
3. remediable CRITICAL vulnerabilities;
4. confirmed Semgrep ERROR rules;
5. SonarQube Quality Gate on new code where available;
6. linters on new or modified code;
7. ORT deny and unknown license policy;
8. HIGH severity and coverage thresholds after project approval.

Reviewdog publishes findings. PR-Agent with Ollama remains advisory.

## New repositories

Every new repository starts with at least manual-nonblocking-v1. The target is
recommended-enterprise-v1. Excluding a mandatory control requires a rationale,
owner, compensating control, approver and expiry date.

