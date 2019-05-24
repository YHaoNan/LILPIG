---
title: JS里的继承
date: 2019-05-21 15:58:27
tags: [js]
categories: js
---

JS中动态语言和prototype的特性让它可以用很多方法实现继承。

### 原型链
我们之前说过原型，可以理解为模板哈。那么想想OOP中的继承，其实不就是继承模板吗，于是大佬们想到了这样继承：
```javascript
//猴子
function Monkey(){}

Monkey.prototype.jump = function(){
    console.log('JUMP!!');
}
Monkey.prototype.run = function(){
    console.log('RUN!!');
}

//人
function Person(){}
//继承 实际上就是初始化一个Monkey对象作为自己的原型
Person.prototype = new Monkey();
Person.prototype.speak = function(){
    console.log('Blah Blah Blah...');
}

var person = new Person();
person.run();
person.speak();
```


当调用person.run时，会在person中找该属性，然后并没有找到，就去prototype中找，它的prototype是个Monkey对象，在Monkey中也没有直接的定义，于是就去Monkey对象的`[[ProtoType]]`中找，一层一层的，所以最后能找到run。

所以Object的`toString`在每个对象中都能访问也是这样的原理。

说道继承就得说重写，这重写应该不难了，就直接在person的Prototype中写或者在Person构造函数中直接定义。

不过有些问题，如果我在父类的函数中定义一些属性，就没法通过原型链的方式继承了。比如：
```javascript
function Monkey(height,weight){
    this.height = height;
    this.weight = weight;
}

...

//这里如果想继承父类的height和weight就很难了。构造函数没法传值，没法让用户创建对象时传入。
function Person(){}
Person.prototype = new Monkey();
...
```
这下就看出问题了。不过这里我们有一个解决办法，我们可以在子类构造器中调用父类的构造器。这下不用原则链就可以实现继承。
```javascript
function Monkey(height,weight){
    this.height = height;
    this.weight = weight;
}

...

function Person(height,weight){
    //调用父类构造函数初始化，传入this，做完这一步会把父类的构造函数中创建的所有属性继承过来
    Monkey.call(this,height,weight);
}
//继承父类的原型，做完这部会把父类的原型继承过来
Person.prototype = new Monkey();
...
```
一般情况下，我们为了保证引用类型不被修改的问题在构造器中初始化一些属性，为了提高公共方法复用性在原型中初始化一些方法。



### 原型式继承
原型式继承提供一个函数，基于一个传入的对象作为原型创建另一个对象，这和很多语言中的`clone`差不多。
```javascript
function object(o){
    function F(){}
    F.prototype = o;
    return new F();
}
```
同样要注意对于引用类型修改值的问题。

ECMAScript5中新增了`Object.create()`方法，其实和上面方法做的差不多。不过她有两个参数，第二个参数是一个对象，第二个对象中有的所有属性都会覆盖第一个参数。

如：
```javascript
var personCopy = Object.create(person,{
    name:'Bot'
});
```
personCopy拥有person的所有属性，但是name会被替换。
