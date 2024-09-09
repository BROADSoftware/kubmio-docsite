
# Tenant configuration

Most of the behavior of Kubmio is configurable per tenant.

## Namespace authorization

A Bucket is a namespaced resource. As such, who is able to create/manage buckets resources can be controlled using Kubernetes RBAC system.

Kubmio add a finest level of validation per MinIO tenant. Each tenant description accept a property `allowedNamespaces`.
Only Kubmio resource manifest declared in one of the listed namespace will be allowed for this tenant.

For example:

```
cat <<'EOF' | kubectl apply -f -
---
apiVersion: kubmio.com/v1alpha1
kind: Tenant
metadata:
  name: default
  namespace: kubmio
spec:
  endpoint: minio.myminioserver.myinfra.com
  credentials:
    secretName: minio-root
  ssl: false
  watchedNamespaces:
    - kubmio
    - app1 
EOF
```

All Kubmio objects creation referring to this tenant, but not in `kubmio` or  `app1` namespace will be rejected.

Of course, if this list is null or empty, all namespaces are allowed.

## Policy action validation

As explained in the [Policy](#policy-action-validation) chapter, the set of valid policy S3 action can be defined by tenant.

By default, a list of all possible actions is injected by the webhook in each tenant definition, if the
`spec.validActions` property is unset:

```
spec:
  ......
  validActions:
  - s3:*
  - s3:CopyObject
  - s3:DeleteObject
  - s3:DeleteObjects
  - s3:DeleteObjectTagging
  - s3:GetObject
  - s3:GetObjectAttributes
  ......
```

This list can be replaced by a more limited one on tenant creation.

All action validation can also be disabled by defining an empty list:

```
spec:
  ......
  validActions: []
```

Note also the default list is configurable during initial Helm installation, by setting the property
`webhooks.defaultValidActions` in the `values` file.

## Other tenant configuration

Here is a short description of other tenant resources properties:

- `nameDecorator`: [As previously described.](#name-decoration)
- `alias`: Value for the `TenantAlias` variable in the data model for name decoration.
- `decommissioned`: If set to `true` this tenant will not accept anymore new resource creation. But managing existing
  one will still be possible.
- `disabled`: If set to `true`, this tenant will be fully disabled.
- `region`: This MinIO region. `us-east-1` by default

## Several tenants on same tenant.

This must be read: 'Several Kubmio tenant definitions pointing on the same physical Minio Tenant'.

This is allowed by Kubmio configuration and could be of great help. For example, let say you have several independent
project, with a development team for each project. But they share a common MinIO server.

You can create a `tenant` for each project, with a name decorator including some unique project identifier, and with a
set of `watchedNamespaces` related to each project.

Doing this way, you have a complete isolation between all the different project.
