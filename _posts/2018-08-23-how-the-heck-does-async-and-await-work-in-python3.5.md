---
layout: post
tille: How the heck does async/await work in Python 3.5? 
description: Try to understand async/await
categories: Python
tags: 
    - python 
    - 翻译
---
**To be continued.**

**[原文链接](https://snarky.ca/how-the-heck-does-async-await-work-in-python-3-5/)**

## async/await在Python3.5中到底是怎么工作的？

作为一个Python的核心开发者，这让我会想去理解这种语言到底是怎么工作的。我意识到总会有那些我不知道每一个细节的模糊角落出现，但是为了有助于问题的解决以及Python的设计，我感觉我应该尝试理解它的核心语义以及这些东西在底层到底是如何工作的？

但是直到最近，我还是不理解`async/await`在`Python3.5`中是怎么工作的。我知道在`Python3.3`中的`yield from`和`Python3.4`中的`asyncio`引出的这种新语法。但是由于我没有做很多网络相关的工作--虽然`asyncio`不是局限于这些工作，但它的确是专注于这些的--这就让我没有真正关注`async/await`。我的意思是我知道：

```python
yield from iterator
```

是（实际上）等效于：

```python
for x in iterator:
    yield x
```

我还知道`asyncio`是一个能够用来异步编程的事件循环框架，还知道这些名词自身是什么意思。但是我从来没有深入到`async/await`语法中去了解这些都是怎么结合到一起的。我觉得我并不了解`Python`中的异步编程，这让我有些苦恼。所以我决定慢慢来，去尝试弄明白这些东西到底是怎么工作的。并且由于我从不同的人那里听到过他们也不太了解这个新的异步编程的世界是怎么工作的，我决定写这篇短文。（是的，这篇post用了太长时间，也写得太长了以至于我的妻子把它标记为essay）。

现在由于我想对这种语法是怎么工作的有一个到位的理解，所以这篇文章有很多`CPython`是如何工作的底层细节。如果细节比你需要的多，或者由于我不想把这篇文章编程一本书而没有介绍`CPython`内部每一个细微差别导致你并不完全理解这些细节，这都是可以的。（举个例子，如果你不明白`code`对象有`flags`，更不用说连`code`对象都不知道，你不需要关心从这篇文章得到的相关`code`对象的东西。）我在每一个版块的尾部尝试提供了一个很便于访问到的总结，所以如果你发现你并不想了解那么多细节的话，你可以跳过它们。

### 协程在Python中的历史

根据[Wikipedia][]，[协程][]是和子例程类似的面向非抢占式多任务的计算机程序组件，它允许多个进入点在一定位置暂停或继续执行。这是一种相当学术的说法，“协程是可以暂停执行的函数”。 如果你对自己解释说，“这听起来像生成器”，你就是对的。

回到`Python2.2`， 生成器（也被叫做生成迭代器，因为生成器实现了迭代器协议）是由[PEP 255][]第一次引进的。主要受到[Icon programming language][]的影响，生成器允许一种非常简单的，在计算下一个值时不浪费内存的创建迭代器的方法（你也可以实现一个实现了`__iter__()`和`__next__()`方法并且不保存迭代器每一个值得类，但这需要很多工作。）举个例子，如果你想创建你自己的range()版本，你可以通过创建一个整型列表以热切的方式完成：

```python
def eager_range(up_to):
    """创建一个整型列表，从0到up_to, 不包括up_to"""
    sequence = []
    index = 0
    while index < up_to:
        sequence.append(index)
        index += 1
    return sequence
```
这种实现的问题是，如果你想要一个很大的序列，比如从`0`到`1000000`，你需要常见一个足够长的能容纳`100000`个整数的列表。但是当生成器被引入`Python`之后，你突然能够轻松的创建一个不需要提前生成整个序列的迭代器。事实上，你需要的只是一次有给一个整型的足够内存就行了。

```python
def lazy_range(up_to):
    """返回0到up_to不包括上界的整型序列"""
    index = 0
    while index < up_to:
        yield index
        index += 1
```

不管函数在做什么，当它遇到一个yield表达式就暂停————虽然它在Python2.5之前还是一个语句————然后能够在接下来继续，这对于减少内存使用以及允许无穷序列等等都是非常有用的。

但是你可能也已经发现了，生成器就是迭代器的样子。既然现在有一种明显更棒的构建迭代的方法（这表现在当你在一个生成器对象上定义一个__iter__()方法的时候），但是人们知道如果我们把生成器中“暂停”的部分去掉，然后添加一种“把东西传入”的方法，Python就猛然有了协程的概念（但是直到我在其他地方说明了，把这些就当做Python里的概念；我们会稍后讨论Python中真正的协程。）然后生成器中的sending方法是在Python2.5加入的，感谢[PEP342][]。除此之外，[PEP342][]给生成器引入了send()方法。这不仅允许我们暂停生成器， 还允许我们把值传回到生成器暂停的地方。继续深入我们的range()例子，你能够使得序列前跳或者后跳一定量：

```python
def jumping_range(up_to):
    """Generator for the sequence of integers from 0 to up_to, exclusive.

    sending a value into the generator will shift the sequence by that amount.
    """

    index = 0
    while index < up_to:
        jump = yield index
        if jump is None:
            jump = 1
        index += jump

if __name__ == "__main__":
    iterator = jumping_range(5)
    print(next(iterator))
    print(iterator.send(2))
    print(next(iterator))
    print(iterator.send(-1))
    for x in iterator:
        print(x)
```

生成器直到Python3.3，当[PEP380][]添加yield from 的时候才再次被更改。严格的说，这个特性通过将从迭代器(刚巧生成器也是）产出值变得更简单来允许你以一种简单的方式重构生成器。

```python
def lazy_range(up_to):
    """Generator to return the sequence of integers from 0 to up_to, exclusive."""
    index = 0
    def gratuitous_refactor():
        nonlocal index
        while index < up_to:
            yield index
            index += 1
    yield from gratuitous_refactor()
```

重构更容易了之后，yield from还能够让你将生成器链到一起，这样值就可以在调用栈中上下浮动，而无需代码做任何特殊的事情。

```python
def bottom():
    # Returning the yield lets the value that goes up the call stack to come right back
    # down.
    return (yield 42)

def middle():
    return (yield from bottom())

def top():
    return (yield from middle())

# Get the generator.
gen = top()
value = next(gen)
print(value)  # Prints '42'.
try:
    value = gen.send(value * 2)
except StopIteration as exc:
    value = exc.value
print(value)  # Prints '84'.
```
#### 总结：
Python2.2中的生成器可以让执行的代码暂停。当Python2.5中引进了回传值到暂停的生成器中的功能，协程的概念在Python中就变得有可能了。然后Python3.3中新加入的yield from让重构生成器以及链接生成器都变得简单了。


### 时间循环是什么？

如果你开始关心`async`/`await`，那么理解时间循环是什么以及它们如何让异步编程变得有可能就很重要了。如果你曾经做过GUI编程————包括网页前后端的工作————那么你就已经和时间循环打过交道了。但是鉴于异步编程的概念作为Python一种新的语言结构，你如果暂时不明白时间循环是什么也是可以的。

回到Wikipedia， 一个[时间循环][]是“这是一种编程结构，用来等待或者分发程序中的时间或者消息”。基本上一个时间循环允许你执行，“当A发生时，执行B”。解释这一点的最简单的例子可能是每个浏览器中的JavaScript时间循环。每当你单击某个内容（“当A发生时”），单击事件就被传递给JavaScript时间循环，它检查是否有任何onclick回调被注册来处理这个单击事件（“do B”）。如果有任何回调被注册，那么回调就会被调用，并且附带单击的细节。事件循环被认为是一个循环是因为它不断的收集时间并对其进行循环，以找到如何处理事件的方法。

在Python里，`asyncio`被添加到标准库中来提供事件循环。在使用`asyncio`进行网络编程的时候需要注意，在这种情况下，事件循环把“when A happens”当做当一个套接字的I/O已经准备好读或者写的时候（通过`selectors`模块）。除了GUI以及I/O，事件循环还通常用来执行另外一个线程或者子程序中的代码以及将事件循环用作调度器（比如[cooperative multiasking][]）。如果你恰好了解Python的GIL，那么在释放GIL是可能和有用的情况下，事件循环是很有用的。

#### 总结
事件循环提供了一种循环使得你可以，“当A发生时，执行B”。基本上，事件循环就是监视一些事件的发生，然后当事件循环关注的事件发生了之后，就调用关注时间发生的代码。在Python3.4中，Python从标准库中以`asyncio`的形式得到一个时间循环。

### `async`和`await`是怎样工作的？
#### 在Python3.4中工作的方式
在Python3.3的生成器到以`asyncio`的形式的事件循环之间，Python3.4用[并发编程][]的形式就足以支持异步编程。异步编程其实就是我们提前不知道执行顺序的编程（这也是异步而不是同步的原因）。并发编程是我们编写执行时与其他部分独立的代码，即使是在一个线程中执行的（并发不是并行）。举个例子，下面是两个Python3.4中数秒的异步，并发的函数调用。


```python
import asyncio

# Borrowed from http://curio.readthedocs.org/en/latest/tutorial.html.
@asyncio.coroutine
def countdown(number, n):
    while n > 0:
        print('T-minus', n, '({})'.format(number))
        yield from asyncio.sleep(1)
        n -= 1

loop = asyncio.get_event_loop()
tasks = [
    asyncio.ensure_future(countdown("A", 2)),
    asyncio.ensure_future(countdown("B", 3))]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```

在Python3.4中， asyncio.coroutine 装饰器用于将函数标记为充当协程的角色，该程序用于异步和它的事件循环。这为Python提供了它对协程的第一个具体定义：一个对象，它实现了添加到[PEP 342][]中的生成器的方法，并由`collections.abc.coroutine`抽象基类表示。这意味着，突然之间，所有的生成器都实现了协程接口，即使它们并不打算以那种方式使用。为了解决这个问题，异步要求将所有要用作协同程序的生成器都必须用`asyncio.coroutine`修饰。

有了这个协程的具体定义（与生成器提供的API相匹配），你就可以在任何[`aysncio.Future`对象][]使用yield from，将这个对象传递到事件循环中，在你等待事情发生的时候暂停协程执行（成为一个future对象是`asyncio`的实现细节，并不重要）。一旦future对象到达了事件循环，它就被监视直到future对象完成任何它需要完成的工作。一旦future完成了自己的工作，事件循环就会注意到，然后暂停等待future结果的协程就会再次启动，然后把它的结果用send()方法传递到协程中。


[Wikipedia]: https://www.wikipedia.org/ "Wikipedia"
