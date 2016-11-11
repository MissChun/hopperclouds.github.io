title: JavaScript之设计模式(单例模式,构造函数模式)
date: 2016-10-10 14:08:08
# 类别
categories:
  - 技术
# 标签
tags:
  - JavaScript
---
作者: 李纯利


## 一、设计模式之单例模式

单例模式就是保证一个类只有一个实例，实现的方法一般是先判断实例存在与否，如果存在直接返回，如果不存在就创建了再返回，这就确保了一个类只有一个实例对象。
<!--more-->
下面我们来看一个单例的最佳实践：

```javascript
    var SingletonTester = (function () {
        //参数：传递给单例的一个参数集合
        function Singleton(args) {
            //设置args变量为接收的参数或者为空（如果没有提供的话）
            var args = args || {};
            //设置name参数
            this.name = 'SingletonTester';
            //设置pointX的值
            this.pointX = args.pointX || 6; //从接收的参数里获取，或者设置为默认值
            //设置pointY的值
            this.pointY = args.pointY || 10;
        }
        //实例容器
        var instance;
        var _static = {
            name: 'SingletonTester',
            //获取实例的方法
            //返回Singleton的实例
            getInstance: function (args) {
                if (instance === undefined) {
                    instance = new Singleton(args);
                }
                return instance;
            }
        };
        return _static;
    })();
    var singletonTest = SingletonTester.getInstance({ pointX: 5 });
    console.log(singletonTest.pointX); // 输出 5
```

## 二、设计模式之构造函数模式
介绍：构造函数用于创建特定类型的对象——不仅声明了使用的对象，构造函数还可以接受参数以便第一次创建对象的时候设置对象的成员值。你可以自定义自己的构造函数，然后在里面声明自定义类型对象的属性或方法。

基本用法：JavaScript没有类的概念，但是有特殊的构造函数。通过new关键字来调用定义的否早函数，你可以告诉JavaScript你要创建一个新对象并且新对象的成员声明都是构造函数里定义的。在构造函数内部，this关键字引用的是新创建的对象。基本用法如下：

```javascript
    function Car(model, year, names) {
        this.model = model;
        this.year = year;
        this.names = names;
        this.output= function () {
            return this.model + "喜欢" + this.names;
        };
    }

    var tom= new Car("大叔", 30, '萝莉');
    var dudu= new Car("欧巴", 24, '御姐');

    console.log(tom.output());
    console.log(dudu.output());
```

上面是一个非常简单的构造函数模式，但是使用继承就很麻烦了，而且output()在每次创建对象的时候都重新定义了，最好的方法是让所有Car类型的实例都共享这个output()方法，这样如果有大批量的实例的话，就会节约很多内存。
解决方法如下：
```javascript
    function Car(model, year, names) {
        this.model = model;
        this.year = year;
        this.names = names;
        this.output= formatCar;
    }

    function formatCar() {
        return this.model + "喜欢" + this.names;
    }
```

这个方法虽然可用,但是我们还有更好的办法哟!

### 构造函数与原型
JavaScript里函数有个原型属性叫prototype，当调用构造函数创建对象的时候，所有该构造函数原型的属性在新创建对象上都可用.下面来看一下上面扩展的代码:

```javascript
    function Car(model, year, names) {
        this.model = model;
        this.year = year;
        this.names = names;
    }
    /*
    注意：这里我们使用了Object.prototype.方法名，而不是Object.prototype
    主要是用来避免重写定义原型prototype对象
    */
    Car.prototype.output= function () {
        return this.model + "喜欢" + this.names;
    };

    var tom = new Car("大叔", 33, '萝莉');
    var dudu = new Car("欧巴", 25, '御姐');

    console.log(tom.output());
    console.log(dudu.output());
```

这里，output()单实例可以在所有Car对象实例里共享使用。
另外：我们推荐构造函数以大写字母开头，以便区分普通的函数。

### 强制使用new
如果不是new来创建对象,直接用在全局调用函数的话,this指向的是全局对象window,下面来验证一下:

```javascript
    //作为函数调用
    var tom = Car("大叔", 30, '萝莉');
    console.log(typeof tom); // "undefined"
    console.log(window.output()); // "大叔喜欢萝莉"
```

这个时候的tom是undefined,而window.output()会正确输出结果,而如果使用new关键字则没有这个问题,验证如下:

```javascript
    //使用new 关键字
    var tom = new Car("大叔", 30, '萝莉');
    console.log(typeof tom); // "object"
    console.log(tom.output()); // "大叔喜欢萝莉"
```

上述的例子展示了不使用new的问题，那么我们有没有办法让构造函数强制使用new关键字呢，答案是肯定的，上代码：
```javascript
    function Car(model, year, names) {
        if (!(this instanceof Car)) {
            return new Car(model, year, names);
        }
        this.model = model;
        this.year = year;
        this.names = names;
        this.output = function () {
            return this.model + "喜欢" + this.names;
        }
    }

    var tom = new Car("大叔", 32, '萝莉');
    var dudu = Car("欧巴", 25, '御姐');

    console.log(typeof tom); // "object"
    console.log(tom.output()); // "大叔喜欢萝莉"
    console.log(typeof dudu); // "object"
    console.log(dudu.output()); // "欧巴喜欢御姐"
```
通过判断this的instanceof是不是Car来决定返回new Car还是继续执行代码，如果使用的是new关键字，则(this instanceof Car)为真，会继续执行下面的参数赋值，如果没有用new，(this instanceof Car)就为假，就会重新new一个实例返回。
