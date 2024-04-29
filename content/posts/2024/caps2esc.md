---
title: 将 CapsLock 重映射到 Esc 和 Ctrl
date: 2024-04-19
summary: 使用 caps2esc ，让 Caps Lock 键在单独按下时充当 Escape 键，在与其他键组合按下时充当 Control 键
tags: [Linux]
categories: 技术分享
---

{{< alert "circle-info" >}}
本篇为 [https://ejmastnak.com/tutorials/arch/caps2esc](https://ejmastnak.com/tutorials/arch/caps2esc) 的整理与翻译
{{< /alert >}}

## 前言

### 目标

使用 `caps2esc` ，让 Caps Lock 键在单独按下时充当 Escape 键，在与其他键组合按下时充当 Control 键。

### 动机

愉快、符合人体工程学，系统范围地将 Caps Lock 映射为非常有用的 Escape 和 Control 键，以及拥有更好的 Vim 或 Emacs 体验。

### 参考

- [caps2esc GitLab 页面](https://gitlab.com/interception/linux/plugins/caps2esc)
- [Ask Ubuntu: 如何安装 caps2esc？](https://askubuntu.com/questions/979359/how-do-i-install-caps2esc)

### 介绍

`caps2esc` 实用程序允许您在 `libevdev` 库级别将 Caps Lock 重新映射到 Escape 和 Control。因为 `libevdev` 相对底层——仅高于操作系统内核——所以此解决方案除了图形环境外，还适用于纯 Linux 控制台。

## 安装使用

### 安装

从 Arch 仓库安装 `caps2esc` 包：

```bash
# Install caps2esc
sudo pacman -S interception-caps2esc
```

这还会将 `interception-tools` 包作为依赖项进行安装。 `interception-tools` 包含一个名为 `udevmon` 的输入设备监视程序，我们将使用它来捕获 Caps Lock 和 Escape。

### 配置

创建配置文件 `/etc/udevmon.yaml` （如果不存在），并在其中添加以下配置：

```yaml
- JOB: "intercept -g $DEVNODE | caps2esc | uinput -d $DEVNODE"
  DEVICE:
    EVENTS:
      EV_KEY: [KEY_CAPSLOCK, KEY_ESC]
```

- 说明（点击展开）
    
    此 `udevmon` 作业在按下 Caps Lock 和 Escape 键（由名称 `KEY_CAPSLOCK` 和 `KEY_ESC` 标识）时运行 shell 命令 `intercept -g $DEVNODE | caps2esc | uinput -d $DEVNODE` ；`udevmon` 将根据需要将 `$DEVNODE` 变量设置为匹配设备的路径（ `/dev` 目录中的某个虚拟文件）。
    
    shell 命令使用 `intercept` 程序获取 Caps Lock 或 Escape 键的输入设备，将键事件管道传输到 `caps2esc` 程序（该程序实现 Caps Lock 到 Escape/Control 逻辑），然后使用 `uinput` 将处理后的输出管道传输回虚拟键设备。（你可以阅读 [Interception Tools/How it works](https://gitlab.com/interception/linux/tools#how-it-works) 以了解详细信息。）
    

Tip：上述 `udevmon` 配置将使 Caps Lock 作为 Escape 和 Control 键工作，并将 Escape 键作为 Caps Lock 键工作。如果您希望 Escape 键仍作为 Escape 键工作，您可以用 `caps2esc -m 1` 替换 `caps2esc` ，它使用 `caps2esc` “最小模式”并且不影响 Escape 键（有关文档，请参见 `caps2esc -h` ）。

您现在只需启动 `udevmon` 程序，我们将使用 `systemd` 单元来执行此操作。

### 配置 `udevmon` 的 `systemd` 服务

创建 `systemd` 服务单元文件 `/etc/systemd/system/udevmon.service` （如果不存在），并在其中添加内容：

```bash
[Unit]
Description=udevmon
Wants=systemd-udev-settle.service
After=systemd-udev-settle.service

# Use `nice` to start the `udevmon` program with very high priority,
# using `/etc/udevmon.yaml` as the configuration file
[Service]
ExecStart=/usr/bin/nice -n -20 /usr/bin/udevmon -c /etc/udevmon.yaml

[Install]
WantedBy=multi-user.target
```

此服务单元以非常高的优先级启动 `udevmon` 程序（ `nice` 允许您设置程序的调度优先级； `-20` niceness 是最高可能的优先级）。确保 `ExecStart` 行（例如 `/usr/bin/udevmon` ）中的 `uvdevmon` 路径与 `which udevmon` 的输出相匹配。

然后启用并启动 `udevmon` 服务：

```bash
# Enable and start the `udevmon` service
sudo systemctl enable --now udevmon.service

# Optionally verify the `udevmon` service is active and running
systemctl status udevmon
```

此时应该已经完成了，尝试使用例如 `<CapsLock>-L` 来清除终端屏幕（就像你通常使用 `<Ctrl>-L` 所做的那样）。如果启用了 `udevmon` 服务，则 `udevmon` 会自动在系统启动时启动。
