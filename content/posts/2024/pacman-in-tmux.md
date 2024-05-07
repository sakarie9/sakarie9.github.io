---
date: "2024-05-07T16:29:35+08:00"
title: "在 tmux 中进行 Arch Linux 的滚动更新"
tags: [Linux]
categories: 技术分享
---

在 tmux 中进行 Arch Linux 的滚动更新

<!-- more -->

## 前言

在 tmux 中进行滚动更新，比直接在终端中直接滚动有着更多好处：

- 防止更新过程中误关闭终端
- 更新后关闭终端也能重新连接回去查看更新日志

但是如果不是重度 tmux 用户，更新前手动进入 tmux 再更新显得有些繁琐

因此本文提出一个在普通终端下直接唤起 tmux 并运行更新命令的脚本，此脚本有以下特点：

- 在终端内运行脚本，如果对应的 Session 不存在，则新建一个 tmux Session 并运行命令
- 如果对应的 Session 已经存在，则直接运行命令，并连接回此 Session
- 如果已经在 tmux 内，直接运行命令

{{< mermaid >}}

graph LR

A[运行脚本]
B[Attach tmux Session]
BN[New tmux Session]
C[Command]

A -- Session 不存在 --- BN
A -- Session 存在 --- B
A -- 已经在Session中 --- C
B --> C
BN --> C

{{< /mermaid >}}

## 使用

其中的 `zsh` 是为了在命令运行完毕后不退出，根据自己使用的替换成 `bash` 等 Shell

```bash
#!/usr/bin/env bash

session_name="paru"
cmd="paru -Syu"

# 检查是否存在的tmux session
if tmux has-session -t $session_name 2>/dev/null; then
  # 检查是否在tmux内
  if [ -n "$TMUX" ]; then
    # 在 tmux 内
    bash -c "$cmd"
  else
    # 不在 tmux 内，则连接到此session并运行命令
    tmux send-keys -t $session_name "$cmd" Enter
    tmux attach-session -t $session_name
  fi
else
  # 如果不存在，则新建session并运行命令
  tmux new-session -s $session_name "${cmd}; zsh"
fi
```
