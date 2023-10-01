# Manuelle Manifest Generation

```
helm -n druid-operator-system template druid-operator /Users/chris/git/github/schmichri/druid-operator/chart  --set env.WATCH_NAMESPACE="druid" > druid-operator.manifest.yaml
```
