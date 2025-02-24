
# Global Helm configuration

The Helm chart default values should be fine for a standard installation.

Anyway, you may need to modify some values to adjust to your context.
Here is a set of values which will be present for each of the 4 deployments.

```
operator|supervisor|sweeper|webhook:
  replicaCount: 2

  image:
    pullSecrets: [ ]
    repository: quay.io/kubotal/kubmio
    # -- Overrides the image tag whose default is the chart appVersion.
    tag:
    pullPolicy: IfNotPresent

  # Allow to change some logger configuration
  logger:
    mode: json   # json, dev
    level: info  # error, warn, info, debug, trace

  resources:
    limits:
      cpu: 600m
      memory: 1Gi
    requests:
      cpu: 200m
      memory: 512Mi
```

See comment for usage.

Beside this, there is few configurations about kubmio behavior at this level:

```
global:
  # To be used on resource name decoration.
  clusterName: mycluster

  tenantNamespace: ""   # If set, all tenant must be created in this namespace. if "", tenant can be created in any namespace

  defaultTenant:
    enabled: true
    name: default
    namespace: kubmio
```

- `global.clusterName` will be used on resource name decoration. See below
- `global.tenantNamespace` will allow to constraint all `tenant` definition to be defined in one specific namespace.
  If empty, such definition can be in any namespace (Providing of course appropriate RBAC rules has been setup)
- `global.defaultTenant` will be added by the webhook to all resources definition where `spec.tenant` is empty
