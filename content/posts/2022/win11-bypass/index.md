---
title: 手动绕过 Windows 11 安装硬件限制
date: 2022-01-20 12:54:34
categories: 技术分享
tags: [Windows]
index_img: /img/posts/win11-bypass-1.webp
banner_img: /img/posts/win11-bypass-1.webp
---

绕过使用原版 Windows 11 镜像安装时的硬件要求（TPM/SecureBoot）

<!-- more -->

### 使用 Windows 镜像启动
   
   U盘写入镜像或虚拟机挂载镜像启动

### 打开命令提示符
   
   在 Windows 安装程序的语言选择界面，按下 `Shift+F10` 打开 CMD 窗口

   ![](/img/posts/win11-bypass-2.webp)

### 编辑注册表
   
   1. 在 CMD 中输入 `regedit` 打开注册表编辑器
   
   2. 进入 `HKEY_LOCAL_MACHINE\SYSTEM\Setup`
   
   3. 右键 `Setup` 选择 `新建-项`，重命名为 `LabConfig`
   
   4. 点击 `LabConfig`，在右面的窗口右键新建一共三个 `DWORD（32位）值`，重命名如下，并将他们的数值数据改为 `1`

    - BypassTPMCheck

    - BypassRAMCheck

    - BypassSecureBootCheck

   ![](/img/posts/win11-bypass-3.webp)

   5. 关闭 CMD 和 注册表编辑器


### 继续安装系统

   ![](/img/posts/win11-bypass-4.webp)

