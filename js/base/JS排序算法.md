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

### 排序算法思路和实现
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

#### 归并排序
* 归并：从两个有序表R[low...mid]和R[mid+1...high]，每次从左边依次去除一个数进行比较，将较小者放入tmp数组中，最后将两段中剩下的部分直接复制到tmp中。这样，tmp是一个有序表，再将它复制到R中。(其中要考虑最后一个子表的长度不足length的情况)
* 排序：自底向上的归并，第一回：length = 1; 第二回：length = 2 \* length...

```
Array.method('merge', function(low, mid, high) {
  var tmp = new Array(), i = low, j = mid + 1, k = 0;
  while (i <= mid && j <= high) {
    if (this[i] <= this[j]) {
      tmp[k] = this[i];
      i++;
      k++;
    } else {
      tmp[k] = this[j];
      j++;
      k++;
    }
  }
  
  while (i <= mid) {
    tmp[k] = this[i];
    i++;
    k++;
  }
  
  while (j <= high) {
    tmp[k] = this[j];
      j++;
      k++;
  }
  
  for(k = 0, i = low; i < high; k++, i++) {
    this[i] = tmp[k];
  }
  return this;
});

Array.method('mergePass', function(length, n) {
  var i;
  for (i = 0; i + 2 * length - 1 < n; i = i + 2 * length) {
    this.merge(i, i + length - 1, i + 2 * length - 1);
  }
  
  if (i + length - 1 < n) {
    this.merge(i, i + length - 1, n - 1);
  }
  return this;
});

Array.method('mergeSort', function() {
  var len = this.length,
      length;
  
  for (length = 1; length < len; length = 2 * length) {
    this.mergePass(length, len);
  }
  
  return this;
});
```

#### 希尔排序
我们在第i次时取gap = (n  / 2)的i次方，然后将数组分为gap组(从下标0开始，每相邻的gap个元素为一组)，接下来我们对每一组进行直接插入排序。
```
Array.method('shellSort', function(){
  var len = this.length, gap = parseInt(len/2), 
      i, j, tmp;
      
  while(gap > 0){
    for(i=gap; i<len; i++){
      tmp = this[i];
      j = i - gap;
      while(j>=0 && tmp < this[j]){
        this[j+gap] = this[j];
        j = j - gap;
      }
      this[j + gap] = tmp;
    }
    gap = parseInt(gap/2);
  }
  
  return this;
});
```