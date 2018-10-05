---
title: Design story of LitPHP
thumb: lit.png
date: 2014-03-02
aliases:
  - /2014/03/design-story-of-litphp/
---

LitPHP([官网](http://litphp.github.io/) | [Repo](http://github.com/litphp/lit)) 昨天正式开源了，这是喜欢框架的我的第一个自认为能拿出手的作品。终于有了这么个作品让我相当地感想万千。先贴上12个类+1个接口的类图

![](https://rawgithub.com/LitPHP/litphp.github.io/115d5ed1f2d1884b2ac1a147c1583fc50c2e4f84/doc/graphs/classes.svg)

### LitPHP的设计哲学

<!--more-->

+ 让程序员选择 @[Backbone](http://backbonejs.org/)

    + 在Backbone官网中，找到[这样一段话](http://backbonejs.org/#FAQ-why-backbone)
        
        > Backbone.js aims to provide the common foundation that data-rich web applications with ambitious interfaces require — __while very deliberately avoiding painting you into a corner by making any decisions that you're better equipped to make yourself__.
        
    + 我太喜欢『paint into corner』这样的说法了，这是我用很多其他框架的直观感受，可以翻成『画地为牢』吧，我把这段话翻译一下套到LitPHP上来
    
        LitPHP希望提供一个性质良好的PHP程序结构。为了避免画地为牢，LitPHP刻意减少功能，不作任何我们认为应当由实际开发者来做出的决策（比如视图层究竟包含哪些职责/如何嵌套，又比如ORM或模板引擎的选择）
        
    + 为什么选择LitPHP？因为它让你选择！

+ 『中间件』及其支点 @[connect](http://www.senchalabs.org/connect/)
    + connect作为一个非常成功的框架，其中间件的理念让nodejs在webserver方面的成就拔高了一截。
    + 中间件(middleware)模式和插件(plugin)模式的区别，在我看来可以理解成插件是『改变原有功能』而中间件是『拼接出需要的功能』。插件主要『覆盖』而中间件主要『连接』。
    + 中间件为了达成连接的目的，需要不动的支点，nodejs天然提供了`request`对象和`response`对象，connect的中间件便以这两个对象为支点，通过这两个对象来协作。
    + LitPHP中，Route相关代码扮演connect的角色，让中间件得以执行，而其他代码就聚焦于扮演支点的角色：`Lit_Result`大约是`response`的角色，而看似突兀又简陋的`Lit_Http_Conversation`，其实扮演了非常重要的`request`的角色。`Lit_App`作为全局工厂，让动态程度不及JS的PHP能够更好的改变/添加各个组件的功能，还提供了context作为全局黑板便于信息交换。

        虽然尚未经过实践，但我可以想象负责缓存的中间件在判断缓存命中后，设置一个`Cache_Hit_Result`后中断后续路由逻辑；负责用户登录的中间件在发现用户未登录时设置跳转Result并中断，否则将用户信息写入app context……
    
+ 无招胜有招
    + LitPHP基本没有`::`符号写死类名，也不存在`final`的方法，而每个方法都不超过10行代码，你可以而且很容易通过继承来控制所有行为。
    + LitPHP没有刻板的MVC划分。只要有接受`Lit_App`的callback，就可以作为C；只要继承`Lit_Result`，或与其组合，就可以作为V；只要不是V，就可以视为M。实际应用中如何分层如何配合，完全由使用者自己决定。

最后贴上QQ群__341338880__，欢迎加群讨论

<a target="_blank" href="http://shang.qq.com/wpa/qunwpa?idkey=e0ce42feb7d8f8b7de8b2ab6bdac26d62e1cd34189ea767127f1e41ea70df0d8"><img border="0" src="http://pub.idqqimg.com/wpa/images/group.png" alt="LitPHP讨论" title="LitPHP讨论"></a>

