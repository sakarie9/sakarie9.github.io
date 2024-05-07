---
title: Heroku+Cloudflare 免费账号实现24/7在线及域名绑定
date: 2022-07-23 11:45:14
categories: 技术分享
tags: [Cloudflare]
---

> **2022/12/28, Heroku 将结束免费方案，此教程同步作废**

<!-- more -->

Heroku 的免费方案下，单个用户所有 app 每月的总运行时长不超过550小时，且不能绑定自己的域名，如果需要全天在线需要绑定银行卡，对于白嫖用户相当不友好。但是我们可以给 Heroku 套上一层 Cloudflare 来实现全月在线和域名绑定


## 原理

使用 Cloudflare Workers 对 Heroku 进行反代并手动实现简单的负载均衡

## 需求

- 一个使用 Cloudflare 进行域名解析的域名
- 两个 Heroku 账户（无需验证信用卡）

## 配置

### Heroku 配置

在两个账号下分别配置两个项目，切勿配置在同一账户下，免费时间是以账户为单位而非实例。配置过程略，根据自己需要选择，最后需要两个这样的 URL `https://******.herokuapp.com/`

### Cloudflare 配置

1. 添加 DNS 记录，类型 `AAAA`，名称自定，IPv6 地址为 `100::`，代理状态打勾，保存

2. Workers - 管理 Workers - 创建服务，服务名称自定，这里假设为 `heroku-dns`，创建服务

3. 点击快速编辑，填入以下代码，手动替换 url1 和 url2，保存并部署
    ```js
    addEventListener(
        "fetch",event => {
            let url=new URL(event.request.url);     
            let localDate = new Date();
            let day = localDate.getDate();

            if (day<15) //每月前十五天使用url1,后十五天使用url2
            {
                url.hostname="******.herokuapp.com"; //url1
            }
            else
            {
                url.hostname="******.herokuapp.com"; //url2
            }
            //console.log(`Request url: ${url.hostname}`);
            let request=new Request(url,event.request);    
            event. respondWith(       
                fetch(request)    
            )  
        }
    )
   ```

4. Workers - HTTP 路由 - 添加路由，`路由` 处填写 `名称.域名.顶级域名/*`，名称处填写第一步中 DNS 记录里填写的名称，域名填写当前在 Cloudflare 添加解析的域名，别忘了后面的 `/*`；服务选择第二步新建的 `heroku-dns`，环境 `production`，保存

5. 此时访问域名就能访问到部署在 Heroku 上的服务，可以在 Workers 的编辑界面测试，取消掉 `console.log` 行的注释，发送请求就能在控制台看到当前使用的是哪个 URL

## 最后

单个账户的 550 小时用半个月是完全没有问题的，可以使用 `uptime` 等监控程序每15分钟一次使 Heroku 永不休眠，避免唤醒时的巨大访问延迟
