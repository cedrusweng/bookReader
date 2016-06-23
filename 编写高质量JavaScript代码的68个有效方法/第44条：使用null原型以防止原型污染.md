---
title: 第44条：使用null原型以防止原型污染
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---
第43条中讲到的就算是用了Object的直接实例，也无法完全避免，Object.prototype对象修改，造成的原型污染。防止原型污染最简单的方式之一就是不使用原型。在ES5之前，并没有标准的方式创建一个空原型的新对象。
## 尝试
设置构造函数的原型属性为null或undefined
```js
function C(){}
C.prototype=null;
```
## 结果
实例化该构造函数仍然得到是Object的实例。
```js
var c=new C();
Object.getPrototypeOf(c) === null;//false
Object.getPrototypeOf(c) === Object.prototype;//true
```
## ES5标准方法
ES5提供了标准方法来创建一个没有原型的对象。Object.create函数能够使用一个用户指定的原型链和一个属性描述符动态地构造对象。属性描述符描述了新对象属性的值及特性。通过简单传递一个null原型参数和一个空的描述符，就可以建立一个真正的空对象。
```js
var x=Object.create(null);
Object.getPrototypeOf(x)==null;//true
```
原型污染无法影响这样的对象。

## 兼容版本
一些不支持Object.create函数的旧的js环境可能支持__proto__属性，对象字面量也支持初始化一个原型链为null的新对象。
```js
var x={__proto__:null};
x instanceof Object;//false
```
上面的方法也很在效，但在有标准方法后，标准方法是更好的选择。

## 注意
__proto__属性是非标准的并且并不是所有环境都可以移植的。js的实现者们并不能保证以后仍然支持它，所以有标准方法，尽量先考虑标准方法。
可以看出，虽然__proto__可以解决问题，但也引入它自身平台不兼容的问题，阻止自由原型对象作为真正健壮的字典实现。更健壮的方法，在下一条中会提到。

## 提示
- 在ES5环境中，使用Object.create(null)创建的自由原型的空对象是不太容易被污染的
- 在一些较老的环境中，考虑使用{__proto__:null}
- 但要注意__proto__既不标准，也不是完全可移植的，并且可能会在未来的js环境中去除
- 绝不要使用"__proto__"名作为字典的key，因为一些环境将其作为特殊的属性对待。
