---
layout: post
title: ES6笔记之参数默认值(译)
description: "在这个简短的笔记中我们聊一聊ES6的又一特性：带默认值的函数参数。正如我们即将看到的，它在某些CASE下会有些微妙的不同。"
modified: 2014-06-20
category: articles
tags: [es6,js,前端]
imagefeature: cover3.jpg
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

* 原文链接:[http://dmitrysoshnikov.com/](http://dmitrysoshnikov.com/ecmascript/es6-notes-default-values-of-parameters/)
* 原文作者:Dmitry Soshnikov
* 译者做了少量补充。<span class="trans">这样的的文字</span>是译者加的，可以选择忽略。

在这个简短的笔记中我们聊一聊ES6的又一特性：带默认值的函数参数。正如我们即将看到的，有些较为微妙的CASE。

### ES5及以下手动处理默认值

在ES6默认值特性出现前，手动处理默认值有几种方式：

{% highlight javascript %}
function log(message, level) {
  level = level || 'warning';
  console.log(level, ': ', message);
}

log('low memory'); // warning: low memory
log('out of memory', 'error'); // error: out of memory
{% endhighlight %}

为了处理参数未传递的情况，我们常看到typeof检测:

{% highlight javascript %}
if (typeof level == 'undefined') {
  level = 'warning';
}
{% endhighlight %}

有时也可以检查arguments.length

{% highlight javascript %}
if (arguments.length == 1) {
  level = 'warning';
}
{% endhighlight %}

这些方法都可以很好的工作，但都过于手动且缺少抽象。ES6规范了直接在函数头定义参数默认值的句法结构。

### ES6默认值：基本例子

默认参数特性在很多语言中普遍存在，其基本形式可能大多数开发者都比较熟悉：

{% highlight javascript %}
function log(message, level = 'warning') {
  console.log(level, ': ', message);
}

log('low memory'); // warning: low memory
log('out of memory', 'error'); // error: out of memory
{% endhighlight %}

参数默认值使用方便且毫无违和感。接下来让我们深入细节实现，扫除默认参数所带来的一些困惑。

### 实现细节

以下为一些函数默认参数的ES6实现细节。

#### 执行时求值

相对其它一些语言（如Python）在定义时一次性对默认值求值，ECMAScript在每次函数调用的执行期才会计算默认值。这种设计是为了避免在复杂对象作为默认值使用时引发一些困惑。接下来请看下面**Python**的例子：

{% highlight python %}
def foo(x = []):
  x.append(1)
  return x

# 我们可以看到默认值在函数定义时只创建了一次
# 并且存于函数对象的属性中
print(foo.__defaults__) # ([],)

foo() # [1]
foo() # [1, 1]
foo() # [1, 1, 1]

print(foo.__defaults__) # ([1, 1, 1],)
{% endhighlight %}

为了避免这种现象，Python开发者通常把默认值定义为`None`，然后为这个值做显式检查：

{% highlight python %}
def foo(x = None):
  if x is None:
    x = []
  x.append(1)
  print(x)

print(foo.__defaults__) # (None,)

foo() # [1]
foo() # [1]
foo() # [1]

print(foo.__defaults__) # ([None],)
{% endhighlight %}

就目前，很好很直观。接下来你会发现，若不了解默认值的工作方式，ES5语义上会产生一些困惑。

### 外层作用域的遮蔽

来看下面的例子:

{% highlight javascript %}
var x = 1;

function foo(x, y = x) {
  console.log(y);
}

foo(2); // 2, 不是 1!
{% endhighlight %}

来上例的`y`输出结果看起来像是`1`，但实际上是`2`,不是`1`。原因是参数中的`x`与全局的`x`不同。由于默认值在函数调用时求值，所以当赋值`=x`时，`x`已经在内部作用域决定了，引用的是参数`x`本身。也就是说，参数`x`被全局的同名变量遮蔽，所以每次默认值中访问`x`时，实际访问到的是参数中的`x`。

### 参数的TDZ（Temporal Dead Zone，暂存死区）

ES6提到所谓的TDZ（暂存死区），意指这样的程序区域：初始化前的变量或参数不能被访问。

考虑到对于参数，不能将自己作为默认值：

{% highlight javascript %}
var x = 1;

function foo(x = x) { // throws!
  ...
}
{% endhighlight %}

赋值`=x`正如我们上面提到的那样，`x`会被解释为参数级作用域中的`x`，而全局的`x`会被遮蔽。但是，`x`位于TDZ，在初始化前不能被访问。因此，它不能自己初始化自己。

注意，上面之前的例子中的`y`却是合法的，因为x在之前已经初始化了（隐式的默认值`undefined`）。所以我们再看下：

{% highlight javascript %}
function foo(x, y = x) { // OK
  ...
}
{% endhighlight %}

这样不会出问题，因为在ECMAScript中，参数的解析顺序是从左到右，所以在对`y`求值时`x`已经可用。

我们提到过参数是和"内部作用域"相关的，在ES5中我们可假设这个"内部作用域"就是函数作用域。但更复杂的情况：可能是函数的作用域，或者，一个只为存储参数绑定的立即作用域。让我们继续探索。

### 有条件的参数立即作用域

事实上，对于一些参数（至少一个）有默认值的情况，ES6会定义一个立即作用域来存储这些参数，并且这个作用域并不会与函数作用域共享。在这方面这是ES6与ES5的一个主要区别。有点晕？不要紧，看下例子你就懂。

{% highlight javascript %}
var x = 1;

function foo(x, y = function() { x = 2; }) {
  var x = 3;
  y(); // 局部变量`x`会被改写乎?
  console.log(x); // no, 依然是3, 不是2
}

foo();

// 而且外层的`x`也未变化
console.log(x); // 1
{% endhighlight %}

在这个例子中，我们有三个作用域：全局环境、参数环境、函数环境：

{% highlight javascript %}
:  {x: 3} // 函数
-> {x: undefined, y: function() { x = 2; }} // 参数
-> {x: 1} // 全局
{% endhighlight %}

现在我们应该清楚了，当作为参数的函数对象`y`执行时，它内部的`x`会被就近解析（也就是上面说的参数环境），函数作用域对其并不可见。

### 编译到ES5

如果我们想把ES6代码编译到ES5，并且需要搞清楚这个立即作用域究竟是什么样的，我们可以得到像这样的东东：

{% highlight javascript %}
// ES6
function foo(x, y = function() { x = 2; }) {
  var x = 3;
  y(); // 局部变量`x`会被改写吗?
  console.log(x); // no, 依然是3, 不是2
}

// 编译到ES5
function foo(x, y) {
  // 设置默认参数
  if (typeof y == 'undefined') {
    y = function() { x = 2; }; // 现在弄清楚了，将会更新参数中的`x`
  }

  return function() {
    var x = 3; // 这里的`x`是函数作用域的
    y();
    console.log(x);
  }.apply(this, arguments);
}
{% endhighlight %}

### 参数级作用域的存在原因

设计参数级作用域的目的究竟是什么？为什么不能像ES5那样可以访问到函数作用域中的变量？原因：参数默认值是函数时，其函数体内的同名变量不应该影响被捕获闭包中的同名绑定。

例：

{% highlight javascript %}
var x = 1;

function foo(y = function() { return x; }) { // 捕获 `x`
  var x = 2;
  return y();
}

foo(); // 正确的应该是 1, 不是 2
{% endhighlight %}

如果我们在函数体内创建函数`y`，它内部的`return x`中的`x`会捕获函数作用域下的`x`，也就是`2`。但是，很明显，参数`y`函数中的`x`应该捕获到全局的`x`，也就是`1`（除非被同名参数遮蔽）。

同时，这里不能在外部作用域下创建函数，因为这样就意味着无法访问这个函数的参数了，所以我们应该这样做：

{% highlight javascript %}
var x = 1;

function foo(y, z = function() { return x + y; }) { // 现在全局`x` 和参数`y`均在参数`z`函数中可见
  var x = 3;
  return z();
}

foo(1); // 2, 不是 4
{% endhighlight %}

### 若不创建参数级作用域

上面的描述的默认值工作方式，在语义上与最开始我们手动实现默认值完全不同，例：

{% highlight javascript %}
var x = 1;

function foo(x, y) {
  if (typeof y == 'undefined') {
    y = function() { x = 2; };
  }
  var x = 3;
  y(); // 局部变量`x`会被改写么?
  console.log(x); // 这次被改写了！输出2
}

foo();

// 而全局的`x`仍然未变化
console.log(x); // 1
{% endhighlight %}

这个事实很有趣：如果函数无默认值，它不会创建这个立即作用域，并且与函数环境共享参数绑定，也就是像ES5那样处理。<span class="trans">这也是为什么说是『有条件的参数立即作用域』</span>

为什么会这样？为什么不每次创建参数级作用域？只是为了优化？非也非也。这么做的原因其实是为了向后兼容ES5：上面手动模拟默认值机制的代码应该更新函数体的`x`（也就是参数`x`，<font class="trans">在相同作用域下实际是同一个变量被重复声明，一次是参数定义，一次是局部变量`x`</font>）。

另外，需要注意到只有变量和函数允许重复声明，而用let/const重复声明参数是不允许的：

{% highlight javascript %}
function foo(x = 5) {
  let x = 1; // error
  const x = 2; // error
}
{% endhighlight %}

## undefined的检测

另外一个有趣的事情是：是否默认值会被应用将取决于初始值<span class="trans">也就是传参</span>是否为`undefined`（在进入上下文时被赋值）。例：

{% highlight javascript %}
function foo(x, y = 2) {
  console.log(x, y);
}

foo(); // undefined, 2
foo(1); // 1, 2

foo(undefined, undefined); // undefined, 2
foo(1, undefined); // 1, 2
{% endhighlight %}

通常情况下在一些编程语言中，带默认值参数会在必选参数的后面，但是，在JavaScript中允许下面的构造：

{% highlight javascript %}
function foo(x = 2, y) {
  console.log(x, y);
}

foo(1); // 1, undefined
foo(undefined, 1); // 2, 1
{% endhighlight %}

## 解构组件的默认值

另一个默认值涉及到的地方是解构组件的默认值。解构赋值的讨论不在本文中详述，但我们可以看一些简单的例子。对于在函数参数中使用解构的处理，与上面描述过的默认值处理相同：也就是必要时会创建两个作用域:

{% highlight javascript %}
function foo({x, y = 5}) {
  console.log(x, y);
}

foo({}); // undefined, 5
foo({x: 1}); // 1, 5
foo({x: 1, y: 2}); // 1, 2
{% endhighlight %}

当然，解构的默认值更加通用，不只在函数参数默认值中可用：

{% highlight javascript %}
var {x, y = 5} = {x: 1};
console.log(x, y); // 1, 5
{% endhighlight %}

## 结论

希望这个简短的记录能帮助大家理解ES6中的默认值特性的细节。需要注意的是，由于这个"第二作用域"是最近才加入到规范草稿中的，因此截至本文撰写时（2014年8月21日），没有任何引擎正确的实现了ES6默认值（它们全部只创建了一个作用域，也就是函数作用域）。默认值显然是一个有用的特性，它使得我们的代码更加优雅和明确。

#### 作者&鸣谢

* 作者：[Dmitry Soshnikov](http://dmitrysoshnikov.com)
* 发布时间：2014年8月21日
* 译者：[Bosn](http://bosn.me)
* 鸣谢：感谢@紫云妃 对术语解释上的指导和帮助
