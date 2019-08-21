---
title: "我不是很懂 Node.js 社区的 DRY 文化"
date: 2018-04-19T02:18:28+08:00
isCJKLanguage: true
---

我一直以为 npm 里下载量较大的 package 是 React 这样不错的包。

今天我才知道我错了。

目前 React 每周下载量是 240 万次。

然而下面我要说的几个包的下载量全都大于 React！

[is-odd](https://www.npmjs.com/package/is-odd)，每周下载 300 万次

源代码如下：

    'use strict';

    var isNumber = require('is-number');

    module.exports = function isOdd(i) {
      if (!isNumber(i)) {
        throw new TypeError('is-odd expects a number.');
      }
      if (Number(i) !== Math.floor(i)) {
        throw new RangeError('is-odd expects an integer.');
      }
      return !!(~~i & 1);
    };

你没有看错，五行核心代码，还依赖了一个 is-number 库。

这个 is-number 库更厉害，每周下载 1000 万次

源代码如下：

    'use strict';

    module.exports = function isNumber(num) {
      var number = +num;

      if ((number - number) !== 0) {
        // Discard Infinity and NaN
        return false;
      }

      if (number === num) {
        return true;
      }

      if (typeof num === 'string') {
        // String parsed, both a non-empty whitespace string and an empty string
        // will have been coerced to 0\. If 0 trim the string and see if its empty.
        if (number === 0 && num.trim() === '') {
          return false;
        }
        return true;
      }
      return false;
    };


后来我发现这两个库的作者是同一个人（该作者水平很高），这个人还写了另外几个库：

*   is-plain-object，每周下载量 330 万
*   is-primitive，每周下载量 350 万，[源代码你自己可以看看](https://github.com/jonschlinkert/is-primitive/blob/master/index.js)
*   isobject，每周下载量 750 万

需要指出的是

1.  webpack、babel 等库都有「间接地」依赖上面的一些包。
2.  这些包的 markdown 代码远远多于 JS 代码，可能它们的 markdown 更值得我们学习

这件事对我的启发：

1.  原来有这么多 JS 程序员不会判断奇数
2.  只要 markdown 写得漂亮，就能迷倒 JS 程序员
3.  1 + '1' 的问题一直在困扰 JS 程序员，我要不要写一个 add() 库解决这个问题呢

> **我终于知道为什么 npm install 总是动不动就下载 300 Mb 的东西了，Node.js 社区强调的 DRY 文化使得 node_modules 臃肿不堪，因为有的库引用了 is-object，有的库引用了 isobject，还有的库引用了 isObject，每个包看起来很 DRY，但是合起来就 wet 得不行了，呵呵。**

Node 社区跟我想得不太一样，说不上好也说不上坏，反正不是很适合我。

* * *

以下是扯淡。

我是看到 Medium 上的一篇《[混乱又危险的 Node.js 生态](https://medium.com/commitlog/the-internet-is-at-the-mercy-of-a-handful-of-people-73fac4bc5068)》才知道这些的，这篇文章里的一个评论我很赞同：

![](/images/i-dont-get-node/1.jpg)

如果你不能在十秒钟内写出一个判断奇数的函数，要么你是一个糟糕的打字员，要么你就不应该当程序员！

还有一些颇为搞笑的评论：

![](/images/i-dont-get-node/2.jpg)
![](/images/i-dont-get-node/3.jpg)
![](/images/i-dont-get-node/4.jpg)
![](/images/i-dont-get-node/5.jpg)
