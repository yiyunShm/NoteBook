[TOC]

## 享元模式

### 介绍
享元模式，允许共享技术有效地支持大量细粒度的对象，避免大量拥有相同内容的小类的开销(如内存耗费)，使大家共享一个类(元类)。

享元模式最适合于解决因创建大量类似对象而累计性能的问题。通过把大量独立对象转化为少量共享对象，可以降低允许web应用程序所需的资源数量。

### 基本实现
享元模式用于减少应用程序所需对象的数量。这是通过将对象的内部状态划分为 *内在数据* 和 *外在数据* 两类而实现的。内在数据是指类的内部方法所需要的信息，没有这种数据的话类就不能正常运转。外在数据则是可以从类身上剥离并存储在其外部的信息。我们可以将内在状态相同的所有对象替换为同一个共享对象，用这种方法可以把对象数量减少到不同内在状态的数量。

假定通过一个类库让系统管理所有的书籍，每个书籍的元数据暂定为如下内容：
```
ID
Title
Author
Genre
Page count
Publisher ID
ISBN
```
还需要定义每本书被借出去的时间和借书人，以及退书日期和是否可用状态：
```
checkoutDate
checkoutMember
dueReturnDate
availability
```
因为book对象设置成如下代码，注意该代码还未被优化：
```
var Book = function(options) {
  this.id = options.id;
  this.title = options.title;
  this.author = options.author;
  this.genre = options.genre;
  this.pageCount = options.pageCount;
  this.publisherID = options.publisherID;
  this.ISBN = options.ISBN;
  this.checkoutDate = options.checkoutDate;
  this.checkoutMember = options.checkoutMember;
  this.dueReturnDate = options.dueReturnDate;
  this.availability = options.availability;
};

Book.prototype = {
  getTitle: function() {
    return this.title;
  },
  getAuthor: function() {
    return this.author;
  },
  getISBN: function() {
    return this.ISBN;
  },
  // other props get method...
  
  // update checkout status
  updateCheckoutStatus: function(bookID, newStatus, checkoutDate, checkoutMember, newReturnDate){
    this.id  = bookID;
    this.availability = newStatus;
    this.checkoutDate = checkoutDate;
    this.checkoutMember = checkoutMember;
    this.dueReturnDate = newReturnDate;
  },
  extendCheckoutPeriod: function(bookID, newReturnDate){
    this.id =  bookID;
    this.dueReturnDate = newReturnDate;
  },
  isPastDue: function(bookID){
    var currentDate = new Date();
    return currentDate.getTime() > Date.parse(this.dueReturnDate);
  }
}
```
程序刚开始可能没问题，但是随着时间的增加，图书可能大批量增加，并且每种图书都有不同的版本和数量，你将会发现系统变得越来越慢。几千个book对象在内存里，可想而知，我们需要享元模式来优化。

我们可以将数据分成内部和外部数据，和book对象相关的数据(title, author等)可以归结为内部属性，而(checkoutMember, dueReturnDate)可以归结为外部属性。这样，如下代码就可以在同一本书里共享同一个对象了，因为不管谁借的书，只要书是同一本，基本信息是一样的。
```
var Book = function(options){
  this.title = options.title;
  this.author = options.author;
  this.genre = options.genre;
  this.pageCount = options.pageCount;
  this.publisherID = options.publisherID;
  this.ISBN = options.ISBN;
};
```

让我们来定义一个基本工厂，用来检查之前是否创建该对象，如果有就返回，如果没有就重新创建并存储以便今后方法，确保每种书只创建一个对象：
```
// factory
var BookFactory = (function() {
  var bookLists = {};
  return {
    createBook: function(options) {
      var book = bookLists[options.ISBN];
      if (book) {
        return book;
      } else {
        book = new Book(options);
        bookLists[options.ISBN] = book;
        return book;
      }
    }
  }
})();

// extrinsic
var BookRecordManager = (function() {
  var bookRecordDatabase = {};
  return {
    addBookReacord: function(options) {
      var _bookOptions, book;
      
      _bookOptions = {
        title: options.title,
        author: options.author,
        genre: options.genre,
        pageCount: options.pageCount,
        publisherID: options.publisherID,
        ISBN: options.ISBN
      };
      book = createBook(_bookOptions);
      bookRecordDatabase[id] = {
        checkoutMember: checkoutMember,
          checkoutDate: checkoutDate,
          dueReturnDate: dueReturnDate,
          availability: availability,
          book: book; // save in cache
      };
    },
    updateCheckoutStatus: function(bookID, newStatus, checkoutDate, checkoutMember, newReturnDate) {
      var record = bookRecordDatabase[bookID];
      
      record.availability = newStatus;
      record.checkoutDate = checkoutDate;
      record.checkoutMember = checkoutMember;
      record.dueReturnDate = newReturnDate;
    },
    extendCheckoutPeriod: function(bookID, newReturnDate) {
      bookRecordDatabase[bookID].dueReturnDate = newReturnDate;
    },
    isPastDue: function(bookID){
      var currentDate, returnDate;
      
      currentDate = new Date();
      returnDate = Date.parse(bookRecordDatabase[bookID].dueReturnDate);
      return currentDate.getTime() > returnDate;
   }
  }
})();
```
通过这种方式，我们做到了将同一种图书的相同信息保存在一个bookmanager对象里，而且只保存一份；相比之前的代码，就可以发现节约了很多内存。

### 享元与DOM
在Javascript对象需要创建HTML内容这种情况下，享元模式特别有用。那种会生成DOM元素的对象如果数目众多的话，会占用过多的内存，使网页陷入泥沼。采用享元模式后，只需创建少许这种对象就可，所有需要这种对象的地方都可以共享。

*工具提示* 就是在桌面应用程序中把鼠标指针移动工具图标上时出现的浮动文本框，方便用户不用点击工具也能知道其用途。如下代码是未经优化的：
```
var ToolTip = function(target, text) {
  this.target = target;
  this.text = text;
  
  this.delayTimeout = null;
  // in milliseconds
  this.delay = 1500;
  // create the html
  this.element = document.createElement('div');
  this.element.style.display = 'none';
  this.element.style.position = 'absolute';
  this.element.style.className = 'tooltip';
  document.getElementByTagName('body')[0].appendChild(this.element);
  this.element.innerHTML = this.text;
  // attach the events
  var that = this;
  addEvent(this.target, 'mouseover', function(e) {
    that.startDelay(e);
  });
  addEvent(this.target, 'mouseout', function(e) {
    that.hide();
  });
};
ToolTip.prototype = {
  startDelay: function(e) {
    if (this.delayTimeout == null) {
      var that = this,
          x = e.clientX,
          y = e.clientY;
      this.delayTimeout = setTimeout(function() {
        that.show(x, y);
      }, this.delay);
    }
  },
  show: function(x, y) {
    clearTimeout(this.delayTimeout);
    this.delayTimeout = null;
    this.element.style.left = x + 'px';
    this.element.style.top = (y + 20) + 'px';
    this.element.style.display = 'block';
  },
  hide: function() {
    clearTimeouut(this.delayTimeout);
    this.delayTimeout = null;
    this.element.style.display = 'none';
  }
}
```
这个类的用法很简单，只管为每一个工具对象创建其实例就好。但是如果网页上有几百个甚至几千个元素都需要用到工具提示，那会出现什么情况呢？这意味着将会出现大量的Tooltip类实例，造成了内存的耗费和低效。

*实现享元模式的一般步骤*：
* 把外在数据从对象中分离
* 创建一个用来实例化Tooltip的工厂
* 创建一个用来保存外在数据的管理器

在这个例子中我们还可以发挥点创造性：用一个单体同时扮演工厂和管理器的角色。此外，由于外在数据可以作为事件监听器的一部分保存，因此没有必要使用一个中心数据库。
```
var ToolManager = (function() {
  var storedInstance = null;
  
  var Tooltip = function() {
    this.delayTimeout = null;
    // in milliseconds
    this.delay = 1500;
    // create the html
    this.element = document.createElement('div');
    this.element.style.display = 'none';
    this.element.style.position = 'absolute';
    this.element.style.className = 'tooltip';
    document.getElementByTagName('body')[0].appendChild(this.element);
  };
  Tooltip.prototype = {
    startDelay: function(e, text) {
      if (this.delayTimeout == null) {
        var that = this,
            x = e.clientX,
            y = e.clientY;
        this.delayTimeout = setTimeout(function() {
          that.show(x, y, text);
        }, this.delay);
      }
    },
    show: function(x, y, text) {
      clearTimeout(this.delayTimeout);
      this.delayTimeout = null;
      this.element.innerHTML = text;
      this.element.style.left = x + 'px';
      this.element.style.top = (y + 20) + 'px';
      this.element.style.display = 'block';
    },
    hide: function() {
      clearTimeouut(this.delayTimeout);
      this.delayTimeout = null;
      this.element.style.display = 'none';
    }
  };
  
  return {
    addTooltip: function(target, text) {
      // get the tooltip object
      var tt = this.getTooltip();
      // atach the evevnt;
      addEvent(target, 'mouseover', function(e) {
        tt.startDelay(e, text);
      });
      addEvent(target, 'mouseout', function(e) {
        tt.hide();
      });
    },
    getTooltip: function() {
      if (storedInstance == null) {
        storedInstance = new Tooltip();
      }
      
      return storedInstance;
    }
  }
})()
```

### 总结
要想把对象转化为享元模式需要满足一些前提条件：
* 网页中必须使用了大量资源密集型对象(对象少的话，优化并不划算)
* 这些对象的数据有部分能被转化为外在数据(即可以从对象内部分离出来)
* 将外在数据分离出去后，独一无二的对象数目相对较少

享元模式不过是一种优化模式，其作用是在满足一系列严格条件下提高代码的运行效率。它不是什么地方都能用，也不应该随便乱用。这种模式在优化代码的同时，也提高了代码的复杂程度，这会给调式和维护造成困难。

这些缺点并非洪水猛兽，它们的存在只是表明只有在必要的时候才应该进行这种优化。你必须在运行效率和可维护性之间进行权衡，然而这种权衡正是工程学的精髓所在。