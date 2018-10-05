---
title: grunt-coco released
date: 2013-06-14
categories: javascript
tags: [grunt, coco]
thumb: grunt-coco.jpg
aliases:
  - /2013/06/grunt-coco-released/
---

最近在看各种自动化工具，有集成化程度高，比起编译工具更像“IDE减文本编辑”的[mimosajs](http://mimosajs.com)，也有被业界广泛认可，开放易配置的[gruntjs](http://gruntjs.com)。作为一个[coco](https://github.com/satyr/coco)语言支持者，看自动化工具当然是先看能不能支持coco。

<!--more-->

grunt的开放模块机制非常稳定靠谱(所以才会被社区高度认可的吧)，根据官网文档亦步亦趋很快就实现出了支持编译coco文件的 `grunt-coco`。说来这是我第一个上npm的repo呢。

### 使用方法
安装

```shell
npm i grunt-coco --save-dev
```

在Gruntfile中引用

```js
grunt.loadNpmTasks('grunt-coco');
```

配置： 目前配置仅支持`separator`、`bare`和`join`，含义和[grunt-contrib-coffee](https://github.com/gruntjs/grunt-contrib-coffee#options)里的相同。`globals`配置项等coco下个版本发布把bug修掉以后就可以有效。

最后附repo传送门 <https://github.com/mcfog/grunt-coco>


##### 题外话
mimosa是一个自动工具和todo框架的合成体，其开放的module机制相对grunt不算清晰，所幸作者正在积极开发中，丢去的[Pull Request](https://github.com/dbashford/mimosa/pull/214)很快就被merge了，下个版本的mimosa就将支持coco。


