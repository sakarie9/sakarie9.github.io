---
date: 2025-12-16T13:48:24+08:00
title: 自定义 Waydroid 的按键绑定
tags:
  - Linux
categories: 技术随笔
---

## 问题

在 Niri + Waydroid 的环境下，在 Waydroid 的焦点下使用 Super（win）键进行窗口操作时，会触发 Waydroid 的谷歌助理

## 处理

将左 Win 键与 Waydroid 解绑：

```shell
sudo mkdir -p /var/lib/waydroid/overlay/system/usr/keylayout/
sudo cp /var/lib/waydroid/rootfs/system/usr/keylayout/Generic.kl /var/lib/waydroid/overlay/system/usr/keylayout/Generic.kl
sudo sed -i "s/key 125/# key 125/" /var/lib/waydroid/overlay/system/usr/keylayout/Generic.kl
```

因为 Waydroid 的 rootfs 是在运行时挂载且似乎无法修改，所以使用 overlay 进行覆盖

`key 125 META_LEFT` 即为左 Win 键的定义，也可以在其中修改其他按键

修改完成后重启 Waydroid 即可
