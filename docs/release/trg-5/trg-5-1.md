---
title: TRG 5.01 - Helm Chart requirements
---

| Author                | Status | Created     | Post-History    |
|-----------------------|--------|-------------|-----------------|
| Catena-X System Team  | Active | 10-Nov-2022 | Initial release |

## Description

All Catena-X/Tractus-X components **must** be installable via Helm and meet following requirements. The product's Helm
chart **has to meet** all of these criteria!

- Helm chart **must be released**.
- Appropriate versioning for `version` and `appVersion` has to be used in `Chart.yaml`.
- The released Helm chart **must not contain any environment specific** `values-xyz.yaml` files.
- Helm chart `values.yaml` file **must contain proper default values/placeholders**.
  - No hostname provided for ingress
  - Ingress is disabled
  - No references to any secret engine service (e.g.: Hashicorp Vault)
  - Dependencies should be prefixed with the `nameOverride` and/or `fullnameOverride` properties
  - Image tag is set to the **Chart.yaml** `appVersion` property
- Helm chart **must be deployable to any environment without overwriting default values** with a simple `helm install`
  command.
- If there is an **Ingress** resource present, it can be turned off, and it is disabled by default.
- Helm dependencies have to be declared in `Chart.yaml` file.

## Why

### Released Helm Charts

Tractus-X components can only be used and bundled with other components of Tractus-X if there is a version of the
component with reliable content. This can only be achieved by creating a release of your Helm chart. We recommend to use
GitHub [Chart Releaser Action](https://github.com/helm/chart-releaser-action).

### Versioning

Versioning is essential when it comes to releasing Helm charts. In an ideal world your chart version (defined
in `version` inside `Chart.yaml`) matches the application version (defined in `appVersion` inside `Chart.yaml`) which
the Helm chart will deploy. This makes it for users easier to get an idea what will be installed in which version.
Please refer also to [_TRG 5.03 - Version Strategy_](trg-5-3.md).

To achieve a proper versioning of your Helm chart keep in mind, that it shall not contain version information that could
change outside the Helm chart, e.g. container image tags (→ don't use container image tag _latest_).

### The `values.yaml` File

The `values.yaml` file is essential for Helm charts. The file **must** contain all values the chart is expecting and
there **must** be no other values files except of `values.yaml` file. Released Helm charts **must** contain
only `values.yaml` file.

### Dependencies

If a Helm chart has dependencies to other Helm charts, e.g. PostgreSQL, these dependencies **must** be specified in
the `Chart.yaml` file. Do not use outdated `requirements.yaml`.
