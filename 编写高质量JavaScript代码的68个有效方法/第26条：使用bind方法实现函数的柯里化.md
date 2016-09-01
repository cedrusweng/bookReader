---
title: 第26条：使用bind方法实现函数的柯里化
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---

bind方法的作用，除了有绑定函数到对象外，我们来看看bind方法的一些其它应用。

## 简单示例

例子：假设有一个装配URL字符串的简单函数。代码如下

<pre class="brush:javascript;gutter:true;">function simpleURL(protocol,domain,path){
   return protocol+'://'+domain+'/'+path;
}
</pre>

要将特定站点的路径字符串构建为绝对路径URL。可以使用ES5中数组的map方法来实现。如下

<pre class="brush:javascript;gutter:true;">var paths=['wengxuesong/','wengxuesong/p/5560484.html','wengxuesong/p/5555714.html'];
var urls=paths.map(function(path){
   return simpleURL('http','www.cnblogs.com',path);
});
urls;//["http://www.cnblogs.com/wengxuesong/", "http://www.cnblogs.com/wengxuesong/p/5560484.html", "http://www.cnblogs.com/wengxuesong/p/5555714.html"]</pre>

这里功能的实现完成正确。
但有没有可以改进的地方呢？里面每一次的迭代都使用相同的协议字符串和网站域名字符串。传给simpleURL函数的前两个参数对于迭代都是固定的，仅第三个参数是变化的。可以通过bind方法来生成一个已经包含两个参数的匿名函数。代码如下：

<pre class="brush:javascript;gutter:true;">var urls=paths.map(simpleURL.bind(null,'http','www.cnblogs.com'));
</pre>

对于simpleURL.bind的调用产生了一个委托到simpleURL的新函数。bind方法的第一个参数提供了接收者的值。这里因为不需要，所以使用null或undefined都可以。simpleURL.bind的其余参数和提供给新函数的所有参数共同组成了传递给simpleURL的参数。
以上代码可以拆分得更细些，代码如下:

<pre class="brush:javascript;gutter:true;">var newSimpleURL=simpleURL.bind(null,'http','www.cnblogs.com');
var urls=paths.map(newSimpleURL);
</pre>

不使用bind方法也是可以实现函数的，下面的一个版本是使用简单的函数实现,代码如下

<pre class="brush:javascript;gutter:true;">var noBindSimpleURL=function(path){
    return simpleURL('http','www.cnblogs.com',path);
}
var urls=paths.map(noBindSimpleURL);
</pre>

这里对比一下，bind方法提供了一种更简单的实现，结构更简单，但对于不了解bind方法的人理解起来可能有一点难度。

## 函数柯里化

将函数与其参数的一个子集绑定的技术称为函数柯里化(curring)，以逻辑学家Haskell curry的名字命名。
比起显式的封闭函数，函数柯里化是一种简洁的、使用更少引用来实现函数委托的技术。

### 个人理解

函数柯里化，只是对参数的处理，也就是简化参数。简图如下

[![1465181669874](http://images2015.cnblogs.com/blog/156514/201606/156514-20160606105632715-381031132.jpg "1465181669874")](http://images2015.cnblogs.com/blog/156514/201606/156514-20160606105632105-1950030231.jpg)

## 提示

*   使用bind方法实现函数柯里化，即创建一个固定需求参数子集的委托函数

*   传入null或undefined作为接收者的参数来实现函数柯里化，从而忽略其接收者