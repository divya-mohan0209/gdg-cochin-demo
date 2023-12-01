# Steps

## Setup

1. Download [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) and [Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. Once done, [Enable Tech Preview feature for Wasm](https://www.docker.com/blog/docker-wasm-technical-preview/) so that you can run Wasm workloads.
3. Download [k3d](https://k3d.io/v5.6.0/)
4. Download [Helm](https://helm.sh/docs/intro/install/)
5. Download [yq](https://github.com/mikefarah/yq)
6. Deploy & install [KEPLER](https://sustainable-computing.io) using Helm
7. Download & install [cert-manager](https://cert-manager.io/docs/installation/)
8. Download [kube-green](https://kube-green.dev)

## Deployment of apps

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

## Checkpoint #1

1. Check whether all the applications have been deployed and are running correctly by executing `kubectl get all`. You should see something like this as the output.

```shell
NAME                                READY   STATUS    RESTARTS   AGE
pod/wasm-spin-646cfdc78d-nv84k      1/1     Running   0          12m
pod/wasm-slight-75d7569758-x2cd4    1/1     Running   0          32s
pod/wasm-wws-5f877c6d6f-dcf8w       1/1     Running   0          32s
pod/wasm-lunatic-787dc678f9-zgtpt   1/1     Running   0          32s

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/kubernetes     ClusterIP   10.43.0.1       <none>        443/TCP   65m
service/wasm-slight    ClusterIP   10.43.103.215   <none>        80/TCP    12m
service/wasm-spin      ClusterIP   10.43.61.114    <none>        80/TCP    12m
service/wasm-wws       ClusterIP   10.43.90.121    <none>        80/TCP    12m
service/wasm-lunatic   ClusterIP   10.43.10.48     <none>        80/TCP    12m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/wasm-spin      1/1     1            1           12m
deployment.apps/wasm-slight    1/1     1            1           12m
deployment.apps/wasm-wws       1/1     1            1           12m
deployment.apps/wasm-lunatic   1/1     1            1           12m

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/wasm-spin-646cfdc78d      1         1         1       12m
replicaset.apps/wasm-slight-75d7569758    1         1         1       12m
replicaset.apps/wasm-wws-5f877c6d6f       1         1         1       12m
replicaset.apps/wasm-lunatic-787dc678f9   1         1         1       12m
```

2. You can also check the response from each of the applications by issuing curl commands as below:
  - Spin

  ```shell
  curl -v http://127.0.0.1:8081/spin/hello
  ```
  - Slight

  ```shell
  curl -v http://127.0.0.1:8081/slight/hello
  ```

  - Lunatic

  ```shell
  curl -v http://127.0.0.1:8081/lunatic/hello
  ```

  - Wasm Workers Server

  ```shell
  curl -v http://127.0.0.1:8081/spin/hello
  ```
## Check Energy consumption

- To do this, clone the kube-prometheus project locally

```
git clone --depth 1 https://github.com/prometheus-operator/kube-prometheus
cd kube-prometheus
```

- Next, install the Kepler Grafana Dashboard. We can do this through the Grafana UI and this step is *optional*.

```
KEPLER_EXPORTER_GRAFANA_DASHBOARD_JSON=`curl -fsSL https://raw.githubusercontent.com/sustainable-computing-io/kepler/main/grafana-dashboards/Kepler-Exporter.json | sed '1 ! s/^/         /'`
mkdir -p grafana-dashboards
cat - > ./grafana-dashboards/kepler-exporter-configmap.yaml << EOF
apiVersion: v1
data:
    kepler-exporter.json: |-
        $KEPLER_EXPORTER_GRAFANA_DASHBOARD_JSON
kind: ConfigMap
metadata:
    labels:
        app.kubernetes.io/component: grafana
        app.kubernetes.io/name: grafana
        app.kubernetes.io/part-of: kube-prometheus
        app.kubernetes.io/version: 9.5.3
    name: grafana-dashboard-kepler-exporter
    namespace: monitoring
EOF
```

```shell
yq -i e '.items += [load("./grafana-dashboards/kepler-exporter-configmap.yaml")]' ./manifests/grafana-dashboardDefinitions.yaml
```

```shell
yq -i e '.spec.template.spec.containers.0.volumeMounts += [ {"mountPath": "/grafana-dashboard-definitions/0/kepler-exporter", "name": "grafana-dashboard-kepler-exporter", "readOnly": false} ]' ./manifests/grafana-deployment.yaml
```

```shell
yq -i e '.spec.template.spec.volumes += [ {"configMap": {"name": "grafana-dashboard-kepler-exporter"}, "name": "grafana-dashboard-kepler-exporter"} ]' ./manifests/grafana-deployment.yaml
```

- Once done, apply the objects in the manifest directory.

```shell
kubectl apply --server-side -f manifests/setup
```
```shell
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done
```
```shell
kubectl apply -f manifests/
```

- In order to view our workloads in the browser, we'll use port-forwarding

```shell
kubectl -n monitoring port-forward svc/grafana 3000
```

This should make the grafana dashboard available on: **http://localhost:3000/**

## Experiment

- We're going to *spam* our wws workload with curl requests.

```shell
while sleep 5; do curl http://127.0.0.1:8081/wws/hello; done
```

- On the grafana dasboard,we will visualize what this will do to the workload.

## Resource shutdown

1. Download the sleepInfo.yaml manifest file from [manifests](../manifests)
2. Tailor it to your needs (change the timing or the application being shutdown)
3. Execute the below command

```shell
kubectl apply -f sleepInfo.yaml
```

## Checkpoint #2

1. Execute `kubectl get all`. If you've used the same sleepInfo.yaml and have only altered the time, the output should look something like:

```shell
NAME                             READY   STATUS    RESTARTS   AGE
pod/wasm-spin-646cfdc78d-nv84k   1/1     Running   0          8m53s

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/kubernetes     ClusterIP   10.43.0.1       <none>        443/TCP   61m
service/wasm-slight    ClusterIP   10.43.103.215   <none>        80/TCP    8m53s
service/wasm-spin      ClusterIP   10.43.61.114    <none>        80/TCP    8m53s
service/wasm-wws       ClusterIP   10.43.90.121    <none>        80/TCP    8m53s
service/wasm-lunatic   ClusterIP   10.43.10.48     <none>        80/TCP    8m53s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/wasm-spin      1/1     1            1           8m53s
deployment.apps/wasm-lunatic   0/0     0            0           8m53s
deployment.apps/wasm-wws       0/0     0            0           8m53s
deployment.apps/wasm-slight    0/0     0            0           8m53s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/wasm-spin-646cfdc78d      1         1         1       8m53s
replicaset.apps/wasm-lunatic-787dc678f9   0         0         0       8m53s
replicaset.apps/wasm-slight-75d7569758    0         0         0       8m53s
replicaset.apps/wasm-wws-5f877c6d6f       0         0         0       8m53s
```

2. You can even execute the curl commands individually against the apps that have been shutdown to verify.
