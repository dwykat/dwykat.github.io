---
layout: post
title: "Jekyll tips"
description: "Jekyll小技巧集合"
categories: [Tips, Jekyll]
tags: [jekyll, tips]
redirect_from:
  - /2018/09/21/
---
**目录：**
* Kramdown table of contents
{:toc .toc}
--- 

# jekyll tips
**使用jekyll博客中用到的小技巧**
## 1. 指定Kramdown的标题id  {#specify-a-id}

**今儿给人分享一个包含有id的链接的时候发现一个问题：标题为中文的id会是乱码并且很长很长，就想着有没有解决方法，然后发现Kramdown的确提供了自定义标题id的功能，使用方法就是在标题后面加上花括号然后里面写上自定义id就好了。**

**还有一个要注意的事情就是花括号里单词之间不能有空格。**

## 2. 给代码块添加行号 {#add-lineno-in-code-block}

```
Kramdown:
  syntax_highlighter_opts:
    block:
      line_numbers: true 
```
