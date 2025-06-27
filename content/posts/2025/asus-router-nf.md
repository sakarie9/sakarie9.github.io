---
date: 2025-06-23T21:40:19+08:00
title: ASUS Router nf_conntrack expectation table full
tags:
  - Network
categories: 技术随笔
summary: "如何解决 kernel: nf_conntrack: expectation table full"
---

## 问题

ASUS RT-AC86U 系统日志中频繁出现

```plain
Jun 23 21:33:47 kernel: nf_conntrack: expectation table full
Jun 23 21:33:47 kernel: nf_conntrack: expectation table full
Jun 23 21:33:47 kernel: nf_conntrack: expectation table full
Jun 23 21:33:47 kernel: nf_conntrack: expectation table full 
```

## 解决

进入路由器的 ssh

使用如下命令查看原始值，此时为 `150`

```bash
cat /proc/sys/net/netfilter/nf_conntrack_expect_max
```

使用如下命令将默认值提高

```bash
nvram set ct_expect_max=1024  
nvram commit
```

重启生效

```bash
reboot
```
