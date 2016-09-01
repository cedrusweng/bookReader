with特性，提供的任何&ldquo;便利&rdquo;都更让其变得不可靠和低效率。

with语句的用法，可以很方便地避免对对象的重复引用。上面的代码整理成下面的形式

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">function status(info){

      var widget=new Widget();

      with(widget){

             setBackground(&lsquo;blue&rsquo;);

             setForeground(&lsquo;white&rsquo;);

             setText(&lsquo;Status:&rsquo;+info);

            show()

       }

}
</pre>
</div>

使用with语句从模块对象中导入变量也很有诱惑力。

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">function f(x,y){

      with(Math){

            return min(round(x),sqrt(y));

       }

}
</pre>
</div>

在上面的这两种情况，使用with使得提取对象的属性，并将这些属性绑定到块的局部变量中变得非常诱人且容易。

很有吸引力，但没做它们应该做的事。

**这两个例子里有两种不同类型的变量。**

*   一种是希望引用with对象的属性的变量，如setBackground,round。
*   另一种是希望引用外部变量绑定的变量，如info,x,y。
*   语法上并没有区分这两种类型的变量，都只是看起来像变量。

## js中所有变量都是相同的

js从最内层的作用域开始向外查找变量。with语句对待一个对象犹如该对象代表一个变量作用域，因此在with代码块的内部，变量查找从搜索给定的变量名的属性开始。如果在这个对象中没有找到该属性，则继续在外部作用域中搜索。

[![image](http://images2015.cnblogs.com/blog/156514/201605/156514-20160516164825138-1788700042.png "image")](http://images2015.cnblogs.com/blog/156514/201605/156514-20160516164824263-983721473.png)

### 在ES5规范中这称为词法环境（在旧版本标准中称为作用域链）。

**该词法环境的最内层作用域由widget对象提供。接下来的作用域用来绑定该函数的局部变量info和widget。接下来一层绑定到status函数。注意在一个正常的作用域中，会有与局部作用域中的变量同样多的作用域绑定存储在与之对应的环境层级中。但是对于with作用域，绑定集合依赖于碰巧在给定时间点时的对象。**

**with块中的每个外部变量的引用都隐式地假设在with对象（以及它的任何原型对象）中没有同名的属性。而在程序的其他地方创建或修改with对象或其原型对象不一定会遵循这样的假设**。js引擎当然也不会读取局部代码来获取你使用了哪些局部变量。

**变量作用域和对象命名空间之间的冲突使得with代码块异常脆弱**。如：with对象获得了一个名为info的属性，那么status函数的行为就会被立即改变。status函数将使用这个属性而不是info参数。在源代码的功能进行扩展时，后面有可能决定所有的widget对象都应该有一个info属性。在原型对象上添加了info属性。这将使status函数变得不可预测。

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">status(&lsquo;connecting&rsquo;);//Status:connecting

Widget.prototype.info=&rdquo;[widget info]&rdquo;;

status(&lsquo;connected&rsquo;);//Status:[widget info]
</pre>
</div>

同样的，如果某人添加名为x或y的属性到Math对象上，那么f函数也悲剧了。

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">Math.x=0;

Math.y=0;

f(2,9);//0
</pre>
</div>

总是很难预测一个特定的对象是否已被修改，或是否可能拥有你不知道的属性。事实证明，人力不可预测的特性对于优化编译器同样不可预测。

通常情况下，js作用域可被表示为高效的内部数据结构，变量查找会非常快速。但**with代码块需要搜索对象的原型链来查找with代码块里的所有变量**，因此，**其运行速度远远低于一般的代码块**。

在js中没有单个特性能作为一个更好的选择直接替代with语句。在某些情况下，最好的替代方法是**简单地将对象绑定到一个简短的变量名上**。

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">function status(info){

    var w=new Widget();

    w.setBackground(&lsquo;blue&rsquo;);

    w.setForebacground(&lsquo;white&rsquo;);

    w.addText(&lsquo;Status:&rsquo;+info);

    w.show();

}
</pre>
</div>

没有任何变量对于w对象的内容是敏感的。所以即使一些代码修改了Widget的原型对象，status函数的行为依旧是可预期的。

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">status(&lsquo;connecting&rsquo;);//Status:connecting

Widget.prototype.info=&rdquo;[widget info]&rdquo;;

status(&lsquo;connected&rsquo;);//Status:connected
</pre>
</div>

在其他情况下，**最好的方法是显示地将局部变量显式地绑定到相关的属性上**。

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">function f(x,y){

     var min=Math.min,round=Math.round,sqrt=Math.sqrt;

    return min(round(x),sqrt(y));

}
</pre>
</div>

再次一旦消除with语句，函数的行为变得可以预测了。

<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">Math.x=0;

Math.y=0;

f(2,9);//2
</pre>
</div>

## 提示

*   避免使用with语句
*   使用简短的变量名代替重复访问的对象
*   显式地绑定局部变量到对象属性上