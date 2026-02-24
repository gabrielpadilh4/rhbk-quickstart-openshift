# RHBK Operator - Basic Quickstart Deployment for OpenShift

#### Prerequisites
- An OpenShift cluster with the Keycloak Operator installed.
- `oc` CLI tool authenticated to your cluster.
- `openssl` for generating local certificates.

#### Deployment Steps

1. Provision the database
Deploy the PostgreSQL StatefulSet, Service, and the associated credentials secret.
   ```
   oc apply -f postgresql.yaml
   ```
   **Note**: The current configuration uses emptyDir for storage. This is suitable for development but will not persist data if the database pod is deleted.

2. Configure TLS Security
Generate a self-signed certificate for 'example.keycloak.org' and store it as a Kubernetes TLS secret. The Operator will create a pass-through route to 'example.keycloak.org'.
   ```
   openssl req -subj '/CN=example.keycloak.org/O=Example Keycloak./C=US' -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
   oc create secret tls keycloak-tls-secret --cert certificate.pem --key key.pem
   ```

3. Deploy Keycloak CR
Apply the Keycloak Custom Resource (CR). This triggers the Operator to configure the Keycloak instance using the PostgreSQL host postgres-db and the TLS secret created in the previous step.
   ```
   oc apply -f keycloak.yaml
   ```

4. Access the Keycloak route 'example.keycloak.org' and use the 'example-kc-initial-admin' secret for the username and password to login in to the RHBK console
   ~~~
   oc get secret example-kc-initial-admin -o jsonpath='{.data.username}' | base64 --decode # username
   oc get secret example-kc-initial-admin -o jsonpath='{.data.password}' | base64 --decode # password
   ~~~
