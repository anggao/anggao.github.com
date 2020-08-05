# Setup Kubernetes Cluster with Kind


## Intro
[kind](https://kind.sigs.k8s.io/) is a tool for running local Kubernetes clusters using Docker container “nodes”.
kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI.

## Setup Golang development

1. Install Go tools. Follow [docs](https://golang.org/doc/install#install).
2. Config `$GOROOT` and `$GOPATH`

- `$GOROOT` is for compiler/tools that comes from go installation. This should be configured to the `go` directory that was installed.
- `$GOPATH` is for your own go projects / 3rd party libraries (downloaded with "go get"). This can be a new, empty directory to start with.
  e.g. I have the following:

```
$ echo $GOROOT $GOPATH
/Users/agao/go /Users/agao/goprojects
```

## Install Kind
Follow [docs](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
Since I have Golang development env, I can just do

```
go get sigs.k8s.io/kind
```
This will put kind in `$(go env GOPATH)/bin`

## Create cluster

```
cat kind-config.yaml

kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    image: kindest/node:v1.17.2
  - role: worker
    image: kindest/node:v1.17.2
  - role: worker
    image: kindest/node:v1.17.2
```

```
kind create cluster --config ./kind-config.yaml --name istio-cluster
```

## Testing

```
❯ kubectl get nodes
NAME                          STATUS   ROLES    AGE     VERSION
istio-cluster-control-plane   Ready    master   2m12s   v1.17.2
istio-cluster-worker          Ready    <none>   95s     v1.17.2
istio-cluster-worker2         Ready    <none>   95s     v1.17.2

❯ kubectl cluster-info
Kubernetes master is running at https://127.0.0.1:39936
KubeDNS is running at https://127.0.0.1:39936/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

❯ kind delete cluster --name istio-cluster
Deleting cluster "istio-cluster" ...
```

