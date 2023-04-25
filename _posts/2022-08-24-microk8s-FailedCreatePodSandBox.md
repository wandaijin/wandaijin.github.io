---
layout: post
title: "microk8s FailedCreatePodSandBox k8s.gcr.io"
categories: microk8s
---

## 问题发现

自建 k8s 时创建 pod 一直不成功，查看日志发现原因是 k8s.gcr.io 无法连接。这里感谢一下 GFW。

```bash
$ microk8s.kubectl get events --sort-by=.metadata.creationTimestamp
.....
Warning  FailedCreatePodSandBox Failed to create pod sandbox: rpc error:
code = Unknown desc = failed to get sandbox image "k8s.gcr.io/pause:3.1": failed to pull image "k8s.gcr.
io/pause:3.1": failed to pull and unpack image "k8s.gcr.io/pause:3.1": failed to resolve reference "k8s.
gcr.io/pause:3.1": failed to do request: Head https://k8s.gcr.io/v2/pause/manifests/3.1: dial tcp 74.125.204.82:443: i/o timeout
....
```

## 解决方式

人工打包并导入镜像文件到 k8s 中去。需要注意的是镜像也有`namespace`。

1. 打包镜像

```bash
$ docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1
$ docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1
$ docker save k8s.gcr.io/pause:3.1 -o pause.tar
```

2. 导入镜像。等待一会儿自动恢复正常。注意`-n`参数。

```bash
$ microk8s.ctr namespace ls
NAME    LABELS
default
k8s.io
$ microk8s.ctr --namespace k8s.io images import pause.tar
$ microk8s.ctr --namespace k8s.io images ls
```

### 参考资料

1. https://hub.docker.com/u/mirrorgooglecontainers
2. https://www.zeng.dev/post/2020-containerd-image-import/
