---
title: js基础：函数的call,apply和bind方法
date: 2018-10-10 12:30:00
tags: 总结
---

js 里的 `call, apply, bind` 方法都是 javascript 的基础方法，不过不经常在业务中使用，这里记录对它们的理解，和一些用法

## call 和 apply 方法

from javascript 权威指南（犀牛书）P189:
我们可以将 call()和 apply()看做是某个对象的方法，通过调用方法的形式来间接调用函数

在 javascript 中，call 和 apply 都是为了改变某个函数运行时的上下文（context）而存在的，换句话说，就是为了改变函数体内部 this 的指向。

听起来有点道理，直观一点的理解：

有一个函数 `f` 和某个对象 `o` ，想通过调用方法的形式 `o.f()` 调用函数 `f`，但又不想通过`o.f = f`把函数`f()`添加为对象`o`的方法（可能只会调用一次），就可以使用 `f.call(o)` 这做这件事，即：

```javascript
f.call(o);
f.apply(o);
```

每行代码的功能类似于：

```javascript
o.tempf = f;
o.tempf();
delete o.tempf;
```

对于 call 和 apply 传入的参数，有如下定义：

1. 第一个实参作为函数内 this 的值，也就是该函数执行的上下文（我理解为函数的调用者），不传会报错，可以传入 null 或 undefined，会被替换为全局对象
2. 对于 call() 来说，第一个实参之后的所有实参就是传入调用函数的参数
3. 遇到参数不定的情况，使用 apply() 来处理，对 apply()传入第一个实参之后的参数是一个数组

   例如这样写：

   ```javascript
   f.call(o, 1, 2, 3);
   f.call(o, [1, 2, 3]);
   // 等同于：
   o.tempf = f;
   o.tempf(1, 2, 3);
   delete o.tempf;
   ```

## bind方法

与 call 和 apply 方法类似，bind 也是用来把 函数的调用者`o`当做参数传进函数`f`来完成功能，区别不过是只会通过`wrapf = f.bind(o)`绑定到调用函数的对象上并返回一个新函数`wrapf`，并不会立即执行

`注意`：传入 bind()的所有实参都会绑定到返回的函数 this，包括第一个实参（函数调用者，上下文）和其他实参（按个数依次绑定给原函数的形参，不够数的话，剩下的参数要在返回的新函数里传入），这种编程技术被叫做"柯里化"（Currying），或者可以这样解释：只传递给函数一部分参数来调用它, 让它返回一个函数去处理剩下的参数

我的理解是，既然生成了一个新的函数`wrapf`，那么其需要传入的参数是可以继承`f`里的传入个数或更少（就像在新函数定义的时候给某些变量固定了值），为了保证这种用法的灵活性，要能将传入参数个数不同时产生的新函数区分开来，即需要实现传入参数个数不同的包装方法产生的是不同的新函数，即：

```javascript
var sum = function(x, y, z) {
  return x + y + z;
};

// 固定原函数的两个参数，只需传入一个参数的新函数
var oneSum = sum.bind(null, 1, 2);
// 这里给sum函数的 x绑定了1，y绑定了2，z需要使用时传入，按照顺序来
// 即等同于：
var tempSum = function(z) {
  return sum(1, 2, z);
};
tempSum(3); // 输出 6
oneSum(3); // 输出 6

// 固定原函数的一个参数，需要传入两个参数的新函数
var twoSum = sum.bind(null, 1);
// 这里给sum函数的 x绑定了1
twoSum(2, 3); // 输出 6
```

## 一些使用场景

虽然 bind，call，apply 在业务中很少见，但也有一些巧妙的使用

1. 比如之前看到了一种函数节流的实现方法：

    ```javascript
    // 节流函数
    function jieliu(fn, delay) {
      return function(args) {
        let self = this;
        let _args = args;
        clearTimeout(fn.id);
        fn.id = setTimeout(() => {
          fn.call(self, _args);
        }, delay);
      };
    }
    // 原函数
    function show(msg) {
      console.log(msg);
    }
    // 包装原函数
    const jieliuShow = jieliu(show, 1000);
    // 使用
    jieliuShow('233');
    ```

    这样的效果就是，频繁调用`jieliuShow`函数的时间间隔小于 1s 时，会不断地重置定时器，不执行`show`，在时间间隔某次大于 1s 时，定时器定时时间到，执行一次`show`函数

2. 有一个数组，求其中的最大值，最小值：

    ```javascript
    var arr = [1, 3, 5, 7, 9];

    Math.max.apply(null, arr);
    Math.min.apply(null, arr);
    ```

    当然最好的办法是使用解构赋值：

    ```javascript
    var arr = [1, 3, 5, 7, 9];

    Math.max(...arr);
    Math.min(...arr);
    ```

3. 重写某个对象里的某个方法：

    这个例子重写了方法使得方法执行前和执行后输出日志消息：

    ```javascript
    // 有一个对象o
    var o = {
      m: function() {
        console.log('233');
      },
    };
    // 将对象o中的m方法替换为另一个方法
    function trace(o, m) {
      let original = o[m];
      o[m] = function() {
        console.log(new Date(), '开始执行', m);
        const result = original.apply(this, arguments);
        console.log(new Date(), '执行完毕', m);
        return result;
      };
    }
    // 使用
    trace(o, 'm');
    o.m();
    ```

所有代码段都可以在浏览器调试窗口直接测试

> 关于函数节流的小demo放在这里 ： https://github.com/ChenSimin0103/Demo/tree/master/js%E5%87%BD%E6%95%B0%E5%9F%BA%E7%A1%80%EF%BC%9Acall%2Capply%E5%92%8Cbind