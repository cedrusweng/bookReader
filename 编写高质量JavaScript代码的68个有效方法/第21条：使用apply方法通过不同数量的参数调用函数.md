---
title: 第21条：使用apply方法通过不同数量的参数调用函数
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---
## apply()方法定义

函数的apply()方法和call方法作用相同，区别在于接收的参数的方式不同。
apply()方法接收两个参数，一个是对象，一个是参数数组。

## apply()作用

### 1、用于延长函数的作用域

示例：

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var color='red';
var o={color:'blue'};
function sayColor(){
    console.log(this.color);
}
sayColor();//"red"
sayColor.apply(o);//"blue"
</pre>
</div>

&nbsp;这里通过apply()方法把函数动态绑定到了对象o上了，这时this指向o对象，得到结果"blue"。

### 2、对象不需要与方法有任何耦合关系

下面举个耦合的例子，看如何通过apply来解决这种耦合。

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var color='red';
var o={color:'blue'};
function sayColor(){
      console.log(this.color);
}
o.sayColor=sayColor;
o.sayColor();//"blue"
</pre>
</div>

<span style="line-height: 1.5;">这里先将函数放到了对象o中，这里对象和方法就紧耦合到一起了，方法的调用必须通过对象o。</span>

没有使用apply()和call()方法那样灵活。
重构上面代码，得到前例中的代码。

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var color='red';
var o={color:'blue'};
function sayColor(){
      console.log(this.color);
}
sayColor();//"red"
sayColor.apply(o);//"blue"
</pre>
</div>

<span style="line-height: 1.5;">这里对象并没有绑定任何方法，只是在需要使用的时候，利用函数的apply或call方法来动态绑定。</span>

对象和方法之间没有耦合在一起。这里还可以通过ES5提供的bind()方法来完成

### 3、实现可变参数函数传参

下面一个计算任意数量数字平均值的函数

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">average(1,2,3);
average(1);
average(3,1,2,3,5,6,7,8,9);
average(1,2,3,5,2,1,5,6,1,10);
</pre>
</div>

&nbsp;average函数是一个称为可变参数或可变元函数(函数的元数是指其期望的参数个数)的例子。

_当然这个函数也可以写成一个接收数组的形式。_

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">averageOfArray([1,2,3]);
averageOfArray([1]);
averageOfArray([3,1,2,3,5,6,7,8,9]);
averageOfArray([1,2,3,5,2,1,5,6,1,10]);
</pre>
</div>

<span style="line-height: 1.5;">使用可变参数的函数更简洁、优雅。可变参数函数具有便捷的语法，至少让调用者预先明确地知道提供了多少个参数。</span>

如果我有这样一个数组

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var scores=getAllScores();
</pre>
</div>

**如何使用average函数计算平均值呢？**

**1.可变参数函数版本。**
这时就可以和apply()方法配合使用，这里因为函数并没用引用this变量，因此第一个参数我们传入一个null。代码如下：

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var scores=getAllScores();
average.apply(null,scores);
</pre>
</div>

**2.直接参数为数组的形式**

这里可以直接传入数组参数。

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var scores=getAllScores();
averageOfArray(scores);
</pre>
</div>

<span style="line-height: 1.5;">以上两种形式，个人觉得都是可以，反而第二种更简单。多知道一种方法，对于遇到别人写的函数时，可以轻松应对，不需要重构代码。这个好处反而更多。</span>

### 4、实现可变参数方法的传值

示例：buffer对象包含一个可变参数的append方法，该方法添加元素到函数内部的state数组中。

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var buffer={
    state:[],
    append:function(){
        for(var i=0,n=arguments.length;i&lt;n;i++){
            this.state.push(arguments[i]);
        }
    }
};
</pre>
</div>

<span style="line-height: 1.5;">这时append方法可以接受任意多个参数。</span>

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">buffer.append('Hello,');
buffer.append('firtName',' ','lastName','!');
buffer.append('newLine');
</pre>
</div>

<span style="line-height: 1.5;">形式如</span>

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">buffer.append(arg1,arg2,arg3,...)
</pre>
</div>

<span style="line-height: 1.5;">借助apply方法的this参数，我们可以指定一个可计算的数组调用append方法</span>

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">buffer.append.apply(buffer,getInputStrings());
</pre>
</div>

**注意：**<span style="color: #ff0000; line-height: 1.5;">这里的buffer很重要，如果传递不同的对象，则append方法将尝试修改该错误对象的state属性。</span>

## 提示

*   使用apply方法指定一个可计算的参数数组来调用可变参数的函数
*   使用apply方法的第一个参数给可变参数的方法提供一个接收者

## 附录一

### average函数

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">function average(){
    var args=[].slice.call(arguments);
    var sum=args.reduce(function(prev,cur){
        return prev+cur;
    });
    return parseInt(sum/args.length,10);
}
</pre>
</div>

### <span style="font-size: 1.17em; line-height: 1.5;">averageOfArray函数</span>

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">function averageOfArray(arr){    
    var sum=arr.reduce(function(prev,cur){
        return prev+cur;
    });
    return parseInt(sum/arr.length,10);
}</pre>
</div>

### ES5 bind()方法

<div class="cledit-section"><span class="lf">这个方法创建一个函数的实例，其this值会被绑定到传给bind()函数的值。<span class="lf">
例如<span class="lf">
</span></span></span></div>
<div class="cledit-section">
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var color='red';
var o={color:'blue'};
function sayColor(){
    console.log(this.color);
}
var oSayColor=sayColor.bind(o);
oSayColor();//"blue"
</pre>
</div>

兼容低版本,[参考使用下面的版本](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)：

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    if (typeof this !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }

    var aArgs   = [].slice.call(arguments, 1),
        fToBind = this,
        fNOP    = function() {},
        fBound  = function() {
          return fToBind.apply(this instanceof fNOP? this: oThis,
                 aArgs.concat(Array.prototype.slice.call(arguments)));
        };

    if (this.prototype) {
      // Function.prototype doesn't have a prototype property
      fNOP.prototype = this.prototype; 
    }
    fBound.prototype = new fNOP();

    return fBound;
  };
}
</pre>
</div>