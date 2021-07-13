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

Install Linkerd:
```bash
LINKERD2_VERSION=stable-2.10.2 curl -sL https://run.linkerd.io/install | sh
linkerd install | kubectl apply -f -
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