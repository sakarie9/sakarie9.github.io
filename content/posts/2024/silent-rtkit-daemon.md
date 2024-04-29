---
title: 让 rtkit-daemon 闭嘴
date: 2024-04-10
slug: silent-rtkit-daemon
summary: "\"Failed to set real-time priority for thread: Operation not permitted\""
tags: [Linux, Network]
categories: 技术随笔
---

在使用某些软件时可能会在 Log 内看到 `Failed to set real-time priority for thread: Operation not permitted (1)` 类似的提示，可以通过安装包 [`realtime-privileges`](https://archlinux.org/packages/?name=realtime-privileges) 并将当前用户添加进 `realtime` 组中解决

但 `rtkit-daemon` 会在 Log 中输出一大堆的 `rtkit-daemon[1498]: Supervising 1 threads of 1 processes of 1 users.` 影响其他日志的浏览

## 解决方案

修改 `rtkit-daemon.service` :

```
# systemctl edit rtkit-daemon

[Service]
LogLevelMax=5
```

其中 LogLevel 对应的可以有以下几个级别

```
0 or emergency, (highest priority messages)
1 or alert,
2 or critical,
3 or error,
4 or warning,
5 or notice,
6 or info
7 or debuginfo (lowest priority messages)
```