[TOC]

## 观察者模式

### 介绍
观察者模式又叫发布订阅模式(Publish/Subscribe)，它定义了一种一对多的关系，让多个观察者对象同时监听某一个主题对象，这个主题对象的状态发生变化时就会通知所有的观察者对象，使得它们能够自动更新自己。

这种模式背后的动机是促进形成松散耦合。在这种模式中，并不是一个对象调用另一个对象的方法，而是一个对象订阅另一个对象的特定活动并在状态改变后获得通知。订阅者也称之为观察者，而被观察的对象成为发布者或者主题。当发生了一个重要的事件时，发布者将会通知(调用)所有订阅者并且可能经常以事件对象的形式传递消息。

### 基本实现
#### 示例一：杂志订阅
假设一个发布者，它每天出版报纸以及月刊杂志，订阅者joe将被通知任何时候所发生的新闻。

该paper对象需要有一个subscribers属性，用于存储所有订阅者的数组。订阅行为只是将其加入到这个数组中。当一个事件发生时，paper将会循环遍历订阅者列表并通知他们。通知意味着调用订阅者对象的某个方法。因此，当用户订阅信息的时候，该订阅者需要向paper的 `subscribe()` 提供它的其中一个方法。

paper也提供了 `unsubscribe()` 方法，该方法表示从订阅者数组(即subscribers属性)中删除订阅者。paper最后一个重要的方法是 `publish()`，它会调用这些订阅者的方法。

发布者paper对象属性如下：
* subscribers 一个数组
* subscribe() 添加订阅者到subscribers数组
* unsubscribe() 从subscribers数组中删除订阅者
* publish() 循环遍历subscribers中的每个元素，并且调用他们注册时所提供的方法。

所有这三种方法都需要一个type参数，因为发布者可能触发多个事件(比如同时发布一本杂志和一份报纸)而用户可能仅选择订阅其中一种，而不是另外一种。
```
var publisher = {
  subscribers: {
    // default event type
    any: []
  },
  subscribe: function(fn, type) {
    type = type || 'any';
    if (typeof this.subscribers[type] === 'undefined') {
      this.subscribers[type] = [];
    }
    
    this.subscribers[type].push(fn);
    return this;
  },
  unsubscribe: function(fn, type) {
    this.visitSubscribers('unsubscribe', nf, type);
    return this;
  },
  publish: function(publication, type) {
    this.visitSubscribers('publish', publishcation, type);
    return this;
  },
  visitSubscribers: function(action, arg, type) {
    var pubtype = type || 'any',
        subscribers = this.subscribers[pubtype],
        i, max = subscribers.length;
        
    for (i = 0; i < max; i += 1) {
      if (action === 'publish') {
        subscribers[i](arg);
      } else {
        if (subscribers[i] === arg) {
          subscribers.splice(i, 1);
        }
      }
    }
  }
};
```
而这里有个函数 `makePublisher()`，它接受一个对象作为参数，通过把上述通用发布者方法复制到该对象中，从而将其转换为一个发布者：
```
function makePublisher(o) {
  var i;
  for (i in publisher) {
    if (publisher.hasOwnProperty(i) && typeof publisher[i] === 'function') {
       o[i] = publisher[i];
    }
  }
  
  o.subscribers = { any: [] };
}
```
现在来看看paper对象和joe对象的实现：
```
var paper = {
  daily: function() {
    this.publish('big news today');
  },
  monthly: function() {
    this.publish('interesting analysis', 'monthly');
  }
};

// be a publisher
makePublisher(paper);

var joe = {
  drinkCoffee: function(paper) {
    // type any
    console.log('just read ' + paper);
  },
  subdayPreNap: function(monthly) {
    console.log('About to fall asleep reading this ' + monthly);
  }
};

// test
paper.subscribe(joe.drinkCoffee)
    .subscribe(joe.sundayPreNap, 'monthly');
```

#### 示例二：键盘按键游戏
先回顾通用publisher对象，然后略微调整它的接口，使之更接近浏览器对象设计：
```
var publisher = {
    subscribers: {
        any: []
    },
    on: function (type, fn, context) {
        type = type || 'any';
        fn = typeof fn === "function" ? fn : context[fn];
        
        if (typeof this.subscribers[type] === "undefined") {
            this.subscribers[type] = [];
        }
        this.subscribers[type].push({fn: fn, context: context || this});
        return this;
    },
    remove: function (type, fn, context) {
        this.visitSubscribers('unsubscribe', type, fn, context);
        return this;
    },
    fire: function (type, publication) {
        this.visitSubscribers('publish', type, publication);
        return this;
    },
    visitSubscribers: function (action, type, arg, context) {
        var pubtype = type || 'any',
            subscribers = this.subscribers[pubtype],
            i, max = subscribers ? subscribers.length : 0;
            
        for (i = 0; i < max; i += 1) {
            if (action === 'publish') {
                subscribers[i].fn.call(subscribers[i].context, arg);
            } else {
                if (subscribers[i].fn === arg && subscribers[i].context === context) {
                    subscribers.splice(i, 1);
                }
            }
        }
    }
};
```

game对象用于记录所有的player对象，因此它可以产生一个分数并且触发 'scorechange' 事件。它还将从浏览器中订阅所有的 'keypress' 事件，并且知道每个键对应的玩家：
```
var game = {
    keys: {},

    addPlayer: function (player) {
        var key = player.key.toString().charCodeAt(0);
        this.keys[key] = player;
    },

    handleKeypress: function (e) {
        e = e || window.event; // IE
        if (game.keys[e.which]) {
            game.keys[e.which].play();
        }
    },
    
    handlePlay: function (player) {
        var i, 
            players = this.keys,
            score = {};
        
        for (i in players) {
            if (players.hasOwnProperty(i)) {
                score[players[i].name] = players[i].points;
            }
        }
        this.fire('scorechange', score);
    }
};
```

`player()` 构造函数接受key，即玩家用于得分所按的键盘的键。另外，每次当创建新的player对象时，一个名为 'newplayer' 的事件将被触发，每次当玩家玩游戏的时候，事件 'play' 将被触发。
```
function Player(name, key) {
    this.points = 0;
    this.name = name;
    this.key  = key;
    this.fire('newplayer', this);
}

Player.prototype.play = function () {
    this.points += 1;
    this.fire('play', this);
};

// test
makePublisher(Player.prototype);
makePublisher(game);

Player.prototype.on('newplayer', 'addPlayer', game);
Player.prototype.on('play', 'handlePlay', game);
game.on('scorechange', scoreboard.update, scoreboard);
window.onkeypress = game.handleKeypress;
```

### 总结
观察者的使用场合就是：当一个对象的改变需要同时改变其它对象，并且它不知道具体有多少对象需要改变的时候，就应该考虑使用观察者模式。

使用观察者模式的好处：
* 支持简单的广播通信，自动通知所有已经订阅过的对象。
* 页面载入后目标对象很容易与观察者存在一种动态关联，增加了灵活性。
* 目标对象与观察者之间的抽象耦合关系能够单独扩展以及重用

总的来说，观察者模式所做的工作就是在解耦，让耦合的双方都依赖于抽象，而不是依赖于具体。从而使得各自的变化都不会影响到另一边的变化。