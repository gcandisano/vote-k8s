# Example Voting App

A simple distributed voting app that runs across multiple Docker containers (and on Kubernetes).

This repository is based on [dockersamples/example-voting-app](https://github.com/dockersamples/example-voting-app) by Docker, Inc., licensed under the [Apache License 2.0](LICENSE). It is not an official Docker project.

**Changes from the original:** Kubernetes manifests in `k8s/` (Deployments, NGINX Ingress, and a Postgres StatefulSet with a persistent volume).

## Stack

Python (vote), Redis (queue), .NET (worker), Postgres (storage), and Node.js (results).

## Docker Compose

With [Docker Desktop](https://www.docker.com/products/docker-desktop) or [Docker Compose](https://docs.docker.com/compose/install/):

```shell
docker compose up
```

- Vote: [http://localhost:8080](http://localhost:8080)
- Results: [http://localhost:8081](http://localhost:8081)

## Kubernetes

Manifests live in `k8s/`. Resources are created in your current namespace (`default` if you have not changed it).

Typical minikube setup: enable the `ingress` addon, and run `minikube tunnel` if the Ingress needs a LoadBalancer.

```shell
kubectl apply -f k8s/
```

NodePort services:

- Vote: port **30000**
- Results: port **30001**

With Ingress (`ingressClassName: nginx`):

- `vote.my-app.com`
- `result.my-app.com`

Add those hosts to your machine's hosts file, pointing at the Ingress IP (on minikube, `minikube ip` or the tunnel address).

To remove:

```shell
kubectl delete -f k8s/
```

The database uses a **StatefulSet** with `volumeClaimTemplates` (1 Gi, `ReadWriteOnce`), so votes persist if the Postgres pod is recreated.

## Architecture

![Architecture diagram](architecture.excalidraw.png)

- A front-end web app in [Python](/vote) which lets you vote between two options
- [Redis](https://hub.docker.com/_/redis/) which collects new votes
- A [.NET](/worker/) worker which consumes votes and stores them in…
- [Postgres](https://hub.docker.com/_/postgres/) backed by a persistent volume
- A [Node.js](/result) web app which shows the results of the voting in real time

## Notes

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to deal with them in Docker and Kubernetes at a basic level.

## License

Copyright 2016 Docker, Inc.

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE).
