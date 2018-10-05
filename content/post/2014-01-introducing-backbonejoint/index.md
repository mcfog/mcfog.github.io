---
title: Introducing Backbone.Joint
thumb: joint.jpg
date: 2014-02-18
aliases:
  - /2014/02/introducing-backbonejoint
---

[接触Backbone](/2013/05/backbone-first-glance/)已经半年有余，也有机会在各种项目中实践。自己的[mcfog/Backbone.Joint](https://github.com/mcfog/backbone.joint/) 也从一个玩票儿小lib慢慢演化成一个经受过生产环境考验的backbone扩展。一直没有机会写点字介绍一下，终于在14年新春之际，码了这么一篇介绍出来。

Backbone.Joint 从13/06/14开始挖坑，7月7日登上github至今，可以说已经进入一个比较稳定的0.1版本。在Readme.md中我写的desc是这样描述这个扩展的

> another extension of backbone.js aims to support common situation with small & flexible codebase

Backbone.Joint主要针对`Backbone.View`，对`Backbone.Model`和`Backbone.Collection`的补充暂时在另一个repo [Backbone.storageEngine](https://github.com/mcfog/backbone.storageEngine) 中。由于最开始的练手性质，开发语言一直是[coco](https://github.com/satyr/coco#readme)，这是一种coffeescript的方言。

<!--more-->

下面按照源码的顺序一点点介绍，想看重点可以直接往下拉到View部分

## 基础设施 [_index.co](https://github.com/mcfog/backbone.joint/blob/0.1/src/_index.co)

#### Joint.$和Joint._
jQuery和underscore的引用，除了内部使用之外，也方便amd风格的情况下可以减少一些dep

#### Joint.assert和Joint.advice
用于内部进行一些断言式判断，在可能的情况下打出console方便调试

#### Joint.before和Joint.after
用于在函数前后追加逻辑，追加逻辑中可以通过`arguments.callee.state[0]`对象实现一些干涉逻辑。

#### Joint.Emitter

这是Joint中所有模块的基础，由`Backbone.Event`的全部事件能力和`extend`方法组成

## Deferred抽象 [defer.co](https://github.com/mcfog/backbone.joint/blob/0.1/src/defer.co)

以jQuery为对象，抽象了Deferred/Promise机制的基本接口，主要做了如下处理

+ 针对jQ1.5~1.7的非标准的then方法补丁，转发到`pipe`方法
+ `Joint.Deferred.defer`方法返回读写分离，分成resolver和promise两部分，而非原本jQ的读写混合在同一个对象中
+ 一个用于将标准Backbone事件转换成promise的`Joint.Deferred.listen`方法，不过注意只是once，而且必须保证在时间触发前调用
+ 其他的when/resolve/reject方法原样转发

## DOM抽象 [dom.co](https://github.com/mcfog/backbone.joint/blob/0.1/src/dom.co)

分离了一些需要用到的DOM操作，如果需要脱离jQ时可以统一移植这里的方法即可。注意在dom对象中存放view对象依赖$.fn.data，Zepto.js需要编译时增加依赖data模块

## **View** [view.co](https://github.com/mcfog/backbone.joint/blob/0.1/src/view.co) 和ViewModel [view_model.co](https://github.com/mcfog/backbone.joint/blob/master/src/view_model.co)

#### Fetch-Serialize-Render
Joint对View的增强首先聚焦于`render`方法，我希望(尽可能)所有的DOM写操作均通过render一个方法来进行。统一DOM写最常见也最容易维护的便是“模板渲染”模式了。Joint将其分解为Fetch-Serialize-Render三步，期望应用继承`Joint.View`后在自己的基类中分别解释这三步来组建起视图渲染体系。

+ fetchTemplate()
    
    拉取模板，一般来说每个视图都需要自己的模板，所以一般需要一个属性来指明使用哪个模板，建议直接使用`template`。模板可以是任何形式，如果是异步拉取，只要返回"thenable"对象，Joint通过`Joint.Deferred.when`，也就是`jQuery.when`来等待。
    
    默认值是`$J.Dom.html($("#" + this.template"))`，也就是在`template`属性中放DOM对象的id，取其源码。和`<script id="TPLNAME" type="x-template">`DOM标签配合。在大型应用中默认值不可取，务必实现自己的拉取模板（以及缓存）逻辑
    
+ serializeData()
    
    组装(序列化)数据，将当前视图的状态抽成一个数据对象用于填充模板。虽然想不出实用的案例，但也还是支持了"thenable"
    
    默认值是`$.extend({}, this.data, this._sync)`，`data`是默认初始化的数据对象，`_sync`和稍后的数据有关，后详。一般来说这个默认值不太需要覆盖。
    
+ renderHtml(template, data)
    
    最终渲染方法，会得到前面两部输出的模板和数据作为参数，这里将他们结合渲染为HTML代码，返回字符串即可。Joint会负责将其填充到dom中
    
    默认值是`_.template(tpl, data)`，使用underscore的模板输出，不过并没有缓存模板结果。如果需要别的模板引擎或者想缓存underscore模板结果，可以改造这个方法。同样支持异步返回"thenable"（比如DustJS的渲染就是异步的接口）
    
#### SubView体系
子视图是经常需要又在Backbone中缺失的概念。Joint给出了自己的SubView实现

+ parent掌握child的引用，将其绑定在自身的一个选择器上，渲染时先渲染自己，然后在selector找到的元素中渲染child
+ child没有parent的引用，也不了解parent的任何信息，应该可以独立于parent工作。但child上的自定义事件会冒泡的parent上。
+ 使用`setView`方法来添加子视图，如果需要一个DOM容器中存在多个子视图，则可以用`appendView`
+ child可以不是`Joint.View`的实例，但parent必须拥有`Joint.View`的特征，才能实现上述功能

当子视图是`Joint.View`时，便同样拥有一切特性，也支持多层子视图嵌套

#### 数据同步机制和ViewModel
如果视图的数据源复杂，或者多个视图共享一个数据源的时候，一个好的数据同步机制就变得很有必要`Joint.View`提供了`sync`和`unsync`两个方法来同步数据。

`sync(syncName, synchronizer, events)` 方法的核心可以简写成
    
```js
    this._sync[syncName] = synchronizer;
    this.listenTo(synchronizer, events, function(){
      this$.render();
    });
```
    
`synchronizer`是需要同步的数据对象(必须具备Backbone.Event)，`events`是绑定的事件，而`syncName`就是在`_sync`属性中的key，在默认的`serializeData`中，`_sync`会被带到数据中，从而传递给模板。

在Backbone中，`Backbone.Model`和`Backbone.Collection`都可以很容易地用作这里的`synchronizer`来使用，只要`events`传各种内置事件([参考文档](http://backbonejs.org/#Events-catalog))，即可完成Model/Collection更新时自动重绘View。

一个更好的数据对象是`Joint.ViewModel`，它基本就是一个`Joint.Emitter`，主要的区别是有一个`sync`方法，会触发`$J:sync`内置事件，而这个事件同样也会触发视图渲染，但这个事件的参数可以影响渲染过程，使后述的局部渲染成为可能。

#### 局部渲染

实际开发中，我发现经常有一些更新页面局部的需求，页面局部可能非常小，或者在DOM中不连续，或者涉及View的多个数据源，又或者含有表单等不适合反复重绘的元素，导致拆分subView难以满足需求。这时就要“局部渲染”出场了，局部渲染是在整个View的内部，通过HTML属性标识出需要更新的页面局部，从而在数据源更新时，精准地替换页面局部。

##### 绑定方法

+ View::sync时，第二个参数(synchronizer)传一个ViewModel对象
+ ViewModel中，当数据有更新时，调用ViewModel::sync(fieldNames) 即可，字段名支持空格分隔或数组表示多个字段同时更新。

    此时相关的View中的DOM元素不会像普通的渲染过程，整个被替换成新的元素，而是仅仅被标识有更新的字段的DOM元素被替换。


##### 标识

+ `j-field` 属性

    表示哪些字段更新时需要更新此DOM元素，空格分隔字段，支持有限的通配符：

    + `syncName.fieldName` syncName即调用View::sync时指定的名字，fieldName即ViewModel::sync时传的名字。
    + `syncName.*` syncName所指的ViewModel中任意字段被更新时均更新此元素

+ `j-id` 属性

    Joint根据这个属性来确认渲染前后DOM元素的对应关系，即渲染后，字段有改变的DOM元素代码覆盖到DOM树上`j-id`相同的节点。

    默认值是和`j-field`相同。如果页面上存在`j-field`相同的元素时，则必须指定不同的`j-id`来避免混淆


    

----

写完感觉好乱，以后慢慢按照Tutorial 和 Document整理吧


