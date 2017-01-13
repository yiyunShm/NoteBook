### 设计更好的 Javascript API
原文链接：[Designing Better JavaScript APIs](http://coding.smashingmagazine.com/2012/10/09/designing-javascript-apis-usability/)

Perter Drucker曾经说过："计算机是白痴"。我们的代码是为了让他人能更有效的理解和使用，而不是仅仅让机器运行。

### 目录
* [连贯接口](#连贯接口)
* [一致性](#一致性)
* [处理参数](#处理参数)
* [可拓展性](#可拓展性)
* [钩子机制](#钩子机制)
* [生成访问器](#生成访问器)
* [可怕的引用](#可怕的引用)
* [连续性问题](#连续性问题)
* [错误处理](#错误处理)
* [向异步出发](#向异步出发)
* [调试连贯接口](#调试连贯接口)
* [文档化API](#文档化API)
* [结语](#结语)

--------------------------------------------------------------------------------
#### 连贯接口
连贯接口通常被称为*链式调用* (尽管不全对)。对于初学者而言它看上去像*jQuery风格*。

##### 链式调用
链式调用的主要思想就是使代码尽可能流畅易读，从而可以更快地被理解。有了链式调用，我们可以将代码 组织为类似语句的片段，增加可读性的同时减少干扰。
```
// regular API
var elem = document.getElementById('footbar');
elem.style.background = 'red';
elem.style.color = 'green';
elem.addEventListener('click', function(evt){
	alert('hello world');
});

// method chaining API
DOMHelper.getElementById('footbar')
	.background('red')
	.color('green')
	.addEvent('click', function(evt) {
    	alert('hello world');
	});
```

##### 命令查询分离
命令查询分离(Command and Query Separation, CQS)是源于命令式编程的一个概念。那些改变对象的状态(内部的值)的函数称为*命令*，而那些检索值的函数成为*查询*。原则上，查询函数返回数据，命令函数返回状态。这个概念在大部分库中都可以看到对应的getter和setter方法。但因为*连贯接口*返回自引用以实现链式方法调用，事实上已经打破了为*命令*设定的规则，因为它们本来不应有返回值。除了这一点(很容易被忽略)以外，我们还(有意)打破这个概念从而使API尽可能保持简单。
```
var $elem = $('#footer');
// CQS - command
$elem.setCss('background', 'green');
// CQS - query
$elem.getCss('background');

// non-CQS - command
$elem.css('background', 'green');
// non-CQS - query
$elem.css('background');
```
这样，getter和setter都被合并到一个单一的方法中，由传入的参数来决定要执行的功能。这使得我们可以暴露更少的方法，从而以更少的代码实现相同的目标。

将getter和setter方法压缩到单一方法中以创建连贯接口并不是必要的，这取决于个人喜好。而且，多函数签名的文档化可能会比较麻烦，这在下面会提到。

##### 向流畅前进
虽然方法链已经为实现流畅的代码完成了大量的工作，但还不够。我们假设要写一个处理日期的简短的库。一个日期间隔开始于某个日期，结束于另一个日期。一个日期并不必要与另一个日期相关联。于是我们得出这个简单的构造器。
```
// create new date intervalar
interval = new DateInterval(startDate, endDate);
// get the calculated number of days the interval spanvar
days = interval.days();
```
虽然看上去是对的，下面这个例子可以看出问题所在：
```
var startDate = new Date(2012, 0, 1);
var endDate = new Date(2012, 11, 31);
var interval = new DateInterval(startDate, endDate);
var days = interval.days();
```
我们为了初始化`DateInterval`而声明了一大堆并不需要的变量和其他东西。更好的解决方案是在Date对象上添加一个函数来返回一个时间间隔。
```
// DateInterval creator for fluent invocation
Date.prototype.until = function(end) {
	// if we weren't given a date, make one
	if(!(end instanceof Date)) {
      end = Date.apply(null, Array.prototype.slice.call(arguments, 0));
	}
	
	return new DateInterval(this, end);
}

var startDate = new Date(2012, 0, 1);
var interval = startDate.until(2012, 11, 31);
var days = interval.days();

// condensed fluent interface call
var days = (new Date(2012, 0, 1))
	.until(2012, 11, 31)
	.days();
```
在最后的例子中我们可以看到，只需要声明更少的变量就可以完成同样的功能，并且执行语句就像一个英语句子。通过这个例子，你会意识到，方法链只是连贯接口的一部分。我们应该要明白代码流——从哪里来，要往哪里去。

但还有一个问题是，上面的例子拓展了原生对象来说明流畅性。和使用分号一样，拓展内置原生对象有它自己的优缺点，这个问题见仁见智。但保持一致性是人人都认可的原则。顺便说一句，及时是那些"不应以自定义方法污染原生对象"的拥护者可能也会接受下面有些技巧性的代码：
```
String.prototype.foo = function(){
  return new Foo(this);
}

"I'm a native object".foo()
	.iAmACustomFunction();
```
通过这种方式，你的自定义函数仍然在你的命名空间下，但是可以通过其他对象访问到他。确保你的代码中`.foo()`对应的方法名是非统称(non-generic)关键字，以避免与其他的API冲突，并确保你的代码中提供了恰当的`.valueOf()`和`.toString()`方法以转换回原始的基本类型。

#### 一致性
Jake Archibald曾经在一张幻灯片上定义了*一致性* [Resuable Code - For Good or For Awesome](http://www.slideshare.net/slideshow/embed_code/5426258?startSlide=59)。永远不要在你的代码中出现类似str_repeat()、str_pos()、substr()这样的函数命名，也不要交换参数位置。如果你在某处声明了`find_in_array(haystack, needle)`，再定义`findInString(needle, haystack)`，这将会使你的代码变得像噩梦一般。

##### 命名
> "There are only two hard problems in computer science: cache-invalidation and naming things." ------- Phil Karlton

在艺术作品中，一致性是一个作品背后不可缺少的观念，或者说设计者如何把一些事物组成连贯的一个整体。协调性，从另一个方面来说，是一个作品相似的布局，这会在考虑整体时产生一种简洁的感觉。

对于与API设计者而言，这些原则可以通过函数命名和参数位置设定的统一来实现：找到一个语义相近的词可以来描述你的函数，并且坚持使用下去——即使在将来某个时候你开始反感这个风格了。
```
'hello world'.slice(start, end);
['a', 'b'].slice(start, end);
```
统一和协调的目的是让API新手感觉熟悉和舒服。虽然每个函数的功能不同，但是语法相同或相似，使API变得熟悉，大大减轻了开发者使用新工具的负担。

##### 处理参数
你的方法如何接受数据比让它们具有可链性更为重要。虽然方法链是非常普遍的，你可以很容易地在你的代码中实现，但是处理参数却不同。你需要想想，你提供的方法最有可能被如何使用。调用你的API的代码会不会重复调用某个函数？为什么会重复调用？如何使你的API帮助开发者减少这种重复调用函数的干扰？
```
jQuery('#some-selector')
	.css('background', 'red')
	.css('color', 'white')
	.css('font-weight', 'bold')
	.css('padding', 10);
	
// receive a map
jQuery('#some-selector').css({
    'background': 'red',
    'color': 'white',
    'font-weight': 'bold',
    'padding': 10
});
```
jQuery的`.on()`方法可以注册事件处理器。和`.css()`一样它也可以接受一组映射格式的事件，但更进一步地，它允许单一处理器可以被多个事件注册：
```
jQuery('#some-selector').on({
  'click': myClickHandler,
  'keyup': myKeyupHandler,
  'change': myChangeHandler
});

jQuery('#some-selector').on('click keyup change', myEventHandler);
```
可以使用下面的*方法模式* 实现上面的函数签名：
```
DateInterval.prototype.values = function(name, value) {
  var map;
  if (jQuery.isPlainObject(name)) {
    map = name;
  } else if (value !== undefined) {
    keys = name.split(' ');
    map = {};
    for (var i = 0, len = keys.length; i < len; i += 1) {
      map[keys[i]] = value;
    }
  } else if (name === undefined) {
    return this.values;
  } else {
    return this.values[name];
  }
  
  for (var key in map) {
    this.values[name] = map[key];
  }
  
  return this;
}
```
如果你需要处理集合，考虑一下你可以为减少API使用者可能需要执行的循环次数做些什么。假设我们有一堆想要设置默认值的`<input>`元素：
```
<input type="text" value="" data-default="foo">
<input type="text" value="" data-default="bar">
<input type="text" value="" data-default="baz">
```
我们也许会以这样一个循环实现：
```
jQuery('input').each(function(){
  var $this = jQuery(this);
  $this.val($this.data('default'));
});
```
如果我们可以绕过这种方式，采用一个简单的回调函数应用到集合中的每一个`<input>`元素呢？jQuery开发者已经想到这点并且允许我们写更少的代码：
```
jQuery('input').val(function(){
  return jQuery(this).data('default');
});
```
正是像这些接收映射参数、回调函数或序列化的属性名的细节，让你的API使用起来不仅更清晰，而且舒服和高效。显然并非你所有的API方法都会从这种方法中收益——何时这样做有意义，何时这样做是浪费时间，都完全取决于你。尽可能人性化地在这方面保持一致。

##### 处理类型
通常定义一个含参函数，你需要决定这个函数接受的参数类型。一个计算两个日期之间间隔的天数的函数会是像这样：
```
DateInterval.prototype.days = function(start, end) {
  return Math.floor((start - end) / 86400000);
}
```
可见，这个函数参数类型是数字——准确来说，一个微秒级的时间戳。尽管这个函数完成了我们所预期的效果，但它还不够通用。如果我们得处理`Date`对象或是代表日期的字符串这样的参数呢？难道使用者每次调用这个函数前都必须先转换数据格式吗？不，只需要集中地对输入进行验证并且转换为我们需要的格式，而不是将参数处理凌乱地分散在调用API的代码中：
```
DateInterval.prototype.days = function(start, end) {
  if (!(start instanceof Date)) {
    start = new Date(start);
  }
  
  if (!(end instanceof Date)) {
    end = new Date(end);
  }
  
  return Math.floor((end.getTime() - start.getTime()) / 86400000);
}
```
有经验的开发者会在上面的实例代码中注意到一个问题，假定start在end日期之前。如果API使用者不小心交换了这两个日期的传入函数，就会得到一个负的日期间隔。如果觉得不合理，那就修复它：
```
DateInterval.prototype.days = function(start, end) {
  if (!(start instanceof Date)) {
    start = new Date(start);
  }
  
  if (!(end instanceof Date)) {
    end = new Date(end);
  }
  
  return Math.abs(Math.floor((end.getTime() - start.getTime()) / 86400000));
}
```
Javascript允许多种形式的类型转换。如果你需要处理基本类型(字符串、数字、布尔型)，这种转换可以如此简单：
```
function castaway(some_string, some_integer, some_boolean) {
  some_string += '';
  some_integer += 0;
  some_boolean = !!some_boolean;
}
```

##### 把`UNDEFINED`看作预期值
有时候你的API事实上期望获得一个`undefined`值来设置一个属性值，可能是为了将一个属性值设为"未置值(unset)"状态，也可能只是优雅地处理错误输入使你的API更加健壮。为了确定`undefined`是不是确实被传入到你的方法中，你可以检查`arguments`对象：
```
function testUndefined(expecting, moreArgument) {
  if (moreArgument === undefined) {
    console.log('moreArgument was undefined');
  }
  if (arguments.length > 1) {
    console.log('but was actually passed in');
  }
}

testUndefined('foo');
// => moreArgument was undefined
testUndefined('foo', undefined);
// => moreArgument was undefined but was actually passed in
```

##### 命名参数
```
event.initModuleEvent('click', true, true, window, 132, 101, 202, true, false, 1, null);
```
`Event.initModuleEvent`的函数签名像是噩梦成真。开发者绝无可能不用查看文档就能想起来每一个参数表示什么。不管你的文档写的多么好，尽你所能让人们不必去查阅它！

##### 其他语言的传参
在类似python/PHP中有命名参数的概念，它允许你在声明一个函数时为参数提供默认值，允许你在调用上下文中声明属性名：
```
function nameAreAwesome(foo=1, bar=2) {
  console.log(foo, bar);
}

nameAreAwesome();
// prints: 1, 2

nameAreAwesome(3, 4)
// prints: 3, 4
```

##### 参数映射
Javascript 不是Python(而且ES.next还很遥远)，要克服"参数森林"的障碍，留给我们的可选方案非常少。jQuery(以及差不多它提高的每一个恰当的API)采用了"option 对象(option objects)"的概念。`jQuery.ajax()`方法签名提供了一个很好的例子。我们只需要传入一个对象，而不是一堆参数：
```
function nightmare(accepts, async, beforeSend, cache, complete, /* and 28 more */) {
  if (accepts === 'text') {
    // prepare for receiving plain text
  }
}

function dream(options) {
  options = options || {};
  if (options.accepts === 'text') {
    // prepare for receiving plain text
  }
}
```
这样不仅避免了疯狂而冗长的函数前面，也使得函数调用更具备描述性：
```
nightmare('text', true, undefined, false, undefined, /* and 28 more */);

dream({
  accepts: 'text',
  async: true,
  cache: false
});
```
此外，在更新的版本中如果我们会引入新的特性，也不必影响到函数签名(添加新的参数)。

##### 默认参数值
jQuery.extend()等可以帮你合并对象，允许你输入预置的option对象进行合并：
```
var default_options = {
  accepts: 'text',
  async: true,
  beforeSend: null,
  cache: false,
  complete: null,
  // ...
};

function dream(options) {
  var o = jQuery.extend({}, default_options, options || {});
  console.log(o.accepts);
}

// make defaults public
dream.default_options = default_options;
dream({ async: false });
```
默认值可以公开访问了，恭喜你拿到了附加分。这样一来，任何人都可以在集中地修改accepts的值为"json"，因而可以避免一再地指定这个选项。注意这个例子中总是会在初次读取选项对象时附加一个`|| {}`操作，从而可以保证无参传入时也能调用这个函数。

##### 好意——也可能是"陷阱"
> "With great power comes great responsibility!"------------Voltaire

类似大部分弱类型语言，Javascript在需要时会自动进行类型转换。一个简单的例子是测试真假与否：
```
var foo = 1;
var bar = true;

if (foo) {
  //yep, this will excute
}

if (bar) {
  // yep, this will excute
}
```
