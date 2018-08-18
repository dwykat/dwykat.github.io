---
layout: post
tille: stack overflow翻译之字典中浮点数key覆盖整型key 
description: stack overflow
categories: Python
tags: 
    - python 
    - stackoverflow
---
### Why can a floating point dictionary key overwrite an integer key with the same value?
{: .lead}
**问题:**

我正在[mypythonquiz](http://www.mypythonquiz.com)答题,然后问题[#45](http://www.mypythonquiz.com/question.php?qid=255)是回答下面代码的输出结果：
<!--break-->

```python
confusion = {}
confusion[1] = 1
confusion['1'] = 2
confusion[1.0] = 4

sum = 0
for k in confusion:
    sum += confusion[k]

print sum
```
代码输出结果为`6`，因为键`1.0`把`1`替换了。这让我觉得有些意外，这是一个有用的语言特性吗？


**Bakuriu的高票回答（76votes）**

首先，这个现象已经在`hash`函数的官方文档里解释的很清楚了:
> hash(object)
    Return the hash value of the object(if it has one). Hash values are integers.They are used to quickly compare dictionary keys during a dictionary loopup. Numeric values that compare equal have the same hash value(even if they are of different types, as is the case for `1` and `1.0`).

> Curabitur blandit tempus porttitor. Nullam quis risus eget urna mollis ornare vel eu leo. Nullam id dolor id nibh ultricies vehicula ut id elit.

其次，`object.__hash__`文档里指出了`hashing`的一个限制：
> object.__hash__(self)
    Called by built-in function `hash()` and for operations on members of hashed collections including `set`, `frozenset`, and `dict.__hash__()` should return an integer.**The only required property is that objects which compare equal have the same hash value;**

这并不是`python`独有的。`Java`也有类似的提醒：如果你打算实现`hashCode`函数，那么为了实现正常功能，你必须依据这样的准则实现：`x.euqals(y)`等价于`x.hashCode() == y.hashCode()`.

所以，在`python`中`1.0 == 1`，因此对`hash`函数来说，它就被强制性的实现`hash(1.0) == hash(1)`. 副作用就是在字典中，`1.0`和`1`作为键值是没有区别的，这也就导致了题目中的现象。


