---
title: LXC 容器中的 Docker 设置
date: 2024-04-10
summary: "在PVE下的LXC容器中运行Docker容器时，若出现unable to apply caps: operation not permitted: unknown.错误，需要在PVE下进行一些配置"
tags: [Docker, Linux]
categories: 技术随笔
---

{{< alert "circle-info" >}}
[https://danthesalmon.com/running-docker-on-proxmox/](https://danthesalmon.com/running-docker-on-proxmox/)
{{< /alert >}}

在 PVE 下的 LXC 容器内运行 Docker 容器时，若出现 `unable to apply caps: operation not permitted: unknown.` 则需要在 PVE 下进行一些配置

在`/etc/pve/lxc/<id>.conf`中添加：

```
lxc.apparmor.profile: unconfined
lxc.cgroup.devices.allow: a
lxc.cap.drop:
```
