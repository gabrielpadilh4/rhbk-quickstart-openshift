1. Start the database
```
oc apply -f postgresql.yaml
```
This creates a database secret, a database statefulset and the database service.

2. Create a TLS secret that will be used by 
```
openssl req -subj '/CN=example.keycloak.org/O=Example Keycloak./C=US' -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
oc create secret tls keycloak-tls-secret --cert certificate.pem --key key.pem
```
This creates the private key + public key of the TLS certificate that will be used by Keycloak.

3. Create the Keycloak CR
~~~
oc apply -f keycloak.yaml
~~~