# Deploying NGINX on OpenShift with Custom TLS Route

This lab demonstrates how to deploy an NGINX application on OpenShift, configure a custom Service Account with proper permissions, expose the deployment through a secure route using a self-signed TLS certificate, and verify access over port **8080**.

## Steps Overview

First, a new OpenShift project is created to isolate the deployment environment:

```bash
oc new-project nginx-deployment
```

A dedicated Service Account named `nginx-sa` is created to run the NGINX pod securely. Since NGINX requires specific permissions to run properly, it is granted the `anyuid` Security Context Constraint:

```bash
oc create serviceaccount nginx-sa
oc adm policy add-scc-to-user anyuid -z nginx-sa
```

Next, an NGINX deployment is created using the unprivileged image from Quay.io:

```bash
oc create deployment nginx-deploy --image=quay.io/nginx/nginx-unprivileged
```

The deployment is then associated with the custom Service Account:

```bash
oc set serviceaccount deployment/nginx-deploy nginx-sa
```

After confirming that the pod is running successfully, the deployment is exposed as a service on port **8080**:

```bash
oc expose deployment nginx-deploy --port=8080
```

To make the service accessible externally, a new route is created by exposing the service:

```bash
oc expose service nginx-deploy --port=8080
```

You can verify that the route was created successfully using:

```bash
oc get route
```

To enable HTTPS access, a self-signed TLS certificate and key are generated using OpenSSL:

```bash
openssl req -x509 -newkey rsa:4096 -nodes -keyout myapp.key -out myapp.crt
```

A secure edge route is then created with the certificate and key attached, still serving on port **8080**:

```bash
oc create route edge --service nginx-deploy --port=8080 --cert=myapp.crt --key=myapp.key
```

Finally, test the application using `curl` with the `-k` option to skip certificate verification:

```bash
curl -k https://nginx-deploy-nginx-deployment.apps-crc.testing
```
---

## Serving a Custom HTML Page with ConfigMap

1. **Create a ConfigMap from your custom HTML file**

   Here we assume you have an `index.html` file ready locally. This command packages it into a ConfigMap:

   ```bash
   oc create configmap index.html --from-file=index.html
   ```

2. **Mount the ConfigMap into the NGINX deployment**

   Mount the ConfigMap to the default NGINX HTML directory so it will serve your custom page:

   ```bash
   oc set volumes deployment/nginx-deploy \
     --add \
     --type=configmap \
     --configmap-name=index.html \
     --mount-path=/usr/share/nginx/html
   ```

3. **Test the deployment**

   Access the app over HTTPS (skip certificate verification with `-k` if using a self-signed certificate):

   ```bash
   curl -k https://nginx-deploy2-nginx-deployment.apps-crc.testing
   ```

You should see your **custom HTML page** served by NGINX.

