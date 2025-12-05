---
date: 2025-04-14T21:14:27+08:00
title: Linux 多显卡设备中的显卡指定
tags:
  - Linux
categories: 技术分享
summary: 多显卡设备中的显卡指定方式和设置
---

## 前言

当 Linux 中存在多张显卡时，理所当然的会存在程序显卡选择上的问题。

例如游戏选择了核显而非独显运行，浏览器选择了解码单元较弱且较为耗电的独显运行。

这些是用户所不希望的行为，所幸，mesa 给我们提供了一些可以手动指定显卡设备的环境变量。

## 默认情况

我们可以使用 `MESA_VK_DEVICE_SELECT=list vulkaninfo` 查看当前可用的显卡信息。

```text
selectable devices:
  GPU 0: 1002:67df "AMD Radeon RX 580 Series (RADV POLARIS10)" discrete GPU 0000:10:00.0
  GPU 1: 1002:1638 "AMD Radeon Graphics (RADV RENOIR)" integrated GPU 0000:30:00.0
```

例如这里拥有两张显卡，`GPU 0` 为独立显卡 (dGPU)，`GPU 1` 为核显 (iGPU)。

使用 `glxinfo` 和 `vkcube` 来进行 `opengl` 和 `vulkan` 应用的简单测试。

```shell
❯ glxinfo | rg Device
    Device: AMD Radeon Graphics (radeonsi, renoir, ACO, DRM 3.61, 6.14.2-2-cachyos) (0x1638)
```

```shell
❯ vkcube
Selected WSI platform: xcb
Selected GPU 1: AMD Radeon RX 580 Series (RADV POLARIS10), type: DiscreteGpu
```

其中 `glxinfo` 选择了核显，而 `vkcube` 选择了独显。

> 根据 [mesa #24750](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/24750) 所述，`vkcube` 似乎总是会优先适用独显

## 环境变量

在 [mesa](https://docs.mesa3d.org/envvars.html) 的文档中，我们注意到其中 [DRI_PRIME](https://docs.mesa3d.org/envvars.html#envvar-DRI_PRIME) 和 [MESA_VK_DEVICE_SELECT](https://docs.mesa3d.org/envvars.html#envvar-MESA_VK_DEVICE_SELECT) 这两个变量是和设备选择相关的。

- DRI_PRIME

   默认 GPU 是 Wayland/Xorg 使用的或连接到显示器的 GPU。此变量允许选择不同的 GPU。它适用于 OpenGL 和 Vulkan（在这种情况下，“选择”意味着该 GPU 将排在报告的物理设备列表的首位）。支持的语法有：

   - `DRI_PRIME=N` : 选择第 N 个非默认 GPU（N > 0）。
   - `DRI_PRIME=pci-0000_02_00_0` : 选择连接到此 PCIe 总线的 GPU
   - `DRI_PRIME=vendor_id:device_id` : 选择匹配这些 ID 的第一个 GPU。

   对于 Vulkan，可以附加 `!` ，这样只有选定的 GPU 会暴露给应用程序（例如：DRI_PRIME=1!）。

- MESA_VK_DEVICE_SELECT

   当设置为“list”时，打印设备列表。当设置为“vid:did”时，从 PCI 设备中选择编号。该 PCI 设备将被设为默认设备。默认设备会在 vkEnumeratePhysicalDevices API 中作为第一个设备返回。使用“vid:did!”的效果与使用 `MESA_VK_DEVICE_SELECT_FORCE_DEFAULT_DEVICE` 变量相同。即被识别为默认的设备将是 vkEnumeratePhysicalDevices API 中唯一返回的设备。

## 配置

由于 `DRI_PRIME` 已经适用于 OpenGL 和 Vulkan，一般情况下只需要配置它就足够了。

> 经测试，在两者同时设置时，`MESA_VK_DEVICE_SELECT` 似乎会优先于 `DRI_PRIME` 对 `vkcube` 生效

虽然 `DRI_PRIME` 支持 `DRI_PRIME=N` 的写法，这里不建议这样使用，因为 `DRI_PRIME=N!` 并不能强制使应用使用序号为 N 的显卡。

```shell
❯ DRI_PRIME=0! vkcube
Selected WSI platform: xcb
Selected GPU 1: AMD Radeon RX 580 Series (RADV POLARIS10), type: DiscreteGpu
❯ DRI_PRIME=1! vkcube
Selected WSI platform: xcb
Selected GPU 0: AMD Radeon RX 580 Series (RADV POLARIS10), type: DiscreteGpu
```

而使用 `DRI_PRIME=vendor_id:device_id!` 时正常运作。

```shell
❯ DRI_PRIME=1002:1638! vkcube
Selected WSI platform: xcb
Selected GPU 0: AMD Radeon Graphics (RADV RENOIR), type: IntegratedGpu
```

因此，在我们想要指定显卡的应用程序运行前加上 `DRI_PRIME=vendor_id:device_id!` 就能强制使其使用指定的显卡了。

## 显卡睡眠

在同时拥有核显和独显的笔记本等设备上，有时用户会希望使用核显完成桌面渲染、视频输出等所有工作，只有在游戏/AI 时，显式调用独显，在不使用独显时希望独显能进入睡眠状态省电。

首先查看显卡对应的 `card*`：

```shell
❯ ls -l /dev/dri/by-path/
lrwxrwxrwx 1 root root  8 12月 5日 18:22 pci-0000:10:00.0-card -> ../card1
lrwxrwxrwx 1 root root 13 12月 5日 18:22 pci-0000:10:00.0-render -> ../renderD128
lrwxrwxrwx 1 root root  8 12月 5日 18:22 pci-0000:30:00.0-card -> ../card2
lrwxrwxrwx 1 root root 13 12月 5日 18:22 pci-0000:30:00.0-render -> ../renderD129
```

```shell
❯ lspci -nn | grep -i vga
10:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Ellesmere [Radeon RX 470/480/570/570X/580/580X/590] [1002:67df] (rev e7)
30:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Cezanne [Radeon Vega Series / Radeon Vega Mobile Series] [1002:1638] (rev c9)
```

这里得到我的独显是 `card1`，用自己的替换下列命令中的 `card1`。

首先确认 `cat /sys/class/drm/card1/device/power/control` 需要为 `auto`。

然后用 `cat /sys/class/drm/card1/device/power/runtime_status` 查看当前显卡状态，`active` 为显卡正在运行，`suspended` 为已睡眠，正常情况下闲置 5s 左右就会进入睡眠。

如果显卡无法进入睡眠，可能有多种情况：

- 被占用
		使用 `sudo fuser -v /dev/dri/renderD128` 查看占用显卡的进程
- 音频驱动占用
		使用 `echo '0000:10:00.1' | sudo tee /sys/bus/pci/drivers/snd_hda_intel/unbind` 解绑显卡的 HDMI 音频输出驱动，注意执行后此显卡将无法输出声音
		`0000:10:00.1` 使用 `lspci -nn | grep -i audio` 获取

## Vulkan

Vulkan 存在的问题是，其在初始化时会遍历唤醒所有显卡，即使 Vulkan 程序在后续不会使用这张显卡。

对此没有什么好的手段规避，只能通过 `DRI_PRIME=vendor_id:device_id!` 的方式强行让程序只在初始化时唤醒一次显卡，后续不再使用。

如果只使用 `DRI_PRIME=vendor_id:device_id`，程序会在后续依然占用显卡，表现为：

```shell
❯ sudo fuser -v /dev/dri/renderD128
                     用户     进程号 权限   命令
/dev/dri/renderD128: sakari    17133 F.... chromium
                     sakari    18849 F.... electron
                     sakari    60517 F...m vkcube
```

权限多了 `m`，即 `Mapped`，有了它显卡就不会进入睡眠。

## 应用

> 使用 `MESA_VK_DEVICE_SELECT=list vulkaninfo` 查看显卡 id，替换下面样例中的 `vid:did`

一般情况下，使用带感叹号 `DRI_PRIME=vid:did!` 的并不必要，其会使 `vid:did` 对应的设备成为应用程序中唯一可见的设备。

建议仅当指定了设备，但应用仍然默认使用另一张显卡或同时使用多张显卡时才使用 `!`。

### Steam

右键游戏 - 属性 - 通用 - 启动选项中填入 `DRI_PRIME=vid:did %command%` 即可。

### 命令行程序

`DRI_PRIME=vid:did example_binary`

或

`env DRI_PRIME=vid:did example_binary`

### 通过 drun 或 launcher 启动的软件

修改其入口 `/usr/share/applications/whatever.desktop`，在 `Exec` 字段最前面插入 `env DRI_PRIME=vid:did`。

例如 `Exec=env DRI_PRIME=vid:did whatever %F`

> 建议将对应的 `.desktop` 文件复制到 `~/.local/share/applications/` 中，避免应用更新后 `.desktop` 被覆盖

### 全局

> 不太推荐，可能会搞坏一些东西

如果在将显卡 A（例如核显）用作所有的桌面渲染、视频输出（连接显示器）；显卡 B 只在运行游戏、AI 或其他更适合在 B 上运行的任务时手动指定调用，则可以考虑将 `DRI_PRIME` 设置为全局变量。这样可以将大部分的日常任务负载从显卡 B 上转移到显卡 A，可以维持显卡 B 的长时间休眠以省下一些电以及降低些许温度（理论上）。

编辑 `/etc/environment`（对系统生效）或者 `~/.profile` （对用户生效），添加 `DRI_PRIME=vid:did`。

### 包装脚本

你可以使用一个脚本来更方便地使用环境变量。或者同时使用 `mangohud` `gamemode` 等。

在 `~/.local/bin` 中创建一个脚本 `exec-dgpu`。

```shell
#!/bin/bash
DRI_PRIME=vid:did "$@"
```

在运行软件时只需要 `exec-dgpu whatever` 即可。
