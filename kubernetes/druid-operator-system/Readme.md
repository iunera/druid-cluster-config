# Druid Operator Deployment

Managed as a FluxCD `HelmRelease` using the `deepbi-charts` Helm repository (`https://charts.deep.bi`).

- **Chart**: `druid-operator` (`>=0.4.0 <0.5.0`, appVersion `v1.3.3`)
- **Helm Repository**: [`deepbi-charts.repo.yaml`](./deepbi-charts.repo.yaml) pointing to `https://charts.deep.bi`
- **Helm Release**: [`druid-operator.helm.yaml`](./druid-operator.helm.yaml)
- **Namespace**: `druid-operator-system`
- **Watched Namespace**: `druid`
- **CRDs**: Maintained declaratively in this directory:
  - [`druid.apache.org_druids.crd.yaml`](./druid.apache.org_druids.crd.yaml)
  - [`druid.apache.org_druidingestions.crd.yaml`](./druid.apache.org_druidingestions.crd.yaml)
- **CRD Install Strategy**: `crds: Skip` on the HelmRelease (Flux Server-Side Apply handles CRD lifecycle and schema updates directly to prevent Helm timeout/race conditions).
