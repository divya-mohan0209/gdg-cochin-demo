# GDG Cochin Demo

This repository contains the scripts, manifests, and steps used during the Demo at GDG Cochin.

## What is the aim?

To demonstrate a POC of the Green Cloud Triangle by leveraging OSS cloud native tooling.

## What will be doing?

We will be deploying simple Hello World applications on the k3d cluster.

The applications will be running on Webassembly runtimes such as Spin, Lunatic, Slight, and Wasm Workers Server.

We will be leveraging kube-green to shut down/start up services per schedule.

## What do we need?

- [k3d](https://k3d.io) => Lightweight wrapper to run Rancher's [k3s](https://k3s.io) distribution on Docker
  - [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
  - [Docker Desktop](https://www.docker.com/products/docker-desktop/)

- [Spin by Fermyon](https://developer.fermyon.com/spin/v2/index) => Framework for building & running event-driven microservice applications with WebAssembly components

- [kube-green](https://kube-green.dev) => Kubernetes add-on to automatically schedule shut down resources when you don't need them.

