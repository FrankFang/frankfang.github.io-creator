---
title: "npm 是干什么的"
date: 2016-12-14T02:16:39+08:00
isCJKLanguage: true
---

网上的 npm 教程主要都在讲怎么安装、配置和使用 npm，却不告诉新人「为什么要使用 npm」。今天我就来讲讲这个话题。

本文目标读者是「不太了解 npm 的新人」，大神您别看了，不然又说我啰嗦了 😂。

## 社区

程序员自古以来就有社区文化：

> 社区的意思是：拥有共同职业或兴趣的人们，自发组织在一起，通过分享信息和资源进行合作。虚拟社区的参与者经常会在线讨论相关话题，或访问某些网站。

前端程序员也有社区，世界上最大的前端社区应该就是 GitHub 了。前端通过 GitHub 来

1.  分享源代码（线上代码仓库）  
2.  讨论问题（Issue 列表）  
3.  收集学习资源和常去的网站（比如我收集的[优质中文前端博客](https://github.com/FrankFang/best-chinese-front-end-blogs)）

加入社区最大的好处之一是，你可以使用别人贡献的代码，你也可以贡献代码给别人用。

## 共享代码

前端是怎么共享代码的呢？

**在 GitHub 还没有兴起的年代，前端是通过网址来共享代码的**

比如你想使用 jQuery，那么你点击 jQuery 网站上提供的链接就可以下载 jQuery，放到自己的网站上使用

![](/images/what-is-npm/1.jpg)

**GItHub 兴起之后，社区中也有人使用 GitHub 的下载功能：**

![](/images/what-is-npm/2.jpg)

## 麻烦

当一个网站依赖的代码越来越多，程序员发现这是一件很麻烦的事情：

1.  去 jQuery 官网下载 jQuery  
2.  去 BootStrap 官网下载 BootStrap  
3.  去 Underscore 官网下载 Underscore  
4.  ……  

有些程序员就受不鸟了，一个[拥有三大美德](http://zhihu.com/question/48406009/answer/134182064)的程序员 [Isaac Z. Schlueter](https://github.com/isaacs) （以下简称 Isaaz）给出一个解决方案：用一个工具把这些代码集中到一起来管理吧！

这个工具就是他用 JavaScript （运行在 Node.js 上）写的 npm，全称是 Node Package Manager

## 具体步骤

NPM 的思路大概是这样的：

1. 买个服务器作为代码仓库（registry），在里面放所有需要被共享的代码
2. 发邮件通知 jQuery、Bootstrap、Underscore 作者使用 npm publish 把代码提交到 registry 上，分别取名 jquery、bootstrap 和 underscore（注意大小写）
3. 社区里的其他人如果想使用这些代码，就把 jquery、bootstrap 和 underscore 写到 package.json 里，然后运行 npm install ，npm 就会帮他们下载代码
4. 下载完的代码出现在 node_modules 目录里，可以随意使用了。

这些可以被使用的代码被叫做「包」（package），这就是 NPM 名字的由来：Node Package(包) Manager(管理器)。

## 发展

Isaaz 通知 jQuery 作者 John Resig，他会答应吗？这事儿不一定啊，对不对。

只有社区里的人都觉得 「npm 是个宝」的时候，John Resig 才会考虑使用 npm。

那么 npm 是怎么火的呢？

npm 的发展是跟 Node.js 的发展相辅相成的。

Node.js 是由一个在德国工作的美国程序员 Ryan Dahl 写的。他写了 Node.js，但是 Node.js 缺少一个包管理器，于是他和 npm 的作者一拍即合、抱团取暖，最终 Node.js 内置了 npm。

后来的事情大家都知道，Node.js 火了。

随着 Node.js 的火爆，大家开始用 npm 来共享 JS 代码了，于是 jQuery 作者也将 jQuery 发布到 npm 了。

所以现在，你可以使用 npm install jquery 来下载 jQuery 代码。

现在用 npm 来分享代码已经成了前端的标配。

## 后续

Node.js 目前由 Ryan Dahl 当时所在的公司 joyent 继续开发。Ryan Dahl 现在已经去研究 AI 和机器学习了，并且[他把 Node.js 的维护权交给了 Isaaz](https://groups.google.com/forum/#!topic/nodejs/hfajgpvGTLY)。（我是不是也应该去研究 AI 和机器学习啊教练）

而 Isaaz 维护了一段时间后，辞职了，成立了一个公司专门维护 npm 的 registry，公司名叫做 [npm 股份有限公司](https://www.npmjs.com/about#about-npm-inc)……谁说开源不能赚钱的~  

## 社区的力量

回顾前端的发展是你会发现，都是社区里的某个人，发布了一份代码，最终影响前端几年的走向。比如 jQuery，比如 Node.js，比如 npm。（其实其他语言也是这样的）

所以，社区的力量是巨大的。

