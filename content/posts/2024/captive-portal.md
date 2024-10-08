---
title: 修改 Captive Portal
date: 2024-04-10
summary: 修改 Captive Portal 解决类原生“无网络”的情况
tags: [Android, Network]
categories: 技术随笔
---

修改 Captive Portal 解决类原生“无网络”的情况

```bash
adb shell settings put global captive_portal_http_url http://connect.rom.miui.com/generate_204

adb shell settings put global captive_portal_https_url https://connect.rom.miui.com/generate_204
```
