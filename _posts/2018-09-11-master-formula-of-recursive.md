---
layout: post
title: "递归时间复杂度计算：master公式"
description: "递归master公式"
categories: [Code]
tags: [code]
redirect_from:
  - /2018/09/11/
---
**老忘，记下来随时翻一翻。**
# $T(n) = aT(\frac{n}{b}) + O(n^d) $

<font size="5">其中： $\log_{b}a > d \quad \Rightarrow \quad O(n^{\log_{b}a})$ <br>
&emsp;&emsp;&emsp;&nbsp;&nbsp;$\log_{b}a < d \quad \Rightarrow \quad O(n^{d})$ <br>
&emsp;&emsp;&emsp;&nbsp;&nbsp;$\log_{b}a = d \quad  \Rightarrow \quad O(n^{d}*\log{n})$ <br>
</font> 
**其中a为递归中子递归个数， n/b为子递归的数据规模。**
