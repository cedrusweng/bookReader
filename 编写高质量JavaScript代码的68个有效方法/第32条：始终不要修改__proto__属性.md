---
title: 第32条：始终不要修改__proto__属性
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---
__proto__属性很特殊，它提供了Object.getPrototypeOf方法所不具备的额外能力，即修改对象原型链接的能力。避免修改__proto__属性的最明显的原因是可移植性的问题。并不是所有的平台都支持修改对象原型的特性，所以无法编写可移植的代码。
避免修改__proto__属性的另一个原因是性能问题。所有现代的js引擎都深度优化了获取和设置对象属性的行为，因为这些都是一些常见的js操作。这些优化都是基于引擎对对象结构的认识上。当更改了对象的内部结构（如添加或删除该对象或其原型链中的对象的属性），将会使一些优化失效。修改__proto__属性实际上改变了继承结构本身，这可能是最具破坏性的修改。
避免修改__proto__属性的最大的原因是为了保持行为的可预测性。对象的原型链通过其一套确定的属性及属性值来定义它的行为。修改对象的原型链就像对其进行“大脑移植”，这会交换对象的整个层次结构。在某些情况下这样的操作可能是有用的，但是保持继承层次结构的相对稳定是一个基本的原则。
可以使用ES5中的Object.create函数来创建一个具有自定义原型链的新对象。对于不支持ES5的运行环境，可以使用兼容代码。
```js
if(typeof Object.create === 'undefined'){
    Object.create=function(prototype){
       function F(){};
       F.prototype=prototype;
       return new F();
    }
}
```
## 提示
- 始终不要修改对象的__proto__属性
- 使用Object.create函数给新对象设置自定义的原型

## 附录一：__proto__的语法
### 浏览器兼容性
桌面浏览器
Chrome  Firefox (Gecko) Internet Explorer Opera Safari
(Yes)    (Yes)               11           (Yes) (Yes)
移动端
Android Chrome for Android  Firefox Mobile (Gecko)  IE Mobile Opera Mobile  Safari Mobile
(Yes) (Yes) (Yes) (Yes) (Yes) (Yes)

### 使用方法
```js
var shape = {};
var circle = new Circle();
in real code.
shape.__proto__ = circle;
console.log(shape.__proto__ === circle); // true
```
```js
var shape = function () {};
var p = {
    a: function () {
        console.log('aaa');
    }
};
shape.prototype.__proto__ = p;

var circle = new shape();

circle.a();//aaa

console.log(shape.prototype === circle.__proto__);//true

//or

var shape = function () {
};
var p = {
    a: function () {
        console.log('a');
    }
};

var circle = new shape();
circle.__proto__ = p;


circle.a(); //  a

console.log(shape.prototype === circle.__proto__);//false

//or

function test() {
}
test.prototype.myname = function () {
    console.log('myname');

}
var a = new test()

console.log(a.__proto__ === test.prototype);//true

a.myname();//myname


//or

var fn = function () {
};
fn.prototype.myname = function () {
    console.log('myname');
}

var obj = {
    __proto__: fn.prototype
};


obj.myname();//myname
```