---
title: VSCode 扩展开发(三) Command详解
date: 2019-06-03 13:15:54
tags: [vscode,editor]
categories: VSCode
---

## 学习目标
本篇对`package.json`中对Command的各种设置都进行了讲解，学完本章你会知道VSCode中的Command究竟是什么，都能干什么以及如何操作它们。

## 开篇寒暄
上一篇中我们已经对Command有了初步的认识，这一篇我们来详细的了解一下它。

Command是VSCode扩展开发中非常重要的一个概念，扩展开发中很多东西都是基于Command的，虽然你可能没有意识到它的存在，但是它已经确确实实的已经为你服务了无数回。

每次你使用`ctrl+/`快捷键生成注释的时候，就是命令在为您服务；每次打开一个文件夹时也是命令在为您服务，还有很多场景都是它在为您服务。

![图片](https://s2.ax1x.com/2019/06/03/VJG9Fe.gif)

从上面可以看出这两个操作是完全一样的，如果你去看VSCode的键盘映射表，也会看到按下`ctrl+/`时，实际上就是执行了那个命令。

## 创建一个新的命令
我想，上一篇文章过后如果你自己练习了，对于创建一个自己的命令应该轻车熟路了，不过没关系，我们再来一次。

我创建一个新的项目，代码在[这里](https://github.com/YHaoNan/vscode-tutorial/tree/master/vsc-extensions-tutorial-3)。


现在，我们在`package.json`中的`contributes`节点添加新的命令，这个命令的功能是将用户当前编辑文件的路径复制到剪贴板。

**PS:从这篇教程以后，VSCode生成器生成的默认命令，也就是Hello World的所有相关代码会被我删除**
```json
"contributes": {
    "commands": [{
        "command": "ext.getCurrentPath",
        "title": "Get Current Path"
    }]
},
```

然后，在`activationEvents`节点中将我们的命令设置为VSCode激活的条件。

```json
"activationEvents": [
    "onCommand:ext.getCurrentPath"
]
```

然后去`extension.ts`中注册我们的命令并绑定回调函数。

```typescript
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
	context.subscriptions.push(
		vscode.commands.registerCommand('ext.getCurrentPath',()=>{
			//To do something...
		})
	);
}

export function deactivate() {}
```
这样一来我们的命令就创建好了～

## 实现逻辑
我们实现逻辑之前，首先要考虑我们需要什么。

我们需要的是将用户当前编辑文件的路径复制到剪贴板。

这个功能就需要我们对VSCode Extensions提供的API有一些了解了，但是我们目前不了解，我们能不能用点别的办法呢？

诶？VSCode其实提供了一些命令我们可以直接调用，其中恰好就有我们需要的命令。

我们可以打开命令面板，输入`Show All Commands`查看所有命令，好像正巧有我们的命令。

![img](http://nsimg.cn-bj.ufileos.com/img-1559542195660.png)

不过要执行一个命令我们需要的是该命令的ID，而不是上图中的`title`，我并不知道一个很好的方法去查询命令的ID，官方给没给所有内建命令的列表我也不知道。我目前的办法是通过`keybindings`的json文件或者在`extensions.ts`中调用`vscode.commands.getCommands`方法获取全部命令的ID。

不管怎么样，最后我们知道了复制当前文件路径的命令ID是`copyFilePath`，我们在`ext.getCurrentPath`的回调方法中调用它。

```typescript
vscode.commands.registerCommand('ext.getCurrentPath',()=>{
    vscode.commands.executeCommand('copyFilePath');
})
```

运行我们的代码，我们会发现确实可行，每次我们执行该命令都会复制当前文件的路径。

你可能会说，你这不就是包装了一下吗，实际上用的还是人家的命令，那我何苦不直接用别人的命令呢？

其实我们在做VSCode扩展开发时\(做其他的开发也一样\)，如果有一个可靠的，可以直接利用的东西，我们就可以拿过来直接用，并且扩展它，没必要重造轮子\(当然学习的时候重造轮子还是有必要的，所以我会在后面把这个自己实现一次\)。如果说直接拿来用人家的是错误的，那我们哪天不是在别人的API下工作呢？那我们应该每天都在犯错吧！VSCode提供了大量的好用的内建命令，我们可以直接使用它们。

## 扩展
我们现在不仅需要复制当前文件路径，我们还希望能直接粘贴到当前的编辑窗口中，并且给用户一个提示。现在我们就需要扩展它。

经过查找，我查到了VSCode有一个`editor.action.clipboardPasteAction`命令用于给当前的编辑窗口发送粘贴动作，所以，我们继续调用VSCode提供的命令完成我们的功能。

```typescript
vscode.commands.registerCommand('ext.getCurrentPath',()=>{
    vscode.commands.executeCommand('copyFilePath');
    vscode.commands.executeCommand('editor.action.clipboardPasteAction');
    vscode.window.showInformationMessage('Done...');
})
```
## menus

### 在编辑器的右键菜单中显示
假设你的这个插件运行了两星期，你突然觉得，每次都要打开命令窗口，输入`Get Current Path`，太麻烦了，你希望可以通过右键菜单来执行命令。我们可以在`package.json`中通过向`contributes`节点添加`menus`完成需求。


```json
"menus": {
    "editor/context": [
        {
            "command": "ext.getCurrentPath"
        }
    ]
}
```
在menus中添加一个`editor/context`节点，这个节点是一个列表，他其中的每一个项目都是编辑器右键菜单中的一个列表项，我们把我们的命令配置到其中，再运行，我们的命令就会出现在菜单中。

![img](http://nsimg.cn-bj.ufileos.com/img-1559544012032.png)

### Group
现在它的位置是在最后一个，我们想让它往上一些，怎么办呢？这时可以通过`group`来指定它的位置。

```typescript
"menus": {
    "editor/context": [
        {
            "command": "ext.getCurrentPath",
            "group": "navigation"
        }
    ]
}
```
再运行，它就在上面了，为什么指定navigation后它就会在上面呢？这是因为VSCode把菜单划分了这么几个区。

![](https://code.visualstudio.com/assets/api/references/contribution-points/groupSorting.png)

它们遵循如下规则：
* `navigation`组在任何情况下都在第一个位置
* `1_modification`组紧跟其后，并且这个组通常是用来修改代码的命令
* `9_cutcopypast`组在倒数第二个，并且里面应该是一些基本的编辑命令
* `z_commands`组是最后一个，用于打开命令面板

这是官方给出的位置和用途，但是从它们的命名方式可以看出，除了`navigation`组，其他都是根据字母顺序来排序的，比如你写了一个自己的组叫`2_test`，那么他就会在`1_modification`和`9_cutcopypast`之间。

不过我不推荐你自定义组，因为VSCode为所有可能出现的操作都准备了组名，具体参考[官方文档](https://code.visualstudio.com/api/references/contribution-points#contributes.menus)。


组内的命令通过`groupName@position`来排序，比如我的命令想出现在`navigation`组的第一个，那么就写`navigation@1`即可。

### 更多显示位置
menus中除了可以设置`editor/context`外，还可以设置其他的位置，如：


* commandPalette  
    在命令面板中显示

* explorer/context  
    在资源管理器\(文件管理侧栏\)的右键菜单中显示

* editor/context  
    在编辑器的右键菜单中显示

* editor/title  
    在编辑器标题栏的右上角显示  
    ![img](http://nsimg.cn-bj.ufileos.com/img-1559544939759.png)  
    即这个位置


* editor/title/context  
    编辑器标题栏右击某个文件时弹出的菜单  
    ![img](http://nsimg.cn-bj.ufileos.com/img-1559545001348.png)  
    即这个位置



* debug/callstack/context  
    debug侧栏的callstack右击弹出的菜单

**PS:以下没经过测试，都是直接翻译的官方文档**
* view/title  
    视图标题菜单

- view/item/context  
    视图选项菜单 
  
- debug/toolbar  
    debug工具栏 

- scm/title  
    SCM标题菜单栏 

- scm/resourceGroup/context  
    SCM资源组菜单栏 

- scm/resource/context  
    SCM资源菜单栏 

- scm/change/title  
    SCM改变标题菜单 

- touchBar  
    MacOS的TouchBar 

扩展我们的代码，我们希望我们的命令在`editor/title`上也能显示，就是编辑窗口标题栏右上角的一堆按钮那里，于是在`menus`中添加如下代码：

```json
"editor/title": [
    {
        "command": "ext.getCurrentPath",
        "group": "navigation"
    }
]
```

运行后右上角确实多了我们的命令，而且按下它也确实好用，不过人家的都是一个图标，我们的是命令的标题。

![img](http://nsimg.cn-bj.ufileos.com/img-1559545317915.png)

这已经丑到无法容忍的地步，我们给它添加一个图标吧，在`commands`节点中添加图标，如下：
```json
"commands": [{
    "command": "ext.getCurrentPath",
    "title": "Get Current Path",
    "icon": {
        "dark": "res/dark/copy_file.svg",
        "light": "res/light/copy_file.svg"
    }
}]
```

注意，设置的时候分dark和light两个，这两个是基于编辑器来说的，也就是说编辑器主题是亮色系，light中的图标才会被启用，但是亮色系的主题的按钮一般都是暗色的，所以light中的图片一般都是偏暗的，dark中的才是偏亮的。

![img](http://nsimg.cn-bj.ufileos.com/img-1559546263196.png)

瞬间好看多了。

## 命令快捷键
假设你又用改进版的插件工作了一周，你觉得用鼠标操作菜单还是太累，这时你想，如果有个快捷键就好了。

VSCode正好提供了快捷键和命令的绑定，您可以在`package.json`的`contributes.keybindings`节点中设置它。

```json
"keybindings":[
    {
        "command": "ext.getCurrentPath",
        "key": "ctrl+shift+alt+g"
    }
]
```

额这个键位是不太好按哈。。。没事，小问题。

这下你很高兴，你编写的这个插件一直陪着你工作，直到有一天，你换了一台mac book。

你下载了一个VSCode，正兴冲冲的要开始一天的工作，结果发现，Mac Book上没有Ctrl键，那该怎么办呢？

VSCode想到了这个问题，所以提供了各个平台下的快捷键，你可以用`windows`、`mac`和`linux`三个节点指定快捷键。

```json
"keybindings":[
    {
        "command": "ext.getCurrentPath",
        "windows": "ctrl+shift+alt+g",
        "linux": "ctrl+shift+alt+g",
        "mac": "cmd+shift+alt+g"
    }
]
```

## when
这下你的插件终于可以兼容全平台了，但是还有一个小小的问题，当你没有打开任何一个文件时按下快捷键，它仍然可以执行，并会给出一个错误提示，告诉你让你先打开一个文件。

![img](http://nsimg.cn-bj.ufileos.com/img-1559546759107.png)

这个提示应该是VSCode内建命令`copyFilePath`弹出的，它看起来一点都不像一个错误提示，而是一个普通通知。

并且，你在没打开文件的时候，命令面板里也能搜索到它，你不希望这样，你希望没有打开文件的时候，你的命令压根就不运行，你需要改进你的插件。

使用`when`可以指定插件在什么情况下运行。

在`keybindings`的命令节点中添加如下代码保证没有打开文件时快捷键不起作用。
```json
"when": "editorIsOpen"
```

并且在`contributes.menus`中添加如下代码保证没有打开文件时在命令面板中不会出现。
```json
"commandPalette": [
    {
        "command": "ext.getCurrentPath",
        "when": "editorIsOpen"
    }
]
```

这样我们就能保证我们的插件只在有打开的文件时好用了。

关于when中的条件，我翻译了官方给出的表格中的部分：

名字 | 为真条件
-|-
editorFocus|一个编辑器有焦点，text和widget都可以
editorTextFocus|一个编辑器中的文本有焦点(光标闪烁)
editorHasSelection|编辑器中有文本被选中
editorHasMutipleSelections|编辑器中有多个文本区域被选中
editorReadOnly|编辑器是只读的
editorLangId|当前编辑器的语言id匹配时，如`editorLangId == typescript`
isLinux丨isMac丨isWindows|在Liunx、Mac或Windows系统下
inDebugMode|一个Debug会话正在运行
debugType|当debug类型匹配时，如：`debugType == 'node'`
inSnippetMode|编辑器在snippet模式中


更多的请参考[官方文档](https://code.visualstudio.com/docs/getstarted/keybindings#_when-clause-contexts)

## 不用内建命令的版本
自己之前立的flag，当然含着泪也要实现 QAQ~

```typescript
context.subscriptions.push(
    vscode.commands.registerCommand('ext.getCurrentPathWithoutBuiltInCommand',()=>{
        const currentEditor = vscode.window.activeTextEditor;
        if(currentEditor){
            const path = currentEditor.document.uri.path;
            vscode.env.clipboard.writeText(path);
            currentEditor.insertSnippet(new vscode.SnippetString(path));
            vscode.window.showInformationMessage('Done...');
        }
    })
);
```
其实用VSCode提供的API也没多少行代码，上面用到的API在下一篇文章中会一一介绍。

## registerCommand的参数
其实上面的代码我们还可以改写成这样的：
```typescript
context.subscriptions.push(
    vscode.commands.registerCommand('ext.getCurrentPathWithoutBuiltInCommand',(path)=>{
        vscode.env.clipboard.writeText(path);
        currentEditor.insertSnippet(new vscode.SnippetString(path));
        vscode.window.showInformationMessage('Done...');
    })
);
```
直接给registerCommand方法一个参数，这样写的话如果是在右键菜单中或者资源管理器菜单中执行命令会自动传入当前文件的路径，而在命令面板中执行则不会传入，也就是path是undefined。

## 参考
* [extension-guides/command](https://code.visualstudio.com/api/extension-guides/command)
* [contributes.menus](https://code.visualstudio.com/api/references/contribution-points#contributes.menus)
* [when-clause-contexts](https://code.visualstudio.com/docs/getstarted/keybindings#_when-clause-contexts)
* [Sorting-of-groups](https://code.visualstudio.com/api/references/contribution-points#Sorting-of-groups)
* [TextEditor](https://code.visualstudio.com/api/references/vscode-api#TextEditor)

## 图标引用
来自阿里图标库[Marlon](https://www.iconfont.cn/user/detail?spm=a313x.7781069.1998910419.dcc7d6115&userViewType=collections&uid=46695)

不过它的主页好像已经找不到这个图片了，[这里](https://www.iconfont.cn/search/index?searchType=icon&q=copy%20file)还能找到。