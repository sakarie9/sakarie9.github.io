---
date: "2024-09-10T17:05:18+08:00"
title: "Waydroid 疑难排解"
tags: [Linux, Android]
categories: 技术分享
---

### Waydroid

Waydroid 的安装参考 [Arch Wiki](<(https://wiki.archlinux.org/title/Waydroid)>)

### 内核模块

需要带有 Binder 内核模块的内核才能运行 Waydroid。

在 Arch Linux 上可以通过安装 `linux-zen` 包来满足需求。

或者使用 DKMS 模块，参考 [DKMS modules](https://wiki.archlinux.org/title/Waydroid)，如果安装了 `linux-zen` 等内核则无需安装 DKMS 模块。

### 无法启动

如果 Waydroid 无法启动，可能是遇到了 [[BUG] Waydroid is not working anymore on my Arch Linux system](https://github.com/waydroid/waydroid/issues/1319)，需要对 `waydroid-image-gapps` 进行降级。

已知的最新的可以正常运行的版本为 `2024-02-17`，从 [SourceForge](https://sourceforge.net/projects/waydroid/files/images/) 获取。

### 花屏

Waydroid 需要使用和桌面混成同样的 GPU，当使用了不同的 GPU 时就会出现渲染错误。

使用 [waydroid-choose-gpu.sh](https://github.com/Quackdoc/waydroid-scripts/blob/main/waydroid-choose-gpu.sh) 指定 Waydroid 使用的 GPU。

### 转译

使用 [Waydroid Script](https://github.com/casualsnek/waydroid_script) 安装转译层。

一般在 AMD 处理器上，`libndk` 运行较好；Intel 处理器上 `libhoudini` 运行较好。

如果出现问题可以尝试安装另一个，更换转译层之前最好将旧的转译层卸载。

在某些游戏，如 Blue Archive 日服和国际服，会出现卡主界面/直接闪退的情况，需要对转译层二进制文件打补丁。

#### libndk

> <https://github.com/waydroid/waydroid/issues/788#issuecomment-2167334937>

- 下载 [scripton_ndk.txt](https://github.com/user-attachments/files/15832700/scripton_ndk.txt)
- 重命名为 `scripton_ndk.sh`，并修改为可执行
- `sudo ./scripton_ndk.sh`

<details>
  <summary> scripton_ndk </summary>

```Bash
#!/bin/bash

# <https://github.com/waydroid/waydroid/issues/788#issuecomment-2167334937>

function CheckHex {

# file path, Ghidra offset, Hex to check

commandoutput="$(od $1 --skip-bytes=$(($2 - 0x101000)) --read-bytes=$((${#3} / 2)) --endian=little -t x1 -An file | sed 's/ //g')"
  if [ "$commandoutput" = "$3" ]; then
echo "1"
else
echo "0"
fi
}

function PatchHex {

# file path, ghidra offset, original hex, new hex

file_offset=$(($2 - 0x101000))
  if [ $(CheckHex $1 $2 $3) = "1" ]; then
    hexinbin=$(printf $4 | xxd -r -p)
    echo -n $hexinbin | dd of=$1 seek=$file_offset bs=1 conv=notrunc
tmp="Patched $1 at $file_offset with new hex $4"
echo $tmp
elif [ $(CheckHex $1 $2 $4) = "1" ]; then
echo "Already patched"
else
echo "Hex mismatch!"
fi
}

ndk_path="/var/lib/waydroid/overlay/system/lib64/libndk_translation.so"

if [ -f $ndk_path ]; then
if [ -w ndk_path ] || [ "$EUID" = 0 ]; then
PatchHex $ndk_path 0x307dd1 83e2fa 83e2ff
PatchHex $ndk_path 0x307cd6 83e2fa 83e2ff

else
echo "libndk_translation is not writeable. Please run with sudo"
fi
else
echo "libndk_translation not found. Please install it first."
fi

```

</details>

#### libhoudini

> <https://github.com/waydroid/waydroid/issues/788#issuecomment-2162386712>

- 下载 [scripton.txt](https://github.com/user-attachments/files/15800844/scripton.txt)
- 重命名为 `scripton.sh`，并修改为可执行
- `sudo ./scripton.sh`

<details>
  <summary> scripton_ndk </summary>

```Bash
#!/bin/bash

function CheckHex {
  #file path, Ghidra offset, Hex to check
  commandoutput="$(od $1 --skip-bytes=$(($2 - 0x100000)) --read-bytes=$((${#3} / 2)) --endian=little -t x1 -An file | sed 's/ //g')"
  if [ "$commandoutput" = "$3" ]; then
    echo "1"
  else
    echo "0"
  fi
}

function PatchHex {
  #file path, ghidra offset, original hex, new hex
  file_offset=$(($2 - 0x100000))
  if [ $(CheckHex $1 $2 $3) = "1" ]; then
    hexinbin=$(printf $4 | xxd -r -p)
    echo -n $hexinbin | dd of=$1 seek=$file_offset bs=1 conv=notrunc
    tmp="Patched $1 at $file_offset with new hex $4"
    echo $tmp
  elif [ $(CheckHex $1 $2 $4) = "1" ]; then
    echo "Already patched"
  else
    echo "Hex mismatch!"
  fi
}

houdini_path="/var/lib/waydroid/overlay/system/lib64/libhoudini.so"

if [ -f $houdini_path ]; then
  if [ -w houdini_path ] || [ "$EUID" = 0 ]; then
    PatchHex $houdini_path 0x4062a5 48b8fbffffff 48b8ffffffff
    PatchHex $houdini_path 0x4099d6 83e0fb 83e0ff
    PatchHex $houdini_path 0x409b42 e8892feeff 9090909090
  else
    echo "Libhoudini is not writeable. Please run with sudo"
  fi
else
  echo "Libhoudini not found. Please install it first."
fi
```

</details>
