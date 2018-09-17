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
print(value)  # Prints '43'.
try:
    value = gen.send(value * 2)
except StopIteration as exc:
    value = exc.value
print(value)  # Prints '84'.
```
#### 总结：
Python2.2中的生成器可以让执行的代码暂停。当Python2.5中引进了回传值到暂停的生成器中的功能，协程的概念在Python中就变得有可能了。然后Python3.3中新加入的yield from让重构生成器以及链接生成器都变得简单了。


### 事件循环是什么？

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

在Python3.4中， `asyncio.coroutine` 装饰器用于将函数标记为协程，用于`asyncio`它的事件循环。这为Python提供了它对协程的第一个具体定义：一个对象，它实现了添加到[PEP 342][]中的生成器的方法，并由`collections.abc.coroutine`抽象基类表示。这意味着，突然之间，所有的生成器都实现了协程接口，即使它们并不是被当做协程使用的。为了解决这个问题，`asyncio`要求将所有要用作协程的生成器都必须用`asyncio.coroutine`修饰。

有了这个协程的具体定义（与生成器提供的API相匹配），你就可以在任何[`aysncio.Future`对象][]上使用`yield from`，将这个对象传递到事件循环中，在等待事情发生的时候暂停其执行（`future`对象是`asyncio`的实现细节，并不重要）。一旦`future`对象到达了事件循环，它就会被监视, 直到`future`对象完成任何它需要完成的工作。一旦`future`完成了自己的工作，事件循环就会监测到，然后处于暂停状态的等待`future`结果的协程就会再次启动，它的结果也被send()方法传递到协程中。

接着分析上面的那个例子。事件循环启动每一个`countdown()`协程调用，一直执行到其中一个的`yield from`和`asyncio.sleep()`函数处。到这里会产生一个`asyncio.Future`对象，并被传递到事件循环中，然后暂停协程的执行。在事件循环中，事件循环监测这个`future`对象直到sleep的这一秒结束（同时也检查它正在监测的其他事物，比如其他的协程）。一旦这一秒结束了，事件循环找到那个提供`future`对象的处于暂停状态的协程，然后把`future`对象的结果发送到这个协程，然后协程继续运行。这个过程一直持续到所有的`countdown()`协程借宿运行，并且事件循环没有监测对象的时候。稍后，我会给你展示一个完整的协程或者事件循环这些东西工作的例子, 但是首先我想解释一下`async`和`await`是怎么工作的。

#### Python3.5: 从`yield from`到`await`
在Python3.4中，出于异步编程的目的把一个函数标记为协程的过程想下面这样：

```python
# 在Python3.5中同样生效
@asyncio.coroutine
def py34_coro():
    yield from stuff()
```

在Python3.5中，加入了`types.coroutine`装饰器和`asyncio.coroutine`一样用来标记一个生成器为协程。你还可以使用`async def`来语法上定义一个函数为协程，虽然它不能包含任何形式的`yield`表达式；只有`return`和`await`被允许从协程中返回一个值。

```python
async def py35_coro():
    await stuff()
```

`async`和`types.coroutine`做的一个关键的事情就是缩减了协程的定义。它把协程从一个只是接口变成了一种确切的类型，把其他生成器和用来做协程的生成器之间的区别更严格了（`inspect.iscoroutine()`函数更严格了，它固定协程必须使用`async`）。

你可能还注意到了除了`async`，在Python3.5的例子中还引入了`await`表达式（只能在`async def`定义的函数中使用）。虽然`await`运行起来跟`yield from`很像，但是`await`表达式接收的对象确是不同的。协程可以在`await`表达式中使用，因为协程就是所有这些东西的基础。但是当你在一个对象上调用`await`时，这个对象技术上需要是一个`awaitable`对象：一个定义了`__await__()`方法并且返回一个迭代器，而不是返回协程本身的对象。协程被认为是`awaitable`对象（这也是为什么`collections.abc.Coroutine`继承自`collections.abc.Awaitable`）。这个定义遵循了Python底层上把大部分的语法定义都转换到一个方法调用上的传统，就像`a+b`在底层实际上是`a.__add__(b)`或者`b.__radd__(a)`。

那么在底层上`yield from`和`awati`有什么区别呢（也就是一个使用`types.coroutine`装饰器定义的生成器和一个用`async def`定义的生成器的区别）？让我们看一下上面的两个例子在Python3.5中的字节码，来了解一下本质区别。`py34_coro()的字节码是：

```python
>>> dis.dis(py34_coro)
  2           0 LOAD_GLOBAL              0 (stuff)
              3 CALL_FUNCTION            0 (0 positional, 0 keyword pair)
              6 GET_YIELD_FROM_ITER
              7 LOAD_CONST               0 (None)
             10 YIELD_FROM
             11 POP_TOP
             12 LOAD_CONST               0 (None)
             15 RETURN_VALUE
```

`py35_coro()`的字节码是：

```python
>>> dis.dis(py35_coro)
  1           0 LOAD_GLOBAL              0 (stuff)
              3 CALL_FUNCTION            0 (0 positional, 0 keyword pair)
              6 GET_AWAITABLE
              7 LOAD_CONST               0 (None)
             10 YIELD_FROM
             11 POP_TOP
             12 LOAD_CONST               0 (None)
             15 RETURN_VALUE
```

忽略那行由于`py34_coro()`拥有`asyncio.coroutine`装饰器造成的区别，两者唯一肉眼可见的区别就是`GET_YIELD_FROM_ITER`字节码和`GET_AWAITABLE`字节码的不同。两个函数都被正确的标记成协程，所以并没有什么区别。在使用`GET_YIELD_FROM_ITER`的情况下，它只是简单的检查一下它的参数是不是一个生成器或者协程，如果不是，它就会对它的参数调用`iter()`方法（只有当`yield from`的字节码在协程里使用的时候，该字节码才能接受一个协程对象，在这个例子中是正确的需要感谢`types.coroutine`装饰器标记了这个生成器就像在C语言层面上用`CO_ITERABLE_COROUTINE` 标记代码对象一样）。

但是，`GET_AWAITABLE`字节码做了一些不同的事情。虽然这个字节码接收一个跟`GET_YIELD_FROM_ITER`接收的一样的协程，但是它不会接收一个没有被标记为协程的生成器。除了协程，这个字节码还像我们前面讨论的那样接收一个`awaitable`对象。这就使得`yield from`表达式和`await`表达式都接收协程对象，同时不同的地方在于它们是否分别接收一个普通的生成器或者`awaitable`对象。

你可能想知道为什么基于`async`的协程和基于生成器的协程在各自的暂停表达式中会接收不同的参数？这样做的原因是Python尽最大的努力确保你不会搞砸了，并且意外的混合和匹配那些恰好拥有相同API的对象。鉴于生成器继承性的实现了协程的API，那么当你希望使用一个协程的时候，就很可能意外的使用了生成器。而且由于并不是所有的生成器都被用编写用在一个基于协程的控制流程中，你需要避免不正确的使用生成器。但是由于Python不是静态编译的，Python能给你的最大保障就是当你使用一个基于生成器的协程的时候执行运行时检查。这意味着当使用`types.coroutine`的时候，Python的编译器分辨不出这个生成器是要被用作协程或者只是用着生成器（记住，语法上写`types.coroutine'并不意味着有人已经提前做了`types = spam`的检查），因此，不同的操作码有不同的限制是由编译器根据它当时的情景给出的。

关于基于生成器的协同程序和基于`async`的协程之间的区别，我想说的一个非常关键的一点是，只有基于生成器的协程才能真正暂停执行并且强制传递一些东西到时间循环。你通常不会注意到这一重要的细节，因为你通常会调用事件循环特定的函数，比如`asyncio.sleep()`由于事件循环实现了它们自己的API，这些函数都必须考虑这个小细节。对于大多数的我们，我们会使用事件循环而不是编写它们，因此我们只编写`aync`协程而不需要真正的关心这个问题。但是如果你和我一样，并且正在好奇为什么你不能编写一些像`asyncio.sleep()`这种只使用`async`协程的函数，那么这将是一个非常好的时刻。

#### 总结
让我们简单总结一下上面的东西。使用`async def`定义一个方法使得这个函数成为协程。另一个制作协程的方式是用`types.coroutine`标记一个生成器——实际上这个标记其实是对代码对象做`CO_ITERABLE_COROUTINE`标记——或者是一个`collections.abc.Coroutine`的子类。你只能使用一个基于生成器的协程的时候才能暂停一个协程调用链。

一个`awaitable`对象要么是一个协程，要么是一个定义了`__await__()`的对象——实际上是`collections.abc.Awaitable`——它返回一个迭代器而不是协程。一个`await表达式基本上就是`yield from`，但是有只能和`awaitable`对象一起使用（纯粹的生成器不能再`await`表达式中使用）。一个`async`函数是一个协程，它要么含有`return`语句——包括Python中每一个函数末尾都有的隐式`return None`语句——要么还包括或者只有`await`表达式（`yield`表达式是不允许使用的）。这个对于`async`函数的限制是为了确保你不会意外的将基于生成器的协程和其他生成器混用，毕竟这两种类型的生成器的期望使用方式是相当不同的。

### 把`async`/`await`当做异步编程的一个API
我想指出的一个关键事情是直到我看了David Beazley's Python Brasil 2015年的幻灯片之前都没有深入思考过的事情。在那个讲座里，David指出了`async/await`实际上是异步编程的一个API（这也是他在twitter上对我重申的）。David说这个的意思是人们不应该把`async`/`await`当成和`asyncio`一样的东西，而是把`asyncio`想象成一个替异步编程实现`async`/`await`API的框架。

David相信`async`/`await`是一个异步编程的API以至于他创建了`curio`项目去实现他自己的事件循环。这让我更清楚的认识到，`async`/`await`允许Python为异步编程提供基础，但是并不将你绑定到特定的事件循环或者其他的底层细节当中（这与直接将事件循环集成到语言中的其他编程语言不通）。这使得`curio`这样的项目，不仅能够在较低的级别上以不同的方式运行（例如，`aasyncio`使用`future`对象来作为和事件循环交流的API，而`curio`使用元组），而且还可以具有不同的关注点和性能特种（例如，`asyncio`有一个完整的框架来实现传输和协议层，这使得它具有扩展性，而`curio`更简单，并且希望用户自己关心这类问题，但是也让它运行的更快了）。

基于Python中（短））的异步编程历史，人们可能认为`async`/`await` == `asyncio`就是可以理解的。我是说`asyncio`是使得在Python3.4中异步编程变得可能以及在Python3.5中添加`async`/`await`的动力因素。但是`async`/`await`的设计目的是要足够灵活到不再需要`asyncio`或者针对该框架扭曲任何关键决策。换句话说，`async`/`await`延续了Python的传统，即设计东西时尽可能灵活，同时又能实用的使用（和实现）。

## 一个例子

到了现在你脑子里可能充斥着各种名词和概念，使得明白所有这些东西时如何使用来给你提供异步编程变得有些困难。为了使得这些都更确切，这是一个完整的（如果设计的）异步编程例子，从事件循环和相关函数到用户代码的端到端示例。这个例子有代表独立火箭发射倒计时的协程，但是看起来确是同步倒计时的。这就是通过并发的异步编程；三个独立的协程会独立运行，而且这一切都在一个线程中完成。

```python
# -*- coding: utf-8 -*-
"""
一个完整的异步编程的例子
"""
import datetime
import heapq
import types
import time


class Task:
    """代表一个协程在重新运行之前应该等待多久

    实现的比较运算符是给heapq用的。不幸的是，由于当datetime.datetime实例相等的时候，比较就会传到协程，而协程没有实现比较方法，两个元素的元组就不能用。
    把这个当成asyncio.Task/curio.Task.
    """

    def __init__(self, wait_until, coro):
        self.coro = coro
        self.waiting_until = wait_until

    def __eq__(self, other):
        return self.waiting_until == other.waiting_until

    def __lt__(self, other):
        return self.waiting_until < other.waiting_until


class SleepingLoop:
    """着重于延迟协程执行的事件循环

    把这个想象成`asyncio.BaseEventLoop/curio.Kernel`。
    """

    def __init__(self, *coros):
        self._new = coros
        self._waiting = []

    def run_unitl_complete(self):
        # 启动所有协程
        for coro in self._new:
            wait_for = coro.send(None)
            heapq.heappush(self._waiting, Task(wait_for, coro))

        # 一直运行到没有任务
        while self._waiting:
            now = datetime.datetime.now()
            # 获取具有最近恢复时间的协程
            task = heapq.heappop(self._waiting)
            if now < task.waiting_until:
                # 我们比计算超前，等待直到下次运行时间
                delta = task.waiting_until - now
                time.sleep(delta.total_seconds())
                now = datetime.datetime.now()
            try:
                # 唤醒协程
                wait_until = task.coro.send(now)
                heapq.heappush(self._waiting, Task(wait_until, task.coro))
            except StopIteration:
                # 协程结束
                pass


@types.coroutine
def sleep(seconds):
    """暂停一个协程指定的秒数

    把这个当成`asyncio.sleep()/curio.sleep()`
    """

    now = datetime.datetime.now()
    wait_until = now + datetime.timedelta(seconds=seconds)
    # 将所有调用栈的协程暂停；`yield`的使用使得这个函数成为一个基于生成器的而不是基于`async`的协程
    actual = yield wait_until
    # 唤醒执行函数栈，返回我们实际上等待了多久
    return actual - now


async def countdown(label, length, *, delay=0):
    """ length秒的发射倒计时，等待delay秒数

    这是一个用户的普通写法。
    """

    print(label, 'waiting', delay, 'seconds before starting countdown')
    delta = await sleep(delay)
    print(label, 'starting after waiting', delta)
    while length:
        print(label, 'T-minus', length)
        waited = await sleep(1)
        length -= 1
    print(label, 'lift-off!')


def main():
    """启动事件循环，倒数三个独立的发射

    这就是一个用户会编写的经典代码。
    """
    loop = SleepingLoop(countdown('A', 5), countdown(
        'B', 3, delay=2), countdown('C', 4, delay=1))
    start = datetime.datetime.now()
    loop.run_unitl_complete()
    print('Total elapsed time is', datetime.datetime.now() - start)


if __name__ == '__main__':
    main()
```
**To be continued.**

[Wikipedia]: https://www.wikipedia.org/ "Wikipedia"

