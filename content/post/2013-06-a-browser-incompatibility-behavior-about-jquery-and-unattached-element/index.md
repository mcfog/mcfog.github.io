---
title: "a browser incompatibility behavior about jquery and unattached element"
date: 2013-06-23
categories: javascript
tags: [dom, jquery]
thumb: browsers.jpg
aliases:
  - /2013/06/a-browser-incompatibility-behavior-about-jquery-and-unattached-element/
---

### 摘要

+ jQuery.fn.hide 会读取元素的display样式
+ 读取尚未插入dom树的脱机元素的样式时，不同浏览器行为不同

### 场景
在一段显示错误提示的代码中，发现firefox下显示异常，错误提示tips的宽度没有自适应内容而是占满了整个容器。inspect后发现，本应是由CSS控制的`display:inline-block`被元素的`style="display:block"`覆盖，chrome下没有这个问题

<!--more-->

### jQuery.fn.show
断点调查后发现在jQ的`show`方法中有这样[一段代码](https://github.com/jquery/jquery/blob/1.7.2/src/effects.js#L57)

``` js
    elem.style.display = jQuery._data( elem, "olddisplay" ) || "";
```

也就是说jQ为了hide和show后能够保持原有的display属性，会将元素原来的display属性暂存起来，这样一个原本`inline-block`的元素经过hide和show之后不会被改成`block`，firefox下问题tips元素的`jQuery._data( elem, "olddisplay" )`值是`block`，所以show的行为没有问题，问题应当是hide的时候这个计算算错了

### jQuery.fn.hide

再来看hide的[相关代码](https://github.com/jquery/jquery/blob/1.7.2/src/effects.js#L81)，中规中矩，用`jQuery.css`获取display属性

``` js
display = jQuery.css( elem, "display" );

if ( display !== "none" && !jQuery._data( elem, "olddisplay" ) ) {
	jQuery._data( elem, "olddisplay", display );
}
```

继续追踪后在`jQuery.css`的实现中发现了[浏览器不一致的来源](https://github.com/jquery/jquery/blob/1.7.2/src/css.js#L179)

```js
if ( (defaultView = elem.ownerDocument.defaultView) &&
		(computedStyle = defaultView.getComputedStyle( elem, null )) ) {

	ret = computedStyle.getPropertyValue( name );
```

这里的elem是一个未加入dom树的“脱机”元素，`defaultView`火狐和chrome计算后都等于`window`，`name`是要获取的属性名，这里是display。chrome下`ret`拿到的值是空字符串，而火狐下拿到的是`block`，从而火狐会将这个值写入`olddisplay`，最终导致show的时候元素被加上`display:block`

### 观察&总结
最后我计算了`getComputedStyle(document.createElement('div'), null)`，发现chrome基本上都是空，而火狐则填充了各种各样的默认属性（顺带一提IE9的行为和火狐类似）

![unattached styles](/images/201306/unattached-styles.jpg)

究竟哪种行为才符合标准已经不重要，总之**避免读取脱机元素的样式**才能避开这里的问题，虽然脱机元素操作完再插入DOM是性能友好的做法，但浏览器兼容性多数情况下还是比性能更重要的

### 本次问题代码fiddle
chrome下显示inline-block，火狐/IE9则显示block

{{< jsfiddle PrmgJ "result,js,css" >}}


