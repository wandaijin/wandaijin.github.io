---
layout: post
title:  "Service snap.microk8s.daemon-kubelite is not running"
categories: microk8s debug
---

# 问题及排查

1. `microk8s.status` 使用`microk8s.start`无法启动服务

```bash
$
$ microk8s.status
microk8s is not running. Use microk8s inspect for a deeper inspection.
$
$ microk8s inspect
.....
 FAIL:  Service snap.microk8s.daemon-kubelite is not running
For more details look at: sudo journalctl -u snap.microk8s.daemon-kubelite
.....
$
$ journalctl -u snap.microk8s.daemon-kubelite
.....
Waiting for the API server
Starting API Server
Flag --insecure-bind-address has been deprecated, This flag has no effect now and will be removed in v1.24.
Flag --insecure-port has been deprecated, This flag has no effect now and will be removed in v1.24.
Error: invalid port value 8080: only zero is allowed
API Server exited invalid port value 8080: only zero is allowed
.....
```

2. 处理结果, 将文件/var/snap/microk8s/current/args/kube-apiserver中的 --insecure-port变更为0

```bash
$ vi /var/snap/microk8s/current/args/kube-apiserver
$
$ cat /var/snap/microk8s/current/args/kube-apiserver
....
--insecure-port=0
....
```


### 参考资料
https://github.com/ktsakalozos/microk8s.io/blob/docs/ports-rework/docs/ports.md
