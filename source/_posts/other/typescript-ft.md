---
title: TypeScript的一些特性
date: 2019-06-08 06:41:21
tags: [typescript]
categories: Others
---
最近学习VSCode扩展开发，奈何TypeScript技术不够硬，又去总结了一些不熟悉的TypeScript的特性记录下来。

## 类型断言
假如你开了严格检查，对于这样的代码，编译器是不会通过的。
```typescript
class Person{
    constructor(public name: string){}
}


function getPerson(): Person | undefined{
    return new Person('王钢弹');
}

let p = getPerson();
console.log(p.name);
```
因为`getPerson`方法的返回值被声明是`Person`或`undefined`，尽管方法中它永远不可能返回undefined，但是编译器就是认为p的值可能是undefined，所以需要进行类型检查：
```typescript
let p = getPerson();
if (p)
    console.log(p.name);
```
但是这显然有些臃肿，我们可以通过类型推断来告诉编译器，虽然方法定义中它可能返回undefined，但是我已经清楚了我所得到的返回值不可能是undefined。

```typescript
let p = getPerson();
console.log((p as Person).name);
console.log((<Person>p).name);
```
## let
在以往的JavaScript中，我们都用`var`来定义变量，`let`在JavaScript中还是一个比较新的概念。但是TypeScript中推荐使用`let`代替`var`。

确实`var`的设计真的像坨屎一样，或者说整个JavaScript都是那样。学JavaScript后我感觉这是一门设计的乱糟糟的语言。

`var`的作用域很奇怪，你可以在包含它的函数，模块，命名空间或全局作用域内部任何位置被访问，但是不包含块作用域，比如：
```typescript
function test(): string{
    if(true){
        var str = 'Hello';
    }
    return str;
}

console.log(test());
```
上面的代码中的`test`方法竟然真的能返回str，而不是undefined。所以这样的代码就会出现问题：
```typescript
for(var i=0;i<10;i++){
    setTimeout(() => {
        console.log(i);
    }, i*10);
}
```
我们预期的结果是输出0~9，但是真正的执行结果是：
```
10
10
10
10
10
10
10
10
10
10
```
因为`var`不认块定义域所以这个`i`实际上是定义在全局定义域的，然后因为setTimeOut有一定的延迟时间，这个时间之后for已经执行完毕，所以每次引用i都是全局中的i，经过for循环后全局中的i一直是10。

为了避免发生不可预期的错误和避免污染命名空间，应该使用`let`替代`var`，`let`认识块作用域的。

```typescript
for(let i=0;i<10;i++){
    setTimeout(() => {
        console.log(i);
    }, i*10);
}
```
这次会输出预期的结果，并且`let`定义的变量不能重定义，并且`let`可以做到屏蔽：
```typescript
for(let i=1;i<=3;i++){
    for(let i=1;i<=3;i++){
        console.log(i);
    }
}
```
第一个循环用`i`作为循环变量，第二个也是，这在其它编程语言里不被允许，因为对于内层循环来说，定义域里已经存在一个`i`了，但是`let`关键字允许这样做，可以用内层的`i`屏蔽掉外层的`i`，但是现实中没人会写出如此语义不清晰的代码吧。

注意，`const`和`let`的作用域是相同的，但是`const`是常量。

## 展开操作
```typescript
let first = [1, 2];
let second = [3, 4];
let bothPlus = [0, ...first, ...second, 5];
//bothPlus = [0,1,2,3,4,5]
```
如果使用展开操作对对象进行克隆操作，只能克隆其中的属性，并不能克隆方法。

## 接口
```typescript
interface Person{
    name: string;
}

function printPersonInfo(person:Person){
    console.log(person.name);
}

printPersonInfo({name: 'Peppa Pig'});
// printPersonInfo({name: 'Peppa Pig',age: 16}) Wrong
```

TypeScript中的接口作为参数时并不一定像其他OOP语言一样，必须传入一个接口的实现类，而是只要传入的对象满足接口的规则就行。

对于非必须的参数，可以这样写：
```typescript
interface Person{
    name: string;
    age?: number;
}

function printPersonInfo(person:Person){
    console.log(person.name);
}

printPersonInfo({name: 'Peppa Pig'});
printPersonInfo({name: 'Peppa Pig',age: 16});
```

`readonly`可以指定对象只读。
```typescript
interface Person{
    name: string;
    age?: number;
    readonly idCard?: number;
}
```

如果传递的对象中有接口中未定义的属性时会抛出异常，可以使用类型断言去避开异常：
```typescript
printPersonInfo({name: 'Peppa Pig',job: 'Front-end Programmer'}); //Wrong
printPersonInfo({name: 'Peppa Pig',job: 'Front-end Programmer'} as Person); //Correct
```
也可以在定义接口时添加索引签名。
```typescript
interface Person{
    name: string;
    age?: number;
    readonly idCard?: number;
    [propName: string]: any;
}
```
然后它可以接受任何属性，不过我觉得，如果条件允许的情况下，尽量不要这样写。


