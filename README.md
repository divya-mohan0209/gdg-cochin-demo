# GDG Cochin Demo

This repository contains the scripts, manifests, and steps used during the Demo at GDG Cochin.

## What is the aim?

To demonstrate a POC of the Green Cloud Triangle by leveraging OSS cloud native tooling.

## What will be doing?

We will be deploying simple Hello World applications on the k3d cluster.

The applications will be running on Webassembly runtimes such as Spin, Lunatic, Slight, and Wasm Workers Server.

We will take a look at some of the energy metrics per deployed pod using the Kepler project and will be leveraging kube-green to shut down/start up services per schedule.

## What do we need?

- [k3d](https://k3d.io) => Lightweight wrapper to run Rancher's [k3s](https://k3s.io) distribution on Docker
  - [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
  - [Docker Desktop](https://www.docker.com/products/docker-desktop/)
 
- [Kepler](https://sustainable-computing.io) => Kubernetes Efficient Power Level Exporter
  - [Helm](https://helm.sh/docs/intro/install/) => Needed to install the Kepler charts
  - [yq](https://github.com/mikefarah/yq) => Lightweight parser for JSON, YAML etc with syntax just like jq. This tool is needed to install the Kepler Grafana Dashboard.

- [kube-green](https://kube-green.dev) => Kubernetes add-on to automatically schedule shut down resources when you don't need them.
  - [cert-manager](https://cert-manager.io/docs/installation/)
