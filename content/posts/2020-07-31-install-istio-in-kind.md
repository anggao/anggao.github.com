---
title: "Istio 1.6.1 Installation in Kind"
date: 2020-07-31T17:52:42+01:00
draft: true
---

## Download Istio

```
curl -L https://istio.io/downloadIstio | sh -
```

## Configure istioctl

To configure the istioctl client tool for your environment,
assume your istio folder is located at `/root/istio-1.6.6`

```
export PATH="$PATH:/root/istio-1.6.6/bin"
```

### istioctl auto-completion

For ZSH users, the istioctl auto-completion file is located in the `tools/_istioctl` directory.
Aadd source the istioctl auto-completion file in your `.zshrc` file as follows:

```
source /root/istio-1.6.6/tools/_istioctl
```
