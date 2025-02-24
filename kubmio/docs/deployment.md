
# Deployment

To install Kubmio, just use the corresponding Helm chart:

```
helm -n kubmio upgrade -i --create-namespace kubmio oci://quay.io/kubotal/charts/kubmio --version 0.5.0-snapshot
```

Helm chart default values should be fine for a standard installation.

Three pods should now be running on the `kubmio` namespace:

```
$ kubectl -n kubmio get pods
NAME                                 READY   STATUS    RESTARTS      AGE
kubmio-supervisor-66ff799f45-r8mh8   1/1     Running   9             10h
kubmio-sweeper-5d7f7b798b-xsmjq      1/1     Running   0             10h
kubmio-webhooks-68c4f4cb68-5g6nx     1/1     Running   0             10h
```
