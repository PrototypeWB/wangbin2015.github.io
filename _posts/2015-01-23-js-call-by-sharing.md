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
最近遇到个有趣的问题：“JavaScript中的值是按值传递，还是按引用传递呢？”

在分析这个问题之前，我们需了解什么是按值传递(call by value)，什么是按引用传递(call by reference)。在计算机科学里，这个部分叫求值策略(Evaluation Strategy)。它影响着在赋值、函数调用传参等情况时，值如何在变量与变量、变量作为实参与函数形参之间传递。

### 按值传递 VS. 按引用传递

按值传递是最常用的求值策略。在按值传递时，函数的形参是被调用时所传实参的*副本*。修改形参的值并不会影响实参。

在按引用传递时，函数的形参接受实参的*隐式引用*，而不再是副本。这意味着函数形参的值如果被修改，实参也会被修改。

按引用传递会使函数调用的追踪更加困难，有时也会引起一些微妙的BUG。

我们看一个C的例子来了解按值传递、按引用传递的差别：

{% highlight c %}
void Modify(int p, int * q)
{
    p = 27; // 按值传递 - 只有局部变量p被修改
    *q = 27; // 通过&b按引用传递给q, q和b一起被修改
}
int main()
{
    int a = 1;
    int b = 1;

    Modify(a, &b);   // a 是按值传递的, b是按引用传递的,
                        // c 是一个指针，该指针按值传递
    // b 改变了， a 未变化
    return(0);
}

{% endhighlight %}

这里我们可以看到：

* a => p按值传递时，修改形参p的值(p = 27)并不影响实参(a)。p只是a的副本。
* b => q是按引用传递，修改形参q的值时也影响到了实参b的值。

### 探究JS值的传递方式

JavaScript的基本类型，是按值传递的。

{% highlight javascript %}
var a = 1;
function foo(x) {
    x = 2;
}
foo(a);
console.log(a); // 仍为1, 未受x = 2赋值所影响
{% endhighlight %}

JavaScript的对象类型，看起来像是按引用传递。但实际不是。

{% highlight javascript %}
var obj = {x : 1};
function foo(o) {
    o.x = 3;
}
foo(obj);
console.log(obj.x); // 3, 被修改了!说明o和obj是同一个对象，o不是obj的副本。不是按值传递。
{% endhighlight %}

如果是按引用传递，修改形参o的*值*，应该影响到实参才对。

{% highlight javascript %}
var obj = {x : 1};
function foo(o) {
    o = 100;
}
foo(obj);
console.log(obj.x); // 仍然是1, obj并未被修改为100.
{% endhighlight %}

因此JavaScript中的对象并不是按引用传递。那么JavaScript语言中的值究竟如何传递的呢？

### 按共享传递 call by sharing

准确的说，JS中的基本类型按值传递，对象类型*按共享传递*的(call by sharing，也叫按对象传递、按对象共享传递)。最早由[Barbara Liskov](http://en.wikipedia.org/wiki/Barbara_Liskov). 在1974年的GLU语言中提出。该求值策略被用于Python、Java、Ruby、JavaScript等多种语言。

该策略的重点是：调用函数传参时，函数接受对象实参*引用的副本*(而不是对象副本)。 它和按引用传递的不同在于：在共享传递中对函数形参的赋值，不会影响实参的值。如下面例子中，不可以通过修改形参o的值，来修改obj的值。

{% highlight javascript %}
var obj = {x : 1};
function foo(o) {
    o = 100;
}
foo(obj);
console.log(obj.x); // 仍然是1, obj并未被修改为100.
{% endhighlight %}

然而实参形参引用相同的对象（虽然引用本身是不同的两个引用，后者是前者的副本），对象在JS中是可变的(mutable），修改对象中的属性，实参也会受到影响。

{% highlight javascript %}
var obj = {x : 1};
function foo(o) {
    o.x = 3;
}
foo(obj);
console.log(obj.x); // 3, 被修改了!
{% endhighlight %}


而对于基本类型，由于它们都是不可变的(immutable)，按共享传递与按值传递(call by value)没有区别，也可理解为JS的基本类型是按值传递的。

### 基本类型的不可变(immutable)性质

基本类型是不可变的(immutable)，只有对象是可变的(mutable).
例如数字值100, 布尔值true, false，修改这些值(例如把1变成3, 把true变成100)并没有什么意义。比较容易误解的，是JavaScript中的string。有时我们会尝试“改变”字符串的内容，但在JavaScript中，任何看似对string值的"修改"操作，实际都是创建新的string值。

{% highlight javascript %}
var str = "abc";
str[0]; // "a"
str[0] = "d";
str; // "abc";
{% endhighlight %}

而对象就不一样了，对象是可变的。

{% highlight javascript %}
var obj = {x : 1};
obj.x = 100;
var o = obj;
o.x = 1;
obj.x; // 1, 被修改
o = true;
obj.x; // 1, 不会因o = true改变
{% endhighlight %}

这里定义变量obj，值是object，然后设置obj.x属性的值为100。而后定义另一个变量o，值仍然是这个object对象，此时obj和o两个变量的值指向同一个对象（共享同一个对象的引用）。所以修改对象的内容，对obj和o都有影响。但对象并非按引用传递，通过o = true修改了o的值，不会影响obj。

所以JS既不是按值传递，也不是按引用传递。而是按共享传递。


### 参考文献
* [Wikipedia.org-Evaluation strategy](http://en.wikipedia.org/wiki/Evaluation_strategy#Call_by_reference)
* [http://dmitrysoshnikov.com/-Evaluation Strategy](http://dmitrysoshnikov.com/ecmascript/chapter-8-evaluation-strategy/)

