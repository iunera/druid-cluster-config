# druid-cluster-config

This repo contains the Configuration of a production-ready kubernetes-native Apache Druid cluster based on [druid-operator](https://github.com/datainfrahq/druid-operator) and [fluxcd](https://fluxcd.io/flux/components/kustomize/kustomizations/) as gitops tool. 
* Kubernetes-native means that there are ...
  * no zookeeper in place for service discovery 😍
  * middlemanager are replaced by kubernetes jobs which allows use to utilize cluster autoscaling 😍
  * horizontal pod autoscaling (HPA) for historical nodes 
* Production-ready means:
  * TLS Encryption on all components
  * OAuth2 Login with Druids UI
  * Service users are enabled in Druid (via basic authentication and local users) 
  * Authorization Concept for different tiers of access
  * Observability is ensured by the enhanced druid-exporter features

## Background and further reading

For architectural context and step-by-step guides to running a production-ready, Kubernetes-native Apache Druid cluster (ZooKeeper-less, MiddleManager-less, TLS, and GitOps with FluxCD and druid-operator), see these in-depth articles:

- [Infrastructure setup for enterprise Apache Druid on Kubernetes — building the foundation](https://www.iunera.com/kraken/big-data-lessons/infrastructure-setup-for-enterprise-apache-druid-on-kubernetes-building-the-foundation/)
- [Installing a production-ready Apache Druid cluster on Kubernetes (Part 2) — Druid deployment preparation](https://www.iunera.com/kraken/big-data-lessons/installing-a-production-ready-apache-druid-cluster-on-kubernetes-part-2-druid-deployment-preparation/)
- [Apache Druid on Kubernetes: production-ready with TLS, MiddleManager-less, ZooKeeper-less, GitOps](https://www.iunera.com/kraken/time-series/apache-druid-on-kubernetes-production-ready-with-tls-mm%E2%80%91less-zookeeper%E2%80%91less-gitops/)

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
---apiVersion: v1
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

We choose fair [code, fair work, fair payment, open  collaboration](https://www.license-token.com)

## [Open Compensation Token License](https://github.com/open-compensation-token-license/license/blob/main/LICENSE.md)

```
Licensed under the OPEN COMPENSATION TOKEN LICENSE (the "License").

You may not use this file except in compliance with the License.

You may obtain a copy of the License at
<https://github.com/open-compensation-token-license/license/blob/main/LICENSE.md>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either expressed or implied. 
See the License for the specific language governing permissions and
limitations under the License.

@octl.sid: 1b6f7a5d-8dcf-44f1-b03a-77af04433496
```
* Why did we [choose the OCTL as alternative to the GNU AGPL v3 License](https://www.license-token.com/wiki/unveiling-gnu-agpl-v3-summary)?
* Why we [do NOT apply Apache 2.0 License](https://www.license-token.com/wiki/the-downside-of-apache-license-and-why-i-never-would-use-it)?

### Need Expert Apache Druid Consulting?

**Maximize your return on data** with professional Druid implementation and optimization services. From architecture design to performance tuning and AI integration, our experts help you navigate Druid's complexity and unlock its full potential.

**[Get Expert Druid Consulting →](https://www.iunera.com/apache-druid-ai-consulting-europe/)**

For more information about our services and solutions, visit [www.iunera.com](https://www.iunera.com).

### Contact & Support

Need help? Let 

- **Website**: [https://www.iunera.com](https://www.iunera.com)
- **Professional Services**: Contact us through [email](mailto:consulting@iunera.com?subject=Druid%20MCP%20Server%20inquiry) for [Apache Druid enterprise consulting, support and custom development](https://www.iunera.com/apache-druid-ai-consulting-europe/)
- **Open Source**: This project is open source and community contributions are welcome