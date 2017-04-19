---
published: true
layout: post
title: python标准库-string
category: python
tags: 
  - python
time: 2017.04.19 21:31:00
excerpt: 余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

---

余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。
近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

<!--more-->

# Python标准库-string-通用字符串操作
源码：Lib/string.py

----------
string模块包含相当数量有用的常量和类，以及一些尚可用于字符串方法的过时的遗留功能。此外，python内置string类支持序列类型方法，在“序列类型-str,unicode,list,tuple,bytearray,buffer,xrange”一节有描述，"String methods"一节中也有特殊字符串方法描述。使用模板字符串或%操作符来输出格式化字符串，这在"String Formatting Operations"一节中有所描述。基于正则表达式的字符串函数请参考re模块。

## 字符串常量
定义于模块的常量如下：
- string.ascii_letters
	- ascii_lowercase和ascii_uppercase的连结，该值和区域设置无关。
- string.ascii_lowercase
	- 小写串'abcdefghijklmnopqrstuvwxyz'。该值和区域设置无关且不会改变。
- string.ascii_uppercase
	- 大写串'ABCDEFGHIJKLMNOPQRSTUVWXYZ'。该值和区域设置无关且不会改变。
- string.digits
	- 字符串'0123456789'
- string.hexdigits
	- 字符串'0123456789abcdefABCDEF'
- string.letters
	- lowercase和uppercase的连结。该值和区域设置有关，当locale.setlocale()调用时更新。
- string.lowercase
	- 包含所有小写字符的字符串。大多数系统上都是'abcdefghijklmnopqrstuvwxyz'。该值和区域设置有关，当locale.setlocale()调用时会更新。
- string.octdigits
	- 字符串'01234567'
- string.punctuation
	- C中被认为是标点字符的ASCII字符串
- string.uppercase
	- 包含所有大写字符的字符串。大多数系统上是'ABCDEFGHIJKLMNOPQRSTUVWXYZ'。该值和区域设置有关，在locale.setlocale()调用时更新。
- string.whitespace
	- 包含所有空白字符的字符串。大多数系统上包括空格，tab，换行，回车和垂直制表符。

## 自定义格式化字符串
2.6新增。
内置str和unicode类通过PEP3101描述的str.format()方法提供了复杂的变量替换和格式化值的能力。string模块的Formatter类允许创建和定制个人的格式化串行为，实现手法和内置的format()方法一致。
- class string.Formatter
	- Formatter类拥有下列公用方法：
	- format(format_string, *args, **kwargs)
		- 主要的API方法。它需要一个格式化字符串和任意一组位置及关键字参数。vformat()调用的包装。
	- vformat(format_string, args, kwargs)
		- 处理格式化串的实际工作。当你想要传递一个预定义字典参数时，他作为分离的函数暴露出来，而不是使用*args和**kwargs语法解包再重新打包字典作为自己的参数。vformat()做分离格式化字符串成字符数据以及替换域的工作。它会调用下面的各种方法。
	- 此外，Formatter定义了一些希望子类替换的方法：
	- parse(format_string)
		- 循环遍历format_string并返回一个迭代元组（literal_text,field_name,format_spec,conversion）。这是vformat()被用于分解成文本或替换字段的字符串。
		- 元组中的值在概念上代表了追随其后的单个替换字段的字面文本。如果没有字面文本（如果两个字段域相继出现有可能会发生），则字面值literal_text是长度为0的字符串。如果没有替换域，则field_name,format_spec和conversion都会是None。
	- get_field(field_name,args,kwargs)
		- 给出parse()返回的字段名field_name，将其转换成对象格式。返回一个元组（obj, used_key）。默认版本使用在PEP3101中定义的字符串格式，例如"0[name]"或"label.title"。args和kwargs和传给vformat()的一样。返回值used_key具有和get_value()的key参数同样的意义。
	- get_value(key,args,kwargs)
		- 检索一个给定的字段值。key参数可以是整型或字符串。如果是整型，则表示args参数的下标索引，如果是字符串，则表示kwargs中的命名参数。
		- args参数被设为vformat()的位置列表参数，kwargs被设为字典参数。
		- 对混合字段名，这些函数仅仅为字段名字的第一个组件所调用；后面的组件控制一般属性和索引操作。
		- 因此，字段表达式'0.name'会引起get_value()使用0这个key参数调用。name属性在get_value()返回后会被内置的getattr()函数调用查找到。
		- 如果元素的索引或关键引用不存在，抛IndexError或KeyError。
	- check_unused_args(used_args,args,kwargs)
		- 如果需要的话检查未使用的参数。该函数的参数是所有参数关键字集合，实际上是格式化字符串的参考（整型或位置参数，命名参数字符串）以及传递给vformat的args和kwargs的引用。通过这些参数可以计算那些没被使用的参数。check_unused_args()在检查失败是假定抛一个异常。
	- format_field(value,format_spec)
		- 简单的调用全局的format()内置函数。该函数被提供是为了让子类覆盖。
	- convert_field(value,conversion)
		- 给定一个转换类型（如parse()方法返回的元组一样）来转换值（由get_field()返回）。默认版本支持's'(str),'r'(repr)和'a'(ascii)转换类型。

## 格式化字符串语法
待续。。。