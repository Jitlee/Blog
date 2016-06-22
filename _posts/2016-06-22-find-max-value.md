---
layout: post
title: 从数组中找出最大数字
description: 需要从js数组中找出最大的数字（数字字符串算不算？）
categories: [HTML5]
image: 
geometry: 
tags: [JS,算法]
---

>已知数组
{% highlight javascript %}
var arr = ["One", "Four", 1.2, { name: "pet", num: 2 }, 3.9, "浏览器", 0.5, "5", "a", function() { return 0; }, 3];
{% endhighlight %}

### 1.循环比较

{% highlight javascript %}

var result = Number.MIN_SAFE_INTEGER;
for(var i = 0, len = arr.length; i < len; i++) {
	if(!isNaN(arr[i]) && arr[i] > result) {
		result = arr[i];
	}
}
console.log("最大的数字是：" + result);

{% endhighlight %}

### 2.使用数组的sort方法倒序取第一个

{% highlight javascript %}

var result = arr.sort(function(a, b){ return isNaN(a) ? 1 : (isNaN(b) ? -1 : (b - a)) })[0];
console.log("最大的数字是：" + result);

{% endhighlight %}

### 3.使用数组的filter方法和sort方法结合

{% highlight javascript %}

var result = arr.filter(function(a) { return !isNaN(a); }).sort().pop();
console.log("最大的数字是：" + result);

{% endhighlight %}


