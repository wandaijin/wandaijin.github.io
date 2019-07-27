---
layout: post
title:  "squid启动失败，提示Couldn't start logfile helper错误"
categories: how-to
---

### 问题
在docker中运行安装并运行`squid`时系统报错FATAL: Couldn't start logfile helper，运行`squid -N -d 3`输出错误信息：
``` bash
# squid -N -d 3
2019/07/27 01:30:06 kid1| Set Current Directory to /var/cache/squid                       
2019/07/27 01:30:06 kid1| Starting Squid Cache version 4.8 for x86_64-alpine-linux-musl...
2019/07/27 01:30:06 kid1| Service Name: squid                                             
2019/07/27 01:30:06 kid1| Process ID 17                                                   
2019/07/27 01:30:06 kid1| Process Roles: worker                                           
2019/07/27 01:30:06 kid1| With 1048576 file descriptors available                         
2019/07/27 01:30:06 kid1| Initializing IP Cache...                                        
2019/07/27 01:30:06 kid1| DNS Socket created at 0.0.0.0, FD 5                             
2019/07/27 01:30:06 kid1| Adding nameserver 108.61.10.10 from /etc/resolv.conf            
2019/07/27 01:30:06 kid1| Logfile: opening log daemon:/var/log/squid/access.log           
2019/07/27 01:30:06 kid1| Logfile Daemon: opening log /var/log/squid/access.log           
2019/07/27 01:30:06 kid1| ipcCreate: fork: (12) Out of memory                             
2019/07/27 01:30:06 kid1| storeDirWriteCleanLogs: Starting...                             
2019/07/27 01:30:06 kid1|   Finished.  Wrote 0 entries.                                   
2019/07/27 01:30:06 kid1|   Took 0.00 seconds (  0.00 entries/sec).                       
2019/07/27 01:30:06 kid1| FATAL: Couldn't start logfile helper                            
2019/07/27 01:30:06 kid1| Squid Cache (Version 4.8): Terminated abnormally.
```

### 原因
squid默认使用daemon(包装syslogd)来写入访问日志。在系统中syslogd并没有启动(`pgrep -l syslogd`查看）。
```bash
# pgrep -l syslogd
```

### 解决方式
在squid配置在配置/etc/squid/squid.conf修改access_log配置。配置文件路径可使用`squid -k parse 2>&1 | grep Configuration`进行查看
```bash
# squid -k parse 2>&1 | grep Configuration
2019/07/27 06:56:51| Processing Configuration File: /etc/squid/squid.conf (depth 0)
# echo -e "access_log stdio:/var/log/squid/access.log squid \n" >> /etc/squid/squid.conf
# tail /etc/squid/squid.conf
.....
access_log stdio:/var/log/squid/access.log squid
```

### 参考资料
https://github.com/squid-cache/squid/blob/master/src/ipc.cc
http://www.squid-cache.org/Doc/config/access_log/
