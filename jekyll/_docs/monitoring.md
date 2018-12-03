---
layout: doc
title: Monitoring Watches
subtitle: Set up monitoring and alerts of watchers
tags: monitoring januswatcher
---

Once you have [JanusWatchers]({{site.baseurl}}/docs/januswatcher/) defined,
you're ready to start monitoring for notify events; perhaps you'll even want to
set up alerts on high priority events. There are generic logfiles included in
both apps, and we provide out-of-the-box metrics handling with
[Prometheus](https://prometheus.io) so you'll be able to receive time-series
data that you can immediately monitor. Ultimately, it will be up to you to use
your logging framework of choice to monitor the way you're used to doing.

## Logfiles

Using the `glog` logging framework that most Kubernetes-related projects also
leverage, you can locate logfiles in both **janusd** and **januscontroller**
containers at `/tmp/janusd.INFO` and `/tmp/januscontroller.INFO`. The
**janusd** logs will show actual `inotify` events that were reported as they
happened, where **januscontroller** log will only include Kubernetes
controller-related log messages such as receiving new watchers, connecting the
controller to daemons via gRPC, and pods being created, deleted, and updated
that a watcher cares about.
