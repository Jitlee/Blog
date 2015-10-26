---
layout: post
title: HTML5之基于Canvas打造2D设计器
description: 使用Canvas的最基本功能，创建复杂的控件和完美的设计器。基于Javascript面向对象的设计，提升框架的可扩展性、易用性
categories: [HTML5]
tags: [Canvas,JS,面向对象，设计器]
---

>前几年帮一朋友把一款Winform的程序，做成Silverlight版，再后来又改造成WPF版的。
现在说要做HTML5版的，没办法继续折腾吧！

[demo] (http://jitlee.github.io/demo/monitor.html)

*由于商业原因，本文演示的DEMO和功能并不包含全部的项目内容*

*本文的设计思想是完全仿照[OpenLayer2.0](http://openlayers.org/two/)，如有不当之处，敬请批评指正*

### 1.实现Javascript的面向对象功能

我的底层代码依然使用OpenLayers的，分别用到了：
- OpenLayers.BaseTypes.js
- OpenLayers.Class.js
- OpenLayers.Element.js
- OpenLayers.Util.js
- OpenLayers.Monitor.js
- OpenLayers.Handler.js
- OpenLayers.Handler.Drag.js

下面两段代码分别实现了类(Class)和继承(inherit)：

{% highlight javascript %}

Monitor.Class = function() {
	var len = arguments.length;
    var P = arguments[0];
    var F = arguments[len-1];
    var C = typeof F.init == "function" ?
        F.init :
        function(){ P.prototype.init.apply(this, arguments); };

    if (len > 1) {
        var newArgs = [C, P].concat(
                Array.prototype.slice.call(arguments).slice(1, len-1), F);
        Monitor.inherit.apply(null, newArgs);
    } else {
        C.prototype = F;
    }
    return C;
}

Monitor.inherit = function(C, P) {
	var F = function() {};
	F.prototype = P.prototype;
	C.prototype = new F;
	var i, l, o;
	for(i=2, l=arguments.length; i<l; i++) {
		o = arguments[i];
	    if(typeof o === "function") {
	    		o = o.prototype;
	    }
	    Monitor.Util.extend(C.prototype, o);
	}
}

{% endhighlight %}

### 2.创建Map对象

Map.js在OpenLayers中，原本是地图容器。由于我要打造设计器并不是地图，
所以不需要做那么复杂的地理信息处理。除了以上代码大部分采用OpenLayers原封代码外，
以下代码都是简单的设计下。

### 3.创建图层基类和矢量图层类

矢量图层实际是一张画布，一个设计器需要多层画布。类似与PS中的层的概念。

### 4.创建修改控件类

使用Drag.js实现对控件的移动和缩放功能

### 5.创建Renderer类

OpenLayers原本

