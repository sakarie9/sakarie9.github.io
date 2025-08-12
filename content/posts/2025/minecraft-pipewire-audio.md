---
date: 2025-08-12T17:41:46+08:00
title: Minecraft 在 Pipewire 上没有声音的问题
tags:
  - Games
  - Linux
categories: 技术随笔
summary:
---

有些特定版本的 Minecraft 可能会在 Linux/Pipewire 中存在没有声音或者直接崩溃的问题

OpenAL 会优先使用 JACK 而非 PipeWire 的 PulseAudio 后端，将其指定为 PulseAudio 可解决此问题

新建 `/etc/openal/alsoft.conf` (或 `~/.alsoftrc`)

```
drivers=pulse
```

或者使用环境变量 `ALSOFT_DRIVERS=pulse` 也用同样的作用

参考：[ArchWiki](https://wiki.archlinux.org/title/Minecraft#Audio_stutters_on_PipeWire_or_Java_crashes_with_SIGFPE)
