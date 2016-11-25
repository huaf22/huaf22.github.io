除了通过构造函数创建对象之外，你也可以通过对象初始化器创建对象。
使用对象初始化器也被称作通过字面值创建对象。

<!--more-->

```
var obj = { property_1:   "value_1",   // property_# may be an identifier...
            2:            "value_2",   // or a number...
            // ...,
            "property n": "value_n" }; // or a string

console.log(Object.getPrototypeOf(obj))
console.log(Object.getPrototypeOf(Object.getPrototypeOf(obj))) 

// obj --> {} ---> null
```
```
function Car(make, model, year, owner) {
  this.make = make;
  this.model = model;
  this.year = year;
  this.owner = owner;
}

var car1 = new Car("Eagle", "Talon TSi", 1993, rand);
var car2 = new Car("Nissan", "300ZX", 1992, ken);

car1.color = "black";
```
为 car1 增加了 color 属性，并将其值设为 "black." 然而，这并不影响其他的对象。
想要为某个类型的所有对象增加新属性，必须将属性加入到 car 对象类型的定义中。

* 所有的 JavaScript 对象继承于至少一个对象。被继承的对象被称作原型，并且继承的属性可通过构造函数的prototype 对象找到。

#### 子类和继承
[原文地址](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Details_of_the_Object_Model)

基于类的语言是通过对类的定义中构建类的层级结构的。在类定义中，可以指定新的类是一个现存的类的子类。子类将继承父类的全部属性，并可以添加新的属性或者修改继承的属性。例如，假设 Employee 类只有 name 和 dept 属性，而 Manager 是 Employee 的子类并添加了 reports 属性。这时，Manager 类的实例将具有所有三个属性：name，dept 和 reports。

JavaScript 通过将构造器函数与原型对象相关联的方式来实现继承。这样，您可以创建完全一样的 Employee — Manager 示例，不过需要使用略微不同的术语。首先，定义 Employee 构造器函数，指定 name 和 dept 属性；然后，定义 Manager 构造器函数，指定 reports 属性。最后，将一个新的 Employee 对象赋值给 Manager 构造器函数的 prototype 属性。这样，当创建一个新的 Manager 对象时，它将从 Employee 对象中继承 name and dept 属性。



http://shiningray.cn/docs/self-the-power-of-simplicity
http://www.cnblogs.com/winter-cn/archive/2009/05/16/1458390.html
