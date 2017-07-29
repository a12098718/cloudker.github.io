---
published: true
layout: post
title: python标准库-textwrap
category: python
tags: 
  - python
time: 2017.05.24 21:18:00
excerpt: 余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

---

余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。
近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

<!--more-->

# python标准库-textwrap-文本的包裹与填充
2.3新增。
源码：Lib/textwrap.py

----------
textwrap模块提供了两个方便的函数，wrap()和fill()，以及TextWrapper、该类处理所有工作，和一个通用函数dedent()。如果你仅仅包装或填充一到两个文本串，方便的函数应该足够好；否则，你应该使用TextWrapper的实例来做得更为高效。

- textwrap.wrap(text[, width[, ...]])
	- 包裹文本中的单个段落使得每行不超过width个字符。返回一个输出行的列表，不包含最后的新行。
	- 可选的关键字参数和TextWrapper实例属性一致，下面有描述。width默认为70。
	- 查看TextWrapper.wrap()方法了解额外的关于wrap()如何表现的详细内容。

- textwrap.fill(text[, width[, ...]])
	- 包装文本的单一段落，返回包含包装段落的单一字符串。fill()相当于`"\n".join(wrap(text, ...))`
	- 特别的，fill()接受的关键字参数和wrap()意义相同。

wrap()和fill()都通过创建一个TextWrapper实例并调用其单一方法来工作。该实例没有被重用，所以对应用来说若要wrap/fill多个文本字符串，创建自己的TextWrapper对象更为高效。

文本宁愿被空格和连字词中连字符后半部分包装，只有这样长字符才会按需要破坏掉，除非TextWrapper.break_long_words已置为false。

- textwrap.dedent(text)
	- 移除text每一行起始的空白字符。
	- 这可以用来做三引号字符串的对齐，依然会保留源代码的对齐格式。
	- 注意到tabs和空格都被视为空白，但是他们永不相等："  hello"和"\thello"被认为没有通用起始空白。（python2.5新特性，旧版本会错误的在搜索通用起始空白前扩充tabs）
	- 例如：
```python
def test():
    # end first line with \ to avoid the empty line!
    s = '''\
    hello
      world
    '''
    print repr(s)          # prints '    hello\n      world\n    '
    print repr(dedent(s))  # prints 'hello\n  world\n'
```

- class textwrap.TextWrapper(...)
	- TextWrapper构造器接受若干数目可选的关键字参数。每个参数和一个实例属性相对应，所以对例子：`wrapper = TextWrapper(initial_indent="* ")`来说，等价于`wrapper = TextWrapper();wrapper.initial_indent = "* "`。
	- 你可以多次重用相同的TextWrapper对象，也可以通过在使用时直接指定实例属性的方式修改它的任意选项。
	- TextWrapper实例属性（构造器的关键字参数）如下：
	- width
		- (默认为70)包裹行的最大长度。只要输入的文本中没有超过width的单词，TextWrapper保证输出行也没有超过width长度的行。
	- expand_tabs
		- （默认为True）如果为真，则所有tab字符会被扩展为空格，使用expandtabs()方法。
	- replace_whitespace()
		- (默认为True)如果为真，tab扩展后文本包裹前，wrap()方法会替换每个空白字符成一个单一空格。会被替换的空白字符：tab,newline,vertical tab, carriage return('\t\n\v\f\r')。
		- 如果expand_tabs为假单replace_whitespace为真，则每个tab字符都会被单一空格替换，和tab替换并不相同。
		- 如果replace_whitespace为假，新行会出现在每个行的中间并引起奇怪的输出。因为这个缘由，文本应该被分割成段落(使用str.splitlines()或相似的)，这样就可以独立包裹。
	- drop_whitespace
		- （默认为True）如果为真，每一行起始和末尾的空白（wrapping之后，indenting之前）会被丢弃。然而段落前的空白，如果后非空白跟随则不会被丢弃。如果整行都是空白，则整行丢弃。
		- 2.6新增：空白在早期版本永远被丢弃。
	- initial_indent
		- (default: '') String that will be prepended to the first line of wrapped output. Counts towards the length of the first line. The empty string is not indented.
	- subsequent_indent
		- (default: '') String that will be prepended to all lines of wrapped output except the first. Counts towards the length of each line except the first.
	- fix_sentence_endings
		- （默认为False）如为True，TextWrapper尝试探测句尾并确保句子被严格的两个空格分割。这通常在使用monospaced字体的文本时被信赖。然而，句子探测算法是优缺点的：它假定句子以小写字母跟随'.','!'或'?'结尾，可能跟随一个'"'或"'"，跟随一个空格。一个问题是算法不可能去探测这两个例子的"Dr."和"Spot."的区别：`[...] Dr. Frankenstein's monster [...]`和`[...] See Spot. See Spot run [...]`。
		- fix_sentence_endings默认为false。
		- Since the sentence detection algorithm relies on string.lowercase for the definition of “lowercase letter,” and a convention of using two spaces after a period to separate sentences on the same line, it is specific to English-language texts.
	- break_long_words
		- （默认为True）如果为True，单词长于width的部分会被破坏以便于确保没有行会超过width长度。如果是假，则长单词不会被破坏，一些行可能会比width长。（长单词会单独输出一行，为了最小化width被超出的个数。）
	- break_on_hyphens
		- （默认为True）如果为True，包装遇到空白和组合词的连字符后半部分时，会被定制成英语。如果为假，则仅有空白会被认为是潜在的分割行的好地方，但是当你想要真正的insecable单词时，你需要将其置False。旧版本的默认行为是允许分割组合词。
		- 2.6新增。

	- TextWrapper还提供了两个公共方法，相当于模块级别的便捷函数：
	- wrap(text)
		- 包裹单一段落，每行长度都不会超过width。所有的包装选项都来自TextWrapper实例的实例属性。返回一个输出行列表，没有最后的新行。如果包裹输出没有内容，则返回列表是空的。
	- fill(text)
		- 包裹文本的单一段落，返回包含包裹段落的单个字符串。