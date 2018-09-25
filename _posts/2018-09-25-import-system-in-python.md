---
layout: post
published: false
title: Python中的import
guid: df4516d22d2d42e8aec10a69b36ac0a4
categories: [Python]
tags: [python]
redirect_from:
    - /2018/09/25/
---

**目录：**
* Kramdown table of contents
{:toc .toc}
* * * 
# 综述 {#general-overview}
import语句会执行两个操作：
1. 寻找给定名字的模块, 其实就是调用`__import__()`函数；
2. 把寻找的结果（也就是`__import__()`的返回值）绑定到本地命名空间的一个变量名上
如果模块是第一次被导入，Python就搜索这个模块，如果找到了，就创建一个`module`对象，然后初始化这个对象。如果找不到，就报`ModuleNotFoundError`错误。


# 包 {#package}
*****Python只有一种`moudle`对象，不管模块是用Python、C或者是其他语言实现的，都会被初始化成这种类型。**

我们可以把包想象成文件系统中的路径，然后把模块想象成路径中的文件。

**需要注意的是，所有的包都是模块，但并不是所有的模块也都是包。换句话说，包是一种特殊的模块。具体来说，所有具有`__path__`属性的模块都被认为是包。**

# 查找过程 {#searching}
## 模块缓存 {#the-module-cache}
import查找的第一个地方就是`sys.modules`，这个映射保存了所有之前已经导入的模块的缓存，也包括中间路径。比如我们之前导入过`foo.bar.baz`，那么`sys.modules`就会包含`foo`，`foo.bar`和`foor.bar.baz`的入口。

在`import`过程中，如果我们在`sys.modules`找到了对应模块名，并且对应该模块名的就是满足导入要求的模块，那么查找过程结束；如果对应该模块名的值为空，那么就会抛出`ModuleNotFoundError`错误。如果在`sys.modules`中找不到对应包名，Python就会继续查找过程。

`sys.modules`是可写的，也就是说我们可以删除其中其中的`key`，我们还可以把`key`设置为`None`, 这样当我们再次导入该`key`对应的包时，就会抛出`ModuleNotFoundError`。

## 查找和载入 {#finders-and-loaders}

