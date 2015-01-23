---
layout: post
title: 解析JS值的传递方式
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
最近在面试、分享的过程中，遇到了一个有趣的问题：“JavaScript中的值是按值传递，还是按引用传递呢？”

在分析这个问题之前，我们需要了解什么是按值传递(call by value)，什么是按引用传递(call by reference)。了解的请跳过。

### 按值传递 VS. 按引用传递

按值传递是最常用的计算策略。在按值传递时，函数的形参是被调用时所传实参的副本。修改形参的值并不会影响实参。

在按引用传递时，函数的形参接受实参的*引用*，而不再是一个副本。这意味着函数形参如果被修改，传入的实参也会被修改。按引用传递会使函数调用的追踪更加困难，有时也会引起一些微妙的BUG。

我们看一个C的例子(取自Wiki)：

{% highlight c %}
void Modify(int p, int * q, int * o)
{
    p = 27; // 按值传递 - 只有局部变量p被修改
    *q = 27; // 按值传递或按引用传递, 取决于调用函数时如何传参
    *o = 27; // 按值传递或按引用传递, 取决于调用函数时如何传参
}
int main()
{
    int a = 1;
    int b = 1;
    int x = 1;
    int * c = &x;
    Modify(a, &b, c);   // a 是按值传递的, b通过创建一个指针来实现按引用传递,
                        // c 是一个指针，该指针按值传递
    // b 和 x 都改变了， a 未变化
    return(0);
}

{% endhighlight %}

这里我们可以看到：
* p按值传递时，修改形参p的值(p = 27)并不会影响实参(a)。
* b是按引用传递，修改形参q(*q = 27)时也影响到了实参b。
* c是一个指针，c是按值传递给了形参o，通过修改该指针指向的实际变量(x)，修改了x的值。但修改指针o的值并不会影响指针p的值。

### 探究JS值的传递方式

JavaScript的基本类型，看起来是按值传递的。

{% highlight javascript %}
var a = 1;
function foo(x) {
    x = 2;
}
foo(a);
console.log(a); // 扔为1, 未受x = 2赋值所影响
{% endhighlight %}

可是对于对象类型，看起来又是按引用传递。

{% highlight javascript %}
var obj = {x : 1};
function foo(o) {
    o.x = 3;
}
foo(obj);
console.log(obj.x); // 3, 被修改了!
{% endhighlight %}

可是如果是按引用传递，修改形参的*值*，应该影响到实参的值才对。

{% highlight javascript %}
var obj = {x : 1};
function foo(o) {
    o = 100;
}
foo(obj);
console.log(obj.x); // 仍然是1, obj并未被修改为100.
{% endhighlight %}

因此JavaScript中的对象并不是按引用传递，实际上，JS中所有的值，都是按*共享*传递。

### 按共享传递 call by sharing

准确的说，JavaScript中的所有值都是按共享传递的(call by sharing，有时也叫按对象传递、按对象共享传递)。最早由Barbara Liskov et al. 在1974年的GLU语言中提出。该计算策略被用于Python、Java、Ruby、JavaScript等等。

按共享传递与按引用传递的不同在于：在共享传递中对函数形参的赋值，不会影响实参的值。例如，下面例子中，不可以通过修改形参o的值，来修改obj的值。

{% highlight javascript %}
var obj = {x : 1};
function foo(o) {
    o = 100;
}
foo(obj);
console.log(obj.x); // 仍然是1, obj并未被修改为100.
{% endhighlight %}

然而，由于JavaScript中的对象，是可变的(mutable），形参实参指向同一个对象，当修改对象中的属性时，实参也会受到影响。

{% highlight javascript %}
var obj = {x : 1};
function foo(o) {
    o.x = 3;
}
foo(obj);
console.log(obj.x); // 3, 被修改了!
{% endhighlight %}


由于JavaScript中的基本类型都是不可变的(immutable)，对基本类型，按共享传递与按值传递(call by value)无异，也可理解为JavaScript基本类型是按值传递的。

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



### 参考
* 参考文献：[Wiki-Evaluation strategy](http://en.wikipedia.org/wiki/Evaluation_strategy#Call_by_reference)
