---
title: 第47条：绝不要在Object.prototype中增加可枚举的属性
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---
之前的几条都不断地重复着for...in循环，它便利好用，但又容易被原型污染。for...in循环最常见的用法是枚举字典中的元素。这里就是从侧面提出不要在共享的Object.prototype中增加可枚举的属性。
这就导致，我们在开发的时候，不能在Object.prototype中添加有用的方法。如，我们想增加一个产生对象属性名数组的allKeys方法将会怎么样？
```js
Object.prototype.allKeys=function(){
    var res=[];
    for(var key in this){
       res.push(key);
    }
    return res;
}
```
上面的方法污染了它自身。
```js
({a:1,b:2,c:3}).allKeys();//['allKeys','a','b','c']
```
可以用之前说的方法改进allKeys方法忽略掉Object.prototype中的属性。
## 独立为函数
把要添加的方法独立为一个函数。
```js
function allKeys(obj){
    var res=[];
    for(var key in this){
       res.push(key);
    }
    return res;
}
```
这样就可以使用，而又可以不污染到原型。for...in循环也不会出错。
## 设置为不可枚举
如果想在Object.prototype中增加属性，ES5提供了Object.defineProperty方法，可以定义一个对象的属性并指定该属性的元数据。通过设置其可枚举性为false使其在for...in循环中不可见。
```js
Object.defineProperty(Object.prototype,'allKeys',{
    value:function(){
        var res=[];
        for(var key in this){
           res.push(key);
        }
        return res;
    },
    writable:true,
    enumerable:false,
    configurable:true
});
```
调用一下可得
```js
({a:1,b:2,c:3}).allKeys();//['a','b','c']
```
上面的代码不会污染其他所有Object实例的所有for...in循环。事实上，当你需要增加一个不应该在for...in循环中出现的属性时，Object.defineProperty方法可以用来定义属性。

## 提示
- 避免在Object.prototype中增加属性
- 考虑编写一个函数代替Object.prototype方法
- 如果你确定需要在Object.prototype中增加属性，使用ES5中的Object.defineProperty方法将它们定义为不可枚举的属性


