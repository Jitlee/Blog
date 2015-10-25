---
layout: post
title: JavaScript之关键字this
description: Javascript作为因特网上最流行的脚本语言，很容易使用，你一定会喜欢它。各种强大的js库，都离不开它都基本语法。本文，为将给大家介绍Javascript中无处不在的this对象的用法。
tagline:
image:
categories: [JavaScript]
tags: [js, 面向对象, this, 指针, 前端]
---

>在其他语言(如C++，C#，Java)中，this指针只能在非静态类的成员函数中调用，它表示当前对象的地址，this作用域在类的内部。

在JavaScript中，this有所不同：

* this指向函数调用的所有者

* 如果函数没有调用者，则函数内部的this指向浏览器的全局变量window对象

JavaScript是一种很灵活的语言，this关键字又是灵活中的灵活，随着函数的使用场合的不同，this的值会发生变化。

下面列举初this的5类用法，详细讨论this的用法：

### 1. 全局代码中的this

{% highlight javascript %}

alert(this); // window

{% endhighlight %}

浏览器中，全局对象为window对象，this则指向window

### 2. 纯粹的函数调用

{% highlight javascript %}

var name = "I am foo";
function foo() {
	alert(this.name);
}
foo(); // I am foo

{% endhighlight %}

由于在JavaScript中，任何一个全局的函数或变量都是window的属性。
所以上述代码中name和foo都是所属与window对象，
foo函数的调用者实际是window，所以foo函数内部的this也是window。

以下代码输出为真：

{% highlight javascript %}
function foo() {
	alert(window == this); // true
}
foo();

{% endhighlight %}

### 3.作为对象的方法调用

{% highlight javascript %}
var name = "I am the window";
var a = {
	name: "I am the a",
	b: function() {
		alert(this.name);
	}
}
a.b(); // I am the a

{% endhighlight %}

上述代码中，b作为对象a的方法被调用，则b内部的this指向它的调用者

接下来分别对上述代码进行小小的改动，看输出的结果：


1.第一种情况：
{% highlight javascript %}
var name = "I am the window";
var a = {
	name: "I am the a",
	b: function() {
		alert(this.name);
	}
}
var c = a.b;
c(); // I am the window

{% endhighlight %}
导致这种结果，还是由于函数c没有调用者，尽管它指向对象a的内部函数b 

2.第二种情况：
{% highlight javascript %}
var name = "I am the window";
var a = {
	name: "I am the a",
	b: function() {
		var c = function() {
			alert(this.name);
		}
		c();
	}
}
a.b(); // I am the window

{% endhighlight %}
c作为内部函数，但是执行时没有调用者，所以c的内部this指向全局对象window  

对于第二种情况，我们在实际开发过程中经常会遇到。
函数c往往作为**回调**函数，当作参数传递，并且c的调用者不能被我们所左右。
对于这种内部函数需要使用外层的this，一般的处理方式是将this作为变量(self,that)保存下来:

{% highlight javascript %}

var name = "I am the window";
var a = {
	name: "I am the a",
	b: function() {
		var self = this;
		var c = function() {
			alert(self.name);
		}
		c();
	}
}
a.b(); // I am the a

{% endhighlight %}

### 4.使用new作为构造函数调用

new关键字可以创建一个新对象，也就是一个Object的实例

{% highlight javascript %}

var name = "I am the window";
function A() {
	this.name = "I am the A";
	this.getName = function() {
		return this.name;
	}
	this.setName = function(name) {
		this.name = name;
	}
}

var a = new A();
alert(a.name); // I am the A
alert(a.getName()); // I am the A

var newName = "I am the new name of A";
a.setName(newName);
alert(a.name); // I am the new name of A
alert(this.getName()); // I am the new name of A

{% endhighlight %}

作为构造函数，函数内部的this指向新创建的对象

### 5.使用apply或call方法调用

>apply和call是函数对象的一个内部方法，他们的作用是改变函数的调用对象，
他们的第一个参数是表示函数的执行对象。

{% highlight javascript %}

var name = "I am the window";
var a = {
	name: "I am the a",
	foo: function(){
		alert(this.name);
	}
}

var b = {
	name: "I am the b"
}
a.foo(); // I am the a
a.foo.apply(b); // I am the b
a.foo.call(b); // I am the b

{% endhighlight %}

实际开发中，apply和call往往用在带有回调函数的方法中，用来改变回调函数的调用对象。

笔者在开发开发API接口时，如果回调函数无需指定调用对象时，会额外给API添加一个scope参数。
如果API的调用者需要回调函数自定义其调用对象时，可以使用scope参数：

{% highlight javascript %}

var name = "I am the window";
function ClassA() {
	this.query = function(scope, callback) {
		var result;
		// begin query
		result = "query is success";
		// end query
		callback.call(scope, result);
	}
}

function ClassB() {
	this.name = "I am class B";
	var a = new ClassA();
	
	function queryCallback(result){
		// result: query is success
		alert(this.name);
	}
	
	this.func1 = function() {
		a.query(this, queryCallback);
	}
	
	this.func2 = function() {
		a.query(a, queryCallback);
	}
	
	this.func3 = function() {
		a.query(null, queryCallback);
	}
}

var b = new ClassB();

b.func1(); // I am class B
b.func2(); // I am class A
b.func3(); // I am the window


{% endhighlight %}

### 总结

- 全局范围内，this指向浏览器的window对象

- 有调用对象的函数，函数内部this指向调用对象

- 没有调用对象的函数，函数指向浏览器的window对象
