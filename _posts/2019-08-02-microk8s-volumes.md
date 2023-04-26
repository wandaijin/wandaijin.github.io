---
layout: post
title: "microk8s存储管理"
categories: microk8s volumes
---

microk8s 是通过 k8s 认证的，下文所有的概念和操作均为 k8s 的概念，如果不是使用 microk8s，可以将`microk8s.kubectl`改成对应的命令，一般为`kubectl`

1. 创建 storageclass

Storage Classes 是一组设置，在创建 PersistentVolume 时需要使用。provisioner 指定存储设备提供方式，可以是本地磁盘，也可以是云提供商等其他方式。具体方式可以参考[官方网站](https://kubernetes.io/docs/concepts/storage/storage-classes/)。

```bash
$ microk8s.kubectl get storageclass
$ cat <<EOF | microk8s.kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
EOF
$ microk8s.kubectl get storageclass #查看列表
$ microk8s.kubectl patch storageclass local-storage -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}' #设为默认
$ microk8s.kubectl get storageclass #查看列表
$ microk8s.kubectl patch storageclass local-storage -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}' #取消默认设置
```

2. 创建 PersistentVolume

```bash
$ microk8s.kubectl get pv
$ mkdir -p /data/k8s-pv/pv001
$ cat <<EOF | microk8s.kubectl apply -f -
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv001
  labels:
    type: local
spec:
  storageClassName: local-storage
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "/data/k8s-pv/pv001"
EOF
$ microk8s.kubectl get pv
```

3. 创建 Persistent Volume Claim 以便 pod 使用, 一个 PersistentVolumeClaim 只能绑定一个 PersistentVolume，一个 PersistentVolume 也只能被一个 PersistentVolumeClaim 绑定。

```bash
$ microk8s.kubectl get pvc
$ cat <<EOF | microk8s.kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-web
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF
$ microk8s.kubectl get pvc
```

4. 删除 PersistentVolumeClaim, 删除 PersistentVolume

如果磁盘正在被使用，则需要先将 pod 删除，然后再依次删除 pvc，pv。

```bash
$ microk8s.kubectl get deployment
$ microk8s.kubectl delete deployment <your-deployment>
$ microk8s.kubectl get service
$ microk8s.kubectl delete service <your-service>
$ microk8s.kubectl get pvc
$ microk8s.kubectl delete pvc pvc-web
$ microk8s.kubectl get pv
$ microk8s.kubectl delete pv pv003
```

### 参考资料

https://kubernetes.io/docs/concepts/storage/storage-classes/
https://kubernetes.io/docs/concepts/storage/volumes/
https://kubernetes.io/docs/concepts/storage/persistent-volumes/
https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
