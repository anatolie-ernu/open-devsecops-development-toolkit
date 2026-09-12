# Mandatory deployment validation

1. Run manual scans for two to four weeks.
2. Calibrate findings and time-bound exceptions.
3. Pin action and container image versions.
4. Activate templates/github/mandatory-devsecops-gate.yml.example.
5. Remove continue-on-error and overrides that convert errors to success.
6. Keep the devsecops-gate job name stable.
7. Require devsecops-gate in branch protection or a ruleset.
8. Disable bypass for ordinary contributors.
9. Make build and deployment depend on the gate.
10. Scan the final immutable image or artifact.
11. Protect the Production environment with required reviewers.
12. Run a negative test proving that merge and deployment are blocked.

~~~yaml
deploy-production:
  needs: [devsecops-gate, build]
  if: needs.devsecops-gate.result == 'success'
  environment: production
  runs-on: ubuntu-latest
  steps:
    - run: ./deploy.sh VALIDATED_ARTIFACT_DIGEST
~~~

Deployment must not use an artifact different from the validated artifact.

