---
date: 2025-12-30T22:31:52+08:00
title: 覆盖 fcitx5 的托盘图标
tags:
  - Linux
  - Unixporn
categories: 技术随笔
summary:
---

## 前言

在使用 `Papirus` 图标主题时，在 `noctalia-shell` 下面，会出现暗色主题下，`fcitx5` 的托盘图标显示为暗色的情况（`waybar` 正常）

错误情况（暗底暗图标）：

![fcitx5-icon-1.webp](fcitx5-icon-1.webp)

正确情况（暗底亮图标）：

![fcitx5-icon-2.webp](fcitx5-icon-2.webp)

## 修复

由于在 `~/.local/share/icons` 下直接放置 `input-keyboard-symbolic.svg` 无效，需要使用自定义主题以覆盖此图标

1 . 建立图标目录

`mkdir ~/.local/share/icons/Papirus-Dark-Fcitx/24x24/devices`

2 . 新建 theme 文件

```
nvim ~/.local/share/icons/Papirus-Dark-Fcitx

[Icon Theme]
Name=Papirus-Dark-Fcitx
Comment=Custom Fcitx5 overrides for Papirus-Dark
Inherits=Papirus-Dark,Adwaita,hicolor

# 定义目录
Directories=24x24/devices

[24x24/devices]
Context=Devices
Size=24
Type=Fixed
```

3 . 复制图标

```
wget https://github.com/PapirusDevelopmentTeam/papirus-icon-theme/raw/refs/tags/20250201/Papirus-Dark/symbolic/devices/input-keyboard-symbolic.svg -O ~/.local/share/icons/Papirus-Dark-Fcitx/24x24/devices/input-keyboard-symbolic.svg
```

完成后在 `QT_QPA_PLATFORMTHEME` 所定义的主题 (`qt6ct`, `qt5ct`) 中选择 `Papirus-Dark-Fcitx`，重启 `quickshell` 即可
