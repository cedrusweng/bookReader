---
title: 第54条：将undefined看做“没有值”
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---
undefined值很特殊，每当js无法提供具体的值时，就会产生undefined。
## undefined值场景
#### 未赋值的变量的初始值即为undefined。
```js
var x;
x;//undefined
```
#### 访问对象不存在的属性也会产生undefined。
```js
var obj={};
obj.x;//undefined
```
#### 一个函数体结尾使用未带参数的return语句，或未使用return语句都会返回值undefined。
```js
function f(){
    return;
}
function g(){}
f();//undefined
g();//undefined
```
#### 未给函数参数提供实参则该参数的值为undefined。
```js
function f(x){
    return x;
}
f();//undefined
```
这几种情况下undefined值表明操作结果并不是一个特定的值。或说叫做“没有值”，但undefined是一个值，很自相矛盾。因为js里的每个操作都要产生点什么，所以就用undefined来填补这些空白了。
## undefined公约
将undefined看做缺少某个特定值是js里的公约。将它用于其他的目的，可能会产生严重的问题。例如，一个用户界面元素库可能支持一个highlight方法用于改变一个元素的背景颜色。
```js
element.highlight();//使用默认的颜色
element.highlight('yellow');//使用黄色
```
如果想提供一种方法来设置一个随机颜色，可能会使用undefined作为特殊的值来实现。
```js
element.highlight(undefined);//使用随机颜色
```
但上面的代码会产生一个歧义，因为undefined在不传入实参的时候，形参的值也是undefined。所以程序无法在默认颜色，还是随机颜色之间做决定。程序产生了二义性无法正确运行。
再比如，程序可能用一个可选的颜色偏好的配置对象。
```js
var config=JSON.parse(preferences);
//...
element.highlight(config.highlightColor);
```
如果config.highlightColor并没有设定，那么它的值也会是undefined。这个时候程序员可能希望得到一个默认的颜色。但由于上面的undefined被做为一个特殊的值，这样像上面说的产生了二义性，产生了一个随机的颜色值。更好的方式是对可能会使用一种特殊的颜色名来实现随机颜色。
```js
element.highlight('random');
```
有时一个API不能够从通常函数可接受的字符串集合中区分出一个特殊的字符串值。在这种情况下，可以使用除undefined以外的其他特殊值，如null或true。这往往会导致可读性下降。
```js
element.highlight(null);
```
如果阅读代码的人没有记住这个函数的具体使用方法，那么这样的代码很难理解。可能会给人以取消元素高亮的误解。一个更具明确、更具描述性的可选做法是将随机情况表示为一个具有random属性的对象
```js
element.highlight({random:true});
```
## 可选参数的实现
undefined有可能出问题的地方是可选参数的实现。理论上arguments对象可检测是否传递了一个参数，但实际上，测试是否为undefined会使API更健壮。例如，一个Web服务器可以接收一个可选的主机名称。
```js
var s1=new Server(80,'example.com');
var s2=new Server(80);//默认为localhost
```
### 参数长度
可以通过判断arguments.length来实现Server构造函数。
```js
function Server(port,hostname){
    if(arguments.length<2){
        hostname='localhost';
    }
    hostname=''+hostname;
}
```
上面的这个实现也是会产生问题，当我第二个参数显式传递从其他源请求的一个值。那么可能变成传递的是一个undefined值，但这个时候arguments.length的值是2,代码被破坏。
```js
var s3=new Server(80,config.hostname);
```
上面代码如果config.hostname没有值，hostname的正常行为应该是设置为'localhost'。上面的代码会产生一个undefined的主机名。
### 检测参数是否为undefined
最好是测试undefined，不传递参数和传递一个表达式结果是undefined的情况都可以覆盖。
```js
function Server(port,hostname){
    if(hostname === undefined){
        hostname='localhost';
    }
    hostname=''+hostname;
    //...
}
```
### 测试参数是否为真
另一种替代方案是测试hostname是否为真。使用逻辑运算符很好实现
```js
function Server(port,hostname){    
    hostname=''+(hostname||'localhost');
    //...
}
```
|| 逻辑运算符或。当第一个参数为真值则会返回第一个参数，否则返回第二个参数。所以hostname值为undefined或空字符串，该表达式(hostname||'localhost')的结果是'localhost'。这里把hostname的值测试了很多种情况，包括其他能转化为false的值。这里对于Server函数是可以的。真值测试是实现参数默认值的一种简明的方式。
### 真值测试
真值测试并不是总是安全的。如果一个函数应该接收空字符串为合法值，真值测试将覆盖空字符串并返回默认值。还有其它的可以转化为false的值，则不应该使用真值测试。例如，一个用于创建用户界面元素的函数可能允许一个元素的宽度或高度为0，但提供默认值却不一样。
```js
var c1=new Element(0,0);//width:0 height:0
var c2=new Element();//width:320 height:240
```
使用真值测试的实现会出现问题
```js
function Element(w,h){
    this.width=w||320;
    this.height=h||240;
}
var c1=new Element(0,0);
c1.width;//320
c1.height;//240
```
这里就不能使用真值而应该使用undefined检测
```js
function Element(w,h){
    this.width=w===undefined?320:w;
    this.height=h===undefined?240:h;
}
var c1=new Element(0,0);
c1.width;//320
c1.height;//240
var c2=new Element();
c2.width;//320
c2.height;//240
```

## 提示
- 避免使用undefined表示任何非特定值
- 使用描述性的字符串值或命名布尔属性的对象，而不要使用undefined或null来代表特定应用标志
- 提供参数默认值应当采用测试undefined的方式，而不是检查arguments.length
- 在允许0、NaN或空字符串为有效参数的地方，绝不要通过真值测试来实现参数默认值

## 相关阅读
- 第3条：当心隐式的强制转换


