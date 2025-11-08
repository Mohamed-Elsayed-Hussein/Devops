# OpenShift – Creating a User and Assigning Cluster Admin Access

This guide walks through creating a new user in OpenShift using an `htpasswd` file for authentication, then granting that user cluster-admin privileges. The process includes creating credentials, updating OAuth configuration, and managing access roles.

---

First, log in as the cluster administrator:

```bash
oc login -u kubeadmin
```

Administrative access is required for configuring authentication and managing roles within the cluster.

Next, create a credentials file that will store user authentication data:

```bash
htpasswd -B user.txt mohamed
```

This command creates (or updates) a file called `user.txt` and adds a user named **mohamed** with a password you’ll be prompted to enter. The `-B` flag stores the password securely using bcrypt encryption.

You can use alternative tools to generate the `htpasswd` file, such as `openssl passwd -apr1 <password>` or `bcrypt`, but `htpasswd` (from the `httpd-tools` package) is the standard and most convenient method supported by OpenShift.

Once the file is ready, create a secret from it:

```bash
oc create secret generic users --from-file=htpasswd=user.txt -n openshift-config
```

Here, the key name `htpasswd` after `--from-file` is not arbitrary—it must be exactly this name because OpenShift’s OAuth expects that specific key when reading credentials.

You can confirm the secret was created successfully by running:

```bash
oc get secrets -n openshift-config
```

After that, open the OpenShift web console and navigate to:
**Administration → Cluster Settings → Configuration → OAuth**.
Add a new identity provider of type **htpasswd** and point it to the secret you just created.

Once saved, the authentication pods will automatically restart to apply the new configuration. You can monitor that process using:

```bash
oc get pods -n openshift-authentication --watch
```

Each few minutes, a cron job triggers a restart of those pods to keep authentication in sync.

To test the new user, log in as follows:

```bash
oc login -u mohamed -p <your_password> https://api.crc.testing:6443
```

You can verify your login session with:

```bash
oc whoami --show-console
```

At this point, the user **mohamed** can log in but does not yet have administrative privileges. Since this user only has the **self-provisioner** role by default, switch back to the kubeadmin account before assigning higher privileges:

```bash
oc login -u kubeadmin
```

Now assign the cluster-admin role to the new user:

```bash
oc adm policy add-cluster-role-to-user cluster-admin mohamed
```

If you want to see other available roles besides `cluster-admin`, you can run:

```bash
oc get clusterroles
```

For more details about RBAC roles and permissions, refer to the Red Hat documentation:
[Using RBAC – OpenShift 4.8 Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.8/html/authentication_and_authorization/using-rbac)

If you later need to add or modify users, you can use the same command as before to update the `user.txt` file, then replace the existing secret:

```bash
htpasswd -B user.txt <username>
oc create secret generic users --from-file=htpasswd=user.txt -n openshift-config --dry-run=client -o yaml | oc replace -f -
```

This approach updates the credentials without recreating the secret manually or reconfiguring OAuth.

