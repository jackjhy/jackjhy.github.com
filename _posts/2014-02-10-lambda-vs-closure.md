---
layout: post
title: "Lambad表达式和闭包的有什么不同"
description: ""
category: ""
tags: [Lambda,clusure]
---
{% include JB/setup %}

Lambda的前世今生
===
Lambda表达式源于Lambda演算，Lambda演算的出现时间要远远早于Closure，在第一台现代计算机在美国诞生之前，Lambda演算就已经由英国科学家邱奇，此人是著名的图灵的老师。Lambda演算通过一系列的约束（其实就是3条限定）定义了什么是可计算函数。

先来说下Lambda演算的表示形式，举个例子，函数 f(x)=x+1的Lambda表达形式Lambda x.x+1,其中lambda x表示x是入参,x+1表示对参数做怎样的变换。f(2) 就可以表示成(lambda x.x+1) 2。注意这里，一般f(2)这种形式，在编程中我们称之为调用，我理解，这也就是Lambda演算中间所说的演算，或者是计算。这种形式，需要注意，后面会提到。另外呢，参数，也可以是Lambda表达式（注意这里就引入的名词，Lambda表达式）。复杂一点的例子呢，以函数f作为lambda表达式入参：(lambda f. f 3) (lambda x.x+1)。首先，这里两个括弧的形式，是一种实际的演算（以现代编程的语言来说，就是调用），第一个括弧定义了函数，第二个括弧中定义了入参。所以应用进去就等同于（Lambda x.x+1）3,继续应用， 3+1。大概来说，这些就是lambda演算的表达形式。

Lambda是左结合的。

Lambda演算的限制，就抄wiki了，实际3条,用bnf形式表达：

* \<expression\>	:=    \<name\> | \<function\> | \<application\>
* \<function\>	:=    λ \<name\>.\<expression\>
* \<application\>	:=    \<expression\>\<expression\>


编程语言中的Lambda
===
在现代编程语言中，支持Lambda实际上已经明确化到是否支持匿名函数，并且函数是否可以作为参数使用。并且，Lambda演算中，Lambda 表达式（又叫项 term）是单参数的，在编程中并没有这个限制，这是由于柯里化（Currying），也就是Haskell语言的作者明确定义的。可以把N元参数的函数变形为一系列一元函数链，实际上就是一条Lambda演算。至于为什么是匿名函数，有名称的函数就不可以？不尽然，scheme中，匿名函数和命名函数实际是同一种实现。

闭包
===
至于闭包，明确的定义就是依赖环境变量的函数，至于java是否有闭包，这取决于到底如何定义函数，java本身是没有定义函数这个概念的，叫方法。如果把java的method等同于函数的化，那么java是没有闭包的。不然，java其实也是支持闭包的。彻底就是概念上的玩意儿，没什么意思。

如上所说，Lambda表达式和Closure在定义上可以没有任何联系，而且两个人也不是同一个划分维度上的东西。但是，在语言的实现上，一般支持函数作为参数的，都支持Closure。这也是为什么Mattin Fowler说["which is why closure is often used as an alternative term for lambda"](http://martinfowler.com/bliki/Lambda.html)

正如同函数式编程，现代编程语言中涉及到Lambda表达式，Closure等，其实更多的是降低了程序员敲键盘的次数。


