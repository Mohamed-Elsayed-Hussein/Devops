# Lab: LimitRange and CronJob in OpenShift

## Lab Scenario

As a Cluster Administrator, your goal is to set up a project where:

* Every Pod automatically receives CPU and memory requests and limits.
* Users can run jobs on a scheduled basis, for example, for log rotation or backups.

---

## Steps

### Step 1 – Create a New Project

Create a new project called `limitrange-lab`:

```bash
oc new-project limitrange-lab
```

This project will be the workspace for the lab.

---

### Step 2 – Define a LimitRange

Inside the project, create a `LimitRange` to enforce resource policies on Pods and containers:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: core-resource-limits
spec:
  limits:
    - type: Container
      max:
        cpu: "1"
        memory: "1Gi"
      min:
        cpu: "100m"
        memory: "128Mi"
      default:
        cpu: "500m"
        memory: "512Mi"
      defaultRequest:
        cpu: "200m"
        memory: "256Mi"
```

Apply it:

```bash
oc apply -f limitrange.yaml
```

This ensures that all new Pods in this project automatically get resource requests and limits if not explicitly set.

> Note: The template for this LimitRange was referenced from Red Hat’s official documentation: [Cluster Administration – Limits](https://docs.redhat.com/en/documentation/openshift_container_platform/3.4/html/cluster_administration/admin-guide-limits)

---

### Step 3 – Verify the LimitRange

Check that the LimitRange exists and that the values for min, max, and default are applied correctly:

```bash
oc get limitrange
oc describe limitrange core-resource-limits
```

---

### Step 4 – Test with a Pod

Create a simple nginx Pod without specifying resource limits:

```bash
oc run httpd-pod --image=default-route-openshift-image-registry.apps-crc.testing/openshift/httpd --dry-run=client -o yaml > pod-httpd.yaml
oc apply -f pod-httpd.yaml
```

Verify that the default CPU and memory values from the LimitRange were automatically applied:

```bash
oc describe pod httpd-pod
```

---

### Step 5 – Create a CronJob

Create a CronJob that runs every minute and executes a simple command:

```bash
oc create cronjob my-job --image=quay.io/quay/busybox  --schedule="*/1 * * * *" -- date
```

This command prints a message every minute and can be used to test scheduled jobs.

---

### Step 6 – Verify the CronJob

Check that the CronJob exists and is scheduled properly:

```bash
oc get cronjobs
oc describe cronjob my-job
oc get jobs
```

Make sure that a Job is being created successfully for each scheduled run.
