---
date: "2024-08-31T10:36:13+08:00"
title: "使用 FFmpeg 转换音频为 Opus：避坑指南与实战经验"
tags: [Linux, Audio]
categories: 技术分享
---

如何使用 FFmpeg 将音频文件转换为 Opus，以及转换过程中遇到的各种坑。

<!-- more -->

---

> **听不出来就是无损！**

---

## 背景

> ~~TL;DR! 可以直接跳到 [脚本](#脚本)~~

近来手机和 NAS 存储空间逐渐不够用了，在想办法处理那些 WAV 文件，将其转换为 FLAC 以压缩空间的时候，想到了 Opus 这个效率很高的编码格式。

虽然把所有无损音频直接转换为有损音频保存收藏不可取，但把手机内保存的音乐统统使用压缩率更好的格式是一个不错的尝试。

首先介绍一下本篇的主角：[Opus](https://wiki.hydrogenaud.io/index.php?title=Opus)

> Opus是一个有损音频压缩的数字音频编码格式，由Xiph.Org基金会开发。Opus集成了两种声音编码的技术：以语音编码为导向的SILK和低延迟的CELT。Opus可以无缝调节高低比特率。在编码器内部它在较低比特率时使用线性预测编码在高比特率时候使用变换编码（在高低比特率交界处也使用两者结合的编码方式）。Opus具有非常低的算法延迟（默认为22.5 ms）[5]，非常适合用于低延迟语音通话的编码，像是网络上的即时声音流、即时同步声音旁白等等，此外Opus也可以透过降低编码比特率，达成更低的算法延迟，最低可以到5 ms。在多个听觉盲测中，Opus都比MP3、AAC、HE-AAC等常见格式，有更低的延迟和更好的声音压缩率。

总之就是非常现代化的一种编码格式，能够在低码率下保持极高的清晰度（与同码率的 MP3 相比）；在中等的码率下（96kbps-128kbps 左右）能够做到接近无损的听感；在更高的码率下（160kbps-192kbps）几乎可以认为无损。

### 听觉测试

在调查是否应该选用 Opus 时，进行了一些简单的听觉测试，选取了两首电吉他声占主导且非常清晰的音乐作为样本。

> 本测试仅娱乐，非常不科学

|     | 1         | 2        | 3        | 4         |
| --- | --------- | -------- | -------- | --------- |
| A   | 96k-opus  | 320k-mp3 | 192k-mp3 | 160k-opus |
| B   | 160k-opus | 128k-mp3 | 320k-mp3 | 96k-opus  |

A 组的样本均由 FLAC 转换而来，B 组的样本均由 B3 的 320kbps 的 MP3 转换而来。

以 MP3 为源文件是为了测试从有损到有损的转换过程中会丢失多少细节，也就是所谓的 “Generation loss” 有多严重。

### 测试结论

> 甲：完全听不出来
>
> 乙：A3＞A2＞A4＞A1，A1、A4、B1、B4 有点糊，事后又添加了 320k-opus 的 A5，依然糊，推测可能是设备的解码器存在问题
>
> 丙：A3 最清楚，A4 最糊

虽然测试非常之不严谨，但我们也能看出来至少 192k-mp3 和 320k-mp3 大家都是分不出来的。

因为测试所用的网站是临时搓出来的，乙和丙的设备上只有 MP3 格式的样本会显示音频时长，Opus 的样本似乎是串流过去的，这会对音质造成多少影响暂时存疑。

但总之，个人是完全听不出来 96k-opus 和 320k-mp3 的区别的。所以转码策略就是：存在无损文件的，转为 128kbps 或者 160kbps 的 Opus；源文件只有有损的，统统转为 96kbps Opus。

这里引用一句话：

> Subjectively, though, if you can't hear the difference, then by definition there was no quality loss, which is the whole point of these codecs: fooling you in that way.

## 编写脚本

使用 FFmpeg 进行音频转码理论上是完全没有难度的，`ffmpeg -i input.flac output.opus` 理论上就能直接完成了。

しかし、

だが、

Butt、

这其中存在着很多很多的坑，下面我们从头开始一步步踏进每一个坑。

### 多线程并行

众所周知，FFmpge 是没有内置的同时处理多文件、或者利用多核心的方式的，在我们处理上千个音频文件的时候会非常非常慢。

所以我们需要用 GNU Parallel 吃满 CPU，一个简单的用法如下。

```bash
find . -type f -name "*.flac" | parallel ffmpeg -i {} {.}.opus
```

将当前目录下的 FLAC 转换为 Opus，乍一看是没问题的，他也确实可以工作。

但是其存在一个非常严重的问题，目前 FFmpeg 不支持 OGG/Opus 文件的专辑封面写入。

### 专辑封面

此事在 [#4448 Support writing album cover art image embedded in ogg / opus metadata](https://trac.ffmpeg.org/ticket/4448) 中亦有记载。

很难想象一个九年前就存在的问题直到今日依然未解决。

就跟上述 Issue 描述的一样，FFmpeg 无法为 Opus 文件嵌入专辑封面，所以我们需要另辟蹊径。

经过一番搜索，发现了使用 `opusenc` 命令进行转换可以保留所有的元数据以及专辑封面。

简单用法如下：

```Bash
ffmpeg -i input.mp3 -f flac - | opusenc - output.opus
```

将 MP3 先转为 FLAC，再通过管道由 opusenc 转为 Opus，这样就跳过了 FFmpeg 直接转为 Opus 的过程。

不直接使用 opusenc 的原因是他只支持几种格式：`The input format can be Wave, AIFF, FLAC, Ogg/FLAC, or raw PCM.`，所以如果需要转换 MP3 到 Opus 时需要先转为 FLAC。

然后这种方式由引入了一个新的问题，有些文件转换为 Opus 后，体积甚至比原来的 MP3 大上了 50%！

经过研究，发现多出来的大小来自专辑封面。似乎 FFmpeg 在将源文件转为 FLAC 的时候顺手将专辑封面也处理了。

原本 MP3 内嵌的 3000x3000 785.4kB JPEG 的专辑封面，转换到 Opus 之后，内嵌封面变成了 3000x3000 12.0MB PNG。

这导致体积从原来的 8.1MB 暴涨到了 23.5MB，本来就是为了节省空间才进行的转码，这显然是不可接受的。

想要转码时不处理封面，需要使用 `-c:v copy`，修改后的命令如下：

```Bash
ffmpeg -i input.mp3 -c:v copy -f flac - | opusenc - output.opus
```

这样就能做到从 MP3 到 Opus 的转码了，除此之外还存在着源文件内的封面就已经很大的情况，我们暂且按下不表，在后续步骤处理。

### 结构化输出

为了将源文件转换后输出到指定的目标目录，需要对目录进行一些拼接。

但是问题来了，我在这里遇到了许多乱七八糟的引号嵌套问题，总之直接放解决的输出的脚本在这里。

```Bash
convert_to_opus() {
  input_file=$1
  relative_path="${input_file#"$SOURCE_DIR"/}"
  output_file="$DEST_DIR/${relative_path%.*}.opus"
  # 已存在则跳过
  if [ -f "output_file" ]; then return 0; fi
  # 创建输出文件所在的目录
  output_file_dir=$(dirname "$output_file")
  mkdir -p "$output_file_dir"
  # -c:v copy is REQUIRED as FFmpeg will convert the album cover, cause extremely large files
  ffmpeg -hide_banner -loglevel warning -i "$input_file" -c:v copy -f flac - | opusenc --quiet --bitrate 128 - "$output_file"
}
# IMPORTANT
export -f convert_to_opus
find "$SOURCE_DIR" -type f -name "*.flac" |
  parallel --progress convert_to_opus
```

将 parallel 的 command 换成函数能避免使用字符串传命令造成的 `globbing and word splitting` 问题。

不要忘了 `export -f` 命令，否则 `parallel` 无法识别函数。

到这里已经差不多能用了，我们还有最后一个问题需要解决。

### 专辑封面压缩

专辑封面压缩。

为什么要压缩呢，举个例子，对于 100M 的 FLAC 而言，5M 的专辑封面只占其体积的 5%，不是什么大事，不缺这点体积。

但是我们为了压榨空间把他转为 Opus 后，专辑封面占的空间相比实际音频就大的难以接受了，100M 的 FLAC 转为 Opus 后音频部分仅有 8M 左右，如果直接将封面嵌进去，8+5=13M 的音频文件里 40% 的体积都用来存图片了，显然是非常不合理的。

因此我们要在嵌入之前对专辑封面进行一次压缩，音频都压了不缺这点质量损失。

首先我们的流程是：从源文件提取出专辑封面 -> 压缩 -> 将源文件的音频部分和压缩后的封面合并转换为 FLAC（原因[如上](#专辑封面)） -> 使用 opusenc 转为 opus。

得到了下面的命令

```Bash
  ffmpeg -i "$input_file" -an -vcodec copy -f image2pipe - |
    ffmpeg -i - -vf "scale='if(gt(iw,1000),1000,iw)':-1" "$cover_convert_file"
  # 转换并嵌入封面
  ffmpeg -i "$input_file" -i "$cover_convert_file" -map 0:a -map 1:v -c:v copy -disposition:v attached_pic -metadata:s:v comment="Cover (front)" -f flac - |
    opusenc - "$output_file"
```

其中 `-vf "scale='if(gt(iw,1000),1000,iw)':-1"` 是将专辑封面大于 1000x 的缩小到 1000x，小于 1000x 的不变。

`-map 0:a -map 1:v -c:v copy -disposition:v attached_pic -metadata:s:v comment="Cover (front)"` 这一串都是写入专辑封面的。

注意到其中我们引入了一个中间文件 `$cover_convert_file`，在并行中需要使用合适的路径避免多线程同时读写一个文件造成的冲突，这里将中间文件使用源文件的目录结构和文件名放在临时文件目录中避免冲突。

### 环境变量

到这里脚本差不多已经完成了，但是还有一个问题，在函数中使用变量时需要提前导出变量。

在上面的 `convert_to_opus()` 中使用了 `SOURCE_DIR` 变量，这样就需要在函数中先进行导出，否则函数内此变量不存在就会为空。

```Bash
export SOURCE_DIR
export DEST_DIR
convert_to_opus() {
  input_file=$1
  relative_path="${input_file#"$SOURCE_DIR"/}"
}
```

## 完整脚本

完整的 Bash 脚本。

将 source_dir 中的所有 `.flac,.mp3,.wav` 转为 `.opus`，放入 dest_dir 中。

`./converter.sh source_dir dest_dir`

```Bash
#!/bin/bash

# 检查参数是否足够
if [ $# -ne 2 ]; then
  echo "Usage: $0 <source_directory> <destination_directory>"
  exit 1
fi

SOURCE_DIR="$1"
DEST_DIR="$2"

# 检查源目录是否存在
if [ ! -d "$SOURCE_DIR" ]; then
  echo "Error: Source directory does not exist."
  exit 1
fi

# 创建目标目录（如果不存在）
mkdir -p "$DEST_DIR"

# 导出环境变量
export SOURCE_DIR
export DEST_DIR

convert_to_opus_compress_cover() {
  input_file=$1
  # 计算输出文件的路径
  relative_path="${input_file#"$SOURCE_DIR"/}"
  output_file="$DEST_DIR/${relative_path%.*}.opus"
  # 缓存路径
  CACHE_DIR="/tmp/opus-convert"
  cover_convert_file="$CACHE_DIR/${relative_path%.*}.jpg"
  mkdir -p $CACHE_DIR
  mkdir -p "$(dirname "$cover_convert_file")"
  # 已存在则跳过
  if [ -f "output_file" ]; then return 0; fi
  # 创建输出文件所在的目录
  output_file_dir=$(dirname "$output_file")
  mkdir -p "$output_file_dir"
  # 提取并压缩封面
  ffmpeg -hide_banner -loglevel warning -i "$input_file" -an -vcodec copy -f image2pipe - |
    ffmpeg -hide_banner -loglevel warning -i - -vf "scale='if(gt(iw,1000),1000,iw)':-1" "$cover_convert_file"
  # 转换并嵌入封面
  ffmpeg -hide_banner -loglevel warning \
    -i "$input_file" -i "$cover_convert_file" -map 0:a -map 1:v -c:v copy -disposition:v attached_pic -metadata:s:v comment="Cover (front)" -f flac - |
    opusenc --quiet --bitrate 96 - "$output_file"
  # 清理缓存
  rm "$cover_convert_file"
}

export -f convert_to_opus_compress_cover

find "$SOURCE_DIR" -type f -name "*.flac" -o -name "*.mp3" -o -name "*.wav" |
  parallel --progress convert_to_opus_compress_cover
# 复制其他文件到目标目录
rsync -a --exclude='Scans/' --include='*/' --include='*.lrc' --include='*.jpg' --include='*.png' --include='*.webp' --exclude='*' "$SOURCE_DIR/" "$DEST_DIR/"
```
