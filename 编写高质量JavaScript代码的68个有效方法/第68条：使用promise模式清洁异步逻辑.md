---
title: 第68条：使用promise模式清洁异步逻辑
tags: 编写高质量javascript代码的68条有效方法
grammar_cjkRuby: true
---
构建异步API的一种流行的替代方式是使用promise(有时也被称为deferred或future)模式。已经在本章讨论过的异步API使用回调函数作为参数。
```js
downloadAsync('file.txt',function(file){
  console.log('file:'+file);
});
```
基于promise的API不接收回调函数作为参数。相反，它返回一个promise对象，该对象通过其自身的then方法接收回调函数。
```js
var p=downloadP('file.txt');
p.then(function(file){
  console.log('file: '+file);
});
```
这里看不出与原先的版本有什么不同。但是promise的力量在于它们的组合性。传递给then方法的回调函数不仅产生影响，也可以产生结果。通过回调函数返回一个值，可以构造一个新的promise。
```js
var fileP=downloadP('file.txt');
var lengthP=fileP.then(function(file){
  return file.length;
});
lengthP.then(function(length){
  console.log('length: '+length);
});
```
理解promise的一种方法是将它理解为表示最终值的对象。它封装了一个还未完成的并发操作，但最终会产生一个结果值。then方法允许我们提供一个代表最终值的一种类型的promise对象，并产生一个新的promise对象来代表最终值的另一种类型，而不管回调函数返回了什么。
从现有的promise中构造新promise的能力带来了很大的灵活性，并且具有一些简单但强大的惯用法。例如，构造一个实用程序来拼接多个promise的结果。
```js
var filesP=join(downloadP('file1.txt'),
               downloadP('file2.txt'),
               downloadP('file3.txt'));
filesP.then(function(files){
  console.log('file1:'+files[0]);
  console.log('file2:'+files[1]);
  console.log('file3:'+files[2]);
});
```
promise库也经常提供一个叫做when的工具函数，其使用类似。
```js
var fileP1=downloadP('file1.txt'),
    fileP2=downloadP('file2.txt'),
    fileP3=downloadP('file3.txt');
when([fileP1,fileP2,fileP3],function(files){
  console.log('file1:'+files[0]);
  console.log('file2:'+files[1]);
  console.log('file3:'+files[2]);
});
```
使promise成为卓越的抽象层级的部分原因是通过then方法的返回值来联系结果，或者通过工具函数如join来构成promise，而不是在并行的回调函数间共享数据结构。本质上是安全的，因为避免了66条中讨论过的数据竞争。即使最小心谨慎的程序员也可能会在保存异步操作的结果到共享的变量或数据结构时犯下简单的错误。
```js
var file1,file2;
downloadAsync('file1.txt',function(file){
  file1=file;
});
downloadAsync('file2.txt',function(file){
  file1=file;
});
```
promise避免这种BUG，简单风格的组合promise避免了修改共享数据。
注意异步逻辑的有序链事实上也可用有序的promise，而不是在62条中展现的笨重的嵌套模式。错误处理会自动地通过promise传播。当你通过promise串联异步操作的集合时，你可以为整个序列提供一个简单的error回调函数，而不是将error回调函数传递给每一步，正如63条中的代码所示。
尽管这样，有时故意创建某些种类的数据竞争是有用的。Promise为些提供了一个很好的机制。例如，一个应用程序可能需要尝试从多个不同的服务器上同时下载同一份文件，而选择最先完成的那个文件。select(或choose)工具函数接收几个promise并产生一个其值是最先完成下载的文件的promise。换句话，几个promise彼此竞争。
```js
var fileP=select(downloadP('http://e1.com/file.txt'),downloadP('http://e2.com/file.txt'),downloadP('http://e3.com/file.txt'));
fileP.then(function(file){
  console.log('file: '+file);
});
```
select函数的另一个用途是提供超时来终止长时间的操作。
```js
var fileP=select(downloadP('http://e1.com/file.txt'),timeoutErrorP(2000));
fileP.then(function(file){
  console.log('file: '+file);
},function(error){
  console.log('I/O error or timeout: '+error);
});
```
这里提供error回调函数作为第二个参数给promise的then方法的机制。
## 提示
- promise代表最终值，即并行操作完成时最终产生的结果
- 使用promise组合不同的并行操作
- 使用promise模式的API避免数据竞争
- 在要求有意的竞争条件时使用select(也被称为choose)