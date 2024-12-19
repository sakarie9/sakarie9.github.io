---
date: "2024-12-19T15:33:43+08:00"
title: "Temporary Workspace with Zram"
tags: [Linux]
categories: 技术分享
summary: Using Zram to create a fast temporary workspace in memory
---

## zram-generator

安装 [zram-generator](https://github.com/systemd/zram-generator)

```bash
sudo pacman -S zram-generator
```

修改配置，根据实际内存大小调整 `zram-size`

`sudo -e /etc/systemd/zram-generator.conf`

```plain
[zram0]
compression-algorithm = lz4 zstd(level=1) (type=idle)
zram-size = 16384

[zram1]
compression-algorithm = lz4
zram-size = 32768
fs-type = ext4
mount-point = /tmpfs
options = "noatime,discard,X-mount.mode=1777"
```

重启后 zram-generator 会自动创建 zram 设备，并将 zram1 挂载到 `/tmpfs` 目录

## 文件系统优化

> 对于 zram 设备上的文件系统，日志等特性几乎无用，并且会占用额外的空间。
>
> ext2 没有日志，但它比 ext4 等文件系统老得多，并且有一些限制。
>
> btrfs 非常现代，但他仍然有太多我们不需要的特性。
>
> 因此我们使用无日志的 ext4 文件系统作为我们 zram 设备的文件系统。

在 make2fs.conf 中添加新的 fs 类型，复制 ext4 配置，并删除 `has_journal` 选项，给新的 fs-type 起一个唯一的名字，比如 `ext4withoutjournal`。

`sudo -e /etc/mke2fs.conf`

```plain
[fs_types]
        ext4 = {
                features = has_journal,extent,huge_file,flex_bg,metadata_csum,64bit,dir_nlink,extra_isize
        }
        ext4withoutjournal = {
                features = extent,huge_file,flex_bg,metadata_csum,64bit,dir_nlink,extra_isize
        }
```

zram-generator 会使用 mkfs.\* 来格式化 zram 设备，因此我们需要创建一个 mkfs.ext4withoutjournal 的软链接。

`sudo ln -s /usr/bin/mke2fs /usr/bin/mkfs.ext4withoutjournal`

然后在 `/etc/systemd/zram-generator.conf` 中修改 fs-type 为 `ext4withoutjournal`。

```plain
[zram1]
compression-algorithm = lz4
zram-size = 32768
fs-type = ext4withoutjournal
mount-point = /tmpfs
options = "noatime,discard,X-mount.mode=1777"
```

重启后生效。

## 性能

> 以下的性能测试非常不严谨，仅供参考

- lz4：最快，但压缩率最低
- lzo：次之，压缩率稍高
- zstd：最慢，压缩率最高

对于 ext4，关闭日志可以提高约 10% 的读取性能。

## Reference

- [Arch Linux Wiki](https://wiki.archlinux.org/title/Zram)
- [Gentoo Wiki](https://wiki.gentoo.org/wiki/Zram)
