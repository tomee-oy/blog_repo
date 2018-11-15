---
title: JavaScript对象创建
date: 2017-01-13 13:55:02
tags: 
  - js
categories: 技术
---
网上看到的很多阐述JS创建对象的方法有3种，而我认为有4种，很多博文没有把第一种方式纳入其中，但我个人认为，尽管这种方式是最low的，但不得不承认它也正是我们平常最喜欢用的创建方式。下面就一一分析一下各个创建方式的优劣：
## 1、基于Object或对象字面量的创建方式
### ①基于Object创建
```bash
 var obj=new Object();
 obj.name='xiaoming';
 obj.age=25;
 obj.say=function(){
     alert(this.name);
 }
```
### ②使用对象字面量创建
```bash
var obj={
    name:'xiaoming',
    age:25,
    say:function(){
        alert(this.name);
    }
}
```
刚刚说了，这两种方式是最简单直接的创建方式。为什么要把这两种方式归为同一类，因为它们的本质和原理完全一样，只是书写形式不同，从效率上来说并无区别。但这两种创建方式的缺点也是显而易见的，假如说我要创建很多个实例，就意味着我要重复多遍相同的代码，这让程序显得非常臃肿，极不科学。对此，第二种创建方式应运而生。
## 2、工厂模式
所谓工厂模式，就要将创建对象的任务封装在一个工厂方法中，在我们需要对象的时候，直接调用这个方法就可以轻松得到一个对象：
```bash
function createObject(name,age){
    var obj=new Object();
    obj.name=name;
    obj.age=age;
    obj.say=function(){
        alert(this.name)
    }
    return obj;
}
```
这种创建方式在我们需要多个实例的情况下，就不必像第一种情况一样手写多段重复的代码，比如，可以采用如下的方式创建多个对象：
```bash
var xiaoming=createObject('xiaoming',25);
var xiaoqiang=createObject('xiaoqiang',30);
```
这种方式的优点在于可以避免众多重复的代码，程序更加简洁。但缺点在于，在一个复杂的系统中，我们并不能去识别对象的类型（这个概念可能不太好懂，结合下面第三种创建方式看就明白了）。
## 3、构造函数模式
构造函数模式解决了第二种创建方式的缺陷：
```bash
    function Person(name,age){
        this.name=name;
        this.age=age;
        this.say=function(){
            alert(this.name);
        }
    }
```
这种方式跟强语言的OOP创建对象的方式就很像了，只不过不是采用class关键字定义而已（ES6已经引入了class的概念）。我们可以像这样创建对象的实例：
```bash
var xiaoming=new Person('xiaoming',25);
var xiaoqiang=new Person('xiaoqiang',30);
```
我们可以用`xiaoming instanceof Person`判断xiaoming是不是Person的一个实例，如果是，这个表达式就返回true，反之返回false。这就是上面第二种创建方式中提到的不能识别对象类型的解决方案。这种创建方式并非最优，原因在于每创建一个实例都要创建一份实例方法，对象方法在每个实例中重复创建，造成内存浪费。
## 4、原型模式
我们创建的每一个函数都有一个prototype属性，这个属性是一个指针，这个指针指向一个对象，即原型对象。原型对象的用途在于存储同一个类型创建的不同实例的共享属性和方法。由此看来，prototype正是我们在第三种创建方式中遇到的问题的解决方案。
```bash
function Person(name,age){
    this.name=name;
    this.age=age;
}
Person.prototype.say=function(){
    alert(this.name);
}
```
这样，所有Person的实例都可以有自己的属性值和公用的say方法。当然，还有一种写法，就是重写Person的原型对象，但要注意的是，这种方式切断了原型对象与构造函数之间的指针关联，所以需要在新的原型对象中将constructor重新指向Person构造函数。
```bash
function Person(name,age){
    this.name=name;
    this.age=age;
}
Person.prototype={
    <font color=red>constructor:Person,</font>
    say:function(){
        alert(this.name);
    }
}
```
至此，4种创建对象的方式就介绍完了，这4种方式层层优化，第4种方式当然是最好的创建对象的方式。

接下来介绍几个关于对象和实例用得着的方法:
***
**`isPrototypeOf()`**
此方法用来判断某个原型对象是否是某个实例的原型对象，如果是，返回true，否则返回false,用法：
```bash
alert(Person.prototype.isPrototypeOf(xiaoming)); //true
```
***
 **`Object.getPrototypeOf()`**
此方法用于获取某个实例的原型对象。返回值是一个对象。用法：
```bash
alert(Object.getPrototypeOf(xiaoming)); //Person
```
***
 **`hasOwnProperty()`**
此方法用于检测某个属性是原型属性还是实例属性。如果是实例属性，返回true,否则返回false。<font color=red>注意：如果这个属性在实例和原型对象中都找不到，同样会返回false。</font>
```bash
alert(xiaoqiang.hasOwnProperty('name')); //true
alert(xiaoqiang.hasOwnProperty('sex')); //false
```
***
 **`in操作符`**
用于判断实例是否能够访问到给定的属性。用法：
```bash
alert('age' in xiaoqiang); //true
alert('sex' in xiaoqiang); //false
```
因此，in操作符结合hasOwnProperty()方法就可以判断某属性到底是实例属性还是原型对象属性，判断条件：
```bash
alert(xiaoming.hasOwnProperty(age) && (age in xiaoming)); //true
Person.prototype.sex='male';
alert(xiaoming.hasOwnProperty(sex) && (sex in xiaoming)); //false
```
如果值为false还有一种情况，就是被检测的属性既不是实例属性也不是原型对象属性，即未定义。

