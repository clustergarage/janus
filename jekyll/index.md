---
layout: home
title: Filesystem Gatekeeper for Kubernetes
hero:
  title: Janus
  subtitle: Filesystem Gatekeeper for Kubernetes
  search: false
categories:
  columns: 3
  title: Browse Topics
  subtitle: Let's not complicate things, just start reading...
section:
  title: Why would I use Janus?
  subtitle: |
    To be able to set permission rules to allow or deny filesystem events on
    specific directories, files, or mountpoints.
cta:
  title: Didn't find an answer to your question?
  subtitle: We'd like to hear your input.
  button_text: Contact Us
  button_url: /contact/
---

## In more words...

In keeping with the "do a single task, really well" mindset, we wanted to make
extra features available for the [Arugs](https://clustergarage.io/argus)
project that didn't entirely fit the specific use case of that app. Breaking it
out into this project made more sense. Leveraging `fanotify`, we based Janus
around the need to set permission rules to allow or deny certain events on
filesystem objects.
