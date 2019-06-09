---
title: VSCode 扩展开发(四) Output、自动完成和Snippets
date: 2019-06-04 07:47:54
tags: [vscode,editor]
categories: VSCode
---

## 学习目标
经过本章，你会了解到如何使用VSCode API操作OutputChannel，以及如何构建一个类似Emmet的智能代码生成器与一些Snippets方面的内容。

## 开篇寒暄
我们平常使用编辑器，最常用到的两个东西就是控制台和编辑器，开发插件的时候我们也有大部分需求是要操作它们的。所以了解如何操作它们是非常重要的一件事。

并且VSCode面向插件提供了非常多的好用的[API](https://code.visualstudio.com/api/references/vscode-api)，这些API几乎囊括了VSCode的所有功能，我们可以使用它们开发我们的插件，对VSCode进行任意功能的扩展。

## OutputChannel
Debug的时候，如果逻辑不是很复杂，不是很难发现Bug，我们基本上就会直接采用控制台输出的方式找问题，而不会去采用BreakPoint。  
除了调试信息，一些日志信息也会被我们输出到控制台，方便我们观察程序的运行轨迹，所以控制台可是我们的好伙伴哦。如果你说你没用过控制台，那真的是没人会信。

试想，如果我们再为一门语言设计VSCode插件，让它能脱离终端，直接在VSCode中运行，那么我们就要把每次执行程序所有输出的信息都放到VSCode的Output面板中。

那它究竟怎么用呢？我们来新建一个项目看看，地址：[这里](https://github.com/YHaoNan/vscode-tutorial/tree/master/vsc-extensions-tutorial-4)

### 创建OutputChannel、添加信息
**注意：以后关于`package.json`中的声明Command，我基本上会省略，除非有新东西需要配置。各位自行去声明。**  

我们可以通过VSCode的`vscode.window.createOutputChannel`去创建一个输出面板。如下：
```typescript
context.subscriptions.push(
    vscode.commands.registerCommand('extension.runBrainfuck',()=>{
        const output = vscode.window.createOutputChannel('BrainFuck Interpreter');
    })
);
```
我们向方法中传递一个字符串，这是输出面板的名称，然后我们就可以引用`output`对象对面板进行操作了。

VSCode的OutputChannel提供了如下API：

##### 属性
* name: string
##### 方法
* append(value: string): void  
    向输出面板中添加字符串
* appendLine(value: string): void  
    向输出面板中添加字符串并自动换行
* clear(): void  
    清空输出面板中的内容
* dispose(): void  
    处理和释放相关资源
* hide(): void  
    隐藏面板
* show(preserveFocus?:boolean): void  
    显示面板  
    当preserveFocus为true的时候不获取焦点
* ~~show(column?:ViewColumn,preserveFocus?:boolean): void~~  
    方法已经过时

我们尝试着往输出面板中输出几行信息。
```typescript
context.subscriptions.push(
    vscode.commands.registerCommand('extension.runBrainfuck',()=>{
        const output = vscode.window.createOutputChannel('BrainFuck Interpreter');
        output.show();
        output.append('Helo ');
        output.appendLine('vsc~');
        output.appendLine('Helo vsc~');
        output.append('Helo Helo~')
    })
);
```
得出如下结果：
```
Helo vsc~
Helo vsc~
Helo Helo~
```
可以说明，append确实是不换行的，而appendLine只是在末尾换行，如果前面用的是append，并且没有换行符的话，appendLine并不会另起一行。

光是这么学不好玩，我们来设计一门语言的VSCode插件，并使用Output面板输出运行结果。什么？你问我设计什么语言的？不知道你听没听过BrainFuck，如果没听过请走[这边](https://segmentfault.com/a/1190000000395225?utm_source=tuicool&utm_medium=referral)。


*PS:以下可能需要一些Brainfuck的知识和一定的阅读源码能力*

关于Brainfuck\(以后简称bf\)解释器，由于时间关系，我没有自己写，而是引用了一个开源的基于TypeScript的解释器`brnfck`，repo在[这里](https://github.com/tobiasholler/brnfck)。

我们分析一下这个解释器的运行原理，整个解释器源码不多：
```typescript
import { TextDecoder } from "util";
import { OutputChannel } from "vscode";
function compileBrainfuck(code: string): string {
    let compiledCode: string = "(function(r,w){var i=0;var t=new Uint8Array(30000);"
    let i: number
    let countChars = function (char: string) {
        let c = 0
        while (code.charAt(i) == char) {
            c++
            i++
        }
        i--
        return c
    }
    for (i = 0; i < code.length; i++) {
        switch (code.charAt(i)) {
            case "<":
                compiledCode += "i-=" + countChars("<") + ";"
                break;
            case ">":
                compiledCode += "i+=" + countChars(">") + ";"
                break;
            case "+":
                compiledCode += "t[i]+=" + countChars("+") + ";"
                break;
            case "-":
                compiledCode += "t[i]-=" + countChars("-") + ";"
                break;
            case ".":
                compiledCode += "w(t[i]);"
                break;
            case ",":
                compiledCode += "t[i]=r();"
                break;
            case "[":
                compiledCode += "while(t[i]!=0){"
                break;
            case "]":
                compiledCode += "}"
                break;
        }
    }
    return compiledCode + "return t;})"
}

export function compileBrainfuckToFunction(code: string): (readFunction: () => number, writeFunction: (byte: number) => void) => Uint8Array {
    return eval(compileBrainfuck(code))
}

//由于用不到该方法所以不用管它
// export function compileBrainfuckToStandalone(code: string): string {
//     return '#!/usr/bin/env node\n(function(){var a=0;' + compileBrainfuck(code) + '(function(){try{return process.argv[2].charCodeAt(a++);}catch(e){return 0;}},function(b){process.stdout.write(String.fromCharCode(b))});})();'
// }
```

我们可以看出这个哥们儿写的代码确实很有趣，它并没有采用大多数人的直接对bf源码进行解释执行的方法，而是利用脚本语言的特性，将bf翻译成了JS代码，并且提供读写方法供外界接入。

它提供了`compileBrainfuck`方法把bf代码转换成字符串类型的JavaScript代码的字符串，并且通过`compileBrainfuckToFunction`方法调用eval来执行它，执行它后会返回一个签名为`(readFunction: () => number, writeFunction: (byte: number) => void) => Uint8Array`的Function，它有两个参数，第一个是读取方法，也就是bf需要从控制台读取一个字符时会回调的方法，你需要返回该字符在字符表中的位置，第二个方法是写入方法，改方法是bf需要向控制台输出一个字符时的回调方法，输出的字符就是那个`byte: number`，是该字符在字符表中的位置。

仔细的看一下，如果我们有这样一段bf源码：
```
++++++ [ > ++++++++++ < - ] > +++++ .
```
则它最终会变成这样的js代码：
```javascript
//为了方便阅读我加上了换行，实际生成的方法是没有换行的。
(function(r,w){var i=0;var t=new Uint8Array(30000);
t[i]+=6;
while(t[i]!=0){
    i+=1;
    t[i]=10;
    i-=1;
    t[i]-=1;
}
t+=1;
t[i]+=5;
w(t[i]);
return t;})
```
现在请用你的人脑作为JS解释器，猜猜执行到最后`t[i]`是什么？其实最后i就是在第二个单元，也就是`t[1]`中循环6次，每次加10，最后再加5对吧，而且用`t[0]`作为循环条件，那么最后的`t[1]`一定是65。那么65对应的字符就是“A”，所以调用`w(t[i])`就应该在控制台输出A。


分析完了这个解释器，我们开始考虑插件的事，输出方法中我们正好可以直接调用OutputChannel的append方法进行输出，但是关于输入，因为OutputChannel不支持，我们先不考虑，让它一直返回0。于是我们创建了一个`coderunner.ts`，写入了如下代码：

```typescript

import * as vscode from 'vscode';
import * as bf from './brainfuck'
export function runCode(code: string,outputChannel: vscode.OutputChannel){
    bf.compileBrainfuckToFunction(code)(()=>{return 0},(byte:number)=>{
        outputChannel.append(String.fromCharCode(byte));
    });
}
```

然后在`extensions.ts`中这样调用它：
```typescript
import * as vscode from 'vscode';
import * as fs from 'fs';
import * as runner from './coderunner'

export function activate(context: vscode.ExtensionContext) {
	context.subscriptions.push(
		vscode.commands.registerCommand('extension.runBrainfuck',(path)=>{
			const outputChannel = vscode.window.createOutputChannel('BrainFuck Interpreter');
			outputChannel.show();
			try{
				const sourceCode = fs.readFileSync(path.path).toString();
				runner.runCode(sourceCode,outputChannel);
			}catch(e){
				outputChannel.appendLine('Error:'+e.message);
			}
		})
	);
}

export function deactivate() {}
```
我们决定使用昨天最后介绍的办法获得文件路径，就是给`registerCommand`方法一个参数那个，并直接使用nodejs的`fs`模块获取它的内容，并丢到`brainfuck-js`中处理。

运行，新建一个文件，写入如下Brainfuck代码：
```
++++++++++[>+++++++>++++++++++>+++>+<<<<-]>++.>+.+++++++..+++.>++.<<+++++++++++++++.>.+++.------.--------.>+.>.
```
然后执行命令，会输出`Hello World!`。

![img](http://nsimg.cn-bj.ufileos.com/img-1559694977826.png)

## 输入功能
上面的插件运行的一直不错，但是直到有一天你遇到了这样的bf代码：
```
> , < ++++ [ > -------- < - ] > .
```
这个代码要求输入一个字符\(我们假定该字符是一个小写英文字母\)，然后输出该字符的大写，这时候我们的bf解释器就不好用了，因为我们的bf解释器还没有输入功能呢！

所以，现在需要给我们的插件添加一个输入功能，但是OutputChannel是不可输入的，如果你尝试输入，它会告诉你不能在只读编辑器中输入。经过再三考虑，我们决定使用VSCode提供的`vscode.window.showInputBox`来完成这个功能。

```typescript
context.subscriptions.push(
    vscode.commands.registerCommand('extension.runBrainfuck',async (path)=>{
        await vscode.window.showInputBox({prompt: 'Input some character'}).then(input=>{
            //To make sure input is not undefined
            input = input?input:'';
            const outputChannel = vscode.window.createOutputChannel('BrainFuck Interpreter');
            outputChannel.show();
            try{
                const sourceCode = fs.readFileSync(path.path).toString();
                const output = runner.runCode(sourceCode,input,outputChannel);
            }catch(e){
                outputChannel.appendLine('Error:'+e.message);
            }
        });
    })
);
```

prompt是inputbox的提示信息，然后我们还需要修改`coderunner.runCode`的代码：
```typescript
export function runCode(code: string,input:string,outputChannel: vscode.OutputChannel){
    var counter = 0;
    bf.compileBrainfuckToFunction(code)(()=>{
        if(counter<input.length)
            return input.charCodeAt(counter++);
        return 0;
    },(byte:number)=>{
        outputChannel.append(String.fromCharCode(byte));
    });
}
```
因为bf中的输入和输出都是以一个字符为基准的，所以在`runCode`方法中提供一个counter用于每次取出输入中的一个字符。

现在试着使用我们的插件运行刚刚的bf代码，会提示你输入，我们输入a，会输出大写的A。

![img](http://nsimg.cn-bj.ufileos.com/img-1559784487604.png)

## 打造极致bf编码体验
你用这个插件开开心心的写了一周bf\(谁会这么干？？xD\)，但是由于bf代码的可读性实在不高，经常写写着就不知道自己写的是啥了，而且遇到连续几个相同的符号一个一个敲的时候还得在心里计数。基于以上问题，我们打算改造一下bf的编码体验。

首先我们打算加入自动完成功能，也就是输入一小段，弹出智能提示，选中后自动补齐一大段。用于解决bf中的连续符号排列问题。我们的目标是，假如我们在编辑器中输入：
```
30+
```
然后按下tab，就会自动生成30个`+`出来，如果你搞过前端开发，这和前端必备插件`emmet`很像。

这会很大程度减轻我们的工作量，而且我们不用在心里数各种bf运算符的个数了。比如这个代码：
```
++++++ [ > ++++++++++ < - ] > +++++ .
```

我们的插件要达到这样的输入效果：

![效果](https://s2.ax1x.com/2019/06/06/VacDcd.gif)

是不是很炫酷呢？

做到这个效果，需要用到`CompletionItemProvider`这个东西，我们需要在`extension.ts`中注册它：
```typescript
context.subscriptions.push(
    vscode.languages.registerCompletionItemProvider(
        {language:  'brainfuck', scheme: 'file'},
        new ac.BrainfuckCompletionItemProvider(),
        '+','-','[')
);
```
首先，因为我们针对语言开发插件，所以要指定一个语言，可以看到`registerCompletionItemProvider`的第一个参数就是语言和一个scheme头。但是我估计除了咱们，没人会设计brainfuck的插件，也没人会用VSCode写brainfuck，所以VSCode肯定不会认识brainfuck是啥。所以我们需要再去`contributes.languages`中注册个语言。

```json
"languages": [
    {
        "id": "brainfuck",
        "aliases": ["Brainfuck","BrainFuck","brainfuck","bf"],
        "extensions": [
            ".bf"
        ]
    }
]
```
`id`即该语言的唯一标识，`aliases`是该语言的别名，在需要输入的地方输入别名时依旧能找到该语言，`extensions`是扩展名，我们写`.bf`。

然后回到刚刚`extensions.ts`那里。
```typescript
context.subscriptions.push(
    vscode.languages.registerCompletionItemProvider(
        {language:  'brainfuck', scheme: 'file'},
        new ac.BrainfuckCompletionItemProvider(),
        '+','-','[')
);
```
第二个参数是我们自己定义的一个`CompletionItemProvider`，这是一个用于进行代码提示和补全的Provider，VSCode的API中有很多这样的Provider，还有的用于跳转定义、弹出悬浮提示等，这类Provider一般都是要实现一个Provider子类，然后并不用去`package.json`中去声明什么的。

第三个参数是一个可变长参数，是触发智能提示的关键字，只要在编辑器中一输入这个关键字，我们的`CompletionItemProvider`的`provideCompletionItems`方法就会被触发，VSCode需要我们在这个方法中返回一个或一些智能提示选项，然后它去展示给用户。我们定义了三个关键字分别是`+`、`-`和`[`。

然后我们来看一下`provideCompletionItems`方法长啥样：
```typescript
provideCompletionItems(document: vscode.TextDocument, position: vscode.Position, token: vscode.CancellationToken, context: vscode.CompletionContext)

返回值
vscode.ProviderResult<vscode.CompletionItem[] | vscode.CompletionList>
```

它有四个参数，我们这里只关心第一个和第二个就好了，其他的您可以去官方文档中查看。

第一个参数是方法被触发时当前编辑器的文档对象，我们主要用它来分析一些带有语义的东西，比如我们的`30+`，要分析出生成多少个加号。

第二个参数则是当前输入关键字的Position对象，你可以获取到当前的行和当前的位置，主要和第一个参数配合使用。

注意，因为我们知道VSCode的插件一开始不会被加载，得发生一定事件时被加载，比如`OnCommand`。现在我们需要当打开一个bf文件时加载插件，就需要在`activationEvents`中加入这个。
```json
"onLanguage:brainfuck"
```

## 思路及实现
先说加和减运算符，因为这两个是最复杂的，我们把最复杂的解决了，简单的也就迎刃而解了。

我们知道，经过我们在`extension.ts`中的注册，现在只要在编辑器中一输入`+`和`-`号的时候就会触发`provideCompletionItems`方法，所以我们只要从这里面找到前面的若干位数字，然后计算出生成的加号和减号的个数就好了。

我们希望以单词为基本单位处理用户的输入，然后产生代码提示。

从关键字，也就是本次输入的加号和减号的位置开始，往前寻找，一旦找到空格或者到了行首，我们就把那个位置记做起始位置，然后从一行中截取起始位置到关键字位置的字符串，作为当前的单词，比如：
```
abcde 56+
```
输入加号后，当前单词就是`56+`。

但是，用户输入的单词并不一定是规则的，比如：
```
abcde abs56+
```
这种情况下，我们就不返回任何代码提示。

除此之外，我发现你输入键盘上的`A-Z`或者其小写形式还有一些其他字符时，也会触发这个方法，再加上之前的不规则输入情况，为了避免错误提示，我们使用正则来判断当前单词是否符合规则，如果符合就生成并返回代码提示，否则就不做任何响应。

那我就直接放代码了

```typescript
import * as vscode from 'vscode';
import { CompletionItem } from 'vscode-debugadapter';
import { stringify } from 'querystring';

const SUPPORT_OPT = ['\\+','\\-'];



export class BrainfuckCompletionItemProvider implements vscode.CompletionItemProvider{
    provideCompletionItems(document: vscode.TextDocument, position: vscode.Position, token: vscode.CancellationToken, context: vscode.CompletionContext): vscode.ProviderResult<vscode.CompletionItem[] | vscode.CompletionList> {
        
        
        var completion: vscode.CompletionItem | undefined;
        //当前行文本
        const currentLineText = document.lineAt(position.line).text;
        //当前单词第一个字符位置 position.character中的下标指向的实际上是关键字的下一个(不知道这么说准确不准确)
        const currentTokenFisrtCharIndex = findCurrentTokenFirstCharIndex(currentLineText,position.character-1);
        //当前单词
        const currentToken = currentLineText.substr(currentTokenFisrtCharIndex,position.character-currentTokenFisrtCharIndex);

        //针对所有支持的操作符(其实就+和-)构造正则表达式 并验证当前单词是否匹配
        SUPPORT_OPT.map(opt=>'^(\\d+)('+opt+'{1})$').forEach(regex=>{
            const matched = currentToken.match(regex);
            //如果匹配了就构造CompletionItem
            if(matched!=null) {
                //插入的文本 将关键字重复x次得出的结果，比如 5+ 则是 +++++
                const insertText = matched[2].repeat(parseInt(matched[1]));
                //创建CompletionItem 它所显示的标题为当前单词 比如 5+
                completion = new vscode.CompletionItem(currentToken);
                //这个是文档，我直接设置成了insertText
                completion.documentation = insertText; 
                //这个是显示出来的解释信息
                completion.detail = 'Insert '+matched[1] +' ' + matched[2];
                //这个是插入的文字，支持SnippetString
                completion.insertText = insertText;
                //替换范围 这里设成了从当前单词第第一个位置到当前单词的最后一个位置
                completion.range = new vscode.Range(position.line,currentTokenFisrtCharIndex,position.line,position.character);
                
            }
        });
        return completion?Promise.resolve(new vscode.CompletionList([completion], true)):null;
    }

    
}


function findCurrentTokenFirstCharIndex(text: string,position: number):number{
    while(position>0&&text.charAt(position)!=' ')
        position--;
    return text.charAt(position)==' '?position+1:position;
}
```

`provideCompletionItems`方法返回一个`ProviderResult`，所有的Provider中的provide方法都返回一个这东西，它实际上是一个Promise，所以我们需要调用`Promise.resolve`。

这样一来，加号和减号的智能提示就可以使用了，快去试试吧。

## 使用Snippet继续扩展
`CompletionItemProvider`确实好用，但是有些时候用不到，一般情况下，我们有动态的代码提示需求要处理的话才会使用它，就像`emmet`。一般的，固定格式的需求我们直接用snippet就好了，Snippet就是输入一小段生成一大段的意思，比如其他语言中的生成`for`语句的Snippet，只需要输入for就把整个的循环体弄出来了。

我们用snippet来扩展bf里的其他操作提示。


bf中经常会看到这样的代码：
```
+++ [ > ++++ < - ] 
```
这是bf构造一个循环常用的手段，因为`]`符号在当前单元不为0的情况下会跳转到前一个`[`，而且循环里一般要控制另一个单元的值，所以一般都会在循环开始之前用一个单元作为循环控制单元，然后在循环中用`>`跳到另一个单元进行累加操作，然后别忘了用`<`跳回去将循环控制单元的值迭代，上面的代码为循环3次，每次为下一个单元的值加4，执行完成后，第二个单元为12。

所以有必要写一个这样的snippet，生成这么个东西：
```
[ > < ]
```
我们在`package.json`中的`contributes`节点下添加snippets相关的配置：
```json
"snippets": [
    {
        "language": "brainfuck",
        "path": "./snippets.json"
    }
]
```
并且在`categories`中添加一个Snippets，否则snippet不会生效。
```json
"categories": [
    "Other","Snippets","Languages"
]
```
然后在项目根目录创建`snippets.json`，并填入如下内容：
```json
{
	"Bf_Jump_In_Unit_And_Loop":{
		"scope": "brainfuck",
		"prefix": "ul",
		"body": [
			"[ > $1 < $2 ]"
		],
		"description": "Create a unit jump body in loop"
	}
}
```
`scope`指定snippet生效的语言，`prifix`则是前缀，这里我们用`ul`\(Unit And Loop\)当前缀，`body`就是选中后插入的大段代码，是一个列表，一项就是插入后的一行，我们只有一行，所以就一项。

`$1`代表位置，当你插入这个snippet后光标会跳转到`$1`的位置，当你输入完成后，按下tab就会跳转到`$2`的位置，你当然还可以设置一些提示信息，就像这样：`${1:tips}`，该位置会插入文字`tips`并且处于选中状态，直接输入就会被替换掉。

`description`则是说明信息。

这样一来Snippet也弄好了，然后试着用我们刚刚写完的插件编写和编译bf代码吧！

未完待续...

## 参考
* [CompletionItemProvider](https://code.visualstudio.com/api/references/vscode-api#CompletionItemProvider)
* [Create Custom Language in Visual Studio Code](https://stackoverflow.com/questions/30687783/create-custom-language-in-visual-studio-code)
* [vscode-emmet opensource project](https://github.com/microsoft/vscode-emmet/)
* [snippet-guide](https://code.visualstudio.com/api/language-extensions/snippet-guide)