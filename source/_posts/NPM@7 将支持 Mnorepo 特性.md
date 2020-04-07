---
title: NPM@7 将支持 Mnorepo 特性
date: 2019-08-04 20:48:28
tags: [NPM, menorepo]
categories: news
---


从架构的角度来看, 将大型单体代码库拆分为较小的、独立封装的一系列模块通常是个好方法。从微服务到可复用组件库, 很多技术都很适合模块化。但从版本发布和源代码管理的角度来看, 它也可能是一场噩梦。

<!-- more -->

#### NPM 存在的问题

当项目有数十乃至数百个代码存储库时, 发现工作也变得愈加困难。每当创建一个新的存储库, 所有人必须将其添加到自己的代码模块中。

访问控制成了单调乏味的工作, 且容易出错, 尤其是得在"需要了解"的基础上一次一个地授予存储库访问权限, 实在令人头疼。新员工经常要经历无止境的依赖项地狱。每个访问请求都会引出它们需要访问的另外两个存储库来。

高度模块化的架构也使版本控制变复杂了。我们可以让一系列相关模块的版本同步更新, 给它们做"快照"（例如 Babel 和 React 就是这么做的）。但要让人们为那么多的包做这种工作（还包括和他们没什么关系的包）就是在自找麻烦。

包之间的依赖项重复会大大增加安装依赖项所需的时间。一些生态系统（特别是 npm）是高度模块化的。模块化会鼓励复用, 但也意味着某些包可能在你开发的每个包里都复制了一份。



#### 解决方案

Monorepo 是针对这个问题的流行解决方案。它意味着你会将所有模块放在同一个代码存储库中, 而不是每个模块配一个代码存储库。然后开发者在开发应用时只用这个"monorepo"就够了。由于所需的内容都放在了一起, 发现、访问控制和版本控制工作都变得更加简单  

显然, 代价就是你只能对所有代码做整体权限控制。但如果你能接受这一点, monorepo 就能在兼顾模块化所有优势的前提下提供非常简单的源代码管理方法  

那么 monorepo 怎样搭配 npm 使用呢？最受欢迎的两大解决方案分别是 Yarn workspaces 和 lerna  


#### 使用 lerna 实现 monorepo  

首先我们全局安装 lerna  

```
npm install-g lerna
```

接下来, 我们需要创建新的 lerna 存储库：

```
mkdir monorepo_example  
cdmonorepo_example  
lerna init  
```

查看 lerna.json 可以看到版本和包的定义位置  

```
cat lerna.json

{
  "packages": [
    "packages/*"
  ],
  "version": "0.0.0"
}
```

如上所示, 我们的 monorepo 中所有包的版本都是 0.0.0。但是我们的 monorepo 中还没有任何包。在我们添加包之前应该先登录到我们的存储库, 以便 lerna 在每个新包上正确设置 publishConfig  

如果你要发布在公共 npm 存储库之外的位置上（例如 Verdaccio）, 则首先需要设置你的存储库    

```
$npm config set registry https://registry.npm.yourcompany.com  
```

接下来, 我们用具有发布权限的用户登录到存储库中。如果你的包没有作用域, 则可以省略 -scope 标志  

```
$npm login -scope test
```

现在为我们的 monorepo 添加一些定域包（scoped package）

```
lerna create @test/ a
lerna create @test/ b
lerna create @test/ c
```

如果你为每个包选择了默认设置, 那么现在测试域内会有三个包（a、b、c）, 版本为 0.0.0。在更新版本之前, 我们需要提交已完成的工作的并为 git 创建一个远程连接：  

```
git add
git commit -m "Initial commit"
git remote add origin git@github.com:username/reponame.git
git push -u origin master
```

现在我们可以使用一个命令来更新所有包的版本  

```
lerna version major
```

此命令不仅会将每个包都更新到 1.0.0, 还会为你推送版本更新到 git。最后一步是将这些包发布到 npm。Lerna 用一个命令就能完成这个任务  

```
$ lerna publish from-git
```

这条命令成功运行后, 你就成功将第一个 monorepo 中的所有包都发布到 npm 了  


#### 总结一下

用 Lerna 创建一个新的 monorepo   

为我们的 monorepo 添加了三个新的定域包  

更新了所有包的版本并使用一条命令提交给了 git  

使用一条命令将所有包发布到了 npm 存储库   

有关 Lerna 的更多信息, 包括解决重复问题的方案, 建议查阅他们提供优秀的 文档   


#### 未来计划

我们也很高兴地宣布, 我们计划为 npm@7 带来一流的 monorepo 支持。如果你在搭配 monorepo 使用 npm, 我们需要你的反馈！请告诉我们你对目前的 monorepo 解决方案的看法。