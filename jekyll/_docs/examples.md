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
  https://raw.githubusercontent.com/clustergarage/janus/master/examples/nginx-janus-watch.yaml
```

This is a basic example of monitoring two different paths for a single event.
The watcher spec has a single subject that watches:

```yaml
paths:
- /etc/nginx
- /etc/init.d/nginx
events:
- modify
```

If you were to change any of the files under `/etc/nginx` it would notify on
that `modify` message.

You could also edit this watcher and update the `paths` to include
`/var/log/nginx`. This should update the watcher, that you can then exec into
the container to generate some messages:

```shell
$ kubectl exec -it <nginx-pod-name> -- /bin/bash

root@<nginx-pod-name>:/# echo "test" >> /var/log/nginx/foo.log
```

This will create a new log file and generate a `MODIFY` event that will show up
in the `janusd` logs:

```shell
$ kubectl logs <janusd-pod-name>

MODIFY file '/var/log/nginx/foo.log' (<nginx-pod-name>:<node-name>)
```

Another interesting test to try is to edit the [JanusWatcher
definition]({{site.baseurl}}/docs/januswatcher/#recursively-watching-a-directory)
with `recursive: true` on the subject to receive all events that happen under
subdirectories of the specified paths as well. For example, editing the
`/etc/nginx/conf.d/default.conf` once it is watching recursively would report
messages when it previously would have not.
