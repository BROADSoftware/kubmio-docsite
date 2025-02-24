
# Buckets

## Description

Kubmio handle most of the MinIO bucket properties. Here is a sample of a full-featured bucket resource:

```
cat <<'EOF' | kubectl apply -f -
---
apiVersion: kubmio.com/v1alpha1
kind: Bucket
metadata:
  name: fullbucket
  namespace: default
spec:
  minioName: app1-fullbucket
  description: Fully described bucket
  tenant: 
    name: default
    namespace: kubmio
  objectLocking: True
  versioning: True
  quota:
    type: hard
    value: "12MiB"
  retention:
    # One of "NONE", "GOVERNANCE", "COMPLIANCE"
    mode: GOVERNANCE
    validity: 100
    # One of "DAYS" or "YEARS"
    unit: DAYS
  tags:
    key1: value1
    key2: value2
EOF
```

Note than:

- `spec.minioName` is the name of the corresponding MinIO 'Bucket' object. By default, it is same as the Kubernetes
  object name. But it can be different. You can define it explicitly in the object definition, or use the `decoration`
  mechanism described above.
- `description:` is an optional field. It is only present on the kubernetes resource, without MinIO counterpart.
- The `tenant` is explicitly set. Useful if there is several ones defined in the K8s cluster.
- `objectLocking` can only be set on Bucket creation
- If `retention` is enabled, `objectLocking` will also be enabled.
- If `objectLocking` is enabled, `versioning` will also be enabled.

Please, check the MinIO documentation for a descriptions of these properties.

## Bucket deletion

Deleting the bucket resource is not just removing it from the Kubernetes database. Il will also deleted the MinIO bucket.
The process is the following:

- On bucket creation, a [`metadate.finalizer`](https://kubernetes.io/docs/concepts/overview/working-with-objects/finalizers/) is set in the bucket resource.
- When the `delete` command is issued, a flag (`.metadata.deletionTimestamp`) is set on the resource.
- The Kubmio controller try to delete the Bucket on the MinIO server.
- if this deletion is successfully, the controller remove the finalizer.
- Then Kubernetes remove the Bucket resource as part of its Garbage Collection processing.

If the bucket contains some data, the deletion will fail and the `bucket` resource will switch in `DELETION` state, with periodic retry.
The user can then remove the data, and wait for the bucket to be effectively removed.

### Manual cleanup

There may be some cases where one may need to cleanup the bucket resources manually. For example:

- The bucket creation failed because the tenant is miss configured.
- A MinIO server was dis-commissioned without cleaning corresponding resource definition.
- The bucket was deleted on the MinIO server, without deleting the corresponding kubernetes resource.
- ....

In such case, one need to manually remove the finalizer. This may be achieved by patching the resource. For example:

```
kubectl -n mybucket-namespace patch buckets/mybucket --type json --patch='[{ "op": "remove", "path": "/metadata/finalizers"}]'
```

Of course, it is up to the user to ensure the bucket is also deleted on the MinIO server.
