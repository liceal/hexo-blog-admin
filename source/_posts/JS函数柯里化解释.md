---
title: JS函数柯里化解释
date: 2020-07-03 09:47:32
tags: js
---

> 函数柯里化是一个很有意思的东西，可以对一个自定义次数调用，并且每次调用参数随意，感觉很 Free，个人觉得功能跟继承类很像。
>
> 引用别人的话：
>
> curry 的概念很简单：只传递给函数一部分参数来调用它，让它返回一个函数去处理剩下的参数。
>
> 你可以一次性地调用 curry 函数，也可以每次只传一个参数分多次调用。
>
> ![](/img/klh.jpg)

<!--more-->

上面的图就是使用函数柯里化，这里也实现一个类似的

## 开始之前，先了解一下`隐式转换`

隐式转换，这个名字听上去比较高级，就是在某些特定的情况下，系统自动进行了类型转换，举个简单的例子

```js
console.log(1 + 1 + "1"); // 21
console.log("1" + 1 + 1); // 111
```

`1+1=2` + `'1'` = `21`，因为后面遇到了字符串类型，所以进行了字符串拼接

`'1'+1='11'` + `1` = `111`，开始字符串相加的时候就成了字符串，那后面都是字符串拼接

还有其他方法`toString`在变量转成`String`类型时候触发，`valueOf`在变量转成`Number`类型的时候触发

> 所以可以利用这个特性，去对函数柯里化

## 例子

跟上面的截图题目差不多

```js
function count() {
  // 第一次执行时，定义一个数组专门用来存储所有的参数
  var _args = [...arguments];
  var step = 1;
  // console.log(`第${step++}次使用，参数：`, arguments);

  // 在内部声明一个函数，利用闭包的特性保存_args并收集所有的参数值
  var _adder = function () {
    // console.log(`第${step++}次使用，隐式转换前，先执行我，参数：`, arguments);
    _args.push(...arguments); // ...[2]  -> 2 -> push(2)
    return _adder; //返回自己实现()()()调用
  };

  // toString 和 valueOf 隐式转换特性，到最后等待触发事件，这里关键代码
  _adder.toString = function () {
    //闭包特性，这个_args如果在此方法中找不到会向上寻找
    var res = _args.reduce((a, b) => {
      return a + b * b;
    });
    // console.log(`隐式转换，计算结果：${res}`);
    return res;
  };

  return _adder;
}
```

### 如果直接输出

```js
console.log(count(1)(2)(3)); // { [Function: _adder] valueOf: [Function] }
console.log(typeof count(1)(2)(3)); // function
```

本身是一个方法，触发不了方法执行，除非触发上面代码的`toString`

### 触发`toString`执行方法

```js
console.log(String(count(1)(2)(3))); // 14
console.log("count: " + count(1, 2)(3)); // count: 14
```

> 这个时候使用，调用触发隐式转换，也就是执行`toString`指向的方法

`toString`触发，顾名思义，转成`String`类型就行了，使用`String(count(1))`或者`''+count(1)`来触发`String`类型的隐式转换触发的方法

### 或者 Number 类型

代码改一下 将 `toString` 换成 `valueOf`

```js
...
_adder.valueOf = function() {
    //闭包特性，这个_args如果在此方法中找不到会向上寻找
    var res = _args.reduce((a, b) => {
        return a + b * b
    })
    // console.log(`隐式转换，计算结果：${res}`);
    return res
}
...
```

```js
console.log(Number(count(1)(2)(3)))); // 14
console.log(count(1,2)(3) + 1); // 15
```

同理，改变成`Number`类型将触发`valueOf`

---

🆗，我们现在用隐式转换，使用了这个()()()方式调用，其实可以不用类型转换。

总所周知，`JavaScript`万物皆对象，上面的代码改一下

```js
...
_adder.abc = function() {
    //这个_args如果在此方法中找不到会向上寻找
    var res = _args.reduce((a, b) => {
        return a + b * b
    })
    // console.log(`隐式转换，计算结果：${res}`);
    return res
}
...
```

```js
console.log(count(1)(2)(3).abc()); // 14
console.log(typeof count(1)(2)(3)); // function
```

> 执行 abc()方法，触发事件

这个更好理解，这里没有改变`_adder`的类型，他依然是一个方法

直接输出了其中一个方法的返回值，差别是需要手动调用，而使用隐式转换特性就能省这一步

---

okay~，如果看完有收获，可以打赏我一杯咖啡！😊
