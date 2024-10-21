---
title: Linux 光标主题配置
date: 2022-12-24 13:40:10
categories: 技术分享
tags: [Linux, Unixporn]
summary: WM 下配置系统全局光标主题
---

WM 下配置系统全局光标主题

<!-- more -->

## 安装

以 [`Catppuccin-Cursors`](https://github.com/catppuccin/cursors/) 为例

参考 [Installation](https://github.com/catppuccin/cursors#installation)

```shell
paru -S catppuccin-cursors-frappe
```

## 配置

### nwg-look

使用 [nwg-look](https://github.com/nwg/nwg-look) 配置

在`鼠标光标`处选择目标主题，应用

### 环境变量

```shell
export XCURSOR_THEME=catppuccin-frappe-light-cursors
export XCURSOR_SIZE=32
```

### GTK

若已使用 nwg-look 配置,则无需修改

```shell
gsettings set org.gnome.desktop.interface cursor-theme catppuccin-frappe-light-cursors
gsettings set org.gnome.desktop.interface cursor-size 32
```

<details>

<summary>过时的内容</summary>

## 下载

以 [`Catppuccin-Frappe-Light-Cursors`](https://github.com/catppuccin/cursors/) 为例

从仓库下载 `Catppuccin-Frappe-Light-Cursors.zip`，解压至 `~/.local/share/icons`

```

icons
├── Catppuccin-Frappe-Light-Cursors
│   ├── cursors
│   └── index.theme

```

## 配置

### Qt

`~/.icons/default/index.theme`

```

[icon theme]
Inherits=Catppuccin-Frappe-Light-Cursors

```

### 环境变量

`~/.zlogin`

```

export XCURSOR_THEME=Catppuccin-Frappe-Light-Cursors
export XCURSOR_SIZE=32

```

### GTK

```

gsettings set org.gnome.desktop.interface cursor-theme Catppuccin-Frappe-Light-Cursors
gsettings set org.gnome.desktop.interface cursor-size 32

```

### Hyprland

`~/.config/hypr/hyprland.conf`

```

exec-once = hyprctl setcursor Catppuccin-Frappe-Light-Cursors 32

```

</details>
