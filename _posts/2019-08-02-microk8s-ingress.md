---
layout: post
title: "microk8s使用ingress对外提供web服务"
categories: microk8s ingress
---

1. 启用 ingress

```bash
$ microk8s.status #查看系统模块
$ microk8s.enable registry ingress
```

2. 创建服务, selector 中是提供服务器 pod 的名称，同时暴露 80 端口出来

```bash
$ cat <<EOF | microk8s.kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: my-web
spec:
  selector:
    app: my-nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
EOF
```

3. 创建 ingress，配置 rule 把流量重定向至指定服务。ingress 允许使用不同厂商[controller 实现](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)

- 多域名

```bash
$ microk8s.kubectl get ingress
$ cat <<EOF | microk8s.kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: demo1.example.com
    http:
      paths:
      - backend:
          serviceName: my-web
          servicePort: 80
  - host: demo2.example.com
    http:
      paths:
      - backend:
          serviceName: my-web2
          servicePort: 80
EOF
```

- 多路径

```bash
$ cat <<EOF | microk8s.kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress-rewrite
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - host: demo3.example.com
    http:
      paths:
      - path: /web(/|$)(.*)
        backend:
          serviceName: my-web
          servicePort: 80
EOF
$ microk8s.kubectl get ingress
$ microk8s.kubectl describe ingress # 查看ingress详情
$ microk8s.kubectl edit ingress #编辑ingress
```

_注意_ nginx 多路径和多域名可以同时存在，可以创建多个 ingress

### 参考资料

https://kubernetes.io/docs/concepts/services-networking/ingress/
https://kubernetes.github.io/ingress-nginx/examples/rewrite/
https://www.regular-expressions.info/refcapture.html
