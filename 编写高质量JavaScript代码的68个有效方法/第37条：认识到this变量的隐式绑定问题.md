
---
title: 第37条：认识到this变量的隐式绑定问题
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---

## CSVReader示例
### 需求
CSV（逗号分隔型取值）文件格式是一种表格数据的简单文本表示
```html
张三,1982,北京,中国
小森,1982,东京,日本
吉姆,1958,纽约,美国
```
现需要编写一个简单的、可定制的读取CSV数据的类。这里的分隔符是基于逗号的，但也可以处理一些其它字符作为分隔符。
### 分析 
构造函数需要一个可选的分隔器字符数组并构造出一个自定义的正则表达式以将每一行分成不同的条目。
```js
function CSVReader(separators){
  this.separators=separators||[','];
  this.regexp=new RegExp(this.separators.map(function(sep){
      return '\\'+sep[0];
  }).join('|'));
}
```
实现一个简单的read方法分两步处理：第一步，将输入字符串分为接待划分的数组。第二步，将数组的每一行分为按单元划分的数据。结果应该是一个二维数组。
```js
CSVReader.prototype.read=function(str){
  var lines=str.trim().split(/\n/);
  return lines.map(function(line){
     return line.split(this.regexp);
  })
};
var reader=new CSVReader();
reader.read('a,b,c\nd,e,f\n');//[['a,b,c'],['d,e,f']]
```
### 问题
这里有个严重而微妙的Bug。传递给line.map的回调函数引用this，它期望能提取到CSVReader对象的regexp属性。然而，map函数将其回调函数的接收者绑定到了lines数组，该lines数组并没有regexp属性。其结果是，this.regexp产生undefined值，使得调用line.split陷入混乱。无法对line进行处理，得到数组。
导致这一Bug的事实：this变量是以不同的方式绑定的。每个函数都有一个this变量的隐式绑定。该this变量的绑定值是在调用该函数时确定的。对于一个词法作用域的变量，可以通过查找显式命名的绑定名来识别出其绑定的接收者。但this变量是隐式地绑定到最近的封闭函数。因此，对于CSVReader.prototype.read函数，this变量的绑定不同于传递给lines.map回调函数的this绑定。
注：上面这段好像有点绕，一会画张图
## 回调函数中的this
### API参数指定
幸运的是，数组的map方法可以传入一个可选参数作为其回调函数的this绑定。所以，修复该Bug的最简单的方法是将外部的this绑定通过map的第二个参数传递给回调函数。

```js
CSVReader.prototype.read=function(str){
  var lines=str.trim().split(/\n/);
  return lines.map(function(line){
     return line.split(this.regexp);
  },this);
};
var reader=new CSVReader();
reader.read('a,b,c\nd,e,f\n');//[['a','b','c'],['d','e','f']]
```
### 词法作用域
但不是所有基于回调函数的API都是考虑周全。提供额外参数指定绑定接收者的。我们需要另外一种获取到外部函数this绑定的方式，以便回调函数仍然能引用它。直截了当的解决方案是使用词法作用域的变量来存储这个额外的外部this绑定的引用。
```js
CSVReader.prototype.read=function(str){
  var lines=str.trim().split(/\n/);
  var self=this;
  return lines.map(function(line){
     return line.split(self.regexp);
  });
};
var reader=new CSVReader();
reader.read('a,b,c\nd,e,f\n');//[['a','b','c'],['d','e','f']]
```
通常会使用self这个变量名，以表明该变量的唯一目的就是作为当前作用域this绑定的额外别名。（我是经常使用 that）
### 函数bind方法
ES5中提供了另一种有效的方法是使用回调函数的bind方法。
```js
CSVReader.prototype.read=function(str){
  var lines=str.trim().split(/\n/);
  return lines.map(function(line){
     return line.split(this.regexp);
  }.bind(this));
};
var reader=new CSVReader();
reader.read('a,b,c\nd,e,f\n');//[['a','b','c'],['d','e','f']]
```

## 提示
- this变量的作用域总是由其最近的封闭函数所确定
- 使用一个局部变量（通常命名为self,me,that）使用this绑定对于内部函数是可用的

## 附录一：数组map方法
### 概述
map() 方法返回一个由原数组中的每个元素调用一个指定方法后的返回值组成的新数组。

### 语法
```js
array.map(callback[, thisArg])
```
### 参数
#### callback:原数组中的元素经过该方法后返回一个新的元素。
-currentValue:callback 的第一个参数，数组中当前被传递的元素。
-index:callback 的第二个参数，数组中当前被传递的元素的索引。
-array:callback 的第三个参数，调用 map 方法的数组。
#### thisArg:执行 callback 函数时 this 指向的对象。
### 描述
map 方法会给原数组中的每个元素都按顺序调用一次 callback 函数。
callback 每次执行后的返回值组合起来形成一个新数组。 
callback 函数只会在有值的索引上被调用；那些从来没被赋过值或者使用 delete 删除的索引则不会被调用。
callback 函数会被自动传入三个参数：数组元素，元素索引，原数组本身。

如果 thisArg 参数有值，则每次 callback 函数被调用的时候，this 都会指向 thisArg 参数上的这个对象。如果省略了 thisArg 参数,或者赋值为 null 或 undefined，则 this 指向全局对象。

注：map不修改调用它的原数组本身（当然可以在 callback 执行时改变原数组）。
使用 map 方法处理数组时，
数组元素的范围是在callback方法第一次调用之前就已经确定了。
在map方法执行的过程中：
原数组中新增加的元素将不会被 callback 访问到；
若已经存在的元素被改变或删除了，则它们的传递到 callback 的值是 map 方法遍历到它们的那一时刻的值；而被删除的元素将不会被访问到。

### 示例
#### 例子：将数组中的单词转换成对应的复数形式.
下面的代码将一个数组中的所有单词转换成对应的复数形式.
```js
function fuzzyPlural(single) {
  var result = single.replace(/o/g, 'e');  
  if( single === 'kangaroo'){
    result += 'se';
  }
  return result; 
}

var words = ["foot", "goose", "moose", "kangaroo"];
console.log(words.map(fuzzyPlural));

// ["feet", "geese", "meese", "kangareese"]
```
#### 例子：求数组中每个元素的平方根
下面的代码创建了一个新数组，值为原数组中对应数字的平方根。
```js
var numbers = [1, 4, 9];
var roots = numbers.map(Math.sqrt);
/* roots的值为[1, 2, 3], numbers的值仍为[1, 4, 9] */
```
#### 例子：在字符串上使用 map 方法
下面的例子演示如在在一个 String  上使用 map 方法获取字符串中每个字符所对应的 ASCII 码组成的数组：
```js
var map = Array.prototype.map
var a = map.call("Hello World", function(x) { return x.charCodeAt(0); })
// a的值为[72, 101, 108, 108, 111, 32, 87, 111, 114, 108, 100]
```
### 使用技巧案例
通常情况下，map 方法中的 callback 函数只需要接受一个参数，就是正在被遍历的数组元素本身。但这并不意味着 map 只给 callback 传了一个参数。这个思维惯性可能会让我们犯一个很容易犯的错误。
```js
// 下面的语句返回什么呢:
["1", "2", "3"].map(parseInt);
// 你可能觉的会是[1, 2, 3]
// 但实际的结果是 [1, NaN, NaN]

// 通常使用parseInt时,只需要传递一个参数.但实际上,parseInt可以有两个参数.第二个参数是进制数.可以通过语句"alert(parseInt.length)===2"来验证.
// map方法在调用callback函数时,会给它传递三个参数:当前正在遍历的元素, 元素索引, 原数组本身.
// 第三个参数parseInt会忽视, 但第二个参数不会,也就是说,parseInt把传过来的索引值当成进制数来使用.从而返回了NaN.

/*
//应该使用如下的用户函数returnInt

function returnInt(element){
  return parseInt(element,10);
}

["1", "2", "3"].map(returnInt);
// 返回[1,2,3]
*/
```
### 兼容旧环境
map 是在最近的 ECMA-262 标准中新添加的方法；所以一些旧版本的浏览器可能没有实现该方法。在那些没有原生支持 map 方法的浏览器中，你可以使用下面的 Javascript 代码来实现它。所使用的算法正是 ECMA-262，第 5 版规定的。假定Object, TypeError, 和 Array 有他们的原始值。而且 callback.call 的原始值也是 Function.prototype.call
```js
// 实现 ECMA-262, Edition 5, 15.4.4.19
// 参考: http://es5.github.com/#x15.4.4.19
if (!Array.prototype.map) {
  Array.prototype.map = function(callback, thisArg) {
    var T, A, k;
    if (this == null) {
      throw new TypeError(" this is null or not defined");
    }
    // 1. 将O赋值为调用map方法的数组.
    var O = Object(this);
    // 2.将len赋值为数组O的长度.
    var len = O.length >>> 0;
    // 3.如果callback不是函数,则抛出TypeError异常.
    if (Object.prototype.toString.call(callback) != "[object Function]") {
      throw new TypeError(callback + " is not a function");
    }
    // 4. 如果参数thisArg有值,则将T赋值为thisArg;否则T为undefined.
    if (thisArg) {
      T = thisArg;
    }
    // 5. 创建新数组A,长度为原数组O长度len
    A = new Array(len);
    // 6. 将k赋值为0
    k = 0;
    // 7. 当 k < len 时,执行循环.
    while(k < len) {
      var kValue, mappedValue;
      //遍历O,k为原数组索引
      if (k in O) {
        //kValue为索引k对应的值.
        kValue = O[ k ];
        // 执行callback,this指向T,参数有三个.分别是kValue:值,k:索引,O:原数组.
        mappedValue = callback.call(T, kValue, k, O);
        // 返回值添加到新数组A中.
        A[ k ] = mappedValue;
      }
      // k自增1
      k++;
    }
    // 8. 返回新数组A
    return A;
  };      
}
```