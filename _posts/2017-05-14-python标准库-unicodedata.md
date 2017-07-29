---
published: true
layout: post
title: python标准库-unicodedata
category: python
tags: 
  - python
time: 2017.05.14 21:12:00
excerpt: 余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

---

余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。
近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

<!--more-->

# python标准库-unicodedata-Unicode数据库
该模块提供了到Unicode字符数据库的访问，其定义了所有Unicode字符的字符属性。数据库的数据时基于UnicodeData.txt文件5.2.0版本，该文件在ftp://ftp.unicode.org/可获取。

该模块使用UnicodeData文件格式5.2.0定义的同名和符号（查看 http://www.unicode.org/reports/tr44/tr44-4.html）。它定义了下列函数：
- unicodedata.lookup(name)
	- 通过name查找字符。如果给定name的字符找到了，则返回对应的Unicode字符。如果找不到则抛KeyError。
- unicodedata.name(unichr[, default])
	- 返回赋予Unicode字符unichr的名字字符串。如果没有定义名字，则返回default，此时如果没有给定default，则抛ValueError。
- unicodedata.decimal(unichr[, default])
	- 返回赋予Unicode字符unichr的整型十进制值。如果没有定义该值，返回default，如果此时没有给定，则抛ValueError。
- unicodedata.digit(unichr[, default])
	- 返回赋予Unicode字符unichr的整型digit数。如果没有定义值，返回default，如果此时没有给定，抛ValueError。
- unicodedata.numeric(unichr[, default])
	- 返回赋予Unicode字符unichr的浮点数。如果没有定义值，返回default，如果此时没有给定，抛ValueError。
- unicodedata.category(unichr)
	- 返回赋予Unicode字符unichr的字符串型通用分类。
- unicodedata.bidirectional(unichr)
	- 返回赋予Unicode字符unichr的字符串型bidirectional类。未定义则返回空串。
- unicodedata.combining(unichr)
	- 返回赋予Unicode字符unichr的整型的权威combing类。如果未定义combining类则返回0。
- unicodedata.east_asian_width(unichr)
	- 返回赋予Unicode字符unichr的字符串型东亚宽度。
	- 2.4新增
- unicodedata.mirrored(unichr)
	- 返回赋予Unicode字符unichr的整型镜像属性。如果字符在双向文本中被标记为“mirrored”则返回1，否则返回0。
- unicodedata.decomposition(unichr)
	- 返回赋予Unicode字符unichr的字符串型字符分解映射。如果没有映射则返回空串。
- unicodedata.normalize(form, unistr)
	- 返回Unicode字符串unistr的标准格式form。有效的值为'NFC','NFKC','NFD','NFKD'。
	- Unicode标准定义了多种Unicode字符串的标准化格式，基于权威和兼容的等价定义。Unicode中多个字符可以表示成多种形式。例如，字符U+00C7（LATIN CAPITAL LETTER C WITH CEDILLA）也可以表示为序列U+0043(LATIN CAPITAL LETTER C)U+0327(COMBINING CEDILLA)。
	- 对每个字符来说，有两种标准格式：标准格式C和标准格式D。标准格式D（NFD）也叫权威型分解，转换每个字符到它的分解格式。标准格式C（NFC）首先应用一个权威型分解，然后再用预组成字符进行组建。
	- 除了这两个格式，还有两种基于兼容等价的额外标准格式。Unicode中支持确定的字符，其标准与其他字符相统一。例如，U+2160（ROMAN NUMERAL ONE）和U+0049（LATIN CAPITAL LETTER I）是相同的字符串。然而，Unicode中为了存在字符集的兼容性这是支持的（例如gb2312）。
	- 标准格式KD（NFKD）将应用兼容性分解，也就是用等价体替代所有兼容性字符。标准格式KC（NFKC）首先应用兼容性分解，然后进行权威型分解。
	- 即使两个unicode字符串都是标准化的且看起来对于人类读者来说是相同的，如果其一结合了字符而其他的没有，则他们是不相同的。
	- 2.3新增。

除此之外，该模块暴露出下列常量：
- unicodedata.unidata_version
	- 该模块使用的Unicode数据库版本。
	- 2.3新增。
- unicodedata.ucd_3_2_0
	- 这是个拥有同整个模块相同方法的对象，但是使用的是Unicode数据库的3.2版本。这是因为应用需要这一特殊版本的Unicode数据库（例如IDNA）。
	- 2.5新增。

例子：
```python
>>> import unicodedata
>>> unicodedata.lookup('LEFT CURLY BRACKET')
u'{'
>>> unicodedata.name(u'/')
'SOLIDUS'
>>> unicodedata.decimal(u'9')
9
>>> unicodedata.decimal(u'a')
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
ValueError: not a decimal
>>> unicodedata.category(u'A')  # 'L'etter, 'u'ppercase
'Lu'
>>> unicodedata.bidirectional(u'\u0660') # 'A'rabic, 'N'umber
'AN'
```