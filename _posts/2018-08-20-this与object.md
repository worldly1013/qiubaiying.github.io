---
layout:     post
title:      this与object
subtitle:  《你不知道的JavaScript》系列上卷
date:       2018-05-29
author:     worldly
header-img: img/post-bg-cat-1.jpg
catalog: true
tags:
    - JS
---


### 前言
JS中七种基础数据类型，string、number、boolean、null、undefined、object、symbol,其中最重要最难理解的也就是对象类型了，如“一等公民”的函数其实就是对象的一个子类型。当对象配合上this指针，着实让人头大了。


#### 一、this

> this指针指向不好判断，其实主要在于分析思路不正确，再加上网上很多错误资料的误导，造成理解偏差。判断this指向时，先分清属于下面4种情况的哪一种，然后按优先级确定this指向。

![](http://dev.fenzhitech.com/res/6aa0dc4cfd00f6eb0ffdd3b60267e0e2.png)

**面试常见坑：**    
1、实现bind函数功能    
```
function foo(something){
  console.log(this.a , something)
  return this.a + something
}

function _bind(fn,obj){
  return function(){
    return fn.apply( obj , arguments )
  }
}

var obj = {a:2}
var bar = _bind(foo,obj)
var res = bar(4)    // 6
```

2、实现new操作符
```
function Animal(name){
  this.name = name;
}

var cat = new Animal('cat')
cat.name            // cat

function _new(name){
  var o = {};
  o.__proto__ = Animal.prototype     // 绑定到Animal的原型上
  Animal.call(o , name)              // this指针指向新对象
  return o
}

var dog = _new('dog')
dog.name          // dog
dog instanceof Animal   // true


function _new(constructor){
  var o = {}
  o.__proto__ = constructor.prototype
  constructor.apply(o , Array.prototype.slice.call(arguments,1))
  return o
}

var dog = _new(Animal,'dog')
dog.name          // dog
```

#### 二、object

![](http://dev.fenzhitech.com/res/2c2bc9f6aa128341d17e433296bc9c8b.png)

![](http://dev.fenzhitech.com/res/9009dbb066ff580e108eff0c6e27c425.png)

**扩展：**
1、对象类型判断

##### 1）typeof
> 对象类型有复合类型如 Array、Date 、RegExp ，采用 *typeof* 判断时只能辨别出基本数据类型，对于复合的对象类型无法更进一步分辨出。基础类型 null采用 *typeof* 得到object

```
typeof true === 'true'  
typeof [1,2,3] === 'object'
typeof null === 'object'
```

##### 2) instanceof
> 通过 *instanceof* 运算符来检测一个构造函数的prototype属性是否存在一个对象的原型链上

```
var arr = [] ,
    reg = /\bis\b/g ,
    str1 = new String(),
    str2 = 'aaa';

arr instanceof Array      // true
arr instanceof Object     // true
arr instanceof RegExp     // false

reg instanceof RegExp     // true
str1 instanceof String     // true
str2 instanceof String    // false
```

##### 3) constructor  
> 实例化的对象都有constructor属性，其constructor属性指针指向的是构造函数，可用来判断未知对象的类型


```
arr.constructor === Array      // true
arr.constructor === Object     // true
arr.constructor === RegExp     // false

reg.constructor === RegExp     // true
str.constructor === String     // true
```

##### 4) Object.prototype.toString.call()
> 调用Object的原型方法toString

```
a = 'hello'
b = []
c = function(){}

Object.toString.call(a) === "[object String]"
Object.toString.call(b) === "[object Array]"
Object.toString.call(c) === "[object Function]"
```
