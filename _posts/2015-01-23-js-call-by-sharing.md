---
layout: post
title: 解析JS的值传递方式
description: ""
modified: 2015-01-23
category: articles
tags: [JS]
imagefeature: cover6.jpg
image:
  feature: texture-feature-05.jpg
  credit: Texture Lovers
  creditlink: http://texturelovers.com
---

<style type="text/css">
    .trans {
        font-size:12px;
        color:#999;
    }
</style>
<section id="table-of-contents" class="toc">
  <header>
    <h3>Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

准确的说，JavaScript是按共享传递的(call by sharing)。

对于基本类型，按共享传递与按值传递(call by value)无异，你可以理解为JavaScript基本类型是按值传递的。
例：
{% highlight javascript %}
var a = 1;
function foo(x) {
    x = 2;
}
foo(a);
console.log(a); // 1, 未受x = 2赋值所影响
{% endhighlight %}

而对于对象，较为特殊。我们可以修改对象中的属性，而变量的值是对象的引用，例如：

{% highlight javascript %}
var obj = {x : 1};
function foo(o) {
    o.x = 3;
}
foo(obj);
console.log(obj.x); // 3, 被修改了
{% endhighlight %}

但与按引用传递不同，修改形参o并不会影响obj.

{% highlight javascript %}
var obj = {x : 1};
function foo(o) {
    o = 100;
}
foo(obj);
console.log(obj.x); // 1, 不会被o = 100影响
{% endhighlight %}

为什么叫“按共享传递”呢？JavaScript的类型中，基本类型是不可变的(immutable)，只有对象是可变的(mutable).
例如数字值100, 布尔值true, false，修改这些值并没有什么意义。比较容易误解的，是JavaScript中的string。但实际string值也是不可变的，例如：

{% highlight javascript %}
var str = "string";
str.substring(5); // "g"
{% endhighlight %}

在JavaScript中，任何看似对string值的"修改"操作，实际都是创建新的string值。
而对象就不一样了，对象值是可变的。

{% highlight javascript %}
var obj = {x : 1};
obj.x = 100;
var o = obj;
o.x = 1;
obj.x; // 1, 被修改
o = true;
obj.x; // 1, 不会因o = true改变
{% endhighlight %}

例如这里定义变量obj，值是object，然后设置obj.x属性为100。而后定义另一个变量，值仍然是这个object对象，此时obj和o两个变量的值指向同一个对象（共享同一个对象的引用）。所以修改这个可变的对象，对obj和o都有影响，但修改obj变量的值（不再指向这个对象)，不会影响到o。

所以这里既不是按值传递，也不是按引用传递。而是按共享传递。



#### 作者&鸣谢

* 作者：[Bosn](http://bosn.me)
* 发布时间：2015年1月23日
* 参考文献：[Wiki-Evaluation strategy](http://en.wikipedia.org/wiki/Evaluation_strategy#Call_by_reference)
