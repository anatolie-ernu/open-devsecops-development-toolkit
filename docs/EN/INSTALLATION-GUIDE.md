# Step by step installation guide

This guide installs the Open DevSecOps Development Toolkit in a new GitHub
project. Start in manual non-blocking mode. Enable mandatory validation only
after findings have been calibrated.

## Step 1 Prepare the workstation or runner

Install Git, Docker Engine, Docker Compose v2 and the basic shell utilities.

~~~bash
git --version
docker --version
docker compose version
bash --version
~~~

The runner account must be able to start Docker containers. Do not install the
production runner on the same host as the public application service.

## Step 2 Clone and validate the toolkit

~~~bash
git clone https://github.com/anatolie-ernu/open-devsecops-development-toolkit.git
cd open-devsecops-development-toolkit
chmod +x scripts/*.sh
scripts/validate-configs.sh
~~~

The validator must complete successfully before files are copied into an
application repository.

## Step 3 Prepare the application repository

Replace the example path with the real repository location:

~~~bash
export APPLICATION_DIR=/opt/example-application
mkdir -p "$APPLICATION_DIR/devsecops"
cp -a examples/sample-application/devsecops/. "$APPLICATION_DIR/devsecops/"
mkdir -p "$APPLICATION_DIR/.github/workflows"
cp templates/github/manual-devsecops-generic.yml   "$APPLICATION_DIR/.github/workflows/manual-devsecops-audit.yml"
chmod +x "$APPLICATION_DIR"/devsecops/scripts/*.sh
~~~

Do not copy generated reports, environment files or tokens.

## Step 4 Adapt project parameters

~~~bash
cd "$APPLICATION_DIR"
cp devsecops/config/project.env.example devsecops/config/project.env
~~~

Change only non-sensitive values such as languages, source directories,
exclusions and the project key. Do not commit project.env if it contains
sensitive data.

Review these files:

- devsecops/config/.semgrep.yml;
- devsecops/config/.mega-linter.yml;
- devsecops/config/gitleaks.toml;
- devsecops/sonar-project.properties.

Exclude vendor, node_modules, build, dist, caches and generated reports.

## Step 5 Install SonarQube Community

On the approved server:

~~~bash
cd open-devsecops-development-toolkit/platform/sonarqube
cp .env.example .env
chmod 600 .env
~~~

Replace every CHANGE_ME value. Generate unique PostgreSQL and technical-account
passwords.

Configure the Linux search-engine parameter:

~~~bash
sudo sysctl -w vm.max_map_count=524288
echo 'vm.max_map_count=524288' | sudo tee /etc/sysctl.d/99-sonarqube.conf
sudo sysctl --system
~~~

Validate and start the stack:

~~~bash
docker compose config
docker compose pull
docker compose up -d
docker compose ps
docker compose logs --tail=100 sonarqube
~~~

Wait for healthy status. Do not expose PostgreSQL. Publish SonarQube behind an
HTTPS reverse proxy and restrict administrative access.

## Step 6 Configure SonarQube

1. Open the HTTPS URL.
2. Change the initial password immediately.
3. Create a separate application project.
4. Generate a project-scoped technical token.
5. Record the project key without copying the token into documentation.
6. Configure the Quality Gate for new code.
7. Choose a New Code period that matches the release cycle.

Store the token only in GitHub Actions Secrets or a secret manager.

## Step 7 Configure GitHub secrets

Open repository Settings, Secrets and variables, Actions, New repository secret.

Create:

| Name | Value |
|---|---|
| SONAR_HOST_URL | HTTPS URL reachable by the runner |
| SONAR_TOKEN | project-scoped technical token |

Do not use an administrator token. For internal SonarQube, use a self-hosted
runner that resolves DNS and validates the TLS certificate.

## Step 8 Run local validation

~~~bash
cd "$APPLICATION_DIR"
devsecops/scripts/validate-configs.sh
devsecops/scripts/run-local-audit.sh
find devsecops/reports -maxdepth 2 -type f -print
~~~

The scan may report problems. During the pilot, the goal is to collect evidence
rather than block development.

## Step 9 Run the first manual GitHub audit

1. Commit files on a dedicated branch.
2. Open a pull request and review the changes.
3. Merge the manual workflow.
4. Open Actions.
5. Select Manual DevSecOps Audit.
6. Select Run workflow.
7. Choose the branch and standard scan depth.
8. Wait for every job.
9. Download Semgrep, Trivy, Gitleaks, MegaLinter and SBOM artifacts.
10. Record confirmed findings.

The manual workflow must not contain push, pull_request or schedule triggers.

## Step 10 Calibrate findings

Run the pilot for two to four weeks. Record severity, validity, owner, due date
and approved exception for every finding. Keep suppressions narrow and expiring.

Recommended blocking order:

1. confirmed secrets;
2. build and tests;
3. remediable CRITICAL vulnerabilities;
4. confirmed Semgrep ERROR rules;
5. SonarQube Quality Gate;
6. ORT denied licenses;
7. HIGH severity and coverage after approval.

## Step 11 Enable the mandatory gate

~~~bash
cd "$APPLICATION_DIR"
cp devsecops/workflows/recommended-blocking.yml.example   .github/workflows/devsecops-gate.yml
~~~

Or copy the toolkit template:

~~~bash
cp /path/to/open-devsecops-development-toolkit/templates/github/mandatory-devsecops-gate.yml.example   .github/workflows/devsecops-gate.yml
~~~

Review the workflow and:

- keep the devsecops-gate job name;
- remove continue-on-error;
- use exit-code 1 for blocking thresholds;
- pin actions and images to approved versions;
- minimize permissions;
- add a timeout;
- retain reports as artifacts even on failure.

## Step 12 Configure branch protection or a ruleset

Open GitHub Settings, Branches or Rules, Rulesets.

1. Select the default branch.
2. Enable Require a pull request before merging.
3. Enable Require status checks to pass.
4. Select exactly devsecops-gate after its first run.
5. Enable Require branches to be up to date.
6. Disable bypass for ordinary contributors.
7. Disallow force pushes and branch deletion.
8. Save the rule.

Require the stable job name, not a step name.

## Step 13 Make build and deployment depend on the gate

~~~yaml
build:
  needs: devsecops-gate
  if: needs.devsecops-gate.result == 'success'
  runs-on: ubuntu-latest
  steps:
    - run: ./build.sh

deploy-production:
  needs: [devsecops-gate, build]
  if: needs.devsecops-gate.result == 'success'
  environment: production
  runs-on: ubuntu-latest
  steps:
    - run: ./deploy.sh VALIDATED_ARTIFACT_DIGEST
~~~

Scan the final image after build. Deployment must use the exact digest of the
validated artifact.

## Step 14 Protect the Production environment

In Settings, Environments, create production and configure:

- required reviewers;
- prevent self-review where available;
- permitted branches or tags;
- environment-specific secrets;
- timeout and rollback procedure.

Production approval does not replace devsecops-gate. Both are required.

## Step 15 Add ORT and SBOM

Run ORT in a separate Analyzer, Scanner, Advisor, Evaluator and Reporter job.
Publish CycloneDX or SPDX and the license report. Legal owners must approve the
allow, review, deny and unknown policy before it blocks a release.

## Step 16 Add PR Agent and Ollama

Install Ollama on an internal host and permit access only from approved runners.
Start PR-Agent in advisory mode. Do not send restricted code, secrets or
internal data to an external model. Human review remains mandatory.

## Step 17 Perform the mandatory negative test

1. Create a test branch.
2. Add a harmless fixture detected by a calibrated rule.
3. Open a pull request.
4. Confirm that devsecops-gate fails.
5. Confirm that merge is disabled.
6. Confirm that build or deployment does not start.
7. Remove the fixture.
8. Confirm that the gate becomes green.
9. Confirm that deployment proceeds only after success.

Do not accept the blocking implementation without this test.

## Step 18 Backup and maintenance

Create a consistent PostgreSQL and SonarQube-volume backup. Test restoration
quarterly. Update actions, images and rules monthly through pull requests.
Review expired exceptions and monitor job duration.

## Step 19 Rollback

For the pilot, disable the manual workflow. For a mandatory gate, approve the
rollback window formally before removing the Required Status Check. Never run
docker compose down -v unless data deletion is approved and a verified backup
exists.

## Final verification

- local validation passes;
- the manual audit publishes all artifacts;
- SonarQube is available only through the approved path;
- tokens are absent from Git;
- devsecops-gate is a Required Status Check;
- build and deployment depend on the gate;
- the final image is scanned;
- the negative test is demonstrated;
- Production requires approval;
- backup and restore are tested.

