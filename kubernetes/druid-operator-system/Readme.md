# Druid Operator Deployment

Managed as a FluxCD `HelmRelease` using the `deepbi-charts` repository (`https://charts.deep.bi`).

- Chart: `druid-operator` (`>=0.4.0 <0.5.0`, appVersion `v1.3.3`)
- Namespace: `druid-operator-system`
- Watched Namespace: `druid`
- CRDs: Managed by Helm (`CreateReplace`)
