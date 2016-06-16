
---
title: 第33条：使构造函数与new操作符无关
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---
当使用函数作为一个构造函数时，程序依赖于调用者是否记得使用new操作符来调用该构造函数。注意：该函数假设接收者是一个全新的对象。
```js
function User(name,pwd){
  this.name=name;
  this.pwd=pwd;
}
```
当调用者，忘记使用new关键字时，那么这个函数的接收者是全局对象。
```js
var u=User('wengxuesong','asdfasdfadf');
u;//undefined
this.name;//'wengxuesong'
this.pwd;//'asdfasdfadf'
```
该函数返回了无意义的undefined，还会修改全局对象。
如果将User函数应用ES5的严格模式，那么它的接收者为undefined
```js
function User(name,pwd){
  'use strict';
  this.name=name;
  this.pwd=pwd;
}
var u=User('wengxuesong','asdfasdfadf');//error:this is undefined
```
上面代码会报错，因为在严格模式下直接运行的函数中的this是undefined，无法给undefined定义属性。
如何才能使User函数在不使用new操作符的情况下，还是按预期工作呢？提供一个不管怎样调用都工作如构造函数的函数。实现该函数的一个简单方法是检查函数的接收者是否是正确的User实例。
```js
function User(name,pwd){
  if(!(this instanceof User)){
     return new User(name,pwd); 
  }
  this.name=name;
  this.pwd=pwd;
}
```
使用这种方式无论如何调用User函数，都会返回一个继承自User.prototype的对象
```js
var x=User('wengxuesong','12123123');
var y=new User('songqiang','sfasdfasdf');
x instanceof User;//true
y instanceof User;//true
```
这个模式的一个缺点是它需要额外的函数调用，因此代价有点高。而且，很难适用于可变参数函数，因为没有一种直接模拟apply方法将可变参数函数作为构造函数调用的方式。
```js
function User(name,pwd){
  var self=this instanceof User?this:Object.create(User.prototype);
  self.name=name;
  self.pwd=pwd;
  return self;
}
```
Object.create需要一个原型对象作为参数，并返回一个继承自该原型对象的新对象。因此，当以函数的方式调用该版本的User函数时，结果将返回一个继承自User.prototype对象的新对象，并且该对象具有已经初始化的name和pwd属性。
Object.create只有在ES5环境中才是有效的，可以通过代码进行兼容。上节已经写过一次了，再写一遍加深印象吧。
```js
if(typeof Object.create === 'undefined'){
    Object.create=function(prototype){
       function F(){};
       F.prototype=prototype;
       return new F();
    }
}
```
注意：这里只实现了单参数的Object.create函数。真实版本的Object.create函数还接受一个可选的参数，该参数描述了一组定义在新对象上的属性描述符。
如果使用new操作符调用新版本的User参数会发生什么？
构造函数覆盖模式，使用new操作符调用该函数的行为就如以函数调用它的行为一样。这能工作完全利益于js允许new表达式的结果可以被构造函数中的显式return语句所覆盖。当User函数返回self对象时，new表达式的结果就变为self对象。该self对象可能是另一个绑定到this的对象。
防范误用构造函数可能并不是太值得去做，尤其是当仅仅是局部使用构造函数时。但是理解如果以错误的方式调用构造函数会造成严重后果很重要。至少文档化构造函数期望使用new操作符调用是很重要的，尤其是在跨大型代码库中其享构造函数或该构造函数来自一个共享库时。

## 提示
- 通过使用new操作符或Object.create方法在构造函数定义中调用自身使得该构造函数与调用语法无关
- 当一个函数期望使用new操作符调用时，清晰地文档化该函数

## 附录一：Object.create方法
以下内容来自Mozilla 开发者社区
Object.create()方法创建一个拥有指定原型和若干指定属性的对象。
### 语法
Object.create(原型对象,[属性对象集])
### 异常
如果原型对象不是null或一个对象值，则抛出一个TypeError异常
### 实现类式继承
#### 单继承
```js
function Shape(){
  this.x=0;
  this.y=0;
}
Shape.prototype.move=function(x,y){
  this.x+=x;
  this.y+=y;
  console.info('Shape moved.');
};

function Rectangle(){
  Shape.call(this);//构造函数借用
}
Rectangle.prototype=Object.create(Shape.prototype);

var rect=new Rectangle();
rect instanceof Rectangle;//true
rect instanceof Shape;//true
rect.move(1,1);//"Shape moved"
```
画张图表示一下上面的关系
![继承关系][1]
#### 多继承
```js
function MyClass(){
  SuperClass.call(this);
  OtherSuperClass.call(this);
}
MyClass.prototype=Object.create(SuperClass.prototype);
mixin(MyClass.prototype,OtherSuperClass.prototype);//mixin
MyClass.prototype.myMethod=function(){
  
};
```
#### propertyObject参数
```js
var o;
//创建原型为null的空对象
o=Object.create(null);

o={};
//以字面量方式创建空对象相当于下面这句代码
o=Object.create(Object.prototype)


o=Object.create(Object.prototype,{
  //创建对象的数据属性
  foo:{writable:true,configurable:true,value:'hello'},
  //创建对象的访问器属性
  bar:{
     configurable:false,
     get:function(){return 10;},
     set:function(val){console.log('setting "o.bar" to',val);}
  }
});

function Constructor(){}
o=new Constructor();
//相当于
o=Object.create(Constructor.prototype);
//如果Constructor里有一些初始化代码，Object.create不能执行那些代码

//应该相当于
o={};
o=Object.create(Constructor.prototype);
Constructor.call(o,args);

//创建一个以另一个空对象为原型，且拥有一个属性p的对象
o=Object.create({},{p:{value:42}});

//省略了的属性默认为false，所以属性p是不可写，不可枚举，不可配置的
o.p=24;
o.p;//42
o.q=12;
for(var prop in o){
  console.log(prop);
}
//'q'
delete o.p;//false

//创建一个可写的，可枚举的，可配置的属性p
o2=Object.create({},{p:{value:42,writable:true,enumerable:true,configurable:true}});
```
### 兼容版
```js
if(typeof Object.create !== 'function'){
  Object.create=(function(){
     function NOP(){}
     var hasOwn=Object.prototype.hasOwnProperty;
     return function(o){
       //1、如果o不是Object或null，抛出一个类型错误异常
       if(typeof o!=='object'){
          throw TypeError('Object prototype may only be an Object or null.');
       }
       //2、使创建的一个新的对象为obj
       //3、设置obj的内部属性[[Prototype]]为o
       NOP.prototype=o;
       var obj=new NOP();
       NOP.prototype=null;//解除NOP函数的prototype的引用
       //4、如果参数有Properties，那么检测并添加属性到obj上。
       if(arguments.length>1){
          var Properties=Object(arguments[1]);
          for(var prop in Properties){
             if(hasOwn.call(Properties,prop)){
                obj[prop]=Properties[prop];
             }
          }
       }
       return obj;
     }
  })();
}
```


  [1]: http://images2015.cnblogs.com/blog/156514/201606/156514-20160614101956245-553595553.jpg "1465870265977.jpg"