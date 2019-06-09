---
title: 手动实现响应式布局
date: 2019-05-19 14:55:27
tags: [前端,css,html]
categories: 前端
---
最近突然心血来潮学前端。。。

在研究BootStrap的时候就很好奇他这个响应式布局是怎么实现的，为什么在不同的设备上可以不借助js实现不同的效果？很迷。。。

于是我就找了找资料，并且手动实现了一个响应式布局的例子。不过我要先说一下，我前端基础很渣的，基本没咋学过，啊哈哈哈。

看下效果，因为就是练习，所以随手写了个页，比较丑：

![GIF](http://prgs9av8q.bkt.clouddn.com/compared.gif)

可以看到随着屏幕的缩小文章和左侧栏从左右排列变成了上下排列，并且左边的一排ITEM没了，右上角多了一个`MENU`按钮，按下之后之前的那一排ITEM会弹出。

这就是响应式布局的核心，就是根据屏幕大小来分配不同的布局。

### 如何实现？
提供了两份css，然后通过指定`media`属性来确定什么条件下使用哪份css中的样式。

```html
<head>
...
    <meta name="viewport"
          content="width=device-width,initial-scale=1.0">
    <link rel="stylesheet" href="common.css">
    <!-- 指定设备屏幕小于等于1000的时候使用该css中的样式 -->
    <link media="(max-width:1000px)" rel="stylesheet" href="mobile.css">
</head>
```
那公共样式得写两份？其实不用，第二个条件满足时只是替换掉第一个中冲突的和添加第一个中没有的。

当第二个条件不满足时，第二个中添加的和修改的玩意都失效。

你可以理解为和`hover`这种伪类的工作方式差不多。

### 代码
[https://github.com/YHaoNan/front-end/tree/master/responsive-layout](https://github.com/YHaoNan/front-end/tree/master/responsive-layout)
