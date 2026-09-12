# Pornire rapidă

## Cerințe

- GitHub Actions activ;
- runner Linux;
- Docker Engine și Docker Compose v2;
- opțional, SonarQube Community accesibil runnerului.

~~~bash
git clone https://github.com/anatolie-ernu/open-devsecops-development-toolkit.git
cd open-devsecops-development-toolkit
chmod +x scripts/*.sh
scripts/validate-configs.sh
~~~

Copiați directorul examples/sample-application/devsecops în aplicație. Copiați
templates/github/manual-devsecops-generic.yml ca
.github/workflows/manual-devsecops-audit.yml.

Stocați SONAR_HOST_URL și SONAR_TOKEN în GitHub Actions Secrets. Porniți
workflow-ul manual și analizați artefactele înainte de activarea modului blocant.

