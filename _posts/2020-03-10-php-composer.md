---
layout: post
title:  "php composer"
categories: php composer
---

1. 安装指定的包

`composer require <vendor>/<package> `

2. 升级到最新的兼容的版本

`composer update`

版本对照, see https://getcomposer.org/doc/articles/versions.md


```php
"require": {
    "vendor/package": "1.3.2", // exactly 1.3.2

    // >, <, >=, <= | specify upper / lower bounds
    "vendor/package": ">=1.3.2", // anything above or equal to 1.3.2
    "vendor/package": "<1.3.2", // anything below 1.3.2

    // * | wildcard
    "vendor/package": "1.3.*", // >=1.3.0 <1.4.0

    // ~ | allows last digit specified to go up
    "vendor/package": "~1.3.2", // >=1.3.2 <1.4.0
    "vendor/package": "~1.3", // >=1.3.0 <2.0.0

    // ^ | doesn't allow breaking changes (major version fixed - following semver)
    "vendor/package": "^1.3.2", // >=1.3.2 <2.0.0
    "vendor/package": "^0.3.2", // >=0.3.2 <0.4.0 // except if major version is 0
}
```

默认使用的tag，如果需要使用branch可使用`dev-*`指定分支

3. 配置使用阿里镜像库

新增配置
`composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/`
取消配置
`composer config --unset repos.packagist`
查看配置
`composer config --list`

4. 配置禁用和使用https

禁用
`composer config secure-http false`

启用(默认值)
`composer config secure-http true`

### 参考资料
https://getcomposer.org/
