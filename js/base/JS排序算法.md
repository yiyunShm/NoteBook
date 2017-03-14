[TOC]

## JS排序算法

### 基础知识
* JS原型
* 排序中的有序区和无序区
* 二叉树的基本知识

### 封装基本方法
添加一个方法，下面的排序中会使用到
```
Function.prototype.method = function(name, func) {
  this.prototype[name] = func;
  return this;
}
```

### 9种排序算法思路和实现
#### 插入排序
从无序区的第一个元素开始和它前面的有序区的元素进行比较，如果比前面的元素小，那么前面的元素向后移动，否则就将此元素插入到相应的位置。
```
Array.method('insertSort', function() {
  var len = this.length,
      i, j, temp;
      
  for(i = 1; i < len; i += 1) {
    tmp = this[i];
    j = i - 1;
    while(j >= 0 && tmp < this[j]) {
      this[j + 1] = this[j];
      j--;
    }
    this[j + 1] = tmp;
  }
  
  return this;
});
```

#### 二分插入排序
先在有序区通过二分查找法找到元素的移动位置，然后通过这个起始位置将后面所有元素后移。
```
Array.method('bInsertSort', function() {
  var len = this.length,
      i, j, tmp, low, high, mid;
      
  for (i = 1; i < len; i += 1) {
    tmp = this[i];
    low = 0;
    high = i - 1;
    // get position
    while (low <= high) {
      mid = parseInt((low + high) / 2);
      if (tmp < this[mid]) {
        high = mid - 1;
      } else {
        low = mid + 1;
      }
    }
    // move elems
    for (j = i - 1; j > high + 1; j -= 1) {
      this[j + 1] = this[j];
    }
    this[j + 1] = tmp;
  }
  
  return this;
});
```

#### 冒泡排序
通过在无序区的相邻元素的比较和替换，使较小的元素浮到最上面
```
Array.method('bubbleSort', function() {
  var len = this.length,
      i, j, tmp;
      
  for (i = 0; i < len; i += 1) {
    for (j = len - 1; j > i; j -= 1) {
      if (this[j] > this[j - 1]) {
        tmp = this[j];
        this[j] = this[j - 1];
        this[j - 1] = tmp;
      }
    }
  }
  
  return this;
});
```

#### 改进的冒泡排序
如果在某次的排序中没有出现交换的情况，那么说明在无序的元素以及是有序的了，就可以直接返回。
```
Array.method('rBubbleSort', function() {
  var len = this.length,
      i, j, tmp, exchange;
      
  for (i = 0; i < len; i += 1) {
    exchange = 0;
    for (j = len - 1; j > i; j -= 1) {
      if (this[j] < this[j - 1]) {
        tmp = this[j];
        this[j] = this[j - 1];
        this[j - 1] = tmp;
        
        exchange = 1;
      }
    }
    
    if (!exchange) break;
  }
  
  return this;
})
```

#### 快速排序
1. 假设第一个元素为基准元素
2. 把所有比基准元素小的放在左边，把所有比基准元素大的放在右边，并把基准元素放在两部分中间(i = j)的位置

```
Array.method('quickSort', function(s, t) {
  if (this.length <= 1) return this;
  
  var left = [], right = [],
      base = this.splice(0, 1)[0],
      len = this.length,
      i, tmp;
      
  for (i = 0; i < len; i += 1) {
    tmp = this[i];
    if (tmp < base) {
      left.push(tmp);
    } else {
      right.push(tmp);
    }
  }
  
  return left.quickSort().concat(base, right.quickSort());
});
```

#### 选择排序
在无序区中选出最小的元素，然后将它和无序区的第一个元素交换
```
Array.method('selectSort', function() {
  var len = this.length,
      i, j, k, tmp;
      
  for (i = 0; i < len; i += 1) {
    for (j = i + 1; j < len; j += 1) {
      tmp = this[i];
      if (tmp > this[j]) {
        this[i] = this[j];
        this[j] = tmp;
      }
    }
  }
  
  return this;
});
```