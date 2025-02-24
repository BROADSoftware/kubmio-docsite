# Group

## Description

A minio `Group` is just a set of `Users`.

A set of attached policies (See `PolicyBinding` above) define what all members of the group can do on the cluster.

A `Group` is also a `kubmio` resource, so creating a group is as simple as:

```
cat <<'EOF' | kubectl apply -f -
---
apiVersion: kubmio.com/v1alpha1
kind: Group
metadata:
    name: group1
spec:
  description: A first group
EOF
```

You can list all created groups:

```
$ kubectl get --all-namespaces groups.kubmio.com
NAMESPACE   NAME     TENANT NS   TENANT    MINIO NAME   ENABLED   STATUS   DESCRIPTION
default     group1   kubmio      default   group1       true      READY    A first group
```

An alias is also provided:

```
$ kubectl get --all-namespaces mgroups
NAMESPACE   NAME     TENANT NS   TENANT    MINIO NAME   ENABLED   STATUS   DESCRIPTION
default     group1   kubmio      default   group1       true      READY    A first group
```

And, to have a detailed look on the created `Group`:

```
$ kubectl -n default get  groups.kubmio.com group1 -o yaml
apiVersion: kubmio.com/v1alpha1
kind: Group
metadata:
  annotations:
  finalizers:
  - kubmio.com/finalizer
  name: group1
  namespace: default
  ..... (Some data removed)
spec:
  description: A first group
  enabled: true
  minioName: group1
  tenant:
    name: default
    namespace: kubmio
status:
  phase: READY
```

- There is a `metadata.finalizers`. This will be used during the Group deletion processing.
- `spec.minioName` is the name of the corresponding MinIO 'Group' object. By default, it is same as the Kubernetes.
  object name. But it can be different. You can define it explicitly in the object definition, or use the `decoration`
  mechanism described previously.
- `spec.description:` is an optional field. It is only present on the kubernetes resource, without MinIO counterpart.
- The `spec.tenant` property is set to the default tenant, a tenant named `default` in the `kubmio` namespace. But,
  it can be set explicitly, is there is several `tenant` resources defined in the cluster.
- The `status.phase` provide information about the current object state. `READY` means the MinIO object counterpart has
  been successfully created.

