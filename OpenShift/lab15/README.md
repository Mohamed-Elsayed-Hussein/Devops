# Creating a DaemonSet in OpenShift with a ServiceAccount

This guide shows how to run a Pod on every node in your OpenShift project and assign it a ServiceAccount with the `anyuid` permission.

---

## Create a ServiceAccount

Start by creating a ServiceAccount for the DaemonSet:

```bash
oc create serviceaccount daemon-sa
```

This ServiceAccount will be used to give the Pods the necessary permissions.

---

## Assign the `anyuid` SecurityContextConstraint

OpenShift uses SecurityContextConstraints (SCC) to control which users and containers can run as which UID. To allow your DaemonSet to run properly, assign the `anyuid` SCC to the ServiceAccount:

```bash
oc adm policy add-scc-to-user anyuid -z daemon-sa
```

The `-z` flag means we are assigning the SCC to the ServiceAccount instead of a regular user.

---

## Prepare the DaemonSet YAML

Here’s an example DaemonSet that uses the ServiceAccount:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset
  labels:
    app: daemonset
spec:
  selector:
    matchLabels:
      app: daemonset
  template:
    metadata:
      labels:
        app: daemonset
    spec:
      serviceAccountName: daemon-sa
      containers:
      - name: httpd
        image: default-route-openshift-image-registry.apps-crc.testing/openshift/httpd
        ports:
        - containerPort: 80
```

Notice how the `serviceAccountName` is set to `daemon-sa`. This ensures all Pods created by the DaemonSet use the ServiceAccount and its permissions.

---

## Apply the DaemonSet

```bash
oc apply -f daemonset.yaml
```

---

## Verify the DaemonSet

Check that the DaemonSet is running and the Pods are deployed on all nodes:

```bash
oc get daemonsets
oc describe daemonset daemonset
oc get pods -o wide
oc logs <pod-name>
```



