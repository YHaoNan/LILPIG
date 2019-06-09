---
title: VSCode 扩展开发(六) 用户设置
date: 2019-06-08 10:46:54
tags: [vscode,editor]
categories: VSCode
---

## 学习目标
经过本节，你将了解如何给你的扩展添加用户设置选项。

## 开篇寒暄
一个优秀的插件需要给用户很多可配置的选项，让用户决定使用哪些功能，而不是把所有功能都给写死在代码中，VSCode当然提供了这个功能，我们看看怎么使用吧。

## 开始
关于用户设置的配置，我们全部都在`package.json`中的`contributes.configuration`节点里配置，并且我们只需要在TypeScript代码中通过`vscode.workspace.getConfiguration`引用即可。

我们现在开始尝试给之前的bf解释器加一些可选项，复制上一章的代码，本章代码在[这里](https://github.com/YHaoNan/vscode-tutorial/tree/master/vsc-extensions-tutorial-6)

我决定加入三个选项，一个是是否开启运算符计数功能，第二个是是否开启自动完成功能，第三个是是否开启循环跳转功能。
```json
"configuration":{
    "title": "Brainfuck Plugin User Settings",
    "properties": {
        "bf.openOptCounter":{
            "type": "boolean",
            "default": true,
            "description": "Whether open the operator counter"
        },
        "bf.openAutoComplete":{
            "type": "boolean",
            "default": true,
            "description": "Whether open the auto compelete"
        },
        "bf.openLoopJump": {
            "type": "boolean",
            "default": true,
            "description": "Whether open the loop jump"
        }
    }
}
```
如上代码配置了我们需要的功能，`properties`中包含的的每一个子节点是一个选项，其id就是每一个节点的节点名，如`bf.openOptCounter`。

我们运行插件，打开设置就能看到我们提供的设置项了。

![img](http://nsimg.cn-bj.ufileos.com/img-1559963277078.png)

当然，也可以在json中配置。

![img](http://nsimg.cn-bj.ufileos.com/img-1559963393773.png)

然后我们就可以在代码中判断选项是否开启，然后根据用户的设置来决定功能是否开启。

hover.ts
```typescript
import * as vscode from 'vscode'
import { disconnect } from 'cluster';
import * as util from './util';


export class BrainfuckHoverProvider implements vscode.HoverProvider{
    provideHover(document: vscode.TextDocument, position: vscode.Position, token: vscode.CancellationToken): vscode.ProviderResult<vscode.Hover> {
        const opt = document.lineAt(position.line).text.charAt(position.character);
        const wordRange = document.getWordRangeAtPosition(position,new RegExp('\\'+opt+'+'));
        //判断选项是否开启
        if((opt=='+' || opt=='-')&&vscode.workspace.getConfiguration().get<boolean>('bf.openOptCounter'))
            return wordRange?new vscode.Hover(new vscode.MarkdownString('### BF Opt Counter  \n\n * **Opt: `'+opt+'`**\n\n* **Length: `'+(wordRange.end.character-wordRange.start.character)+'`**')):null;
        else if((opt=='[' || opt==']')&&vscode.workspace.getConfiguration().get<boolean>('bf.openLoopJump')){
            const map = util.getLoopOptPairMap(document);
            const target = util.getMatchedOpt(map,position);
            if(target)
                return wordRange?new vscode.Hover(new vscode.MarkdownString('### Loop Opt Pair\n\n* **Matched: `'+target.character+'`**, press ctrl and click to jump here.')):null;
        }
    }

}

```
definition.ts
```typescript
import * as vscode from 'vscode';
import * as util from './util';


export class BrainfuckDefinitionProvider implements vscode.DefinitionProvider{
    provideDefinition(document: vscode.TextDocument, position: vscode.Position, token: vscode.CancellationToken): vscode.ProviderResult<vscode.Location | vscode.Location[] | vscode.LocationLink[]> {
        if(vscode.workspace.getConfiguration().get<boolean>('bf.openLoopJump')){
            const char = document.lineAt(position.line).text.charAt(position.character);
            console.log(char);
            if(char != '[' && char != ']')return null;
            const map = util.getLoopOptPairMap(document);
            const target = util.getMatchedOpt(map,position);
            if(target)
                return new vscode.Location(document.uri,target);
        }
    }
}
```

autocomplete.ts
```typescript
import * as vscode from 'vscode';
import { CompletionItem } from 'vscode-debugadapter';
import { stringify } from 'querystring';

const SUPPORT_OPT = ['\\+','\\-'];



export class BrainfuckCompletionItemProvider implements vscode.CompletionItemProvider{
    provideCompletionItems(document: vscode.TextDocument, position: vscode.Position, token: vscode.CancellationToken, context: vscode.CompletionContext): vscode.ProviderResult<vscode.CompletionItem[] | vscode.CompletionList> {
        
        if(!vscode.workspace.getConfiguration().get<boolean>('bf.openAutoComplete')) return;
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

用户设置还支持如下类型：`string`、`array`、`number`、`boolean`、`object`、`integer`、`null`。


## 参考
* [contributes.configuration](https://code.visualstudio.com/api/references/contribution-points#contributes.configuration)