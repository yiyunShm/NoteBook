[TOC]

## Javascript事件循环

### 单线程
Javascript语言的一大特点就是单线程，也就是说，同一个时间只能做一件事，这与它的用途有关。

作为浏览器语言，Javascript的主要用途是与用户互动，以及DOM操作。这决定了它只能是单线程，否则会带来很复杂的同步问题。比如，假定Javascript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时，浏览器应该以哪个线程为准？

所以，为了避免复杂性，从一诞生，Javascript就是单线程，这已经成了这门语言的核心特征，将来也不会改变。

为了利用多核CPU的计算能力，HTML5提出Web Worker标准，允许Javascript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM。所以这个标准并没有改变Javascript单线程的本质。

### 任务队列
单线程就意味着，主线程只会做一件事情，就是从任务队列里面取任务、执行任务，再取任务、再执行。当任务队列为空时，就会等待直到任务队列变成非空。而且主线程只有在将当前的任务执行完成后，才会去取下一个任务。这种机制就叫做事件循环机制，取一个任务并执行的过程叫做一次循环。

为了解决IO设备的缓慢问题，所有任务又可以分成两种:
* 同步任务: 在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务。
* 异步任务: 不进入主线程、而进入"任务队列"（task queue）的任务，只有"任务队列"通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

具体来说，异步执行的运行机制如下:
1. 所有同步任务都在主线程上执行，形成一个执行栈
2. 主线程之外，还存在一个"任务队列"。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件
3. 一旦执行栈中的所有同步任务执行完毕，系统就会读取"任务队列"，并获取事件对应的异步任务，于是结束等待状态，进入执行栈，开始执行。
4. 主线程重复以上三步。

![Task Queue](https://github.com/yiyunShm/NoteBook/tree/master/js/base/images/task_queue.jpg)

>一个浏览器环境(unit of related similar-origin browsing contexts)只能有一个事件循环(Event loop)，而一个事件循环可以有多个任务队列(Task queue)，每个任务都有一个任务源(Task source)
>
>相同任务源的任务，只能放到一个任务队列中。
>
>不同任务源的任务，可以放到不同任务队列中。
>
>单独的任务队列中的任务总是按先进先出的顺序执行，但是不保证多个任务队列中的任务优先级，具体实现可能会交叉执行。

### 事件和回调函数
"任务队列"是一个事件的队列(也可以理解成消息的队列)，IO设备完成一项任务，就在"任务队列"中添加一个事件，表示相关的异步任务可以进入"执行栈"了。主线程读取"任务队列"，就是读取里面有哪些事件。

"任务队列"中的事件，除了IO设备的事件以外，还包括一些用户产生的事件(比如鼠标点击、页面滚动等等)。只要指定过回调函数，这些事件发生时就会进入"任务队列"，等待主线程读取。

所谓"回调函数"(callback)，就是那些会被主线程挂起来的代码。异步任务必须指定回调函数，当主线程开始执行异步任务，就是执行对应的回调函数。

"任务队列"是一个先进先出的数据结构，排在前面的事件，优先被主线程读取。主线程的读取过程基本上是自动的，只要执行栈一清空，"任务队列"上第一位的事件就自动进入主线程。但是，由于存在后文提到的"定时器"功能，主线程首先要检查一下执行时间，某些事件只有到了规定的时间，才能返回主线程。

### Event Loop
主线程从"任务队列"中读取事件，这个过程是循环不断的，所以整个的这种运行机制又称为Event Loop（事件循环）。

为了更好地理解Event Loop，请看下图(转引自Philip Roberts的演讲[Help, I'm stuck in an event-loop](http://vimeo.com/96425312))。

![Event Loop](https://github.com/yiyunShm/NoteBook/tree/master/js/base/images/event_loop.png)

上图中，主线程运行的时候，产生堆(heap)和栈(stack)，栈中的代码调用各种外部API，它们在"任务队列"中加入各种事件(click, load, done)。只要栈中的代码执行完毕，主线程就会去读取"任务队列"，依次执行那些事件所对应的回调函数。

### setTimeout
setTimeout()方法不是ecmascript规范定义的内容，而是属于BOM提供的功能。查看w3school的定义，setTimeout() 方法用于在指定的毫秒数后调用函数或计算表达式。
```
setTimeout(fn, millisec)
```
其中fn表示要执行的代码，可以是一个包含javascript代码的字符串，也可以是一个函数。第二个参数millisec是以毫秒表示的时间，表示fn需推迟多长时间执行。

调用setTimeout()方法之后，该方法返回一个数字，这个数字是计划执行代码的唯一标识符，可以通过它来取消超时调用。

HTML5标准规定了setTimeout()的第二个参数的最小值（最短间隔），不得低于**4毫秒**，如果低于这个值，就会自动增加。在此之前，老版本的浏览器都将最短间隔设为10毫秒。另外，对于那些DOM的变动（尤其是涉及页面重新渲染的部分），通常不会立即执行，而是每16毫秒执行一次。这时使用requestAnimationFrame()的效果要好于setTimeout()。

需要注意的是，setTimeout()只是将事件插入了"任务队列"，必须等到当前代码（执行栈）执行完，主线程才会去执行它指定的回调函数。要是当前代码耗时很长，有可能要等很久，所以并没有办法保证，回调函数一定会在setTimeout()指定的时间执行。

### Node.js的Event Loop
Node.js也是单线程的Event Loop，但是它的运行机制不同于浏览器环境。

![Node System](https://github.com/yiyunShm/NoteBook/tree/master/js/base/images/node_system.png)

根据上图，node.js的运行机制如下：
1. V8引擎解析Javascript脚本。
2. 解析后的代码，调用Node API。
3. Libuv库负责Node API的执行，它将不同的任务分配给不同的线程，形成一个Event Loop，以异步的方式将任务的执行结果返回给V8引擎。
4. V8引擎再将结果返回给用户。

除了setTimeout和setInterval这两个方法，Node.js还提供了另外两个与"任务队列"有关的方法：process.nextTick和setImmediate。它们可以帮助我们加深对"任务队列"的理解。

process.nextTick方法可以在当前"执行栈"的尾部----下一次Event Loop（主线程读取"任务队列"）之前----触发回调函数。也就是说，它指定的任务总是发生在所有异步任务之前。setImmediate方法则是在当前"任务队列"的尾部添加事件，也就是说，它指定的任务总是在下一次Event Loop时执行，这与setTimeout(fn, 0)很像。
```
process.nextTick(function A() {
  console.log(1);
  process.nextTick(function B(){console.log(2);});
});

setTimeout(function timeout() {
  console.log('TIMEOUT FIRED');
}, 0)
// 1
// 2
// TIMEOUT FIRED
```
上面代码中，由于process.nextTick方法指定的回调函数，总是在当前"执行栈"的尾部触发，所以不仅函数A比setTimeout指定的回调函数timeout先执行，而且函数B也比timeout先执行。这说明，如果有多个process.nextTick语句（不管它们是否嵌套），将全部在当前"执行栈"执行。

现在，再看setImmediate:
```
setImmediate(function A() {
  console.log(1);
  setImmediate(function B(){console.log(2);});
});

setTimeout(function timeout() {
  console.log('TIMEOUT FIRED');
}, 0);
```
上面代码中，setImmediate与setTimeout(fn,0)各自添加了一个回调函数A和timeout，都是在下一次Event Loop触发。那么，哪个回调函数先执行呢？答案是不确定。运行结果可能是1--TIMEOUT FIRED--2，也可能是TIMEOUT FIRED--1--2。
>node中的底层loop有3种运行模式，DEFAULT, UV_RUN_ONCE,UV_RUN_NOWAIT具体差异详见libuv文档 。 刚启动node解析执行脚本时以UV_RUN_ONCE的模式执行也就是说第一次loop是UV_RUN_ONCE模式，在每一个loop中先检测timers，然后进入io_poll此时会阻塞等待事件发生，事件发生后执行setImmediate中的回调，当以UV_RUN_ONCE模式执行的时候会再次检测timers 。照这个原理来说，setImmediate应该比setTimeout先执行。
>
>setTimeout(fn,0) 底层实际执行的是 setTimeout(fn,1), 此时会向loop队列中注册一个超时事件，假设注册时当前时间戳是1000，它的过期时间就是1001。按上面的说法1ms 第一次loop执行完后先执行setImmediate回调，再执行timer回调。但是有一点别忘了，您的计算机此时还有其他任务在处理，cpu不是全部用来跑你的node进程的，也就是说虽然cpu很快，但执行指令总得花点时间吧，从setTimeout(fn,0)注册一个1ms的超时事件到第一次执行timers可能就已经花了1ms的时间，此时系统时间已经到1001了，那么对应的timer回调当然就会先于setImmediate执行，所以说Immediate 和 Timeout 的先后顺序不确定。

令人困惑的是，Node.js文档中称，setImmediate指定的回调函数，总是排在setTimeout前面。实际上，这种情况只发生在递归调用的时候。
```
setImmediate(function (){
  setImmediate(function A() {
    console.log(1);
    setImmediate(function B(){console.log(2);});
  });

  setTimeout(function timeout() {
    console.log('TIMEOUT FIRED');
  }, 0);
});
// 1
// TIMEOUT FIRED
// 2
```
上面代码中，setImmediate和setTimeout被封装在一个setImmediate里面，它的运行结果总是1--TIMEOUT FIRED--2，这时函数A一定在timeout前面触发。至于2排在TIMEOUT FIRED的后面（即函数B在timeout后面触发），是因为setImmediate总是将事件注册到下一轮Event Loop，所以函数A和timeout是在同一轮Loop执行，而函数B在下一轮Loop执行。

我们由此得到了process.nextTick和setImmediate的一个重要区别：多个process.nextTick语句总是在当前"执行栈"一次执行完，多个setImmediate可能则需要多次loop才能执行完。事实上，这正是Node.js 10.0版添加setImmediate方法的原因，否则像下面这样的递归调用process.nextTick，将会没完没了，主线程根本不会去读取"事件队列"！

另外，由于process.nextTick指定的回调函数是在本次"事件循环"触发，而setImmediate指定的是在下次"事件循环"触发，所以很显然，前者总是比后者发生得早，而且执行效率也高(因为不用检查"任务队列")。

### 参考链接
* [JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)
* [Promise && setTimeout](https://www.zhihu.com/question/36972010)
* [关于任务和微任务的入队与调度](https://github.com/rowone/blog/issues/3)