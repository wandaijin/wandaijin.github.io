---
layout: post
title:  "ingress调试"
categories: microk8s ingress
---

1. 定位ingress

```bash
$ microk8s.kubectl get ingress
$ microk8s.kubectl describe ingress # 查看ingress详情
$ microk8s.kubectl get pods -A -o wide
...
nginx-ingress-microk8s-controller-7hssx   1/1     Running   87         246d
...
```
- 查看日志输出
```bash
$ microk8s.kubectl logs -f  nginx-ingress-microk8s-controller-7hssx # 跟踪日志
```

- 登录到pod的shell上，进行命令交互
```bash
$ microk8s.kubectl exec nginx-ingress-microk8s-controller-7hssx -i -t -- bash
```

- 直接在pod上运行命令，并返回结果
```bash
$ microk8s.kubectl exec nginx-ingress-microk8s-controller-7hssx -- date
```

- 查看ingress版本

```bash
$ microk8s.kubectl exec -it nginx-ingress-microk8s-controller-7hssx -- /nginx-ingress-controller --version
-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:    0.24.1
  Build:      git-ce418168f
  Repository: https://github.com/kubernetes/ingress-nginx
-------------------------------------------------------------------------------
```

2. 设置DNS

```bash
$ microk8s enable dns # 启用
$ microk8s kubectl -n kube-system edit configmap/coredns
$ microk8s disable dns # 停用
```

3. 查看某个命名空间下的问题

```bash
$ microk8s.kubectl describe  pods coredns-f7867546d-rg5g7  --namespace=kube-system
```

### 参考资料
https://kubernetes.io/docs/concepts/services-networking/ingress/
https://microk8s.io/docs/addon-dns
