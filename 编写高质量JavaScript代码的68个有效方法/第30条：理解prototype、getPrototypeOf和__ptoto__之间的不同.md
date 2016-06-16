---
title: 第30条：理解prototype、getPrototypeOf和__ptoto__之间的不同
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---
原型包括三个独立但相关的访问器。这三个单词都是对单词prototype做了一些变化。
- C.prototype用于建立由new C()创建的对象的原型
- Object.getPrototypeOf(obj)是ES5中用来获取obj对象的原型对象的标准方法
- obj.__proto__是获取obj对象的原型对象的非标准方法
## 一个例子
要理解这些访问器，我们拿一个典型的js数据类型作例子。
假设User构造函数需要通过new操作符来调用。它需要两个参数，即姓名和密码的哈希值，并将它们存储在创建的对象中。
代码如下：
```js
function User(name,pwd){
	this.name=name;
	this.pwd=pwd;
}
User.prototype.toString=function(){
	return '[User '+this.name+']';
}
User.prototype.checkPwd=function(pwd){
    return hash(pwd)===this.pwd;
}
var u=new User('cedrusweng','$sdf99kaslf7');
```
根据上面的这个代码我们可以画出下面这个图，以表示各种关系。
我们每次在使用
```js
function Fn(){
    
}
```
相当于调用下面这样
```js
var fn=new Function("函数体"[,参数]);
```
其中Function是一个构造函数，它的原型对象中包含我们之前讲到过的call、apply、bind等属性和方法。fn是Function的一个实例对象。
所以这里的可以得到第一部分的图

下面再就构造函数User和它的原型对象之间的关系画出一个图


User函数带有一个默认的prototype的属性，其包含一个开始几乎为空的对象。上面的例子中添加了两个方法到原型对象中。当使用new操作符创建User的实例时，产生的对象u得到了自动分配的原型对象，该原型对象被存储在User.prototype中。

最后来一张完整的图


注意：new操作符调用构造函数，会产生一个新的对象实例，这个对象是以构造函数为模板，创建一份私有的性属性和方法，所有的实例都会继承原型对象。当访问u.name时，u对象会首先在它的私有属性中进行搜索，如果有则会返回，如果没有，则会查找对象的原型对象。当访问u.checkPwd时，私有属性和方法不存在时，会返回存储在User.prototype中的方法。

## 原型有关的方法
首先，构造函数的prototype属性用来设置新实例的原型关系。
其次，ES5中的函数Object.getPrototypeOf()可以用于检索现有对象的原型。
如上面的例子，可以使用下面代码来检测对象u的原型对象。
```js
Object.getPrototypeOf(u)===User.prototype;//true
```
最后，一些环境提供了非标准的方法检索对象的原型，即特殊的__proto__属性。这可作为在不支持ES5的Object.getPrototypeOf方法的环境中的一个兼容方法。在这些环境中可以使用下面代码完成检测
```js
u.__proto__===User.prototype;//true
```
下面画一张图，表明它们之间的关系



## 最后的说明
js程序员往往将User描述为一个类，尽管它跟一个函数差不多。js中的类本质上是一个构造函数与一个用于类（User.prototype）实例间共享方法的原型对象的结合。
下面是一个User类的概念图。




User函数给该类提供了一个公共的构造函数，而User.prototype是实例之间共享方法的一个内部实现。User和u的变通用法都不需要直接访问原型对象。

## 提示
- C.prototype属性是new C()创建的对象的原型
- Object.getPrototypeOf(obj)是ES5中检索对象原型的标准函数
- obj.__proto__是检索对象原型的非标准方法
- 类是由一个构造函数和一个关联的原型组成的一种设计模式

function createFn(a)
    var fn;
    if(a){
       fn=function(){console.log(1)}
    }else{
       fn=function(){console.log(2)}
    }
    return fn;















