---
layout: post
title: "使用pip一定更新所有包"
categories: [Python]
tags: [pip]
redirect_from:
  - /2017/10/25/
---

**记录以备后用**

- python脚本

```python
import pip
from subprocess import call

for dist in pip.get_installed_distributions():
    call("pip install --upgrade " + dist.project_name, shell=True)
```

- 一种更简单点儿的方法

```shell
pip freeze > requirements.txt
then do
pip install -r requirements.txt --upgrade
```



