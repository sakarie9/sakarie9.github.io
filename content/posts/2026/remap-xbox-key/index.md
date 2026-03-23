---
date: 2026-03-23T10:12:04+08:00
title: 重映射 Xbox 手柄的按键
tags:
  - Linux
  - Games
categories: 技术分享
summary: 在 Linux 下使用 Interception + evsieve 重映射 Xbox 手柄的西瓜键或其他键来实现执行任意命令/脚本，比如截屏
---

在 Linux 下使用 Interception + evsieve 重映射 Xbox 手柄的西瓜键或其他键来实现执行任意命令/脚本，比如截屏

<!-- more -->

## 前言

如果你曾试图在 Linux 下，将 Xbox 手柄的西瓜键重映射为全局快捷键（比如截图），你大概率经历过以下绝望的瞬间：

1. 不想永远在后台挂着 Steam Input。
2. 试过底层映射，`evtest` 明明抓到了修改后的按键，但桌面环境（如 Niri, KDE, Sway 等）却毫无反应。

利用 **interception-tools** 强大的底层抓取能力，配合 **evsieve** 优雅的事件路由机制，打造一个零延迟、免写 C 代码脚本、且完美绕过桌面环境安全限制的终极方案。

---

### 为什么选择 Interception + evsieve？

[interception / linux / Interception Tools · GitLab](https://gitlab.com/interception/linux/tools)

[KarsMulder/evsieve: A utility for mapping events from Linux event devices.](https://github.com/KarsMulder/evsieve)

在 Linux 底层生态中，单纯克隆一个手柄并把 F12 塞进去是行不通的。问题可能在于 Wayland 的核心输入库 `libinput` 一旦发现一个设备“既有手柄摇杆，又有键盘按键”，就会将其判定为畸形的“缝合怪设备”，并强硬地静默丢弃所有的键盘信号。（*待考证，即使使用 udev 为手柄添加键盘 ENV 也行不通*）

**evsieve** 的杀手锏在于它的 事件域分离功能。它可以将原手柄的信号劈成两半：一半输出为纯正的虚拟键盘（专门负责发西瓜键），另一半输出为纯正的虚拟手柄（负责打游戏）。这样，所有的桌面系统和游戏平台都能完美识别，互不干扰。

## 安装及配置

### 安装必备工具

无论你使用什么发行版，都需要安装这两个核心包。以 Arch Linux 为例：

```bash
# 安装 interception-tools (系统级事件拦截框架)
sudo pacman -S interception-tools
# 安装 evsieve (输入事件路由器) 
paru -S evsieve
```

确保 `udevmon` 守护服务处于开启状态：

```bash
sudo systemctl enable --now udevmon
```

### 编写配置文件

在 `/etc/interception/` 目录下编辑或创建主配置文件：

```bash
sudo nano /etc/interception/udevmon.yaml
```

填入以下极其精简、但蕴含黑魔法的配置。这里我们以将西瓜键映射为 **F12** 为例：

```yaml
- JOB: 'evsieve --input $DEVNODE grab --map btn:mode key:f12@kb --output @kb name="Evsieve Virtual Xbox Keyboard" --output name="Evsieve Virtual Xbox Controller"'
  DEVICE:
    EVENTS:
      EV_KEY: [BTN_MODE]
    NAME: ".*Xbox.*"
```

#### 原理解析

- `$DEVNODE grab`：`udevmon` 自动匹配插入的手柄，并以独占模式（Grab）接管它，让物理手柄在内核中隐身，防止产生双重输入冲突。
- `--map btn:mode key:f12@kb`：这是破局的核心！拦截 `BTN_MODE`（西瓜键），将其修改为 `KEY_F12`，并为其打上 `@kb` 的域标签。
- `--output @kb name="Xbox Virtual KB"`：将所有带有 `@kb` 标签的事件（即 F12），单独输出到一个名为 `Evsieve Virtual Xbox Keyboard` 的虚拟键盘中。你的桌面环境会立刻、毫无阻碍地识别到这个键盘敲击。
- `--output name="Evsieve Virtual Xbox Controller"`：将剩下所有没有打标签的事件（摇杆、十字键、ABXY 等），原封不动地输出到一个虚拟手柄中。你在 Steam 或模拟器里打游戏不会受到任何影响。

### 重启服务

保存配置文件后，只需重启后台监控服务让配置生效：

```bash
sudo systemctl restart udevmon
```

按下你的 Xbox 西瓜键，你的系统接收到的将是一个极其干净的 F12 信号。你只需要去系统设置（或窗口管理器的配置文件）中，将 F12 绑定到你的截图命令即可。

### 其他

除了绑定 F12，还可以绑定一些不太可能使用到的键，比如 F24。将上面配置中的 F12 全部修改为 F24 即可。

然后在系统设置或 wm 配置中将 F24 绑定到截图脚本上，比如 niri：

```bash
F24 { spawn-sh "FILE=~/Pictures/Screenshots-Games/$(date +'%Y%m%dT%H%M%S').png; grim -o HDMI-A-2 \"$FILE\" && notify-send -a Screenshot -u low -i \"$FILE\" '截图已保存'"; }
```

---

`interception-tools` 还能做到其他事，比如：

{{< article link="/posts/2024/caps2esc/" >}}
