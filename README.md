# Emojivoto Kustomization
Kustomization of [a microservice application](https://github.com/BuoyantIO/emojivoto) that allows users to vote for their favorite emoji,
and tracks votes received on a leaderboard.

The application is composed of the following 3 services:

* web: Web frontend and REST API
* emoji: gRPC API for finding and listing emoji
* voting: gRPC API for voting and leaderboard

![](https://i.imgur.com/eyT3gQu.png)

`vote-bot` deployment is responsible for generating some traffic. It votes on emoji randomly as follows:
- It votes for :doughnut: 15% of the time.
- When not voting for :doughnut:, it picks an emoji at random.
### Environment
* A Kubernetes cluster
* Jaeger backend installation
* Linkerd (`stable-2.10.2`) and Linkerd Jaeger plugin installation
* Ingress controller installation with tracing enabled and injected to Linkerd

Create a local cluster with one node and using [K3d](https://github.com/rancher/k3d):
```bash
k3d cluster create mycluster --agents 1 -p "80:80@loadbalancer" -p "8000:30080@agent[0]" --k3s-server-arg "--no-deploy=traefik"
```
We expose host port 80 and forward to port 80 of the service load balancer in our K3d cluster. Also, we expose host port 8000 and forward to node port 30080 of our K3d cluster. We will later use these ports to access web UI and Jaeger UI respectively.

Install Linkerd:
```bash
LINKERD2_VERSION=stable-2.10.2 curl -sL https://run.linkerd.io/install | sh
linkerd install | kubectl apply -f -
```
Install Linkerd visualization plugin:
```bash
linkerd viz install | kubectl apply -f -
```
Check Linkerd installation:
```bash
linkerd check
```
Install Jaeger backend:
```bash
kustomize build jaeger | kubectl apply -f -
```
Install Linkerd Jaeger plugin and configure Linkerd Opencensus collector to send spans to our Jaeger backend:
```bash
linkerd jaeger install --set collector.jaegerAddr='http://jaeger-collector.tracing:14268/api/traces' | kubectl apply -f -
```
Install Traefik, an ingress controller, with tracing enabled and injected to Linkerd:
```bash
kustomize build ingress | kubectl apply -f -
```
### Build with tracing enabled
The following command deploys all services with tracing enabled and injects Linkerd proxies:
```bash
kustomize build emojivoto/tracing | kubectl apply -f -

# or

kubectl kustomize emojivoto/tracing | kubectl apply -f -

# or

kubectl apply -k emojivoto/tracing
```
### Have Fun UI
- Visit emojivoto web at http://localhost.
- Visit Jaeger UI at http://localhost:8000.
- Visit Linkerd dashboard by executing `linkerd viz dashboard`.
### Clean Up
Delete the cluster:
```bash
k3d cluster delete mycluster
```
