---
title: JS野生程序员进修指南
date: 2017-05-15 21:08:00
tags: [笔记]
toc: true
description: 如何培养代码洁癖。
---
### 基本
`缩进`
4个空格，而不是一个制表符

`结尾`
分号结尾

`行长度`
80个字符

`换行`
符号结尾，第二行追加2单位(8字符)缩进

`空行`
* 方法之间
* 方法中局部变量和第一条语句之间
* 注释之前
* 逻辑片段之间

`命名`
* Camel Case
* 变量前缀为名词
* 函数前缀为动词
* 表数量 count, length, size
* 循环体 i, j, k
* 常量使用大写字母，下划线分隔单词
* 构造函数使用Pascal Case


`函数名参考`

动词  |  含义
------|------
can   |返回布尔值
has   |返回布尔值
is    |返回布尔值
get   |返回非布尔值
set   |保存一个值


`直接量`
* 字符串 单引号、双引号皆可
* 数字，不使用八进制；不以小数点结尾

`null`
* 对象占位符
* 不要 检测是否传入某个参数
* 不要 检测一个未初始化的变量

`undefined`
未被初始化的变量初始值

`对象直接量`
```javascript
//不好的写法
var book = new Object();
book.title = "JavaScript";
book.author = "Mike";

//好的写法
var book = {
    title: "JavaScript",
    author: "Mike"
}
```

`数组直接量`
```javascript
//不好的写法
var colors = new Array("red", "blue", "green");
var numbers = new Array(1, 2, 3);

//好的写法
var colors = ["red", "blue", "green"];
var numbers = [1, 2, 3];
```

### 注释
`单行注释`
//something
* 独占一行，解释下一行，注释前留一空行，缩进与下一行保持一致
* 尾部注释与代码保持一单位缩进，不应超过最大长度限制

`多行注释`
```javascript
/*
 * 第一行注释
 * 第二行注释
 */
```

`注释时机`
* 难于理解的代码
* 可能被理解错误的代码
* 浏览器兼容性

`文档注释`
```javascript
/**
这是一个文档注释
**/
```

### 语句和表达式
`基本`
```javascript
//好的写法
if (condition) {
    doSomething();
}

//不好的写法
if (condition)
    doSomething();

if (condition) doSomething();

if (condition) { doSomething(); }
```
缩进应对齐
所有块语句都应使用花括号
* if
* for
* while
* do...while...
* try...catch...finally

`花括号对齐方式`
```javascript
//好的写法
if (condition) {
    doSomething();
} else {
    doSomethingElse();
}

//不好的写法
if (condition)
{
    doSomething();
}
else
{
    doSomethingElse();
}
```

`块语句间隔`
```javascript
//好的写法
if (condition) {
    doSomething();
}

//不好的写法
if(condition){
    doSomething();
}

if ( condition ) {
    doSomething();
}
```

`switch`
```javascript
switch(condition) {
    case "first":
         //代码
         break;

    case "second":
         //代码
         break;

    case "third":
         //代码
         break;

    default:
         //代码
}
```

`with`
禁用

`for`
避免使用continue

`for-in`
不应遍历数组

### 变量、函数和运算符
`变量声明`
* 注意提升机制

```javascript
//好的写法
function doSomethingWithItems(items) {
    var value = 10,
        result = value + 10,
        i,
        len;

    for (i=0, len=items.length; i < len; i++) {
        doSomething(items[i]);
    }
}
```

`函数声明`
* 先定义后使用
* 不应在块语句内定义
* 函数名与左括号间无空格

`立即调用函数`
```javascript
//好的写法
var value = (function() {
  //函数体

  return {
      message: "Done"
  }
}());
```

`严格模式`
* 应在函数体内使用，而非全局

`相等`
* 应使用 '===', '!==', 而不是 '==', '!='

`eval()`
* 避免使用 eval(), Function
```javascript
//不好的写法
var myfunc = new Function("alert('Hi!')");

setTimeout("document.body.style.background='red'", 50);

setInterval("document.title = 'It is now'" + (new Date()), 1000);

//而应使用匿名函数代替
setTimeout(function() {
    document.body.style.background='red';
}, 50);
```

`原始包装类型`
```javascript
//不好的写法
var name = new String("Mike");
var author = new Boolean(ture);
var count = new Number(10);
```
