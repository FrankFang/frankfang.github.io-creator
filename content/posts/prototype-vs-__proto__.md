---
title: "prototype 和 __proto__ 的区别"
date: 2018-02-09T02:21:39+08:00
isCJKLanguage: true
---


![](/images/prototype-vs-__proto__/1.jpg)

1.  你的 JS 代码还没运行的时候，JS 环境里已经有一个 `window` 对象了
2.  `window` 对象有一个 Object 属性，`window.Object` 是一个函数对象
3.  `window.Object` 这个函数对象有一个重要属性是 prototype，干什么用的等会说
4.  `window.Object.prototype` 里面有这么几个属性 toString（函数）、valueOf（函数）

好，目前先知道这些就够了。

然后我们写一句代码

    var obj = {}
    obj.toString()

这句代码做了啥？为什么 obj 有 toString() 属性？

![](/images/prototype-vs-__proto__/2.jpg)

1. 这句话大概是让 obj 变量指向一个空对象，这个空对象有个 `__proto__` 属性指向 `window.Object.prototype`。
2. 这样你在调用 `obj.toString()` 的时候，obj 本身没有 toString，就去 `obj.__proto__` 上面去找 toString。
3. 所以你调用 `obj.toString` 的时候，实际上调用的是 `window.Object.prototype.toString`
4. 那么 `window.Object.prototype.toString` 是怎么获取 obj 的内容的呢？
    那是因为 `obj.toString()` 等价于 `obj.toString.call(obj)`
5. 同时 `obj.toString.call(obj)` 等价于 `window.Object.prototype.toString.call(obj)`
    这句话把 obj 传给 toString 了。

## 再看复杂一点的

回到第一幅图

![](/images/prototype-vs-__proto__/1.jpg)

我们写一句代码

    var arr = []
    arr.push(1) // [1]

请问这两句话做了什么？

![](/images/prototype-vs-__proto__/3.jpg)

1. 看红色部分，`var arr = []` 大概会让 arr 指向一个空对象，然后 `arr.__proto__` 指向 `window.Array.prototype`。（其实 arr 有一个 length:0，不过这里就忽略吧）
2. 这样你在调用 `arr.push` 的时候，arr 自身没有 push 属性，就去 `arr.__proto__` 上找 push
    因此 `arr.push` 实际上是 `window.Array.prototype.push`
3. `arr.push(1)` 等价与 `arr.push.call(arr,1)`
4. `arr.push.call(arr,1)` 等价于 `window.Array.prototype.push.call(arr, 1)`

## 再再复杂一点

`arr.valueOf()` 做了什么?

1. arr 自身没有 valueOf，于是去 `arr.__proto__` 上找
2. `arr.__proto__` 只有 pop、push 也没有 valueOf，于是去 `arr.__proto__.__proto__` 上找
3. `arr.__proto__.__proto__` 就是 `window.Object.prototype`
    所以 `arr.valueOf` 其实就是 `window.Object.prototype.valueOf`

4. `arr.valueOf()` 等价于 `arr.valueOf.call(arr)`
    `arr.valueOf.call(arr)` 等价于 `window.Object.prototype.valueOf.call(arr)`

看，JavaScript 其实很优美很简单。

只是你想复杂了而已：

- prototype 指向一块内存，这个内存里面有共用属性，而 `__proto__` 指向**同一块内存**。
- prototype 和 `__proto__` 的不同点在于：prototype 是构造函数的属性，而 `__proto__` 是对象的属性
- 如果没有 prototype，那么共用属性就没有立足之地
- 如果没有 `__proto__`，那么一个对象就不知道自己的共用属性有哪些。

这里隐含了一个难点……构造函数也是对象！先不纠结这个。

## 反证法

假设我们把 `__proto__` 去掉，那么

    var obj = {}
    obj.toString() // 报错，没有 toString 方法

所以你只能这样声明一个对象咯：

    var obj = {
      toString: window.Object.prototype.toString,
      valueOf: window.Object.ptototype.valueOf
    }
    obj.toString() // '[object Object]'

现在知道 `__proto__` 帮你省多少代码了吧？

假设我们删掉 prototype，包括 `window.Object.prototype` 和 `window.Array.prototype`。

那么 `window.Object.prototype.toString` 也一并被删除了。

然后我们基本就没法写代码了……

    var obj = {}
    obj.toString() // toString 不存在，因为 toString 没有定义过啊

prototype 的意义就是把共有属性预先定义好，给之后的对象用。

自己想想吧~

新人搞不懂原型大抵是因为

1.  不懂内存、引用
2.  不懂链表、树等数据结构
3.  不知道函数是一种对象
4.  被 Java 的 class 关键字毒害了
5.  还有一种可能是因为没遇到我

这幅图我还可以继续讲，把 JS 所有基础知识都能串起来。

比如很多人不懂什么是伪数组，很简单：

1.  如果一个数组的 `__proto__` 直接或间接指向 `Array.prototye`（用到了数组的共用属性），那么就是真数组
2.  如果一个数组的 `__proto__` 没有直接或间接指向 `Array.prototye`，那么就是伪数组


```
var realArr = {0: 'a', 1:'b', length: 2}
realArr.__proto__ = Array.prototype
// 这就是真数组（并不完全是）
// 基本等价于 realArr = ['a', 'b']
realArr.push !== undefined // true

var fakeArr = {0: 'a', 1:'b', length: 2}
// 这就是伪数组
realArr.push === undefined // true
```


完。
