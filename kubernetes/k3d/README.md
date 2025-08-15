# K3D Local Cluster Setup Cheat Sheet

Setup local k3s Cluster using k3d with or without extra agent(s) +
k3d-managed registry and ingress controller.

- [K3D Local Cluster Setup Cheat Sheet](#k3d-local-cluster-setup-cheat-sheet)
  - [Terminology](#terminology)
  - [1 Server, 0 Agent, k3d-managed Local Registry, 8080 ingress Example](#1-server-0-agent-k3d-managed-local-registry-8080-ingress-example)
  - [1 Server, 2 Agents](#1-server-2-agents)
  - [Checking exposed ports](#checking-exposed-ports)
  - [k3d auto-configured `~/.kube/config`](#k3d-auto-configured-kubeconfig)
  - [Pushing to Local Registry](#pushing-to-local-registry)
  - [Delete image registry](#delete-image-registry)

K3d uses [`https://hub.docker.com/_/registry`](https://hub.docker.com/_/registry)
for local image registry. K3d uses `Traefik` for ingress controller.

## Terminology

- A server node is defined as a host running the k3s server command,
  with control-plane and datastore components managed by K3s.
- An agent node is defined as a host running the k3s agent command,
  without any datastore or control-plane components.
- Both servers and agents run the `kubelet`, container runtime, and CNI
  which allow use case of using a single server node as complete cluster.

## 1 Server, 0 Agent, k3d-managed Local Registry, 8080 ingress Example

Registry DOESN'T need to be pre-created through `k3d registry create`.

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
- `8081` mapped to additional rules specified in command above
- `5555` for image registry (this one not behind Ingress controller)

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

## Pushing to Local Registry

Local registry setup using command above uses `5555` as exposed port, hence to
push into this local registry, you need to tag image with something like
`localhost:5555/image-name:v1` and push it accordingly.

```
docker pull nginx:latest
docker tag nginx:latest localhost:5555/nginx:latest
docker push localhost:5555/nginx:latest
```

## Delete image registry

https://k3d.io/v4.4.8/usage/commands/k3d_registry_delete/

```sh
# Clear all
k3d cluster delete --all
# Specific registry
k3d cluster delete k3s-default-registry
```
