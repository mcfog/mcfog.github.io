---
title: An elegant design to unify async and sync callback based on promise object
date: 2013-06-09
categories: javascript
tags: [design, jquery]
thumb: promise.jpg
aliases:
  - /2013/06/an-elegant-design-to-unify-async-and-sync-callback-based-on-promise-object/
---

在JS代码的设计中，“回调”是非常重要而有效的手段，这里讨论的是框架需要获取回调结果的，更加注重IoC的回调。（另一种回调的使用往往更接近订阅者模式，强调信息的单向下发）往往框架代码需要获取某些信息，但如何获取的逻辑需要留待使用者实现，此时回调就是非常直接的选择。

获取回调的输出信息有最直接的使用返回值（同步），但异步有时是无法避免的。本文不准备讨论设计回调时应该设计成同步返回还是异步返回，而是讨论如何简洁而优雅地兼容两者，使回调既能够直接返回结果，又可以通知框架等待异步返回结果。

<!--more-->

在JS中，可以和回调交互的地方一共有3个，分别是`this`对象、参数(`arguments`)和返回值。我们先看看利用this和arguments来实现的兼容同步异步的大致做法，再来看看利用返回值并引入deferred机制后对代码的提升。

### 替换this
[backbone layoutmanager](http://layoutmanager.org)中采用了替换`this`对象的方式实现兼容同步和异步的模板获取/渲染。好处是业务代码看起来相对清晰，但由于替换掉了this，比较依赖this的逻辑会碰到麻烦。另外实现相对复杂

``` javascript
//同步回调
function fetchSync(path) {
  return _.template($(path).html());
}
//异步回调
function fetchAsync(path) {
  var done = this.async();

  $.get(path, function(contents) {
    done(_.template(contents));
  }, "text");
}
```

大致框架实现

``` javascript
var makeAsync = function(done) {
    var handler = {};
    handler.async = function() {
        handler._isAsync = true;
        return done;
    }

    return handler;
}

exports.AwesomeFunction = function(path, callback) {
    var done = function(result) {
        //handle result
    };
    var handler = makeAsync(done);
    var result = callback.call(handler, path);
    if(!handler._isAsync) return done(result);
}
```
可以发现为了制造出this对象以及判断是否异步的逻辑比较生硬，另外替换掉this的做法不是所有场景都可以接受。回调和框架的耦合也比较强，离开了框架的话回调由于有`done=this.async()`基本没法用。

### 追加arguments
对参数动手脚来实现异步/同步兼容的话，一般选择在参数列表最前/最后追加 _回调的回调_ 用来和框架交换数据。

``` javascript
//同步回调
function fetchSync(path, done) {
  done(_.template($(path).html()));
}
//异步回调
function fetchAsync(path, done) {
  $.get(path, function(contents) {
    done(_.template(contents));
  }, "text");
}
```

大致框架实现

``` javascript
exports.AwesomeFunction = function(path, callback) {
    callback(path, function(result) {
        //handle result
    });
}
```
比起上一种方法，这种方法的优点是实现简洁明了，如果需要可以保持this，缺点是需要改变函数签名，对签名不定长的情形比较恶心。另外，同步返回的回调不能简单地return返回值也是比较严重的缺点。

### __利用返回值和deferred机制__

[deferred/promise机制](http://wiki.commonjs.org/wiki/Promises/A)天生对回调非常友好，只要定义回调返回普通值=同步，返回promise=异步，就可以很优雅地做到兼容同步和异步。所需的仅仅是一个简单的包装器`isPromise(foo) ? foo : makePromise(foo)`的实现。这里以jQuery为例，`jQuery.when`就是一个不错的包装器

``` javascript
//同步
function fetchSync(path) {
  return _.template($(path).html());
}
//异步
function fetchAsync(path) {
  return $.get(path, "text").then(function(contents) {
    return _.template(contents);
  });
}
```

大致框架实现

``` javascript
exports.AwesomeFunction = function(path, callback) {
    $.when(callback(path)).then(function(result) {
        //handle result
    }, function() {
        //handle error
    });
}
```

利用返回值的好处在于完整保留了`this`和`arguments`，同步的情形下无论回调依赖this还是参数都无需包装回调，而异步的情况下，返回promise的回调的适用范围要比前两种机制的回调广泛的多。另外promise天生比较完备的错误处理，`jQuery.when`对多个参数的支持都使得这种方法在实践中具备很强的可操作性。

### 举一反三……

无论是设计框架还是切分方法，对于“异步获取信息”的情景来说，返回一个发布者(promise)比接受回调要更加灵活。

+ 接受回调会影响方法签名，而且通常浪费了返回值，返回发布者对象对整体程序的语义有很大地提升（参数=输入，返回值=输出）
+ 接受回调限定了订阅者的数量为1（兼容0和多个都需要不少兼容逻辑），而发布者通常不关心订阅者的数量
+ 接受回调要求调用方在调用时就准备好所有的订阅者，无法追加订阅者，而发布者不仅可以很容易地追加订阅者，其本身作为一个对象还可以被传递
+ 发布者可以有更灵活的方法来处理嵌套、并发以及异常等各种情况

--EOF--


