---
title: JS创建对象的几种方法
date: 2019-05-21 15:58:27
tags: [js]
categories: js
---
### 对象字面量
```javascript
var Person = {
    name:"Pikachu",
    age:16,
    sayHello:function(){
        console.log("Hello,I'm "+this.name);
    }
}
```
如上创建了一个Person对象，他叫Pikachu，但是如果我还想再创建一个Person对象，就得重新写，没有一个用于创建对象的模板，熟悉OOP的应该已经意识到了，就是没有`类`这种东西\(事实上js确实没有\)。这样会产生很多重复代码。

### 工厂模式
如果你对设计模式有所了解，工厂模式你肯定不陌生。可以使用工厂模式创建一个Object的实例并且扩展它的属性后返回。
```javascript
function createPerson(name,age){
    var obj = new Object();
    o.name = name;
    o.age = age;
    o.sayHello = function(){
        console.log("Hello,I'm "+this.name);
    }
}

var person1 = createPerson('Pikachu',12);
var person2 = createPerson('Peppa Pig',5);
```
这样你就有了一个用于创建对象的模板，可以直接调用该工厂方法创建对象。

不过工厂模式有问题，就是创建出的对象没有明确的类型，就是Object类型的。而且对于`sayHello`这种方法，每个对象都可以共用，但是使用工厂模式每次创建一个对象就会生成一个新的`sayHello`。

### 构造函数模式
刚刚工厂模式中用到了这样一行：`var obj = new Object();`，这就是用到了Object类的构造函数来创建Object对象。

如何定义一个构造函数？
```javascript
function Person(name,age){
    this.name = name;
    this.age = age;
    this.sayName = function(){
        console.log("Hello,I'm "+this.name);
    }
}

var person1 = new Person('Pikachu',12);
var person2 = new Person('Peppa Pig',5);
```

这样就可以创建对象了，并且创建出的对象的类型明确指定为`Person`。当你使用new关键字时，Person函数会被当做构造函数使用，并且返回一个创建好的对象。而当你不用new时，它就会被当做普通函数使用，这时其中的this指向引用时的作用域。

不过这个`sayName`方法复用的问题好像还没解决。

不过我们知道，函数也是一个对象，我们从全局创建一个函数，然后放到Person中，不就实现了复用吗？

```javascript
function sayName(){
    console.log("Hello,I'm "+this.name);
}

function Person(name,age){
    this.name = name;
    this.age = age;
    this.sayName = sayName;
}
```
不过问题是解决了，但是在全局中声明这个函数好像有点怪怪的，而且可读性也不高，很容易让人不理解。

### 原型模式
原型模式解决了这个问题，所为原型，你可以看做一个对象的模板，我们每创建一个函数都有`prototype`属性，它指向一个对象，当我们用这个函数初始化对象的时候，这个对象中的`[[Prototype]]`属性指向这个函数的`prototype`指向的对象。这时就做到了模板的功能。

PS:这不是纯原型模式，而是原型和构造函数混合使用，纯原型模式有问题，基本不会用，所以不说。

```javascript
function Person(name,age){}
Person.prototype.sayName = function(){
    console.log("Hello,I'm "+this.name);
}

var person1 = new Person('Pikachu',12);
var person2 = new Person('Peppa Pig',13);

console.assert(person1.sayName == person2.sayName);
```
此时person1和person2共用一个sayName。

有一点要说明的是，javascript访问一个对象的属性或者是方法时会经过如下步骤(忽略继承)：  
1. 在自身的属性中找
2. 如果步骤一没找到，在`[[Prototype]]`指向的对象中找

所以如上代码中的断言成立不是因为person1和person2的sayName属性相同(虽然可以看做相同)，它们根本没有这两个属性，它俩访问到的sayName都是prototype中定义的。

所以，当执行如下代码时：
```javascript
function Person(name,age){
    this.name = name;
    this.age = age;
}
Person.prototype.sayName = function(){
    console.log("Hello,I'm "+this.name);
}

var person1 = new Person('Pikachu',12);
var person2 = new Person('Peppa Pig',13);

person1.sayName = function(){
    console.log('PikaPika~~');

person1.sayName();//PikaPika~~
person2.sayName();//Hello,I'm Peppa Pig

console.assert(person1.sayName == person2.sayName);//失败的断言
}
```
修改person1的sayName时并不会影响到person2，而是在person1中加入了一个新的`sayName`属性，而不用原型中的了。而person2中依旧没有sayName，所以还会使用原型中的。

考虑一个问题，我们既然用了原型，就是因为希望多个对象共享属性模板，我们肯定不希望模板中的内容被修改吧。目前来看原型无论如何都不会被修改，不过如果原型中存了一个引用类型可就不一定了。

比如说：
```javascript
function Person(name,age){
    this.name = name;
    this.age = age;
}

Person.prototype.friends = ['Robot']
```
给原型中加入一个朋友属性，是一个数组，数组是引用类型，我们虽然无法通过对象对原型中的默认属性进行修改，但是对于引用类型，只要它提供了方法，我们仍然可以直接修改它。

比如：
```javascript
var person1 = new Person('Pikachu',12);
person1.friends.push('Robot2');
var person2 = new Person('Peppa Pig',13);
person2.friends; //Robot,Robot2
```
我们调用了person1.friends的push方法，这时我们并没有对friends属性用`=`号直接赋值，而是调用了push方法添加了一个值，这时就会对其他对象造成影响。所以对于引用类型尽量不要使用原型。(这也就是开头说的不用纯原型模式的原因)。

### 直接重写原型
每次添加一个属性都需要写冗长的代码，其实我们还可以重新重写原型。
```javascript
function Person(name,age){
    this.name = name;
    this.age = age;
}

Person.prototype = {
    xxx:xxx,
    xxx:xxx,
    xxx:xxx
}
```
不过我们思考下，这么做是直接把原型给替换了，那么原型中本来存在的东西就会消失。每个原型中其实本身有一个`constructor`属性，指向对应的方法。比如`Person.prototype.constructor`指向`Person`。使用这个就可以确定类型了，`instanceof`关键字就是根据这个来判定的。所以当你直接重新写了这个原型的话，那么就无法通过`instanceof`去判断它的类型了。

比如：
```javascript
function Person(name,age){
    this.name = name;
    this.age = age;
}

var p1 = new Person('Pikachu',15);
p1 instanceof Person //true
p1 instanceof Object //true

Person.prototype = {
    xxx:xxx,
    xxx:xxx,
    xxx:xxx
}

p1 instanceof Person //false
p1 instanceof Object //true
```
