---
title: 第28条：不要信赖函数对象的toString方法
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---

js函数有一个非凡的特性，即将其源代码重现为字符串的能力。

<pre class="brush:html;gutter:true;">(function(x){
   return x+1
}).toString();//"function (x){   return x+1}"</pre>

反射获取函数源代码的功能很强大，使用函数对象的toString方法有严重的局限性。
toString方法的局限性
ECMAScript标准对函数对象的toString方法的返回结果（即该字符串）并没有任何要求。这意味着不同的js引擎将产生不同的字符串，甚至产生的字符串与该函数并不相关。

如果函数是使用纯js实现的，那么js引擎会试图提供该函数的源代码的真实表示。

## 一个失败的例子

<pre class="brush:html;gutter:true;">(function(x){
     return x+1
}).bind(16).toString();//"function () { [native code] }"</pre>

### 失败原因：

使用了由宿主环境的内置库提供的函数。

*   由于许多宿主环境中，bind函数是由其他编程语言实现的（通常是c++）。宿主环境提供的是一个编译后的函数，在此环境下该函数没有js的源代码供显示。

*   由于标准允许浏览器引擎改变toString方法的输出，很容易使编写的程序一个js系统中正确运行，在其他js系统中却无法正确运行。程序对函数的源代码字符串的具体细节很敏感，即使js的实现有一点细微的变化都可能破坏程序。

*   由toString方法生成的源代码并不展示闭包中保存的与内部变量引用相关的值
<pre class="brush:html;gutter:true;">(function(x){
   return function(y){
      return x+y;
   }
})(42).toString();//"function (y){      return x+y;   }"</pre>

注意：尽管函数实际上是一个绑定x为42的闭包，但结果字符串仍然包含一个引用x的变量。

从某种意义上说，js的toString方法的这些局限使其用来提取函数源代码并不是特别有用和值得信赖。通常应该避免使用它。对提取函数源代码相当复杂的使用应当采用精心制作的js解释器和处理库。将js函数看作是一个不该违背的抽象是最稳妥的。

## 提示

*   当调用函数的toString方法时，并没有要求js引擎能够精确地获取到函数的源代码

*   由于在不同的引擎下调用toString方法的结果可能不同，所以绝不要信赖函数源代码的详细细节

*   toString方法的执行结果并不会暴露存储在闭包中的局部变量

*   通常情况下，应该避免使用函数对象的toString方法

## 附录一：toString方法

不同数据类型调用toString方法的结果。
toString方法是Object原型对象中的一个方法，所以继承自这个类的对象都会继承这个方法，并可以对toString方法进行覆盖。
js标准库中的5种简单数据类型：Undefined,Null,Boolean,Number和String。还有一种复杂的数据类型Object，Object的本质是一组无序的名值对组成。

### 简单数据类型

<pre class="brush:html;gutter:true;">//数字
(Undefined).toString();//"error"
(Null).toString();//error
(true).toString();//"true"
(1).toString();//"1"
('111').toString();//"111"</pre>

可以看出其中除了Undefined和Null类型外，为什么其它几个基本类型可以运行呢。
这在我们之前的文章《[[Effective JavaScript 笔记] 第4条：原始类型优于封闭对象](http://www.cnblogs.com/wengxuesong/p/5474106.html)》中讲到，当简单数据类型调用toString方法会首先把原始类型转换成包装对象。
此时对应的包装对象为

*   数字为Number对象

*   布尔值为Boolean对象

*   字符串为String对象

这些对象也都是继承自Object对象的，并重写了各自的toString方法。
但Undefined类型和Null类型都只有一个值undefined,null，并没有对应的封装对象。
虽然typeof null的值是"object"，但并没用。

### 引用类型

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">//Object对象
({a:10,b:20}).toString();//"[object Object]"
//Date对象
(new Date).toString();//"Tue Jun 07 2016 15:37:15 GMT+0800 (中国标准时间)"
//RegExp对象
(/^sss$/g).toString();//"/^sss$/g"
//Function对象
function aa(){return "bb"}
aa.toString();//"function aa(){return "bb"}"
//window对象
window.toString();//"[object Window]"
//Math对象
Math.toString();//"[object Math]"
</pre>
</div>

<span style="line-height: 1.5;">看到上面的toString方法，Object,window,Math是使用Object原型方法。其它对象都使用了自身覆盖的toString方法。</span>

### typeof操作符

对以上所有类型使用typeof操作符时会得到以下的结果

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">typeof 1;//"number"
typeof '1';//"string"
typeof true;//"boolean"
typeof undefined;//"undefined"
typeof (function a(){});//"function"
typeof null;//"object"
typeof {};//"object"
typeof (new Date);//"object"
typeof [];//"object"
typeof window;//"object"
typeof Math;//"object"
typeof (/sdfsf/g);//"object"
</pre>
</div>

可以看出，想使用单单的typeof操作符来对类型进行判断几乎是不可能的。

有人可能会说对于返回object字符串，可以使用构造函数来判断类型即instanceOf方法。

<pre class="brush:html;gutter:true;">({}) instanceof Object;//true
(new Date) instanceof Date;//true
([]) instanceof Array;//true
(/sdfsf/g) instanceof RegExp;//true</pre>

然后null类型只要

<pre class="brush:html;gutter:true;">var a=null;
a===null;//true;</pre>

好像可以实现下面这样的类型判断代码了

<pre class="brush:html;gutter:true;">function getType(obj){
    if(typeof obj !== 'object'){
        return typeof obj;
    }else{   
        if(obj===null){
            return 'null';
        }
        if(obj===window){
            return 'window'; 
        }
        if(obj===Math){
            return 'Math'
        }      
        if((obj) instanceof Date){
            return 'date';
        }
        if((obj) instanceof Array){
            return 'array';
        }
        if((obj) instanceof RegExp){
            return 'regexp';
        } 
        if((obj) instanceof Object){
            return 'object';
        }

    }
}
</pre>

上面代码是否可以运行测试一下，并没有问题

<pre class="brush:html;gutter:true;">getType(1);//"number"
getType(true);//"boolean"
getType('1');//"string"
getType(undefined);//"undefined"
getType(function(){});//"function"
getType(/sf/);//"regexp"
getType(null);//"null"
getType(window);//"window"
getType({});//"object"
getType([]);//"array"</pre>

但这里要注意的一个问题就是，这个代码里的对于object类型的检测一定要放到最后面。
如下所示，所有对象都是继承自Object，所以instanceof检测所有对象是否为Object类型的实例返回都是true

<pre class="brush:html;gutter:true;">([]) instanceof Array;//true
([]) instanceof Object;//true</pre>

看到以上代码是不是觉得太复杂麻烦了，有没有一种更简单的方法来对类型进行判断呢？答案当然是有，下面来看toString方法的运用。

### toString应用

如上面所说，继承自Object的对象都有toString方法，但每个对象实现了各自的toString方法，导致无法用toString方法进行类型判断。这里可以利用之前讲到过的call或apply方法来调用Object.prototype.toString方法。

<pre class="brush:html;gutter:true;">function getType(obj){
   var toString=Object.prototype.toString;
   return toString.call(obj);   
}
</pre>

测试一下各类型会得到如下结果

<pre class="brush:html;gutter:true;">getType(1);//"[object Number]"
getType(true);//"[object Boolean]"
getType('1');//"[object String]"
getType(undefined);//"[object Undefined]"
getType(function(){});//"[object Function]"
getType(/sf/);//"[object RegExp]"
getType(null);//"[object Null]"
getType(window);//"[object Window]"
getType({});//"[object Object]"
getType([]);//"[object Array]"
getType(Math);//"[object Math]"</pre>

所有类型都可以区分出来，是不是很简单呀！

还有个特殊的值需要注意NaN.

<pre class="brush:html;gutter:true;">getType(NaN);//"[object Number]"</pre>

NaN是一个Number类型的特殊值，它是表达是一个不是一个数字的值，这里对这个值也要进行处理。可以关注之前文章《[[Effective JavaScript笔记]第3条：当心隐式的强制转换](http://www.cnblogs.com/wengxuesong/p/5463026.html)》里关于NaN的内容。处理代码如下

<pre class="brush:html;gutter:true;">function isReallyNaN(x){
   return x!==x;
}
</pre>

&nbsp;

## 备忘：

这里需要去了解一下，js解释器的知识。
相关的链接有：
[javascript设计模式之解释器模式详解](http://www.jb51.net/article/50680.htm)
[javascript设计模式 - 解释器模式(interpreter)](http://www.cnblogs.com/webFrontDev/p/3533129.html)
[Chrome V8](https://developers.google.com/v8/)