---
layout: post
title:  "microk8s使用docker安装配置gogs"
categories: microk8s gogs
---

1. 创建存储卷(本地存储)，以便挂载gogs数据目录

```bash
$ mkdir -p /data/k8s-pv/pv002
$ cat <<EOF | microk8s.kubectl apply -f -
kind: PersistentVolume
apiVersion: v1
metadata:
 name: pv002
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
   path: "/data/k8s-pv/pv002"
EOF
$ cat <<EOF | microk8s.kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: pvc-gogs
spec:
 storageClassName: local-storage
 accessModes:
   - ReadWriteOnce
 resources:
   requests:
     storage: 30Gi
EOF
```

2. 创建gogs应用

```bash
$ cat <<EOF | microk8s.kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
 name: gogs-deployment
spec:
 replicas: 1
 selector:
   matchLabels:
     app: gogs
 strategy:
   type: Recreate
 template:
   metadata:
     labels:
       app: gogs
   spec:
     containers:
     - image: gogs/gogs
       name: private-gogs
       ports:
       - containerPort: 22
       - containerPort: 3000
       volumeMounts:
       - name: gogs-persistent-storage
         mountPath: /data
     volumes:
     - name: gogs-persistent-storage
       persistentVolumeClaim:
         claimName: pvc-gogs
EOF
```

3. 创建服务暴露http(3000)端口, 创建完成访问'gogs.example.com'进行页面[安装设置](https://gogs.io/docs/installation/configuration_and_run.html)

```bash
$ cat <<EOF | microk8s.kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: gogs-service
spec:
  selector:
    app: gogs
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
EOF
$ cat <<EOF | microk8s.kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: gogs.example.com
      http:
        paths:
        - backend:
            serviceName: gogs-service
            servicePort: 80
          path: /
EOF
```

4. 暴露ssh端口
- 配置gogs，暴露ssh端口到集群外部

* 对外暴露ssh端口(172.16.13.100)

```bash
$ cat <<EOF | microk8s.kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: gogs-ssh-service
spec:
  externalIPs:
  - 172.16.13.100
  type: NodePort
  ports:
  - port: 22
    targetPort: 22
    protocol: TCP
  selector:
    app: gogs
EOF
```

* 配置_/data/k8s-pv/pv002/gogs/conf/app.ini_文件中server部分

>DISABLE_SSH = false
>SSH_DOMAIN = 172.16.13.100
>SSH_PORT = 22
>SSH_LISTEN_PORT = 22

- 配置gogs，暴露ssh端口到集群内部（使用headless和dns）

```bash
$ cat <<EOF | microk8s.kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: gogs-ssh
spec:
  clusterIP: None
  ports:
  - port: 22
    targetPort: 22
    protocol: TCP
  selector:
    app: gogs
EOF
```

* 配置_/data/k8s-pv/pv002/gogs/conf/app.ini_文件中server部分

>DISABLE_SSH = false
>SSH_DOMAIN = gogs-ssh.default.svc.cluster.local
>SSH_PORT = 22
>SSH_LISTEN_PORT = 22




### 参考资料
https://gogs.io/docs/installation
https://gogs.io/docs/advanced/configuration_cheat_sheet
https://github.com/gogs/gogs/blob/master/conf/app.ini
https://kubernetes.io/docs/concepts/services-networking/service/#headless-services
https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/
