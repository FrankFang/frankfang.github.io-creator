---
title: "Rx.js 思想入门"
date: 2018-03-13T02:24:08+08:00
isCJKLanguage: true
---

原版视频：[Async JavaScript at Netflix](https://www.youtube.com/watch?v=XRYN2xt11Ek) by Jafar Husain  
原视频语速极快，我反复听了好几遍。本节课就是我对该演讲的翻译和理解。

我觉得这个视频对 Rx.js 新手实在是太友好了，基本一听就懂，所以花了一天时间汉化以及重新演绎。

* * *

本节课主要关注如何通过「换一种思路」来解决「异步」问题。

我们的所有网页应用都是异步的：

1.  脚本加载
2.  播放器
3.  数据访问
4.  动画
5.  DOM事件绑定、数据事件绑定

## 异步编程

```js
function play(movieId, cancelButton, callback){
    let movieTicket
    let playError
    let tryFinish = () =>{
        if(playError){
            callback(null, playError)
        }else if(movieTicket && player.initialized){
            callback(null, movieTicket)
        }
    }
    cancelButton.addEventListener('click', ()=>{ playError = 'cancel' })
    if(!player.initialized){
        player.init((error)=>{
            playError = error
            tryFinish()
        })
    }
    authorizeMovie(movieId, (error, ticket)=>{
        playError = error
        movieTicket = ticket
        tryFinish()
    })
}
```

我们可以看到，异步编程中的状态（state）是很难跟踪的

![](/images/what-is-rxjs/1.jpg)

当项目变复杂时，你很难理解某个状态是如何变化的。

另一方面，使用回调时，try...catch 语法基本是没用的

![](/images/what-is-rxjs/2.jpg)

另外，如果你监听了一个事件却忘了销毁它，很容易造成内存泄露。这在异步编程很常见。

![](/images/what-is-rxjs/3.jpg)

## 预备知识一：Iterator & Observer

为了解决这些问题，让我们回到 1994 年。1994 年有一本书叫做《设计模式》

这本书讲了很多编程套路（编程套路就是设计模式）

今天我们只关注其中的两个设计模式

1.  Iterator 迭代器
2.  Observer 观察者

## 迭代器

```js
function makeIterator(array){
    var nextIndex = 0;

    return {
       next: function(){
           return nextIndex < array.length ?
               {value: array[nextIndex++], done: false} :
               {done: true};
       }
    };
}

var it = makeIterator(['a', 'b']);
console.log(it.next().value); // 'a'
console.log(it.next().value); // 'b'
console.log(it.next().done);  // true

```

ES 6 提供了一个语法糖来达成迭代器模式，这个语法糖叫做生成器（Generator）

```js

function* idMaker() {
  var index = 0;
  while(true)
    yield index++;
}

var gen = idMaker();

console.log(gen.next().value); // 0
console.log(gen.next().value); // 1
console.log(gen.next().value); // 2

```

所谓迭代器模式就是你可以用 .next() API 来「依次」访问下一项。（next只是一个函数名而已，可以随意约定）

1.  如果有下一项，你就会得到 `{value: 下一项的值, done: false}`
2.  如果没有下一项，你就会得到 `{value: null, done: true}`

## 观察者模式

这个模式则是监听一个对象的变化，一旦对象发生变化，就调用你提供的函数。（JS 已废弃 Object.observe()，请使用 Proxy API 代替）

```js

var user = {
  id: 0,
  name: 'Brendan Eich',
  title: 'Mr.'
};

// 创建用户的greeting
function updateGreeting() {
  user.greeting = 'Hello, ' + user.title + ' ' + user.name + '!';
}
updateGreeting();

Object.observe(user, function(changes) {
  changes.forEach(function(change) {
    // 当name或title属性改变时, 更新greeting
    if (change.name === 'name' || change.name === 'title') {
      updateGreeting();
    }
  });
});

```

## 两种模式的区别

假设 A 是一个迭代器，那么 B 可以主动使用 A.next() 来要求 A 产生变化。（B主动要求A变化）  
假设 B 是一个观察者，在观察着 A，那么 A 一旦变化，A 就会主动通知 B。（A变化之后B被动接收通知）

或者这么说：在观察者模式里，被观察的人在迭代观察者（调用观察者的一个函数）。  
再说清楚一点：观察者就是一个迭代器，被观察的人一旦有变化，就会调用观察者的一个函数。

```js
user .on change
    observer.next()
```

只不过，观察者永远可以 .next()，不会结束。而迭代器是会结束的，即返回 `{done: true}`。

## 预备知识二：Array V.S. Event

```js
    Array: [ {x:1,y:1}, {x:2, y:2}, {x:10,y:10} ]
    Event: {x:1,y:1} ... {x:2, y:2} ... {x:10, y:10}
```

数组和事件，有啥区别？

他们都是 collection（数据集、集合）。

为了阐述它俩之间的相同点，我们来举两个例子。

首先我们介绍 Array 的 4 个操作：

![](/images/what-is-rxjs/4.jpg)

用这几个 API 我们可以做一些 amazing 的事情，在 Netflix 我们主要向用户展示一些好看的剧集。

我们需要展示评分最高的剧集给用户。能不能用上面的操作做到呢？

```js
let getTopRatedFilms = user => 
    user.videoLists
        .map( videoList => 
            videoList.videos
                .filter( video => video.rating === 5.0)
        ).concatAll()

getTopRatedFilms(currentUser)
    .forEach(film => console.log(film) )

```

好，如果我现在告诉你，一个拖曳操作能用类似的代码实现，你相信吗？  
你肯定在心里想：这不可能！

是时候展示真正的技术了：

```
let getElemenetDrags = el =>
    el.mouseDowns
        .map( mouseDown => 
            document.mouseMoves
                .takeUntil(document.mouseUps)
        )
        .concatAll()

getElementDrags(div)
    .forEach(position => img.position = position )

```

能做到这一切，都是因为 Observable（大意：可被观察的对象）

## Observable

> Observable = Collections + Time

## 来历

![](/images/what-is-rxjs/5.jpg)

## 用途

Observable 可以表示

*   事件
*   数据请求
*   动画

而且可以方便的把这三种东西组合起来，因此，异步操作变得很简单。

将事件转化为 Observable 的 API 很简单


    var mouseDowns = Observable.fromEvent(element, 'mouseDown')


之前我们是如何操作事件的？——监听（或者叫做订阅）


```js
// 订阅或监听
let handler = e => console.log(e)
document.addEventListener('mousemove', handler)

// 取消订阅或去掉监听
document.removeEventListener('mousemove', handler)
```

现在我们怎么对事件进行操作呢？——forEach

```
// 订阅
let subscription = mouseMoves.forEach(e => console.log(e) )
// 取消订阅
subscription.dispose()
```

将事件包装成 Observable 对象，可以方便地使用 forEach/map/filter/takeUntil/concatAll 等 API 来操作，比之前的方式容易很多。

为了处理失败情况，forEach 还可以接收两个额外的参数：

![](/images/what-is-rxjs/6.jpg)

看起来有点像 Promise 对吧。

为了跟清楚地阐述如何使用 forEach/map/filter/takeUntil/concatAll 等 API 来操作 Observable 对象，我现在发明一种新的语法：

![](/images/what-is-rxjs/7.jpg)

这个语法的规则是

1.  {1...2} 表示这个对象会一开始发射一个1，一段时间后发射一个2
2.  {1...2......3}表示发射1，一段时间后发射2，两段时间后发射3（也就是说 ... 表示一段时间，...... 表示两段时间）

## forEach

```
> {1......2............3}.forEach(console.log)
1
一段时间后 
一段时间后 
2
一段时间后 
一段时间后 
一段时间后 
一段时间后
3
```

## map


```js
> {1......2............3}.map(x=>x+1)
2
一段时间后 
一段时间后
3
一段时间后 
一段时间后
一段时间后 
一段时间后
4
```

## filter

```
> {1......2............3}.filter(x=>x>1)
一段时间后 
一段时间后
2
一段时间后 
一段时间后
一段时间后 
一段时间后
3
```

## concatAll

考虑 race conditions（竞态问题）


```js
{
    ...{1}
    ......{2..................3}
    ............{}
    ..................{4}
}.concatAll()

{
    ...1...2..................3...4
}
```

## takeUntil

```js
> {...1...2............3}.takeUntil(
  {............4}    
)
{...1...2...}
```


这个 API 给我们一个启发：其实我们根本不需要去取消订阅，只需要告诉系统订阅到什么情况为止就行了；就是在另一个事件出现时，系统自动取消上一个事件的订阅即可。

实际上很多人都不会去取消订阅，以至于出现内存泄露。比如我们经常监听 window 的 onload 事件，却基本没人去取消监听 window 的 onload 事件。

我已经有五年没有做过「取消监听」这件事了。但是我的代码却没有内存泄露，因为：

![](/images/what-is-rxjs/8.jpg)

再回头看我们的 drag 代码：

```js
let getElemenetDrags = el =>
    el.mouseDowns
        .map( mouseDown => 
            document.mouseMoves
                .takeUntil(document.mouseUps)
        )
        .concatAll()

getElementDrags(div)
    .forEach(position => img.position = position )

```

## 第二个例子

![](/images/what-is-rxjs/9.jpg)

这个 demo 的难点有两个：

1.  如果用户依次输入 abcdef，请问你应该发送几个请求？答案是用函数防抖，发一次请求。
2.  如果用户输入 a，然后 300 毫秒后输入 b，那么你会发两个请求，一个请求查询 a 相关的热词，一个请求查询 ab 相关的热词，你能保证这两个请求响应的顺序吗？答案是不能。（竞态问题）

使用 Observable 来思考这个问题

```js
let search = 
    keyPresses
        .debounce(250) // 原文是 throttle，但我个人认为原文写错了，我已经在 Twitter 上询问了演讲者，尚未得到回复
        .map(key =>
            getJSON('/search?q=' + input.value)
                .retry(3)
                .takeUntil(keyPresses)
        )
        .concatAll()
search.forEach(
    results => updateUI(results),
    error => showMessage(error)
)

```

## 最开始的回调地狱

最后我们本文回到最开始的代码

```js
function play(movieId, cancelButton, callback){
    let movieTicket
    let playError
    let tryFinish = () =>{
        if(playError){
            callback(null, playError)
        }else if(movieTicket && player.initialized){
            callback(null, movieTicket)
        }
    }
    cancelButton.addEventListener('click', ()=>{ playError = 'cancel' })
    if(!player.initialized){
        player.init((error)=>{
            playError = error
            tryFinish()
        })
    }
    authorizeMovie(movieId, (error, ticket)=>{
        playError = error
        movieTicket = ticket
        tryFinish()
    })
}
```

通过改变思维方式，你可以写出这样的代码


```
let authorizations = 
    player.init()
        .map(()=>
            playAttempts
                .map(movieId=>
                    player.authorize(movieId)
                        .retry(3)
                        .takeUntil(cancels)
                )
                .concatAll()
        )
        .concatAll()

authorizations.forEach(
    license => player.play(license),
    error => showError(error)
)
```

## 总结

以上就是 Rx.js 的思想，如果你需要更实际的练习，你可以点击以下链接：

*   英文教程：http://reactivex.io/learnrx/ （演讲者自己写的教程）
*   中文教程：[https://www.google.com/search?q=site%3Azhihu.com+太狼+rxjs](https://www.google.com/search?q=site%3Azhihu.com+太狼+rxjs) （我推荐太狼在知乎上写的 Rx.js 教程）

