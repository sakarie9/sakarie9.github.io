---
date: 2025-05-07T18:39:33+08:00
title: Arch 应用软件包补丁
tags:
  - Linux
categories: 技术随笔
summary:
---

> 参考 [Patching packages - ArchWiki](https://wiki.archlinux.org/title/Patching_packages)

## 前言

例如，要对 [waybar-git](https://github.com/Alexays/Waybar) 应用一个 尚未合并的 pr (假设为 #114514)

如果不使用 [Arch build system](https://wiki.archlinux.org/title/Arch_build_system)，需要手动构建包，不利于软件包的管理

因此通过修改 PKGBUILD 的方式，对软件包进行 patch

## 步骤

### 获取 patch

对于 GitHub 或者 GitLab，可以在 URL 后附加 `.patch` 来为特定提交或合并/拉取请求生成补丁

例如 `https://github.com/Alexays/Waybar/pull/114514.patch`

### PKGBUILD

从 Arch Linux GitLab 或者 AUR 获得对应包的 PKGBUILD，如果包括其他文件需要一并下载

将获取到的 patch 链接或者 patch 文件添加到将 PKGBUILD 中的 `source` 内，同时添加对应的 checksum

然后应用补丁，在 `prepare()` 中添加

```shell
prepare() {
    cd $pkgname-$pkgver # edit
    patch -Np1 -i ../114514.patch
}
```

### 构建

完成上述步骤和就可以使用 `makepkg` 或者 `extra-x86_64-build` 构建包了

注意如果软件源中的包更新后版本比 patch 过的包更新，则 patch 过的包会被覆盖，需要更新 PKGBUILD 后重新构建
