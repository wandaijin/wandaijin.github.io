---
layout: post
title:  "Ubuntu 19.04 安装 microk8s"
categories: k8s microk8s install
---

[microk8s](https://microk8s.io/)是Canonical公司开发的精简版k8s, 是一个单节点k8s, 占用资源少，安装便捷。适合在开发环境中使用。microk8s使用snap命令进行一键安装。[snap](https://snapcraft.io/docs)是Canonical公司开发的面向桌面，云环境，IOT的软件管理器。

1. 安装

```bash
# lsb_release -a # 查看ubuntu版本号
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 19.04
Release:	19.04
Codename:	disco
# apt-get install -y snapd
# snap install microk8s --classic # 安装microk8s
# echo 'export PATH=$PATH:/snap/bin' > /etc/profile.d/microk8s.sh #设置命令路径
# source  /etc/profile.d/microk8s.sh # 当前shell中设置路径，接下来就可以直接使用microk8s.*命令
```

2. 启用常用的模块，在完成后pod的正常状态是Running，如果一直处在ContainerCreating，可以查看/var/log/syslog, 可能需要科学上网。

```bash
# microk8s.status #查看系统模块
# microk8s.enable dns dashboard
# microk8s.kubectl get pods --all-namespaces
NAMESPACE     NAME                                              READY   STATUS              RESTARTS   AGE
kube-system   coredns-f7867546d-g9vd7                           0/1     Running             0          2m
kube-system   heapster-v1.5.2-6b794f77c8-6k26g                  0/4     ContainerCreating   0          8s
kube-system   kubernetes-dashboard-7d75c474bb-htt69             0/1     ContainerCreating   0          11s
kube-system   monitoring-influxdb-grafana-v4-6b6954958c-h442d   0/2     ContainerCreating   0          11s
```

3. 登录陆dashbord页面

- 获取登录ip地址，这里kubernetes-dashboard对应的ip是10.152.183.218, 在浏览器中输入https://10.152.183.218

```bash
# microk8s.kubectl get service --all-namespaces
NAMESPACE     NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes             ClusterIP   10.152.183.1     <none>        443/TCP                  8m58s
kube-system   heapster               ClusterIP   10.152.183.54    <none>        80/TCP                   4m35s
kube-system   kube-dns               ClusterIP   10.152.183.10    <none>        53/UDP,53/TCP,9153/TCP   6m22s
kube-system   kubernetes-dashboard   ClusterIP   10.152.183.218   <none>        443/TCP                  4m37s
kube-system   monitoring-grafana     ClusterIP   10.152.183.204   <none>        80/TCP                   4m36s
kube-system   monitoring-influxdb    ClusterIP   10.152.183.17    <none>        8083/TCP,8086/TCP        4m36s
```

- 获取dashbord登录密钥token

```bash
# token=$(microk8s.kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
# microk8s.kubectl -n kube-system describe secret $token
ame:         default-token-nxfpm
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 62354db7-c8a3-46c9-9127-d05cd7090eba

Type:  kubernetes.io/service-account-token

Data
====
namespace:  11 bytes
token:      eyJhbGciO.....gjA
ca.crt:     1115 bytes
```

4. 停止microk8s

```bash
# microk8s.stop
```

5. 启动microk8s

```bash
# microk8s.start
```

6. 卸载microk8s, 数据文件也会被删除

```bash
# microk8s.reset #stop all running pods
# snap remove microk8s
```

_备注_
_snap的数据目录是`/var/snap/microk8s/`, 如果需要对microk8s进行配置，可以在`/var/snap/microk8s/current/args/`找到对应的配置项，更改完成记得重启microk8s_

### 参考资料
https://microk8s.io/docs/
