---
layout: post
title: HTML5之基于Canvas打造2D设计器
description: 使用Canvas的最基本功能，创建复杂的控件和完美的设计器。基于Javascript面向对象的设计，提升框架的可扩展性、易用性
categories: [HTML5]
image: /assets/media/canvas_design.jpg
geometry: /assets/media/canvas_desgin_control.jpg
tags: [Canvas,JS,面向对象，设计器]
---

>前几年帮一朋友把一款Winform的程序，做成Silverlight版，再后来又改造成WPF版的。
现在说要做HTML5版的，没办法继续折腾吧！

*由于商业原因，本文演示的DEMO和功能并不包含全部的项目内容*

*本文的设计思想是完全仿照[OpenLayer2.0](http://openlayers.org/two/)，如有不当之处，敬请批评指正*

[demo](http://jitlee.github.io/demo/monitor.html)

效果图：

<img src="{{ site.BASE_PATH }}{{page.image}}" width="85%"/>

---

### 1.实现Javascript的面向对象功能

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

设计器的最大容器, 装载所有的图层、控件、和图形

{% highlight javascript %}

Monitor.Map = Monitor.Class({
	div:null, // DIV
	
	viewportDiv: null, // 可是范围DIV
	
	layers: null,  // 所有图层
	
	controls: null, // 所有控件
	
	options: null, // 配置参数
	
	init: function(div, options) {}, // 构造函数
	
	addLayer: function(layer) {}, // 添加图层
	
	removeLayer: function(layer) {}, // 移除图层
	
	addControl: function(control) {}, // 添加控件
	
	ClASS_NAME: "Monitor.Map"
});

{% endhighlight %}

### 3.创建图层基类和矢量图层类

矢量图层实际是一张画布，一个设计器需要多层画布。类似与PS中的层的概念。

{% highlight javascript %}

Monitor.Layer.Vector = Monitor.Class(Monitor.Layer, {

	options: null,			
	
	renderer: null,			// 图形渲染器
	
	map: null,					
	
	features: null,			// 所有要素
	
	init: function(name, options) {}, // 构造函数
	
	setMap: function(map) {}, // 设置map对象
	
	addFeatures: function(features) {}, // 添加要素并且渲染
	
	removeFeatures: function(features) {}, // 移除要素
	
	getFeatureFromXY: function(xy) {},  // 鼠标探测要素信息
	
	redraw: function() {}, // 重绘
	
	ClASS_NAME: "Monitor.Layer.Vector"
});

{% endhighlight %}

### 4.创建修改控件类

实现对控件的移动和缩放功能

{% highlight javascript %}

Monitor.Control.ModifyFeature =  Monitor.Class(Monitor.Control, {
	
	layer: null, 						// 临时图层
	
	layers: null,						// 能够编辑的图层数组
	
	selectedFeature: null,	// 当前选择的要素
	
	selectedFeatures: null,	// 当前选中的要素数组
	
	locked: false,					// 是否锁定
	
	knobSize: 20,						// 编辑图形的手柄大小，单位px
	
	resizeFeature:null,			// 当前缩放的要素对象
	
	events: null,						// 消息总线
	
	init: function(layers, options) {},	// 构造函数
	
	dragstart: function(xy) {},		// 开始拖动
	
	dragmove: function(xy) {},		// 拖动
	
	dragend: function(xy) {},			// 拖动结束
	
	moveFeatures: function(xy) {},	//  移动控件
	
	resizeFeatures: function(xy) {},	// 缩放控件
	
	select: function(feature) {},			// 高亮要素
	
	unselect: function(feature) {},		// 反高亮要素
	
	unselectAll: function(exceptFeature) {}, // 取消所有高亮
	
	activate: function() {},	// 激活
	
	deactivate: function() {},	// 反激活
	
	CLASS_NAME: "Monitor.Control.ModifyFeature"
	
});

{% endhighlight %}

### 5.创建Renderer类

图形渲染类，使用Canvas渲染作为核心渲染器，以后可以扩展到SVG，甚至VML。
由于设计过于匆忙，还未来得及实现绘制接口，实际项目中，是直接调用Canvas的接口。

{% highlight javascript %}

Monitor.Renderer = Monitor.Class({

	layer: null,		// 所在矢量图层
	
	canvas: null,		// 画布
	
	context: null,	// 画布上下文
	
	matrix: null,		// 变换矩阵
	
	init: function(layer, options) {}, // 构造函数
	
	updateSize: function() {},	// 重绘 
	
	setOrigin: function(x, y) {}, // 设置原点
	
	setStyle: function(pen) {},	// 设置画笔、填充和字体样式等
	
	setTransform: function(matrix) {}, // 设置矩阵变换
	
	eraseFeatures: function(features) {}, // 擦除要素
	
	erase: function() {},	// 擦除全部
	
	clear: function(x, y, width, height) {}, // 擦除指定区域
	
	drawRectangle: function(pen, x, y, width, height, matrix) {}, // 绘制矩形
	
	drawLine: function(pen, x1, y1, x2, y2, matrix) {}, // 绘制
	
	drawCircle: function(pen, x, y, r, matrix) {}, // 绘制正圆
	
	drawText: function(pen, text, x, y, alginX, alginY, matrix) {}, // 绘制文本
	
	CLASS_NAME: "Monitor.Renderer"
	
});

{% endhighlight %}

### 6.创建Geometry基类

Geometry即为我们刚刚创建设计器需要设计的对象了

{% highlight javascript %}

Monitor.Geometry = Monitor.Class({
	id: null,
	width: 0,	// 基本属性：宽度
	height:0, // 基本属性：高度
	x: 0, 	// 基本属性：x
	y: 0, 	// 基本属性：y
	backgroundColor: "#fff",	// 基本属性：背景色
	foregroundColor: "#333", 	// 基本属性：字体色
	minWidth: 50,		// 基本属性：最小宽度
	minHeight:50,		// 基本属性：最大宽度
	bounds: null,		// 边界
	init: function(options) {}, // 构造函数
	
	draw: function(renderer) {}, // 绘制
	
	isPointInRange: function(x, y) {}, // 点是否在范围内
	
	move: function(x, y) {},	// 移动
	
	resize: function(w, h) {},	// 缩放
	
	getBounds: function() {},	// 获取边界
	
	CLASS_NAME: "Monitor.Geometry"
});

{% endhighlight %}

### 7.扩展图形类

由于项目用到的图形控件多达几十种，无法一一列出，我抽取其中一种来处来展示

效果图如下：
<img src="{{ site.BASE_PATH }}{{page.image}}" width="85%"/>


{% highlight javascript %}

var DLBiaoPan = Monitor.Class(Monitor.Geometry, {
	min: 0,
	max: 100,
	value: 0,
	backgroundColor: "#ddd",
	title: "仪表",
	
	init: function(options) {
		Monitor.Geometry.prototype.init.apply(this, arguments);
	},
	
	draw: function(renderer) {
		Monitor.Geometry.prototype.draw.apply(this, arguments);
		var w = this.width;
		var h = this.height;
		
		renderer.drawRectangle({ fill: this.backgroundColor }, 0,0,w,h);
		
		renderer.drawLine({ stroke: "#eee", strokeThickness: 3 },0,0,w,0);
		renderer.drawLine({ stroke: "#eee", strokeThickness: 3 },0,0,0,h);
		renderer.drawLine({ stroke: "#aaa", strokeThickness: 3 },w,0,w,h);
		renderer.drawLine({ stroke: "#aaa", strokeThickness: 3 },0,h,w,h);
		
		renderer.drawCircle({ fill: "#333", stroke:"#bbb", strokeThickness: 0.02*w }, w*0.08, h*0.08, w*0.03);
		renderer.drawCircle({ fill: "#333", stroke:"#bbb", strokeThickness: 0.02*w }, w*0.92, h*0.08, w*0.03);
		renderer.drawCircle({ fill: "#333", stroke:"#bbb", strokeThickness: 0.02*w }, w*0.08, h*0.92, w*0.03);
		renderer.drawCircle({ fill: "#333", stroke:"#bbb", strokeThickness: 0.02*w }, w*0.92, h*0.92, w*0.03);
		
		renderer.drawCircle({ fill: "#274e13", stroke:"#333", strokeThickness: 1 }, w*0.5, h*0.5, w*0.41);
		renderer.drawCircle({ fill: "#fff", stroke:"#333", strokeThickness: 1 }, w*0.5, h*0.5, w*0.38);
		
		renderer.drawCircle({ fill: "#333" }, w*0.5, h*0.5, w*0.02);
		
		renderer.drawText({ fill:this.foregroundColor, font: w*0.08+ "px 宋体" }, this.title, w*0.5, h * 0.75, -0.5, 0.5);
		
		var start = 0.75*Math.PI;
		var degree, preDegree;
		var style = { stroke:"#333", strokeThickness: 0.008*w };
		var cos,sin;
		{
			preDegree = 1.5*Math.PI / 20.0;
			for(var i = 0; i < 21; i++) {
				degree = start + preDegree * i;
				sin = Math.sin(degree);
				cos = Math.cos(degree);
				renderer.drawLine(style,w*0.5 + w*0.32*cos,h*0.5+h*0.32*sin,w*0.5 + w*0.36*cos,h*0.5+h*0.36*sin);
			}
		}
		{
			var textStyle = { fill:"#333", font: w*0.05+ "px 宋体" };
			preDegree = 1.5*Math.PI / 10.0;
			var perValue = (this.max - this.min) / 10.0;
			style.strokeThickness = 0.01*w;
			for(var i = 0; i < 11; i++) {
				degree = start + preDegree * i;
				sin = Math.sin(degree);
				cos = Math.cos(degree);
				renderer.drawLine(style,w*0.5 + w*0.32*cos,h*0.5+h*0.32*sin,w*0.5 + w*0.36*cos,h*0.5+h*0.36*sin);
				renderer.drawText(textStyle, i * perValue, w*0.5+w*0.24*cos, h*0.5+h*0.24*sin, -0.5, 0.5);
			}
		}
		
		style.stroke = "red";
		style.strokeThickness = 0.02*w;
		degree = start + 1.5*Math.PI * this.value / (this.max - this.min);
		sin = Math.sin(degree);
		cos = Math.cos(degree);
		renderer.drawLine(style,w*0.5,h*0.5,w*0.5 + w*0.30*cos,h*0.5+h*0.30*sin);
		
		this.getBounds();
	},
	
	CLASS_NAME: "DLBiaoPan"
});


{% endhighlight %}


