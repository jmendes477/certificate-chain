
# Commands  certificate-chain
1.Generate full-chain of self-signed certificates.

2.Import into java keystore (.jks)
## Chain Certificates:
Root CA 
```shell
openssl genrsa -out root.key 2048
```
```shell
openssl req -new -key root.key -out root.csr -config root_req.config
```
```shell
openssl ca -in root.csr -out root.pem -config root.config -selfsign -extfile ca.ext -days 1095
```
Intermediate Certificate
```shell
openssl genrsa -out intermediate.key 2048
```
```shell
openssl req -new -key intermediate.key -out intermediate.csr -config intermediate_req.config
```
```shell
openssl ca -in intermediate.csr -out intermediate.pem -config root.config -extfile ca.ext -days 730
```
Leaf Certificate
```shell
openssl genrsa -out leaf.key 2048
```
```shell
openssl req -new -key leaf.key -out leaf.csr -config leaf_req.config
```
```shell
openssl ca -in leaf.csr -out leaf.pem -config intermediate.config -days 365
```
## Create Chain with Intermediate and Root CA: 
```shell
cat intermediate.pem | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' >> intermediate_and_root_chain.pem
```
```shell
cat root.pem | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' >> intermediate_and_root_chain.pem
```
Convert pem into pkcs12
```shell
openssl pkcs12 -export -inkey leaf.key -in leaf.pem -certfile intermediate_and_root_chain.pem -out keystore.p12 -name "my-cert-alias"
```
Import into java keystore jks
```shell
keytool -importkeystore -destkeystore keystore.jks -srcstoretype PKCS12 -srckeystore keystore.p12
```
