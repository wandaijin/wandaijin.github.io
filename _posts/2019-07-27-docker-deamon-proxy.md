---
layout: post
title:  "Docker Daemon Proxy 配置"
categories: docker config
---

### 问题
使用k8s时pod一直创建不成功，一直处在`ContainerCreating`状态:
```bash
# kubectl get pods --all-namespaces
NAMESPACE              NAME                                          READY   STATUS              RESTARTS   AGE
kube-system            coredns-b7464766c-8vq4r                       0/1     ContainerCreating   0          8d
kube-system            helm-install-traefik-qtgf8                    0/1     ContainerCreating   0          8d
kubernetes-dashboard   kubernetes-dashboard-6f89577b77-v46fg         0/1     ContainerCreating   0          8d
kubernetes-dashboard   kubernetes-metrics-scraper-79c9985bc6-sgjg2   0/1     ContainerCreating   0          8d
```

查看系统日志`tail -f /var/log/syslog`，提示pull失败
>failed to pull image "k8s.gcr.io/pause:3.1"

单独使用`docker pull`命令，提示网络超时
```bash
# docker pull k8s.gcr.io/pause:3.1
Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```

### 原因
你懂得：Great Firewall of China

### 解决方式
配置Docker Daemon的代理，使用此方式的前提是一个可用的outside代理。

```bash
# mkdir -p /etc/systemd/system/docker.service.d
# cat > /etc/systemd/system/docker.service.d/http-proxy.conf << EOF
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:7001/"
Environment="HTTPS_PROXY=http://127.0.0.1:7001/"
Environment="NO_PROXY=127.0.0.1,localhost
EOF
# systemctl daemon-reload
# systemctl restart docker
# systemctl show --property=Environment docker
Environment=HTTP_PROXY=http://127.0.0.1:7001/ HTTPS_PROXY=http://127.0.0.1:7001/
```

_扩展：linux系统环境变量设置_
_1. 临时环境变量`export YOUR_ENV=yourenv`_
_2. 个人环境变量`vi ~/.bash_profile`_
_3. 全局环境变量`vi /etc/environment`_

### 参考资料
https://docs.docker.com/config/daemon/systemd/#httphttps-proxy
https://elegantinfrastructure.com/docker/ultimate-guide-to-docker-http-proxy-configuration/
https://stackoverflow.com/questions/23111631/cannot-download-docker-images-behind-a-proxy
https://www.serverlab.ca/tutorials/linux/administration-linux/how-to-configure-proxy-on-ubuntu-18-04/
