# druid-cluster-config

This repo contains the Configuration of a production-ready kubernetes-native Apache Druid cluster based on [druid-operator](https://github.com/datainfrahq/druid-operator) and [fluxcd](https://fluxcd.io/flux/components/kustomize/kustomizations/) as gitops tool. 

# Installation
The Repo is included in fluxcd with following setup.

## Gitops
```
---
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: druid-cluster-config
  namespace: flux-system
spec:
  interval: 1m0s
  ref:
    branch: main
  timeout: 60s
  url: ssh://git@github.com/iunera/druid-cluster-config
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: druid-cluster-config
  namespace: flux-system
spec:
  interval: 10m0s
  path: ./kubernetes/
  prune: true
  sourceRef:
    kind: GitRepository
    name: druid-cluster-config
```

## Persistency

Currently we have the legacy approach of PVCs for the deepstorage. In future development (when we have time) we will migrate to S3 Object storage.

```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: iuneradruid-deepstorage
  namespace: druid
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 500Gi
  volumeMode: Filesystem
```

The postgres serving as metastore database wants to have a PVC, too.
```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: iuneradruid-metastore-postgres-pvc
  namespace: druid
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
  volumeMode: Filesystem
```


## Cluster internal TLS encryption

Details here: https://github.com/apache/druid/blob/master/docs/operations/security-overview.md#enable-tls


### Create keystore & Truststore
```
keytool -keystore keystore.jks -storepass $(pwgen 64 -n1 -s | tr -d '\n' | tee keystorepassword) -keypass  $(cat keystorepassword) -genkey -alias druid  -keyalg RSA -keysize 4096 -validity 3650 -dname "CN=druid" -storetype JKS 

keytool -export -alias druid -keystore keystore.jks -rfc -file druid.cert -storepass $(cat keystorepassword)


# optional: create a truststore based on java defaults truststore
cp -v $JAVA_HOME/lib/security/cacerts ./truststore.jks

# create trust the new cert
keytool -import -file druid.cert -storepass changeit -alias druid -keystore truststore.jks -noprompt -trustcacerts -storetype JKS
```

### Create Secret

```
# to cluster
kubectl --namespace=druid \
  create secret generic keystores \
  --from-file=keystore.jks \
  --from-file=keystorepassword \
  --from-file=truststore.jks


# or to a yaml file 
# you should encrypt the file with sops
kubectl --namespace=druid\
  create secret generic keystores \
  --from-file=keystore.jks \
  --from-file=keystorepassword \
  --from-file=truststore.jks \
  --dry-run=client -o yaml \
> druid-jks-keystores-secret.yaml
```

## Deploy the Metastore Database (Postgres)
To deploy PostgreSQL using Helm, following manifest [kubernetes/druid/postgres](./kubernetes/druid/postgres). 

## Druid Cluster Deployment 
Inside the [kubernetes/druid/druidcluster/](./kubernetes/druid/druidcluster) the whole deployment of our central cluster is accomplished.

# License
[Open Compensation Token License, Version 0.20](https://github.com/open-compensation-token-license/license/blob/main/LICENSE.md)
