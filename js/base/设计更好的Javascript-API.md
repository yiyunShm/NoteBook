[TOC]
## 设计更好的 Javascript API
原文链接：[Designing Better JavaScript APIs](http://coding.smashingmagazine.com/2012/10/09/designing-javascript-apis-usability/)

Perter Drucker曾经说过："计算机是白痴"。我们的代码是为了让他人能更有效的理解和使用，而不是仅仅让机器运行。
![Time spent](https://github.com/yiyunShm/NoteBook/blob/master/js/images/time_spent.png)

--------------------------------------------------------------------------------
### 连贯接口
连贯接口通常被称为*链式调用* (尽管不全对)。对于初学者而言它看上去像*jQuery风格*。

#### 链式调用
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

#### 命令查询分离
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

#### 向流畅前进
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

---------------------------------------------------------------------------
### 一致性
Jake Archibald曾经在一张幻灯片上定义了*一致性* [Resuable Code - For Good or For Awesome](http://www.slideshare.net/slideshow/embed_code/5426258?startSlide=59)。永远不要在你的代码中出现类似str_repeat()、str_pos()、substr()这样的函数命名，也不要交换参数位置。如果你在某处声明了`find_in_array(haystack, needle)`，再定义`findInString(needle, haystack)`，这将会使你的代码变得像噩梦一般。

#### 命名
> "There are only two hard problems in computer science: cache-invalidation and naming things." ------- Phil Karlton

在艺术作品中，一致性是一个作品背后不可缺少的观念，或者说设计者如何把一些事物组成连贯的一个整体。协调性，从另一个方面来说，是一个作品相似的布局，这会在考虑整体时产生一种简洁的感觉。

对于与API设计者而言，这些原则可以通过函数命名和参数位置设定的统一来实现：找到一个语义相近的词可以来描述你的函数，并且坚持使用下去——即使在将来某个时候你开始反感这个风格了。
```
'hello world'.slice(start, end);
['a', 'b'].slice(start, end);
```
统一和协调的目的是让API新手感觉熟悉和舒服。虽然每个函数的功能不同，但是语法相同或相似，使API变得熟悉，大大减轻了开发者使用新工具的负担。

--------------------------------------------------------------------------------------
### 处理参数
![Good Intentions](https://github.com/yiyunShm/NoteBook/blob/master/js/images/good_intentions.png)

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

#### 处理类型
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

#### 把`UNDEFINED`看作预期值
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

#### 命名参数
```
event.initModuleEvent('click', true, true, window, 132, 101, 202, true, false, 1, null);
```
`Event.initModuleEvent`的函数签名像是噩梦成真。开发者绝无可能不用查看文档就能想起来每一个参数表示什么。不管你的文档写的多么好，尽你所能让人们不必去查阅它！

#### 其他语言的传参
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

#### 参数映射
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

#### 默认参数值
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

#### 好意——也可能是"陷阱"
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
我们相当习惯了这种自动转换。正是因为太习惯，以至于我们忘记了，即使是有些值是真实存在的，从布尔值的角度它可能并不会被判为true。有些API设计得如此的弹性以至于有些过于聪明了。看看`jQuery.toggle()`方法的签名吧：
```
.toggle(/*int*/[duration] [,/*function*/ callback])
.toggle(/*int*/[duration] [,/*string*/ easing] [,/*function*/ callback])
.toggle(/*bool*/ showOrHide)
```
要解释明白为什么这些行为表现完全不同需要费点时间
```
var foo = 1;
var bar = true;
var $hello = jQuery('.hello');
var $world = jQuery('.world');

$hello.toggle(foo);
$world.toggle(bar);
```
我们的预期是在两种情况下都使用`showOrHide`签名。然而事实上，`$hello`会以一秒的`duration`执行切换。这不是jQuery中的bug，这只是一个与预期不符的案例。即使你是一个有经验的jQuery开发者，你也会不时被这种问题绊倒。

你尽可以如你所愿添加尽可能多的便利——但同时不要牺牲API的简洁性和健壮性。如果你的代码中也提供了类似的API，考虑一下提供一个单独的方法，例如`.toggleIf(bool)`来替代。不论采用什么方法，记得保持你的API的一致性！

----------------------------------------------------------------------
### 可拓展性
![Developing Possibilities](https://github.com/yiyunShm/NoteBook/blob/master/js/images/developing_possibilities.png)

在option对象部分，我们谈到了可拓展的配置。我们来讨论下API使用者拓展核心和API本身。这一点很重要，因为它可以使你的代码关注重要的事情，同时可以使API使用者自己处理边界情况。好的API设计都很简约。提供丰富的配置项当然很好，但是过多的配置项会导致你的API变得臃肿晦涩。关注主要的应用场景，只提供API使用者需要的大部分功能，剩下的应该留给他们决定。为了允许API使用者拓展你的代码以适应他们的需要，你有很多的选择。

#### 回调函数
回调函数可以用来根据配置实现可拓展性。你可以使用回调函数允许API用户覆盖你的代码中的某些部分。当你感觉某些任务可能不会像你提供的默认的代码那样处理，将这些代码重构为一个可配置的回调函数，来允许API使用者易于重载：
```
var default_options = {
  //...
  position: function($elem, $parent) {
    $elem.css($parent.position());
  }
};

function Widget(options) {
  this.options = jQuery.extend({}, default_options, options || {});
  this.create();
}

Widget.prototype.create = function() {
  this.$thingie = $('<div></div>').appendTo(this.$container);
  return this;
};

Widget.prototype.show = function() {
  this.options.position(this.$thingie, this.$container);
  this.$thingie.show();
  return this;
}

var widget = new Widget({
  position: function($elem, $parent) {
    var position = $parent.position();
    // position $elem at the lower right corner of $parent
    position.left += $parent.width();
    position.top += $parent.height();
    $elem.css(position);
  }
});
widget.show();
```
回调函数也是一种常见的允许API使用者定制你的代码创建元素的方式：
```
// default create callback doesn't do anything
default_options.create = function($thingie){};

Widget.prototype.create = function() {
  this.$container = $('<div></div>').appendTo(document.body);
  this.$thingie = $('<div></div>').appendTo(this.$container);
  // execute create callback to allow decoration
  this.options.create(this.$thingie);
  return this;
};

var widget = new Widget({
  create: function($elem) {
    $elem.addClass('my-style-stuff');
  }
});
widget.show();
```
每当API接受回调函数时，确保文档化其签名，并提供示例帮助API使用者定制代码。确保回调函数所执行的上下文(`this`的指向)参数都保持一致性。

#### 事件
当需要处理DOM时，事件自然而然出现。在大型的应用中我们以各种机制(例如pubsub)使用事件使模块通讯变得可能。当处理UI控件时，事件尤为有用并且很自然。像jQuery这样的库提供了简单的接口，允许你很容易实现这方面的需求。

当有事情发生时事件介入最佳——这正是事件的得名由来。根据外部环境显示或隐藏一个控件，当控件显示更新它——这些都是很常见的交互。借助于jQuery的事件接口，这些都很容易实现，甚至允许使用事件委托：
```
Widget.prototype.show = function() {
  var event = jQuery.Event('widget:show');
  this.$container.trigger(event);
  if (event.isDefaultPrevented()) {
    //event handler prevents us from showing
    return this;
  }
  
  this.options.position(this.$thingie, this.$container);
  this.$thingie.show();
  return this;
};

// listen for all widget:show events
$(document.body).on('widget:show', function(event) {
  if (Math.random() > 0.5) {
    // prevent widget from showing
    event.preventDefault();
  }
  // update widget's data
  $(this).data('last-show', new Date());
});

var widget = new Widget();
widget.show();
```
你可以任意选择事件名。避免在处理专有的事件上使用原生事件，并且考虑将你的事件放入命名空间下。jQuery UI 的事件名都是由部件名和事件名组合而成的，例如`dialogshow`。我觉得这样难以阅读所以常将其改为`dialog:show`的默认写法，主要是因为这样一看就知道它是默认事件，而不是某些浏览器特定的私有实现。

--------------------------------------------------------------------------
### 钩子机制
传统的getter/setter方法特别能收益于钩子机制。钩子机制通常在数量和如何注册方面有别于回调函数。回调函数通常应用于特定任务的实例级，而钩子则往往应用于全局级别自定义值或是调度自定义行为。为了演示钩子如何使用，让我们看看`jQuery's cssHooks`中的例子：
```
//define a custom css hook
jQuery.cssHooks.custombox = {
  get: function(elem, computed, extra) {
    return $.css(elem, 'borderRadius') == '50%' ? 'circle' : 'box';
  },
  set: function(elem, value) {
    elem.style.borderRadius = value == 'circle' ? '50%' : '0';
  }
};

// have .css() use that hook
$('#some-selector').css('custombox', 'circle');
```
通过注册`custombox`这个钩子，jQuery`.css()`方法拥有了可以处理一个之前无法处理的css属性的能力。在[jQuery hooks](http://blog.rodneyrehm.de/archives/11-jQuery-Hooks.html)一文中，谈到了jQuery提供的一些其他钩子，以及在实践中如何应用。你可以像处理回调一样提供钩子：
```
DateInterval.nameHooks = {
  "yesterday" : function() {var d = new Date();
    d.setTime(d.getTime() - 86400000);
    d.setHours(0);
    d.setMinutes(0);
    d.setSeconds(0);
    return d;
  }
};

DateInterval.prototype.start = function(date) {
  if (date === undefined) {
    return new Date(this.startDate.getTime());
  }
  
  if (typeof date === 'string' && DateInterval.nameHooks[date]) {
    date = DateInterval.nameHooks[date]();
  }
  
  if (!(date instanceof Date)) {
    date = new Date(date);
  }
  
  this.startDate.setTime(date.getTime());
  return this;
};

var di = new DateInterval();
di.start('yesterday');
```
从某种程度上讲，钩子是一系列被设计为以你自己的代码来处理自定义值的回调函数。有了钩子，你可以将差不多任何东西都保持在可控范围内，同时提供API使用者定制的选择。

-----------------------------------------------------------------------------------
### 生成访问器
![duplication](https://github.com/yiyunShm/NoteBook/blob/master/js/images/duplication.png)

任何一个API多半都会有完成类似工作的多种访问方法(getters, setters, executors)。回到`DateInterval`的例子，我们应该会提供`start()`和`end()`方法以允许对时间间隔的操作。可以像这样简单解决：
```
DateInterval.prototype.start = function(date) {
  if (date === undefined) {
    return new Date(this.startDate.getTime());
  }

  this.startDate.setTime(date.getTime());
  return this;
};

DateInterval.prototype.end = function(date) {
  if (date === undefined) {
    return new Date(this.endDate.getTime());
  }

  this.endDate.setTime(date.getTime());
  return this;
};
```
如你所见，这里有很多重复性代码。采用生成器模式可以提供一种 DRY(Don't Repeat Yourself) 解决方案：
```
var accessors = ["start", "end"];
for (var i = 0, length = accessors.length; i < length; i++) {
  var key = accessors[i];
  DateInterval.prototype[key] = generateAccessor(key);
}

function generateAccessor(key) {
  var value = key + "Date";
  return function(date) {if (date === undefined) {
      return new Date(this[value].getTime());
    }

    this[value].setTime(date.getTime());
    return this;
  };
}
```
这种方式允许你生成多种类似的访问器方法，而不是单独定义每个方法。如果你的访问器方法需要更多的数据以配置，而不是简单的字符串，考虑像下面的方式书写代码：
```
var accessors = {
  'start': { 'color': 'green' },
  'end': { 'color': 'end' }
};
for (var key in accessors) {
  DateInterval.prototype[key] = generateAccessors(key, accessors[key]);
}

function generateAccessors(key, accessor) {
  var value = key + 'Date';
  return function(date) {
    // setting something up
    // using 'key' and 'accessor.color'
  }
}
```
在*处理参数*那一节我们讨论到方法模式，允许你的getters/setters方法接受多种实用的类型，例如映射和数组。这种方法模式本身就是非常通用的，并且可以很容易转为一个生成器：
```
function wrapFlexibleAccessor(get, set) {
  return function(name, value) {
    var map;
    if (jQuery.isPlainObject(name)) {
      map = name;
    } else if (value !== undefined) {
      var keys = name.split(' ');
      for (var i = 0, len = keys.length; i < len; i += 1) {
        map[keys[i]] = value;
      }
    } else {
      return get.call(this, name);
    }
    
    for (var key in map) {
      set.call(this, name, map[key]);
    }
    
    return this;
  }
}

DateInterval.prototype.values = wrapFlexibleAccessor(
  function(name) {
    return name !== undefined ? this.values[name] : this.values;
  },
  function(name, value) {
    this.values[name] = value;
  }
);
```
本文并不深入讲述符合DRY原则的代码。如果你对这个主题比较生疏，[Rebecca Murphey](https://twitter.com/rmurphey)的[Patterns for DRY-er Javascript](http://rmurphey.com/blog/2010/07/12/patterns-for-dry-er-javascript/)一文和[Mathias Bynens](https://twitter.com/mathias)的幻灯片[how DRY impacts Javascript performance](http://slideshare.net/mathiasbynens/how-dry-impacts-javascript-performance-faster-javascript-execution-for-the-lazy-developer)都是很好的教程。

### 恐怖的引用
不同于其他语言，Javascript中不存在按*引用传递*和*按值传递*的概念。按值传递是比较安全的做法，可以确保你的API输入和输出的数据在外部被修改时不会更改内部的状态。按引用传递往往是为了保持较低的内存开销，按引用传递的值可能会在你的API之外的任何地方被修改并影响其内部状态。

在 JavaScript 中无法判断参数应该按应用传递还是按值传递。基本类型（字符串、数字、布尔值）都被处理为*按值传递*，但是对象（任何对象，包括 Array、Date）都以类似于按引用的方式进行处理。如果你初次接触这个话题，下面这个例子可以启发你：
```
// by value
function addOne(num) {
  num = num + 1; // yes, num++; does the samereturn num;
}

var x = 0;
var y = addOne(x);
// x === 0 <--// y === 1

// by reference
function addOne(obj) {
  obj.num = obj.num + 1;
  return obj;
}

var ox = {num : 0};
var oy = addOne(ox);
// ox.num === 1 <--// oy.num === 1
```
如果你不注意，对对象的按引用处理有可能会反过来给你带来麻烦。回到`DateInterval`的例子，看看下面这个棘手的问题：
```
var startDate = new Date(2012, 0, 1);
var endDate = new Date(2012, 11, 31)
var interval = new DateInterval(startDate, endDate);
endDate.setMonth(0); // set to january
var days = interval.days(); // got 31 but expected 365 - ouch!
```
除非 DateInterval 的构造器为它接受的值创建拷贝（clone是创建拷贝的术语），否则任何在原始对象上的改变都会直接反映到 DateInterval 的内部。这往往不是我们所想要或是所期望的。

注意，你的 API 中的返回值同样存在这样的隐患。如果你只是返回一个内部对象，你的 API 外部的任何变化都会反映到内部数据中。毫无疑问这并非你想要的。`jQuery.extend()`、`_.extend()` 以及 Protoype 的`Object.extend` 让你可以轻松摆脱引用之怖。

如果这里总结得还不够，你还可以读一读 O’Reilly 的[JavaScript – The Definitive Guide](http://docstore.mik.ua/orelly/webprog/jscript/index.htm)一书中[By Value Versus by Reference](http://docstore.mik.ua/orelly/webprog/jscript/ch11_02.htm)，讲得非常棒。

-------------------------------------------------------------------
### 连续性问题
在连贯接口中，链上的每个方法都会被执行，不论对象主体处于什么状态。考虑在一个不包含任何 DOM 元素的 jQuery 实例上调用一些方法：
```
jQuery('.wont-find-anything')
  // executed although there is nothing to execute against
  .somePlugin().someOtherPlugin();
```
在非链式的代码中，我们可以避免这些方法被执行：
```
var $elem = jQuery('.wont-find-anything');
if ($elem.length) {
  $elem.somePlugin().someOtherPlugin();
}
```
只要我们将方法链接起来，我们就无法避免这样的事情发生——我们无法逃离这条链。对象可能遇到这种情形，即方法实际上不做任何事仅仅`return this;`，只要 API 开发者能意识到这一点就没事。依据你的方法的内部实现，在前面加一个简单的`is-empty`检测可以会有帮助：
```
jQuery.fn.somePlugin = function() {
  if (!this.length) {
    // "abort" since we've got nothing to work withreturn this;
  }

  // do some computational heavy setup tasks
  for (var i = 10000; i > 0; i--) {
    // I'm just wasting your precious CPU!
    // If you call me often enough, I'll turn
    // your laptop into a rock-melting jet engine
  }

  return this.each(function() {
    // do the actual job
  });
};
```

-------------------------------------------------------------------------------
### 处理错误
![Fail faster](https://github.com/yiyunShm/NoteBook/blob/master/js/images/fail_faster.png)

我之前说我们无法从链中逃出来，其实是骗你的——对于这条规则有一个`Exception`(请不要介意这个双关语)

通过抛出错误（异常）我们就可以强制退出。抛出错误往往被认为是当前执行流的蓄意中止，往往可能是因为你陷入无法恢复的状态。但是当心——并不是所有的错误都会帮助开发者调试：
```
// jQuery accepts this
$(document.body).on('click', {});

// on click the console screams
// TypeError: ((p.event.special[l.origType] || {}).handle || l.handler).apply is not a function 
//   in jQuery.min.js on Line 3
```
遇到这样的错误信息是调试时最痛苦的事。不要浪费他人的时间。如果 API 使用者用得不对，请告知他：
```
if (Object.prototype.toString.call(callback) !== '[object Function]') {
  // see note
  throw new TypeError("callback is not a function!");
}
```
注意：`typeof callback === "function"`不应被使用，因为老式浏览器会认为对象是function，事实上它们不是。Chrome（直到版本 12）中的RegExp就是如此。方便起见，使用`jQuery.isFunction()`或是`_.isFunction()`。

对于语言（内置弱类型域 (weak-typing domain)）不在意严格的输入验证这一点，我接触过的大部分库采取了无视的态度。老实说，我也只在预感开发者会出错的时候在代码中进行校验。没有人真的做了，但是我们都应该去做。程序员是一个懒惰的群体——我们不会只是为了写代码或者某些我们并不真正相信的理由而写代码。Perl 6 的开发者们已经意识到这是个问题，并且决定引入叫做*参数约束*的东西。在 JavaScript 中，它可能会是这样实现：
```
function validateAllTheThings(a, b {where typeof b === "numeric" and b < 10}) {
  // Interpreter should throw an Error if b is
  // not a number or greater than 9
}
```
尽管语法上看上去很丑陋，主要思想是要使输入验证称为这门语言的一个顶级公民。JavaScript 与这样的东西差之千里。这样也不错——不管怎样，我也不愿看到函数签名中塞满这样一些约束。承认这个（弱类型语言中的）问题是这个故事中有意思的部分。

JavaScript 既不弱也不低等，我们只是需要更努力一点工作以使我们的代码变得真正健壮。使代码具有健壮性并不意味着不论接受什么数据，只要挥挥魔杖就能得到结果。健壮性是指不接受垃圾并且告之开发者。

换个角度考虑输入验证：在你的 API 后面加几行代码，就可以确保开发者不必花费几个小时来跟踪诡异的错误，结果发现原来他们意外地给你的代码中传入了字符串而不是数字。这种时候你应该告诉用户输入有误，他们实际上会喜欢你这么做的。

--------------------------------------------------------------------------
### 向异步出发
目前我们只讨论了同步的 API。异步方法通常接受一个回调函数，从而在某个任务完成时通知外部环境。虽然在连贯接口中这样并不是非常合适：
```
Api.protoype.async = function(callback) {
  console.log("async()");
  // do something asynchronous
  window.setTimeout(callback, 500);
  return this;
};

Api.protoype.method = function() {
  console.log("method()");
  return this;
};

// running things
api.async(function() {
  console.log('callback()');
}).method();

// prints: async(), method(), callback()
```
这个例子演示了什么情况下异步方法`async()`虽然开始执行但立即返回，却会导致`method()`在`async()`真正完成前就被调用了。某些时候我们需要这么做，但通常我们都期望`method()`在`async()`完成任务之后才会被执行。

#### 延迟机制——Promise
某种程度上，我们可以借助`Promise`来解决同步和异步 API 调用混搭导致的混乱。jQuery 称之为延迟机制。用延迟替代常见的this，从而迫使你从方法链中强行退出。这起初看上去有点怪，但是可以有效地避免在调用一个异步方法之后继续同步执行：
```
Api.prototype.async = function() {
  var deferred = $.Deferred();
  console.log("async()");

  window.setTimeout(function() {
    // do something asynchronous
    deferred.resolve("some-data");
  }, 500);

  return deferred.promise();
};

api.async().done(function(data) {
  console.log("callback()");
  api.method();
});
// prints: async(), callback(), method()
```
延迟对象使你可以使用`.done()`、`.fail()`、`.always()`注册一些处理器，当异步任务完成或失败时，或者不关心状态如何，再调用它们。关于延迟机制更详细的介绍参见[Promise Pipelines In JavaScript](http://sitr.us/2012/07/31/promise-pipelines-in-javascript.html)。

----------------------------------------------------------------------------------
### 调试连贯接口
虽然连贯接口更便于开发中使用，但就可调试性而言，会带来一些限制。

对于任何代码，测试驱动开发(TDD) 是减少调试需求的一种简单方法。在使用 TDD 完成 URI.js 中，就调试代码而言，我没有遇到什么严重的痛苦。然而，TDD 仅仅减少了调试的需要——并不会完全替代之。

有网民声称，可以在单独的行中书写链中的每个部件，从而在堆栈跟踪时获得正确的行号。
```
foobar.bar()
  .baz()
  .bam()
  .someError();
```
这种技巧确实有它的好处（尽管不包括更好的调试技术）。像上面例子中这样书写代码更易于阅读。基于行的差异（在例如 SVN、GIT 这样的版本控制系统中有用到）也会带来细微的优势。关于智能调试，（目前）只有 Chrome 会将`someError()`展示在第四行，而其他浏览器则仍将其看作第一行。

添加一个简单的方法 log 你的对象会很有用——尽管这样会被视为“手工调试”，并且会被那些习惯了“真实”的调试器的人看不惯：
```
DateInterval.prototype.explain = function() {
  // log the current state to the console
  console.dir(this);
};

var days = (new Date(2012, 0, 1))
  .until(2012, 11, 31) // returns DateInterval instance
  .explain() // write some infos to the console
  .days(); // 365
```

#### 函数名
在本文中你随处可见很多`Foo.prototype.something = function(){}`这种风格的演示代码。这种风格是为了保持例子比较简洁。当编写 API 时你可能会考虑如下方式之一来使你的控制台正确地识别出函数名：
```
// statement function
Foo.prototype.something = function something() {
  // yadda yadda
};

// Webkit
Foo.prototype.something = function() {
  // yadda yadda
};
Foo.prototype.something.displayName = "Foo.something";
```
第二种方式中displayName是由 WebKit 引入的，之后被 Firebug/Firefox 所采纳。`displayName`需要多写点代码，但是允许任意的名字，包括命名空间或是关联对象。两者中的任何一种都对于处理匿名函数很有帮助。

关于这个话题的更多细节参见[kangax](https://twitter.com/kangax)的[Named function expressions demystified](http://kangax.github.com/nfe/)。

---------------------------------------------------------------------------------
### 文档化API
软件开发中最困难的任务之一就是文档化。几乎所有人都讨厌做这件事，然而所有人都感叹他们需要使用的工具的文档纰漏或是缺失。目前有各种各样据说能提供帮助和自动文档化你的代码的工具：
* [YUIDoc](http://yui.github.com/yuidoc/) (requires Node.js, npm)
* [JsDoc ToolKit](https://github.com/p120ph37/node-jsdoc-toolkit) (requires Node.js, npm)
* [Markdox](https://github.com/cbou/markdox) (requires Node.js, npm)
* [Dox](https://github.com/visionmedia/dox) (requires Node.js, npm)
* [Docco]((requires Node.js, Python, CoffeeScript) (requires Node.js, Python, CoffeeScript)
* [JSDuck](https://github.com/senchalabs/jsduck) (reqires Ruby, gem)
* [JSDoc 3](https://github.com/jsdoc3/jsdoc) (requires Java)

所有这些工具都会在某些方面不尽如人意。JavaScript 是一种非常动态的语言，尤其在表达方式上特别多样化。这使得很多东西对这些工具而言比较困难。下面重点列出了一些我决定采用普通的 HTML、markdown 或是DocBoock（如果这个项目足够大）制定文档的原因。譬如，jQuery 同样遇到了这些问题，但根本不在代码中文档化 API。

1. 你需要文档化的并不仅仅是函数签名，但是大多数工具都只关注于此；
2. 示例代码可以为解释工作原理带来极大的帮助，但普通的 API 文档通常无法以合理的折衷来阐释；
3. API 文档解释幕后的东西（流、事件等等）时会遭遇滑铁卢；
4. 文档化多签名方法往往实在很痛苦；
5. 文档化使用 option 对象的方法通常并不简单；
6. 生成方法不容易被文档化，默认回调也是。

如果你不能（或不想）调整你的代码以适应列出的文档化工具之一，类似Document-Bootstrap这样的项目可能会节省你一些时间来建立你自己酝酿的文档。

确保你的文档化不只是一些生成的 API 文档。你的用户会感激你提供的示例。告诉他们你的软件如何工作的，以及当执行某件事时都会牵涉到哪些事件。如果有助于他们理解你的软件到底做了些什么，为他们画一张图。最重要的是，保持你的文档与你的代码同步！

#### 自解释的代码
提供优秀的文档并不会使开发者不用阅读你的代码——你的代码本身就是文档的一部分。当文档不够用时（每个文档都是有限的），开发者会回到阅读源代码获取答案。事实上，你也是他们中的一员。很可能你会一边又一遍地阅读你自己的代码，几周、几个月甚至几年之间。

你应该编写可以自解释的代码。大部分时候这并不是个问题，只有当你为（函数、变量等等）命名殚精竭虑、保持核心概念时才会涉及到。如果你发现你在写代码注释以文档化你的代码如何工作，你很可能在浪费时间——你的时间，还有读者的时间。在你的代码中的注释应该解释为何你以这种特殊的方式解决问题，而不是解释你如何解决问题。如何解决问题应该在你的代码中很明显，所以不要自我重复。注意，使用注释以标示你的代码中的区块，或是解释普通概念，这些都是完全可接受的。

---------------------------------------------------------------------------
### 结语
* API 是你（提供者）和用户（消费者）之间的契约。不要在版本之间发生变化。
* 你应该投入跟解决 我的软件内部如何工作？ 的问题同样多的时间，来解决 用户会如何使用我的软件？ 这个问题。
* 只要一些简单的技巧你就可以很显著地减少开发者的辛苦（就代码行数而言）
* 尽可能早地处理非法输入——抛出错误
* 好的 API 都是弹性的，更好的 API 让你避免犯错

继续阅读[Reusable Code for good or for awesome](http://vimeo.com/35689836)，这是Jake Archibald关于设计 API 的一番讨论。早在 2007 年 Joshua Bloch 在 Google 技术讲座上做了题为[How to Design A Good API and Why it Matters](http://www.youtube.com/watch?v=heh4OeB9A-c)演讲。虽然他的讨论并不集中于 JavaScript，他解释的基本原理仍然适用。

既然你已经掌握了关于 API 设计的最新进展，读一读Addy Osmani写的[Essential JS Design Patterns](http://addyosmani.com/resources/essentialjsdesignpatterns/book/)来了解更多关于如何组织你的内部代码吧。