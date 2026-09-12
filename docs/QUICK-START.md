# Quick start

Requirements: GitHub Actions, a Linux runner and Docker Engine with Compose v2.

~~~bash
git clone https://github.com/OWNER/open-devsecops-development-toolkit.git
cd open-devsecops-development-toolkit
chmod +x scripts/*.sh
scripts/validate-configs.sh
~~~

Copy examples/sample-application/devsecops into the application. Copy
templates/github/manual-devsecops-generic.yml to
.github/workflows/manual-devsecops-audit.yml. Store SONAR_HOST_URL and
SONAR_TOKEN in GitHub Actions Secrets, then run the workflow manually.

