# Validarea obligatorie înainte de deployment

1. Rulați scanările manual timp de două până la patru săptămâni.
2. Calibrați constatările și excepțiile cu termen de expirare.
3. Fixați versiunile acțiunilor și imaginilor container.
4. Activați templates/github/mandatory-devsecops-gate.yml.example.
5. Eliminați continue-on-error și valorile care transformă erorile în succes.
6. Păstrați stabil numele jobului devsecops-gate.
7. Configurați devsecops-gate ca required status check.
8. Dezactivați bypass pentru contributorii obișnuiți.
9. Condiționați buildul și deploymentul de rezultatul gate-ului.
10. Scanați imaginea sau artefactul final imuabil.
11. Protejați mediul Production cu required reviewers.
12. Executați un test negativ care demonstrează blocarea merge-ului și deploymentului.

~~~yaml
deploy-production:
  needs: [devsecops-gate, build]
  if: needs.devsecops-gate.result == 'success'
  environment: production
  runs-on: ubuntu-latest
  steps:
    - run: ./deploy.sh VALIDATED_ARTIFACT_DIGEST
~~~

Deploymentul nu trebuie să utilizeze un artefact diferit de cel validat.

