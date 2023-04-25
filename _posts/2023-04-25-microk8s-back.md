---
layout: post
title: "microk8s back up yml"
categories: microk8s
---

# 备份 deployment

```bash
$ microk8s.kubectl get deployment
$ microk8s.kubectl get deployment <name> -o yaml
```

# 备份 service

```bash
$ microk8s.kubectl get service
$ microk8s.kubectl get service <name> -o yaml
```

# 备份 ingress

```bash
$ microk8s.kubectl get ingress
$ microk8s.kubectl get ingress <name> -o yaml
```

# 备份 pv `--all-namespaces`

```bash
$ microk8s.kubectl get pv
$ microk8s.kubectl get pv --all-namespaces -o yaml
```

# 备份 pvc `--all-namespaces`

```bash
$ microk8s.kubectl get pvc
$ microk8s.kubectl get pvc --all-namespaces -o yaml
```

# 备份 secret `--all-namespaces`

```bash
$ microk8s.kubectl get secret
$ microk8s.kubectl get secret --all-namespaces -o yaml
```
