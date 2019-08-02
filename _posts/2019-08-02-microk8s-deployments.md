---
layout: post
title:  "microk8s deployment"
categories: k8s deployment
---

1. 增加一个部署

```bash
# microk8s.kubectl get deployment
# cat <<EOF | microk8s.kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-nginx
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: my-nginx
    spec:
      containers:
      - image: nginx:1.17
        name:my-nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: nginx-persistent-storage
        persistentVolumeClaim:
          claimName: pvc-nginx
EOF
# microk8s.kubectl get deployment
```

2. 删除部署，同时会删除pod

```bash
# microk8s.kubectl delete deployment nginx-deployment
```


### 参考资料
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
