---
layout: doc
title: Examples
subtitle: Test Janus with some example deployments
tags: examples
---

#### Topics
{:.no_toc}
* TOC
{:toc}

## Kubernetes

Whether you're running a vanilla Kubernetes cluster with minikube, or in a
cloud-provided one such as GKE, we provide a set of examples to test out
Janus located in the [examples/
folder](https://github.com/clustergarage/janus/tree/master/examples) of the
GitHub repo.

### NGiNX Example

```shell
kubectl run nginx --image=nginx
kubectl apply -f \
  https://raw.githubusercontent.com/clustergarage/janus/master/examples/nginx-janus-guard.yaml
```

This is a basic example of guarding two different paths for all permission
events and making a decision whether to allow or deny them. The guard spec has
a single subject that guards:

```yaml
allow:
- /etc/nginx
deny:
- /etc/init.d/nginx
events:
- all
```

If you were to change any of the files under `/etc/nginx` it would allow for
that. However, if you were to attempt to read the contents of
`/etc/init.d/nginx`, you would receive an `operation not permitted` error.
