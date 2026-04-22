---
date: 2026-04-22T15:14:41+08:00
title: 让 systemd-boot 引导 Windows
tags: [Linux, Windows]
categories: 技术分享
summary: 使用 systemd-boot 同时引导 Linux 和 Windows，包括从另一块磁盘引导 Windows 的配置方法
---

## 前言

本文介绍如何使用 systemd-boot 同时引导 Linux 和 Windows。

### 目标

通过 systemd-boot 启动菜单选择进入 Linux 或 Windows 系统。

### 环境

- Linux 安装在第一块磁盘的 ESP 分区
- Windows 安装在另一块磁盘
- 使用 UEFI 启动模式

### 参考

- [Arch Wiki: systemd-boot](https://wiki.archlinux.org/title/Systemd-boot)
- [Systemd-boot 文档](https://www.freedesktop.org/software/systemd/man/systemd-boot.html)

## 原理

systemd-boot 是一个简单的 UEFI 启动管理器，它读取 `/boot/loader/entries/` 目录下的配置文件来生成启动菜单。

Windows 原生使用自己的 EFI 启动项，存储在其所在磁盘的 ESP 分区中。当 Linux 和 Windows 在同一磁盘时，systemd-boot 可以直接切换启动上下文；但当 Windows 在另一块磁盘时，需要显式加载该磁盘的 EFI 分区中的 Windows 启动文件。

如果你的 Windows 和 Linux 安装在不同的硬盘上，拥有各自独立的 EFI 分区，开机后 systemd-boot 菜单里是不会出现 Windows 的。为了跨硬盘引导，我们有两种方案。

方案一：复制大法（简单，但需偶而维护）

最简单粗暴的方法，是把 Windows 硬盘 EFI 分区里的 `EFI/Microsoft` 文件夹，整个复制到 Linux 的 EFI 分区里（例如 `/boot/EFI/Microsoft`）。 由于复制过来的引导文件内部指向了 Windows C 盘的 UUID，所以它能顺利把系统拉起来。

缺点： 遇到 Windows 跨版本大更新（如 23H2 升 24H2）时，微软只会更新它自己盘里的引导文件。你 Linux 盘里的这份“副本”一旦落后太多，可能会导致引导失败，需要你手动重新复制一次。

方案二：UEFI Shell 链式跳板（Arch Wiki 推荐，一劳永逸）

这是更优雅的解法。我们不复制 Windows 的文件，而是利用 `edk2-shell` 作为跳板，让它去另一块盘上精确呼叫 Windows 的引导程序。

## UEFI Shell 链式跳板配置

### 1. 准备 edk2-shell 文件

在 Arch 系发行版中安装包并复制到 EFI 分区（假设你的 EFI 挂载在 `/boot`）：

```bash
sudo pacman -S edk2-shell
sudo cp /usr/share/edk2-shell/x64/Shell.efi /boot/shellx64.efi
```

### 2. 提取硬件别名

很多教程会让你去 UEFI Shell 里找 `FS0` 或 `FS1`。**千万别这么做！** 这些别名在插拔 U 盘时会发生漂移，导致找不到系统。我们需要提取绑定了 UUID 的终极别名。

1. Linux 下找线索： 运行 `lsblk -o NAME,SIZE,PARTUUID`，找到你 Windows EFI 分区的大小和 PARTUUID 记下来。
2. 进入 UEFI Shell： 重启并在 systemd-boot 菜单选择 `UEFI Shell`。
3. 定位分区： 输入 `map -b` 看看哪个 `FSx:` 像你的 Windows EFI 分区。输入 `FS1:` 进去，敲 `ls EFI/Microsoft/Boot` 看看有没有 `bootmgfw.efi`。
4. 提取长别名： 确认无误后，输入 `map FS1`（替换为你的实际编号）。 在输出结果中，找到 `Alias(s):` 后面那一长串以 `HD` 开头的字符串，例如 `HD2c`。把这个绝对不会变的别名记下来下面要用到。

### 3. 创建 systemd-boot 条目

回到 Linux 系统，创建 Windows 引导项配置文件：

```bash
sudo nano /boot/loader/entries/windows.conf
```

填入以下内容（替换你的 HD 别名）：

```
title   Windows
efi     /shellx64.efi
options -nointerrupt -nomap -noversion HD2c:EFI\Microsoft\Boot\Bootmgfw.efi
```

### 4. 验证配置

```bash
# 查看启动条目
bootctl list

# 测试配置文件语法
bootctl status
```

如果配置正确，`bootctl list` 应该显示两个条目：Linux 和 Windows。

## 常见问题

### Q: 启动 Windows 时提示找不到文件？

确认 Windows EFI 分区中有 `EFI/Microsoft/Boot/bootmgfw.efi` 文件：

```bash
ls /mnt/windows_efi/EFI/Microsoft/Boot/bootmgfw.efi
```

如果文件不存在，可能需要修复 Windows EFI 启动项。

### Q: 如何在启动时选择默认系统？

编辑 `/boot/loader/loader.conf`：

```
timeout 5
default linux.conf
```

将 `default` 设置为默认条目的文件名

或者将默认选择改为记忆上次的选项：

```
timeout 5
default @saved
```

这样如果用户不干预，就会自动启动上一次经由 `systend-boot` 选择的启动项

## 总结

systemd-boot 提供了简洁的启动管理体验。配置过程主要涉及创建正确的条目文件，无需复杂的分区调整或第三方工具。
