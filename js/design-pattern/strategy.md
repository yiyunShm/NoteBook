[TOC]

## 策略模式

### 介绍
策略模式定义了算法家族，分别封装起来，让他们之间可以互相替换。

使用策略模式的其中一个例子是解决表单验证的问题。可以创建一个具有 `validate()` 方法的验证器(validator)对象。无论表单的具体类型是什么，该方法都会被调用，并且总是返回相同的结果，一个未经验证的数据列表以及任意的错误消息。

### 基本实现
假设有一下数据块，可能来自网页上的一个表单，而你需要验证它是否有效：
```
var data = {
  first_name: 'Super',
  last_name: 'Man',
  age: 'unknown',
  username: 'o_O',
};
```

在这个具体的例子中，为了使验证器知道什么是最好的策略，首先需要配置该验证器，并且设置认为是有效且可接受的规则。
```
// core
var validator = {
  types: {},
  messages: [],
  config: {},
  
  validate: function(data) {
    var i, msg, type, checker, result_ok;
    
    this.messages = [];
    for (i in data) {
      if (data.hasOwnProperty(i)) {
        type = this.config[i];
        checker = this.types[type];
        
        if (!type) {
          continue;
        }
        
        if (!checker) {
          throw {
            name: 'validationError',
            message: 'no handler to validate type ' + type
          };
        }
        
        result_ok = checker.validate(data[i]);
        if (!result_ok) {
          msg = 'Invalid value for *' + i + '*, ' + checker.instructions;
          this.messages.push(msg);
        }
      }
      return this.hasErrors();
    },
    hasErrors: function() {
      return this.messages.length !== 0;
    }
  }
};

validator.config = {
  first_nmae: 'isNonEmpty',
  age: 'isNumber',
  username: 'isAlphaNum'
};
```

现在我们看看如何实现validator的验证程序。用于检查的可用算法也是对象，它们具有一个预定义的接口，提供了一个 `validate()` 方法和一个可用于提示错误消息的一行帮助信息。
```
validator.types.isNonEmpty = {
  validate: function(value) {
    return value !== '';
  },
  instructions: 'the value cannot be empty'
};

validator.types.isNumber = {
  validate: function(value) {
    return !isNaN(value);
  },
  instructions: 'the value can only be a valid number, e-g. 1, 3.14 or 2010'
};

validator.types.isAlphaNum = {
  validate: function(value) {
    return !/[^a-z0-9]/i.test(value);
  },
  instructions: 'the value can only contain characters and numbers, no special symbols'
};
```

### 总结
策略模式定义了一系列算法，从概念上来说，所有的这些算法都是做相同的事情，只是实现不同，他可以以相同的方式调用所有的方法，减少了各种算法类与使用算法类之间的耦合。

从另外一个层面上来说，单独定义算法类，也方便了单元测试，因为可以通过自己的算法进行单独测试。

实践中，不仅可以封装算法，也可以用来封装几乎任何类型的规则，是要在分析过程中需要在不同时间应用不同的业务规则，就可以考虑是要策略模式来处理各种变化。