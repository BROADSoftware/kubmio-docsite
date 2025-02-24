# Policy

## Description

Kubmio will also allow to define MinIO policy as Kubernetes resources:

For example, to provide full access to the bucket we created previously:

```
cat <<'EOF' | kubectl apply -f -
---
apiVersion: kubmio.com/v1alpha1
kind: Policy
metadata:
  name: policy1
spec:
  description: "Grant access to bucket1"
  statements:
    - effect: Allow
      actions:
        - "*"
      resources:
        - bucket: bucket1
EOF
```

The way you define a `Policy` with kubmio is similar, but not strictly identical to how you define Policy directly in MinIO.

You may display the resulting kubernetes manifest:

```
$ kubectl get policy.kubmio.com policy1 -o yaml
apiVersion: kubmio.com/v1alpha1
kind: Policy
metadata:
  finalizers:
  - kubmio.com/finalizer
  name: policy1
  namespace: default
  ....(Some data removed)
spec:
  description: Grant access to bucket1
  minioName: policy1
  statements:
  - actions:
    - s3:*
    effect: Allow
    resources:
    - bucket: bucket1
      paths:
      - /*
  tenant:
    name: default
    namespace: kubmio
status:
  minioPolicy:
    Statement:
    - Action:
      - s3:*
      Effect: Allow
      Resource:
      - arn:aws:s3:::bucket1
      - arn:aws:s3:::bucket1/*
    Version: "2012-10-17"
  phase: READY
```

- There is a `metadata.finalizers`. This will be used during the Policy deletion processing.
- `spec.description:` is an optional field. It is only present on the kubernetes resource, without MinIO counterpart.
- `spec.minioName` is the name of the corresponding MinIO 'Policy' object. By default, it is same as the Kubernetes
  object name. But it can be different. You can define it explicitly in the object definition, or use the `decoration`
  mechanism described previously.
- The `spec.tenant` property is set to the default tenant, a tenant named `default` in the `kubmio` namespace. But,
  it can be set explicitly, is there is several `tenant` resources defined in the cluster.
- As a Minio Policy, a Kubmio policy is made of several `spec.statements`. With the following fields:
    - actions` is a list of S3 actions, as defined in the [MinIO documentation](https://min.io/docs/minio/linux/administration/identity-access-management/policy-based-access-control.html#id5).
      The 's3:' prefix is added by the Kubmio webhook, if not present.
    - `effect` is `Allow` or `Deny`. See the MinIO documentation.
    - `resources` is a list of targeted MinIO buckets, with a path associated. If no path was provided, the Kubmio webhook
      will add one with `/*`.
- `status.minioPolicy` is the resulting MinIO policy, as it is declared in MinIO.
- The `status.phase` provide information about the current object state. `READY` means the MinIO object counterpart has
  been successfully created.

Of course, the referenced Buckets must be managed by the same Tenant than the Policy

An obvious question at this point is 'Why don't declare directly the MinIO policy in the Kubernetes manifest ?'.
There is at least two reasons for this:

## Name decoration

By using this feature, the name of the Kubernetes Bucket resource and the name of the Minio Bucket
will be different. Beside this, a user can explicitly define the MinIO name of the resource.

As we want this renaming mechanism to be transparent for the user, all the reference to object (bucket, policy,
accessKey, group) are made by the k8s resource name, not the Minio name which may be decorated.

So, Kubmio will have to handle transparently this renaming when creating the MinIO policy.

## Bucket access control.

A usual with Kubernetes, the authorization perimeter is the namespace. So, a namespace owner wants to be sure to
control who is accessing its buckets and how (RO, RW, ...)

So, we can't afford to directly manage policies as resource in an open way (providing json as free form), as this
will allow any user to create a policy to access any buckets.

The security is ensured by the following rules:
- A policy is namespaced and can only reference buckets in the same namespace
- PolicyBinding (Described later) can only bind policy from its own namespace.
- PolicyBinging.subject (Who will be able to use this Policy) can reference a User or a Group in another namespace.
  This is how the bucket/policy owner delegate access to other namespace.

## Dynamic bucket handling

To enforce the rule 'A Policy can only reference a bucket in the same namespace', we can't directly use the MinIO bucket
name pattern mechanism. But Kubmio support its own system. For example, if you define a policy with
`spec.statements.resources.bucket: bucket-*`, this will apply to all buckets which name begin with `bucket-`

Let say there is one bucket named `bucket-a`. If you create a policy *in the same namespace* with the following
resources definition:

```
  resources:
  - bucket: bucket-*
    paths:
    - /mydata
```

The resulting MinIO policy Resource part will be:

```
  Resource:
  - arn:aws:s3:::bucket-a
  - arn:aws:s3:::bucket-a/mydata/*
```

Now, let's say you create a new bucket `bucket-b` still in the same namespace. The resulting Minio policy Resource part
will be:

```
  Resource:
  - arn:aws:s3:::bucket-a
  - arn:aws:s3:::bucket-a/mydata/*
  - arn:aws:s3:::bucket-b
  - arn:aws:s3:::bucket-b/mydata/*
```

The bucket creation was automatically detected and the policy adjusted accordingly.

> Technically, a MinIO policy can't be updated. So, Kubmio will delete/recreate the policy on the MinIO server.

If you now delete the bucket `bucket-a`, the MinIO Policy will again be automatically adjusted accordingly:

```
  Resource:
  - arn:aws:s3:::bucket-b
  - arn:aws:s3:::bucket-b/mydata/*
```

If you now create a bucket `bucket-c` in ANOTHER NAMESPACE, it will be ignored for our policy.

A specific case: If there is no bucket matching the pattern, MinIO will not accept an empty Resource array. So Kubmio
will generate a policy with:

```
  "Resource":
     - "arn:aws:s3:::dummy_bucket"
```

> This will be also the case if you define a policy without pattern matching, but the referenced bucket does not exist.

## S3 Action validation

Kubmio check each action set in the policy against a list of valid action, defined by tenant. This to:

- Trap typos or other errors as soon as possible. Before Policy creation, thanks to the webhook.
- As this list is defined by tenant, the set of actions can be limited by the administrator during tenant creation.

By default, this list is fulfilled with all possible action during Tenant creation.

Refer to [Tenant configuration](#tenant-configuration) for more information.
