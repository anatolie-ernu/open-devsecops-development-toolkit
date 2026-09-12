# Ghid de instalare pas cu pas

Acest ghid instalează Open DevSecOps Development Toolkit într-un proiect GitHub
nou. Începeți în mod manual și neblocant. Activați validarea obligatorie numai
după calibrarea rezultatelor.

## Pasul 1 Pregătirea stației sau runnerului

Instalați Git, Docker Engine, Docker Compose v2 și utilitarele shell de bază.

~~~bash
git --version
docker --version
docker compose version
bash --version
~~~

Contul folosit de runner trebuie să poată porni containere Docker. Nu instalați
runnerul de producție pe aceeași gazdă cu serviciul public al aplicației.

## Pasul 2 Clonarea toolkitului

~~~bash
git clone https://github.com/anatolie-ernu/open-devsecops-development-toolkit.git
cd open-devsecops-development-toolkit
chmod +x scripts/*.sh
scripts/validate-configs.sh
~~~

Validatorul trebuie să se termine fără eroare înainte de copierea fișierelor
în proiectul aplicației.

## Pasul 3 Pregătirea repository-ului aplicației

Înlocuiți calea din exemplu cu repository-ul real:

~~~bash
export APPLICATION_DIR=/opt/example-application
mkdir -p "$APPLICATION_DIR/devsecops"
cp -a examples/sample-application/devsecops/. "$APPLICATION_DIR/devsecops/"
mkdir -p "$APPLICATION_DIR/.github/workflows"
cp templates/github/manual-devsecops-generic.yml   "$APPLICATION_DIR/.github/workflows/manual-devsecops-audit.yml"
chmod +x "$APPLICATION_DIR"/devsecops/scripts/*.sh
~~~

Nu copiați rapoarte generate, fișiere .env sau tokenuri.

## Pasul 4 Adaptarea parametrilor proiectului

~~~bash
cd "$APPLICATION_DIR"
cp devsecops/config/project.env.example devsecops/config/project.env
~~~

Modificați numai valori neconfidențiale precum limbajele, directoarele sursă,
directoarele excluse și cheia proiectului. Nu comiteți project.env dacă ajunge
să conțină date sensibile.

Verificați următoarele fișiere:

- devsecops/config/.semgrep.yml;
- devsecops/config/.mega-linter.yml;
- devsecops/config/gitleaks.toml;
- devsecops/sonar-project.properties.

Excludeți vendor, node_modules, build, dist, cache și rapoartele generate.

## Pasul 5 Instalarea SonarQube Community

Pe serverul aprobat:

~~~bash
cd open-devsecops-development-toolkit/platform/sonarqube
cp .env.example .env
chmod 600 .env
~~~

Înlocuiți toate valorile CHANGE_ME din .env. Generați parole unice pentru
PostgreSQL și conturile tehnice.

Pe Linux configurați parametrul necesar motorului de căutare:

~~~bash
sudo sysctl -w vm.max_map_count=524288
echo 'vm.max_map_count=524288' | sudo tee /etc/sysctl.d/99-sonarqube.conf
sudo sysctl --system
~~~

Validați și porniți stackul:

~~~bash
docker compose config
docker compose pull
docker compose up -d
docker compose ps
docker compose logs --tail=100 sonarqube
~~~

Așteptați starea healthy. Nu expuneți PostgreSQL. Publicați interfața SonarQube
prin reverse proxy HTTPS și limitați accesul administrativ.

## Pasul 6 Configurarea SonarQube

1. Deschideți URL-ul HTTPS al instanței.
2. Schimbați imediat parola inițială.
3. Creați un proiect separat pentru aplicație.
4. Generați un token tehnic limitat la proiect.
5. Notați cheia proiectului fără a copia tokenul în documentație.
6. Configurați Quality Gate pentru cod nou.
7. Stabiliți perioada pentru New Code conform ciclului de release.

Tokenul se salvează numai în GitHub Actions Secrets sau într-un secret manager.

## Pasul 7 Configurarea secretelor GitHub

În repository: Settings, Secrets and variables, Actions, New repository secret.

Creați:

| Nume | Valoare |
|---|---|
| SONAR_HOST_URL | URL HTTPS accesibil runnerului |
| SONAR_TOKEN | tokenul tehnic al proiectului |

Nu utilizați un token de administrator. Dacă SonarQube este intern, folosiți un
runner self-hosted care poate rezolva DNS-ul și valida certificatul TLS.

## Pasul 8 Validarea locală

~~~bash
cd "$APPLICATION_DIR"
devsecops/scripts/validate-configs.sh
devsecops/scripts/run-local-audit.sh
find devsecops/reports -maxdepth 2 -type f -print
~~~

Analiza poate raporta probleme. În etapa pilot, scopul este obținerea dovezilor,
nu blocarea dezvoltării.

## Pasul 9 Primul audit GitHub manual

1. Comiteți fișierele într-un branch dedicat.
2. Deschideți un pull request și verificați modificările.
3. Integrați workflow-ul manual.
4. Deschideți Actions.
5. Selectați Manual DevSecOps Audit.
6. Apăsați Run workflow.
7. Selectați branch-ul și adâncimea standard.
8. Așteptați finalizarea tuturor joburilor.
9. Descărcați artefactele Semgrep, Trivy, Gitleaks, MegaLinter și SBOM.
10. Înregistrați constatările confirmate.

Workflow-ul manual nu trebuie să conțină trigger push, pull_request sau schedule.

## Pasul 10 Calibrarea

Rulați pilotul între două și patru săptămâni. Pentru fiecare constatare
înregistrați severitatea, validitatea, proprietarul, termenul și excepția
aprobată. Suprimările trebuie să fie înguste și să aibă expirare.

Ordinea recomandată pentru blocare este:

1. secrete confirmate;
2. build și teste;
3. vulnerabilități CRITICAL remediabile;
4. reguli Semgrep ERROR confirmate;
5. SonarQube Quality Gate;
6. licențe ORT deny;
7. HIGH și coverage după aprobare.

## Pasul 11 Activarea gate-ului obligatoriu

~~~bash
cd "$APPLICATION_DIR"
cp devsecops/workflows/recommended-blocking.yml.example   .github/workflows/devsecops-gate.yml
~~~

Sau utilizați exemplul distribuit de toolkit:

~~~bash
cp /path/to/open-devsecops-development-toolkit/templates/github/mandatory-devsecops-gate.yml.example   .github/workflows/devsecops-gate.yml
~~~

Revizuiți workflow-ul și:

- păstrați numele jobului devsecops-gate;
- eliminați continue-on-error;
- utilizați exit-code 1 pentru pragurile blocante;
- fixați acțiunile și imaginile la versiuni aprobate;
- limitați permisiunile;
- adăugați timeout;
- păstrați rapoartele ca artefacte chiar la eșec.

## Pasul 12 Branch Protection sau Ruleset

În GitHub deschideți Settings, Branches sau Rules, Rulesets.

1. Selectați ramura principală.
2. Activați Require a pull request before merging.
3. Activați Require status checks to pass.
4. Selectați exact devsecops-gate după prima sa rulare.
5. Activați Require branches to be up to date.
6. Dezactivați bypass pentru contributorii obișnuiți.
7. Interziceți force push și ștergerea ramurii.
8. Salvați regula.

Nu selectați un nume instabil de step. Required Status Check trebuie să fie
jobul devsecops-gate.

## Pasul 13 Condiționarea buildului și deploymentului

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

Scanați imaginea finală după build. Deploymentul trebuie să folosească exact
digestul artefactului validat.

## Pasul 14 Protejarea mediului Production

În Settings, Environments creați production și configurați:

- required reviewers;
- prevenirea autoaprobării unde este disponibilă;
- branchuri sau taguri permise;
- secrete specifice mediului;
- timeout și procedură de rollback.

Aprobarea Production nu înlocuiește devsecops-gate; ambele sunt necesare.

## Pasul 15 ORT și SBOM

Rulați ORT într-un job separat pentru Analyzer, Scanner, Advisor, Evaluator și
Reporter. Publicați CycloneDX sau SPDX și raportul de licențe. Politica allow,
review, deny și unknown trebuie aprobată juridic înainte să blocheze release-ul.

## Pasul 16 PR Agent și Ollama

Instalați Ollama pe o gazdă internă. Permiteți acces numai runnerului aprobat.
Configurați PR-Agent inițial în mod consultativ. Nu trimiteți cod restricționat,
secrete sau date interne către un model extern. Review-ul uman rămâne obligatoriu.

## Pasul 17 Testul negativ obligatoriu

1. Creați un branch de test.
2. Adăugați o mostră inofensivă detectată de o regulă calibrată.
3. Deschideți pull request.
4. Confirmați că devsecops-gate eșuează.
5. Confirmați că merge-ul este dezactivat.
6. Confirmați că buildul sau deploymentul nu pornește.
7. Eliminați mostra.
8. Confirmați că gate-ul devine verde.
9. Confirmați că deploymentul poate continua numai după succes.

Nu considerați implementarea blocking acceptată fără acest test.

## Pasul 18 Backup și mentenanță

Faceți backup consistent al PostgreSQL și al volumelor SonarQube. Testați
restaurarea trimestrial. Actualizați lunar acțiunile, imaginile și regulile prin
pull request. Revizuiți excepțiile expirate și monitorizați durata joburilor.

## Pasul 19 Rollback

Pentru pilot, dezactivați workflow-ul manual. Pentru gate-ul obligatoriu,
aprobați formal fereastra de rollback înainte de eliminarea Required Status
Check. Nu ștergeți volumele SonarQube cu docker compose down -v decât dacă
eliminarea datelor este aprobată și există backup verificat.

## Verificare finală

- validatorul local trece;
- auditul manual produce toate artefactele;
- SonarQube este accesibil numai pe traseul aprobat;
- tokenurile nu sunt în Git;
- devsecops-gate este Required Status Check;
- buildul și deploymentul depind de gate;
- imaginea finală este scanată;
- testul negativ este demonstrat;
- Production necesită aprobare;
- backupul și restaurarea sunt testate.

