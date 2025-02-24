# Policy binding

## Description

This resource will allow to attach a Policy to a User or a Group

For example, to attach the policy 'policy1' to the user 'User1' in de 'default' namespace:

```
cat <<'EOF' | kubectl apply -f -
---
apiVersion: kubmio.com/v1alpha1
kind: PolicyBinding
metadata:
  name: user1-policy1
spec:
  description: Grant user1 access to Policy1
  policy:
    name: policy1
  subject:
    kind: User
    name: user1
EOF
```

- `spac.policy.name` is the K8s name of the targeted Policy.
- `spec.subject.kind` must be set either to `User` or `Group`.
- `spac.subject.name` is the K8s name of the targeted User or Group.

You may display the resulting kubernetes manifest:

```
$ kubectl get policybinding.kubmio.com user1-policy1 -o yaml
apiVersion: kubmio.com/v1alpha1
kind: PolicyBinding
metadata:
  finalizers:
  - kubmio.com/finalizer
  name: user1-policy1
  namespace: default
  ....(Some data removed)
spec:
  description: Grant user1 access to Policy1
  policy:
    kind: Policy
    name: policy1
  subject:
    kind: User
    name: user1
    namespace: default
  tenant:
    name: default
    namespace: kubmio
status:
  isGroup: false
  phase: READY
  policyMinioName: policy1
  subjectMinioNameOrDn: user1
```

- There is a `metadata.finalizers`. This will be used during the PolicyBinding deletion processing.
- `spec.description:` is an optional field. It is only present on the kubernetes resource, without MinIO counterpart.
- `spec.policy.kind: policy` has been added as a default by the Kubmio webhook. This will allow for future extension, 
  with other type of Policy.
- There is no `spec.policy.namespace`, as the Policy MUST be in the same namespace than the binding, for security reason
  (Refer to Policy description)
- `spec.subject.namespace: default` has been added as a default with the current namespace. But any namespace can be set 
  here, to delegate permissions to other party.
- The `spec.tenant` property is set to the default tenant, a tenant named `default` in the `kubmio` namespace. But,
  it can be set explicitly, is there is several `tenant` resources defined in the cluster.
- The `status.phase` provide information about the current object state. `READY` means the MinIO object counterpart has
  been successfully created.
- The `status.policyMinioName` is set to the name of the policy on the MinIO server, which may be different of the `spec.policy.name`.
- The `status.subjectMinioNameOrDn` is set to the name of the user or group on the MinIO server, which may be different of the `spec.subject.name`.

Of course, targeted User and Policy must belong to the same tenant as the PolicyBinding.

> Note there is no `spec.minioName` as a binding does not generate an object on the MinIO server.

