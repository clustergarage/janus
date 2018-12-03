# Janus

This repository fronts the **Janus** application suite. It includes several components that make up the documentation, configuration files linked in the docs, and a set of common examples to get you started.

**tl;dr--** [get started here](https://clustergarage.io/janus/docs/getting-started/)!

## Purpose

The files that build the [documentation](https://clustergarage.io/janus/) are hosted and maintained here. Configuration files are hosted in order to be linked as raw YAML that can be `apply`ed to your running cluster. In addition to those, a suite of common examples that allow you to familiarize yourself with how to set up watchers on common applications in both vanilla Kubernetes as well as OpenShift variants.

## Configuration Files

Located under:
- `configs/janus-k8s.yaml` &mdash; for vanilla Kubernetes clusters
- `configs/janus-k8s-secure.yaml` &mdash; for vanilla (SSL/TLS enabled) Kubernetes clusters
- `configs/janus-openshift.yaml` &mdash; for OpenShift clusters
- `configs/janus-openshift-secure.yaml` &mdash; for (SSL/TLS enabled) OpenShift clusters

Presents the production configuration files to stand it up in your running cluster; the steps to use these are summarized in the [getting started](https://clustergarage.io/janus/docs/getting-started/) section of the documentation.

## Running Examples

Located under:
- `examples/*`

Provides sample configurations to create and monitor JanusWatchers with various applications that will be outlined in the [examples](https://clustergarage.io/janus/docs/examples/) documentation.

## Helm Repository

If using the popular Kubernetes package manager Helm, we maintain and host a Helm repository in this area as well.

Located under:
- `helm/*`

Further information on how to use the Helm variation of **Janus** is also located in the [getting started](https://clustergarage.io/janus/docs/getting-started/) section of the documentation.
