---
title: "jQuery 都过时了，那我还学它干嘛？"
date: 2018-10-15T16:22:04+08:00
---

今天饥人谷的新生问我

> 请问现在我还需要学习 jQuery 吗？听你在知乎说 jQuery 已经过时了，是不是就不用学了？



## 短答案

jQuery 还是可以学一学的，学了之后对写代码和封装库很有帮助。

现在的「新人」依然可以学习 jQuery 的思想，因为以「新人」的水平，直接理解 Vue / React 的思想难度较大，jQuery 是一个很不错的中间过渡，因为 jQuery 也蕴含了非常多的编程套路。

但是如果你不想学，就不学吧。直接去学 Vue / React 会难一点，但也能学会。




## 长答案：

jQuery 当然过时了。

距离我上次在项目中使用 jQuery，可能已经快两年时间了（除去上课时演示功能时用 jQuery）。回想我学习 jQuery 的过程，还挺神奇的。

当年我在大学的技术小组里做 C# 网站开发，需要用到 jQuery 特效，组里的一名小伙伴会用一点 jQuery，很快就用 .animate 做出了让我啧啧称奇的特效。我觉得 jQuery 好神奇啊，虽然我当时连 JS 都不会。

于是我立马买了一本《锋利的 jQuery》，硬看。


什么叫「硬看」呢？因为我不会 JS，而且我并没有照着书上敲代码，仅仅使用眼睛「看 jQuery 代码」。神奇的是——我居然很快看懂了几乎整本书。以至于那位会用 jQuery 的小伙伴遇到 bug 问我时我能直接给出解答，看起来他并没有看《锋利的 jQuery》这本书（笑）。



到了 2018 年，几乎已经没有新项目会使用 jQuery 来开发了；即使有，也是一件不值得拿出来炫耀的事情。那为什么我还是建议学习 jQuery 呢？



原因如下。



## jQuery 教你如何设计 API

上文说到我一个不会 JS 的人居然能看懂 jQuery 的书，其实这不是因为我厉害，而是因为 jQuery 的 API 设计得太人性化了！

举几个例子给大家看看：

第一个是 jQuery 对事件监听的简化

    // 那时，如果不用 jQuery，监听事件（兼容 IE 6）你要这么写
    if (button.addEventListener)  
      button.addEventListener('click',fn);
    else if (button.attachEvent) { 
      button.attachEvent('onclick', fn);
    }else {
      button.onclick = fn;
    }

    // 但是如果你用 jQuery，你只需要这么写
    $(button).on('click', fn)

第二个是 jQuery 对元素选择的简化

    // 如果你想获取 .nav > .navItem 对应的所有元素，用 jQuery 是这样写的
    $('.nav > .navItem')

    // 在 IE 6 上，你得这么写
    var navItems = document.getElementsByClassName('navItem')
    var result = []
    for(var i = 0; i < navItems.length; i++){
      if(navItems[i].parentNode.className.match(/\bnav\b/){
        result.push(navItems[i])
      }
    }

有没有发现 jQuery 的代码一读就读懂了？可读性非常强！

当时我作为一个新人，每每看到 jQuery 那优雅的 API，都禁不住去思考 jQuery 到底是怎么实现的，我自己能不能实现出来（但我并不推荐看 jQuery 源码）。本着这样的想法，我学会了很多编程技巧。

为什么有些人代码水平老是提不高了，就是因为不会造轮子，不会设计优雅的 API，更不会实现优雅的 API，只会调用其他库或框架提供的功能（中枪的举手）。

而 jQuery 则提供了一个简单而又经典的范例供大家学习。



不信的话我们就来看看 jQuery 用到了哪些所谓的设计模式（其实就是编程套路）吧。

0. 发布订阅模式

        var eventHub = $({})
        eventHub.on('xxx', function(){ console.log('收到') })
        eventHub.trigger('xxx')

0. 用原型继承实现插件系统

        $.fn.modal = function(){ ... }
        $('#div1').modal()

    Vue 2 的插件也是类似的思路哦

0. 事件委托

        $('div').on('click', 'span', function(){...})

    说实话，你在 2018 年找前端让他写一个事件委托，我保证 90% 写出来的代码都是有「明显」bug 的。

0. 链式调用

        $('div').text('hi').addClass('red').animate({left: 100})

0. 函数重载（伪）

        $(fn)
        $('div')
        $(div)
        $($(div))
        $('span', '#scope1')

    你会发现 $ 这个函数的参数可以是函数、字符串、元素和 jQuery 对象，甚至还能接受多个参数，这种重载是怎么做到的？

0. 命名空间

        // 你的插件在一个 button 上绑定了很多事件
        $button.on('click.plugin', function(){...})
        $button.on('mouseenter.plugin', function(){...})
        // 然后你想在某个时刻移除以上所有事件
        $button.off('.plugin')

    如果你不用 jQuery 就很麻烦了。

0. 高阶函数

        var fn2 = $.proxy(fn1, asThis, param1)

    $.proxy 接受一个函数，返回一个新的函数。

其他就不一一列举了。


## jQuery 的 API 风格依然在流行

我们把 jQuery 和 Axios 做一下对比：

    $.ajax({url:'/api', method:'get'})
    $.get('/api').then(fn1,fn2)

    axios({ url: '/api', method: 'get'})
    axios.get('/api').then(fn1, fn2)

为什么 2018 年流行的 axios 跟 jQuery.ajax 这么相像呢？

因为 jQuery 的 API 实在太好用了！搞得新库根本没法超越它，没有办法设计出更简洁的 API 了。毕竟 jQuery 也是在前端界流行近十年。

所以你学了 jQuery 很容易过渡其他类似的新库。



## jQuery 也能做 MVC

很多人以为前端框架是从 Vue、React 和 Angular 才开始的，其实 jQuery 时代早就有基于 jQuery 的 MV* 库了，比如著名的 Backbone.js 和 Marionette.js。

看看下面的 Backbone 应用代码

    var TodoView = Backbone.View.extend({
        tagName:  'div',
        template: _.template($('#item-template').html()),
        events: {
            'click .toggle': 'xxx',
        },
        initialize: function () {
            this.listenTo(this.model, 'change', this.render);
        },
        render: function () {
            if (this.model.changed.id !== undefined) {return; }
            this.$el.html(this.template(this.model.toJSON()));
            return this;
        }
    });

AngularJS、Vue 1.x、Vue 2.x 其实都是顺着 Backbone MVC 的思路慢慢优化、改造得来的，如果你提前了解 Backbone 作为知识铺垫，那么理解 Vue 是非常容易的。如果面试官问你 MVC 和 MVVM 的区别，你也是很容易就可以答出来的。




最后就引用我之前的一个回答作为结尾来说明学习 jQuery 意义：

[完全理解jQuery源代码，在前端行业算什么水平？](https://www.zhihu.com/question/20521802/answer/290354219)

> 说明你

> 1. 精通正则表达式
> 2. 了解闭包
> 3. 了解原型链
> 4. 精通 DOM API
> 5. 了解各种设计模式（事件、Promise、伪重载、装饰器模式等）
> 6. 精通 DOM 事件
> 7. 了解旧浏览器的各种特性（bug）
> 8. 了解模块化
> 9. 了解浏览器渲染原理
> 10. 精通 AJAX
> 11. 了解 HTTP 请求

> 可以秒杀中国 80% 的前端
