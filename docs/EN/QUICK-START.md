# Quick start

## Requirements

- GitHub Actions enabled;
- Linux runner;
- Docker Engine and Docker Compose v2;
- optional SonarQube Community instance reachable by the runner.

~~~bash
git clone https://github.com/anatolie-ernu/open-devsecops-development-toolkit.git
cd open-devsecops-development-toolkit
chmod +x scripts/*.sh
scripts/validate-configs.sh
~~~

Copy examples/sample-application/devsecops into the application. Copy
templates/github/manual-devsecops-generic.yml to
.github/workflows/manual-devsecops-audit.yml.

Store SONAR_HOST_URL and SONAR_TOKEN in GitHub Actions Secrets. Run the workflow
manually and review its artifacts before enabling blocking mode.

