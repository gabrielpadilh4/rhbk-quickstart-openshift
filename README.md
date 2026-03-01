# RHBK Operator - Basic Quickstart Deployment for OpenShift

This guide covers the basic deployment of Red Hat build of Keycloak Operator in Red Hat Openshift.

## Prerequisites
- An OpenShift cluster with the Keycloak Operator installed.
- `oc` CLI tool authenticated to your cluster.
- `openssl` for generating local certificates.

## Configuration Setup
Before deploying, update the `keycloak.yaml` with your specific environment details:
- **hostname**: A valid hostname where RHBK will be exposed. Usually, for POC's I use the **.apps.<clustername>.<domain>** Openshift subdomain. E.g: `rhbk.apps.myclustername.mydomain.com`
It is possible to get the Openshift subdomain with:
   ~~~
   oc get route console -n openshift-console -o jsonpath='{.spec.host}' | sed 's/console-openshift-console.//' && echo
   ~~~

## Deployment Steps

1. Provision the database
Deploy the PostgreSQL StatefulSet, Service, and the associated credentials secret.
   ```
   oc apply -f postgresql.yaml
   ```
   **Note**: The current configuration uses emptyDir for storage. This is suitable for development but will not persist data if the database pod is deleted. **Note**: Sometimes a cluster may not have the image `postgresql:15-el9`, check for a valid image with `oc get images -n openshift-image-registry | grep postgresql`

2. Configure TLS Security
Generate a self-signed certificate for `rhbk.apps.myclustername.mydomain.com` and store it as a Kubernetes TLS secret. The Operator will create a pass-through route to `rhbk.apps.myclustername.mydomain.com`.
   ```
   openssl req -subj '/CN=rhbk.apps.myclustername.mydomain.com`/O=Example Keycloak./C=US' -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
   oc create secret tls keycloak-tls-secret --cert certificate.pem --key key.pem
   ```
   **Note**: Remember to change the "CN" with your RHBK hostname. E.g: `rhbk.apps.myclustername.mydomain.com`

3. Deploy Keycloak CR
Apply the Keycloak Custom Resource (CR). This triggers the Operator to configure the Keycloak instance using the PostgreSQL host postgres-db and the TLS secret created in the previous step.
   ```
   oc apply -f keycloak.yaml
   ```

4. Access the Keycloak route `rhbk.apps.myclustername.mydomain.com` and use the 'rhdh-keycloak-initial-admin' secret for the username and password to login in to the RHBK console
   ~~~
   oc get secret rhdh-keycloak-initial-admin -o jsonpath='{.data.username}' | base64 --decode && echo # username
   oc get secret rhdh-keycloak-initial-admin -o jsonpath='{.data.password}' | base64 --decode && echo # password
   ~~~
