
# Overview

Kubmio is a [MinIO Object Store](https://min.io/) resource operator. Aim is to handle MinIO resources, such as:

- Buckets
- Policies
- Objects
- Users
- Groups
- AccessKeys
- ...

Kubmio does not manage installation and configuration of a MinIO cluster or tenant in Kubernetes. This is the role of the [MinIO operator](https://github.com/minio/operator)

## Main features:

- Support all MinIO Objects as Kubernetes Resources.
- Able to handle several MinIO clusters
- Handle most of the resources properties.
- Resources naming decoration, to ensure global uniqueness and allow corporate naming rules enforcing.
- Support for LDAP connected MinIO Storage Server

## Requirement

- Access to a Kubernetes cluster. You can use [KIND](https://sigs.k8s.io/kind) to get a local cluster for testing, or run against a remote cluster.
- Support of admission webkooks
- [cert-manager](https://cert-manager.io/)
- An account with full admin rights.
- A MinIO Server with admin credentials.
