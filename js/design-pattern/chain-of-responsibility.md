[TOC]

## 职责链模式

### 介绍
职责链模式可以用来消除请求的发送者和接收者之间的耦合关系。这是通过一个由隐式地对请求进行处理的对象组成的链而做到的。链中的每个对象可以处理请求，也可以将其传给下一个对象。

### 基本实现
职责链由多个不同类型的对象组成。发送者是发出请求的对象，而接收者则是链中那些接收这种请求并且对其进行处理或传递的对象。请求本身有时也是一个对象，它封装着与操作有关的所有数据，其典型的运转流程大概是：
* 发送者知道链中的第一个接收者，它向这个接收者发出请求。
* 每一个接收者都对请求进行分析，然后要么处理它，要么将其往下传。
* 每一个接收者知道的其它对象只有一个，即它在链中的下家(successor)。
* 如果没有任何接收者处理请求，那么请求将从链上离开。不同的实现对此有不同的反应，既可能无声无息，也可能抛出一个错误。

```
var NO_TOPIC = -1;
var Topic;

function Handler(s, t) {
  this.successor = s || null;
  this.topic = t || 0;
}

Handler.prototype = {
  handle: function() {
    if (this.successor) {
      this.successor.handle();
    }
  },
  has: function() {
    return this.topic != NO_TOPIC;
  }
};
```
Handler只是接受2个参数，第一个是继任者(用于将处理请求传下去)，第二个是传递层级(可以用于控制在某个层级下是否执行某个操作，也可以不用)，Handler原型暴露了一个handle方法，这是实现该模式的重点，先来看看如何使用上述代码：
```
var app = new Handler({
  handle: function() {
    console.log('app handle');
  }
}, 3);

var dialog = new Handler(app, 1);
dialog.handle = function() {
  console.log('dialog before ...');
  // do something
  console.log('dialog after ...');
};

var button = new Handler(dialog, 2);

button.handle();
```
因为在dialog处改写了 `handle()`，该代码的执行结果即是 `dialog.handle()` 里的处理结果，而不在是给app传入的参数里定义的handle的执行操作。

那能不能做到自身处理完以后，然后在让继任者继续处理呢？答案是肯定的，就是在调用的handle以后，需要利用原型的特性调用如下的代码：
```
Handler.prototype.handle.call(this);
```
该句话的意思是说，调用原型的handle方法，来继续调用其继任者(也就是successor)的handle方法，以下代码表现为：button/dialog/app三个对象定义的handle都会执行。
```
var app = new Handler({
    handle: function () {
        console.log('app handle');
    }
}, 3);

var dialog = new Handler(app, 1);
dialog.handle = function () {
    console.log('dialog before ...')
    // 这里做具体的处理操作
    Handler.prototype.handle.call(this); //继续往上走
    console.log('dialog after ...')
};

var button = new Handler(dialog, 2);
button.handle = function () {
    console.log('button before ...')
    // 这里做具体的处理操作
    Handler.prototype.handle.call(this);
    console.log('button after ...')
};

button.handle();
```
通过代码的运行结果我们可以看出，如果想先自身处理，然后再调用继任者处理的话，就在末尾执行 `Handler.prototype.handle.call(this);` 代码，如果想先处理继任者的代码，就在开头执行 `Handler.prototype.handle.call(this);` 代码。

### 总结
借助职责链模式，可以动态选择由哪个对象处理请求，这意味着你可以使用只有在允许期间才能知道的条件来把任务分派给最恰当的对象。

在已经有现成的链或层次体系的情况下，职责链模式更加有效。与组合模式的结合使用就属于这种情况，你可以重用组合对象的结构来传递请求，直到找到一个可以处理请求的对象。在此情况下，不用编写粘合性代码来实例化那些对象或建立链，因为那些东西早已准备妥当。籍此可以实现把请求转给恰当的处理程序的方法。