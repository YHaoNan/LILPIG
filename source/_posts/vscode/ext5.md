---
title: VSCode 扩展开发(五) 悬停提示、跳转定义
date: 2019-06-07 09:33:54
tags: [vscode,editor]
categories: VSCode
---

## 学习目标
经过本章，您将会学到VSCode提供的一些其他Provider的用法，比如悬停提示、跳转定义等。

## 开篇寒暄
上一章我们学了用于自动完成的`CompletionItemProvider`，使用VSCode提供的ProviderAPI，我们可以方便的扩展VSCode的功能，VSCode还提供了一些其他的Provider，这一章里面我们研究其中常用的几个。

## 悬停提示
VSCode中的悬停提示由`HoverProvider`提供，我们可以在`extension.ts`中调用`vscode.languages.registerHoverProvider`来注册一个Provider。

这是一个HoverProvider的实现类：
```typescript
import * as vscode from 'vscode'

export class BrainfuckHoverProvider implements vscode.HoverProvider{
    provideHover(document: vscode.TextDocument, position: vscode.Position, token: vscode.CancellationToken): vscode.ProviderResult<vscode.Hover> {
        //do something...     
    }

}
```

这个和`CompletionItemProvider`也差不多，只不过是当鼠标放置在源代码上时`provideHover`会被触发，我们通常在这个方法里通过`document`和`position`取到当前鼠标位置的那个单词，然后做一些提示操作。

我们复制了上一章的项目，基于上一章修改，地址在[这里](https://github.com/YHaoNan/vscode-tutorial/tree/master/vsc-extensions-tutorial-5)

编写bf代码时，我们经常碰到一堆加号或减号，为了确定它们到底是多少个，我们经常要一个一个数，而且很容易出问题。所以我们决定扩展插件的功能，只要鼠标悬浮在一连串的加或减之间时，就弹出悬浮提示，告诉用户有多少个运算符，效果如下：

![效果](https://s2.ax1x.com/2019/06/07/VwY8gK.gif)

那么如何制作捏？我们需要先找出悬浮位置的一段连续的加或减，我们可以按照昨天编写代码提示时的方法自己编写一个寻找函数，但是这次我们使用VSCode给我们提供好的方法，那就是`document.getWordRangeAtPosition`，这个方法传入一个position和一个正则表达式，从position处寻找符合正则表达式的字符串，并返回范围对象。

```typescript
import * as vscode from 'vscode'
import { disconnect } from 'cluster';

export class BrainfuckHoverProvider implements vscode.HoverProvider{
    provideHover(document: vscode.TextDocument, position: vscode.Position, token: vscode.CancellationToken): vscode.ProviderResult<vscode.Hover> {
        //先判断悬停的字符是否是加或减
        const opt = document.lineAt(position.line).text.charAt(position.character);
        if(opt!='+' && opt!='-')
            return null;
        //获取单词 这里的正则就是 \++(加号出现一次以上) 或\-+(减号出现一次以上)
        const wordRange = document.getWordRangeAtPosition(position,new RegExp('\\'+opt+'+'));

        //返回一个vscode.Hover对象，这就是一个悬浮提示的文本信息，我们这里构造一个MarkdownString用于显示文本，当有选中单词位置有多个悬浮提示的话，将放在一起显示
        return wordRange?new vscode.Hover(new vscode.MarkdownString('### BF Opt Counter  \n\n * **Opt: `'+opt+'`**\n\n* **Length: `'+(wordRange.end.character-wordRange.start.character)+'`**')):null;
    }

}
```

## 跳转定义
跳转定义的功能由`DefinitionProvider`提供，该Provider有如下方法：
```typescript
import * as vscode from 'vscode'
export class BrainfuckDefinitionProvider implements vscode.DefinitionProvider{
    provideDefinition(document: vscode.TextDocument, position: vscode.Position, token: vscode.CancellationToken): vscode.ProviderResult<vscode.Location | vscode.Location[] | vscode.LocationLink[]> {
        
    }
}
```
`provideDefinition`在按下Ctrl并点击时生效，position是点击的位置，我们写一个简单的Brainfuck的循环跳转插件，就是这样的效果：

![效果](https://s2.ax1x.com/2019/06/07/V0lWF0.gif)

这个插件相比实际业务逻辑中的跳转定义简单很多，但弄懂了这个，我估计写出一个真正的跳转定义插件也不在话下。

我们需要利用栈和散列表来做这个`[`配对功能，具体代码我们就不放上来了，可以在本章的github上找到，提供了两个API：
```
getLoopOptPairMap(document): Map<string,Position>
    - 返回括号对的散列表
getMatchedOpt(map,position): Position
    - 返回map中与position对应的位置 也就是找到文档中对应的另一个括号的position对象
```

*（PS:这个配对的代码还是别跟我学了，看看就行了，我感觉我Java写多了，写啥都像Java 等我深入学习一下TypeScript后也许会再回来改这个代码）*

然后我们在`BrainfuckDefinitionProvider`中调用这个API进行跳转：
```typescript
import * as vscode from 'vscode';
import * as util from './util';


export class BrainfuckDefinitionProvider implements vscode.DefinitionProvider{
    provideDefinition(document: vscode.TextDocument, position: vscode.Position, token: vscode.CancellationToken): vscode.ProviderResult<vscode.Location | vscode.Location[] | vscode.LocationLink[]> {
        const char = document.lineAt(position.line).text.charAt(position.character);
        if(char != '[' && char != ']')return null;
        const map = util.getLoopOptPairMap(document);
        const target = util.getMatchedOpt(map,position);
        if(target)
            return new vscode.Location(document.uri,target);
    }

}
```
`provideDefinition`方法中，如果找到了应该跳转的定义位置，就返回一个Location对象，如果没找到，则不返回(undefined)或者返回null。

构造Location对象需要传入跳转到的文件的uri，因为我们是在本文件中跳转，所以直接引用了`document.uri`，第二个是跳转到的位置。

## 参考
* [HoverProvider](https://code.visualstudio.com/api/references/vscode-api#HoverProvider)