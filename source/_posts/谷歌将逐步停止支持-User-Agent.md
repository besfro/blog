---
title: 谷歌将逐步停止支持 User Agent
date: 2020-04-03 11:22:30
category: news
tags: [user agent,chrome]
---

谷歌将逐步停止支持 User Agent, Chrome 将从 v81 开始逐步淘汰 User Agent, 2020 年 9 月中旬发布的 v85 预计将会完全移除用户代理字符串

<!-- more -->

每一个 HTTP 请求, 都会将 User Agent 作为请求头的一部分发送到服务器。但是这个头字段现在变得越来越长和难以理解  
在 Chrome on iOS 被标识为
```
User-Agent: Mozilla/5.0 (iPhone; CPU iPhone OS 12_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) CriOS/69.0.3497.105 Mobile/15E148 Safari/605.1
```
在 Chrome on Android 被标识为
```
User-Agent: Mozilla/5.0 (Linux; Android 9; Pixel 2 XL Build/PPP3.180510.008) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.87 Mobile Safari/537.36
```
这真是一大串字符, 包含了很多信息。 Safari 采取了一些措施对该字符串进行删减, Chrome 使用了另一种方式, 具体有以下改变 

- 用户代理将冻结
  - navigator.appVersion
  - navigator.platform
  - navigator.productSub
  - navigator.vendor
  - navigator.userAgent  
  - -
- 引入几个头部字段 
  - Sec-CH-UA: "Chrome"; v="73"
  - Sec-CH-UA-Platform: "Windows"
  - Sec-CH-UA-Arch: "ARM64"
  - Sec-CH-UA-Model: "Pixel 2 XL"
  - **Sec-CH-UA-Mobile: ?1**
  - Sec-CH-UA-Full-Version: "73.1.2343B.TR"  
  - -
- **navigator.getUserAgent()** 将被注入到 Javascript APIs


2020 年 9 月中旬发布的 Chrome85 预计将会完全移除用户代理字符串。其它浏览器供应商，包括 Mozilla Firefox、Microsoft Edge 以及 Apple Safari，都表示支持这一举措。但目前还不清楚他们何时会采取行动。




### 相关文章
[Infoq - Chrome Phasing out Support for User Agent](https://www.infoq.cn/article/A3flGDTKpszgHKRQoQMQ)
[WIGG ua-client-hints](https://github.com/WICG/ua-client-hints)
[Browser_detection_using_the_user_agent](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Browser_detection_using_the_user_agent)