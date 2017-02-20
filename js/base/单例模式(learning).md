[TOC]

## 单体模式

### 介绍
单体(Singleton)模式的思想在于保证一个特定类仅有一个实例。这意味着当你第二次使用同一个类创建新对象的时候，应该得到与第一次创建对象完全相同的对象。

### 基本实现
但是，在Javascript中没有类，只有对象。当你创建一个新对象时，实际上没有其他对象与其类似，因此新对象已经是单体了。使用对象字面量创建一个简单的对象也是一个单体的例子。
```
var obj = {
  myprop: 'my value'
}
```
如果在以后要拓展该对象，你可以添加自己的私有成员和方法，然后使用闭包在其内部封装这些变量和声明，对外只暴露你想要暴露的public成员。
```
var obj = (function() {
  // private variable
  var _prop = 'something private';
  // pirvate method
  var _method = function() {
    // do something
  };
  
  // public variable and method
  // is ok to call outside
  return {
    prop: 'something public',
    method: function() {
      // do something
    }
  }
})();
```

### 使用new操作符
Javascript中并没有类，因此对单体咬文嚼字的定义严格来说并没有意义。但是Javascirpt具有new语法可使用构造函数来创建对象，而且有时可能需要使用这种语法的单体来实现。这种思想的重点在于使用同一个构造函数以new操作符来创建多个对象时，应该仅获得指向完全相同的对象的新指针。
```
  // expect uni is equal to uni2 after new action
  var uni = new Universe();
  var uni2 = new Universe();
  
  uni === uni2; // true
```
实现的方法有多种选择：
* 使用全局变量存储该实例，但是不推荐。因为在一般原则下，全局变量是有缺点的，任何人可以自由覆盖。
* 在构造函数的静态属性中缓存该实例。方法很简洁，但缺点在于静态属性是公开可访问的属性，在外部代码中可能被修改导致实例丢失。
* 可以将该实例包装在闭包中。这样可以保证该实例的私有性且不会被构造函数之外的代码修改，代价是带来了额外的内存开销。

#### 静态属性实例
```
function Universe() {
  // make sure prop instance is an object and created by this constructor
  if (typeof Universe.instance === 'object' && Universe.instance instanceof Universe) {
    return Universe.instance;
  }
  
  // do new
  this.start_time = 0;
  this.bang = 'Big';
  
  // cache
  Universe.instance = this;
  
  return this;
}

var uni = new Universe();
var uni2 = new Universe();
uni === uni2; // true
```

#### 闭包中的实例
1.通过覆写构造函数的形式来实现闭包：
```
function Universe() {
  // cache
  var instance = this;
  
  this.start_time = 0;
  this.bang = 'Big';
  
  Universe = function() {
    return instance;
  }
}

var uni = new Universe();
var uni2 = new Universe();
uni === uni2; // true
```

上面的例子存在一个问题，虽然看上去uni === uni2，但实际上因为重写了构造函数，所以构造函数Universe()会丢失所有在重定义以后添加到它里面的属性。可以通过下面的测试看到这个问题：
```
// add prop to Universe
Universe.prototype.nothing = true;

var uni = new Universe();
// add prop to Universe again after uni get created
// in fact, this prop 'everything' has added to new constructor Universe
Universe.prototype.everything = true;

var uni2 = new Universe();

// test
uni.nothing; // true
uni2.nothing; // true

uni.everything; // undefined
uni2.everything; // undefined

uni.constructor.name // 'Universe'
uni.constructor === Universe; // false
```
之所以uni.constructor不再与Universe()构造函数相同，是因为uni.contructor仍然指向了原始的构造函数，而不是重新定义的那个构造函数。

从需求上来说，如果需要使原型和构造函数指针按照预期的那样运行，可以通过一些调整来实现：
```
function Universe() {
  // cache
  var instance;
  
  this.start_time = 0;
  this.bang = 'Big';
  
  Universe = function Universe() {
    return instance;
  }
  // share prop in prototype
  Universe.prototype = this;
  
  instance = new Universe();
  instance.constructor = this.constructor;
  
  return instance;
}
```

2.借助立即函数生成闭包来保存实例：
```
var Universe = (function() {
  var instance;
  
  return function() {
    if (instance) {
      return instance;
    }
    
    instance = this;
    
    this.start_time = 0;
    this.bang = 'Big';
  }
})();

var uni = new Universe();
var uni2 = new Universe();
uni === uni2; // true
```