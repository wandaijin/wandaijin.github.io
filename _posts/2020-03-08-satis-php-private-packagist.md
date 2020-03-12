---
layout: post
title:  "PHP轻量级私有包管理工具satis的安装"
categories: php package manager
---
satis是一款轻量级的PHP包管理工具。

1. 配置

```
vi satis.json
```
- 权限设置
```json
  {
    "repositories": [{
      "type": "composer",
      "url": "ssh2.sftp://example.org",
      "options": {
        "ssh2": {
          "username": "composer",
          "pubkey_file": "/home/composer/.ssh/id_rsa.pub",
          "privkey_file": "/home/composer/.ssh/id_rsa"
        }
      }
    }]
  }
```

- 库设置，设置指定的库，不使用`"require-all": true`
```json
  {
      "repositories": [
        { "type": "vcs", "url": "https://github.com/mycompany/privaterepo" }
      ],
      "require": {
        "company/package": "*"
      }
  }
```

2. 构建
```
php bin/satis build satis.json output
```

3. 使用
  - 本地文件
  ```
  composer create-project --repository=/pathto/satis/output/packages.json --remove-vcs  project yourProjectname

  ```

  如果需要使用mater分支的最新版本，在命令行最后加上`master-dev`参数

  - 发布到web

### 参考资料
https://getcomposer.org/doc/articles/handling-private-packages-with-satis.md
https://github.com/composer/satis
