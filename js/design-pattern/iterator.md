[TOC]

## 迭代器模式

### 介绍
迭代器模式提供一种方法顺序一个聚合对象中各个元素，而又不暴露该对象内部表示。对象的消费者并不需要知道如何组织数据，所有需要做的就是取出单个数据进行工作。

### 基本实现
在迭代器模式中，对象需要提供一个 `next()` 方法。一次调用 `next()` 必须返回下一个连续的元素。当然，在特定数据结构中，"下一个"所代表的意义由你来决定。

假定对象名为agg，可以在类似下面这样的一个循环中通过简单调用 `next()` 即可访问每个数据元素：
```
var element;
while (element = agg.next()) {
  // do something...
  console.log(element);
}
```
在迭代器模式中，聚合对象通常还提供一个较为方便的 `hasNext()` 方法，因此，该对象的用户可以为使用该方法来确定是否已经到达了数据的末尾。此外，还有另一种顺序访问所有元素的方法，这次是使用 `hasNext()`，其用法如下所示：
```
while (agg.hasNext()) {
  // do something...
  console.log(agg.next());
}
```
当实现迭代器模式时，私下的存储数据和指向下一个可用元素的指针(索引)是很有意义的。为了演示一个实现示例，让我们假定数据只是普通数组，而"特殊"的检索下一个连续元素的逻辑为返回每隔一个的数组元素。
```
var agg = (function() {
  var index = 0,
      data = [1, 2, 3, 4, 5],
      length = data.length;
      
  return {
    next: function() {
      var element;
      if (!this.hasNext()) {
        return null;
      }
      element = data[index];
      index = index + 2;
      return element;
    },
    hasNext: function() {
      return index < length;
    }
  }
})();
```
为了提供更简单的访问方式以及多次迭代数据的能力，你的对象可以提供额外的便利方法：
* `rewind()` 重置指针到初始位置
* `current()` 返回当前元素，因为不可能在不前进的指针情况下使用 `next()` 执行该操作

```
var agg = (function() {
  // [snip..]
  return {
    // [snip..]
    rewind: function() {
      index = 0;
    },
    current: function() {
      return data[index];
    }
  }
}());
```
### 构造函数实现
该模式可以为传入数据生成迭代器，注入不同的数据，选择不同的递增步长：
```
var Agg = function(options) {
  this.data = options.data;
  this.step = options.step;
  this.index = 0;
  
  return this;
};

Agg.prototype = {
  constructor: Agg,
  next: function() {
    var element;
    if (!this.hasNext()) {
      return null;
    }
    
    element = this.current();
    this.index += this.step;
    
    return element;
  },
  hasNext: function () {
    return this.index < this.data.length;
  },
  rewind: function () {
    this.index = 0;
    return this.current();
  },
  current: function () {
    return this.data[this.index];
  }
}
```

### 总结
迭代器的使用场景是：对于集合内部结果常常变化各异，我们不想暴露其内部结构的话，但又想让客户代码透明的访问其中元素，这种情况下我们可以使用迭代器模式。