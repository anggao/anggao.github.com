---
title: "Setup Kubernetes Cluster with Kind"
date: 2020-02-01T22:39:50+01:00
tags:
  - k8s
---

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
kind create cluster --config ./kind-config.yaml --name test-cluster
```

