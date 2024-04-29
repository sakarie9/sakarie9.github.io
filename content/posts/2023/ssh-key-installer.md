---
title: SSH 密钥一键配置脚本
date: 2023-12-23
tags: [Linux]
categories: 技术随笔
---

{{< alert "circle-info" >}}
本文内容来自：[SSH 密钥一键配置脚本 使用教程 - P3TERX](https://p3terx.com/archives/ssh-key-installer.html)
{{< /alert >}}

## 📝 **语法及选项说明**

`bash <(curl -fsSL s.sakari.top/key.sh) [选项...] <参数>`

- `o` - 覆盖模式，必须写在最前面才会生效
- `g` - 从 GitHub 获取公钥，参数为 GitHub 用户名
- `u` - 从 URL 获取公钥，参数为 URL
- `f` - 从本地文件获取公钥，参数为本地文件路径
- `p` - 修改 SSH 端口，参数为端口号
- `d` - 禁用密码登录

## 🤗 **使用方法**

### 安装公钥

#### 从 GitHub 获取公钥

在 [GitHub 密钥管理页面](https://p3terx.com/go/aHR0cHM6Ly9naXRodWIuY29tL3NldHRpbmdzL2tleXM) 添加公钥，比如我的用户名是 `sakarie9`，那么在主机上输入以下命令即可：

```bash
bash <(curl -fsSL s.sakari.top/key.sh) -g sakarie9
```

#### 从 URL 获取公钥

把公钥上传到网盘，通过网盘链接获取公钥：

```bash
bash <(curl -fsSL s.sakari.top/key.sh) -u https://example.com/key.pub
```

#### 从本地文件获取公钥

通过 FTP 的方式把公钥传到 VPS 上，然后指定公钥路径：

```bash
bash <(curl -fsSL s.sakari.top/key.sh) -f ~/key.pub
```

### 覆盖模式

使用覆盖模式（`-o`）将覆盖 `/.ssh/authorized_keys` 文件，之前的密钥会被完全替换掉，选项必须写在最前面才会生效，比如：

```bash
bash <(curl -fsSL s.sakari.top/key.sh) -o -g sakarie9
```

或者

```bash
bash <(curl -fsSL s.sakari.top/key.sh) -og sakarie9
```

### 禁用密码登录

在确定使用密钥能正常登录后禁用密码登录：

```bash
bash <(curl -fsSL s.sakari.top/key.sh) -d
```

### 修改 SSH 端口

把 SSH 端口修改为 `2222`：

```bash
bash <(curl -fsSL s.sakari.top/key.sh) -p 2222
```

### 一键操作

安装密钥、修改端口、禁用密码登录一键操作：

```bash
bash <(curl -fsSL s.sakari.top/key.sh) -og sakarie9 -p 2222 -d
```

### 一键操作 不修改端口

安装密钥、禁用密码登录一键操作：

```bash
bash <(curl -fsSL s.sakari.top/key.sh) -og sakarie9 -d
```
