---
title: 第45条：使用hasOwnProperty方法以避免原型污染
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---
之前的43条，44条讨论了属性的枚举，但都没有彻底地解决属性查找中原型污染的问题。看下面关于字典的一些操作
```js
'zhangsan' in dict;
dict.zhangsan;
dict.zhangsan=22;
```
js的对象操作总是经继承的方式工作的。即使是一个空的对象字面量也是继承了Object.protoype属性。
```js
var dict={};
'zhangsan' in dict;//false
'lisi' in dict;//false
'wangwu' in dict;//false
'toString' in dict;//true
'valueOf' in dict;//true
```
无法避免从Object.prototype对象继承方法。
## 去除原型污染
Object.prototype提供了hasOwnProperty方法。当测试字典条目时，它可以避免原型污染，这正好可以解决之前的问题。看一段代码
```js
dict.hasOwnProperty('zhangsan');//false
dict.hasOwnProperty('toString');//false
dict.hasOwnProperty('valueOf');//false
```
可以利用上面代码的特性，来对属性查找使用，可以避免其受原型污染
```js
dict.hasOwnProperty('zhangsan')?dict.hasOwnProperty('zhangsan'):undefined;
dict.hasOwnProperty('x')?dict.hasOwnProperty('x'):undefined;
```
这里还有一个问题，我们是使用dict对象的hasOwnProperty方法，但其实它自身并没有这个方法，而是继承自Object.prototype对象。如果dict字典对象有一个同为"hasOwnProperty"名称的属性，那么原型中的hasOwnProperty方法不会被访问到。这里会优先读取自身包含的属性，找不到才会从原型链中查找。
```js
dict.hasOwnProperty=10;
dict.hasOwnProperty('zhangsan');//这里会产生一个错误
```
虽然字典很少会存储这样的属性名。但小概率事件也会发生，因为你不知道处理的数据，是不是来自第三方。下面就介绍一种最安全的方法，不做任何假设。这里不用字典对象来访问hasOwnProperty方法，可能被改写。
### 使用Object.prototpye.hasOwnProperty
这里直接使用Object.prototype中的hasOwnProperty方法，然后使用函数的call方法，把函数的接收者绑定到字典对象。
首先，先提取出hasOwnProperty方法
```js
var hasOwn=Object.prototype.hasOwnProperty;
```
或
```js
var hasOwn={}.hasOwnProperty;
```
然后，确定函数运行的接收者，使用call方法来指定
```js
hasOwn.call(dict,'zhangsan');
```
这里就可以安全地调用Object.prototype.hasOwnProperty方法来对字典对象的属性名进行检测了。
```js
var dict={};
dict.zhangsan=12;
hasOwn.call(dict,'hasOwnProperty');//false
hasOwn.call(dict,'zhangsan');//true

dict.hasOwnProperty=10;
hasOwn.call(dict,'hasOwnProperty');//true
hasOwn.call(dict,'zhangsan');//true
```
为了避免所以地方都插入上面的检测代码，可以把该模式抽象为Dict的方法。
## Dict初级版
Dict构造函数封装了所有在单一数据类型定义中编写健壮字典的技术细节。代码好下
```js
function Dict(elements){
    this.elements=elements||{};
}
Dict.prototype.has=function(key){
    return {}.prototype.hasOwnProperty.call(this.elements,key);
};
Dict.prototype.get=function(key){
    return this.has(key)?this.elements[key]:undefined;
};
Dict.prototype.set=function(key,val){
    this.elements[key]=val;
};
Dict.prototype.remove=function(key){
    delete this.elements[key];
};
```
这样的实现比使用js默认的对象语法更健壮，而且也同样方便使用。
```js
var dict=new Dict({
    zhangsan:12,
    lisi:23,
    wangwu:40
});
dict.has('zhangsang');//true
dict.has('lisi');//true
dict.has('toString');//false
```
## __proto__自身污染
之前在44条中提到，在一些特殊js环境中，特殊的属性名__proto__可能导致自身的污染问题。在某些环境中,__proto__只是简单地继承自Object.prototype，因此空对象是真正的空对象。
一些环境下
```js
var empty=Object.create(null);
'__proto__' in empty;//false
var hasOwn={}.hasOwnProperty;
hasOwn.call(empty,'__proto__');//false
```
在其它环境下
```js
var empty=Object.create(null);
'__proto__' in empty;//true
var hasOwn={}.hasOwnProperty;
hasOwn.call(empty,'__proto__');//false
```
某些环境下
因为存在一个实例属性__proto__而永久污染所有的对象
```js
var empty=Object.create(null);
'__proto__' in empty;//true
var hasOwn={}.hasOwnProperty;
hasOwn.call(empty,'__proto__');//true
```
在不同的环境中，__proto__的值无法确定
```js
var dict=new Dict();
dict.has('__proto__');//无法确定
```
为了达到代码的可移植性和安全性，只能对__proto__关键字增加一些操作。
## Dict最终版
下面就是Dict更安全、更复杂的最终实现。
```js
function Dict(elements){
    this.elements=elements||{};
    this.hasSpecialProto=false;
    this.specialProto=undefined;
}
Dict.prototype.has=function(key){
    if(key === '__proto__'){
        return this.hasSpecialProto;
    }
    return {}.prototype.hasOwnProperty.call(this.elements,key);
};
Dict.prototype.get=function(key){
    if(key === '__proto__'){
       return this.specialProto;
    }
    return this.has(key)?this.elements[key]:undefined;
};
Dict.prototype.set=function(key,val){
    if(key === '__proto__'){
        this.hasSpecialProto=true;
        this.specialProto=val;
    }else{
        this.elements[key]=val;
    }
};
Dict.prototype.remove=function(key){
    if(key === '__proto__'){
        this.hasSpecialProto=false;
        this.specialProto=undefined;
    }else{
        delete this.elements[key];
    }    
};
```
不管环境处不处理__proto__属性，以上代码都能工作。
```js
var dict=new Dict();
dict.has('__proto__');//false
```

## 提示
- 使用hasOwnProperty方法避免原型污染
- 使用词法作用域和call方法避免覆盖hasOwnProperty方法
- 考虑在封装hasOwnProperty测试样板代码的类中实现字典操作
- 使用字典类避免将'__proto__'作为key来使用




