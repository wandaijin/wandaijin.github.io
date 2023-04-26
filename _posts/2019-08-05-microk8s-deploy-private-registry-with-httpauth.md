---
layout: post
title: "microk8s部署私有registry，并使用htpasswd设置http认证"
categories: k8s microk8s
---

1. 创建本地永久存储卷

```bash
# mkdir -p /data/k8s-pv/pv001
# cat <<EOF | microk8s.kubectl apply -f -
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv001
  labels:
    type: local
spec:
  storageClassName: local-storage
  capacity:
    storage: 30Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "/data/k8s-pv/pv001"
EOF
#cat <<EOF | microk8s.kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-registry
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 30Gi
EOF
```

2. 使用 htpasswd 创建密码

```bash
# apt install -y apache2-utils
# htpasswd -Bbn myusername password20GAt > htpasswd.txt
# microk8s.kubectl create secret generic registry-auth-pass --from-file=./htpasswd.txt
# microk8s.kubectl describe secret registry-auth-pass
#
 registry-auth-pass -o yaml #查看密码
```

3. 创建部署任务，使用 docker 镜像*registry:2*, 增加*env*设置启用 http 认证

```bash
# cat <<EOF | microk8s.kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: private-registry
spec:
  replicas: 1
  selector:
    matchLabels:
      app: registry
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: registry
    spec:
      containers:
      - env:
        - name: REGISTRY_AUTH
          value: htpasswd
        - name: REGISTRY_AUTH_HTPASSWD_REALM
          value: Registry Realm
        - name: REGISTRY_AUTH_HTPASSWD_PATH
          value: /auth/htpasswd.txt
      - image: registry:2
        name: private-registry
        ports:
        - containerPort: 80
        volumeMounts:
        - name: registry-persistent-storage
          mountPath: /var/lib/registry/
        - name: htpasswd-volume
          readOnly: true
          mountPath: /auth/
      volumes:
      - name: registry-persistent-storage
        persistentVolumeClaim:
          claimName: pvc-registry
      - name: htpasswd-volume
        secret:
          secretName: registry-auth-pass
EOF
```

4. 创建服务，配置 ingress

```bash
#cat <<EOF | microk8s.kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: private-registry
spec:
  selector:
    app: registry
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
EOF
# cat <<EOF | microk8s.kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: registry.example.com
      http:
        paths:
        - backend:
            serviceName: private-registry
            servicePort: 80
          path: /
EOF
```

5. 使用配置。在 docker 客户端配置密码

```
# docker login registry.example.com
Username:
Password:
```

6. 配置 k8s 使用搭建的 private registry.

```bash
# microk8s.kubectl create secret docker-registry private-registry-auth --docker-server=registry.wandaijin.com --docker-username=myusername --docker-password=password20GAt
# cat <<EOF | microk8s.kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: foo
spec:
  containers:
    - name: foo
      image: registry.example.com/app:v1
  imagePullSecrets:
    - name: private-registry-auth
EOF
```

### 参考资料

https://docs.docker.com/registry/deploying/
https://kubernetes.io/docs/concepts/configuration/secret/
https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod
