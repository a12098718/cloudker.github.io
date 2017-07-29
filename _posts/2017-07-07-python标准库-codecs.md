---
published: true
layout: post
title: python标准库-codecs
category: python
tags: 
  - python
time: 2017.07.07 22:58:00
excerpt: 余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

---

余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。
近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

<!--more-->

# python标准库-codecs-编解码注册和基类
该模块定义了标准Python编解码的基类，并提供了访问内部Python编解码注册的方法，其用于管理编解码以及处理查找进程的错误处理。

它定义了下面的函数:

- codecs.encode(obj[, encoding[, errors]])
	- 使用codec注册的encoding来编码obj。默认encoding为'ascii'。
	- 错误可以用于设置渴望错误处理的机制。默认错误处理是'strict'，意味着编码错误会抛出ValueError（或一个更为特定的编码子类，如UnicodeEncodeError）。参考Codec Base Classes获取错误控制的更多信息。
	- 2.4新增。
- codecs.decode(obj[, encoding[, errors]])
	- 使用encoding解码obj。默认encoding为'ascii'。
	- 错误可以用于设置渴望错误处理的机制。默认错误处理是'strict'，意味着编码错误会抛出ValueError（或一个更为特定的编码子类，如UnicodeEncodeError）。参考Codec Base Classes获取错误控制的更多信息。
	- 2.4新增。
- codecs.register(search_function)
	- 注册一个编码搜索函数。搜索函数期望携带一个参数，全部小写的编码名称，并返回一个拥有以下属性的CodecInfo对象：
		- name 编码名称
		- encode 无国籍编码函数
		- decode 无国籍解码函数
		- incrementalencoder 增加的编码类或工厂函数
		- incrementaldecoder 增加的解码类或工厂函数
		- streamwriter 写入流类或工厂函数
		- streamreader 读入流类或工厂函数
	- 多种函数或类都携带下面的参数：
	- encode和decode：它们必须是和Codec实例的encode()/decode()拥有同样接口的函数或方法。函数/方法期望工作于无国籍的模式。
	- incrementalencoder和incrementaldecoder。它们需要作为工厂函数并提供下面的接口：
		- factory(errors='strict')
	- 工厂函数必须返回支持基类IncrementalEncoder和IncrementalDecoder分别定义的接口的对象。Incremental codecs可以保留国籍。
	- streamreader和streamwriter。它们需要作为工厂函数并提供下面的接口：
		- factory(stream, errors='strict')
	- 工厂函数必须返回支持基类StreamReader和StreamWriter分别定义的接口的对象。Stream codecs可以保留国籍。
	- errors的可能值：
		- 'strict'：编码错误发生时抛出异常
		- 'replace'：替换畸形数据为合适的替换标记，比如'?'或'\ufffd'
		- 'ignore'：忽略畸形数据并在无更多提示下继续
		- 'xmlcharrefreplace'：以适当的XML字符参考替换（仅对编码）
		- 'backslashreplace'：用反斜杠转义替换（仅对编码）
	- 也可能是通过register_error()定义的其他错误处理名。
	- 当搜索函数找不到给定的编码时，应该返回None。
- codecs.lookup(encoding)
	- 查看python的codec注册信息并返回一个CodecInfo对象。
	- 编码会首先在注册缓存中查找。如果没有找到，则已注册的搜索函数列表会进行搜索。如果没有找到CodecInfo对象，则抛LookupError。否则，CodecInfo对象会保存在缓冲并返回给调用者。

为了简单的访问各种codecs，模块提供了这些额外的函数，它们使用lookup()来进行codec查找：
- codecs.getencoder(encoding)
	- 查找给定的encoding codec并返回它的编码函数。
	- 如果没有找到encoding则抛LookupError。
- codecs.getdecoder(encoding)
	- 查找给定的encoding codec并返回它的解码函数。
	- 如果没有找到encoding则抛LookupError。
- codecs.getincrementalencoder(encoding)
	- 查找给定的encoding codec并返回它的incremental编码类或工厂函数。
	- 如果encoding找不到或codec不支持incremental编码器则抛LookupError。
	- 2.5新增。
- codecs.getincrementaldecoder(encoding)
	- 查找给定encoding的codec并返回它的incremental解码类或工厂函数。
	- 如果encoding找不到或codec不支持incremental解码器则抛LookupError。
	- 2.5新增。
- codecs.getreader(encoding)
	- 查找给定encoding codec并返回它的StreamReader类或工厂函数。
	- 如果encoding找不到则抛LookupError。
- codecs.register_error(name, error_handler)
	- 注册错误控制函数error_handler使用name作为名字。error_handler可以在编解码期间遇到一个错误时被调用，name指定作为错误的参数。
	- 对编码来说，error_handler可以以一个UnicodeEncodeError实例调用，它包含了错误的信息。错误控制器必须抛出它或一个不同的异常抑或是返回一个元组，元组中包含了输入中不可编码的部分以及一个编码应该继续下去的位置。编码器会编码替代物并在该特定处继续编码原始的输入。负数位置值会被当成是从结尾开始倒数的位置。如果结果位置越界了，则抛出IndexError。
	- 解码和翻译工作也是相似的，除了UnicodeDecodeError或UnicodeTranslateError会传递给控制器并且错误处理器的替代品会被直接放置到输出中。
- codecs.lookup_error(name)
	- 返回此前注册的以name为名字的错误处理器。
	- 如果没有找到，抛LookupError。
- codecs.strict_errors(exception)
	- 实现strict错误控制器：每个编码或解码错误都会抛UnicodeError。
- codecs.replace_errors(exception)
	- 实现replace的错误控制：畸形数据会被替换为合适的替代字符比如字节字符串中用'?'，'\ufffd'在Unicode字符串。
- codecs.ignore_errors(exception)
	- 实现ignore错误控制：畸形数据被忽略，编解码过程会继续且无额外的提示。
- codecs.xmlcharrefreplace_errors(exception)
	- 实现xmlcharrefreplace错误控制（仅对编码而言）：不可编码字符会被替换为合适的XML字符。
- codecs.backslashreplace_errors(exception)
	- 实现backslashreplace错误控制（仅对编码而言）：不可编码字符会被替换为反斜杠转义序列。

为了简单的使用编码文件或流工作，该模块也定义了这些通用函数：
- codecs.open(filename,mode[,encoding[,errors[,buffering]]])
	- 使用给定mode打开编码文件并返回一个提供转换编解码的包裹版本。默认文件模式是'r'，意味着以读模式打开文件。
	- 注意：包裹版本仅接受codecs定义的对象格式，例如，对大部分内置codecs为Unicode对象。输出也同样是codec依赖的并且通常是一个Unicode对象。
	- 注意：文件总是以二进制模式打开，尽管没有指定二进制模式。这是为了防止使用8位值编码而引起数据丢失。这意味对读写来说着没有自动的'\n'转换。
	- encoding指定了该文件所用的编码。
	- errors用来定义错误控制。默认为'strict'，也就是说当编码错误发生时会抛ValueError。
	- buffering和内置的open()函数具有同样的意义。默认为行缓冲区。
- codecs.EncodedFile(file, input[, output[, errors]])
	- 返回一各支持编码翻译转换的包裹版本。
	- 写入包裹文件的字符串会根据给定的input编码解析，此后使用输出编码以字符串形式写入到原始文件。中间的编码通常是Unicode，但这依赖于特定的codecs。
	- 如果output没有给定，默认和input相同。
	- errors用于定义错误控制。默认为'strict'，在遭遇编码错误时会抛出ValueError。
- codecs.iterencode(iterable, encoding[, errors])
	- 使用一个incremental编码器来交互式编码由iterable提供的输入。该函数是个生成器。errors（和其他关键字参数一样）会通过incremental编码器传递。
	- 2.5新增。
- codecs.iterdecode(iterable,encoding[,errors])
	- 使用incremental解码器来交互式对iterable提供的输入解码。该函数是个生成器。errors（和其他关键字参数一样）是通过incremental解码器传递。
	- 2.5新增

该模块也提供下列的常量，它们对于读写平台依赖文件很有用：
- codecs.BOM
- codecs.BOM_BE
- codecs.BOM_LE
- codecs.BOM_UTF8
- codecs.BOM_UTF16
- codecs.BOM_UTF16_BE
- codecs.BOM_UTF16_LE
- codecs.BOM_UTF32
- codecs.BOM_UTF32_BE
- codecs.BOM_UTF32_LE

这些常量定义了各种使用UTF-16和UTF-32数据流的Unicode字节序列标记（BOM）编码，指示使用的流或文件的字节序并在UTF-8时作为一个Unicode标志。BOM_UTF16可以是BOM_UTF16_BE或BOM_UTF16_LE，这依赖于平台字节序，BOM是BOM_UTF16的别名，BOM_LE对BOM_UTF16_LE和BOM_BE对BOM_UTF16_BE也是如此。其他的用UTF-8和UTF-32编码表示BOM。

## Codec基础类
codecs模块定义了一个基础类集合，该集合定义了接口，且在python中写你自己的codecs时也更为易用。

每个codec都需要定义四个接口来使得它在python中可以作为一个codec使用：无国籍编码，无国籍解码，读入流和写入流。杜如流和写入流典型的复用了无国籍编解码来实现它的文件标准。

Codec类为无国籍编解码器定义了接口。

为了简单化标准化错误控制，encode()和decode()方法可通过提供errors字符串参数来实现不同的错误控制体制。下列字符串值被定义并再所有python codecs中实现：

| 值 | 意义 |
| --- | --- |
| 'strict' | 抛UnicodeError(或一个子类)；这是默认值 |
| 'ignore' | 忽略异常字符并继续 |
| 'replace' | 用合适的字符替代；python可能在内置Unicode codecs解码时使用官方的U+FFFD REPLACEMENT CHARACTER，在编码时使用'?' |
| 'xmlcharrefreplace' | 替换合适的XML字符（仅对编码） |
| 'backslashreplace' | 以反斜杠转义序列替代(仅对编码) |

register_error()扩展的值集合也是允许的。

### Codec对象
Codec类定义了这些方法和无国籍编解码器的函数接口：
- Codec.encode(input[, errors])
	- 编码input对象并返回一个元组（输出对象，length consumed）。在Unicode上下文中，当codecs没有约束使用Unicode时，编码转换一个Unicode对象成一个平坦字符串使用一个特别的字符设置编码（例如：cp1252或iso-8859-1）。
	- errors定义了应用的错误控制。默认为'strict'控制。
	- 该方法不会在Codec实例中存储状态。对codecs使用StreamWriter则需要保持状态这是为了让编码更为高效。
	- 编码器必须可以控制长度为0的输入并于此种状况下返回一个输出对象类型的空对象。 
- Codec.decode(input[, errors])
	- 解码对象input并返回一个元组（output对象，length consumed）。在Unicode上下文中，解码转换一个使用特别的字符集编码的平坦字符串成一个Unicode对象。
	- input必须是一个提供bf_getreadbuf缓冲区槽的对象。python字符串，缓冲区对象和内存映射文件都是可用的input。
	- errors定义了应用的错误控制。默认为'strict'控制。
	- 该方法不会再Codec实例中存储状态。对codecs使用StreamReader需要保持状态值，这是为了让解码更为高效。
	- 解码器必须可以控制0长度的input并于此种状况下返回一个输出对象类型的空对象。

IncrementalEncoder和IncrementalDecoder类为incremental编解码提供基本的接口。编解码输入在一次调用无国籍编解码函数后尚未完成，而是要多次调用incremental编解码器的encode()/decode()方法。incremental编解码器在方法调用期间保持了编解码过程的回溯。

合并的输出对encode()/decode()方法的调用和所有的单一输入合成一体是一样的，这里的输入会用无国籍编解码器编解码。

### IncrementalEncoder对象
2.5新增。

IncrementalEncoder类用于在多步中编码输入。它定义额了下列的方法，每个incremental encoder为了python codec注册的兼容性必须定义它们。

- class codecs.IncrementalEncoder([errors])
	- IncrementalEncoder实例的构造器。
	- 所有incremental编码器必须提供这个构造器接口。可以免费追加关键字参数，但是只有这里定义的这个会在python codec注册中使用。
	- IncrementalEncoder可以通过提供的errors参数实现不同的错误控制机制。这些参数是预定义的：
		- 'strict'：编码错误发生时抛出异常
		- 'replace'：替换畸形数据为合适的替换标记
		- 'ignore'：忽略畸形数据并在无更多提示下继续
		- 'xmlcharrefreplace'：以适当的XML字符参考替换
		- 'backslashreplace'：用反斜杠转义替换
	- errors参数会被指定成同名属性。分配该属性可以在IncrementalEncoder对象生命期间让切换不同的错误控制策略变为可能。
	- errors参数可设置的值也可以由register_error()扩展。
	- encode(object[, final])
		- 编码object（使用当前编码器的状态做解释）并返回编码的对象。如果这是encode()最后的调用，final必须为true（默认为false）。
	- reset()
		- 重置编码器到初始状态。

### IncrementalDecoder对象
IncrementalDecoder类用于在多步中解码输入。它定义了下面的方法，每个incremental解码器都必须定义它们，这是为了和python的codec注册相兼容。

- class codecs.IncrementalDecoder([errors])
	- IncrementalDecoder实例的构造器。
	- 所有的incremental解码器必须提供该构造器接口。可以额外增加关键字参数，但是只有这里定义的这个会被python codec注册所用。
	- IncrementalDecoder可以通过提供一个errors参数实现不同的错误控制机制。这些参数是预定义的：
		- 'strict'：编码错误发生时抛出异常
		- 'replace'：替换畸形数据为合适的替换标记
		- 'ignore'：忽略畸形数据并在无更多提示下继续
	- errors参数会被指定成同名属性。分配该属性可以在IncrementalDecoder对象生命期间让切换不同的错误控制策略变为可能。
	- errors参数可设置的值也可以由register_error()扩展。
	- decode(object[, final])
		- 解码对象（使用当前解码器状态做解释）并返回解码的对象。如果这是最后一次调用decode()则final必须置true（默认为false）。如果final是true解码器必须完全解码输入并刷新所有缓冲区。如果这不可能（例如，因为输入结尾有不完整的字节序列）则它必须初始化一个错误控制，就像在无国籍情境下（这应该抛出一个异常）。
	- reset()
		- 重置解码器到初始状态。

StreamWriter和StreamReader类提供了通用的功用接口，它们可以用来非常简单的实现新的编码子模块。查看encodings.utf_8获取一个使用它的例子。

### StreamWriter对象
StreamWriter类是Codec的子类，定义了下列的方法，每个写入流都必须定义它们，以便于兼容python codec注册。
- class codecs.StreamWriter(stream[, errors])
	- StreamWriter实例的构造器。
	- 所有的写入流都必须提供该构造器接口。可以额外增加关键字参数，但是只有这里定义的这个会被python codec注册使用。
	- stream必须是一个类文件对象，打开的是写二进制数据。
	- StreamWriter可以通过提供errors关键字参数来实现不同的错误控制机制。这些参数是预定义的：
		- 'strict'：编码错误发生时抛出异常
		- 'replace'：替换畸形数据为合适的替换标记
		- 'ignore'：忽略畸形数据并在无更多提示下继续
		- 'xmlcharrefreplace'：以适当的XML字符参考替换
		- 'backslashreplace'：用反斜杠转义替换
	- errors参数会被指定成同名属性。分配该属性可以在StreamWriter对象生命期间让切换不同的错误控制策略变为可能。
	- errors参数可设置的值也可以由register_error()扩展。
	- write(object)
		- 写编码的对象内容到stream
	- writelines(list)
		- 写连锁的字符串列表到stream（可能通过复用write()方法）
	- reset()
		- 使用保持态冲刷并重置codec缓冲区
		- 调用该方法应该确保输出的数据被放置在一个干净的状态，它允许附加新的新鲜数据而不用重新扫描整个流来恢复状态。

除了上面的方法，StreamWriter也必须继承underlying stream的所有其他方法和属性。

### StreamReader对象
StreamReader类是Codec的子类，定义了下列的方法，每个读入流都必须定义它们，以便于兼容python codec注册。

- class codecs.StreamReader(stream[, errors])
	- 所有读入流都必须提供该构造器接口。可以额外增加关键字参数，但是只有这里定义的一个会被python codec注册使用。
	- stream必须是类文件对象，打开读二进制数据。
	- StreamReader可以通过提供errors关键字参数来实现不同的错误控制机制。这些参数是预定义的：
		- 'strict'：编码错误发生时抛出异常
		- 'replace'：替换畸形数据为合适的替换标记
		- 'ignore'：忽略畸形数据并在无更多提示下继续
	- errors参数会被指定成同名属性。分配该属性可以在StreamReader对象生命期间让切换不同的错误控制策略变为可能。
	- errors参数可设置的值也可以由register_error()扩展。
	- read([size[, chars[, firstline]]])
		- 解码流为数据并返回结果对象
		- chars指示从stream读入的字符个数。read()永不会返回超过chars个字符，如果没有足够的字符则返回的个数会少于chars。
		- size指示为了解码从流中读入的近似最大字节数。解码器可以适当修改这一设置。默认为-1表示尽可能的读入和解码。size尝试去组织在一步之内解码巨大文件。
		- firstline指示返回首行就足够了，如果后面遇到解码错误或有更多行。
		- 该方法应该使用贪婪的读入策略，这意味着它会在允许情况下使用编码和给定的size定义尽可能读数据，例如，如果可选的编码结束或状态标记对流是可用的，这些也应该被读入。
		- 2.4更改：增加chars参数
		- 2.4.2更改：增加firstline参数
	- readline([size[, keepends]])
		- 从输入流读入一行并返回解码数据
		- size，如果给定，会向传递给read()方法一样生效。
		- 如果keepends是false，行结束会在返回的行中剥掉。
		- 2.4更改：增加keepends参数。
	- readlines([sizehint[, keepends]])
		- 读入输入流所有可读的行并返回一个行列表。
		- 行结束是通过codec的解码器方法实现的，如果keepends为true，则返回的列表中会保留。
		- sizehint如果给定，则和read()方法的size用法是一致的。
	
除了上面的方法，StreamReader必须从underlying stream继承其他方法和属性。

下面两个基本类是为了方便才引入的。它们不被codec注册所需要，但是实际中可能更为好用。

### StreamReaderWriter对象
StreamReaderWriter允许包装双读写模式的流。

该设计是为了可以使用lookup()返回的工厂函数来构造实例。

- class codecs.StreamReaderWriter(stream, Reader, Writer, errors)
	- 创建一个StreamReaderWriter实例。stream必须是一个类文件对象。Reader和Writer必须为提供StreamReader和StreamWriter实例的工厂函数或类。错误控制和此前的读入写入流是一致的。

StreamReaderWriter实例定义了组合的StreamReader和StreamWriter类的实例。它们从underlying stream继承所有其他方法和属性。

### StreamRecoder对象
StreamRecoder提供一个frontend-backend编码数据视图，又是在处理不同编码环境时很有用。

这一设计使得可以用lookup()返回的工厂函数构建实例。

- class codecs.StreamRecoder(stream, encode, decode, Reader, Writer, errors)
	- 创建一个StreamRecoder实例，实现两个转换：encode和decode工作在前端(read()的input和write()的output)当Reader和Writer工作在后端（读和写流）
	- 你可以使用这些对象来做直接透明的重编码，例如从Latin-1到UTF-8或反过来。
	- stream必须是类文件对象。
	- encode,decode必须附着于Codec实例。Reader，writer必须为分别提供StreamReader和StreamWriter实例对象的工厂函数或类。
	- encode和decode被前端翻译所需要，Reader和Writer为后端翻译所需要。中间的格式取决于两个codecs的集合，例如，Unicode codecs会使用Unicode作为中间的编码。
	- 错误控制和读入流和写入流定义相同。

StreamRecoder实例定义了StreamReader和StreamWriter类的接口组合。它们继承所有的underlying stream的其他方法和属性。

## 编码和Unicode
Unicode字符串以code points序列(是一个精确的Py_UNICODE数组)的形式存储在内部。依赖于python编译的方式（通过--enable-unicode=ucs2或--enable-unicode=ucs4，采用默认格式），Py_UNICODE要么是16-bit，要么是32-bit数据类型。一旦一个Unicode对象超出了CPU和内存界限，CPU字节序以及这些数组的字节如何存储就成为一个问题。转换一个unicode对象成一个字节序列的过程称为编码，以已知的字节序列重建unicode对象的过程叫解码。有很多不同的方法来转换（这些方法也叫编码）。最简单的方法就是映射code points 0-255到0x0-0xff。这意味着一个包含超过U+00ff的code points unicode对象不能用该方法编码（称为'latin-1'或'iso-8859-1'）。unicode.encode()会抛出一个UnicodeEncodeError，看起来像：`UnicodeEncodeError: 'latin-1' codec can't encode character u'\u1234' in position 3: ordinal not in range(256)`。

有另一组编码（charmap编码）选择一个所有的unicode code points的不同子集并决定code points到0x0-0xff的映射。想看这是如何工作的，可以简单地打开如encodings/cp1252.py（使用windows优先的编码方式）一探究竟。有一个256字节的字符串常量展示了哪个字符映射到了哪个字节值。

所有这些编码都只能编码unicode定义的1114112个code points到256。一个简单直接的方法是存储每个Unicode code point，存储其为4个连续的字节。有两种可能：按大端存储字节或小端存储。这两个编码分别称为UTF-32-BE和UTF-32-LE。它们的缺点是如果你使用了UTF-32-BE在小端机器上你需要总是交换编解码的字节。UTF-32规避了这一问题：字节永远会按自然地字节序存储。当这些字节通过一个不同的字节序的CPU读取，则字节序需要交换。为了能够探测UTF-16或UTF-32的字节序，有一个称为BOM("Byte Order Mark")的标记。这是个Unicode字节U+FFFE。该字符可以预置在每个UTF-16或UTF-32字节序的前面。这个字符的交换版本0xFFFE是一个合法的字符且不在Unicode文本中显示。所以在UTF-16或UTF-32字节序中当第一个字符为U+FFFE时字节序需要在解码时进行转换。不幸的是，字符U+FEFF有一个第二目的：ZERO WIDTH NO-BREAK SPACE，没有宽度且不允许字分割的字符。他可以用于给相关算法暗示。Unicode 4.0使用U+FEFF作为ZERO WIDTH NO-BREAK SPACE已经不被推荐（假定该角色使用U+2060（WORD JOINER））。虽然如此，Unicode软件仍然必须可以控制U+FEFF成两个角色：作为BOM他是个设备用于决定编码字节的存储布局，一旦解码字节序列成Unicode字符串则消失；作为一个ZERO WIDTH NO-BREAK SPACE，它是个普通的字符，如同其他那样被解码。

有另一个编码可以编码全部的Unicode字符范围：UTF-8。UTF-8是一个8-bit编码，意味着UTF-8字节序不存在问题。UTF-8字节序的每个字节包含两部分：标记位和payload位。标记为是一个0到4的序列，Unicode字符像这样编码（x作为payload位，当使用给定的Unicode字符作为连结时）：

| 范围 | 编码 |
| --- | ---- |
| U-00000000 ... U-0000007F | 0xxxxxxx |
| U-00000080 ... U-000007FF | 110xxxxx 10xxxxxx |
| U-00000800 ... U-0000FFFF | 1110xxxx 10xxxxxx 10xxxxxx |
| U-00010000 ... U-0010FFFF | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx |

Unicode字符最小的有意义位是右面最多x位。

UTF-8是一个8位编码，没有BOM需求且任何的U+FEFF字符在解码字符串中被认为是ZERO WIDTH NO-BREAK SPACE。

没有外界信息则不可能去可靠地决定哪一种编码会使用去编码一个Unicode字符串。每个charmap编码都可以解码任意随机字节序。然而，UTF-8是不可能的，UTF-8字节序有一个结构不允许任意的字节序列。想要增加UTF-8编码可被探测的可靠性，微软为它的Notepad程序引进了一个UTF-8变种（python 2.5称为"utf-8-sig"）：在任何Unicode字符写入文件前，一个UTF-8编码BOM（看起来像是字节序列：0xef,0xbb,0xbf）会被写入。一个被任意charmap编码的文件字节序不大可能以这样的序列开头（可能映射为:
	LATIN SMALL LETTER I WITH DIAERESIS 
	RIGHT-POINTING DOUBLE ANGLE QUOTATION MARK 
	INVERTED QUESTION MARK
在iso-8859-1中）。这增加了utf-8-sig编码被正确识别的可能性。因此这里对于生成的字节序列来说，BOM没有用于去决定使用的字节序，但是作为一个标记辅助猜测编码。编码utf-8-sig codec可以写0xef,0xbb,0xbf作为文件的3个首字节。解码utf-8-sig可以跳过这三个字节。在UTF-8中，使用BOM不被推荐且通常应该规避。

## 标准编码
python有一些内置编码，用C函数实现或用字典的映射表。下列表通过名字列出了编码，还有一些通用别名，以及可能用的语言。别名列表和语言列表都不详尽。注意到拼写替代仅在此种情形下不同或是使用一个连词替代下划字符也是有效的别名。因此，例如，'utf-8'在'utf_8'编码中是有效的别名。

字符集的大多数支持同样的语言。它们变化于独立的字符（例如，不管EURO SIGN是否支持），在字节到代码位置的赋值中。特别对European语言，下列变种典型存在：
- ISO 8859 代码集
- 微软代码页，典型的源于8859代码集，但是使用额外图形字符替代了控制字符
- IBM EBCDIC代码页
- IBM PC代码页，ASCII兼容

|Codec | Aliases | Languages |
| --- | --------- | -------- |
|ascii 	|646, us-ascii 	|English|
|big5 	|big5-tw, csbig5 |	Traditional Chinese|
|big5hkscs 	|big5-hkscs, hkscs 	|Traditional Chinese|
|cp037 	|IBM037, IBM039 |	English|
|cp424 	|EBCDIC-CP-HE, IBM424 |	Hebrew|
|cp437 	|437, IBM437 |	English|
|cp500 	|EBCDIC-CP-BE, EBCDIC-CP-CH, IBM500 |	Western Europe|
|cp720 	| - | 	Arabic|
|cp737 	| - | 	Greek|
|cp775 	|IBM775 	|Baltic languages|
|cp850 	|850, IBM850 	|Western Europe|
|cp852 	|852, IBM852 	|Central and Eastern Europe|
|cp855 	|855, IBM855 	|Bulgarian, Byelorussian, Macedonian, Russian, Serbian|
|cp856 	 |-| 	Hebrew|
|cp857 |	857, IBM857 |	Turkish|
|cp858 |	858, IBM858 	|Western Europe|
|cp860 	|860, IBM860 |	Portuguese|
|cp861 |	861, CP-IS, IBM861 |	Icelandic|
|cp862 |	862, IBM862 	|Hebrew|
|cp863 |	863, IBM863 |	Canadian|
|cp864 	|IBM864 	|Arabic|
|cp865 	|865, IBM865 	|Danish, Norwegian|
|cp866 	|866, IBM866 	|Russian|
|cp869 |	869, CP-GR, IBM869 	|Greek|
|cp874 |-|	  	Thai|
|cp875 	|-|  	Greek|
|cp932 |	932, ms932, mskanji, ms-kanji |	Japanese|
|cp949 |	949, ms949, uhc |	Korean|
|cp950 	|950, ms950 	|Traditional Chinese|
|cp1006 |-|	  	Urdu|
|cp1026 |	ibm1026 |	Turkish|
|cp1140 	|ibm1140 	|Western Europe|
|cp1250 	|windows-1250 	|Central and Eastern Europe|
|cp1251 	|windows-1251 	|Bulgarian, Byelorussian, Macedonian, Russian, Serbian|
|cp1252 	|windows-1252 	|Western Europe|
|cp1253 	|windows-1253 	|Greek|
|cp1254 |	windows-1254 	|Turkish|
|cp1255 	|windows-1255 	|Hebrew|
|cp1256 	|windows-1256 |	Arabic|
|cp1257 |	windows-1257 |	Baltic languages|
|cp1258 	|windows-1258 |	Vietnamese|
|euc_jp 	|eucjp, ujis, u-jis |	Japanese|
|euc_jis_2004 	|jisx0213, eucjis2004 |	Japanese|
|euc_jisx0213 |	eucjisx0213 |	Japanese|
|euc_kr 	|euckr, korean, ksc5601, ks_c-5601, ks_c-5601-1987, ksx1001, ks_x-1001 |	Korean|
|gb2312 	|chinese, csiso58gb231280, euc- cn, euccn, eucgb2312-cn, gb2312-1980, gb2312-80, iso- ir-58 	|Simplified Chinese|
|gbk 	|936, cp936, ms936 	|Unified Chinese|
|gb18030 	|gb18030-2000 |	Unified Chinese|
|hz |	hzgb, hz-gb, hz-gb-2312 |	Simplified Chinese|
|iso2022_jp 	|csiso2022jp, iso2022jp, iso-2022-jp |	Japanese|
|iso2022_jp_1 	|iso2022jp-1, iso-2022-jp-1 |	Japanese|
|iso2022_jp_2 	|iso2022jp-2, iso-2022-jp-2 	|Japanese, Korean, Simplified Chinese, Western Europe, Greek|
|iso2022_jp_2004 |	iso2022jp-2004, iso-2022-jp-2004 |	Japanese|
|iso2022_jp_3 	|iso2022jp-3, iso-2022-jp-3| 	Japanese|
|iso2022_jp_ext |	iso2022jp-ext, iso-2022-jp-ext |	Japanese|
|iso2022_kr 	|csiso2022kr, iso2022kr, iso-2022-kr |	Korean|
|latin_1 	|iso-8859-1, iso8859-1, 8859, cp819, latin, latin1, L1 	|West Europe|
|iso8859_2 |	iso-8859-2, latin2, L2 |	Central and Eastern Europe|
|iso8859_3 |	iso-8859-3, latin3, L3| 	Esperanto, Maltese|
|iso8859_4 |	iso-8859-4, latin4, L4 	|Baltic languages|
|iso8859_5 	|iso-8859-5, cyrillic 	|Bulgarian, Byelorussian, Macedonian, Russian, Serbian|
|iso8859_6 	|iso-8859-6, arabic| 	Arabic|
|iso8859_7 	|iso-8859-7, greek, greek8 |	Greek|
|iso8859_8 	|iso-8859-8, hebrew| 	Hebrew|
|iso8859_9 	|iso-8859-9, latin5, L5 	|Turkish|
|iso8859_10 |	iso-8859-10, latin6, L6 |	Nordic languages|
|iso8859_11 	|iso-8859-11, thai 	|Thai languages|
|iso8859_13 |	iso-8859-13, latin7, L7 |	Baltic languages|
|iso8859_14 |	iso-8859-14, latin8, L8 |	Celtic languages|
|iso8859_15 |	iso-8859-15, latin9, L9| 	Western Europe|
|iso8859_16 |	iso-8859-16, latin10, L10 |	South-Eastern Europe|
|johab 	|cp1361, ms1361 	|Korean|
|koi8_r 	|-|  	Russian|
|koi8_u 	|-|  	Ukrainian|
|mac_cyrillic 	|maccyrillic 	|Bulgarian, Byelorussian, Macedonian, Russian, Serbian|
|mac_greek|	macgreek |	Greek|
|mac_iceland |	maciceland |	Icelandic|
|mac_latin2 |	maclatin2, maccentraleurope |	Central and Eastern Europe|
|mac_roman 	|macroman 	|Western Europe|
|mac_turkish |	macturkish 	|Turkish|
|ptcp154 	|csptcp154, pt154, cp154, cyrillic-asian |	Kazakh|
|shift_jis 	|csshiftjis, shiftjis, sjis, s_jis| 	Japanese|
|shift_jis_2004 |	shiftjis2004, sjis_2004, sjis2004 |	Japanese|
|shift_jisx0213 |	shiftjisx0213, sjisx0213, s_jisx0213 |	Japanese|
|utf_32 |	U32, utf32 |	all languages|
|utf_32_be 	|UTF-32BE |	all languages|
|utf_32_le |	UTF-32LE |	all languages|
|utf_16 |	U16, utf16 |	all languages|
|utf_16_be 	|UTF-16BE 	|all languages (BMP only)|
|utf_16_le |	UTF-16LE 	|all languages (BMP only)|
|utf_7 	|U7, unicode-1-1-utf-7 |	all languages|
|utf_8 	|U8, UTF, utf8 |	all languages|
|utf_8_sig 	| - |	all languages|

## python特殊编码
python有一些特殊的预定义编码，编码名字离开了python则毫无意义。它们在下表列出基于期望的输入和输出类型（注意到文本编码是codecs最普通的用法，codec基础设置的根本提供了任意数据转换而不是仅仅是文本编码）。对非对称编码来说，规定的目的描述了编码方向。

下列编码提供了unicode到str的编码方法以及str到unicode的解码方法[1]。和Unicode文本编码相似[2]。

| 编码 | 别名 | 目的 |
| --- | ---------- | --------- |
|idna | - |	Implements RFC 3490, see also encodings.idna|
|mbcs 	|dbcs 	|Windows only: Encode operand according to the ANSI codepage (CP_ACP)|
|palmos 	|-|  	Encoding of PalmOS 3.5|
|punycode 	|-|  	Implements RFC 3492|
|raw_unicode_escape 	 |-| 	Produce a string that is suitable as raw Unicode literal in Python source code|
|rot_13 |	rot13 |	Returns the Caesar-cypher encryption of the operand|
|undefined 	  |-|	Raise an exception for all conversions. Can be used as the system encoding if no automatic coercion between byte and Unicode strings is desired.|
|unicode_escape 	|-|  	Produce a string that is suitable as Unicode literal in Python source code|
|unicode_internal |-|	  	Return the internal representation of the operand|

2.3新增：idna和punycode编码。

下列编码提供了str到str的编解码[2]。

|编码| 	别名|	目的 | 	编解码器 |
| -- | ---- | ------ | -------- |
|base64_codec 	|base64, base-64 	|Convert operand to multiline MIME base64 (the result always includes a trailing '\n') |	base64.encodestring(), base64.decodestring()|
|bz2_codec |	bz2 	|Compress the operand using bz2 |	bz2.compress(), bz2.decompress()|
|hex_codec 	|hex 	|Convert operand to hexadecimal representation, with two digits per byte 	|binascii.b2a_hex(), binascii.a2b_hex()|
|quopri_codec |	quopri, quoted-printable, quotedprintable 	|Convert operand to MIME quoted printable 	|quopri.encode() with quotetabs=True, quopri.decode()|
|string_escape 	|-|Produce a string that is suitable as string literal in Python source code |-|	 
|uu_codec 	|uu |	Convert the operand using uuencode| 	uu.encode(), uu.decode()|
|zlib_codec |	zip, zlib 	|Compress the operand using gzip |	zlib.compress(), zlib.decompress()|

> [1]str对象也可以替代unicode对象作为输入被接受。它们隐式的通过使用默认编码方法解码自身转换成unicode。如果转换失败，则它指引编码操作抛出UnicodeDecodeError。
> [2] (1,2)Unicode对象也可以替代str对象作为输入被接受。它们隐式的通过使用默认编码方法编码自身转换为str。如果转换失败，则它指引解码操作抛出UnicodeEncodeError。

## encodings.idna - 国际化应用域名
2.3新增。

该模块实现了RFC 3490（国际化应用域名）和RFC 3492（Nameprep: A Stringprep Profile for Internationalized Domain Names (IDN)）。它在punycode编码和stringprep上构筑。

这些RFC一起定义了一种在域名上支持非ASCII字符的标准。域名包含非ASCII字符（比如www.Alliancefrançaise.nu）会被转成ASCII兼容编码（ACE, such as www.xn--alliancefranaise-npb.nu）。域名的ACE格式是会使用在标准不允许的任意字符处，比如DNS查询，HTTP Host字段等等。转换会在应用中携带。如果对用户可见：应用应该在线路上透明的转换Unicode域标签到IDNA，在显示给用户前转换回ACE标签到Unicode。

python支持这一转换，有多个方法：idna编码执行Unicode和ACE的转换，分割输入字符串成基于RFC3490 3.1节定义的分割字符的标签，按需转换每个标签成ACE，相反地基于`.`分隔符分割输入字符串成标签并转换任意的ACE标签成unicode。在这之上，socket模块透明的转换Unicode host名称到ACE，以便于应用在把他们传递给socket模块时不需要关心host名字自身的转换。基于此，模块拥有host名称作为函数参数，如httplib和ftplib，接受Unicode host名称（httplib也透明的发送一个IDNA hostname在Host字段，如果他根本不能成功发送该字段）。

当接收到host名字时（比如在反向名称查询），没有自动的到Unicode的转换会执行：应用渴望表现这些host名称给用户，它们应该被解码成Unicode。

模块encodings.idna也实现了nameprep过程，执行确定的host名称的标准化，实现大小写不敏感的国际化域名，以及统一相似的字符。nameprep函数如果值得信赖则可以被直接使用。

- encodings.idna.nameprep(label)
	- 返回label的nameprepped版本。这一实现承担当前的查询字符串，因此AllowUnassigned是true。
- encodings.idna.ToASCII(label)
	- 转换label到ASCII，以RFC 3490的定义为准。UseSTD3ASCIIRules假定为false。
- encodings.idna.ToUnicode(label)
	- 转换label到Unicode，以RFC 3490的定义为准。

## encodings.utf_8_sig - UTF-8携带BOM标记的编码
2.5新增。

该模块实现了一个UTF-8编码的变体：在编码带BOM的UTF-8时，会预先考虑UTF-8编码字节。对有状态的编码器仅处理一次（第一次写到字节流）。在解码一个可选的带BOM的UTF-8编码时，起始的数据会被忽略。