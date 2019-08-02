---
layout: post
title:  "k8s.gcr.io timeout"
categories: k8s tips
---

>failed to pull image "k8s.gcr.io/pause:3.1"
由于国内网络环境的原因，在构建k8s时无法拉取境外文件，微软提供免费的镜像供大家使用。

```bash
# tail -f /var/log/syslog
# docker pull gcr.azk8s.cn/google_containers/pause:3.1
# docker images
# docker tag gcr.azk8s.cn/google_containers/pause:3.1 k8s.gcr.io/pause:3.1
# docker images
# docker rmi gcr.azk8s.cn/google_containers/pause:3.1
# docker images
```

### 参考资料
http://mirror.azure.cn/help/docker-registry-proxy-cache.html
http://mirror.azure.cn/help/gcr-proxy-cache.html
