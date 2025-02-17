---
title: TRG 0.0 - DRAFT Helm test
---

:::caution
Proposed release date: "mandatory after": 19th of May 2023
:::

| Author               | Status | Created     | Post-History  |
|----------------------|--------|-------------|---------------|
| Catena-X System Team | Draft  | 23-Feb-2023 | Draft release |

## Description

Helm charts and their dependend images (the application to test) need to be validated by installing them and verifing that they are running successfully.

Helm test is the technical solution helm provides to verify that a helm chart works as expected. It basically installs the helm chart (which installs and runs the application) and then allows an additional
container to run. This test container can do everything, especially calling an api, executing tests etc.

This needs to be running in a Github Action in the repository which contains the Helm Chart, which gets released.

Example Github Action Workflow:

- Build all necessary container images / pull the container image under test
- Create [kind](https://kind.sigs.k8s.io/) setup (kind creates a kubernetes cluster through container images, github action supports this which allows you to create a kubernetes cluster inside the github action run)
- Make the necessary container images available through the kind setup (see github action example, it uses the local kind registry)
- Prepare and execute helm test
- Fail on a failing helm test

Technical requirements:

- A Github action exist which builds or uses the helm chart which gets released
- The Github action can be triggered manually through Github WebUI [manually running a workflow](https://docs.github.com/en/actions/managing-workflow-runs/manually-running-a-workflow)
- Helm test verifies that the application is up and running
  - Example verifications:
    - Frontend: Frontend returns valid page
    - API: API is reachable and returns valid response (access denied / test user can login)

Example .github/workflows/helm-test.yaml

```yaml
name: Lint and Test Charts

on:
    pull_request:
    workflow_dispatch:

jobs:
    lint-test:
        runs-on: ubuntu-latest
        steps:
            - name: Checkout
              uses: actions/checkout@v3
              with:
              fetch-depth: 0

            - name: Kubernetes KinD Cluster
              uses: container-tools/kind-action@v1
        
            - name: Build image
              uses: docker/build-push-action@v3
              with:
                context: .
                push: true
                tags: kind-registry:5000/app-dashboard:testing
        
            - name: Set up Helm
              uses: azure/setup-helm@v3
              with:
                version: v3.9.3
        
            - uses: actions/setup-python@v4
              with:
                python-version: '3.9'
                check-latest: true
        
            - name: Set up chart-testing
              uses: helm/chart-testing-action@v2.3.1
        
            - name: Run chart-testing (list-changed)
              id: list-changed
              run: |
                changed=$(ct list-changed --target-branch ${{ github.event.repository.default_branch }})
                if [[ -n "$changed" ]]; then
                   echo "::set-output name=changed::true"
                fi
            - name: Run chart-testing (lint)
              run: ct lint --validate-maintainers=false --target-branch ${{ github.event.repository.default_branch }}
        
            - name: Run chart-testing (install)
              run: ct install --charts charts/app-dashboard
              if: steps.list-changed.outputs.changed == 'true'
```

## Why

Helm charts are defined as the sane default for cloud native applications in Eclipse Tractus-X. To make sure that the aligned quality verification of applications are as close as possible to the production setup,
it is critical that the test setup is doing what it would do on a production setup. Therefore Helm test will provide this mechanism together with kind and allows us to run e2e verifications directly on GitHub Actions.

GitHub Actions simplifies the open source infrastructure requirements as otherwise a test k8s cluster would be required, which might not be available in the long run.

This is the base for future e2e tests.

This also allows to move the responsibility of verifing the helm chart functionality from the system team back to the development team.
