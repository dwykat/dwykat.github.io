---
layout: post
title: "PEP 3333 -- Python Web Server Gateway Interface 阅读笔记[to be continued]"
description: "PEP 3333 Python Web Server Gateway Interface 阅读笔记"
categories: [Python, PEP]
tags: [python, pep]
redirect_from:
  - /2018/09/11/
---
**目录：**
* Kramdown table of content
{:toc .toc}
* * *

# PEP 3333 -- Python Web Server Gateway Interface 阅读笔记
## 前言
PEP 3333 是PEP 333的更新，对于原来和PEP 333兼容的应用和服务器，它们和PEP 3333仍然是兼容的。

对于Python3来说，编写应用或者服务器必须要遵循下面两个标题命名的版块里提到的规则：`A Note On String Types`,和`Unicode Issues`。

## 摘要
如果有人问你WSGI是啥，就可以参考下面这句话回答他了：“This document specifies a proposed standard interface between web servers and Python web applications or frameworks, to promote web application protability across a variety of web servers. ”

## 出发点和目标（Original Rationale and Goals）
**出发点：** <br>
Python目前有很多web应用程序框架，众多的选择给Python新用户带来一个问题：在选择web框架的同时，也限制了他们对于可用的web服务器的选择，反过来也是一样。

作为比较，Java也有很多的web应用框架，但是Java的`servlet`API可以让任何Java web应用框架在支持这种API的web服务器上运行。

所以，Python实现一个这种API也是有必要的。

**目标：**<br>

1.&emsp;鉴于现在还没有支持WSGI的服务器或者框架，所以WSGI必须容易实现，开发者的上手难度也要降到最小；需要注意的是，对于框架作者来说容易实现，并不意味着对web应用作者也是这样。所以WSGI没有添加花里胡哨的像返回对象和cookie处理这些可能会影响现有框架处理这些问题的东西，而是呈现了一个没有装饰的接口。WSGI的目标是能让现有服务器和应用或者框架之间更容易交流，而不是创建一个新的web框架。

还要注意的是，这个目标也阻止了WSGI使用任何在现发行的Python版本中没有的东西作为依赖。因此，本规范没有提出或要求新的标准库模块，而且WSGI中的任何内容都不需要大于2.2.2的Python版本。（不过，对于未来版本的Python来说，在标准库提供的web服务器中包含对该接口的支持将是一个好主意。）

2.&emsp;为了让现有的以及未来的框架和服务器更容易实现WSGI，WSGI也应该能很容易的创建请求预处理、响应后处理以及其它基于WSGI的`middleware`组件。

（文档里还yy了一下前景：）如果中间件足够简单且健壮，并且WSGI在服务器和框架中被广泛使用，那么就有可能出现一种全新的Python web应用程序框架：一个由松散耦合的WSGI中间件组件组成的框架。事实上，现有的框架作者甚至可能会选择重构他们的框架的现有服务，变得更像是和WSGI一起使用的库，而不像一个完整统一的框架。这可能会使得应用开发者可以去选择对于某一功能的最佳组合组件，而不是需要接受一整个框架的好处和坏处。

3.&emsp;最后应该提到，目前版本的WSGI没有对部署应用规定任何特殊机制。

## 规范综述（Specification Overview）
WSGI接口包括两端：服务器或者网关端，和应用或者框架端。服务器端调用一个由应用端提供的可调用对象。

下面这两段话有点儿迷糊，先记录原话和当前理解：

> In addition to "pure" servers/gateways and applications/frameworks, it is also possible to create "middleware" components that implement both sides of this specification. Such components act as an application to their containing server, and as a server to a contained application, and can be used to provide extended APIs, content transformation, navigation, and other useful functions.

对包含它的服务器表现的像应用，对包含它的应用表现的像服务器。

> Throughout this specification, we will use the term "a callable" to mean "a function, method, class, or an instance with a __call__ method". It is up to the server, gateway, or application implementing the callable to choose the appropriate implementation technique for their needs. Conversely, a server, gateway, or application that is invoking a callable must not have any dependency on what kind of callable was provided to it. Callables are only to be called, not introspected upon.

一个`callable`可以指一个函数，一个方法，一个类或者一个定义了`__call__`方法的实例。由实现`the callable`的服务器、网关或者应用程序根据它们的需要选择适当的实现技术。另一方面，调用`callable`的服务器、网关或应用程序禁止依赖提供给它的`callable`的类型。也就是说，`callbles`只是用来被调用的，而不是被内省（获取它们的类型）。

### 一个字符串类型需要注意的地方（A Note On String Types）

通常，HTTP处理的是字节，这就意味着这个规范主要就是关于如何处理字节。

字节内容总会有某种文本解释（textual interpretation），在Python中，字符串是处理文本最方便的方式。

但是在很多Python版本和实现中，字符串是`Unicode`，而不是字节。这就需要在一个可用的API和在HTTP上下文的文本中正确的转换字节和文本之间进行谨慎的权衡。

也基于此，WSGI定义了两种"string"：
* "Native" strings（总是使用`str`类型实现。），用于请求/响应头和元数据。
* "Bytestrings"（在Python3中使用`bytes`类型实现，在其他地方使用`str`类型实现），用于请求和响应的主体（比如`POST/PUT`输入数据和HTML页面的输出）。

但是不要搞混了：即使Python的`str`类型底层实际上`Unicode`，`native strings`的内容也必须能够通过`Latin-1`编码转换到字节。（细节参见下面的`Unicode Issues`章节）。

## 应用/框架端
应用程序对象（`application object`）只是一个接受两个 参数的可调用对象。术语`object`不应该被误解为需要一个实际的对象实例：函数、方法、类或带有`__calll__`方法的实例都可以作为应用程序对象使用。应用程序对象必须能够被多次调用，因为几乎所有的服务器/网管（CGI除外）都会发出重复请求。

（注意：虽然我们把它叫做应用程序对象，但这不应该被解释为应用程序开发者会使用WSGI作为web编程的API。WSGI是一个面向框架和服务器开发者的工具，而没有直接支持应用程序开发者的倾向。）

下面是两个应用程序对象；一个是函数，另一个是类：

```python
HELLO_WORLD = b"Hello world!\n"

def simple_app(environ, start_response):
    """Simplest possible application object"""
    status = '200 OK'
    response_headers = [('Content-type', 'text/plain')]
    start_response(status, response_headers)
    return [HELLO_WORLD]

class AppClass:
    """Produce the same output, but using a class

    (Note: 'AppClass' is the "application" here, so calling it
    returns an instance of 'AppClass', which is then the iterable
    return value of the "application callable" as required by
    the spec.

    If we wanted to use *instances* of 'AppClass' as application
    objects instead, we would have to implement a '__call__'
    method, which would be invoked to execute the application,
    and we would need to create an instance for use by the
    server or gateway.
    """

    def __init__(self, environ, start_response):
        self.environ = environ
        self.start = start_response

    def __iter__(self):
        status = '200 OK'
        response_headers = [('Content-type', 'text/plain')]
        self.start(status, response_headers)
        yield HELLO_WORLD
```

## 服务器/网关端
服务器或者网关为每一个从HTTP接收到的请求调用一次请求调用一次该请求对应的可调用应用程序。下面是一个简单的CGI网关，一个接收应用程序对象的函数。

```python
mport os, sys

enc, esc = sys.getfilesystemencoding(), 'surrogateescape'

def unicode_to_wsgi(u):
    # Convert an environment variable to a WSGI "bytes-as-unicode" string
    return u.encode(enc, esc).decode('iso-8859-1')

def wsgi_to_bytes(s):
    return s.encode('iso-8859-1')

def run_with_cgi(application):
    environ = {k: unicode_to_wsgi(v) for k,v in os.environ.items()}
    environ['wsgi.input']        = sys.stdin.buffer
    environ['wsgi.errors']       = sys.stderr
    environ['wsgi.version']      = (1, 0)
    environ['wsgi.multithread']  = False
    environ['wsgi.multiprocess'] = True
    environ['wsgi.run_once']     = True

    if environ.get('HTTPS', 'off') in ('on', '1'):
        environ['wsgi.url_scheme'] = 'https'
    else:
        environ['wsgi.url_scheme'] = 'http'

    headers_set = []
    headers_sent = []

    def write(data):
        out = sys.stdout.buffer

        if not headers_set:
             raise AssertionError("write() before start_response()")

        elif not headers_sent:
             # Before the first output, send the stored headers
             status, response_headers = headers_sent[:] = headers_set
             out.write(wsgi_to_bytes('Status: %s\r\n' % status))
             for header in response_headers:
                 out.write(wsgi_to_bytes('%s: %s\r\n' % header))
             out.write(wsgi_to_bytes('\r\n'))

        out.write(data)
        out.flush()

    def start_response(status, response_headers, exc_info=None):
        if exc_info:
            try:
                if headers_sent:
                    # Re-raise original exception if headers sent
                    raise exc_info[1].with_traceback(exc_info[2])
            finally:
                exc_info = None     # avoid dangling circular ref
        elif headers_set:
            raise AssertionError("Headers already set!")

        headers_set[:] = [status, response_headers]

        # Note: error checking on the headers should happen here,
        # *after* the headers are set.  That way, if an error
        # occurs, start_response can only be re-called with
        # exc_info set.

        return write

    result = application(environ, start_response)
    try:
        for data in result:
            if data:    # don't send headers until body appears
                write(data)
        if not headers_sent:
            write('')   # send headers now if body was empty
    finally:
        if hasattr(result, 'close'):
            result.close()
```

## 中间件：左右逢源的组件（Middleware: Components that Play Both Sides）

要注意的是，一个对象对于一些应用可能发挥服务器的作用，对于一些服务器，又可能表现得像应用。这种中间件组件能够执行以下功能：
* 根据目标URL，重写了对应`environ`之后，将一个请求路由到不同的应用程序对象
* 可以让多个应用或者框架在一个进程中并行运行
* 通过网络转发请求和响应，实现负载均衡和远程处理
* 执行内容后处理，比如应用XSL样式表

一般来说，中间件对于服务器/网关端和应用/框架端的接口都是透明的，而且不应该需要特殊支持。用户想要把中间件嵌入应用中只需要简单的把中间件提供给服务器，就好像中间件也是一个应用一样；而且（如果想要把中间件嵌入服务器）配置中间件组件调用应用程序，就好像中间件组件是一个服务器一样。当然，这个中间件包裹的应用实际上可能是另外一个包裹着另一个应用的中间件，以此类推，就创造出了被称为中间件栈的东西。

在很大程度上，中间件必须遵循服务器端和应用端的限制和需求。在一些情况下，中间件的要求要比一个单纯的服务器或者应用还要严格，这些情况会在规范中指出。

下面是一个很随意的例子，功能是将`text/plain`响应转换成pigLatin。

```python

m piglatin import piglatin

class LatinIter:

    """Transform iterated output to piglatin, if it's okay to do so

    Note that the "okayness" can change until the application yields
    its first non-empty bytestring, so 'transform_ok' has to be a mutable
    truth value.
    """

    def __init__(self, result, transform_ok):
        if hasattr(result, 'close'):
            self.close = result.close
        self._next = iter(result).__next__
        self.transform_ok = transform_ok

    def __iter__(self):
        return self

    def __next__(self):
        if self.transform_ok:
            return piglatin(self._next())   # call must be byte-safe on Py3
        else:
            return self._next()

class Latinator:

    # by default, don't transform output
    transform = False

    def __init__(self, application):
        self.application = application

    def __call__(self, environ, start_response):

        transform_ok = []

        def start_latin(status, response_headers, exc_info=None):

            # Reset ok flag, in case this is a repeat call
            del transform_ok[:]

            for name, value in response_headers:
                if name.lower() == 'content-type' and value == 'text/plain':
                    transform_ok.append(True)
                    # Strip content-length if present, else it'll be wrong
                    response_headers = [(name, value)
                        for name, value in response_headers
                            if name.lower() != 'content-length'
                    ]
                    break

            write = start_response(status, response_headers, exc_info)

            if transform_ok:
                def write_latin(data):
                    write(piglatin(data))   # call must be byte-safe on Py3
                return write_latin
            else:
                return write

        return LatinIter(self.application(environ, start_latin), transform_ok)


# Run foo_app under a Latinator's control, using the example CGI gateway
from foo_app import foo_app
run_with_cgi(Latinator(foo_app))
```
