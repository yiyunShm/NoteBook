[TOC]

## Javascript Deep Learning
晚上闲着无事刷着博客，看到了几个非常有意思的小题目，顺手记录下来。偶尔回看一下，也能更好的巩固基础

原文链接：
Baranovskiy: [So, you think you know JavaScript?](http://dmitry.baranovskiy.com/post/91403200)
Nicholas C. Zakas: [Answering Baranovskiy’s JavaScript quiz](http://www.nczonline.net/blog/2010/01/26/answering-baranovskiys-javascript-quiz/)

### 题目
题一：
```
if (!("a" in window)) {
  var a = 1;
}
alert(a);
```
题二：
```
var a = 1,
    b = function a(x) {
      x && a(--x);
    };
alert(a);
```
题三：
```
function a(x) {
  return x * 2;
}
var a;
alert(a);
```
题四：
```
function b(x, y, a) {
  arguments[2] = 10;
  alert(a);
}
b(1, 2, 3);
```
题五：
```
function a() {
  alert(this);
}
a.call(null);
```

### 解析
#### 题目一
```
if (!("a" in window)) {
  var a = 1;
}
alert(a);
```
从代码上看想要的表述的意思是：如果window不包含a属性，就声明一个变量a，然后赋值为1.
随意一看的结果可能会以为打印出1，然而答案是"undefined"。至于原因，我们需要了解Javascript里的3个知识点。

首先，所有全局变量都挂载在window对象下，即是window的属性。语句`var a = 1`等价于`window.a = 1`；可以通过下面的方法来检测全局变量是否声明：
```
"variable" in window
```

第二，所有变量的声明都在范围作用域的顶部，即声明提升：
```
alert("a" in window);
var a;
```
此时，尽管声明是在alert之后，alert弹出的依然是true，这是因为Javascript引擎首先会扫描所有的变量声明，然后在执行程序。最终的代码效果是这样的：
```
var a;
alert("a" in window);
```

第三，这个题目主要的意思是，变量声明提前了，但是变量赋值被保留在原位置。可以将题目代码拆分如下：
```
var a;
if (!("a" in window)) {
  a = 1;
}
alert(a);
```
这样代码描述的就非常清晰了：首先声明a，然后判断a是否在存在，如果不存在就赋值为1，很明显a永远在window里存在，这个赋值语句永远不会执行，所以结果是undefined。

#### 题目二
```
var a = 1,
    b = function a(x) {
        x && a(--x);
    };
alert(a);
```
这个题目看起来比实际复杂，alert的结果是1；这里依然有3个重要的知识点需要我们去理解。

首先，我们知道了"变量声明提前"；

第二，函数声明也是提前的，和变量声明一样，在代码执行前就完成了扫描。说明一点，函数声明是如下代码：
```
function method(arg1, arg2) {
  // do something
}
```
函数表达式不是函数声明，相当于变量赋值：
```
var method = function(arg1, arg2) {
  // do something
};
```

第三，函数声明会覆盖变量声明，但不会覆盖变量赋值。先看一个例子：
```
function value(){
    return 1;
}
var value;
alert(typeof value);    //"function"
```
尽管变量声明在下面定义，但是变量value依然是function，也就是说这种情况下，函数声明的优先级高于变量声明的优先级。但如果该变量value在声明后被赋值了，那结果就完全不一样了：
```
function value(){
    return 1;
}
var value = 1;
alert(typeof value);    //"number"
```
代码可以拆分如下：
```
// variable statement
var value;
// function statement
function value(){
    return 1;
}
alert(typeof value);    //"function"

value = 1;
alert(typeof value);    //"number"
```
重新回到题目，这个函数其实是一个有名函数表达式，函数表达式不像函数声明一样可以覆盖变量声明，但你可以注意到，变量b是包含了该函数表达式，而该函数表达式的名字是a；不同的浏览器对a这个名词处理有点不一样，在IE里，会将a认为函数声明，所以它被变量初始化覆盖了，就是说如果调用a(--x)的话就会出错，而其它浏览器在允许在函数内部调用a(--x)，因为这时候a在函数外面依然是数字。基本上，IE里调用b(2)的时候会出错，但其它浏览器则返回undefined。

理解上述内容之后，该题目换成一个更准确和更容易理解的代码应该像这样：
```
var a = 1,
    b = function(x) {
        x && b(--x);
    };
alert(a);
```
这样的话，就很清晰地知道为什么alert的总是1了.

进入执行上下文： 这里出现了名字一样的情况，一个是函数声明，一个是变量声明。那么，填充VO(Variable Object)的顺序是:  函数声明 -> 函数的形参 -> 变量声明。
```
var a = 1;
function A(a) {
  console.log(a); // a()
  function a() {};
  console.log(a); // a()
}

A(a);
```

#### 题目三
```
function a(x) {
    return x * 2;
}
var a;
alert(a);
```
在题目二中有说明函数声明与变量声明的关系和影响，遇到同名声明时，VO不会重新定义

#### 题目四
```
function b(x, y, a) {
    arguments[2] = 10;
    alert(a);
}
b(1, 2, 3); //10
```
关于这个题目，NC搬出了262-3的规范出来解释：活动对象是在进入函数上下文时刻被创建的，它通过函数的arguments属性初始化。arguments属性的值是Arguments对象：
```
AO = {
  arguments: <ArgO>
};
```
Arguments对象是活动对象的一个属性，它包括如下属性：
1. callee — 指向当前函数的引用(ES5严格模式下被弃用)
2. length — 真正传递的参数个数
3. properties-indexes (字符串类型的整数) 属性的值就是函数的参数值(按参数列表从左到右排列)。 properties-indexes内部元素的个数等于arguments.length. properties-indexes 的值和实际传递进来的参数之间是共享的。

这个共享其实不是真正的共享一个内存地址，而是2个不同的内存地址，使用JavaScript引擎来保证2个值是随时一样的，当然这也有一个前提，那就是这个索引值要小于你传入的参数个数，也就是说如果你只传入2个参数，而还继续使用arguments[2]赋值的话，就会不一致，例如：
```
function b(x, y, a) {
    arguments[2] = 10;
    alert(a);
}
b(1, 2); //undefined
```
这时候因为没传递第三个参数a，所以赋值10以后，alert(a)的结果依然是undefined，而不是10，但如下代码弹出的结果依然是10，因为和a没有关系。
```
function b(x, y, a) {
    arguments[2] = 10;
    alert(arguments[2]);
}
b(1, 2); // 10
```

#### 题目五
```
function a() {
    alert(this);
}
a.call(null); // window
```
这个题目可以说是最简单的，也是最诡异的，我们也要了解两个知识点。

首先，就是this值是如何定义的，当一个方法在对象上调用的时候，this就指向该对象：
```
var object = {
    method: function() {
        alert(this === object);    //true
    }
}
object.method();
```
上面的代码，调用method()的时候this被指向到调用它的object对象上。

如果一个function的定义不是属于一个对象属性的时候(也就是单独定义的函数)，函数内部的this就等价于全局对象(浏览器全局对象为window)，例如：
```
function method() {
    alert(this === window);    //true
}
method(); 
```

第二，call方法作为一个function执行代表该方法可以让另外一个对象作为调用者来调用，call方法的第一个参数是对象调用者，随后的其它参数是要传给调用method的参数（如果声明了的话），例如：
```
function method() {
    alert(this);
}
method();    // [object window]
method.call(document);   // [object HTMLDocument]
```
另外，根据ECMAScript262规范规定：如果第一个参数传入的对象调用者是null或者undefined的话，call方法将把全局对象(也就是window)作为this的值。所以，不管你什么时候传入null，其this都是全局对象window，所以该题目可以理解成如下代码：
```
function a() {
    alert(this);
}
a.call(window);
```

### 拓展
1. 找出数字数组中最大的元素(使用Math.max)
```
Math.max.apply(null, [1, 3, 5, 15]);
```

2. 转化一个数字数组为function数组(每个function执行都弹出相应的数字)
```
var a = [1, 3, 5].map(function(v, i) {
  return function() {
    return v;
  }
});
```

3. 给object数组进行排序(排序条件是每个元素对象的属性个数)
```
[{a: 1, b: 2, c: 3}, {d: 4}, {e: 5, f: 6}].sort(function(prev, next) {
  return Object.keys(prev).length - Object.keys(next).length;
});
```

4. 利用JavaScript打印出Fibonacci数(不使用全局变量)
```
function Fibonacci(n) {
  var numArr = [1, 1];
  
  numArr = [1, 1];
  for (i = 0; i < n; i += 1) {
    if (!numArr[i]) {
      numArr[i] = numArr[i - 1] + numArr[i - 2];
    }
  }
  
  return numArr;
}
```

5. 实现如下语法的功能：var a = (5).plus(3).minus(6); //2
```
Number.prototype.plus = function(a) {
  return Number(this + a);
}

Number.prototype.minus = function(a) {
  return Number(this - a);
}
```

6. 实现如下语法的功能：var a = add(2)(3)(4); //9
```
function add(num) {
  function _add(args) {
    num += Number(args);
    return _add;
  }
  
  _add.toString = _add.valueOf = function() {
    return num;
  }
  
  return _add;
}
```