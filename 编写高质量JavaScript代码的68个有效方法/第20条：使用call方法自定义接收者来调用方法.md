---
title: 第20条：使用call方法自定义接收者来调用方法
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---

## 不好的实践

函数或方法的接收者（即绑定到特殊关键字this的值）是由调用者的语法决定的。
方法调用语法将方法被查找的对象绑定到this变量,(可参阅之前文章《[理解函数调用、方法调用及构造函数调用之间的不同](http://www.cnblogs.com/wengxuesong/p/5526533.html)》)。
有时需要使用自定义接收者来调用函数，因为该函数可能并不是期望的接收者对象的属性。
可以将方法作为一个新的属性添加到接收者对象中。使用如下代码：

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var obj={
   temporary:function(a,b,c){console.log('obj')} 
}
var f=function(arg1,arg2,arg3){console.log('f')};
var arg1=0,arg2=1,arg3=2;

obj.temporary=f;//标识1
var result=obj.temporary(arg1,arg2,arg3);
delete obj.temporary;
</pre>
</div>

<span style="line-height: 1.5;">标识1，如果temporary的属性名在对象中已经存在，则会对对象造成功能的破坏。而且取任何名称，都没法确保不和原来的对象名重名，可能这个对象并不是自己创建的。对象可以对属性进行冻结或密封以防止修改和添加(详细见附录)任何新属性。</span>

## 函数对象call方法

**基本描述：**用途是在特定的作用域下来执行函数，实际上等于设置函数体内this对象的值。
函数对象具有一个内置的方法call来自定义接收者。

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">f.call(obj,arg1,arg2,arg3)
</pre>
</div>

<span style="line-height: 1.5;">此行为和直接调用函数自身很相似。</span>

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">f(arg1,arg2,arg3)
</pre>
</div>

<span style="line-height: 1.5;">不同点在于，第一个参数指定了一个显式的接收者对象（即指定函数内部this的指向）</span>

**1、当调用的方法已经被删除、修改或者覆盖时，call方法就可以派上用场了。
**以hasOwnProperty方法为例，因为这个方法是Object的方法，所有对象都可以访问调用这个方法。

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var obj={
    foo:'good'
};
obj.hasOwnProperty('foo');//true
</pre>
</div>

<span style="line-height: 1.5;">然后当这个方法被覆盖时</span>

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">obj.hasOwnProperty=1;
obj.hasOwnProperty('foo');//Uncaught TypeError: obj.hasOwnProperty is not a function(&hellip;)
</pre>
</div>

<span style="line-height: 1.5;">这时，call方法就可以派上用场了</span>

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var hasOwnProperty={}.hasOwnProperty;
delete obj.hasOwnProperty;
hasOwnProperty.call(obj,'foo');//true
hasOwnProperty.call(obj,'hasOwnProperty');//false</pre>
</div>

**2、定义高阶函数时call方法也特别实用。
**高阶函数的一个惯用法是接收一个可选的参数作为调用该函数的接收者。
示例：有一个表示键值对列表的对象，提供了名为forEach的方法。

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var table={
    entries:[],
    addEntry:function(key,value){
        this.entries.push({key:key,value:value});
    },
    forEach:function(f,thisArg){
        var entries=this.entries;
        for(var i=0,n=entries.length;i&lt;n;i++){
            var entry=entries[i];
            f.call(thisArg,entry.key,entry.value,i);
        }
    }
};
</pre>
</div>

<span style="line-height: 1.5;">上面这个例子里允许table对象的使用者，将一个方法作为table.forEach的回调函数f，并为该方法提供一个合理的接收者。例如，实现将table的内容复制到另一个中。</span>

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">table1.forEach(table2.addEntry,table2);
</pre>
</div>

<span style="line-height: 1.5;">我们把上面的代码直接带入上面的代码大家可以晰看到它的执行过程。</span>

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var entries=table1.entries;
for(var i=0,n=entries.length;i&lt;n;i++){
    var entry=entries[i];
    table2.addEntry.call(table2,entry.key,entry.value,i);
}
</pre>
</div>

<span style="line-height: 1.5;">这段代码从table2中提取addEntry方法，forEach方法将table2作为接收者，并反复调用该addEntry方法。</span>

## 提示

*   使用call方法自定义接收者来调用函数

*   使用call方法可以调用在给定的对象中不存在的方法

*   使用call方法定义高阶函数允许使用者给回调函数指定接收者

## 附录一：对象属性

注：以下内容出自《javascript高级程序设计语言》第3版

### 属性类型

ECMAScript-262第5版在定义只有内部才用的特性时，描述了属性的特征。描述这些特性是为了实现JS引擎用的，因此js中不能直接访问它们。为了表示特性是内部值，该规范把它们放到了两对儿方括号中。

例如：[[Enumerable]]。

ECMAScript中有两种属性：数据属性和访问器属性。

### 数据属性

数据属性包含一个数据值的位置。在这个位置可以读取和写入值。

数据属性有4个描述其行为的特性。

*   [[Configurable]]:表示能否通过delete删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为访问器属性。
*   [[Enumerable]]:表示能否通过for-in循环返回属性。像前面例子中那样直接在对象上定义的属性，它们的这个特性默认值为true。
*   [[Writable]]:表示能否修改属性的值。
*   [[Value]]:包含这个属性的数据值。读取属性值的时候，这个位置读;写入属性值的时候，把新值保存在这个位置。这个特性的默认值为undefined。

示例：

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var person={
    name:'Nicholas'
}
</pre>
</div>

<span style="line-height: 1.5;">这里创建的一个名为name的属性，为它指定的值是'Nicholas'。</span>

也就是说person对象的name属性的[[Value]]特性将被设置为'Nicholas'，对这个值的任何修改都将反映在这个位置。

### 修改属性的方法

修改属性默认特性的方法，必须使用 ES5的Object.defineProperty()方法。

这个方法接收三个参数：

*   属性所在的对象
*   属性的名字
*   一个描述符对象

描述符对象的属性必须是:configurable,enumerable,writable和value。设置其中的一或多个值，可以修改对应的特性值。

例如：

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var person={};
Object.defineProperty(preson,'name',{
    writable:false,
    value:'li lei'
});
person.name;//"li lei"
person.name='han mei mei';
person.name;//"li lei"
</pre>
</div>

<span style="line-height: 1.5;">这里把name属性设置为只读的属性，所以无法对name进行修改。严格模式下会报错。</span>

下面是一个不可配置的示例：

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var person={};
Object.defineProperty(preson,'name',{
    configurable:false,
    value:'li lei'
});
person.name;//"li lei"
delete person.name;
person.name;//"li lei"
</pre>
</div>

<span style="line-height: 1.5;">把configurable设置为false，表示不能从对象中删除属性。如果在严格模式下，上面的代码会报错。一旦把属性定义为不可配置的，就不能再把它变回为可配置的。此时再调用Object.defineProperty()方法修改除writable之外的特性，都会导致错误。</span>

注意：在调用Object.defineProperty()方法时，如果不指定，configurable,enumerable,writable特性的默认值都是false。多数情况下，都没必要利用Object.defineProperty()方法提供的这些的高级功能。对于理解js对象非常有用。

### 访问器属性

访问器属性不包含数据值;它们包含一对儿getter和setter函数（不过，这两个函数都不是必需的）。

读取访问器属性时，会调用getter函数，这个函数负责返回有效的值；在写入访问器属性时，会调用setter函数并传入新值，这个函数负责决定如何处理数据。

访问器属性有如下4个特性：

[[Configurable]]:表示能否通过delete删除属性从而重新定义属性，能否修改属性的特性或者能否把属性修改为数据属性。对于直接在对象上定义的属性，这个特性的默认值为true。

[[Enumerable]]:表示能否通过for-in循环返回属性。对于直接在对象上定义的属性，这个特性的默认值为true。

[[Get]]:在读取属性时调用的函数。默认值为undefined。

[[Set]]:在写入属性时调用的函数。默认值为undefined。

访问器属性不能直接定义，必须使用Object.defineProperty()来定义。

示例:

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var book={
    _year:2004,
    edition:1
};

Object.defineProperty(book,'year',{
    get:function(){
        return this._year;
    },
    set:function(newVal){
        if(newVal&gt;2004){
            this._year=newVal;
            this.edition+=newVal-2004;
        }
    }
});
book.year=2005;
book.edition;//2
book._year;//2005
book.year;//2005
</pre>
</div>

### &nbsp;定义多个属性

直接上代码

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var book={};
Object.defineProperties(book,{
  _year:{
    value:2004
  },
  edition:{
    value:1
  },
  year:{
    get:function(){
      return this._year;
    },
    set:function(newVal){
      if(newVal&gt;2004){
        this._year=newVal;
        this.edition+=newVal-2004;
      }
    }
  }
})
</pre>
</div>

Object.defineProperties()方法，可能通过描述符一次定义多个属性。

接收两个对象参数：

*   第一个对象是要添加和修改其属性的对象
*   第二个对象的属性与第一个对象中要添加或修改的属性一一对应

### 读取属性的特性

Object.getOwnPropertyDescriptor()方法，可以取得给定属性的描述符。

接收两个参数：

*   属性所在的对象
*   要读取其描述符的属性名称

返回值是一个对象，视属性的类型不同，对象的属性也不同，具体参见上面的属性类型。

示例

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">var book={};
Object.defineProperties(book,{
  _year:{
    value:2004
  },
  edition:{
    value:1
  },
  year:{
    get:function(){
      return this._year;
    },
    set:function(newVal){
      if(newVal&gt;2004){
        this._year=newVal;
        this.edition+=newVal-2004;
      }
    }
  }
})
var descriptor=Object.getOwnPropertyDescriptor(book,'_year');
descriptor;//Object {value: 2004, writable: false, enumerable: false, configurable: false}
descriptor=Object.getOwnPropertyDescriptor(book,'year');
descriptor;//Object {configurable:false,enumerable:false,get:function(){...},set:function(newVal){...}
</pre>
</div>