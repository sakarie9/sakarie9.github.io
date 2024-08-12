---
title: 罗技鼠标无线模式下在 Linux 里的滚轮问题
date: 2024-08-12
categories: 技术分享
tags: [Linux]
---

## 前言

罗技的某些鼠标 （G502、G903）的滚轮，存在步进/无极滚动两种模式，在 Linux 下，步进模式的滚轮会出现滚一下、动两格的情况

这个问题虽然在浏览器等场景不太明显，滚动可能只会导致几个像素的差异，但在终端中的滚动往往会偏移几行，或者在音乐播放器中通过滚动调整音量，基本上永远都没法滚动到想要的音量上

在 Minecraft 中问题就更严重了，滑动滚轮会来回在几个快捷栏之间跳动，想要准确利用滚轮滚动到目标快捷栏会非常恼人

## 解决方案

相关的讨论：

[Mouse scrolling behaves oddly on wayland](https://www.reddit.com/r/hyprland/comments/1dylqsi/help_mouse_scrolling_behaves_oddly_on_wayland/)

[HID++ Logitech G903 generates full scroll wheel events with every hi-res tick when attached via USB](https://bugzilla.kernel.org/show_bug.cgi?id=216885)

TL;DR

这个问题是由内核模块 `hid_logitech_dj` 和 `hid_logitech_hidpp` 所造成的

使用以下命令临时禁用这两个模块：`sudo rmmod hid_logitech_dj hid_logitech_hidpp`

如果确实有用，进行如下步骤永久禁用模块

1. 新建文件 `/etc/modprobe.d/99-custom-no-logitech-driver.conf`

2. 在文件中添加以下两行

```conf
blacklist hid_logitech_dj
blacklist hid_logitech_hidpp
```

3. 重新插拔无线接收器

注意，禁用掉这两个模块后，能修改鼠标配置的程序（如 Piper 等），将会无法使用，确保配置完成之后再进行以上操作
