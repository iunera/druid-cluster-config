


https://github.com/apache/druid/blob/master/docs/operations/security-overview.md#enable-tls


# Create keystore & Truststore
```
keytool -keystore keystore.jks -storepass $(pwgen 64 -n1 -s | tr -d '\n' | tee keystorepassword) -keypass  $(cat keystorepassword) -genkey -alias druid  -keyalg RSA -keysize 4096 -validity 3650 -dname "CN=druid" -storetype JKS 

keytool -export -alias druid -keystore keystore.jks -rfc -file druid.cert -storepass $(cat keystorepassword)


# optional: create a truststore based on java defaults truststore
cp -v $JAVA_HOME/lib/security/cacerts ./truststore.jks

# create trust the new cert
keytool -import -file druid.cert -storepass changeit -alias druid -keystore truststore.jks -noprompt -trustcacerts -storetype JKS
```


# Create Secret

```
# to cluster
kubectl --namespace=druid \
  create secret generic keystores \
  --from-file=keystore.jks \
  --from-file=keystorepassword \
  --from-file=truststore.jks


# to file
kubectl --namespace=druid\
  create secret generic keystores \
  --from-file=keystore.jks \
  --from-file=keystorepassword \
  --from-file=truststore.jks \
  --dry-run=client -o yaml \
> druid-jks-keystores-secret.yaml

```

# Cleanup

```
rm keystore.jks
rm keystorepassword
rm truststore.jks
rm druid.cert
```