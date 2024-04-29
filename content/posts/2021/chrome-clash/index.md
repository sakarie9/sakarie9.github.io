---
title: Chrome+SwitchyOmega+Clash的处理
date: 2021-04-03 12:20:44
categories: 技术分享
tags: [Windows, Network]
index_img: /img/posts/Clash.webp
banner_img: /img/posts/Clash.webp
---

## 症状

在使用Chrome+SwitchyOmega+Clash的环境下，SwitchyOmega的“代理”操作会将代理请求发送至Clash，由Clash判断是否使用代理。这样就造成了用户想要某个域名走代理，但是Clash规则中默认该域名直连，最终还是直连的情况。

<!-- more -->

## 解决方案

### Clash配置

配置Clash将来自Chrome的流量全部使用代理

进入Clash的Settings页面，选择Profiles下的Parsers，编辑配置文件预处理，参考 [配置文件预处理](https://docs.cfw.lbyczf.com/contents/parser.html)

其中`URL`修改为配置文件的URL，`节点选择`修改为配置文件内的`proxy-groups`下的`name`，根据情况选择

```yaml
parsers: # array
  - url: URL
    yaml:
      prepend-rules:
        - PROCESS-NAME,chrome.exe,节点选择
```

### SwitchyOmega配置

设置自动切换规则，可以参考 [Proxy SwitchyOmega](https://proxy-switchyomega.com/settings/) 配置

注意在SwitchyOmega内设置为直连的域名将不经过Clash直接访问，设置为代理的域名将无视Clash规则直接代理
