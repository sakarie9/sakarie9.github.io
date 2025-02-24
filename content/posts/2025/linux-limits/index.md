---
date: "2025-02-24T14:59:25+08:00"
title: "提高Linux的文件描述符限制"
tags: [Linux]
categories: 技术分享
---

## 为什么需要调整文件描述符限制？

### 1. 默认限制的局限性

Arch Linux 默认的 open files 限制为：

```bash
$ ulimit -n
1024  # 普通用户默认 soft limit
$ ulimit -Hn
524288 # hard limit
```

**这个限制对于以下场景明显不足**：

- 高并发 Web 服务（Nginx/Apache）
- 数据库系统（MySQL/Redis）
- 大数据处理（Spark/Hadoop）
- 游戏服务器（需要加载大量资源文件）

### 2. 不调整的风险案例

```bash
# 典型错误日志示例
java.io.IOException: Too many open files
nginx: [alert] 4096 worker_connections exceed open file resource limit: 1024
Unhandled Exception: EETypeRva:0x00667F40: Too many open files
```

## 配置

### 步骤 1：创建配置文件

创建文件 `/etc/security/limits.d/10-nofile-limits.conf`：

```conf
# - nofile - max number of open file descriptors
* soft nofile 8192    # 所有用户 soft limit
# * hard nofile 524288 # 保留系统默认 hard limit
```

### 步骤 2：立即生效配置

```bash
# 对于已登录用户需要重新登录
ssh localhost  # 快速重新建立会话

# 验证当前限制
ulimit -Sn && ulimit -Hn
# 应输出：
# 8192
# 524288
```

### 步骤 3：系统级监控（可选）

```bash
# 查看全局文件描述符使用
watch -n 5 "cat /proc/sys/fs/file-nr"

# 输出示例：
# 7584  0       3254236
# 分别表示：已分配 | 未使用 | 系统上限
```

## 高级技巧与排错

### systemd 服务的特殊处理

使用 systemd 管理的服务需要额外配置：

```bash
# 编辑 systemd 全局配置
sudo nano /etc/systemd/system.conf

# 修改以下参数：
DefaultLimitNOFILE=8192:524288
```

重启服务生效：

```bash
sudo systemctl daemon-reload
sudo systemctl restart your-service
```

### 常见问题排查

**Q：修改后限制未生效？**

- 确认用户已重新登录
- 检查 `sshd_config` 是否启用 `UsePAM yes`
- 使用 `cat /proc/<PID>/limits` 验证进程实际限制

**Q：应该设置多大的值？**
应用类型 | 推荐 soft limit
--------------|-----------------
Web 服务器 | 65535-131072
数据库 | 262144-524288
桌面应用 | 16384-32768

## 结语

通过合理的文件描述符限制配置，可以在 **系统稳定性** 和 **应用性能** 之间取得平衡。建议定期监控文件描述符使用情况：

```bash
# 统计各进程打开文件数
lsof -n | awk '{print $1}' | sort | uniq -c | sort -nr | head
```

> 📘 **扩展阅读**：Linux 内核关于文件管理的 [官方文档](https://www.kernel.org/doc/Documentation/sysctl/fs.txt)
