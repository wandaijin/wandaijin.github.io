---
layout: post
title:  "microk8s配置已有证书"
categories: microk8s gogs
---

如果已经获取到证书，需要配置到k8s中，并在ingress中使用，可以按照以下步骤进行操作

1. 根据已有证书生成secret

```bash
$ microk8s.kubectl create secret tls your-tls-secret --key your.key --cert your.pem
$ microk8s.kubectl get secret your-tls-secret -o yaml #check secret
```

2. 在ingress中使用secret

```bash
$ cat <<EOF | microk8s.kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: web-ingress
spec:
  tls:
    - hosts:
      - demo.example.com
      # This assumes acme-crt-secret exists and the SSL
      # certificate contains a CN for demo.example.com
      secretName: your-tls-secret
  rules:
    - host: demo.example.com
      http:
        paths:
        - path:
          backend:
            serviceName: my-web
            servicePort: 80
EOF
```

_扩展:_
_查看实时网速`watch -n 1 "/sbin/ifconfig eth0 | grep bytes"`_

### 参考资料

https://stackoverflow.com/questions/49803845/kubernetes-ingress-tls
