---
title: 第22条：使用arguments创建可变参数的函数
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---

[第21条讲述使用可变参数的函数average](http://www.cnblogs.com/wengxuesong/p/5545281.html)。该函数可处理任意数量的参数并返回这些参数的平均值。

## 如何创建可变参数的函数

### 1、实现固定元数的函数

书上的版本
 <div class="cnblogs_Highlighter"><pre class="brush:javascript;gutter:true;">function averageOfArray(a){
    for(var i=0,sum=0,n=a.length;i&lt;n;i++){
        sum+=a[i];
    }
    return sum/n
}
</pre></div>

<span style="line-height: 1.5">使用ES5 Array.reduce方法的版本</span>

<div class="cnblogs_Highlighter"><pre class="brush:javascript;gutter:true;">function averageOfArrayES5(a){
    if({}.toString.call(a) !== '[Object Array]')a=[].slice.call(a);
    var n=a.length;
    var sum=a.reduce(function(prev,cur){return prev+cur;});
    return sum/n;
}
</pre></div>

<span style="line-height: 1.5">调用方法</span>

<div class="cnblogs_Highlighter"><pre class="brush:javascript;gutter:true;">averageOfArray([2,7,1,8,3,4,5]);
</pre></div>

<span style="line-height: 1.5">averageOfArray函数定义了一个形参，即参数列表中的变量a。当调用函数averageOfArray时，只需提供单个参数，即数字数组。</span>

### 2、利用arguments实现可变元数函数

利用js的一个事实，即给每个函数都隐式地提供了一个名为arguments的局部变量。arguments对象是一个类数组。它为每个实参提供了一个索引属性，还包含一个length属性用来指示参数的个数。从而可以通过遍历arguments对象的每个元素来实现可变元数的函数

<div class="cnblogs_Highlighter"><pre class="brush:javascript;gutter:true;">function average(){
    for(var i=0,sum=0,n=arguments.length;i&lt;n;i++){
        sum+=arguments[i];
    }
    return sum/n;
}
</pre></div>

<span style="line-height: 1.5">同样使用ES5 Array.reduce方法的版本，这里需要对arguments类数组对象转化为真正的对象。我们使用Array.prototype.slice()方法结合之前讲的函数的call方法来处理。</span>

代码如下：

<div class="cnblogs_Highlighter"><pre class="brush:javascript;gutter:true;">var obj={
   "0":100,
   "1":"sdfsafd",
   length:2,
   each:function(){

   }
}
Array.prototype.slice.call(obj);//[100, "sdfsafd"]
</pre></div>

<span style="line-height: 1.5">然后把参数对象转化一下</span>

<div class="cnblogs_Highlighter"><pre class="brush:javascript;gutter:true;">function average(){
    var a=[].slice.call(arguments);
    var sum=a.reduce(function(prev,cur){return prev+cur;});
    return sum/n;
}
</pre></div>

<span style="line-height: 1.5">注：其中[].slice.call()方法和Array.prototype.slice.call()方法，效果相同。类似的Object.prototype.toString.call()和({}).toString.call()方法。想了解更详细，可以自行查找原型链的相关知识。</span>

可变参数函数提供了灵活的接口。不同的调用者可使用不同数量的参数来调用它们。但它们自身也失去了一点便利。
想使用计算数组参数来调用可变参数的函数，只能使用apply()方法。如果提供了一个便利的可变参数的函数，也最好提供一个需要显示指定数组的固定元数的版本。

### 3、提供一个轻量级的封装

<div class="cnblogs_Highlighter"><pre class="brush:javascript;gutter:true;">function average(){
    return averageOfArray(arguments);
}
</pre></div>

这里编写了一个轻量级的封装，并委托给固定元数的版本来实现可变参数的函数。

## 实现函数的重载

可以根据传入参数的情况，实现适当的功能。 

<div class="cnblogs_Highlighter"><pre class="brush:javascript;gutter:true;">function doAdd(){
  if(arguments.length==1){
    return arguments[0]+10;
  }else if(arguments.length==2){
    return arguments[0]+arguments[1];
  }
}
doAdd(10);//20
doAdd(30,20);//50
</pre></div>

<span style="line-height: 1.5">arguments对象和命名参数一起使用。</span>&nbsp;

<div class="cnblogs_Highlighter"><pre class="brush:javascript;gutter:true;">function doAdd(num1,num2){
  if(arguments.length==1){
    return num1+10;
  }else if(arguments.length==2){
    return num1+num2;
  }
}
doAdd(10);//20
doAdd(30,20);//50
</pre></div>

<span style="line-height: 1.5">下面的代码对于使用jquery的同学应该不会很陌生。</span>

<div class="cnblogs_Highlighter"><pre class="brush:javascript;gutter:true;">$('body').css('font-size');
$('body').css('font-size',30);
</pre></div>

<span style="line-height: 1.5">用得就是类似的特性。</span>

<span style="line-height: 1.5">对于这方面再进行扩展一下。比如根据传入参数的个数及参数的类型，来完成对应不同的功能。jquery里的很多API，都是用相关的方法来实现的，有兴趣的可以看一下jquery的源码。</span>

## 提示

- 使用隐式的arguments对象实现可变参数的函数
- 考虑对可变参数的函数提供一个额外的固定元数的版本，从而使用者无需借助apply方法。

## 附录：arguments对象

arguments对象是一个类数组的对象，包含着传入函数中的所有参数。
下面我们示例看一下，这个arguments对象到底包含哪些东西。

<div class="cnblogs_Highlighter"><pre class="brush:javascript;gutter:true;">function a(){
   console.log(arguments);
}
a(1,2,3,6,7);//
</pre></div>

<span style="line-height: 1.5">得到如下的图示</span>

[![1464760773729](http://images2015.cnblogs.com/blog/156514/201606/156514-20160601144238352-1571421111.jpg "1464760773729")](http://images2015.cnblogs.com/blog/156514/201606/156514-20160601144237649-1878119964.jpg)

### <span style="line-height: 1.5; font-size: 1.17em">arguments对象属性</span><span style="line-height: 1.5; font-size: 1.17em">：</span>

#### 索引值 "0","1"代表对应参数。

<div class="cnblogs_Highlighter"><pre class="brush:javascript;gutter:true;">function a(){
   console.log(arguments[3]);
}
a(1,2,3,6,7);//6</pre></div>

#### length代表参数长度。

<div class="cnblogs_Highlighter"><pre class="brush:javascript;gutter:true;">function a(){
   console.log(arguments.length);
}
a(1,2,3,6,7);//5</pre></div>

#### callee是一个指针指向拥有这个arguments对象的函数。

<div class="cnblogs_Highlighter"><pre class="brush:javascript;gutter:true;">function a(){
   console.log(arguments.callee);
}
a(1,2,3,6,7);//function a(){...}</pre></div>

#### caller是一个在ES3并没有定义的属性。

这个属性中保存着调用当前函数的函数的引用，如果是全局作用域中调用当前函数，它的值为null。

<div class="cnblogs_Highlighter"><pre class="brush:javascript;gutter:true;">function a(){
   console.log(arguments.caller);
}
a(1,2,3,6,7);//null</pre></div>

在严格模式下，arguments.callee和arguments.caller都会报错。
ES5中arguments.caller的值始终是undefined。
arguments的值永远与对应命名参数的值保持同步。代码示例:

<div class="cnblogs_Highlighter"><pre class="brush:javascript;gutter:true;">function b(name,age){
   arguments[1]=20;
   return name+'是'+age+'岁!';
}
b('li lie',30);//"li lie是20岁!"
b('han mei mei');//"han mei mei是undefined岁!"</pre></div>

arguments对象的值会自动反应到对应的命名参数上。所以每次都对argument[1]的修改，也会修改age的值，结果它们都是20。但这并不是说读取这两个值会访问相同的内存空间；它们的内存空间是独立的，但它们的值会同步。
当只传入一个参数时，arguments[1]的值不会反应到命名参数中，因为arguments对象的长度是由传入的参数决定的，不是由定义函数时的命名参数的个数决定的。此时arguments[1]虽然有值，但命名参数是不会同步值的。
没有传递值的命名参数将自动被赋undefined值。和定义变量但没初始化一样。
**<span style="color: #ff0000">注意：</span>**在严格模式下对arguments对象的赋值会变得无效。就算像上面那样arguments[1]=20，age的值也不会改变。另外重写arguments对象的值会导致错误（代码不会执行）。
**ECMAScript中的所有参数传递的都是值，不可能通过引用传递参数。**

<pre class="brush:javascript;gutter:true;">var a=[1,2,3,4,5];
function b(c){c=[]};
b(a);
a;//[1, 2, 3, 4, 5]</pre>