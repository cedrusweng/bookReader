---
title: 第25条：使用bind方法提取具有确定接收者的方法
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---

js里方法和属性值为函数，就像一个东西两种称呼一个样，比如土豆，也叫马铃薯，一个样。既然一样，那就可以对对象的方法提取出来为函数，然后把提取出来的函数作为回调函数直接传递给高阶函数。

## 高阶函数是什么

玩过套娃娃游戏没，没玩过，没事，我也没玩过。
大致就是下面这个样子
[![1464860371895](http://images2015.cnblogs.com/blog/156514/201606/156514-20160603113242696-998024598.jpg "1464860371895")](http://images2015.cnblogs.com/blog/156514/201606/156514-20160603113242071-1285869396.jpg)
呃，好吧，这才是真正的。
[![1464860406270](http://images2015.cnblogs.com/blog/156514/201606/156514-20160603113244055-386931397.jpg "1464860406270")](http://images2015.cnblogs.com/blog/156514/201606/156514-20160603113243258-1871497500.jpg)
就是多层函数，以函数为参数或返回值的函数。有点绕，没事看看上面的图就明白了。想了解怎么实现个简单的[请点这里](http://www.cnblogs.com/wengxuesong/p/5531242.html)。
好了，函数拿出来了，给高阶函数做参数传进去了。
这里面很容易会忘记把传进去的函数绑定到当前对象上，自由惯了，没办法，自由很重要，但没有绑定的对象，你什么都做不了。比如没有女朋友，你就得靠双手了，伤身呀。

## 一个示例

来看看下面这段代码提提神。

<pre class="brush:javascript;gutter:true;">var you={
    gF:[],
    add:function(s){
        this.gF.push(s);
    },
    all:function(){
        return this.gF.join("-");
    }
}
</pre>

比如你不希望一次只交一个女朋友，想同时交往多个。ES5的forEach方法，在每个源数组（多个女朋友）元素上重复地调用add方法（交往），就可以把多个女孩子加到你的后宫了。

<pre class="brush:javascript;gutter:true;">var girls=["西施","王昭君","貂蝉","杨贵妃"];//按人物历史出场顺序,呵呵
girls.forEach(you.add);
</pre>

想想看着四大美人，来出来一见。

<pre class="brush:javascript;gutter:true;">you.all();//error:Cannot read property 'push' of undefined</pre>

什么都有人呢，都哪去了。
因为you.add的接收者并不是你(you对象)。函数的接收者取决于它是如何被调用的，上面并没有调用它，只是把它传给了高阶函数forEach。但forEach的实现使用全局对象，这个时候你加女朋友变成了，在全世界(window对象)里找gF属性，因为这个gF没有为undefined，所以也就没有push方法，这就报错了。
问题找到了，美女们还往哪里跑~

<pre class="brush:javascript;gutter:true;">var girls=["西施","王昭君","貂蝉","杨贵妃"];//按人物历史出场顺序,呵呵
girls.forEach(you.add,you);
you.all();//"西施,王昭君,貂蝉,杨贵妃"</pre>

完美运行，是不是想想都美了。

## 匿名函数

但不是每个高阶函数都像forEach这样善解人意，提供一个其回调函数的接收者。如果遇到个不解风情的函数怎么办？我们可以创建一个调用对象方法的函数来运行。法子如下

<pre class="brush:javascript;gutter:true;">var girls=["西施","王昭君","貂蝉","杨贵妃"];//按人物历史出场顺序,呵呵
girls.forEach(function(s){
    you.add(s);
});
you.all();//"西施,王昭君,貂蝉,杨贵妃"</pre>

好了，事情就这样解决了。

## bind方法

NO,还有东西要讲，现在请bind方法全场，它是在ES5标准库才有的函数方法。主要作用就是为函数指定一个对象为其接收者，你也可以理解为，就是指定函数里的this指向哪个对象的。现在就看看上面的情况，bind方法如何解决：

<pre class="brush:javascript;gutter:true;">var girls=["西施","王昭君","貂蝉","杨贵妃"];//按人物历史出场顺序,呵呵
girls.forEach(you.add.bind(you));
you.all();//"西施,王昭君,貂蝉,杨贵妃"</pre>

怎么样，是不是很牛掰。那bind都弄了啥呢？这里you.add.bind(you)生成了一个全新的函数，这个函数行为和you.add是一致的，但它里把this定死了，死心踏地地就是you了，原来的函数you.add的接收者保持不变。

<pre class="brush:javascript;gutter:true;">you.add===you.add.bind(you);//false</pre>

这样就可以在很多的场合共享函数了。特别是里面有this关键词的原型方法。只要使用bind就再也不用担心this的指向问题了。不用在外层作用域用变量存储this了。

<pre class="brush:javascript;gutter:true;">var obj={
    txt:'hello',
    sayHello:function(){
       console.log(this.txt);
    }
}
document.body.onclick=obj.syaHello.bind(obj);//"hello"
</pre>

事件绑定再也不用担心了。
bind方法在ES5之前需要兼容写法，[详细请点击查看](http://www.cnblogs.com/wengxuesong/p/5545281.html#wxs-h-11)。
下面就又到了书上的提示内容了，记住一小点受用很多噢。

## 提示

*   注意，提取一个方法不会将方法的接收者绑定到该方法的对象上

*   当给高阶函数传递对象方法时，使用匿名函数在适当的接收者上调用该方法

*   使用bind方法创建绑定到适当接收者的函数

**附录：这次没有附录，本节里的内容，在之前都有，没有引进新的知识点。**