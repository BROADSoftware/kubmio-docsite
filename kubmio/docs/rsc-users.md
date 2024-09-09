# User

## Description

A minio `User` is an abstraction representing a human person, or an entity interacting with the MinIO cluster

A `User` is defined by an UserName and a Password.

A set of attached policies (See `PolicyBinding` below) define what this user can do on the cluster.

Also, a user can also be member of a `Group`. In such case, it will also benefit of all policies attached to the group.

With kubmio, creating a MinIO user is as simple as:

```
cat <<'EOF' | kubectl apply -f -
---
apiVersion: kubmio.com/v1alpha1
kind: User
metadata:
  name: user1
spec:
  description: A simple User
EOF
```

You can list all created users:

```
$ kubectl get --all-namespaces user.kubmio.com
NAMESPACE   NAME    TENANT NS   TENANT    MINIO NAME   ENABLED   SECRET NAME         STATUS   DESCRIPTION
default     user1   kubmio      default   user1        true      kubmio-user-user1   READY
```

As for the `buckets` an alias is provided, with a 'm' as prefix:

```
$ kubectl get --all-namespaces musers
NAMESPACE   NAME    TENANT NS   TENANT    MINIO NAME   ENABLED   SECRET NAME         STATUS   DESCRIPTION
default     user1   kubmio      default   user1        true      kubmio-user-user1   READY
```

And have a deeper look in the `User` object:

```
$ kubectl get users.kubmio.com user1 -o yaml
apiVersion: kubmio.com/v1alpha1
kind: User
metadata:
  finalizers:
  - kubmio.com/finalizer
  name: user1
  namespace: default
  ....(Some data removed)
spec:
  description: A simple User
  enabled: true
  minioName: user1
  secret:
    name: kubmio-user-user1
    passwordField: password
    userNameField: userName
  tenant:
    name: default
    namespace: kubmio
status:
  phase: READY
  secretChecksum: e/dVTe11O7Dy+6vMzumfcSr7KZOWty8brm5A9QoVfBI=
```

- There is a `metadata.finalizers`. This will be used during the User deletion processing.
- `spec.minioName` is the name of the corresponding MinIO 'User' object. By default, it is same as the Kubernetes.
  object name. But it can be different. You can define it explicitly in the object definition, or use the `decoration`
  mechanism described previously.
- `spec.description:` is an optional field. It is only present on the kubernetes resource, without MinIO counterpart.
- The `spec.tenant` property is set to the default tenant, a tenant named `default` in the `kubmio` namespace. But,
  it can be set explicitly, is there is several `tenant` resources defined in the cluster.
- `spec.secret.name` is the name of a secret hosting the user's credential. This secret has been automatically generated
  by kubmio on the `User` object creation, with the data fields defined by `spec.secret.passwordField` and
  `spec.secret.userNameField`.
- The `status.phase` provide information about the current object state. `READY` means the MinIO object counterpart has
  been successfully created.


The user's password has been randomly generated. It can be retrieved with:

```
$ kubectl get secret kubmio-user-user1 -o=jsonpath='{.data.password}' | base64 -d && echo
meR4TZYFd6GXWPQBMAqA
```

The user login is the `spec.minioName`. For convenience, Kubmio copy it in the secret, as `data.userName` property:

```
$ kubectl get secret kubmio-user-user1 -o=jsonpath='{.data.userName}' | base64 -d && echo
user1
```

With this credential, 'user1' can log on the minio console. Or, it can define an alias for using the `mc` MinIO CLI:

```
mc alias set user1 "https://minio.myminioserver.myinfra.com" user1 meR4TZYFd6GXWPQBMAqA
```

> If the Tenant is configured with [name decoration](#name-decoration), the provided name must be the decorated one,
(The `spec.minioName` value)

> This will not works if MinIO is configured with an LDAP server. One must use an `AccessKey` instead

## Disabling a user

A `User` has a property `enabled`. This is the counterpart of the MinIO user's flag. Modifying the property will
switch the state of the MinIO user.

```
kubectl patch user.kubmio.com user1 --type json --patch='[{ "op": "replace", "path": "/spec/enabled", "value": false }]'
```

## Setting your own password

You may also prefer to provide your on password for a user. For this, you may patch the existing secret:

```
kubectl patch secret kubmio-user-user1  --patch='{"stringData": { "password": "mypassword" }}'
```

> The provided password length must be between 8 and 40

Kubmio will forward the password change to MinIO.

You can also reset the existing password by clearing the current one:

```
kubectl patch secret kubmio-user-user1  --patch='{"stringData": { "password": "" }}'
```

In such case, kubmio will generate a new one.


You can also create the secret PRIOR to create the user itself:

```
kubectl create secret generic kubmio-user-user2 --from-literal=password='mypassword2'
```

Then you can create the `User`:

```
cat <<'EOF' | kubectl apply -f -
---
apiVersion: kubmio.com/v1alpha1
kind: User
metadata:
  name: user2
spec:
EOF
```

For this to works, the secret name must be `kubmio-user-<k8sUserName>` and there must be a field `data.password`
with a valid password.

The `User` k8s resource will take ownership of the `secret`. This means the secret will be automatically deleted with the user

