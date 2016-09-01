---
title: 第23条：永远不要修改arguments对象
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---

<p>arguments对象并不是标准的Array类型的实例。arguments对象不能直接调用Array方法。</p>
<h2>arguments对象的救星call方法</h2>
<p>使得arguments可以品尝到数组方法的美味，知道可以吃，下面就是怎么吃的问题了。不管怎么吃，先吃一口试试。</p>
<pre class="brush:javascript;gutter:true;">function callMethod(obj,method){
    var shift=[].shift;
    shift.call(arguments);
    shift.call(arguments);
    return obj[method].apply(obj,argumetns);
}
</pre>
<p>感觉很棒的样子，色香都具备了，拿筷子尝一下吧。</p>
<pre class="brush:javascript;gutter:true;">var obj={
    add:function(x,y){return x+y;}
};
callMethod(obj,'add',17,25);//这里应该是42，不运行我都知道</pre>
<p>放到chrome控制台运行一下吧，哇，好鲜艳，有红色。<br /><a>//Uncaught</a> TypeError: Cannot read property 'apply' of undefined(&hellip;)<br />为什么会出错呢，回去找找菜谱（高3），看哪里不对。原来arguments对象并不是函数参数的副本。所有命名参数都是arguments对象中对应索引的别名。上节好像也有，<a href="http://www.cnblogs.com/wengxuesong/p/5549553.html#wxs-h-7">回去看一下</a>，下面列出一个arguments对象索引对应命名参数的列表</p>
<pre class="brush:javascript;gutter:true;">arguments对象：
索引   形参    实参
0     obj     obj
1     method  'add'
2             17
3             25   
</pre>
<p>看着这个可以想一下，这个材料（arguments对象），经过加工(两次shift)。<br />arguments对象变成下面这个样子了</p>
<pre class="brush:javascript;gutter:true;">arguments对象：
索引   形参    实参
0     obj     17
1     method  25 
</pre>
<p>此时我们吃到得已经是变了味的东西了，唉，这个味还能和预期一样嘛。不出错才怪，这个糟糕的过程在JS内部这样执行。这时的obj['add']变成了17[25]。<br />17被转化为Number对象，然后查找属性"25",很明显不存在，得到undefined值，然后查找undefined的'apply'属性并将它当方法调用,最后我们的色香被味扼杀了。</p>
<p>这次的失败，我们可以总结出，arguments对象和命名参数的关系是脆弱的。修改arguments对象也要小心命名参数的改变。ES5下情况更复杂。严格模式，函数参数不支持对其arguments对象取别名。到底如何，写代码看一下就知道了。代码如下</p>
<pre class="brush:javascript;gutter:true;">function strict(x){
   "use strict";
   arguments[0]='我说还不行嘛';
   return x===arguments[0];
}
function nostrict(x){
   arguments[0]='我说还不行嘛';
   return x===arguments[0];
}
strict('我就不说');//false
nostrict('我就不说');//true</pre>
<p>所以，不管说不说，我们还是不要动arguments对象了。惹不起我还躲不起嘛，我自己复制一份参数中的元素到新数组中，这样可以吧。</p>
<pre class="brush:javascript;gutter:true;">var args=[].slice.call(arguments);
</pre>
<p>这是什么，很牛的样子。小朋友，我给你讲讲哈。当不使用额外的参数调用数组的slice方法时，它会复制整个数组，结果是一个真正的男人（数组）。要不我们来试试复制个数组试试。</p>
<pre class="brush:javascript;gutter:true;">var a=[1,2,3,4,5,6,7];
var b=a;
var c=a.slice();
a.shift();
b;//[2, 3, 4, 5, 6, 7]
c;//[1, 2, 3, 4, 5, 6, 7]</pre>
<p>看到没要真正独立还是要复制一下的，要不会被影响的。<br />扯远了，回到最初的那个问题，动手修正callMethod函数，把"味"补上。</p>
<pre class="brush:javascript;gutter:true;">function callMethod(obj,method){
    var args=[].slice.call(arguments,2);
    return obj[method].apply(obj,args);
}
</pre>
<p>运行一下</p>
<pre class="brush:javascript;gutter:true;">var obj={
    add:function(x,y){return x+y;}
};
callMethod(obj,'add',17,25);//42</pre>
<p>对就是这个味（结果）。回味一下，这个过程要注意哪些。下面是书里给的提示。</p>
<h2>提示</h2>
<ul>
<li>
<p>永远不要修改arguments对象</p>
</li>
<li>
<p>使用[].slice.call(arguments)将arguments对象复制到一个真正的数组中再进行修改</p>
</li>
</ul>
<h2>附录一：Array队列和操作方法</h2>
<h3>队列方法</h3>
<h3>shift()方法</h3>
<p>移除数组中的第一个项并返回该项，同时将数组长度减1。</p>
<pre class="brush:javascript;gutter:true;">var a=[1,2,3];
var b=a.shift();
b;//1
a;//[2,3]</pre>
<h3>push()方法</h3>
<p>接收任意数量的参数，把它们逐个添加到数组末尾，并返回修改后数组的长度。</p>
<pre class="brush:javascript;gutter:true;">var a=[1,2,3];
var b=a.push(0);
var c=a.push(4,5,7);
var m=a.push([9,0])
a;//[1, 2, 3, 0, 4, 5, 7, [9,0]]
b;//4
c;//7
m;//8</pre>
<h3>操作方法</h3>
<h3>concat()方法</h3>
<p>基于当前数组中的所有项创建一个新数组。具体来说，先创建一个数组的副本，然后将接收的参数添加到这个副本的尾部，最后返回新构建的数组。<br />不传参可以当复制数组来使用</p>
<pre class="brush:javascript;gutter:true;">var a=[2,3,4];
var b=a.concat();
a.push(2,1);
a;//[2,3,4,2,1]
b;//[2,3,4]</pre>
<p>传参是一个或多个数组，将这些数组的第一项复制到新数组后，如果不是数组则简单添加。</p>
<pre class="brush:javascript;gutter:true;">var a=[1,2,3,4,6];
var b=[1,[2,3],4];
var c="good";
var d=a.concat(b,c);
d;//[1, 2, 3, 4, 6, 1, [2,3], 4, "good"]</pre>
<p>上面的代码说明数组复制，不进行递归。</p>
<h3>slice()方法</h3>
<p>可以基于当前数组中的一或多个项创建一个新数组。(不影响原始数组)</p>
<h4>不传参可以用来复制数组</h4>
<pre class="brush:javascript;gutter:true;">var a=[2,3,4];
var b=a.slice();
a.push(2,1);
a;//[2,3,4,2,1]
b;//[2,3,4]</pre>
<h4>传一个或两个参数时</h4>
<pre class="brush:javascript;gutter:true;">var a=[2,3,4];
var b=a.slice(1);
var c=a.slice(1,2);
a;//[2,3,4]
b;//[3,4]
c;//[3]</pre>
<p><strong><span style="color: #ff0000;">注意：</span></strong></p>
<ul>
<li>
<p>参数为索引位置，范围为[arg1,arg2),产生的新数组包括arg1位置项，不包括arg2位置的项。</p>
</li>
<li>
<p>参数可以为负数，当为负数时，对应的索引位置为数组长度加上该数来确定。</p>
</li>
</ul>
<h3>splice()方法</h3>
<p>最强大的数组方法，主要用途是向数组中部插入项，有3种使用方式：</p>
<ul>
<li>
<p><strong>删除：</strong>可以删除任意数量的项，只需指定2个参数：要删除的第一项的位置和要删除的项数。<br />splice(0,2);删除数组中的前两项。范围为[arg1,arg1+arg2)</p>





</li>
<li>
<p><strong>插入:</strong>可以指定位置插入任意数量的项，只需要提供3个参数：起始位置，0，和要插入的项。<br />splice(2,0,'good','morning');会从当前数组的位置2开始插入"godd","morning";</p>





</li>
<li>
<p><strong>替换:</strong>可以向指定位置插入任意数量的项，且同时删除任意数量的项，只需要指定3个参数：起始位置，要删除的项数和要插入的任意数量的项。splice(2,1,'good','morning');删除位置2项并添加"godd","morning"</p>





</li>





</ul>
<p>splice()方法返回一个数组，该数组中包含从原始数组中删除的项。<span style="color: #ff0000;">注意：会影响原数组。</span></p>