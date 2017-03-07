[TOC]
## HTML APIs设计
原文链接：[HTML APIs: What They Are And How To Design A GoodOne](https://www.smashingmagazine.com/2017/02/designing-html-apis)

作为一个Javascript开发人员，我们经常忘记并不是所有人都和我们一样具备相同的知识体系来参与到工作中。这是个诅咒：当我们在某一方面有所专长，便难以记起作为新手时感受到的困惑。

我们为自己的框架或库写了一堆Javascirpt来初始化或配置相关内容，然后心安理得的认为人们会很容易理解那些Javascirpt的作用和意义。然而，事实却是我们的一些用户很难使用它们，只能疯狂地复制和粘贴文档示例，不停的调整直到它们正常工作。

你可能会想说，"HTML和CSS的开发人员应该都知道怎么使用Javascript吧？"。抱歉，下面是我的调查结果，也是我能知道的唯一数据。(如果你知道任何适当的研究，请在评论中说明一下)
![How comfortable are you with Javascirpt](https://github.com/yiyunShm/NoteBook/blob/master/js/images/js_poll_opt.png)

有一半的HTML & CSS开发人员难以习惯Javascript.

我们看一个jQuery UI autocomplete插件的官方示例：
```
<div class="ui-widget">
  <label for="tags">Tags: </label>
  <input id="tags">
</div>

$( function() {
  var availableTags = [
    "ActionScript",
    "AppleScript",
    "Asp",
    "BASIC",
    "C"
  ];
  $( "#tags" ).autocomplete({
    source: availableTags
  });
} );
```
看着很简单，即使不会Javascript也能明白，是吗？不，一个非程序员在看了这个示例后会有各种各样的问题在脑海中飘过。"这段代码我要放在哪里？"，"这些大括号，圆括号，冒号是干什么的，我需要怎么加？"，"如果我的元素对象没有ID要怎么办"等等。即使这么一小段的代码也需要使用者了解对象字面量、数组、变量、字符串、如何获得对DOM元素的引用、事件等等。对JS开发者，这可能是微不足道的事，但对于HTML开发人员来说，这或许是一个艰辛的过程。

我们现在来考虑一下HTML5的等效代码表示：
```
<div class="ui-widget">
  <label for="tags">Tags: </label>
  <input id="tags" list="languages">
  <datalist id="languages">
    <option>ActionScript</option>
    <option>AppleScript</option>
    <option>Asp</option>
    <option>BASIC</option>
    <option>C</option>
  </datalist>
</div>
```
这样的形式对于HTML开发人员，还有JS开发人员都更加清晰。我们可以看到所有东西都设置在一个地方，不用担心什么时候去初始化，如何获取DOM元素，也不需要知道怎么去配置，更不需要知道哪个函数接收什么参数来初始化插件。对于更高级的用例，还有一个JavaScript API，允许动态创建所有这些属性和元素。 它遵循最基本的API设计原则之一：它使得简单的交互更加容易，复杂的操作成为可能。

这为我们关于HTML API上了重要的一课：不仅是那些在Javascript技能上受限的人有所收获。对于日常的开发任务，即使我们，JS开发者，也经常为了HTML的书写便利而牺牲了JS的灵活性。然而，在我们编写自己的库时，却不知为何遗忘了这一点。

那么，什么是HTML API？ 根据维基百科，API（或应用程序编程接口）是“一组用于构建应用软件的子例程定义，协议和工具”。在HTML API中，定义和协议在HTML本身中，工具看起来 在HTML中进行配置。 HTML API通常由可在现有HTML上使用的某些类和属性模式组成。Web组件可以让我们自定义元素标签，甚至为`<game>`，虚拟DOM可以让一个完整的结构藏匿在js 或 css 中。但这不是一篇关于Web组件的文章，Web组件为HTML API设计者提供更多的功能和选项，但是好的(HTML) API设计的原理是一样的。

HTML API改善了设计师和开发人员之间的协作，减轻了后者的负担，并使设计人员能够创建更高保真的模型。在你的库中包含一个HTML API不仅使社区更具包容性，它最终还是会为你带来好处。

并不是所有的库都需要HTML API。HTML API在UI库(如图库，拖放，折叠，制表符，轮播等)中有更大的作用。根据经验，如果HTML开发者无法理解库的功能，那这个库就不需要HTML API。 例如，简化或帮助组织代码的库不需要HTML API。 MVC框架或DOM工具库会需要有什么样的HTML API呢？

到目前为止，我们已经讨论了什么是HTML API，为什么它是有用的，什么时候需要它。 本文的其余部分是关于如何设计一个好的。

### 初始化选择器
...

### 最小化初始标记
...

### 设置
...

### 继承
...

### 全局设置
...

### 文档
...

### 关于Web组件
...

### HTML API库
...