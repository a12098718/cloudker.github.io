---
published: true
layout: post
title: python标准库-struct
category: python
tags: 
  - python
time: 2017.05.12 20:38:00
excerpt: 余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

---

余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。
近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

<!--more-->

# python标准库-struct-以二进制数据包解析字符串
该模块在python值和表示成python字符串的C结构体之间执行转换。这可以用来控制二进制数据在文件中的存储以及其他源发起的网络连接。它使用Format Strings作为布局C结构体的紧凑描述形式，并试图与python值之间进行转换。

> 注意：默认情况下，对卷入进来的C类型来说，打包一个包含pad字节的C结构体作为结果是为了保持适当的对齐，对齐会在解包时被考虑。这一行为是精选的以便于保证结构的字节和C结构在内存中的布局能够一一对应。如果想要控制平台依赖的数据格式化或忽略隐式的pad字节，可使用standard尺寸和对齐来取代native尺寸和对齐：参考字节序，尺寸和对齐(Byte Order, Size, and Alignment)一节获取详细内容。

## 函数和异常
该模块定义了下列的异常和函数：

- exception struct.error
	- 在多种情况下会抛该异常；参数是一个字符串用来描述错误是什么
- struct.pack(fmt, v1, v2, ...)
	- 返回一个根据指定格式fmt包含值v1,v2...打包的字符串。参数必须严格匹配fmt需要的值。
- struct.pack_into(fmt, buffer, offset, v1, v2, ...)
	- 根据fmt格式打包`v1, v2, ...`，把打包字节写入buffer中，从offset偏移处开始写。注意到offset是一个必须的参数。
	- 2.5新增。
- struct.unpack(fmt, string)
	- 解包字符串（假设是由`pack(fmt, ...)`打包），根据fmt指定的格式拆解。结果是一个元组，哪怕只包含一个元素。字符串必须严格的包含fmt要求的数据的个数（`len(string)`必须和`calcsize(fmt)`相等）。
- struct.unpack_from(fmt, buffer[, offset=0])
	- 解包buffer，根据fmt给定的格式拆解。结果是一个元组，哪怕只包含一个元素。buffer至少要有fmt要求的数据的个数（`len(buffer[offset:])`必须至少为`calcsize(fmt)`）。
	- 2.5新增。
- struct.calcsize(fmt)
	- 返回和给定fmt格式一致的结构体的尺寸。

## 格式化字符串
格式化字符串用于指定期望的打包和解包的数据布局。它们由Format Characters构成，其用于指定打包和解包的类型。此外，有一些特殊字符用于控制Byte Order,Size and Alignment。

### 字节序，尺寸和对齐
默认情况下，C类型表示成机器的本地模式和字节序，通过跳过pad字节来保持适当的对齐如果需要的话（这要看C编译器的法则）。

可替代的是，第一个字符可以用来指定字节序，尺寸和数据对齐，按照以下表格中的描述：

| 字符 | 字节序 | 尺寸 | 对齐 |
| --- | ------ | ---- | --- |
| @ | 本地 | 本地 | 本地 |
| = | 本地 | 标准 | 无 |
| < | 小端 | 标准 | 无 |
| > | 大端 | 标准 | 无 |
| ！ | 网络（大端） | 标准 | 无 |

第一个字符如果不是以上中的某一个，则假定为'@'。

本地字节序是大端还是小端，取决于宿主系统。例如，英特尔x86和AMD64(x86-64)是小端，摩托罗拉68000和PowerPC G5是大端。ARM和Intel Itanium可以切换字节序。使用sys.byteorder来检查系统的字节序。

本地尺寸和对齐取决于C编译器的sizeof操作符。这通常也结合本地字节序。

标准尺寸取决于格式化字符，查看Format Characters一节的表格。

注意'@'和'='的区别：都是用本地字节序，但是对齐和尺寸，后者用的是标准化模式。

'!'给那些声称自己不记得网络字节序是大端还是小端的卑微的灵魂。（玉涵：这波嘲讽满分:)）。

没有办法去指定非本地字节序（强制字节转换）；'<'和'>'使用要得当。

注意：
1. Padding仅仅自动在连在一起的结构成员中添加。编码结构体的首尾不会加pad。
2. 使用非本地尺寸和对齐时不会加pad，例如：'<','>','='和'!'。
3. 结构尾部想要按所需要的特殊类型对齐，则可以用重复的0来补充到尾部。查看Examples一节。

### 格式化字符
格式化字符有以下意义；在C和Python值之间转换应该是由它们的类型显式的给定。当使用标准尺寸时，‘标准化尺寸’一列指出了打包值的尺寸，也就是，当格式化字符串用'<,>,!,='中的某个起始。当使用本地化尺寸时，打包值的尺寸具有平台依赖性。

| 格式 |	C类型 | Python类型 | 标准尺寸 | 注意事项 |
| x | pad byte | no value | - | - | 	  	 
| c | char | string of length 1 | 1 | - | 	 
| b | signed char | integer | 1 | (3) |
| B | unsigned char | integer | 1 | (3) |
| ? | _Bool | bool | 1 | (1) |
| h | short integer | 2 | (3) |
| H | unsigned short | integer | 2 | (3) |
| i | int |	integer | 4 | (3) |
| I | unsigned int | integer | 4 | (3) |
| l | long | integer | 4 | (3) |
| L | unsigned long | integer |	4 |	(3) |
| q | long long | integer | 8 | (2), (3) |
| Q | unsigned long long | integer | 8 | (2), (3) |
| f | float | float | 4 | (4) |
| d | double | float | 8 | (4) |
| s | char[] | string | - | - |	  	 
| p | char[] | string | - | - |	  	 
| P | void * | integer | - | (5), (3) |

注意：
1. '?'转换码和C99标准定义的_Bool类型一致。如果type不可用，则以char模仿。在标准模式，始终表示一个字节。2.6新增。
2. 'q'和'Q'转换码仅在平台C编译器支持C的`long long`类型或Windows的`__int64`的本地模式下可用。它们在标准模式下总是可用的。2.2新增。
3. 当试图使用任何一个整型转换码打包一个非整型时，如果非整型有一个`__index__()`方法则会被用来作为生成转换值的函数。如果没有这个函数，则抛TypeError，`__int__()`方法会被继续尝试。这个`__int__()`不再推荐，会抛一个DeprecationWarning。2.7新增：使用`__index__()`方法。2.7之前都用`__int__()`方法转换，DeprecationWarning仅在浮点数转换时抛出。
4. 'f'和'd'转换码，打包表示使用IEEE 754二进制32或64格式，不管浮点数是否被平台所使用。
5. 'P'格式字符仅对本地字节序可用。字节序字符'='在宿主系统上选择大小端字节序。struct模块不会以本地字节序解析，因此'P'不可用。


格式化字符可以用一个表示重复的数字count来前导，例如，'4h'表示'hhhh'。

空白字符会被忽略，尽管数字count和它的格式不该包含空白字符。

对's'格式字符，数字count以字符串尺寸解析，和其他的意义不同，例如'10s'表示一个10字节字符串，'10c'表示10个字符。如果count没有给定，默认为1。打包时，字符串会被截断或用null字节做pad以修正正合适的格式。解包时，结果字符串就是严格的指定字节数。一个特殊情况是'0s'表示一个空字符串('0c'表示的是0个字节)。

'p'格式字符串编码‘Pascal string’，意味着一个短变量长度字符串会以修正字节数来存储。第一个字节存储字符串长度和255中较小的一个。后面的字节如果字符串传给pack()太长了（长于count-1），只有count-1个字节会被存储。如果比count-1小，则以null字节填充保证所有字节都被使用。注意对unpack()来说，'p'会耗尽count字节，但是返回的串永远不会大于255个字符。

对'P'格式化字符来说，返回值是Python整型或长整型，这取决于保存一个指向整型类型的指针所要的大小。NULL指针作为python整型0返回。打包指针尺寸值时，python整型或长整型会被使用。例如，Alpha和微处理器使用64位指针，表示的是python的长整型用以存储，其他平台是32位指针，使用python的整型即可。

对'?'格式字符，返回值是True或False。打包时，使用参数对象的真值。本地或标准bool表示中的0或1会被打包，任何非0值在解包时都被认为是真值True。

### 例子
> 所有例子都假定本地字节序，尺寸以及对其格式，在一个大端机子山。

简单的打包/解包三个整数：
```python
>>> from struct import *
>>> pack('hhl', 1, 2, 3)
'\x00\x01\x00\x02\x00\x00\x00\x03'
>>> unpack('hhl', '\x00\x01\x00\x02\x00\x00\x00\x03')
(1, 2, 3)
>>> calcsize('hhl')
8
```

解包字段可以被命名，通过制定它们变量或包裹命名元组的结果：
```python
>>> record = 'raymond   \x32\x12\x08\x01\x08'
>>> name, serialnum, school, gradelevel = unpack('<10sHHb', record)

>>> from collections import namedtuple
>>> Student = namedtuple('Student', 'name serialnum school gradelevel')
>>> Student._make(unpack('<10sHHb', record))
Student(name='raymond   ', serialnum=4658, school=264, gradelevel=8)
```

格式化字符的顺序会对尺寸有影响，因为padding需要满足不同的对齐需求：
```python
>>> pack('ci', '*', 0x12131415)
'*\x00\x00\x00\x12\x13\x14\x15'
>>> pack('ic', 0x12131415, '*')
'\x12\x13\x14\x15*'
>>> calcsize('ci')
8
>>> calcsize('ic')
5
```

下面的格式'llh0l'指定了两个pad字节在最后，假定long是4字节边界对齐：
```python
>>> pack('llh0l', 1, 2, 3)
'\x00\x00\x00\x01\x00\x00\x00\x02\x00\x03\x00\x00'
```

这仅在本地尺寸和对齐生效时有用；标准尺寸和对齐不会强迫对齐：
也可以看：array模块（打包二进制均匀数据以存储），xdrlib模块（打包解包XDR数据）。

## 类
struct模块也有自己定义的类型：
- class struct.Struct(format)
	- 返回一个新的Struct对象，根据format格式读写二进制数据。创建一个Struct对象来调用它的方法要比调用struct函数更为高效，因为format字符串仅仅需要编译一次。
	- 2.5新增
	- 编译的Struct对象支持下列方法和属性：
	- pack(v1, v2, ...)
		- 和pack()函数等同，使用编译的format。(len(result)等同于self.size)
	- pack_into(buffer, offset, v1, v2, ...)
		- pack_into()函数等同，使用编译的format。
	- unpack(string)
		- 和unpack()等同，使用编译的format。（len(string)必须等同self.size）
	- unpack_from(buffer, offset=0)
		- 和unpack_from()等同，使用编译的format。(len(buffer[offset:])至少为self.size)
	- format
		- 格式化字符串，用于构造Struct对象
	- size
		- 与format一致的结构体计算出的尺寸