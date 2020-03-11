---
layout:     post
title:      ---
layout:     post
title:      async和await
subtitle:   
date:       2019-04-18
author:     worldly
header-img: img/post-bg-tech-1.jpg
catalog: true
tags:
    - JS
---

### 前言
函数异步执行是我们经常遇见的场景，初始时我们采取回调的方式，但如果遇到异步嵌套过多时，则会出现回调地狱的窘境。后来 *ES6* 新出 *Promise* 后，异步代码变成了同步代码的写法，代码逻辑瞬间也清晰明了了。今天再学习 *ES6* 的新函数 *async function* ，这将使异步操作变得更加方便。

#### 一、async
> *async function* 声明用于定义一个返回 *AsyncFunction* 对象的异步函数。异步函数是指通过事件循环异步执行的函数，它会通过一个隐式的 Promise 返回其结果。但是如果你的代码使用了异步函数，它的语法和结构会更像是标准的同步函数

[async function MDN 介绍](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function)

简单点说：就是如果函数前面加上关键字 *async* , 那么这个函数就会返回 *promise* ,即使你没有亲自返回一个 *promise* 函数，其函数也会自动把值包装为 *Promise* 的 *resolve* 返回

```
// 返回一个promise
async function aa() {
    return new Promise(resolve => {
        setTimeout(function(){
            resolve('aaaaaa');
        }, 1000);
    });
}

aa().then(res => {
    console.log(res); // 1s后输出 'aaaaaa'
});

typeof aa === 'function'; // true
Object.prototype.toString(aa) === '[object AsyncFunction]'; // true
Object.prototype.toString(aa()) === '[object Promise]'; // true



// 返回一个非promise
async function a() {
    return 1;
}
const b = a();
console.log(b); // Promise {<resolved>: 1}

a().then(res => {
    console.log(res); // 1
})
```

如果 *async function* 抛出异常，则可以通过 *Promise* 函数的 *catch* 方法捕获到

```
async function aa(){
  return bbb
}

aa()
.then((res) => {
    console.log(res)
})
.catch((error) => {
    console.log(error)   // ReferenceError: bbb is not defined
})

```

#### 二、await

*await* 操作符用于等待一个 *Promise* 对象。它只能在异步函数 *async function* 中使用。await 表达式会暂停当前 *async function* 的执行，等待 *Promise* 处理完成。若 *Promise* 正常处理(fulfilled)，其回调的 *resolve* 函数参数作为 *await* 表达式的值，继续执行 *async function*。若 *Promise* 处理异常(rejected)，*await* 表达式会把 *Promise* 的异常原因抛出。另外，如果 *await* 操作符后的表达式的值不是一个 *Promise*，则返回该值本身。

注意： 当代码执行到 *await* 语句时，会暂停执行，直到 *await* 后面的 *promise* 正常处理

```
const p = function() {
    return new Promise(resolve => {
        setTimeout(function(){
            resolve(1);
        }, 1000);
    });
};

const fn = async function() {
    const res = await p();
    console.log(res);
    const res2 = await 2;
    console.log(res2);
};

fn(); // 1s后，会输出1, 紧接着，会输出2


// 把await放在try catch中捕获错误
const p2 = function() {
    return new Promise(resolve => {
        console.log(ppp);
        resolve();
    });
};

const fn2 = async function() {
    try {
        await p2();
    } catch (e) {
        console.log(e); // ppp is not defined
    }
};

fn2();
```

#### 三、任务队列

我们有一堆任务，我们需要按照一定的顺序执行这一堆任务，拿到最终的结果。这里，把这一堆任务称为一个任务队列，在实际操作中，我们可以把这些任务放在数组中处理。

##### 1、同步任务

任务队列中的函数都是同步函数。这种情况比较简单，我们可以采用reduce很方便的遍历

```
const fn1 = function(i) {
    return i + 1;
};
const fn2 = function(i) {
    return i + 2;
};
const fn3 = function(i) {
    return i * 100;
};

const taskList = [fn1, fn2, fn3];
let a = 1;
const res = taskList.reduce((sum, fn) => {
    sum = fn(sum);
    return sum;
}, a);

console.log(res); // 400
```

##### 2、异步任务

异步任务队列时，我们就可以采用 *async function* 函数配合 *for of* 来遍历执行了

```
const fn1 = () => {
  return new Promise(resolve => {
    setTimeout(() => {
      console.log('fn1 --- 1s后执行 ---' + formatDate(new Date()));
      resolve()
    },1000)
  })
}

const fn2 = () => {
  return new Promise(resolve => {
    setTimeout(() => {
      console.log('fn2 --- 2s后执行 ---' + formatDate(new Date()));
      resolve()
    },2000)
  })
}

const fn3 = () => {
  return new Promise(resolve => {
    setTimeout(() => {
      console.log('fn3 --- 3s后执行 ---' + formatDate(new Date()));
      resolve()
    },3000)
  })
}

;(async function(){
    for(let fn of taskList) {
        await fn();
    }
})();

```

辛苦了这么多，终于看到结果了：
![](http://dev.fenzhitech.com/res/31a0817fde2d8586831d27c84704db90.png)
