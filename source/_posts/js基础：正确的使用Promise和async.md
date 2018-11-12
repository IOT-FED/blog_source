---
title: js基础：正确使用Promise和Async函数
date: 2018-10-24 12:30:00
tags: 总结
author: simin
---

众所周知，js 是单线程运行的，需要一定时间才能得到结果的操作，需要被异步执行，而不是阻塞线程
异步编程是有点难以理解的，而且都是以异步回调函数的方式实现，我还记得第一次看到回调函数的时候，想的是：`一个函数竟然可以作为另一个函数的参数传入并使用`，而且这些函数写的一层套一层，超过 4，5 次基本读起来就非常绕了，看到这样的代码都要手动记录一下，心很累，就像这样：

```javascript
function f1(x) {
  return x + 1
}
··· 一些无关代码
function f2(y, callback) {
  return callback(y+1)
}
··· 另一些无关代码
function f3(z, callback1, callback2) {
  return callback1(z+1, callback2)
}
f3(1, f2, f1)
```

或是这样：

```javascript
// 一些有先后依赖关系的异步方法，前一个异步方法的返回值用作后一个异步方法的参数
asyncFunc1(){
  ···一些操作取到args1，取不到的话return出去做错误处理
  asyncFunc2(args1){
    ···一些操作取到args2，取不到的话return出去做错误处理
    asyncFunc3(args2){
      ···一些操作取到args3，取不到的话return出去做错误处理
      asyncFunc4(args3){
        // 继续或其他操作
      }
    }
  }
}
```

很明显，异步方法嵌套起来难以阅读，难以维护，这时候 Promise 出现了，从此异步编程的写法变得简单起来

## 什么是 Promise

Promise 对象，是一个承诺，承诺异步回调函数将来一定会执行，不管是执行成功，还是执行失败，

这是一些使用 Promise 入门知识点：

- 一个 Promise 对象创建后有三种状态：`pending(处理中)`，`resolve(执行成功)`，`reject(执行失败)`，状态只能变化一次，从`pending`到`resolve`或`reject` 。
- 一个 Promise 对象有两个参数：`resolve` 和 `reject`，相当于它执行后的回调函数，创建之后的链式使用中，`resolve`里带的参数传给了`.then()`，`reject`里带的参数传给了`.catch()`
- 想要链式的写异步方法，只需要在`.then()`里的处理再返回一个 Promise 对象，就可以开心的在后面，继续`.then()`了，而且`.catch()`只需要写一次，集中处理，任何步骤中进入了`reject`的话，都会结束链式调用，转去处理错误

这是一个简单的例子，会在 1s 后随机输出执行成功或是执行失败：

```javascript
// 定义
const myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    if (Math.random() > 0.5) {
      resolve('执行成功');
    } else {
      reject('执行失败');
    }
  }, 1000);
});
// 使用
myPromise
  .then(res => {
    console.log(res);
  })
  .catch(err => {
    console.log(err);
  });
```

这里改造上面的那个多个异步操作嵌套的话，只需要把每个异步操作都封装为返回 Promise 对象的函数，就可以链式调用了：

```javascript
// 一些有先后依赖关系的异步方法，前一个异步方法的返回值用作后一个异步方法的参数，使用Promise
// 异步方法封装，给success和msg设置值 来自定义这些异步操作每一步的执行逻辑，是否成功
function asyncFunc1() {
  return new Promise((resolve, reject) => {
    if (success) resolve(msg);
    else reject('asyncFunc1请求失败');
  });
}
function asyncFunc2(judge) {
  return new Promise((resolve, reject) => {
    if (judge) resolve(msg);
    else reject('asyncFunc2请求失败');
  });
}
function asyncFunc3(judge) {
  return new Promise((resolve, reject) => {
    if (judge) resolve(msg);
    else reject('asyncFunc3请求失败');
  });
}
function asyncFunc4(judge) {
  return new Promise((resolve, reject) => {
    if (judge) resolve(msg);
    else reject('asyncFunc4请求失败');
  });
}
// 使用
asyncFunc1()
  .then(res => {
    return asyncFunc2(res);
  })
  .then(res => {
    return asyncFunc3(res);
  })
  .then(res => {
    return asyncFunc4(res);
  })
  .then(res => {
    console.log(res);
  })
  // 所有reject的错误，都在这里统一处理
  .catch(err => {
    console.log(err);
  });
```

- 使用`Promise.all()`方法和`Promise.race()`方法可以并行执行异步任务，在一些情况下很实用，例如对于两个 Promise 对象 p1，p2，`Promise.all(p1, p2).then()`会在两个 Promise 都返回结果后继续执行，而`Promise.race()`会在先回来一个结果后就继续执行，而不管另一个了,

此例中可观察到两个方法在执行时间上的差异：

```javascript
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('执行成功');
  }, 2000);
});
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('执行成功');
  }, 3000);
});

console.time('p1');
console.time('p2');
Promise.all([p1, p2]).then(() => {
  console.timeEnd('p1');
});
Promise.race([p1, p2]).then(() => {
  console.timeEnd('p2');
});
// 输出结果：p2: 2000.76904296875ms p1: 3000.4541015625ms
```

## Promise 的好处

我的理解是：1. 对每一个异步操作都有了状态的管理，结构有了规范的限制，错误能统一处理 2. 嵌套起来的异步操作可以写为链式调用，结构简单，易读好改

其他人的理解：

## ES7 里的 async 函数

没错，异步编程的最高境界，就是，看起来和同步编程一样，async 函数可以做这件事

这是一些使用 async 的入门知识点：

- 使用`async` 定义函数，使用`await` 在异步代码的执行语句之前，表明这是异步操作，并接受返回一个 Promise 对象
- 因为在 async 函数里可以当做同步执行，所以可以使用 try-catch 来做错误处理
- async 函数默认返回一个 Promise 对象，其状态会根据 async 是否执行完，是否进入错误状态来确定 返回的 Promise 对象的状态

简单的例子：

```javascript
// 定义一个异步方法
function getVal() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (Math.random() > 0.5) resolve('done');
      else reject('不是想要的数字');
    }, 1000);
  });
}

// 定义此异步函数，使用try-catch做错误处理
async function judge() {
  try {
    let val = await getVal();
    console.log(val);
  } catch (err) {
    console.log(err);
  }
}
// 使用
judge();
```

async 函数内部有创建任务和执行任务的机制：

```javascript
// 定义两个一段时间后才返回值的方法
function p1() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('执行成功');
    }, 2000);
  });
}
function p2() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('执行成功');
    }, 3000);
  });
}

// 这个方法里，写的是有问题的，因为两个异步方法后一个并不依赖前一个的返回值
// 这样写会让两条await串行执行，浪费了时间
async function judge1() {
  console.time('judge1');
  let v1 = await p1();
  let v2 = await p2();
  console.timeEnd('judge1');
}
judge1(); // 输出结果：judge1: 5000.76904296875ms

// 将任务创建和任务执行分开写，两个await语句就是并行的
async function judge2() {
  console.time('judge2');
  let v1 = p1();
  let v2 = p2();
  let v11 = await v1;
  let v22 = await v2;
  console.timeEnd('judge2');
}
judge2(); // 输出结果：judge2: 3000.76904296875ms
```

建议await后直接跟一个Promise对象，而不是一个函数，貌似跟个函数，它就默认这些操作有先后关系了（虽然前一个异步操作的结果并没有在后一个异步操作中当做条件使用 -_-||）

## 一些例子

  1. 经典的例子，使用 Promise 实现红黄绿三色灯的间隔闪烁

```javascript
// 定义亮灯函数
function green() {
  console.log('green');
}
function yellow() {
  console.log('yellow');
}
function red() {
  console.log('red');
}
// 把亮灯操作封装为Promise对象，传入亮灯函数和时间间隔，自定义
function flash(cb, time) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      cb(), resolve();
    }, time);
  });
}
// 定义亮灯循环
function control() {
  Promise.resolve()
    .then(function() {
      return flash(red, 1000);
    })
    .then(function() {
      return flash(green, 1000);
    })
    .then(function() {
      return flash(yellow, 1000);
    })
    .then(function() {
      control();
    });
}
// 开始
control();
```

  2. 多重错误处理

   一个常见的需求，使用 axios 这种返回 Promise 对象的网络请求工具时，由客户端或服务端自身的问题导致的错误（非 200），会进入`catch()`里进行处理，但如果协议里还自定义了错误状态码，在正常收到接口返回内容(`resolve`)之后，还需要根据内容中的错误状态码进入`.catch()`进行错误处理，怎么做？

   一个简单的做法就是两层 Promise 嵌套(一不小心又写成嵌套了 -_-||)，内层处理协议中的错误状态码，外层处理 axios 请求不成功的错误，内外层  `reject` 之后都会跑到外层的`.catch()`里，再统一处理

这里模仿这种场景，写了一个对[-2, 2] 的随机数进行两次判断，筛选出大于 1 的数，否则进入外层的错误处理：

```javascript
// 需要的数字必需为正数
function positiveNum() {
  return new Promise((resolve, reject) => {
    const getNum = (Math.random() - 0.5) * 4;
    if (getNum >= 0) resolve(getNum);
    else reject(`这是个负数：${getNum}`);
  });
}
// 需要的数字必需大于1
function moreThanOne() {
  return new Promise((resolve, reject) => {
    // 注意这里内层Promise在.then()里reject的话，就进入的外层Promise的.catch()里进行处理，是关键
    positiveNum()
      .then(res => {
        if (res > 1) resolve(res);
        else reject('这个数不大于1：' + res);
      })
      .catch(err => {
        reject(err);
      });
  });
}
// 使用
moreThanOne()
  .then(res => {
    console.log(`符合的输出：${res}`);
  })
  .catch(err => {
    console.log(`错误的输出：${err}`);
  });
```

  3. 这是一些以前积累的一些比较绕的陷阱问题，理解以后更懂 Promise 的机制

```javascript
// 示例1
var test = new Promise((resolve, reject) => {
  console.log(1);
  resolve();
  console.log(2);
});
test.then(() => {
  console.log(3);
});
console.log(4);

// 问题分析：因为Promise的构造函数是同步执行的，而.then()是异步执行的，所以打印信息是：1 - 2 - 4 - 3

// 示例2
var test = new Promise((resolve, reject) => {
  resolve('success1');
  reject('error');
  resolve('success2');
});
test
  .then(res => {
    console.log(res);
  })
  .catch(err => {
    console.log(err);
  });

// 问题分析：Promise的状态一经改变，就不会再发生变化，构造函数里的3条不同处理逻辑只会执行第1条并相应更改状态，打印的信息是：success1

// 示例3
Promise.resolve(1)
  .then(res => {
    console.log(res);
    return 2;
  })
  .catch(err => {
    console.log(err);
  })
  .then(res => {
    console.log(res);
  });

// 问题分析：因为直接执行的是resolve(1)，所以下面执行的是第一个.then()，没有reject()所以不会执行.catch()，继续执行第二个.then()，打印的信息是：1 - 2

// 示例4
var test = new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log('once');
    resolve('success');
  }, 1000);
});
var start = Date.now();
test.then(res => {
  console.log(res, Date.now() - start);
});
test.then(res => {
  console.log(res, Date.now() - start);
});

// 问题分析：因为一个Promise对象只能被修改一次状态，所以第二次执行.then()时test已经变为了resolve状态，相当于同步执行了第二个.then的内容。输出为：once - 1000 - 1001

// 示例5
var pr1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('success');
  }, 1000);
});
var pr2 = pr1.then(res => {
  throw new Error('error!');
});
console.log('pr1', pr1);
console.log('pr2', pr2);
setTimeout(() => {
  console.log('pr1', pr1);
  console.log('pr2', pr2);
}, 2000);

// 问题分析：因为pr2的构造中使用了pr1.then()，构造时会执行，但pr1中有1s的定时器；pr2构造中会报错，所以报提示信息；
// 流程是：1. 构造pr2时 pr1.then()开始执行，但延时1s，接下来打印信息pr1,pr2，都是出于pending状态
//        2. 1s后pr1进入resolve状态，同时使得pr2构造报错，打印报错信息，pr2进入reject状态
//        3. 2s后再次打印pr1和pr2，输出pr1(resolve状态)，pr2(reject状态)
// 输出内容为:  pr1(pending状态)，pr2(pending状态) - error! - pr1(resolve状态)，pr2(reject状态)
```

> 一些代码段放在这里 ： https://github.com/ChenSimin0103/Demo/tree/master/js%E5%9F%BA%E7%A1%80%EF%BC%9A%E6%AD%A3%E7%A1%AE%E7%9A%84%E4%BD%BF%E7%94%A8Promise%E5%92%8Casync%E5%87%BD%E6%95%B0