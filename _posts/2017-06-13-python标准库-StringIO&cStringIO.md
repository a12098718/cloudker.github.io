---
published: true
layout: post
title: python标准库-StringIO&cStringIO
category: python
tags: 
  - python
time: 2017.06.13 9:26:00
excerpt: 余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

---

余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。
近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

<!--more-->

# python标准库-StringIO-像文件般读写字符串
该模块实现了一个file-like类，StringIO，可以读写字符串缓冲区（也被看做是内存文件）。查看文件对象章节了解相关描述。（对标准字符串，查看str和unicode相关章节。）

- class StringIO.StringIO([buffer])
	- 当一个StringIO对象被创建，它可以通过传递一个已有字符串给构造器来进行初始化。如果没有指定字符串，则StringIO起始为空。这两种情况下，初始化的文件指针位置都在起始0处。
	- StringIO对象可以接受Unicode或8位字符串，但是二者的混合体则需要甄别。如果二者均使用，在调用getvalue()时，8位字符串若不能被解析为7位ASCII（使用了第8位），则会抛出UnicodeError。

下面两个StringIO对象的方法需要特殊的说明：
- StringIO.getvalue():
	- 在StringIO对象close()之前的任意时刻，检索"file"的全部内容。查看上面关于混合使用Unicode和8位字符串的提示；这种混合可能会引起该方法抛UnicodeError。
- StringIO.close()
	- 释放内存缓冲区。尝试用一个已经关闭了的StringIO对象继续进行该操作，会引起一个ValueError。

示例使用：
```python
import StringIO

output = StringIO.StringIO()
output.write('First line.\n')
print >>output, 'Second line.'

# Retrieve file contents -- this will be
# 'First line.\nSecond line.\n'
contents = output.getvalue()

# Close object and discard memory buffer --
# .getvalue() will now raise an exception.
output.close()
```

# cStringIO-快速版StringIO
模块cStringIO提供一个类似StringIO模块的接口。过度使用StringIO.StringIO对象可能会比该模块使用StringIO()函数更有效率。

- cStringIO.StringIO([s])
	- 返回一个String-like流来读和写。
	- 尽管这是个返回内置类型对象的工厂方法，也没有办法通过子类去构筑你自己的版本。在其上设置属性是不可能的。在那些情境下，使用原始的StringIO模块。
	- 和StringIO不同的是，该模块不接受不能被编码成平坦的ASCII字符串的Unicode串。
	- 另一个不同是当调用StringIO()时，若携带一个字符串参数，则该模块创建的是只读对象。不像一个对象创建时若没有指定字符串参数，它就没有写方法（感觉这里原句有问题，我直接译过来了（Unlike an object created without a string parameter, it does not have write methods.））。这些对象不是一般性可见的。它们作为StringI和StringO出现在tracebacks中。

下面两个数据对象也同样被提供：
- cStringIO.InputType
	- 带有字符串参数使用StringIO()创建的类型对象
- cStringIO.OutputType
	- 不带参数使用StringIO()创建的类型对象

该模块也有C API；参考该模块的源码来获取更多信息。

例子使用方法：
```python
import cStringIO

output = cStringIO.StringIO()
output.write('First line.\n')
print >>output, 'Second line.'

# Retrieve file contents -- this will be
# 'First line.\nSecond line.\n'
contents = output.getvalue()

# Close object and discard memory buffer --
# .getvalue() will now raise an exception.
output.close()
```