---
layout: doc
title: Defining a JanusGuard
subtitle: Allow or deny important filesystem events on your deployments
tags: introduction janusguard
---

Once you have **Janus** installed on your cluster, you are ready to start
setting up guards for your deployments. All possible configurations of the how,
and what, of setting up a JanusGuard on your deployments are described
below.

#### Topics
{:.no_toc}
* TOC
{:toc}

## Required definition

At a bare minimum, the fields you need to provide are the `selector`, which
works just like any other label selector in Kubernetes. `matchLabels` and
`matchExpressions` are both supported, as described in the [labels and
selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels)
documentation. The `subjects` array allows you to define any number of path and
event combination to guard.

For example, you have an important path that should never receive any kind of
access events. You can set a subject as the example below, in order to deny
permission to any `access` inode event at that path location. This includes any
change to the path itself, as well as children inside the path.

```yaml
apiVersion: januscontroller.clustergarage.io/v1alpha1
kind: JanusGuard
metadata:
  name: myguard
spec:
  selector:
    matchLabels:
      app: myapp
  subjects:
  - allow:
    - /path/to/allow
    deny:
    - /path/to/deny
    events:
    - access
```

#### List of all possible `events`

- `access` &mdash; an application wants to read a file or directory, for
  example using `read` or `readdir`
- `open` &mdash; an application wants to open a file or directory
- `all` &mdash; includes all events listed above

#### Limitations

- If guarding a directory that is symlinked, you will need to guard the
  **source** directory, not the destination. A symlink is a special kind of
  file and does not behave exactly like an actual directory. In the case of
  guarding the destination, you would only receive events on that file but not
  on any events on any child objects under it.
