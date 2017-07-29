---
published: true
layout: post
title: python标准库-fpformat
category: python
tags: 
  - python
time: 2017.06.20 15:55:00
excerpt: 余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

---

余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。
近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

<!--more-->

# python标准库-fpformat-浮点数转换
2.6起不再推荐：fpformat模块在python 3中已移除。

fpformat模块定义了处理在百分百纯粹的python中表示为浮点数的函数。

> 注意：该模块不是必需的：这里的任何东西都可以通过%字符串介入符来完成，字符串介入符曾在String Formatting Operations节中描述。

fpformat模块定义了下列的函数和一个异常：
- fpformat.fix(x, digs)
	- x格式为`[-]ddd.ddd`，有着digs数量的数在小数点后且小数点前至少有一个数。如果digs<=0，小数点无作用。
	- x可以是一个数，或是一个看起来像是前者的字符串。digs是整数。
	- 返回值是字符串。
- fpformat.sci(x, digs)
	- x格式为`[-]d.dddE[+-]ddd`，有着digs数量的数在小数点后且小数点前严格有一个数字。如果digs<=0，保留一个数字，小数点无作用。
	- x可以是一个真实数字，或者是一个看起来像前者的字符串。digs是整数。
	- 返回值是字符串。
- exception fpformat.NotANumber
	- 当字符串传递给fix()或sci()作为x参数时，如果其看起来不像个数字，则抛出异常。这是ValueError的子类，标准异常是字符串。该异常的值正是这个引起异常抛出的不适当格式化串。

例子：
```python
>>> import fpformat
>>> fpformat.fix(1.23, 1)
'1.2'
```