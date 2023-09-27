# Manual Manifest Generation

```
helm -n druid-operator-system template druid-operator ./druid-operator/chart --set env.WATCH_NAMESPACE="druid" | tee druid-operator.manifest.yaml
```
