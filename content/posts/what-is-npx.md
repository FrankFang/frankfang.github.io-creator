---
title: "npx 是干什么的"
date: 2017-07-12T02:20:53+08:00
isCJKLanguage: true
---

最近我在更新 npm 5.2.0 的时候发现会买一送一，自动安装了 npx。

一个月后 npx 就是前端必会的知识，相信我 ; )

（不要问我是不是抄袭，看看文章发表时间你就知道我是原创了，本文发表于 2017 年 7 月 12 日）

## npx 是什么

根据 [zkat/npx](https://github.com/zkat/npx) 的描述，npx 会帮你执行依赖包里的二进制文件。

举例来说，之前我们可能会写这样的命令：

    npm i -D webpack
    ./node_modules/.bin/webpack -v

如果你对 bash 比较熟，可能会写成这样

    npm i -D webpack
    `npm bin`/webpack -v

有了 npx，你只需要这样

    npx webpack -v

也就是说 npx 会自动查找当前依赖包中的可执行文件，如果找不到，就会去 PATH 里找。如果依然找不到，就会帮你安装！

npx 甚至支持运行远程仓库的可执行文件，如

    $ npx github:piuccio/cowsay hello
    npx: 1 安装成功，用时 1.663 秒
     _______
    < hello >
     -------
            \   ^__^
             \  (oo)\_______
                (__)\       )\/\
                    ||----w |
                    ||     ||

再比如 npx http-server 可以一句话帮你开启一个静态服务器！（第一次运行会稍微慢一些）

    $ npx http-server
    npx: 23 安装成功，用时 48.633 秒
    Starting up http-server, serving ./
    Available on:
      http://127.0.0.1:8080
      http://192.168.5.14:8080
    Hit CTRL-C to stop the server


你也试试吧~

