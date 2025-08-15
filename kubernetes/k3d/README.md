# K3D Local Cluster Cheat Sheet

Setup local k3s cluster using k3d with or without extra agent(s) +
k3d-managed registry and ingress controller.

- [K3D Local Cluster Cheat Sheet](#k3d-local-cluster-cheat-sheet)
  - [Terminologies \& components](#terminologies--components)
  - [1 Server, 0 Agent, k3d-managed local registry, 8080 ingress Example](#1-server-0-agent-k3d-managed-local-registry-8080-ingress-example)
  - [1 Server, 2 Agents](#1-server-2-agents)
  - [Checking exposed ports](#checking-exposed-ports)
  - [k3d auto-configured `~/.kube/config`](#k3d-auto-configured-kubeconfig)
  - [Pushing and pulling images from local registry](#pushing-and-pulling-images-from-local-registry)
  - [Adding additional ingress port mapping](#adding-additional-ingress-port-mapping)
  - [Removing image registry (containers)](#removing-image-registry-containers)

## Terminologies & components

- A server node is defined as a host running the k3s server command,
  with control-plane and datastore components managed by K3s.
- An agent node is defined as a host running the k3s agent command,
  without any datastore or control-plane components.
- Both servers and agents run the `kubelet`, container runtime, and CNI
  which allow use case of using a single server node as complete cluster.

---

- k3d deploys [`https://hub.docker.com/_/registry`](https://hub.docker.com/_/registry)
  for image registry
- k3d uses `Traefik` as [ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/).

## 1 Server, 0 Agent, k3d-managed local registry, 8080 ingress Example

Command below doesn't require Registry to be pre-created using `k3d registry create`.

```sh
# --api-port is not required, but useful for demo below
k3d \
cluster create k3s-default \
--api-port 6550 \
-p "8081:80@loadbalancer" \
--registry-create k3s-default-registry:0.0.0.0:5555
```

## 1 Server, 2 Agents

```sh
k3d \
cluster create k3s-default \
--api-port 6550 \
-p "8081:80@loadbalancer" \
--registry-create k3s-default-registry:0.0.0.0:5555 \
--agents 2
```

## Checking exposed ports

k3d placed `Traefik` for ingress controller in container called `serverlb`.
This `serverlb` is meant to sit between host and cluster.

![K3d ports](/assets/k3d-ports.png)

If you create cluster using command above, you can check exposed ports
by `serverlb` and `registry` container as such:

- `6550` exposes Kubernetes API to host
- `8081` exposes additional ports for ingress, as shown above
- `5555` exposes image registry interface (this one not behind Ingress controller)

## k3d auto-configured `~/.kube/config`

Output of `kubectl config view` after creating new k3d cluster.

```yaml
apiVersion: v1
clusters:
  - cluster:
      certificate-authority-data: DATA+OMITTED
      server: https://0.0.0.0:6550
    name: k3d-k3s-default
contexts:
  - context:
      cluster: k3d-k3s-default
      user: admin@k3d-k3s-default
    name: k3d-k3s-default
current-context: k3d-k3s-default
kind: Config
preferences: {}
users:
  - name: admin@k3d-k3s-default
    user:
      client-certificate-data: DATA+OMITTED
      client-key-data: DATA+OMITTED
```

## Pushing and pulling images from local registry

Local registry setup using command above uses `5555` as exposed port, hence to
push into this local registry FROM THE LOCAL HOST MACHINE, you need to tag image
with something like `localhost:5555/image-name:v1` and push it accordingly.

Example:

```
docker build . -t localhost:5555/example-repo-name/example-image-name:latest
docker push localhost:5555/example-repo-name/example-image-name:latest
```

When authoring the `Deployment` manifest, simply use the `registry` container
name and its designated internal port (e.g `5000`), example:
`image: k3s-default-registry:5000/image-name`.

When creating a cluster with `--registry-create`, k3d setups hostname resolution
when Pod specifies `k3s-default-registry` to be resolved appropriately in order
to use local image registry to create a container.

## Adding additional ingress port mapping

Use command such [`k3d cluster edit --port-add 5432:5432@loadbalancer`](https://k3d.io/v5.8.3/usage/commands/k3d_cluster_edit/)

## Removing image registry (containers)

https://k3d.io/v4.4.8/usage/commands/k3d_registry_delete/

```sh
# Clear all
k3d cluster delete --all
# Specific registry
k3d cluster delete k3s-default-registry
```
