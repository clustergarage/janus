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

## Guarding only directories

An extra flag can be provided to `fanotify` that ensures the guarded path is
specifically marked on a directory type.

In the case of the example below, it would attempt to guard `file.ext` as a
directory; if this is not an actual directory it will ignore it from any
listeners and will not receive any events on that file.

```yaml
subjects:
- allow:
  - /path/to/guard
  - /file.ext
  events:
  - modify
  onlyDir: true
```

## Custom tagging

Custom tags can be added per-subject to allow you to later query specific
portions of your guards. These are defined in a YAML-style map and at logging
time will be condensed into a comma-separated list of `key=value` pairs. In the
case of the below example, when logging an event, the tags will should up as:
`foo=bar,lorem=ipsum` that you can later use your favorite logging aggregator
and query language to target this specific subject.

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
    - /path/to/guard
    events:
    - modify
    tags:
      foo: bar
      lorem: ipsum
```

## Custom logging format

If you want to specify your own logging format when a guard is notified of an
update, you can do so with the `logFormat` option. This takes a format string
with specifiers that we will later interpolate with real values.

```yaml
apiVersion: januscontroller.clustergarage.io/v1alpha1
kind: JanusGuard
metadata:
  name: myguard
spec:
  selector:
    matchLabels:
      app: myapp
  logFormat: "event = {event}; path = {path}; file = {file}; response = {allow}"
  subjects:
  - allow:
    - /path/to/guard
    events:
    - modify
```

The default log format is `<{response}> {event} {ftype} '{path}' ({pod}:{node}) {tags}`.

> Example output using the default format:
`<ALLOW> ACCESS_PERM file '/path/to/file.ext' (foo-1-pod:barnode) foo=bar`

#### List of all possible `logFormat` specifiers

- `pod` &mdash; name of the pod
- `node` &mdash; name of the node
- `response` &mdash; evaluates to "allow" or "deny"
- `event` &mdash; fanotify event that was observed
- `path` &mdash; name of the directory+file path
- `ftype` &mdash; evaluates to "file" or "directory"
- `tags` &mdash; list of custom tags in key=value comma-separated list

