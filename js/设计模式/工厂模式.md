[TOC]

## 工厂模式

### 介绍
工厂模式的目的是为了创建对象，它通常在类或者类的静态方法中实现，具有下列目标：
* 当创建相似对象时执行重复操作
* 在编译时不知道具体类型(类)的情况下，为工厂客户提供一种创建对象的接口

其中，在静态类语言中第二点显得更为重要。因为静态语言创建类的实例是非平凡的，即事先(在编译时)并不知道实例所属的类。而在Javascript中，这部分目标实现起来相当容易。

### 基本实现
简单工厂模式又称为静态工厂方法(Static Factory Method)模式，它属于类创建型模式。在简单工厂模式中，可以根据参数的不同返回不同类的实例。简单工厂模式专门定义一个类来负责其他类的实例，被创建的实例通常都具有共同的父类。
```
var CarShop = function(){};
CarShop.prototype = {
  constructor: CarShop,
  sellCar: function(type) {
    var car;
    
    switch(type) {
      case 'benz':
        car = new Benz();
        break;
      case 'bwm':
        car = new BWM();
        break;
      case 'laos':
        car = new Laos();
        break;
      default:
        car = new Fort();
        break;
    }
    
    Interface.ensureImplements(car, Car);
    car.assemble();
    car.wash();
    
    return car;
  }
}
```
`Interface.ensureImplements` 接口在工厂模式中起着很重要的作用。如果不对对象进行某种类型的检查以确保其实现了必需的方法，工厂模式带来的好处也就所剩无几了。

实现该工厂模式并没有特别的困难。所有需要做的就是寻找能够创建所需类型对象的构造函数。在这种情况下，简洁的命名习惯可用于将对象类型映射到创建该对象的构造函数中。继承部分仅是可以放进工厂方法的一个公用重复代码片段的范例，而不是对每种类型的构造函数的重复。
```
function CarMaker() {}
CarMaker.prototype.drive = fucntion() {
  return 'Vroom, i have ' + this.doors + ' doors';
}
// abstract method
CarMaker.prototype.output = function() {
  throw new Error('Unsupported operation on an abstract class.');
}

CarMaker.factory = function(type) {
  var constr = type,
      newcar;
      
  if (typeof CarMaker[constr] !== 'function') {
    throw new Error('Unsupported type on an abstract class');
  }
  
  // create object
  newcar = new CarMaker[constr]();
  // inherit from CarMaker
  CarMaker[constr].prototype = new CarMaker();
  CarMaker[constr].prototype.constructor = CarMaker[constr];
  
  return newcar;
}

CarMaker.Compact = function() {
  this.doors = 4;
  this.output = function() {
    console.log('hello, i am compact');
  };
};
CarMaker.Convertible = function() {
  this.doors = 2;
  this.output = function() {
    console.log('hi, what are you doing');
  };
};
CarMaker.SUV = function() {
  this.doors = 24;
  this.output = function() {
    console.log('hahaha, stupid boy');
  };
};
```

以下几种情景下工厂模式特别有用：
* 对象的构建十分复杂
* 需要依赖具体环境创建不同实例
* 处理大量具有相同属性的小对象