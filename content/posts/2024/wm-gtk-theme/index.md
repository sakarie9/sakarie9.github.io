---
date: "2024-11-23T19:35:45+08:00"
title: "WM 下的 GTK 主题设置"
tags: [Linux, Unixporn]
categories: 技术分享
---

### GTK 2 & 3

使用 [nwg-look](https://github.com/nwg-piotr/nwg-look) 应用 [adw-gtk3](https://github.com/lassekongo83/adw-gtk3) 主题

安装所需的包 `pacman -S nwg-look adw-gtk-theme`

在 nwg-look 中的组件一栏选择 `adw-gtk3-dark`，右侧颜色方案选择倾向暗色

也可以根据需要设置图标主题和鼠标光标

点击应用，保存设置

### GTK4

保持默认，不需要在 `~/.config/gtk-4.0/` 下进行配置

### libadwaita

想要使用 libadwaita 的应用，例如 nautilus，正确使用暗色，需要进行一些配置

1. 在 sway 下，需要安装 `xdg-desktop-portal-{gtk,wlr}`

   `-gtk` 是必须的，参考 [i3/#5896](https://github.com/i3/i3/discussions/5896#discussioncomment-8556941)

   在 Arch Linux 上，似乎并不需要上述 issue 的配置，只需要安装 `xdg-desktop-portal-gtk`

2. 正确设置环境变量

   [sway wiki](https://github.com/swaywm/sway/wiki#systemd-and-dbus-activation-environments)

   需要如下配置以正确设置环境变量

   ```conf
   exec systemctl --user import-environment WAYLAND_DISPLAY DISPLAY XDG_CURRENT_DESKTOP SWAYSOCK I3SOCK XCURSOR_SIZE XCURSOR_THEME
   ```
