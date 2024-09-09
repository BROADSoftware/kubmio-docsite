# Architecture

`kubmio` is made of 3 static deployments and a dynamic one.

- The dynamic one is `operator`, which is in charge of handling all resource for a `tenant`.
  There will be one `operator` deployment per `tenant`.
- `supervisor` is the controller in charge of managing the `tenants` and their associated deployment.
- `sweeper` is a controller in charge of several cleanup task, such a handling 'orphan' resources (Resources without Tenant).
- `webhook` is a webhook handler, in charge if validating kubmio resources and setting default values.

