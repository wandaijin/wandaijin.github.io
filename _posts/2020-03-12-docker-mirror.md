---
layout: post
title:  "Docker 国内镜像"
categories: docker registry
---

1. 修改配置文件`daemon.json`，然后使用命令`docker info`检查是否生效

```json
{
  "registry-mirrors": [
    "https://dockerhub.azk8s.cn"
  ]
}
```


### 参考资料
https://github.com/Azure/container-service-for-azure-china/blob/master/aks/README.md#22-container-registry-proxy
https://docker_practice.gitee.io/install/mirror.html
