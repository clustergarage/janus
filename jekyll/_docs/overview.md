---
layout: doc
title: Overview
subtitle: Deep dive into Janus and its architecture
tags: introduction
redirect_from: /docs
---

**Janus** is a set of custom Kubernetes resources that acts as a gatekeeper for
filesystem permission events on specified paths. It provides a rich set of
configurations to run alongside your existing Kubernetes deployments to ensure
that outside processes can or cannot open or access these paths.

In deploying **Janus** into your cluster, a couple of components will be
initialized. First, the controller listens directly for Kubernetes events and
is responsible for syncing cluster state with the daemons running on each node
to make sure specified pods that need to be guarded, are. Second, the daemon is
run on each node to ensure that it can monitor running pods on the same node.
It is charged with marking inodes via `fanotify` to allow you to set guards for
certain filesystem events on the desired paths inside the pod.

We will provide some common patterns using sets of popular tools further in
this documentation.

## Architecture

{% mermaid %}
graph RL
    A((procfs)) -->|fanotify| B[janusnotify]
    subgraph daemon
    C{janusd} --> B
    end
    D[janus-controller] ==>|gRPC| C
    C .-> D
    subgraph controller
    E(K8s listers<br />K8s informers) --> D
    F(JanusGuard CRDs) --> D
    end
{% endmermaid %}

### Custom Resource Definition

In order to describe the set of pods you want to create a guard for, what
allow and deny paths you are interested in guarding, and for what events,
you'll soon find yourself creating a **JanusGuard**
`CustomResourceDefinition`. That is, a Kubernetes custom type that will be
configured in your cluster, that gives a specification to define a guard type
with all the fields and flags the controller will understand, configure, and
pass along to the daemon so it can guard appropriately in the fashion you
describe.

### Controller

**janus-controller** is the glue between the Kubernetes lifecycle and the
daemons listening for events and sending permission responses on the
filesystems. It responds to events such as pod listers add, update, and delete,
making sure that the proper state of the daemon that is running on the same
node as that pod has the guard either started or not on that daemon.

This controller operates in an idempotent fashion, in such cases that the
controller pod running is terminated and a new one spins up, it immediately
receives all the events upon startup that it would have received anyway over
time; this way we can reconstruct the proper state immediately. When a daemon
is not gracefully terminated, upon startup of a new daemon pod, we reconcile
that pod's current state (nothing) with what we expect it should be, and
all addition calls should be immediately fired; same as the previous case but
in reverse.

The other set of types the controller listens for is the **JanusGuard**
`CustomResourceDefinition` described above. This is the definition that you
will create to define what selector, paths, events, and optional flags to
create the notion of a filesystem gatekeeper.


### Daemon

The **janusd** daemon will be deployed on all cluster nodes via a `DaemonSet`.
It starts a gRPC server to communicate back-and-forth with the controller,
described above. When it receives a message from the controller to create a
guard, it takes the desired configuration and spawns a **janusnotify** process
with a custom set of rules. For example, if one wishes to guard the entire
mountpoint that a directory or file resides on, this can be configured with an
optional flag.

{% mermaid %}
sequenceDiagram
    participant A as janusd
    participant B as janusnotify
    participant C as procfs
    A->>+B: Creates child processes
    B->>C: Creates fanotify marks
    Note over B,C: Polling for fanotify events...
    loop fanotify event
      C->>B: Write permission response
      opt exit signal
          B-->C: Stop polling and exit
      end
    end
{% endmermaid %}

Once the **janusnotify** process receives an `fanotify` message that the user
configured to guard, it writes a response to allow or deny that event, based on
how you configure your guard paths.

Upon receiving an additional create message, if the guard already exists, it
will handle it in an idempotent way, and simply update with the current
configuration. When receiving a delete message, if the guard exists, it will
shut down the processes involved and remove this guard from the daemon's
current state.

There is an additional state check called periodically by the controller that
reconciles what the daemon knows its state is with what the controller expects
the current state to be. Any additional creation and deletion methods may be
called due to any differences. This also allows for handling non-graceful
terminations of either daemon or controller pods, thus the proper state can
always be recreated in these cases.
