[TOC]

## 设计和构建你自己的Javascript代码库
原文链接：[design-and-build-your-own-javascript-library](https://www.sitepoint.com/design-and-build-your-own-javascript-library/)

代码库：我们一直在使用它们。代码库是开发者把他们会在项目中使用到的代码打包起来形成的，这总能节省和避免重复造轮子。拥有一个可重复使用的包，不管是开源的还是闭源的，总比重复构建一样特性的包或者从过去的项目中手动复制粘贴要好。

除了被打包的代码，还可以更精确的形容代码库吗？除了少数的例外，代码库通常只是一个文件，或者是在同一个文件夹里的几个文件。它的代码应该可以单独保存和在你的项目中正常使用。库文件允许你根据项目的不同来调整结构或者行为。

在这篇文章中，将会解释如何构建库文件。尽管大部分的方法可以应用到其他语言，但这篇文章主要讲述的是构建Javascript库文件。

-------------------------------------------------------------------------------
### 为什么要构建Javascript库
首先和最重要的，库文件可以让现有的代码方便重复利用。你不需要挖出陈旧的项目来复制文件，只需要引入库文件。这也可以让你的应用组件化，让应用的代码库更小更容易维护。

任何可以让实现一个具体的功能更容易或者可以被重复利用的抽象代码，都可以被打包进库文件。jQuery是一个很好的例子。尽管jQuery的API有大量简化的DOM API，在跨浏览器DOM操作比较困难的过去有着相当重要的意义。

如果一个开源项目变得流行并且让许多开发者使用它，人们很有可能通过提出问题或者贡献代码来参加它的开发。不管怎样，都对库和依赖它的项目有所帮助。

一个流行的开源库也会带来很好的机会。公司可能会对你的工作质量表示认可并给你发offer。可能公司会要求你把你的项目整合进他们的应用。毕竟，没人可以比你更了解你的项目。

当然，它可能只是一个习惯——享受编程，帮助他人并在这个过程中学习和成长。

### 范围和目标
在写下第一行代码之前，你需要确定你的库包含的功能——你需要设置一个目标，你可以专注于你想利用这个库来解决的问题。要牢记你的代码库的原本形式，在接口设计上要更容易使用和记忆。API越简单，使用成本就越低。引入一个Unix的设计哲学：
>只做一件事情并把它做好

问一下你自己：你的代码库要解决什么问题？你打算怎么解决它？你将会一个人完成全部工作，还是再引入别人的代码库？

不管代码库体积有多大，尝试制作一个流程图。列出你想要的每一个特性，并将它们尽可能的打散，直到你有一个小巧但是能解决问题的代码库。就像[minimum viable product](https://www.sitepoint.com/mvp-minimum-viable-product/)。这回成为你的第一个版本。从这里开始，你可以在每一个新特性建立里程碑。从本质上来说，你把你的项目变成了比特级的代码块，让每一个特性完成的更好和更有趣。

### API设计
我认为，我们可以从API使用者的角度去开发一个代码库，设计原则是以用户为中心。在本质上，你正在创造你的代码库大纲，给予它更多的思考和让选择它的人使用起来更方便。在同时，你需要思考在什么地方需要定制，这会在后面讨论。

终极的API测试是在自己的项目中使用你的代码库，在替换之后看是否满足你想要的特性，并尽可能让你的代码库表达的更直观，在边界条件下更灵活的使用，还有定制化。

这是一个关于用户代理字符串的代码库大纲可能会有的样子：
```
// Start with empty UserAgent string
var userAgent = new UserAgent();

// Create and add first product: EvilCorpBrowser/1.2 (x11; Linux; en-us)
var application = new UserAgent.Product('EvilCorpBrowser', '1.2');
application.setComment('x11', 'Linux', 'en-us');
userAgent.addProduct(application);

// Create and add second produce: Blink/20420101
var engine = new UserAgent.Product('Blink', '20420101');
userAgent.addProduct(engine);

// EvilCorpBrowser/1.2 (X11; Linux; en-us) Blink/20420101
userAgent.toString();

// Make some more changes to engine product
engine.setComment('Hello World');

// EvilCorpBrowser/1.2 (X11; Linux; en-us) Blink/20420101 (Hello World)
userAgent.toString();
```
根据代码的复杂度，你可能会在组织结构上花费一些时间。利用设计模式组织你的代码库是一个很好的办法，不仅可以有效地解决一些技术问题，还能避免新特性带来的大面积重构。

### 灵活性和定制性
灵活性是让代码库变得强大的要素，但是界限的明确总是很困难的。`chart.js`和`D3.js`是很好的例子。这两个代码库都是用来进行数据可视化的，`chart.js`可以很简单的创建不同形式的内置图表。但是如果你想对图像进行更多的掌控，`D3.js`才是你需要的。

有好几种方法可以把控制权交给用户：配置、暴露公共方法，通过回调和事件。

配置代码库通常在初始化之前完成。但是一些代码库允许在允许时对配置进行修改。配置通常被细小的部分限制，只有为了之后的使用而修改它们的数值才是被允许的。
```
// Configure at initialization
var userAgent = new UserAgent({
  commentSeparator: ';'
});

// Run-time configure using a public method
userAgent.setOption('commentSeparator', '-');

// Run-time configure using a public property
userAgent.commentSeparator = '-';
```
方法通常是暴露给实例使用的，比如说从实例中获取数据，或者设置实例的数据和执行操作。
```
var userAgent = new UserAgent();

// A getter to retrieve comments from all products
userAgent.getComments();

// An action to shuffle the order of all products
userAgent.shuffleProducts();
```
回调通常是在公共的方法中被传递的，通常在异步操作后执行用户的代码。
```
var userAgent = new UserAgent();

userAgent.doAsyncThing(function asyncThingDone(){
  //Run code after async thing is done
});
```
事件有很多种可能。有点像回调，除了增加事件句柄是不应该触发操作。事件通常用于监听，你可能会猜到，这可是事件！更像回调的是，你可以提供更多的信息和返回一个数值给代码库去进行操作。
```
var userAgent = new UserAgent();

// Validate a product on addition
userAgent.on('product.add', function(e, product){
  var shouldAddProduct = product.toString().length < 5;
  
  // Tell the library to add the product or not
  return shouldAddProduct;
});
```
在一些例子中，你可能允许用户对你的代码库进行拓展。因此，你需要暴露一些公共方法或者属性来让用户填充，像Angular的模块`angular.module('myModule')`和jQuery的`jQuery.fn.myPlugin`或者什么都不做，只是简单的让用户获取你的代码库的命名空间：
```
// AngryUserAgent module
// Has access to UserAgent namespace
(function AngryUserAgent(UserAgent) {
  // Create new method .toAngryString
  UserAgent.prototype.toAngryString = function() {
    return this.toString().toUpperCase();
  }
})(UserAgent);

// Application code
var userAgent = new UserAgent();
// ...

// EVILCORPBROWSER/1.2 (X11; LINUX; EN-US) BLINK/20420101
userAgent.toString();
```
在后面的例子中，允许你的用户获取代码库的命名空间，让你在对拓展和插件的定义方面上的控制变小了。为了让插件遵循一些约定，你可以(或者是应该)写下文档。

### 测试
对测试驱动开发[test-driven development](https://en.wikipedia.org/wiki/Test-driven_development)来说，写下大纲是良好的开始。简单来说，指的是在你写实际代码库之前，在你写下测试准则的时候。如果测试检查的是你的代码特性是否跟期待的一样，以及你在写代码库之前写测试，这就是行为驱动开发。这可以确定你的代码是可以正常工作的。

Jani Hartikaninen讲述了如何利用Mocha来进行单元测试[Unit Test Your Javascript Using Mocha and Chai](https://www.sitepoint.com/unit-test-javascript-mocha-chai/)。在使用Jasmine, Travis, Karma测试Javascript[Testing Javascirpt with Jasmine, Travis, and Karma](https://www.sitepoint.com/testing-javascript-jasmine-travis-karma/)这篇文章中，Tim Evko战士了怎么通过另一个叫做Jasmine的框架来设置良好的测试流程。这两个测试框架都是非常流行的，但还有适应别的需求的其他框架。

在这篇文章前面撰写的大纲，已经讲述了它期待怎样的输出。这是一切测试的开始：从期望出发。一个Jasmine测试像是这样：
```
describe('Basic usage', function() {
  it('should generate a single product', function() {
    // create a single product
    var product = new UserAgent.Product('EvilCorpBrowser', '1.2');
    product.setComment('x11', 'Linux', 'en-us');
    
    expect(product.toString())
    	.toBe('EvilCorpBrowser/1.2 (X11; Linux; en-us)');
  });
  
  it('should combine several products', function() {
    var userAgent = new UserAgent();

    // create and add first product
    var application = new UserAgent.Product('EvilCorpBrowser', '1.2');
    application.setComment('x11', 'Linux', 'en-us');
    userAgent.addProduct(application);
    
    // create and add second product
    var engine = new UserAgent.Product('Blink', '20420101');
    userAgent.addProduct(engine);
    
    expect(userAgent.toString())
    	.toBe('EvilCorpBrowser/1.2 (X11; Linux; en-us) Blink/20420101');
  });
  
  it('should update products correctly', function() {
    var userAgent = new UserAgent();
    
    // create and add first product
    var application = new UserAgent.Product('EvilCorpBrowser', '1.2');
    application.setComment('x11', 'Linux', 'ex-us');
    userAgent.addProduct(application);
    
    // Update first product
    application.setComment('x11', 'Linux', 'nl-nl');
    
    expect(userAgent.toString())
    	.toBe('EvilCorpBrowser/1.2 (X11; Linux; nl-nl)');
  });
});
```
当API结构设计让人满意后，便可以开始考虑代码库应该如何被使用。

### 模块加载器兼容性
你或许使用过模块加载器，使用你的代码库的开发者也可能在使用加载器，所以你会希望自己的代码库与模块加载器是兼容的。

虽然有很多的加载器，例如AMD, CommnJS, RequireJS等，但实际上不需要挑选兼容的版本。通用模块定义(UMD)是为了支持多种加载器而制定的规则。你可以在网上找到不同风格的代码段，或者从这里[UMD Github repository](https://github.com/umdjs/umd)学习并让它与你的代码库兼容。从其中的某个模板开始，或者使用你喜欢的构建工具[add UMD with your favorite build tool](https://github.com/umdjs/umd#build-tools)，你就不必再担心模块加载器的问题了。

如果你希望用上ES2015的`import/export`语法，我建议使用Bable和[Babel's UMD plugin](http://babeljs.io/docs/plugins/transform-es2015-modules-umd/)来将代码转换成ES5。通过这个方法你可以在你的项目中使用ES2015，同时生成兼容性良好的代码库。

### 文档
文档的编写应当从项目的名字和描述之类基本的信息开始。这会对别人明白你的代码库做了什么和是否对他们有用有所帮助。

你可以提供像是适用范围和目标之类的信息来更好的给用户提供信息，而提供路线图可以让他们明白在未来可能会有什么新变化以及他们可以提供怎样的帮助。

#### API，教程和例子
当然，你需要确保用户知道如何去使用你的代码库。这从API文档开始。教程和例子是很好的补充，但编写他们会是一个庞大的工作。然而，内联文档[Inline documentation](http://usejsdoc.org/)不会这样。下面是一些利用JSDoc的，可以被解析和转换成文档页的注释

#### 元任务
一些用户想对你的代码库作出改进。在大多数情况中，会是贡献代码，但有一些会创建一个私人使用的定制版本。对于这些用户，为类似构建代码库，运行测试，生成，转换和下载数据之类的元任务提供文档很有帮助的。

#### 贡献
当你对你的代码库进行开源，得到代码贡献是很有帮助的。为了引导贡献者，你可以增加一些关于贡献代码的步骤和需要满足的标准的文档。这会对你审阅和接纳贡献的代码和他们正确的贡献代码有所帮助。

#### 授权许可
最后一点，使用授权许可。从技术上讲，即时你没有选择任何一种技术许可，你的代码库依然是拥有版权的，但不是每一个人都知道这一点。

我发现[ChooseALicense.com](http://choosealicense.com/)这个网站可以让你一个不需要成为法律专家也能挑选授权许可。在挑选授权许可之后，只需要在项目的根目录下添加 `LICENSE.txt` 文件。

### 打包和发布到包管理器
对一个好的代码库来说，版本是很重要的。如果你想要加入重大的变化，用户可能需要保留他们现在正在使用的版本。

[Semantic Versioning](http://semver.org/)是流行的版本命名标准，或者叫它SemVer。SemVer版本包括三个数字，每一个代表不同程度的改变：重大改变，微小的改变和补丁

在你的Git仓库加入版本和发布
如果你有一个git仓库，你可以在你的仓库添加版本数字。你可以把它想象成你的仓库的快照。我们也叫它标签[Tags](https://git-scm.com/book/en/Git-Basics-Tagging)。可以通过开启终端和输入下面的文字来创造标签：
```
# git tag -a [version] -m [version message]
git tag -a v1.2.0 -m "Awesome Library v1.2.0"
```

#### 发布一个通用仓库
##### npm
许多编程语言自带有包管理器，或者是第三方包管理器。这可以允许我们下载关于这些语言的特定代码库。比如PHP的[Composer](http://www.sitepoint.com/mastering-composer-tips-tricks/)和Ruby的[RubyGems](https://rubygems.org/)

Node.js,一种独立的JavaScript引擎，拥有[npm](https://www.npmjs.com/)，如果你对npm不熟悉，我们有一个很好的教程[beginner’s guide](http://www.sitepoint.com/beginners-guide-node-package-manager/)。

默认情况下，你的npm包会发布为公共包。不要害怕，你也可以发布私有包[private packages](https://www.npmjs.com/npm/private-packages), 设置一个私有的注册[private registry](https://docs.npmjs.com/misc/registry#can-i-run-my-own-private-registry)， 或者根本不发布[avoid publishing](https://docs.npmjs.com/misc/registry#i-dont-want-my-package-published-in-the-official-registry-its-private).

为了发布你的包，你的项目需要有一个 package.json 文件。你可以手动或者交互问答的方式来创建。通过输入下面的代码来开始问答：
```
npm init
```
这个版本属性需要跟你的git标签吻合。另外，请确定有`README.md`文件。像是GitHub，npm在你的包的展示页使用它。

之后，你可以通过输入下面的代码来发布你的包：
```
npm publish
```
就是这样！你已经成功发布了你的npm包。

##### Bower
几年前，有另一个叫做Bower的包管理器。这个包管理器，实际上不是为了特定的语言准备的，而是为了互联网准备的。你可以发现大部分是前端资源。在Bower发布你的包的关键一点是你的代码库是否跟它兼容。

如果你对Bower不熟悉，我们也有一个教程[beginner’s guide](http://www.sitepoint.com/package-management-for-the-browser-with-bower/)。

跟npm一样，你也可以设置一个私有仓库[private repository](https://github.com/bower/registry#installation)。你可以通过问答的方式避免发布。

有趣的是，在最近的一两年，很多人转为使用npm管理前端资源。近段npm包主要是跟JavaScript相关，大部分的前端资源也发布在了npm上。不管怎样，Bower仍然流行。我明确的推荐你在Bower上发布你的包。

我有提到Bower实际上是npm的一种模块，并最初是得到它的启发吗？它们的命令是很相似的，通过输入下面的代码产生`bower.json`文件：
```
bower init
```
跟npm init类似，指令是很直白的，最后，发布你的包：
```
bower register awesomelib https://github.com/you/awesomelib
```
像是把你的代码库放到了野外，任何人可以在他们的Node项目或者网络上使用它！

### 总结
核心的产品是库文件。确定它解决了问题，容易和适合使用，你会使得你的团队或者许多开发者变得高兴。

