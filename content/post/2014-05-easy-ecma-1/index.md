---
title: Easy ECMA 1
date: 2014-05-08
thumb: ecma_org.png
aliases:
  - /2014/05/easy-ecma-1/
---

简单易懂的ECMA规范导读1
========

### 序
最近混[SF](http://segmentfault.com/)，恰巧又逢工作方面有了NodeJS的机会，迫切地有教别人怎么写JS的需求，
我发现JS这个东西其实真没那么容易理解。

为了加深和纠正自己对JS的理解，也为了以后能直接甩别人一脸文章，所以开始挖这样一个大坑：__简单易懂的ECMA规范导读__。
希望能以专题的形式有线索地基于ECMA标准介绍Javascript的方方面面。本文不是ECMA标准的中文翻译，也不是Javascript的入门教程，
本文虽然以JS的常见问题切入，但并不适合想要快速了解这些问题的人（Google才是快速了解问题的正解）。
本文的聚焦于标准如何决定了JS的各种行为，JS引擎的水面下在发生些什么。

本文描述的是[ECMA262的5.1版本](http://www.ecma-international.org/ecma-262/5.1/) 也是现在最为流行和主流的标准，
现代浏览器和NodeJS默认均遵循此标准。尽量以英文原版为基础，为了流畅，可能会使用某些名词的中文翻译，
但会将匹配的英文名词以`此种样式`中出现一次以避免误解。

### Topic1. that's this

我们的第一个话题是：this指向哪里?

<!--more-->

#### 什么是this
[11.1.1 The this Keyword](http://www.ecma-international.org/ecma-262/5.1/#sec-11.1.1)

> The this keyword evaluates to the value of the ThisBinding of the current execution context.

计算this关键字时，取当前执行上下文的ThisBinding的值

[10.3 Execution Contexts](http://www.ecma-international.org/ecma-262/5.1/#sec-10.3)

执行上下文`Execution Context` 从逻辑上形成栈结构，栈顶(活跃)的执行上下文包含了追踪当前正在执行的代码的全部状态。

执行上下文包含了LexicalEnvironment、VariableEnvironment和ThisBinding三部分，在这个话题中我们主要关心ThisBinding，
也就是代码中出现this所代指的值的绑定

#### 全局代码中的this

从最简单的开始

[10.4.1 Entering Global Code](http://www.ecma-international.org/ecma-262/5.1/#sec-10.4.1)
在进入全局代码的流程中，规范明确指出：全局代码对应的执行上下文中，

> Set the ThisBinding to the global object.

所以全局代码中，this指向全局对象

#### 函数调用表达式时提供的this值

__注意：this值`this value`是不同于this关键词的概念，是调用[[Call]]内部方法的参数之一，并不等同于用户代码中的this关键字__

函数调用表达式`CallExpression`的过程中，按照[标准的描述](http://www.ecma-international.org/ecma-262/5.1/#sec-11.2.3)，
计算this值的伪代码如下

    if Type(ref) is <Reference>
        if IsPropertyReference(ref)
            thisValue := getBase(ref)
        else # assert Type(getBase(ref)) is <Environment Record>
            thisValue := getBase(ref).ImplicitThisValue()
    else
        thisValue := undefined

+ ref是函数调用参数左侧(括号左侧)的表达式计算的结果
+ `<Reference>`引用类型常见的有
    + 标示符引用[Identifier Reference](http://www.ecma-international.org/ecma-262/5.1/#sec-11.1.2)
     即变量引用，引用base是环境记录`Environment Record`
    + 字面量引用[Literal Reference](http://www.ecma-international.org/ecma-262/5.1/#sec-11.1.3)
    引用base也是环境记录
    + 属性访问[Property Accessors](http://www.ecma-international.org/ecma-262/5.1/#sec-11.2.1)
    包括点运算和`[]`运算，引用base是左值
+ 非引用类型常见的有
    + 全部ECMA内置函数和所有用户定义函数的返回结果（例外是host objects，也就是假设DOM之类的宿主对象如果需要，
    可以定义一些函数返回引用）
+ 环境记录是前述的执行上下文中的LexicalEnvironment和VariableEnvironment的构成要素
    + 不考虑with语句的话，环境记录只有[Declarative Environment Records](http://www.ecma-international.org/ecma-262/5.1/#sec-10.2.1)一种，它的`ImplicitThisValue`始终返回undefined

综上所述，排除with语句的情况下，想让thisValue不是undefined，就只有属性访问一种办法而已。

计算得到thisValue后，调用被调函数func的`[[call]]`内部方法，提供thisValue作为this的值


#### new表达式时提供的this值

new表达式`NewExpression`的[执行过程](http://www.ecma-international.org/ecma-262/5.1/#sec-11.2.2)
基本上委托给了`[[Construct]]`内部方法，我们看这个方法的[定义](http://www.ecma-international.org/ecma-262/5.1/#sec-13.2.2)，
关注其中第8步

> Let result be the result of calling the [[Call]] internal property of F, providing
> obj as the this value and providing the argument list passed into [[Construct]] as args.

其中F是构造函数，而obj是本次新建的对象，非常清楚。

#### this值如何转变为ThisBinding

前两节我们描述了两种触发函数体内代码的办法各自如何构造this值，但正如前述，this关键字的值是执行上下文中的ThisBinding决定的，
this值转变为ThisBinding的过程发生在调用`[[Call]]`内部方法时，我们看[标准](http://www.ecma-international.org/ecma-262/5.1/#sec-13.2.1)

> Let funcCtx be the result of establishing a new execution context for function
> code using the value of F's [[FormalParameters]] internal property, the passed
> arguments List args, and the this value as described in 10.4.3.

阿哈，第一步就是建立新的执行上下文，其中thisBinding的构建在[10.4.3 Entering Function Code
](http://www.ecma-international.org/ecma-262/5.1/#sec-10.4.3)中描述,

    if function-code is <strict code>
        ThisBinding = thisArg
    else if thisArg is null or thisArg is undefined
        ThisBinding = global object
    else if Type(thisArg) is not <Object>
        ThisBinding = ToObject(thisArg)
    else
        ThisBinding = thisArg

这就是魔术的秘密

+ null 和 undefined 的this在此时会绑定为全局对象
+ 其他三种非对象 String / Boolean / Number 在此时被转化为对象（auto boxing）
+ 但是，__strict mode下不进行任何转换__

That's `this` in Javascript.

-----

本节思考题：

+ 找找看关于apply和call的标准，为何this会变？
+ 找找看关于bind的标准，哪里体现了bind后的函数内的this无视环境和调用方式，总是固定值？
+ with语句并不是一个良好的实践，所以我避开了它，不过这可以作为阅读标准的练习：with语句如何影响this关键词？请试着写一句代码展示此种影响
+ eval的内部的this如何确定？


