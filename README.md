# Emojivoto Kustomization
Kustomization of [a microservice application](https://github.com/BuoyantIO/emojivoto) that allows users to vote for their favorite emoji,
and tracks votes received on a leaderboard.

The application is composed of the following 3 services:

* web: Web frontend and REST API
* emoji: gRPC API for finding and listing emoji
* voting: gRPC API for voting and leaderboard

![](https://i.imgur.com/tYYDHHH.png)

In addition, `vote-bot` deployment generates some traffic. It votes on emoji randomly as follows:
- It votes for :doughnut: 15% of the time.
- When not voting for :doughnut:, it picks an emoji at random.
### Prerequisite
* A Kubernetes cluster
* Linkerd installation
### Build with distributed tracing enabled
```bash
kustomize build tracing | kubectl apply -f -

# or

kubectl kustomize tracing | kubectl apply -f -

# or

kubectl apply -k tracing
```