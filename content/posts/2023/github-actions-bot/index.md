---
title: 作为 github-actions[bot] 进行 commit
date: 2023-03-10 15:30:10
categories: 技术随笔
tags: [Linux, Git]
---

作为 github-actions[bot] 进行 commit，避免污染 commit 记录

<!-- more -->

### 使用方式

```shell
git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
git config user.name "github-actions[bot]"
git commit
```

### 效果

![Preview](github-actions-bot.jpg)
