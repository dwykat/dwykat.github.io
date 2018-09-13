---
layout: post
title: "给Jekyll博客添加Latex公式支持"
description: "给Jekyll博客添加Latex公式支持"
categories: [Tips]
tags: [tips, jekyll]
redirect_from:
  - /2018/09/13/
---

 昨天不清楚Jekyll能不能够支持Latex公式，所以写master的公式的时候就直接ipad手写截图了，今天还要在博客里写一些公式，就找了找有没有解决方法，果然是有的，体验非常棒。

**工具官网：** [MathJax](https://www.mathjax.org/) 

**使用方法：** 

把下面的JS调用代码以及配置插入到你Jekyll博客的header文件中：

```html
  <script type="text/x-mathjax-config">
    MathJax.Hub.Config({
      tex2jax: {
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
        inlineMath: [['$','$']]
      }
    });
  </script>
  <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script> 
```

**Note:** 默认的MathJax配置和Latex有所不同，主要体现在默认的inline公式是要用`\( \)`来标示，而Latex是用两个\$符号。所以上面配置里将默认修改成了Latex的标示方法。

**效果如下：**

$$W (s_{i}\leftarrow s_{j})=\frac{1}{1+\exp[ (p_{i}-p_{j})/K]}$$

<br>
* * *
**refs：**[StackOverflow](https://stackoverflow.com/questions/26275645/how-to-supported-latex-in-github-pages)
