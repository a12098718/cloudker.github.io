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
str.format()方法和Formatter类使用相同的格式化字符串语法（尽管对于Formatter来说，它的子类可以自定义格式化字符串语法）。

格式化字符串包含用{}包围的替代域。在{}之外的内容都是字面文本，这些文本无需替换简单的拷贝到输出即可。如果你需要在字面意义上包含一个花括号字符，可以写成`{{`和`}}`。

替代域语法如下：
```
replacement_field ::=  "{" [field_name] ["!" conversion] [":" format_spec] "}"
field_name        ::=  arg_name ("." attribute_name | "[" element_index "]")*
arg_name          ::=  [identifier | integer]
attribute_name    ::=  identifier
element_index     ::=  integer | index_string
index_string      ::=  <any source character except "]"> +
conversion        ::=  "r" | "s"
format_spec       ::=  <described in the next section>
```
用正式的术语来说，替代域可以以field_name开始来指定值会被格式化并插入到输出而不是替代域的那些对象。field_name可以跟随一个conversion域，前置一个感叹号'!'，一个format_spec，前置一个':'。这些为替代域指定了一个非默认格式。

也可以看看“迷你语言格式化说明书”一节。

field_name本身以arg_name开始，arg_name可以是数字或一个关键字。如果是数字，则表示一个位置参数，如果是关键字，表示命名关键字参数。如果数字的arg_names是0，1，2，。。。这样的格式化字符串，它们都可以被忽略，且数字0，1，2，。。。会自动按顺序插入。因为arg_name没有划定界限，使用格式化字符串指定一个任意的字典键是不可能的。arg_name可以以任意数字索引或属性表达式跟随。格式表达式`'.name'`通过`getattr()`获取命名属性，`[index]`使用索引`__getitem__()`查找元素。

2.7更新：位置参数可以被省略，因此`{} {}`和`{0} {1}`等价。

一些简单的格式化字符串例子：
```
"First, thou shalt count to {0}"  # References first positional argument
"Bring me a {}"                   # Implicitly references the first positional argument
"From {} to {}"                   # Same as "From {0} to {1}"
"My quest is {name}"              # References keyword argument 'name'
"Weight in tons {0.weight}"       # 'weight' attribute of first positional arg
"Units destroyed: {players[0]}"   # First element of keyword argument 'players'.
```
conversion域会引起强制类型转换。通常来说，格式化一个值由其本身的`__format__()`方法完成。然而，在一些情况下强制一个类型转成string是不被信赖的，此时需要覆盖它本身的格式化方法。通过在调用`__format__()`之前转换该值，可以绕开标准的转换逻辑。

一些例子：
```
"Harold's a clever {0!s}"        # Calls str() on the argument first
"Bring out the holy {name!r}"    # Calls repr() on the argument first
```
format_spec域包含一个值应该如何被表示的说明书，包含诸如域宽度，对齐，铺垫，十进制精度等等的细节说明。每种值类型可以定义它自己的"格式化迷你语言"或format_spec解释器。

大部分内置类型支持一个通用格式化迷你语言，在下一节会描述。

format_spec域也可以包含嵌套的替代域。这些嵌套替代域可以包含一个域名称，转换标志以及格式化说明，但是更深的嵌套则不被允许。format_spec中的替代域在format_spec字符串解释前是可替代的。这允许动态指定一个值的格式化。

查看“格式化例子”一节获取更多实例。

### 迷你语言格式化说明
格式化说明用于具有替换域的格式化字符串操作中，用于指示每个值如何表示（参考“格式化字符串语法”）。也可以直接传递给内置的format()函数。每个可可视化的类型都可以定义格式化说明如何被解释。

大部分内置类型实现了下面的格式化说明选项，尽管一些选项仅仅支持数值类型。

一个普通的转换是空串和你此前在该值调用的str()产生的结果相同。一个非空格式化串代表性的修改这个值。

标准格式化说明的常见格式如下：
```
format_spec ::=  [[fill]align][sign][#][0][width][,][.precision][type]
fill        ::=  <any character>
align       ::=  "<" | ">" | "=" | "^"
sign        ::=  "+" | "-" | " "
width       ::=  integer
precision   ::=  integer
type        ::=  "b" | "c" | "d" | "e" | "E" | "f" | "F" | "g" | "G" | "n" | "o" | "s" | "x" | "X" | "%"
```
如果指定了一个有效的align值，则可以用一个前置的fill字符，该字符可以是任意字符，默认是空格如果省略的话。在使用str.format()方法时，fill字符不能是花括号'{'或'}'。然而，在内嵌的替代域中可以插入花括号。这个限制不影响format()方法。

各种对齐选项的意义如下：

| 选项 | 意义 | 
| ---- | --- |
| '<' | 利用空格强迫域左对齐（大多数对象的缺省选项）|
| '>' | 利用空格强迫域右对齐（数字型缺省选项）|
| '=' | 强制在十进制数的符号后填充。用于按这样的格式打印域'+000000120'。这一对齐选项仅对数字类型有效。默认用0填充。|
| '^' | 利用空格强迫居中 |

注意除非指定了最小域宽，域宽始终和填充的data保持同样尺寸，而对齐选项在此情形下毫无意义。

sign选项仅仅对数字类型有效，可以是下列的值之一：

| 选项 | 意义 |
| --- | ---- |
| '+' | 指示一个应该用于正或负数的符号 |
| '-' | 指示一个只能用于负数的符号（这是默认行为） |
| space | 指示在正数前应有前导的空白，负数前应有前导的'-'号 |

‘#’选项仅对整数有效，且仅仅是二进制，八进制或十六进制输出。如果存在，则其指定输出时前置'0b','0o','0x'。

‘,’选项用于标识数以千计的分隔符。对本地已知的分隔符，可以用‘n’整型表示类型来代替。

2.7更改：增加了','选项（参考PEP 378）。

width是一个十进制数，定义了域宽的最小值。如果没有指定，默认就由填充的内容所决定。

当没有显示的给出对其格式时，对数值类型来说，以'0'字符前导的width域使用0填充，且区分符号。

precision是十进制数，表示在小数点后需要表示多少位，对浮点数来说后接一个'f'或'F'，或在小数点前后用'g'或'G'格式化浮点小数。对非数值类型，该域表示最大域宽——也就是说，可以填充多少个字符。对整型数来说precision不可用。

最后，type决定了数据应该如何被表示。

可用的字符串表示类型如下：

| 类型 | 意义 |
| --- | ---- |
| 's' | 字符串格式。这是string的默认类型，可以被省略 | 
| None | 和's'等同 |

整型可用表示类型如下：

| 类型 | 意义 |
| --- | ---- |
| 'b' | 二进制格式 |
| 'c' | 字符。转换整型成对应的unicode字符 |
| 'd' | 十进制整数 |
| 'o' | 八进制 |
| 'x' | 十六进制，小写 |
| 'X' | 十六进制，大写 |
| 'n' | 数字。和'd'等同，除了它使用了当前的地域设置插入了适当的数字分隔符 |
| None | 和'd'一样 |

除了上述类型，整型还可以用下面的浮点数表示类型格式化（除了'n'和None）。当如此做时，float()用于在转换前转换整型成浮点数。

可用的浮点数和小数的表示类型如下：

| 类型 | 意义 |
| --- | ---- |
| 'e' | 指数标志。用字母'e'表示指数，按科学计数法打印数值。默认精度为6。|
| 'E' | 指数标志。和'e'相同，除了使用'E'表示指数以外 |
| 'f' | 修正小数。按定点小数显示。默认精度为6 |
| 'F' | 修正小数。和'f'一样 |
| 'g' | 通用格式。对精度p>=1来说，它会取舍到p有效数字，格式化结果用定点小数或科学计数法,取决于它的量级。精确的规则如下：假设结果用'e'来表示，精度p-1可以有指数exp。则如果-4<=exp<p，数字以'f'表示，精度为p-1-exp。否则，按精度p-1，类型'e'表示。两种情况尾随的0都是无关紧要的，这些无关紧要的0和小数都会被移除，如果小数的后面没有有意义的数值跟随。正负无穷，正负0，nan，会被格式化为inf,-inf,0,-0和nan。|
| 'G' | 通用格式。和'g'相同，除了切换使用'E'类型表示如果数字过大的话。|
| 'n' | 数字。和'g'相同，除了它使用当前区域设置来插入一些适当的数字分隔符。|
| '%' | 百分数。数值乘上100然后按'f'格式显示，后接一个百分号。|
| None | 和'g'相同 |

### 格式化示例
本节包含了str.format()语法的实例，与旧的%格式化相比较。

多数情况下语法和旧的%格式一样，只是多了{}和:替代%而已。例如'%03.2f'可以写成'{:03.2f}'。

新的格式化语法也支持新的和不同的选项，下面给出例子：

利用位置访问参数:
```python
>>> '{0}, {1}, {2}'.format('a', 'b', 'c')
'a, b, c'
>>> '{}, {}, {}'.format('a', 'b', 'c')  # 2.7+ only
'a, b, c'
>>> '{2}, {1}, {0}'.format('a', 'b', 'c')
'c, b, a'
>>> '{2}, {1}, {0}'.format(*'abc')      # unpacking argument sequence
'c, b, a'
>>> '{0}{1}{0}'.format('abra', 'cad')   # arguments' indices can be repeated
'abracadabra'
```

通过名字访问参数：
```python
>>> 'Coordinates: {latitude}, {longitude}'.format(latitude='37.24N', longitude='-115.81W')
'Coordinates: 37.24N, -115.81W'
>>> coord = {'latitude': '37.24N', 'longitude': '-115.81W'}
>>> 'Coordinates: {latitude}, {longitude}'.format(**coord)
'Coordinates: 37.24N, -115.81W'
```

访问参数属性：
```python
>>> c = 3-5j
>>> ('The complex number {0} is formed from the real part {0.real} '
...  'and the imaginary part {0.imag}.').format(c)
'The complex number (3-5j) is formed from the real part 3.0 and the imaginary part -5.0.'
>>> class Point(object):
...     def __init__(self, x, y):
...         self.x, self.y = x, y
...     def __str__(self):
...         return 'Point({self.x}, {self.y})'.format(self=self)
...
>>> str(Point(4, 2))
'Point(4, 2)'
```

访问参数元素：
```python
>>> coord = (3, 5)
>>> 'X: {0[0]};  Y: {0[1]}'.format(coord)
'X: 3;  Y: 5'
```

替换%s和%r:
```python
>>> "repr() shows quotes: {!r}; str() doesn't: {!s}".format('test1', 'test2')
"repr() shows quotes: 'test1'; str() doesn't: test2"
```

文本对齐，指定宽度：
```python
>>> '{:<30}'.format('left aligned')
'left aligned                  '
>>> '{:>30}'.format('right aligned')
'                 right aligned'
>>> '{:^30}'.format('centered')
'           centered           '
>>> '{:*^30}'.format('centered')  # use '*' as a fill char
'***********centered***********'
```

替换%+f,%-f,% f,指定符号：
```python
>>> '{:+f}; {:+f}'.format(3.14, -3.14)  # show it always
'+3.140000; -3.140000'
>>> '{: f}; {: f}'.format(3.14, -3.14)  # show a space for positive numbers
' 3.140000; -3.140000'
>>> '{:-f}; {:-f}'.format(3.14, -3.14)  # show only the minus -- same as '{:f}; {:f}'
'3.140000; -3.140000'
```

替换%x和%o，转换值到两个不同进制：
```python
>>> # format also supports binary numbers
>>> "int: {0:d};  hex: {0:x};  oct: {0:o};  bin: {0:b}".format(42)
'int: 42;  hex: 2a;  oct: 52;  bin: 101010'
>>> # with 0x, 0o, or 0b as prefix:
>>> "int: {0:d};  hex: {0:#x};  oct: {0:#o};  bin: {0:#b}".format(42)
'int: 42;  hex: 0x2a;  oct: 0o52;  bin: 0b101010'
```

使用逗号作为以千计的分隔符：
```python
>>> '{:,}'.format(1234567890)
'1,234,567,890'
```

表示百分数：
```python
>>> points = 19.5
>>> total = 22
>>> 'Correct answers: {:.2%}'.format(points/total)
'Correct answers: 88.64%'
```

使用特殊的类型格式化：
```python
>>> import datetime
>>> d = datetime.datetime(2010, 7, 4, 12, 15, 58)
>>> '{:%Y-%m-%d %H:%M:%S}'.format(d)
'2010-07-04 12:15:58'
```

嵌套参数和更复杂的例子：
```python
>>> for align, text in zip('<^>', ['left', 'center', 'right']):
...     '{0:{fill}{align}16}'.format(text, fill=align, align=align)
...
'left<<<<<<<<<<<<'
'^^^^^center^^^^^'
'>>>>>>>>>>>right'
>>>
>>> octets = [192, 168, 0, 1]
>>> '{:02X}{:02X}{:02X}{:02X}'.format(*octets)
'C0A80001'
>>> int(_, 16)
3232235521
>>>
>>> width = 5
>>> for num in range(5,12):
...     for base in 'dXob':
...         print '{0:{width}{base}}'.format(num, base=base, width=width),
...     print
...
    5     5     5   101
    6     6     6   110
    7     7     7   111
    8     8    10  1000
    9     9    11  1001
   10     A    12  1010
   11     B    13  1011
```

## 模板字符串
2.4新增。

模板提供了更简单的字符串替换，在PEP 292中有所描述。模板支持$替换而不是常规的%替换，使用下列法则：
- `$$`替换成单个$
- `$identifier`替换一个键为identifier的映射成员。默认情况，identifier必须是python标记符的拼写。遇到第一个非标记符字符时，$解析终止。
- `${identifier}`和`$identifier`一样。当有效的标记符跟随一个占位符，但又不是占位符的一部分时使用。诸如`${noun}ification`。

其余的用法都会抛ValueError。

string模块提供了Template类用于实现这些规则。Templates方法如下：
- class string.Template(template)
	- 构造器有一个单一参数，是个模板字符串。
	- substitute(mapping[, **kws])
		- 执行模板替换，返回新串。mapping可以是任意的字典类对象。或者，你可以提供关键字参数作为占位符。当mapping和kws都给定时且存在副本，从kws获取的占位优先。
	- safe_substitute(mapping[, **kws])
		- 和substitute()类似，只是当占位符不在mapping和kws中时，不会抛KeyError，而是使用原始的占位符作为替代出现。同时，和substitute()不同的是，任意错误的使用$不会抛ValueError而是简单的返回一个'$'。
		- 其他异常也可能出现，该方法被称作是“安全的”因为替换永远是试图返回一个可用字符串而不是抛异常。另一方面，safe_substitute()可能是非安全的其他东西，因为它默认忽略畸形的模板跟随字符，不匹配的括号以及不合法的（python标识符）占位符。
	- Template实例也支持一个公用的数据属性：
	- template
		- 传递给构造器template参数的对象。通常来讲，你不应该改变他，但它也不强制是只读的。
这里有一个如何使用模板的例子
```python
>>> from string import Template
>>> s = Template('$who likes $what')
>>> s.substitute(who='tim', what='kung pao')
'tim likes kung pao'
>>> d = dict(who='tim')
>>> Template('Give $who $100').substitute(d)
Traceback (most recent call last):
...
ValueError: Invalid placeholder in string: line 1, col 11
>>> Template('$who likes $what').substitute(d)
Traceback (most recent call last):
...
KeyError: 'what'
>>> Template('$who likes $what').safe_substitute(d)
'tim likes $what'
```
高级用法：你可以继承Template类来定制占位符语法，定界字符，或者完整的正则表达式来传递模板字符串。为了如此做，你可以覆盖这些类属性：
- delimiter——字面值字符串描述了引介分隔符的占位符。默认值是$。注意到这不应该是一个正则表达式，实现体可以调用re.escape()，如果需要的话。
- idpattern——正则表达式，描述了non-braced占位符（适当情况自动加braces）。默认值是正则表达式[_a-z][_a-z0-9]*。

或者，你可以提供整个正则表达式，覆盖类属性pattern。如果这样做，值必须是正则表达式对象，具有4个命名分组。捕捉的分组对应上面的规则，以及无效的站位符规则：
- escaped——匹配escape序列的组，例如$$是默认模式。
- named——匹配unbraced占位符名字；捕捉组不应该包含定界符。
- braced——匹配brace enclosed占位符名字；不应该包含定界符或braces。
- invalid——匹配任意其他的定界符语法（通常是一个单定界符），它应该在正则表达式最后出现。

## 字符串函数
以下是可用来操作字符串和Unicode对象的函数。它们不可用作字符串对象的方法。

- string.capwords(s[, sep])
	- 用str.split()分割参数成词，用str.capitalize()大写每个首字母，使用str.join()加入大写单词。如果可选的第二个参数sep缺省或者是None，用空白字符替换，前后的多余空白字符被移除，否则sep用于分割和加入。
- string.maketrans(from,to)
	- 返回一个传递给translate()合适的翻译表，这可以映射每个在from中的字符到to中同样的位置；from和to必须拥有相同长度。
> 注意：不要使用继承自`lowercase`和`uppsercase`的字符串作为参数；在一些区域，它们不具有相同长度。对大小写转换，始终要用`str.lower()`和`str.upper()`。 

## 不推荐的字符串函数
下列函数也是作为字符串和Unicode对象定义的方法；可以看String Methods一节获取更多信息。你应该不使用这些函数，尽管直到python 3它们才会被移除。这些函数在这一模块中定义：
- string.atof(s)
	- 2.0起不推荐，使用float()代替。
	- 转换string到float。string必须是浮点的字面值文本，可以有一个前导符号(+、-)。注意这一行为与内置函数float()相同。
	- 当传入无效值或无穷大时，字符串可能会返回，具体取决于底层C库。
- string.atoi(s[, base])
	- 2.0起不再推荐，使用int()代替。
	- 转换字符串到给定base进制的整数。string必须由十进制数组成，可以有一个前导符号（+、-）。base默认为10，如果base是0，则依赖于string的前导字符：0x或0X表示16，0表示8，其他表示10。如果是16进制，0x或0X前导始终被接受，尽管不被需要。这一方法和内置函数int()一致。
- string.atol(s[, base])
	- 2.0起不再推荐，使用long()代替。
	- 基本同atoi。只是可以尾随一个l或L字符。
- string.capitalize(word)
	- 返回一个word的拷贝，仅仅将首字母大写。
- string.expandtabs(s[, tabsize])
	- 用一或多个空格替代tabs展开一个字符串，依赖于当前列和给定的tab尺寸。列数在换行后会重置为0。他不会理解其他非打印字符或转义序列。tab尺寸默认为8。
- string.find(s,sub[,start[,end]])
	- 返回s中切片范围start:end中包含sub子串的最小索引。如果找不到返回-1。
- string.rfind(s,sub[,start[,end]])
	- 和find差不多，从后往前查找。
- string.index(s,sub[,start[,end]])
	- 和find()基本一致，只是在找不到时抛ValueError。
- string.rindex(s,sub[,start[,end]])
	- 和rfind()基本一致，只是在找不到时抛ValueError。
- string.count(s,sub[,start[,end]])
	- 返回切片中sub子串匹配的数量。
- string.lower(s)
	- 返回所有字符转为小写的s拷贝
- string.split(s[, sep[, maxsplit]])
	- 返回字符串s的单词列表。如果sep缺省或为None，单词以空白字符作为分割（空格，tab，换行，回车，换页）。如果给定了，则作为分割单词的字符串。返回列表会比string中非覆盖分隔符出现的次数多一个。如果maxsplit给定了，那么至多拆分maxsplit次，剩余的字符串整个作为最后一个元素出现（因此，列表最多有maxsplit+1个元素）。如果maxsplit是-1，则没有拆分次数限制（尽可能的拆分）。
	- split对空串的行为取决于sep的值。如果sep没有给定，或者指定成None，结果还是个空串。如果sep是其他任何值，结果就是一个包含一个元素的列表，这个元素是一个空字符串。
- string.rsplit(s[, sep[, maxsplit]])
	- 返回一个字符串s的字符列表，从后向前扫描。单词列表结果和split()相同，除非第三个可选参数maxsplit显示的制定了非0值。如果maxsplit给定，至多拆分maxsplit次，此时左侧剩余未拆分的字符串作为第一个列表元素返回（因此，列表至多有maxsplit+1个元素）。
	- 2.4新增
- string.splitfields(s[, sep[, maxsplit]])
	- 和split()行为相同。（过去，split()仅仅只有一个参数，splitfields只有两个参数。）
- string.join(words[, sep])
	- 使用sep从中间连结列表或元组中的单词。sep默认值是单个空白字符。`string.join(string.split(s, sep), sep)`永远等于`s`。
- string.joinfields(words[, sep])
	- 和joint()行为相同。（过去，join()仅仅只有一个参数，joinfields()仅有两个参数。）注意到string对象没有joinfields方法，使用join()方法替代。
- string.lstrip(s[, chars])
	- 返回移除前导字符的字符串拷贝。如果chars缺省或为None，移除空白字符。如果给定，则chars必须是一个字符串；字符串开始处和chars匹配的字符会被删除。
	- 2.2.3更新：增加chars参数。chars参数在2.2版本之前不能被传递。
- string.rstrip(s[, chars])
	- 返回移除后导字符的字符串拷贝。如果chars缺省或为None，移除空白字符。如果给定，则chars必须是一个字符串；字符串结尾处和chars匹配的字符会被删除。
	- 2.2.3更新：增加chars参数。chars参数在2.2版本之前不能被传递。
- string.swapcase(s)
	- 返回s的拷贝，所有小写字母转为了大写，大写字母转为了小写。
- string.translate(s, table[, deletechars])
	- 删除s中出现在deletechars中的字符。使用table翻译字符，table必须是一个256字符的字符串，它给定了每个字符的值，通过它的次序索引。如果table是None，则仅仅执行删除字符的操作。
- string.upper(s)
	- 返回s的拷贝，但是所有小写字母都被转成了大写。
- string.ljust(s, width[, fillchar])
- string.ljust(s, width[, fillchar])
- string.center(s, width[, fillchar])
	- 这些函数分别制定左对齐，右对齐，居中，字符串宽度由width指定。它们返回一个至少width字符宽的，由fillchar做padding填充的字符串。该字符串永不会截断。
- string.zfill(s, width)
	- s是数字字符串，该函数用数字0垫在左边，直到width长度。带符号的字符串也可以正确处理。
- string.replace(s, old, new[, maxreplace])
	- 返回s的拷贝，所有子串old由new替代。如果可选参数maxreplace给定，则至多替换到maxreplace次。