---
layout: post
title:  "MacOS开机自启动程序设置"
categories: mac
---

### 文件存放目录
mac使用launchctl来管理自启动程序。launchctl加载预定义路径下的xml文件，来安装自启动脚本。脚本存放在5个目录之中：

1. /System/Library/LaunchDaemons/
2. /Library/LaunchDaemons/
3. /System/Library/LaunchAgents
4. /Library/LaunchAgents
5. ~/Library/LaunchAgents


### 增加一个自动文件


```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>KeepAlive</key>
  <true/>
  <key>Label</key>
  <string>user01.docker.mariadb</string>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/docker</string>
    <string>run</string>
    <string>--rm</string>
    <string>--name</string>
    <string>mariadb104</string>
    <string>-p</string>
    <string>127.0.0.1:3306:3306</string>
    <string>-v</string>
    <string>/Users/user01/mariadb/var/lib/mysql:/var/lib/mysql</string>
    <string>-v</string>
    <string>/Users/user01/mariadb/run:/var/run/mysqld</string>
    <string>-d</string>
    <string>mariadb:10.4</string>
  </array>
  <key>RunAtLoad</key>
  <true/>
</dict>
</plist>
```

#### 卸载服务

``` bash
# launchctl unload -w /Users/user01/Library/LaunchAgents/user01.docker.mariadb.plist
```

#### 启动服务

``` bash
# launchctl load -w /Users/user01/Library/LaunchAgents/user01.docker.mariadb.plist
```

#### 列出所有服务

``` bash
# launchctl list
```

### 参考资料
1. https://en.wikipedia.org/wiki/Launchd
2. https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html
3. https://stackoverflow.com/questions/42637339/how-to-start-docker-for-mac-daemon-on-boot
4. https://developer.apple.com/library/archive/technotes/tn2083/_index.html#//apple_ref/doc/uid/DTS10003794
