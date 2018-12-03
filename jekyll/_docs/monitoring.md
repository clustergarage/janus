---
layout: doc
title: Monitoring Guards
subtitle: Set up monitoring of guards
tags: monitoring janusguard
---

Once you have [JanusGuards]({{site.baseurl}}/docs/janusguard/) defined, you're
ready to start guarding filesystem events. There are generic logfiles included
in both apps.

## Logfiles

Using the `glog` logging framework that most Kubernetes-related projects also
leverage, you can locate logfiles in both **janusd** and **januscontroller**
containers at `/tmp/janusd.INFO` and `/tmp/januscontroller.INFO`. The
**janusd** logs will show actual `fanotify` events that were granted or denied
access to as they happened, where **januscontroller** log will only include
Kubernetes controller-related log messages such as receiving new guards,
connecting the controller to daemons via gRPC, and pods being created, deleted,
and updated that a guard cares about.
