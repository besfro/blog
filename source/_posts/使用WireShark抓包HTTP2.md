---
title: 使用 WireShark 抓包 HTTP2
date: 2019-07-04 08:38:13
tags: [HTTP2, WireShark]
categories: posts
---

如果我们要使用 HTTP2, 我们要怎么来对 HTTP2 的流量进行分析呢, 本文将介绍如何配置 WireShark 来抓包 HTTP2

<!-- more -->

#### HTTP2 使用现状

HTTP2 (超文本传输协议2) 2015年5月份正式发表。 HTTP2 保留了 1.1 的大部分语义, 不会破坏现有工作, 同时加入的新特性使程序可以有更好的加载速度  

据 Myssl 的统计, 至6月份国内 HTTP2 的使用站点已经超过了 **50%**

![http2 statistic](/images/posts/http2.statistic.png)


那么抓包 HTTP2 的请求, 可以使用 WireShark 来进行分析调试  
WireShark 是一个强大的网络封包分析工具, 新版 Wireshark 增加了对 HTTP2 的支持
下载地址 [https://www.wireshark.org/#download](https://www.wireshark.org/#download)

#### 配置 WireShark 调式 HTTP2

打开 WireShark, 选择一个网卡  

![capture](/images/posts/capture.jpg)

如果不确定哪个网卡, 在工具栏上点击  **捕获 > 选项**

![capture](/images/posts/capture.net.card.jpg)

WireShare 抓包原理是**读取网卡数据**, 想要解密 HTTPS 数据, 可以通过以下方法配置  
我们知道 TLS 完成握手之后会保存 session ticket 以便恢复时可以快速会话。Chrome 支持将会话密钥保存在外部文件。前提是系统存在一个变量 **SSLKEYLOGFILE**, 配置并重启浏览器

![system.var](/images/posts/system.var.jpg)

配置 WireShark, 编辑 > 首选项 > Protocols > TLS
**Pre-Master-Secret log filename** 对应系统变量 SSLKEYLOGFILE 配置的文件地址  
TLS debug file 是调试日志, 可以配置上, 具体如下图

![wireshark.tls.set](/images/posts/wireshark.tls.set.jpg)

配置成功后可以直接设置过滤 HTTP2 请求  

![wireshark.filter](/images/posts/wireshark.filter.jpg)



#### 参考文献

[Wiki HTTP2](https://zh.wikipedia.org/wiki/HTTP/2)
[WireShark TLS Decryption](https://wiki.wireshark.org/TLS#TLS_Decryption)