---
title: JavaScript继承
date: 2017-01-17 15:59:50
tags: [js,继承]
categories: 技术
---
&emsp;&emsp;前面总结了JS对象的创建，既然有对象的创建，自然就离不开继承，本篇将总结一下JS几种实现继承的方法：
## 一、原型链继承
***
&emsp;&emsp;我们知道每个对象都有一个指定的原型对象，这个原型对象是可以被所有对象的实例访问到的，因此可以采用将子类的原型指针指向要继承的父类的实例。代码如下：
```javascript
function Parent(){
    this.name='tom';
    this.age=25;
}
Parent.prototype.sayName=function(){
    alert('我是'+this.name);
}
function Child(){
    this.sex='male';
}
Child.prototype=new Parent(); //继承
Child.prototype.constructor=Child;
var child=new Child();
child.sayName();
```
&emsp;&emsp;这种继承方式存在两个严重的问题：
> 1、由于原型对象中的属性和方法可以被所有的实例共享，因此，假如某个属性值是个引用类型，比如：数组。那么当一个实例当中修改了数组的值，其余实例中的该属性值都会发生变化。这显然是不科学的；
2、子类在继承父类的时候，无法动态地给父类构造函数传递参数。因此，这种情况下即使子类继承到了父类的属性也没有意义，因为父类中的属性没办法取到值。你可能会说：我们可以在`Child.prototype=new Parent();`这一句的时候给父类属性赋值。但请注意，这一句只会执行一次，那就是在为Child重新指定原型对象的时候，而并不会在每次创建实例的时候执行。所以，即使在这一句能设值，那也是在创建实例时不能改变的。

## 二、借用构造函数继承
***
&emsp;&emsp;针对第一种继承方式的问题，第二种继承方式有所改善，构造函数继承方式可以向父类构造函数传递参数了。直接上代码说明：
```javascript
function Parent(name,age){
    this.name=name;
    this.age=age;
    this.book=['book1','book2','book3'];
    this.sayAge=function(){
        alert(this.age);
    }
}
Parent.prototype.sayName=function(){
    alert('我是'+this.name);
}
function Child(name,age){
    Parent.apply(this,arguments);// 继承
    this.sex='male';
}
var child=new Child('Jerry',22);
child.sayAge(); //22
child.sayName(); //Uncaught TypeError: child.sayName is not a function
```
&emsp;&emsp;把这段代码复制到控制台运行，结果是，可以弹出22，但运行到sayName()方法的时候，程序找不到sayName方法。尽管构造函数方式也能实现继承，但它的缺点也是显而易见的：
> 1、存在于父类中的方法没办法实现共享，这跟创建对象时的构造函数方式有着相同的问题，每创建一个实例也要创建一份同样的方法，造成内存浪费
2、刚刚的报错已经提示很明显了，为什么会找不到sayName方法？因为子类实例根本无法访问到父类原型对象的属性和方法！

## 三、组合继承
***
&emsp;&emsp;组合继承，是由原型链继承和借用构造函数继承两种方式结合，取二者之所长形成的一种继承模式，因此叫做组合继承。
```javascript
function Parent(name,age){
    this.name=name;
    this.age=age;
    this.book=['book1','book2','book3'];
}
Parent.prototype.say=function(){
    alert('我是'+this.name+'，我的书：'+this.book);
}

function Child(name,age,sex){
    Parent.apply(this,arguments); //继承属性
    this.sex=sex;
}
Child.prototype=new Parent(); //继承公用方法
Child.prototype.constructor=Child;
var child1=new Child('tom',25,'male');
var child2=new Child('jerry',20,'female');
child2.book.push('book4');
child1.say(); //我是tom，我的书：book1,book2,book3
child2.say(); //我是jerry，我的书：book1,book2,book3,book4
```
&emsp;&emsp;运行这段代码，你会发现，say方法是放在父类原型对象中的，程序可以正常访问到，而且，不同的实例各自有一份引用类型的实例，一个实例对数组的改变不会影响其他实例。这种模式就要求程序员在创建对象的写法上要规范了，如果我们把公用方法写在父类的构造函数中，无论我们采用哪种继承方式，都会有重复的方法被创建占用内存。
&emsp;&emsp;这种继承方式已经解决了之前提出的所有问题，算是继承中用得最多的一种模式。但它也并非最优，**实际上我们调用了两次父类的构造函数：一次是在为子类创建原型对象的时候，另一次是在执行子类构造函数的时候也会调用父类的构造函数**。
## 四、寄生组合式继承
***
&emsp;&emsp;其实，仔细想想，我们执行`Child.prototype=new Parent();`这一句，无非就是想让子类的原型指针指向父类的原型对象嘛，那我们没必要创建父类的实例，我们只需要有一份父类原型对象的拷贝不就行了？
```javascript
function object(o){
    function F(){}
    F.prototype=o;
    return new F();
} 
function Parent(name,age){
    this.name=name;
    this.age=age;
    this.book=['book1','book2','book3'];
}
Parent.prototype.say=function(){
    alert('我是'+this.name+'，我的书：'+this.book);
}

function Child(name,age,sex){
    Parent.apply(this,arguments); //继承属性
    this.sex=sex;
}
var prototype=object(Parent.prototype); //这里的prototype就是子类需要继承的原型对象**
prototype.constructor=Child; //把原型对象的constructor属性重新指向构造函数
Child.prototype=prototype; //将原型指针指向新的原型对象
var child1=new Child('tom',25,'male');
var child2=new Child('jerry',20,'female');
child2.book.push('book4');
child1.say(); //我是tom，我的书：book1,book2,book3
child2.say(); //我是jerry，我的书：book1,book2,book3,book4
```
&emsp;&emsp;这种方式单独定义了一个object方法，这个方法返回一个空对象的实例，这个实例的原型指针指向Parent的原型对象。所以，我们只需要将子类的原型指针指向object方法返回的这个对象，就可以继承到父类的原型对象的方法和属性了。
&emsp;&emsp;这种方法避免了两次调用父类构造函数，用一个不包含任何自定义属性的对象来替代之前创建父类实例进行原型继承的方式。