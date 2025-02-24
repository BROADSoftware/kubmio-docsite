# GroupBinding

## Description

To include a `User` in a `Group`, a `GroupBinding` resource must be created:

```
cat <<'EOF' | kubectl apply -f -
---
apiVersion: kubmio.com/v1alpha1
kind: GroupBinding
metadata:
  name: group1-user1
spec:
  description: "Add user1 to group1"
  group: group1
  user:
    name: user1
EOF
```

You may display the resulting kubernetes manifest:

```
$ kubectl get groupbindings group1-user1  -o yaml
apiVersion: kubmio.com/v1alpha1
kind: GroupBinding
metadata:
  finalizers:
  - kubmio.com/finalizer
  name: group1-user1
  namespace: default
  ..... (Some data removed)
spec:
  description: Add user1 to group1
  group: group1
  tenant:
    name: default
    namespace: kubmio
  user:
    name: user1
    namespace: default
status:
  groupMinioName: group1
  phase: READY
  userMinioName: user1
```

- There is a `metadata.finalizers`. This will be used during the PolicyBinding deletion processing.
- `spec.description:` is an optional field. It is only present on the kubernetes resource, without MinIO counterpart.
- `spec.group` is the targeted `Group`. It must be in the same namespace than the `GroupBinding`.
- The `spec.tenant` property is set to the default tenant, a tenant named `default` in the `kubmio` namespace. But,
  it can be set explicitly, is there is several `tenant` resources defined in the cluster.
- `spec.user.name` is the targeted `User` name. 
- `spec.user.namespace` us the targeted `User` namespace. By default, the same namespace of the `GroupBinding`. 
  But it can be explicitly set to any namespace.
- The `status.groupMinioName` is set to the name of the Group on the MinIO server, which may be different of the `spec.group.name`.
- The `status.phase` provide information about the current object state. `READY` means the MinIO object counterpart has
  been successfully created.
- The `status.userMinioName` is set to the name of the User on the MinIO server, which may be different of the `spec.user.name`.

Of course, targeted User and Group must belong to the same tenant as the PolicyBinding.

> Note there is no `spec.minioName` as a binding does not generate an object on the MinIO server.
