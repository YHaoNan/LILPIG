---
title: VSCode 扩展开发(二) 第一个插件
date: 2019-06-03 08:00:54
tags: [vscode,editor]
categories: VSCode
---

## 学习目标
本篇您将使用VSCode官方的代码生成工具生成您的第一个插件，并扩展它，且会对插件开发的流程有一个大致的了解。

## 环境安装
**PS:开发VSCode Extensions需要node环境和git，请自行安装nodejs和git。**

VSCode官方给开发者提供了扩展代码生成工具，可以使用npm安装它们。
```bash
npm install -g yo generator-code
```

## 您的第一个插件
刚刚我们安装了官方提供的代码生成工具，我们现在用它来生成我们的第一个VSCode插件。

在您想要生成项目的目录下运行`yo code`，之后他为了帮助您生成不同类型的VSCode Extension，会询问您一些问题。
```bash
➜  vscode-tutorial yo code

     _-----_     ╭──────────────────────────╮
    |       |    │   Welcome to the Visual  │
    |--(o)--|    │   Studio Code Extension  │
   `---------´   │        generator!        │
    ( _´U`_ )    ╰──────────────────────────╯
    /___A___\   /
     |  ~  |     
   __'.___.'__   
 ´   `  |° ´ Y ` 

? What type of extension do you want to create? (Use arrow keys)
❯ New Extension (TypeScript) 
  New Extension (JavaScript) 
  New Color Theme 
  New Language Support 
  New Code Snippets 
  New Keymap 
  New Extension Pack 
(Move up and down to reveal more choices)
```
可以看到，它在询问你想创建什么类型的扩展，并且提供了很多类型。这里我们选择`New Extension (TypeScript)`，如果您使用JavaScript开发请选择第二个。

然后它又开始询问你一些问题：
```
# 您的扩展叫什么名字 hello-world
? What's the name of your extension? hello-world
# 您扩展的id 唯一标识 hello-world
? What's the identifier of your extension? hello-world
# 您扩展的介绍 My first VSCode Extension.
? What's the description of your extension? My first VSCode Extension.
# 是否初始化一个git仓库 Yes
? Initialize a git repository? Yes
# 使用什么包管理器 npm
? Which package manager to use? npm
```

全部填写完毕后，它就会自动生成一个以你的扩展名为名字的文件夹，所有的代码都在这里。

我们使用VSCode打开它，这里推荐直接使用命令行打开：
```bash
code hello-world
```
因为如果您把这个插件目录添加到VSCode的工作区的话，某些情况下VSCode无法确定您要把哪一个目录当成插件运行，您也就无法测试您插件。而用命令只是打开一个文件夹，不存在这个问题。

## 目录结构
可以看到一个VSCode Extension的目录结构如下：

![img](http://nsimg.cn-bj.ufileos.com/img-1559521012403.png)

其实这么多文件，我们只需要关心两个。src中有一个`extension.ts`，这是我们编写插件的主文件，我们插件的代码就是在这里面编写。另一个就是外层的`package.json`，这是nodejs项目都有的一个文件，我们编写插件时有很多东西要在这里注册后才能使用。

其他的目录用的不多了，src下的test是测试目录，测试代码可以在这里编写，node_modules是我们项目中用到的依赖库的文件夹，和我们没太大关系。

## 运行
点击F5运行我们的插件\(我在我的Windows环境和Linux环境下都需要第二次点击F5才可以，第一次不会有任何反应\)。

然后你会看到弹出了一个新的VSCode窗口，在这里我们按`Ctrl+Shift+P`打开命令窗口，输入Hello World。

![img](http://nsimg.cn-bj.ufileos.com/img-1559521444192.png)

可以看到，有这么一个命令，这是VSCode的插件生成工具帮我们自动生成的。我们按回车执行它，会发现右下角弹出一个通知消息。

![img](http://nsimg.cn-bj.ufileos.com/img-1559521510184.png)

至此，您的第一个VSCode Extension就开发完成了，再见！

## WTF！！！
停，我还什么都没干呢啊！怎么的就开发完了啊？？

刚刚都是VSCode代码生成器帮我们自动生成的，我们来看看它都干了什么？我们先关注下`extension.ts`。
```typescript
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {

	console.log('Congratulations, your extension "hello-world" is now active!');
    //关注这一行和里面的匿名函数
	let disposable = vscode.commands.registerCommand('extension.helloWorld', () => {
		vscode.window.showInformationMessage('Hello World!');
	});
    //每注册一个命令都要调用这个方法把返回结果push进去
	context.subscriptions.push(disposable);
}

export function deactivate() {}
```

尽管现在我们不知道VSCode提供的这些花里胡哨的API都是干嘛的，但是根据语义，我们能分析出来它调用`vscode.commands.registerCommand`注册了一个命令，并且给了一个类似id的东西`extension.helloWorld`还有一个匿名函数，函数里调用`vscode.window.showInformationMessage`显示了一条信息，是`Hello World!`。

而`activate`和`deactivate`则是我们的扩展的两个生命周期函数，`activate`就是当我们的扩展被激活，而`deactivate`则是我们的扩展被关闭。

至此我们就知道了整个插件的大致的运行过程，就是注册命令之后传递一个匿名方法，然后命令被调用，匿名方法就会被回调。不妨我们照猫画虎，再注册一个。

```typescript
let disposable = vscode.commands.registerCommand('extension.helloWorld', () => {
    vscode.window.showInformationMessage('Hello World!');
});

let disposable2 = vscode.commands.registerCommand('extension.shadiao', ()=>{
    vscode.window.showInformationMessage('无论我变成什么亚子，都鱼女无瓜！');
});
context.subscriptions.push(disposable);
//虽然我暂时不知道这是干嘛的但是 模仿上一行
context.subscriptions.push(disposable2);
```

可以看到我们完全照猫画虎注册了一个`extension.shadiao`，我们运行一下看看。

现在你可能已经打开了测试窗口，并且按下了`Ctrl+Shift+P`打开了命令窗口，但是你现在愣住了，你不知道干什么了。

刚刚生成器帮我注册那个，我是输入了Hello World就找到了对应的命令，但是我自己注册这个我应该输入什么呢？？

于是你回去看了看`extension.ts`的代码，发现确实没有`Hello World`的痕迹，虽然`registerCommand`方法有一个`extension.helloWorld`参数，但这个显然不是。

诶？？这个`extension.helloWorld`是干嘛的？我刚刚说过是命令的id对吧，那注册这个id有什么用呢？？我们去另一个比较有用的文件`package.json`中看看。

你看到了这样一段代码：
```json
"contributes": {
    "commands": [{
        "command": "extension.helloWorld",
        "title": "Hello World"
    }]
}
```

分析分析，大概知道怎么回事了，`contributes.commands`节点是一个放置所有注册的命令的列表，你注册过的所有命令必须在这里声明。

这里面自带这个应该是生成器帮我们生成的，使用`command`指定命令的id，使用`title`指定命令的标题，这下你应该看出来了，命令面板中就是使用命令标题来寻找一个命令的。所以我们把我们自己注册那个也声明一下。

```json
"commands": [{
    "command": "extension.helloWorld",
    "title": "Hello World"
},{
    "command": "extension.shadiao",
    "title": "你为什么总是带着面罩啊？"
}]
```

好，我们预测一下现在运行会是什么效果，我们在命令面板里输入：`你为什么总是带着面罩啊？`，然后下面就会提示：`无论我变成什么亚子，都鱼女无瓜！`\(Ohhh这是个梗，估计你们这帮搞技术的秃瓢是不会理解的...Ohh对不起对不起xD\)。

好，我们运行它，然后发现，事情好像并不是我们想象的那样，当我们执行这个命令时，下面弹出了一条错误提示：

![img](http://nsimg.cn-bj.ufileos.com/img-1559522859156.png)

什么？命令没找到？VSCode瞎了吗？我明明package.json里也写了，extension.ts里也写了，你怎么还找不到？你是个成熟的编辑器了，应该学会自己找到我的命令了。

去查了官方文档，原来是因为VSCode考虑到性能问题使用了类似懒加载的模式，并不是一打开就加载所有的扩展，所以当我们从命令窗口选择我们的命令时，我们的扩展还没被激活，所以将命令和处理函数绑定的方法`registerCommand`还没有被实际的调用，所以就会出现这个问题。

那我们的扩展什么时候被激活呢？VSCode提供了一个`activationEvents`选项去设置它，这个选项包含了一个列表，通过这个列表指定我们的扩展何时被激活。

在`package.json`中找到`activationEvents`这个节点，它肯定存在，是代码生成器帮我们生成的，目前应该是这样的：
```json
"activationEvents": [
    "onCommand:extension.helloWorld"
],
```
可以看到目前这个列表里有一个项目，从语义上分析，我们知道这是当命令`extension.helloWorld`被调用时，激活扩展。

看到这，我想你一定知道了应该把我们刚刚注册那个命令也添加进这个列表，让我们这个命令被调用时，扩展也能激活。确实应该是这样，但是我们不妨想一想，如果我们现在不添加我们的命令到激活列表，而是直接运行后先调用`extension.helloWorld`之后，插件被激活了，再调用我们那个命令应该就不会出现命令未找到这样的提示了。我们试试。

![img](http://nsimg.cn-bj.ufileos.com/img-1559538458860.png)

确实，两个命令都正确的执行了。但是如果在生产环境中，我们不知道用户会先执行哪个命令，所以我们应该把所有的命令都加进去\(除非特殊需求\)。

所以，修改`package.json`的`activationEvents`节点：
```json
"activationEvents": [
    "onCommand:extension.helloWorld",
    "onCommand:extension.shadiao"
],
```

这样一来，先执行我们定义的命令也可以了。

当然，`activationEvents`还有别的激活条件，不单单是当命令被调用时，这些就是后话了。

至此，我们的第一个插件和它的执行流程就理清了，收拾收拾，看看下一篇吧！

## 参考文档
* [get-started/your-first-extension](https://code.visualstudio.com/api/get-started/your-first-extension)
* [extension-guides/command](https://code.visualstudio.com/api/extension-guides/command)