# Steps

## Setup

1. Download [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) and [Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. Once done, [Enable Tech Preview feature for Wasm](https://www.docker.com/blog/docker-wasm-technical-preview/) so that you can run Wasm workloads.
3. Download [k3d](https://k3d.io/v5.6.0/)
4. Download [kube-green](https://kube-green.dev)

## Deployment

1. Create a k3d cluster
```shell
k3d cluster create wasm-cluster --image ghcr.io/deislabs/containerd-wasm-shims/examples/k3d:v0.10.0 -p "8081:80@loadbalancer" --agents 2
```
2. Enable Runtime Classes
```shell
kubectl apply -f https://github.com/deislabs/containerd-wasm-shims/raw/main/deployments/workloads/runtime.yaml
```
3. Deploy the applications
```shell
kubectl apply -f https://github.com/deislabs/containerd-wasm-shims/raw/main/deployments/workloads/workload.yaml
```
