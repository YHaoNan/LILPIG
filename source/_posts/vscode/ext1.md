---
title: VSCode 扩展开发(一) 概述
date: 2019-06-03 07:22:54
tags: [vscode,editor]
categories: VSCode
---
## 目录
* [VSCode 扩展开发(一) 概述](#)
* [VSCode 扩展开发(二) 第一个插件](/post/vscode-ext2)   
    - 使用官方工具生成第一个插件，并了解它的工作原理
* [VSCode 扩展开发(三) Command详解](/post/vscode-ext3)   
    - 详解`Command`，并且实现一个复制文件路径的小插件
* [VSCode 扩展开发(四) Output、自动完成和Snippets](/post/vscode-ext4)   
    - 讲解`OutputChannel`、`CompletionItemProvider`和`Snippets`，实现针对Brainfuck语言的类似Emmet的插件
* [VSCode 扩展开发(五) 悬停提示、跳转定义](/post/vscode-ext5)  
    - 讲解`HoverProvider`、`DefinitionProvider`，为Brainfuck插件添加功能
* [VSCode 扩展开发(六) 用户设置](/post/vscode-ext6)  
    - 讲解如何向用户提供设置项，完善Brainfuck插件
* [VSCode 扩展开发(七) 实战-翻译插件(没写)](/post/vscode-ext7)  
    - 实战插件，利用HoverProvider和OutputChannel的功能完成翻译插件
* [VSCode 扩展开发(八) 实战-Markdown图片上传(没写)](/post/vscode-ext8)  
    - 实战插件，利用UCloud API制作嵌入在VSCode中的Markdown图床

---

以下是还没写的 并且还没打算的
* [VSCode 扩展开发(九) Snippets](/post/vscode-ext9)
* [VSCode 扩展开发(十) WebView](/post/vscode-ext10)
* [VSCode 扩展开发(十一) TreeView](/post/vscode-ext11)
* [VSCode 扩展开发(十二) 实战-上班偷懒之知乎插件](/post/vscode-ext12)
* [VSCode 扩展开发(十三) 实战-上班偷懒之微信插件](/post/vscode-ext13)
* [VSCode 扩展开发(十四) 打包与发布](/post/vscode-ext14)
* [VSCode 扩展开发(番外一) ColorTheme](/post/vscode-ext15)
* [VSCode 扩展开发(番外二) IconTheme](/post/vscode-ext16)
* [VSCode 扩展开发(番外三) 自定义Language](/post/vscode-ext17)
#### 说明
这个目录是暂时的，因为还没写完，有可能一边写一边改。


## 起源
目前国内外能找到的VSCode Extensions开发的教程大都说的很浅，用官方的工具生成个示例就算讲完了，就算有比较深入的，也很零碎，不太系统，再加上官方给的文档不太友好，系统的学习VSCode Extensions开发有点难度。所以我打算写一套关于这个的课程，由浅入深的学习VSCode Extensions开发。
## 技术栈
VSCode Extensions开发的技术栈如下：
* Nodejs
* TypeScript

如果您对以上两门技术不是很了解，请先去了解下。不过这里我要说一下，由于Nodejs是致力于用JavaScript开发后端的，所以你能看到的课程基本都是面向后端开发的。但是Nodejs又不仅仅能开发后端，它的应用场景很多，VSCode Extensions就用到了它，所以您如果只是想学习VSCode Extensions开发的话，大可不必把网上的教程全部撸一遍，这里我推荐菜鸟教程的[Nodejs课程](https://www.runoob.com/nodejs/nodejs-tutorial.html)。

TypeScript是微软推出的一款编程语言，是JavaScript的超集，用于解决JavaScript没有静态类型检测的问题。所有的TypeScript代码最后都要编译成JavaScript代码才可以在生产环境中运行。所以必然也可以直接用JavaScript开发，不过本套课程采用TypeScript，关于二者的选择，请您自己做决定。

TypeScript可以在[TypeScript handbook 中文版](https://zhongsp.gitbooks.io/typescript-handbook/content/doc/handbook/)中学习。
## 为什么要学习VSCode扩展开发
关于这个问题就要说道为什么当初我要学习VSCode扩展开发了。

既然是搞技术，大家肯定会写技术博客吧！你们博客的图片都放哪？大部分人选择了用各大云服务提供商的对象存储服务搞一个图床，那就会涉及到图片上传的问题。

可是我不能每次上传图片都去登录到云服务提供商的控制台，上传完成后再手动把链接复制回来？那岂不是太麻烦了吗？

于是，有的人开发了图床工具，每次我们只需要把图片保存，拖动到图床工具，他就会自动返回给你链接，这样我们就不用登录到控制台了。

不过用着用着，你会发现这还是有点麻烦，每次还要从编辑器切换到图床工具，传好了再切回来，体验真的太差了，而且浪费了很多时间和精力。

于是你想到去VSCode的扩展商店里搜索对应关键字，幸运的人可能碰到了别人开发的基于他这个对象存储厂商的图床工具，直接使用，不幸的人\(比如我用的是UCloud\)会发现扩展商店里根本没有自己想要的插件。

于是我就花了几天时间学习了VSCode的插件开发，并且开发出了属于我自己的图片自动上传插件，并且还支持备份功能。

你平常最多的时间就是在编辑器里，编辑器是否强大决定了你的工作效率，而决定21世纪的编辑器是否强大的唯一标准就是扩展插件是不是够多。不过，总是有的时候你会遇到一个插件市场中已有插件满足不了的需求，或者可以满足但不是最优的需求，所以我们需要自己掌握插件开发，在必要的时候造轮子出来提高工作效率。

## 目标
目标是做一套让中国开发者可以快速入门VSCode扩展开发的教程。

本套文章深入浅出讲解VSCode扩展开发，准备在个人博客\(个人博客流量有限QAQ~~\)、简书和Github上发布。

本套文章的源码都会发布到Github上。

如果可能的话，会在Bilibili发布教程视频或直播。

## 说明
#### 错误
因为本人的VSCode扩展开发主要学习来源是官方文档，但是本人英语不是很好，所以哪里有错误的地方欢迎及时在Github的Issue上指正。

#### 代码
关于本篇文章的配套源码，我会发布在Github上，以每一课一个项目的形式。