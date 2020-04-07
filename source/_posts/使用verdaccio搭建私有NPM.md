---
title: 使用 verdaccio 搭建私有 NPM
date: 2019-09-28 01:06:18
category: posts
tags: [verdaccio, npm]
---

本篇分享如何搭建一个私有 NPM 服务来管理内部模块 

<!-- more -->

#### 为什么要搭建私有 NPM
NPM (node 包管理器), 用来解决模块管理问题。这使得用们可以很方便的去使用和管理一个模块。NPM 上的包是公开的, 所有人都可以下载。 内部的模块并不能发布到 NPM 上, 那么就需要一个私有的 NPM 来管理内部模块  

这里使用 verdaccio 来搭建。 这是一个从 sinopia frok 过来的一个项目, 目前第4个版本

#### 安装

```
npm i -g verdaccio
```

安装完运行 verdaccio 

![运行](/images/posts/verdaccio.run.jpg)

第一次执行会生成一个 config 文件  

#### 配置 config

```
storage: ./storage       // 这里会缓存安装过的文件, 下次安装不用到 upstream 
plugins: ./plugins       // 插件
web:
  title: Verdaccio       // 标题

```

配置用户和可添加最大用户数, 通过 npm adduser 会在目录下生成 htpasswd 文件  

```
auth:
  htpasswd:
    file: ./htpasswd    // 这里记录这用户名和密码 .e.g. clc:$6B6QAAfdF:autocreated 2019-03-28T18:18:47.022Z
    max_users: 1000     // 用户上限  设置为 -1 禁止添加
```

配置上游, 可以配置多个。可以为不同的组分配上游, 这将决定包会从哪里下载, 上传到哪里  

```
uplinks:
  taobao:
    url: https://registry.npm.taobao.org/     // url
    max_fails: 5                              // 失败重试次数
    fail_timeout: 2m                          // 超时时间
  npmjs:
    url: https://registry.npmjs.org/
    max_fails: 5
    fail_timeout: 2m
```

配置包地址前缀, 这里建议配上

``` 
url_prefix: /verdaccio/
```

包分组, 这是一个强大的分组功能。可以配置多个作用域, 并配置权限、上游  
例如可以为公司内部包分配一个作用域 @company, 配置可发包用户, 可下载用户  
外部域可以配置 proxy 上传到 npm 或则 cnpm

```
packages:
  '**':
    access: $all  // username-用户名、$authenticated-登陆验证、$all
    publish: $authenticated
    unpublish: $authenticated
    proxy: npmjs  // 上游

  '@company/*':
    access: $authenticated
    publish: admin
    unpublish: $authenticated

```
[Packages 详细配置](https://verdaccio.org/docs/zh-CN/packages)

配置监听地址

```
listen:
  https://localhost:9384
```

开启https [https 详细配置](https://verdaccio.org/docs/zh-CN/ssl)

```
https:
  key:  /root/.config/verdaccio/verdaccio-key.pem
  cert: /root/.config/verdaccio/verdaccio-cert.pem
  ca:   /root/.config/verdaccio/verdaccio-csr.pem
```

#### 如何使用

先配置 registry, 也可以使用 nrm 来管理 registry

```
npm config set registry https://npm.yourhost.com
```

添加用户, 登陆

```
npm adduser
npm login
```

发布删除包

```
npm publish
npm unpublish
```

发布到内部域, 配置包 pageage.json

```
{
  "name": "@company/vue-host-editer"
}
```

这是一份 verdaccio 的配置样例 [verdaccio configure example](https://github.com/besfro/code-war/blob/example/verdaccio%20configure.ymal)


PS: NPM 解决了模块管理问题, 但对管理更新并没有解决的方案, 如果项目数量和依赖模块达到一定数量, 那将是地狱。 verdaccio 也支持 monorepo 管理模式 [Configure to Verdaccio Monorepo](https://github.com/verdaccio/monorepo/blob/master/CONTRIBUTING.md)


#### 参考文献
[Verdaccio home page](https://verdaccio.org/)  
[Verdaccio Configure](https://verdaccio.org/docs/zh-CN/configuration)  
[Verdaccio Monorepo](https://github.com/verdaccio/monorepo/blob/master/CONTRIBUTING.md)  