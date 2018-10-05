---
title: Undocumented stylus built-in functions
date: 2013-06-14
categories: css
tags: stylus
thumb: stylus.jpg
aliases:
  - /2013/06/undocumented-stylus-built-in-functions/
---

Stylus的文档更新相对不怎么及时，这里记录一下如何找到内置函数的列表，目前最新的内置函数列表，以及其中一些比较有用的文档中没有提及的

<!--more-->

### 获取最新的内置函数

源码文件 <https://raw.github.com/LearnBoost/stylus/master/lib/functions/index.styl>

用浏览器打开，执行

``` javascript
document.body.innerHTML.match(/^([-\w]+)\(.+\)/gm).join('\n')
```

来获取最新的内置函数列表，当然由于文件不长，直接目测也行

### 目前最新的内置函数列表及简单介绍

+ 断言/打印
    + -string(arg)
        + debug用来打印变量用
    + require-color(color)
        + 要求输入是一个颜色，否则在控制台报错，下同
    + require-unit(n)
    + require-string(str)
+ 数学
    + math(n, fn)
        + 返回js的 `Math[fn](n)` 单位维持不变
    + adjust(color, prop, amount)
    + abs(n)
    + sin(n)
    + cos(n)
    + min(a, b)
    + max(a, b)
    + ceil(n, precision = 0)
    + floor(n, precision = 0)
    + round(n, precision = 0)
    + sum(nums)
    + avg(nums)
    + remove-unit(n)
        + 移除单位
    + percent-to-decimal(n)
    + odd(n)
        + 是否为单数
    + even(n)
        + 是否为双数
+ 获取颜色信息
    + alpha(color)
    + hue(color)
    + saturation(color)
    + lightness(color)
    + light(color)
        + 颜色是否为亮？
    + dark(color)
        + 颜色是否为暗？
+ 颜色调整
    + desaturate(color, amount)
        + 减小饱和度
    + saturate(color, amount)
        + 增加饱和度
    + darken(color, amount)
        + 减小亮度
    + lighten(color, amount)
        + 增加亮度
    + fade-out(color, amount)
        + 降低透明度
    + fade-in(color, amount)
        + 提升透明度
    + spin(color, amount)
        + 转色调
    + mix(color1, color2, weight = 50%)
        + 混色
    + invert(color)
        + 反色
+ helpers
    + last(expr)
    + keys(pairs)
    + values(pairs)
    + join(delim, vals...)
    + add-property(name, expr)

--EOF--
