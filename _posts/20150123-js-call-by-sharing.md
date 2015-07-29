---
layout: post
title: JS是按值传递还是按引用传递?
description: ""
modified: 2015-01-23
category: js
tags: [js,前端]
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
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->
最近遇到个有趣的问题：“JS中的值是按值传递，还是按引用传递呢？”

在分析这个问题之前，我们需了解什么是按值传递(call by value)，什么是按引用传递(call by reference)。在计算机科学里，这个部分叫求值策略(Evaluation Strategy)。它决定变量之间、函数调用时实参和形参之间值是如何传递的。

### 按值传递 VS. 按引用传递

按值传递(call by value)是最常用的求值策略：函数的形参是被调用时所传实参的*副本*。修改形参的值并不会影响实参。

按引用传递(call by reference)时，函数的形参接收实参的*隐式引用*，而不再是副本。这意味着函数形参的值如果被修改，实参也会被修改。同时两者指向相同的值。

按引用传递会使函数调用的追踪更加困难，有时也会引起一些微妙的BUG。

按值传递由于每次都需要克隆副本，对一些复杂类型，性能较低。两种传值方式都有各自的问题。

我们先看一个C的例子来了解按值和引用传递的区别：

{% highlight c %}
void Modify(int p, int * q)
{
    p = 27; // 按值传递 - p是实参a的副本, 只有p被修改
    *q = 27; // q是b的引用，q和b都被修改
}
int main()
{
    int a = 1;
    int b = 1;
    Modify(a, &b);   // a 按值传递, b 按引用传递,
                     // a 未变化, b 改变了
    return(0);
}

{% endhighlight %}

这里我们可以看到：

* a => p按值传递时，修改形参p的值并不影响实参a，p只是a的副本。
* b => q是按引用传递，修改形参q的值时也影响到了实参b的值。

### 探究JS值的传递方式

JS的基本类型，是按值传递的。

{% highlight javascript %}
var a = 1;
function foo(x) {
    x = 2;
}
foo(a);
console.log(a); // 仍为1, 未受x = 2赋值所影响
{% endhighlight %}

再来看对象：

{% highlight javascript %}
var obj = {x : 1};
function foo(o) {
    o.x = 3;
}
foo(obj);
console.log(obj.x); // 3, 被修改了!
{% endhighlight %}

说明o和obj是同一个对象，o不是obj的副本。所以不是按值传递。
但这样是否说明JS的对象是按引用传递的呢？我们再看下面的例子：

{% highlight javascript %}
var obj = {x : 1};
function foo(o) {
    o = 100;
}
foo(obj);
console.log(obj.x); // 仍然是1, obj并未被修改为100.
{% endhighlight %}

如果是按引用传递，修改形参o的*值*，应该影响到实参才对。但这里修改o的值并未影响obj。
因此JS中的对象并不是按引用传递。那么究竟对象的值在JS中如何传递的呢？

### 按共享传递 call by sharing

准确的说，JS中的基本类型按值传递，对象类型*按共享传递*的(call by sharing，也叫按对象传递、按对象共享传递)。最早由[Barbara Liskov](http://en.wikipedia.org/wiki/Barbara_Liskov). 在1974年的GLU语言中提出。该求值策略被用于Python、Java、Ruby、JS等多种语言。

该策略的重点是：调用函数传参时，函数接受对象实参*引用的副本*(既不是按值传递的对象副本，也不是按引用传递的隐式引用)。 它和按引用传递的不同在于：在共享传递中对函数形参的赋值，不会影响实参的值。如下面例子中，不可以通过修改形参o的值，来修改obj的值。

{% highlight javascript %}
var obj = {x : 1};
function foo(o) {
    o = 100;
}
foo(obj);
console.log(obj.x); // 仍然是1, obj并未被修改为100.
{% endhighlight %}

然而，虽然引用是副本，引用的对象是相同的。它们共享相同的对象，所以修改形参对象的属性值，也会影响到实参的属性值。

{% highlight javascript %}
var obj = {x : 1};
function foo(o) {
    o.x = 3;
}
foo(obj);
console.log(obj.x); // 3, 被修改了!
{% endhighlight %}


对于对象类型，由于对象是可变(mutable)的，修改对象本身会影响到共享这个对象的引用和引用副本。而对于基本类型，由于它们都是不可变的(immutable)，按共享传递与按值传递(call by value)没有任何区别，所以说JS基本类型既符合按值传递，也符合按共享传递。


var a = 1; // 1是number类型，不可变
var b = a;
b = 6;

据按共享传递的求值策略，a和b是两个不同的引用(b是a的引用副本)，但引用相同的值。由于这里的基本类型数字1不可变，所以这里说按值传递、按共享传递没有任何区别。


### 基本类型的不可变(immutable)性质

基本类型是不可变的(immutable)，只有对象是可变的(mutable).
例如数字值100, 布尔值true, false，修改这些值(例如把1变成3, 把true变成100)并没有什么意义。比较容易误解的，是JS中的string。有时我们会尝试“改变”字符串的内容，但在JS中，任何看似对string值的"修改"操作，实际都是创建新的string值。

{% highlight javascript %}
var str = "abc";
str[0]; // "a"
str[0] = "d";
str; // 仍然是"abc";赋值是无效的。没有任何办法修改字符串的内容
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

### 术语的不同版本

需要注意的是，求值策略中的“引用”和求值策略本身都是抽象概念，这里的引用和语言具体的引用(例如C++的&a, C#的ref参数)可以不同，请不要混淆。

由于JS在传递对象类型的值时，是按值传递引用的副本，参考Dmitry的博文([链接](http://dmitrysoshnikov.com/ecmascript/chapter-8-evaluation-strategy/#terminology-versions))目前，对JS的求值策略有两种解释：

* JS采取的都是”按值传递”的求值策略, 其中对象类型较为特殊，实际为按值传递了引用(即传递引用的副本，而不是按引用传递引用)。从这个角度，说对象也是按值传递也是有道理的。(虽然笔者不是十分赞同).
* 引入“按共享传递”的求值策略，它让我们精确的区分按共享传递与经典的按值传递、按引用传递的不同。在这种情形下，可以按传参类型区分：“基本类型按值传递、而对象按共享传递。”（笔者更倾向的描述方式）

### 结论

虽然关于JS的求值策略有诸多争议和不同版本，博主比较倾向的结论是：

"JS中基本类型是按值传递的，对象类型是按共享传递的。"

语言抽象概念并非博主创造或臆造，文中所涉理论理论均有参考，详见下面之参考文献。

另感谢博客园园友@京山游侠 @greatim的精彩讨论和补充。

### 参考文献
* [Wikipedia.org-Evaluation strategy](http://en.wikipedia.org/wiki/Evaluation_strategy#Call_by_reference)
* [http://dmitrysoshnikov.com/-Evaluation Strategy](http://dmitrysoshnikov.com/ecmascript/chapter-8-evaluation-strategy/)

