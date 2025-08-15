# K3D Dev Cluster Setup

Setup development k3s Cluster using k3d with or without extra agent(s) + k3d-managed registry and Traefik ingress controller.

- [K3D Dev Cluster Setup](#k3d-dev-cluster-setup)
  - [Terminology](#terminology)
  - [1 Server, 0 Agent, k3d-managed Local Registry with 8080 mapped ingress](#1-server-0-agent-k3d-managed-local-registry-with-8080-mapped-ingress)
  - [1 Server, 2 Agents](#1-server-2-agents)
  - [k3d auto-configured `~/.kube/config`](#k3d-auto-configured-kubeconfig)
  - [Testing Registry](#testing-registry)
  - [Cleanup](#cleanup)

K3d uses `https://hub.docker.com/_/registry` for this registry.

## Terminology

- A server node is defined as a host running the k3s server command,
  with control-plane and datastore components managed by K3s.
- An agent node is defined as a host running the k3s agent command, without any datastore or control-plane components.
- Both servers and agents run the `kubelet`, container runtime, and CNI
  which allow use case of using a single server node as complete cluster.

## 1 Server, 0 Agent, k3d-managed Local Registry with 8080 mapped ingress

Registry DOESN'T need to be pre-created through `k3d registry create`.

```sh
# --api-port is not required
k3d \
cluster create k3s-default \
--api-port 6550 \
-p "8081:80@loadbalancer" \
--registry-create k3s-default-registry
```

## 1 Server, 2 Agents

```sh
# --api-port is not required
k3d \
cluster create k3s-default \
--api-port 6550 \
-p "8081:80@loadbalancer" \
--registry-create k3s-default-registry \
--agents 2
```

## k3d auto-configured `~/.kube/config`

Output of `kubectl config view` after k3d cluster.

You can see it updates `~/.kube/config`
to communicate with exposed k3s CAPI at `6550`

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

## Testing Registry

Check registry container running in host docker:

```sh
docker ps -f name=k3s-default-registry
```

or through IDE plugin that allows seeing running containers in host machine:

![k3d cluster containers](/assets/k3d-default-containers.png)

## Cleanup

```sh
# Clear all
k3d cluster delete --all
# Specific registry
k3d cluster delete k3s-default-registry
```
