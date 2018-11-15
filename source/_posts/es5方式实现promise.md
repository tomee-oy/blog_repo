---
title: es5方式实现promise
date: 2018-09-22 17:32:20
tags: [js, es6]
categories: 技术
---

&emsp;&emsp;关于如何使用es5来实现es6中`promise`功能，重点在于创建`promise`的时候，它的回调函数是立即执行的，而其内部的`resolve`和`reject`方法又是通过`then`方法传进来的。那么，如何保证运行到调用成功或失败的操作时，其对应的回调函数是已经定义好了的呢？

&emsp;&emsp;创建promise对象时传进去的回调函数是立即执行的。为了保证在执行到成功或者失败的时候，其对应的回调函数已经定义了，我们可以借助`javascript`消息队列的特性来达到我们的目的。

```javascript
function MyPromise(fn){
	this.value
	this.resolvedFun = function(){}
	this.rejectedFun = function(){}
	fn(this.resolve.bind(this), this.reject.bind(this))
}

MyPromise.prototype.resolve = function(val){
	var self = this
	this.value = val
	setTimeout(function(){
		self.resolvedFun(self.value)
	}, 0)
}

MyPromise.prototype.reject = function(val){
	var self = this
	this.value = val
	setTimeout(function(){
		self.rejectedFun(self.value)
	}, 0)
}

MyPromise.prototype.then = function(resolvedFun, rejectedFun){
	this.resolvedFun = resolvedFun
	this.rejectedFun = rejectedFun
}

// 用法如下
var fn=function(resolve, reject){
  console.log('begin to execute!');
  var number=Math.random();
  if(number<=0.5){
    resolve('less than 0.5');
  }else{
    reject('greater than 0.5');
  }
}

var p = new MyPromise(fn);
p.then(function(data) {
  console.log('resolve: ', data);
}, function(data) {
  console.log('reject: ', data);
});

```

解释一下上面的代码：  

&emsp;&emsp;因为`MyPromise`的实例化和调用它的`then`方法，一前一后地被同步调用，在`MyPromise`内部的代码就要访问调用`then`方法时传进去的成功和失败的回调函数。如何才能让`then`方法中的回调函数“后来居上”呢？刚说了，`new MyPromise()`和`then`是一前一后的同步代码，也就是它们的执行是属于同一个事件循环的，我们只要把对失败或者成功的回调放到本次事件循环结束后再调用，就能保证在调用成功或者失败的操作时，它们的回调函数都是已经定义了的。所以我们在`resolve`和`reject`的实现中采用了`setTimeout`包装，在调用`then`方法的时候，将实例的`resolvedFun`和`rejectedFun`方法指向传进来的成功和失败的函数。`setTimeout`在本轮循环结束后才执行，所以，其内部要访问的`resolvedFun `和`rejectedFun `是早就被指定好的，这样，整个流程就已经完整了。

&emsp;&emsp;接下来就是给`MyPromise`加状态，我们知道es6中的`promise`一旦状态确定后就不会再被更改了。在上面的例子中成功和失败的结果都会被输出，这是不合理的。所以将`MyPromise`做如下修改：

```javascript
function MyPromise(fn){
	this.value
	this.status='pending'
	this.resolvedFun = function(){}
	this.rejectedFun = function(){}
	fn(this.resolve.bind(this), this.reject.bind(this))
}

MyPromise.prototype.resolve = function(val){
	var self = this
	if(this.status == 'pending'){
		this.status = 'resolve'
		this.value = val
		setTimeout(function(){
			self.resolvedFun(self.value)
		}, 0)
	}
}

MyPromise.prototype.reject = function(val){
	var self = this
	if(this.status == 'pending'){
		this.status == 'reject'
		this.value = val
		setTimeout(function(){
			self.rejectedFun(self.value)
		}, 0)
	}
}

MyPromise.prototype.then = function(resolvedFun, rejectedFun){
	this.resolvedFun = resolvedFun
	this.rejectedFun = rejectedFun
}
```

修改过后再调用，就只会输入一组结果了，要么成功，要么失败。
接下来就是要实现`MyPromise`的链式调用功能。

```javascript
function MyPromise(fn){
	this.value
	this.status='pending'
	this.resolvedFun = function(){}
	this.rejectedFun = function(){}
	fn(this.resolve.bind(this), this.reject.bind(this))
}

MyPromise.prototype.resolve = function(val){
	var self = this
	if(this.status == 'pending'){
		this.status = 'resolve'
		this.value = val
		setTimeout(function(){
			self.resolvedFun(self.value)
		}, 0)
	}
}

MyPromise.prototype.reject = function(val){
	var self = this
	if(this.status == 'pending'){
		this.status == 'reject'
		this.value = val
		setTimeout(function(){
			self.rejectedFun(self.value)
		}, 0)
	}
}

MyPromise.prototype.then = function(resolvedFun, rejectedFun){
	var self = this
	return new MyPromise(function(resolve_next, reject_next){
		function resolvedFunWrap(){
			var result = resolvedFun(self.value)
			resolve_next(result)
		}
		function rejectedFunWrap(){
			var result = rejectedFun(self.value)
			reject_next(result)
		}
		self.resolvedFun = resolvedFunWrap
		self.rejectedFun = rejectedFunWrap
	})
}

```

&emsp;&emsp;之前的`then`方法很简单，只需要将成功和失败的回调函数赋给`MyPromise`实例的两个对应方法即可。但要实现链式调用的话，首先证明`then`方法返回的是一个`MyPromise`对象，其次就是本次成功或失败的结果要作为下一次成功或失败调用的函数的参数传进去。所以要对外层传进来的成功和失败的回调函数进行包装。
