[TOC]

## 代理模式

### 介绍
代理模式，为其他对象提供一种以控制对这个对象的访问。它与外观模式(facade)的区别在于，在外观模式中你所拥有的是合并了多个方法调用的便利方法。代理则介于对象的客户端和对象本身之间，并且对该对象的访问进行保护。

### 基本实现
代理对象并不会在另一对象的基础上添加方法或修改其方法，也不会简化那个对象的接口。它实现的接口与本体完全相同，所有对它进行的方法的调用都会被传递给本体。

这种模式可能看起来像是额外的开销，但是出于性能因素考虑它却非常有用。使用这种模式的其中一个例子是我们可以称为延迟初始化(lazy initialization)的方法。
```
var Book = function(name) {
  this.bookname = name;
};

var Student = function(book) {
  this.book = book;
};
Student.prototype.borrowBook = function() {
  console.log('I have borrow ' + this.book.bookname);
};

// proxy
var Teacher = function(book) {
  this.book = book;
};
Teacher.prototype.init = function() {
  if (!this.student || !(this.student instanceof Student)) {
    this.student = new Student(this.book);
  }
};
Teacher.prototype.borrowBook = function() {
  // lazy initialization
  this.init();
  this.student.borrowBook();
};
```

下图举例说明了这种情况，即首先由客户端发出一个初始化请求，然后代理以一切正常作为响应，但实际上却并没有将该消息传递到本体对象，直到客户端明显需要本体对象完成一些工作的时候，代理才将两个消息一起传递。
![relation between client and real subject](https://github.com/yiyunShm/NoteBook/blob/master/js/images/proxy_relation.png)

### 模式应用
当本体对象执行一些开销很大的操作时，代理模式就显得非常有用。在Web应用中，可以执行的开销最大的操作就是网络请求，因此，尽可能合并更多的HTTP请求就显得非常重要。

假定一个可以播放选定艺术家视频的小程序。
![proxy example](https://github.com/yiyunShm/NoteBook/blob/master/js/images/proxy_example.png)
在该页面中有视频标题清单，当用户点击一个视频标题时，该标题下面的区域将展开以显示更多关于该视频的信息，并且还能启动播放功能。

链接列表HTML：
```
<p><span id="toggle-all">Toggle Checked</span></p>
<ol id="vids">
    <li>
      <input type="checkbox" checked>
      <a href="http://new.music.yahoo.com/videos/--2158073">Gravedigger</a>
    </li>
    <li>
      <input type="checkbox" checked>
      <a href="http://new.music.yahoo.com/videos/--4472739">Save Me</a>
    </li>    
    <li>
      <input type="checkbox" checked>
      <a href="http://new.music.yahoo.com/videos/--45286339">Crush</a>
    </li>
    <li>
      <input type="checkbox" checked>
      <a href="http://new.music.yahoo.com/videos/--2144530">Don't Drink The Water</a>
    </li>
    <li>
      <input type="checkbox" checked>
      <a href="http://new.music.yahoo.com/videos/--217241800">Funny the Way It Is</a>
    </li>    
    <li>
      <input type="checkbox" checked>
      <a href="http://new.music.yahoo.com/videos/--2144532">What Would You Say</a>
    </li>    
</ol>
```

事件处理程序
```
// init selector method
var $ = function (id) {
    return document.getElementById(id);
};

$('vids').onclick = function (e) {
    var src, id;

    e = e || window.event;
    src = e.target || e.srcElement;

    // not link
    if (src.nodeName.toUpperCase() !== "A") {
        return;
    }
    // stop propagatin
    if (typeof e.preventDefault === "function") {
        e.preventDefault();
    }
    e.returnValue = false;

    id = src.href.split('--')[1];

    // play videos
    if (src.className === "play") {
        src.parentNode.innerHTML = videos.getPlayer(id);
        return;
    }
        
    src.parentNode.id = "v" + id;
    // first click to show info
    videos.getInfo(id);
};

$('toggle-all').onclick = function (e) {
    var hrefs, i, max, id;

    hrefs = $('vids').getElementsByTagName('a');
    for (i = 0, max = hrefs.length; i < max; i += 1) {
        // ignore play
        if (hrefs[i].className === "play") {
            continue;
        }
        
        if (!hrefs[i].parentNode.firstChild.checked) {
            continue;
        }

        id = hrefs[i].href.split('--')[1];
        hrefs[i].parentNode.id = "v" + id;
        videos.getInfo(id);
    }
};
```

#### 无代理实现
videos对象有三个方法：
* `getPlayer()` 返回HTML请求以播放Flash视频(与本模式讨论内容无关)
* `updateList()` 该回调函数接收所有来自Web服务的数据，并且生成HTML代码用于拓展信息片段。
* `getInfo()` 该方法用于切换信息片段的可见性，并且还在http对象的调用中将updateList() 作为回调函数传递出去。

```
var videos = {
    // init play
    getPlayer: function (id) {...},
    updateList: function (data) {...},

    getInfo: function (id) {
        var info = $('info' + id);

        if (!info) {
            // get http without proxy
            http.makeRequest([id], "videos.updateList");
            return;
        }

        if (info.style.display === "none") {
            info.style.display = '';
        } else {
            info.style.display = 'none';
        }
    }
};
```

http对象只有一个方法，该方法产生JSONP格式以请求Web服务：
```
var http = {
    makeRequest: function (ids, callback) {
        var url = 'http://query.yahooapis.com/v1/public/yql?q=',
            sql = 'select * from music.video.id where ids IN ("%ID%")',
            format = "format=json",
            handler = "callback=" + callback,
            script = document.createElement('script');

            sql = sql.replace('%ID%', ids.join('","'));
            sql = encodeURIComponent(sql);

            url += sql + '&' + format + '&' + handler;
            script.src = url;

        document.body.appendChild(script);
    }
};
```

#### 进入代理
无代理实现代码运行良好，当有6个视频同时切换时，6个独立的请求将被发送到Web服务上，于是我们可以更进一步。

通过proxy对象接管http和videos之间的通信，试图使用一个简单的逻辑将多个请求合并起来：即一个50ms的视频缓冲区。videos对象并不直接调用http服务而是调用proxy，然后在proxy转发请求之前一直等待。通过proxy合并请求的方法，显著降低了服务器的负载。
```
var videos = {
    // init play
    getPlayer: function (id) {...},
    updateList: function (data) {...},

    getInfo: function (id) {
        var info = $('info' + id);

        if (!info) {
            // get proxy
            proxy.makeRequest(id, videos.updateList, videos);
            return;
        }

        if (info.style.display === "none") {
            info.style.display = '';
        } else {
            info.style.display = 'none';
        }
    }
};

var proxy = {
    ids: [],
    delay: 50,
    timeout: null,
    callback: null,
    context: null,
    makeRequest: function (id, callback, context) {
        
        // add to the queue
        this.ids.push(id);
        
        this.callback = callback;
        this.context  = context;
        
        // set up timeout
        if (!this.timeout) {
            this.timeout = setTimeout(function () {
                proxy.flush();
            }, this.delay);
        }
    },
    flush: function () {
        
        http.makeRequest(this.ids, "proxy.handler");
                
        // clear timeout and queue
        this.timeout = null;
        this.ids = [];
        
    },
    handler: function (data) {        
        var i, max;
        
        // single video
        if (parseInt(data.query.count, 10) === 1) {
            proxy.callback.call(proxy.context, data.query.results.Video);
            return;
        }
        
        // multiple videos
        for (i = 0, max = data.query.results.Video.length; i < max; i += 1) {
            proxy.callback.call(proxy.context, data.query.results.Video[i]);
        } 
    }
};
```

下图分别举例说明了生成三轮往返消息到服务器(无代理)与使用代理时仅有一轮往返消息相比较的情景。
![http request without proxy](https://github.com/yiyunShm/NoteBook/blob/master/js/images/proxy_none.png)
![http request with proxy](https://github.com/yiyunShm/NoteBook/blob/master/js/images/proxy_with.png)

#### 缓存代理
在例子中，客户端对象(videos)足够聪明到不会再次请求同一个视频信息。但是实际情况并不会总是如此。代理可以通过将以前的请求结果缓存到新的cache属性中，从而更进一步的保护本体对象http的访问。因为直接从缓存中取出消息，从而也节省了网络的往返消息。
![http cache after proxy ](https://github.com/yiyunShm/NoteBook/blob/master/js/images/proxy_cache.png)

### 总结
代理模式一般适用于如下场合：
* 远程代理，也就是为了一个对象在不同的地址空间提供局部代表，这样可以隐藏一个对象存在于不同地址空间的事实，就像web service里的代理类一样。
* 虚拟代理，根据需要创建开销很大的对象，通过它来存放实例化需要很长时间的真实对象，比如浏览器的渲染的时候先显示问题，而图片可以慢慢显示（就是通过虚拟代理代替了真实的图片，此时虚拟代理保存了真实图片的路径和尺寸。
* 安全代理，用来控制真实对象访问时的权限，一般用于对象应该有不同的访问权限。
* 智能指引，只当调用真实的对象时，代理处理另外一些事情。例如C#里的垃圾回收，使用对象的时候会有引用次数，如果对象没有引用了，GC就可以回收它了。