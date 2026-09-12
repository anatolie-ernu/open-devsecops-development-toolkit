# Complete DevSecOps CI implementation guide

## 1 Objective and scope

The platform introduces repeatable source-code controls without requiring code
to be sent to SaaS services. It covers code quality, SAST, secrets,
dependencies, containers, IaC, SBOM and licenses. Initial operation is manual
and does not change branch protection.

## 2 Architecture

~~~mermaid
flowchart TD
  A[Manual workflow] --> B[Checkout]
  B --> C[Semgrep]
  B --> D[Trivy]
  B --> E[Gitleaks]
  B --> F[MegaLinter]
  B --> G[Optional build and tests]
  C --> H[SARIF artifacts and reports]
  D --> H
  E --> H
  F --> H
  G --> H
  H --> I[Review and calibration]
~~~

SonarQube is a separate central service. Enable its scanner only after
SONAR_HOST_URL and SONAR_TOKEN are configured securely.

## 3 Pilot execution mode

- only workflow_dispatch triggers execution;
- permissions default to contents read;
- no pull_request, push, schedule or deployment trigger;
- branch protection remains unchanged;
- scanners continue far enough to publish their reports;
- the summary job identifies available evidence;
- artifacts exclude environment files, keys and complete source archives.

A manual workflow may be red when it finds problems. It does not block
development because branch protection does not require it.

## 4 Installing SonarQube

Minimum pilot requirements are Linux x86_64, Docker Engine with Compose v2,
4 vCPU, 8 GiB RAM, 20 GiB storage, organizational DNS and TLS, and separate
PostgreSQL and SonarQube-volume backups.

~~~bash
cd platform/sonarqube
cp .env.example .env
# replace every CHANGE_ME value
docker compose config
docker compose up -d
docker compose ps
~~~

Do not expose PostgreSQL. Publish SonarQube through an HTTPS reverse proxy.
Change the initial administrator password and create a dedicated project token.

For backup, save PostgreSQL consistently with pg_dump and protect the
sonarqube_data, sonarqube_extensions and sonarqube_logs volumes. Test restore
quarterly in isolation.

## 5 GitHub secrets

| Secret or variable | Requirement | Use |
|---|---:|---|
| SONAR_HOST_URL | Sonar only | HTTPS URL reachable by runner |
| SONAR_TOKEN | Sonar only | non-administrative project token |

The base workflow requires no PAT. Keep GITHUB_TOKEN read-only. Use an approved
self-hosted runner when an internal SonarQube address is not Internet reachable.

## 6 Pilot controls and proposed thresholds

| Control | Pilot | Proposed calibrated threshold |
|---|---|---|
| Semgrep | reporting | confirmed ERROR |
| Trivy CVE | report HIGH and CRITICAL | CRITICAL; approved exceptions for HIGH |
| Trivy misconfiguration | reporting | HIGH and CRITICAL |
| Gitleaks | immediate reporting | any valid secret |
| MegaLinter | reporting | errors on new or modified files |
| Coverage | collect where available | 80 percent on new code, per-project approval |
| Licenses | inventory and SBOM | legally approved deny list |
| SonarQube | main or manual baseline | new-code Quality Gate where supported |

Do not enable all blocking thresholds at once. Run manually for at least two to
four weeks, classify findings and create owned, expiring exceptions.

## 7 Operational workflow

1. Select a branch and start the workflow.
2. Choose standard or extended scan depth.
3. Review the summary and download artifacts.
4. Classify findings as valid, false positive, accepted or remediated.
5. Record accepted risk rationale, owner and expiry.
6. Rerun after remediation.
7. Retain relevant release evidence.

## 8 SBOM and licenses

Trivy produces CycloneDX inventory. For complete legal control, run ORT in a
separate Analyzer, Scanner, Advisor, Evaluator and Reporter flow. Legal counsel
must approve the policy. Recommended categories are allow, review, deny and
unknown.

## 9 Reviewdog and PR Agent

Reviewdog publishes linter output on changed lines and requires pull-request
write permission. PR-Agent may use local Ollama, but remains advisory and must
never be the only blocking criterion. Model prompt injection and minimize
permissions before activation.

## 10 Rollback

Disable or remove the separate pilot workflow. Build, deployment and branch
protection remain unchanged. Stop SonarQube without the -v flag. Removing
volumes is destructive and is not part of normal rollback.

## 11 Pilot acceptance criteria

- the workflow is visible only for manual execution;
- no existing workflow is modified;
- scanners publish evidence even when findings exist;
- default permissions are read-only;
- SonarQube remains disabled until secrets exist;
- no real address, password or key is versioned;
- documentation covers operation, exceptions and rollback;
- YAML and shell syntax validation passes.

