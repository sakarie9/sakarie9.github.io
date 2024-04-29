---
title: Windows 下为 PowerShell/CMD/Git 设置代理
date: 2021-03-17 17:01:51
categories: 技术分享
tags: [Windows, Git, Network]
index_img: /img/posts/proxy.webp
banner_img: /img/posts/proxy.webp
---

## PowerShell

### 1. 在 `PowerShell` 窗口中运行如下指令

```bash
if (!(Test-Path -Path $PROFILE )) { New-Item -Type File -Path $PROFILE -Force }
notepad $PROFILE
```

### 2. 在打开的记事本中添加如下代码

注意修改第九行代理地址

```bash
# Set-Proxy command
if ($env:HTTP_PROXY -ne $null){
    Write-Output "Proxy Enabled as $env:HTTP_PROXY";
}

Function SetProxy() {
    Param(
        # 改成自己的代理地址
        $Addr = 'http://127.0.0.1:19810',
        [switch]$ApplyToSystem
    )

    $env:HTTP_PROXY = $Addr;
    $env:HTTPS_PROXY = $Addr;
    $env:http_proxy = $Addr;
    $env:https_proxy = $Addr;

    [Net.WebRequest]::DefaultWebProxy = New-Object Net.WebProxy $Addr;
    if ($ApplyToSystem) {
        $matchedResult = ValidHttpProxyFormat $Addr;
        # Matched result: [URL Without Protocol][Input String]
        if (-not ($matchedResult -eq $null)) {
            SetSystemProxy $matchedResult.1;
        }
    }
    Write-Output "Successful set proxy as $Addr";
}
Function ClearProxy() {
    Param(
        $Addr = $null,
        [switch]$ApplyToSystem
    )

    $env:HTTP_PROXY = $Addr;
    $env:HTTPS_PROXY = $Addr;
    $env:http_proxy = $Addr;
    $env:https_proxy = $Addr;

    [Net.WebRequest]::DefaultWebProxy = New-Object Net.WebProxy;
    if ($ApplyToSystem) { SetSystemProxy $null; }
    Write-Output "Successful unset all proxy variable";

}
Function SetSystemProxy($Addr = $null) {
    Write-Output $Addr
    $proxyReg = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings";

    if ($Addr -eq $null) {
        Set-ItemProperty -Path $proxyReg -Name ProxyEnable -Value 0;
        return;
    }
    Set-ItemProperty -Path $proxyReg -Name ProxyServer -Value $Addr;
    Set-ItemProperty -Path $proxyReg -Name ProxyEnable -Value 1;
}
Function ValidHttpProxyFormat ($Addr) {
    $regex = "(?:https?:\/\/)(\w+(?:.\w+)*(?::\d+)?)";
    $result = $Addr -match $regex;
    if ($result -eq $false) {
        throw [System.ArgumentException]"The input $Addr is not a valid HTTP proxy URI.";
    }

    return $Matches;
}
Set-Alias set-proxy SetProxy
Set-Alias clear-proxy ClearProxy
```

### 3. 保存，重新打开PowerShell

设置当前窗口代理 ：`set-proxy`

设置当前窗口代理 + 系统代理：`set-proxy -ApplyToSystem`

取消当前窗口代理：`clear-proxy`

取消当前窗口代理 + 系统代理：`clear-proxy -ApplyToSystem`

`set-proxy`和`SetProxy`，`clear-proxy`和`ClearProxy`可以互相替换

## CMD

设置代理，窗口关闭后失效

```bash
set http_proxy=http://127.0.0.1:19810
set https_proxy=http://127.0.0.1:19810
```

## Git

设置Http代理，永久生效

```bash
git config --global https.proxy https://127.0.0.1:19810
git config --global http.proxy http://127.0.0.1:19810
```

取消代理

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```
