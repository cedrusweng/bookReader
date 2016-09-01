---
title: 第24条：使用变量保存arguments对象
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---

迭代器（iterator）是一个可以顺序存取数据集合的对象。其一个典型的API是next方法。该方法获得序列中的下一个值。

## 迭代器示例

###### <a name="e9a298e79baeefbc9ae5b88ce69c9be7bc96e58699e4b880e4b8aae4bebfe588a9e79a84e587bde695b0efbc8ce5ae83e58fafe4bba5e68ea5e694b6e4bbbbe6848fe695b0e9878fe79a84e58f82e695b0efbc8ce5b9b6e4b8bae8bf99e4ba9be580bce5bbbae7ab8be4b880e4b8aae8bfade4bba3e599a8e38082"></a>题目：希望编写一个便利的函数，它可以接收任意数量的参数，并为这些值建立一个迭代器。

测试代码好下：

<pre class="brush:javascript;gutter:true;">var it=values(1,4,1,4,2,1,3,5,6);
it.next();//1
it.next();//4
it.next();//1</pre>

###### <a name="e58886e69e90efbc9ae794b1e4ba8evaluese587bde695b0e99c80e8a681e68ea5e694b6e4bbbbe6848fe5a49ae4b8aae58f82e695b0efbc8ce8bf99e9878ce5b0b1e99c80e8a681e794a8e588b0e4b88ae4b880e88a82e8aeb2e588b0e79a84e69e84e5bbbae58fafe58f98e58f82e695b0e79a84e587bde695b0e79a84e696b9e6b395e38082e784b6e5908ee9878ce99da2e79a84e8bfade4bba3e599a8e5afb9e8b1a1e69da5e9818de58e86argumentse5afb9e8b1a1e79a84e58583e7b4a0e38082"></a>分析：由于values函数需要接收任意多个参数，这里就需要用到上一节讲到的构建可变参数的函数的方法。然后里面的迭代器对象来遍历arguments对象的元素。

### 初步编码

<pre class="brush:javascript;gutter:true;">function values(){
    var i=0,n=arguments.length;
    return {
        hasNext:function(){
            return i&lt;n;
        },
        next:function(){
            if(this.hasNext()){
                return arguments[i++];
            }
            throw new Error("已经到达最后啦");
        }
    }
}
</pre>

用上面的测试代码进行测试

<pre class="brush:javascript;gutter:true;">var it=values(1,4,1,4,2,1,3,5,6);
it.next();//undefined
it.next();//undefined
it.next();//undefined</pre>

### 错误分析

代码运行结果并不正确，下面就对初始的编码程序进行分析。

<pre class="brush:javascript;gutter:true;">function values(){
    var i=0,n=arguments.length;//这里没有错误，arguments是values里的内置对象
    return {
        hasNext:function(){
            return i&lt;n;
        },
        next:function(){
            if(this.hasNext()){
                return arguments[i++];//错误出现在这里，arguments是next方法函数的内置对象。
            }
            throw new Error("已经到达最后啦");
        }
    }
}
</pre>

这里的指代错误，很像是另一个让人头痛的对象this。处理this的指向时，通常是使用变量和保存正确的this。然后在其它地方使用这个变量。那么arguments对象的解决方案就出来了，借助一个变量来存储，这样arguments对象的指代就没有问题了。

### 再次编码

<pre class="brush:javascript;gutter:true;">function values(){
    var i=0,n=arguments.length,arg=arguments;
    return {
        hasNext:function(){
            return i&lt;n;
        },
        next:function(){
            if(this.hasNext()){
                return arg[i++];
            }
            throw new Error("已经到达最后啦");
        }
    }
}
</pre>

运行测试代码

<pre class="brush:javascript;gutter:true;">var it=values(1,4,1,4,2,1,3,5,6);
it.next();//1
it.next();//4
it.next();//1</pre>

结果和预期的相同。

## 提示

*   当引用arguments时当心函数嵌套层级

*   绑定一个明确作用域的引用到arguments变量，从而可以在嵌套的函数中引用它

## 附录一：迭代器

迭代器（iterator）有时又称游标(cursor)是程序设计的软件设计模式，可在容器上遍历的接口，设计人员无需关心容器的内容。

### 迭代器UML类图

[![1464858512522](http://images2015.cnblogs.com/blog/156514/201606/156514-20160602171633805-876304017.jpg "1464858512522")](http://images2015.cnblogs.com/blog/156514/201606/156514-20160602171633024-1904900423.jpg)

### 迭代器js实现

对设计模式了解一点点，但具体项目中，有得多的也就是工厂模式，其它很少用，下面是一个简单的实现，不对的地方，欢迎交流。
代码如下

<pre class="brush:javascript;gutter:true;">function List(){
    this.data=[];
}
List.prototype={
    add:function(){
        var args=[].slice.call(arguments)
        this.data=this.data.concat(args);       
    },
    remove:function(i){
        this.data.splice(i,1);
    },
    iterator:function(){
        return new Iterator(this);
    }
}

function Iterator(list){
    this.list=list;
    this.cur=0;
};
Iterator.prototype={
    hasNext:function(){
        return this.cur&lt;this.list.data.length-1;
    },
    next:function(){
        if(this.hasNext()){
            return this.list.data[this.cur++];
        }
        throw new Error('已经到底了~');
    },
    remove:function(){
        this.list.remove(this.cur);
    }
}

var list1=new List();
var it=list1.iterator();
list1.add(3,2,3,4,1,6,10,8,9);
it.next();//3
it.next();//2
it.next();//3
</pre></div>