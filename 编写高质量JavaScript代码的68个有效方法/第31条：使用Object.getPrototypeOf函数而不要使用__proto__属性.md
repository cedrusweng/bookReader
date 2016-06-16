---
title: 第31条：使用Object.getPrototypeOf函数而不要使用__proto__属性
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---
ES5引入Object.getPrototypeOf函数作为获取对象原型的标准API，但由于之前的很多js引擎使用了一个特殊的__proto__属性来达到相同的目的。但有些浏览器并不支持这个__proto__属性，所以并不是完全兼容的。
例如对于拥有null原型的对象，不同的环境结果就不同了。
```js
var empty=Object.create(null);
'__proto__' in empty;//一些环境会返回false,另一些会返回true
```
这就导致结果的不一致，从而影响到依赖这个判断的相关代码。
无论在什么情况下，Object.getPrototypeOf函数都是有效的，而且它是提取对象原型更加标准、可移植的方法。由于__ptoto__属性会污染所有的对象，因此它会导致大量的bug。
对于一些没有提供该ES5 api的js环境，可以很容易利用__proto__属性来实现Object.getPrototypeOf函数。
```js
if(typeof Object.getPrototypeOf === 'undefined'){
    Object.getPrototypeOf=function(obj){
         var t=typeof obj;
         if(!obj || (t !== 'object' && t !== 'function')){
            throw new Error("不是一个对象！");
         }
         return obj.__proto__;
    }
}
```
上面代码对宿主环境是否实现Object.getPrototypeOf函数进行判断，在ES5标准的环境中也很安全。

## 提示
- 使用符合标准的Object.getPrototypeOf函数而不要使用非标准的__proto__属性
- 在支持__proto__属性的非ES5环境中实现Object.getPrototypeOf函数
