---
layout: post
title:  "microk8s使用docker安装配置npm私有库 verdaccio"
categories: microk8s verdaccio
---

# 安装

1. 创建存储卷(本地存储)，以便挂载verdaccio数据目录(按容量大小自动绑定pvc到pv)

```bash
$ mkdir -p /data/k8s-pv/pv003
$ cat <<EOF | microk8s.kubectl apply -f -
kind: PersistentVolume
apiVersion: v1
metadata:
 name: pv003
 labels:
   type: local
spec:
 storageClassName: local-storage
 capacity:
   storage: 5Gi
 accessModes:
   - ReadWriteOnce
 persistentVolumeReclaimPolicy: Retain
 hostPath:
   path: "/data/k8s-pv/pv003"
EOF
$ cat <<EOF | microk8s.kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: pvc-verdaccio
spec:
 storageClassName: local-storage
 accessModes:
   - ReadWriteOnce
 resources:
   requests:
     storage: 5Gi
EOF
```

2. 创建目录和文件，[链接](https://github.com/verdaccio/docker-examples/tree/master/docker-local-storage-volume)

*注* 1. 如果使用路径方式部署需要在config.yaml文件中定义 `url_prefix`项
*注* 2. 修改`storage: /verdaccio/data`, 使用默认的`storage: /verdaccio/storage`，数据不会保存到磁盘

```bash
$ cd /data/k8s-pv/pv003
$ mkdir conf
$ cd conf/
$ vi config.yaml
$ touch htpasswd
$ sudo chown -R 10001:65533 /data/k8s-pv/pv003
```

2. 创建verdaccio应用

```bash
$ cat <<EOF | microk8s.kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
 name: verdaccio-deployment
spec:
 replicas: 1
 selector:
   matchLabels:
     app: verdaccio
 strategy:
   type: Recreate
 template:
   metadata:
     labels:
       app: verdaccio
   spec:
     containers:
     - image: verdaccio/verdaccio:4.2.1
       name: verdaccio
       ports:
       - containerPort: 4873
       volumeMounts:
       - name: verdaccio-persistent-storage
         mountPath: /verdaccio
     volumes:
     - name: verdaccio-persistent-storage
       persistentVolumeClaim:
         claimName: pvc-verdaccio
EOF
```

3. 创建服务暴露http(4873)端口

```bash
$ cat <<EOF | microk8s.kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: verdaccio-service
spec:
  selector:
    app: verdaccio
  ports:
    - protocol: TCP
      port: 80
      targetPort: 4873
EOF
```

```bash
- 使用单独域名部署

$ cat <<EOF | microk8s.kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: verdaccio.example.com
      http:
        paths:
        - backend:
            serviceName: verdaccio-service
            servicePort: 80
          path: /
EOF
```

- 使用路径方式部署，使用版本`verdaccio/verdaccio:4.2.1`, 后面的版本有个bug未修复[链接](https://github.com/verdaccio/verdaccio/issues/1523). 配置文件中加入`url_prefix: /verdaccio/`项


```bash
$ cat <<EOF | microk8s.kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - host: demo.example.com
    http:
      paths:
      - path: /verdaccio(/|$)(.*)
        backend:
          serviceName: verdaccio-service
          servicePort: 80
EOF
```

# 使用方式

1. 在服务器端添加用户

```
$ htpasswd  -c /path/of/htpasswd-store-file
```

2. 在客户端登录用户
```
$ npm adduser --registry http://demo.example.com
$ npm profile set password --registry http://demo.example.com
```

### 参考资料
https://verdaccio.org/docs/en/installation
https://github.com/verdaccio/website/blob/master/docs/config.md
https://github.com/verdaccio/verdaccio/issues/1523
https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
