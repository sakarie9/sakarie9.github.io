---
title: 批量修改 Git 历史中的用户邮箱
date: 2024-02-04
summary: 如何批量修改Git仓库中的提交作者邮箱。通过使用git filter-repo命令，并需要强制推送才能修改远程仓库。
tags: [Git]
categories: 技术随笔
---

批量修改 Git 仓库中的 commit author email

## 安装

安装 [`git-filter-repo`](https://github.com/newren/git-filter-repo)

## 修改

将 `E1@example.com` 修改为 `E2@example.com`

```bash
git filter-repo --email-callback 'return email if email != b"E1@example.com"else b"E2@example.com"'
```

## 注意

修改后需要强制推送才能修改远程仓库 `git push -f`