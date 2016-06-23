---
title: 第43条：使用Object的直接实例构造轻量级的字典
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---
js对象的核心是一个字符串属性名与属性值的映射表。使用对象实现字典易如反掌，字典是可变长的字符串与值的映射集合。
## for...in
js提供了枚举一个对象属性名的利器--for...in循环。
```js
var dict={zhangsan:34,lisi:24,wangwu:62};
var people=[];
for(var name in dict){
  people.push(name+":"+dict[name]);
}
people;//["zhangsan:34", "lisi:24", "wangwu:62"]
```
### 使用自定义对象
但一些自定义的对象还会继承其原型对象中的属性，for...in循环除了枚举"自身"属性外，原型链中的属性也不会放过。下面看一下例子
```js
function NaiveDict(){  
}
NaiveDict.prototype.count=function(){
  var i=0;
  for(var name in this){
     i++;
  }
  return i;
};
NaiveDict.prototype.toString=function(){
  return '[Object NaiveDict]';
};
var dict=new NaiveDict();
dict.zhangsan=24;
dict.lisi=34;
dict.wangwu=62;
dict.count();//5
```
上面的代码，实例化一个dict对象，并添加了私有属性(zhangsan,lisi,wangwu)并继承了原型对象的方法(toString,count)，所以当调用count方法时，这个时候的for...in循环，不仅枚举了了私有属性也枚举了原型对象的方法属性(zhangsan,lisi,wangwu,toString,count)。
### 使用数组类型
利用js的灵活性，我们可以给任意的类型的对象添加属性，因此给数组添加属性似乎能工作。
```js
var dict=new Array();
dict.zhangsan=24;
dict.lisi=34;
dict.wangwu=62;
dict.zhangsang;//24
```
当我们给数组的原型对象添加一些原型方法，也就是前面讲的猴子补丁ployfill的使用时，这个时候我们再枚举上面的dict对象的属性时，错误就出现了。
```js
Array.prototype.first=function(){
  return this[0];
};
Array.prototype.last=function(){
  return this[this.length-1];
};
Array.prototype.eq=function(idx){
  if(idx<0)idx=this.length+idx;
  return this[idx];
};
```
运行一下枚举
```js
var names=[];
for(var name in dict){
  names.push(name);
}
names;//["zhangsan", "lisi", "wangwu","first","last"]
```
## 轻量级字典的正确姿势
### 首要原则
应该仅仅将Object的直接实例作为字典，而不是子类，或其它对象。
上面的数组的例子可以改成下面这些的代码
```js
var dict={};
dict.zhangsan=24;
dict.lisi=34;
dict.wangwu=62;
dict.zhangsang;
var names=[];
for(var name in dict){
  names.push(name);
}
names;//["zhangsan", "lisi", "wangwu"]
```
### 需要注意
还是无法避免对于Object.prototype对象的修改，但可以将风险仅仅局限在Object.prototype。
### 优点
虽说所有人都可以修改Object.prototype对象，也会对for...in循环造成影响。但相比之下，增加属性到Array.prototype中是合理的。如之前给不支持数组标准方法的环境中将这些方法增加到Array.prototype中。这些属性也会导致for...in循环，坚持Object的直接实例原则，可以使得for...in循环摆脱原型污染的影响。对于构建行为正确的字典，这是个必要非充分条件。

## 提示
- 使用对象字面量构建轻量级字典
- 轻量级字典应该是Object.prototype的直接子类，以使for...in循环免受原型的污染

## 附录：for...in
以下内容来自：[Mozilla 开发者社区][1]
以任意序迭代一个对象的可枚举属性。每个不同的属性，语句都会被执行一次。
### 语法
```js
for(variable in object){...}
```
### 参数
- variable:每次迭代，一个不同的属性名将赋予variable
- object:可枚举属性被迭代的对象

### 描述
- 只言遍历可枚举属性。
- 循环将迭代对象的所有可枚举属性和从它的构造函数的prototype继承而来的

#### 删除、添加或修改属性
for...in循环以任意序迭代一个对象的属性。如果一个属性在一次迭代中被修改，在稍后被访问，其在循环中的值是其在稍后时间的值。一个在被访问之前已经被删除的属性将不会在之后被访问。在迭代进行时被添加到对象的属性，可能在之后的迭代被访问，也可能被忽略。通常，在迭代过程中最好不要在对象上进行添加、修改或删除属性的操作，除非是对当前正在被访问的属性。因为访问的无序性，都无法保证，添加、修改或删除的属性，会在之后被访问。

### Array迭代和for...in
数组索引是可枚举的整数名，其他方面和普通对象属性没有区别。for...in的无序性，无法保证枚举是按索引顺序进行，但会返回所有可枚举的属性，包括非整数名称和继承的属性。
如果次序很重要，可以使用for循环来迭代数组。

#### 仅迭代自身的属性
只需要对象本身的属性，不包括它继承来的。可以使用getOwnPropertyNames()或执行hasOwnProperty()来确定某属性是否是对象自身的。

### 例子
#### 返回一个对象的所有可枚举属性
```js
var obj={a:2,b:1,c:30};
for(var prop in obj){
  console.log('obj.'+prop+'='+obj[prop]);  
}
// "obj.a = 2"
// "obj.b = 1"
// "obj.c = 30"
```

#### 只返回对象自身的可枚举属性
```js
var triangle = {a:1, b:2, c:3};

function ColoredTriangle() {
  this.color = "red";
}

ColoredTriangle.prototype = triangle;

var obj = new ColoredTriangle();

for (var prop in obj) {
  if( obj.hasOwnProperty( prop ) ) {
    console.log("o." + prop + " = " + obj[prop]);
  } 
}
// "o.color = red"
```


  [1]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...in
