---
published: true
layout: post
title: python标准库-difflib
category: python
tags: 
  - python
time: 2017.06.29 17:28:00
excerpt: 余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

---

余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。
近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

<!--more-->

# python标准库-difflib-计算δ的助手
2.1新增。

该模块提供用于比对序列的类和函数。他可以用于诸如比对文件，产生多种格式的差异信息，包括HTML和文本以及统一的差异格式。对于比对目录和文件，也可以看看filecmp模块。

- class difflib.SequenceMatcher
	- 这是个灵活的类，用于比对任意类型的序列对，只要序列元素是可哈希的。基础算法早先于后来20世纪80年代Ratcliff和Obershelp发布的命名夸张的“格式塔模式匹配”算法，也稍有想象力一些。它的想法在于找到匹配的最长连续子序列，其不包含“垃圾”元素（Ratcliff和Obershelp算法没有定位垃圾）。同样的想法被递归的应用于每个序列片段到匹配序列的左和右侧。它不会产生最小的编辑序列，而是趋向于产生对人类看起来更为友好的匹配。
	- 时间：基础Ratcliff-Obershelp算法最坏时间是立方时间复杂度，期望情况则为平方时间复杂度。SequenceMatcher在最坏情况是平方时间复杂度且期望情况下行为依赖于一种复杂的取决于序列通常拥有多少个元素的方法，最好时间是线性的。
	- 自动垃圾启发式：SequenceMatcher支持一种启发式以自动的对待确定的序列项为垃圾。启发式计算每个独立的元素在序列中出现的次数。如果序列的某个元素副本（除了第一个）超过1%，且序列至少有200个元素，这一元素就被标记为“流行的”且在序列匹配时会被视作垃圾。在创建SequenceMatcher时，这一启发式可以通过设置autojunk参数为False来关闭。
	- 2.7.1新增：autojunk参数。
- class difflib.Differ
	- 用于比较文本行序列的类，产生人类可读的差别或δ。Differ使用SequenceMatcher来比对行序列，也比对包含相似（接近匹配）行的字符序列。
	- Differ的每行差别以两个字母的标志码起始：
	- 以'?'起始的行尝试引导内部行差异，且均不在任一输入序列。如果序列包含tab字符则这些行可能有些混乱。

| Code | 意义 |
| ---- | --- |
| '-' | 序列1的特殊行 |
| '+' | 序列2的特殊行 |
| '  ' | 两个序列通用行 |
| '?' | 任一输入序列都没有的行 |

- class difflib.HtmlDiff
	- 该类可以用于产生HTML表格（或一个完整的html文件包含表格）来显示边对边、行对行用于内部行修改高亮的文本比较。表格可以通过完整或上下文差异模式来生成。
	- 该类的构造器：
	- `__init__(tabsize=8, wrapcolumn=None, linejunk=None, charjunk=IS_CHARACTER_JUNK)`
		- 初始化HtmlDiff实例。
		- tabsize是可选的关键字参数，用于指定tab空白字符数，默认为8。
		- wrapcolumn是一个可选的关键字用于指定那些截断和包装的行的列数，默认是None也就是说行没有包装。
		- linejunk和charjunk都是可选关键字参数，传递给ndiff()（HtmlDiff用于产生边对边HTML差异）。查看ndiff()文档查找参数默认值和描述。
	- 下列方法是公用的：
	- `make_file(fromlines, tolines [, fromdesc][, todesc][, context][, numlines])`
		- 比较fromlines和tolines（字符串序列）并且返回返回一个字符串，它是一个完整的包含一个显示行对比差异高亮的table的HTML文件。
		- fromdesc和todesc是可选参数用于指定from/to文件列头部字符串（默认都是空串）。
		- context和numlines都是可选的关键字参数。当上下文差异需要被显示时设置context为True，否则设置False则显示全部文件。numlines默认为5。当context是True，numlines控制差异高亮前后的文本行数。context为False时，当使用"next"超链接时numlines控制下一个差异高亮的引导文本行数（设0会引起"next"超链接放置下一个差异高亮在浏览的头部而不会有引导文字）。
	- `make_table(fromlines, tolines, [, fromdesc][, todesc][, context][, numlines])`
		- 比较fromlines和tolines(字符串列表)并且返回一个完整的显示内部行差异高亮的HTML table字符串。
		- 参数和make_file()一致。
	- Tools/scripts/diff.py是一个该类的从头至尾的命令行，包含了使用的很好的用例。
	- 2.4新增。
- `difflib.context_diff(a, b[, fromfile][, tofile][, fromfiledate][, tofiledate][, n][, lineterm])`
	- 比对a和b（字符串列表）；返回文本差异格式的δ（一种通用符表示差异行）。
	- 文本差异是一种简洁的方式展示那些仅仅不一致的行。差异以前/后的风格展示。文本行数默认为3行。
	- 默认情况下，差异控制行（那些用`***`或`---`）会以一个尾随的新行创建。这很有用因为输入是从file.readlines()差异结果中创建的，这对于使用file.writelines()很合适因为输入和输出都有尾随的新行。
	- 对输入来说如果想没有尾随的新行，则设置lineterm参数为""，以便于输出可以有统一的新行空闲。
	- 文本差异格式通常有一个头部，标志文件名和修改次数。任何或所有的这些都可以通过fromfile,tofile,fromfiledate,tofiledate来指定。修改次数通常用ISO 8601格式表示，字符串默认是空白。
	- 2.3新增。

```python
>>> s1 = ['bacon\n', 'eggs\n', 'ham\n', 'guido\n']
>>> s2 = ['python\n', 'eggy\n', 'hamster\n', 'guido\n']
>>> for line in context_diff(s1, s2, fromfile='before.py', tofile='after.py'):
...     sys.stdout.write(line)  
*** before.py
--- after.py
***************
*** 1,4 ****
! bacon
! eggs
! ham
  guido
--- 1,4 ----
! python
! eggy
! hamster
  guido
```

- `difflib.get_close_matches(word, possibilities[, n][, cutoff])`
	- 返回一个最优的“足够好”匹配列表。word是一个可靠的近似匹配序列（通常是一个字符串），possibilities是序列列表表示哪些去匹配字（一个字符串列表）。
	- 可选参数n(默认为3)是近似匹配所能返回的最大的数，n必须大于0。
	- 可选参数cutoff(默认为0.6)是[0,1]之间的浮点数。Possibilities that don’t score at least that similar to word are ignored.
	- The best (no more than n) matches among the possibilities are returned in a list, sorted by similarity score, most similar first.

```python
>>> get_close_matches('appel', ['ape', 'apple', 'peach', 'puppy'])
['apple', 'ape']
>>> import keyword
>>> get_close_matches('wheel', keyword.kwlist)
['while']
>>> get_close_matches('apple', keyword.kwlist)
[]
>>> get_close_matches('accept', keyword.kwlist)
['except']
```

- `difflib.ndiff(a,b[, linejunk][, charjunk])`
	- 比较a和b（字符串列表），返回一个Differ风格δ（生成的区别行通用符）。
	- 可选参数linejunk和charjunk供过滤函数（或None）使用。
	- linejunk:接收单字符串参数的函数，如果字符串是垃圾则返回true，否则返回false。默认为None，从2.3版本开始。在这之前，默认为module-level函数IS_LINE_JUNK()，过滤没有可视化字符的行，除了至多一个'#'的情形。python 2.3，潜在的SequenceMatcher类会处理一个动态行分析，判断哪些行过于频繁出现而视为噪音构成，这通常效果优于2.3前的版本。
	- charjunk:接收一个字符（长度为1的字符串）的函数，如果字符是垃圾则返回，否则返回false。默认是module-level函数IS_CHARACTER_JUNK(),过滤空白字符（空白或tab；注意：包含新行于此是个坏主意）。
	- Tools/scripts/ndiff.py是一个命令行，彻头彻尾用到了该函数。

```python
>>> diff = ndiff('one\ntwo\nthree\n'.splitlines(1),
...              'ore\ntree\nemu\n'.splitlines(1))
>>> print ''.join(diff),
- one
?  ^
+ ore
?  ^
- two
- three
?  -
+ tree
+ emu
```

- difflib.restore(sequence,which)
	- 返回两个序列的差异。
	- 给定一个由Differ.compare()或ndiff()产生的序列，提取源于1或2文件（参数which）的行，剥离行前缀。
	- 例如：

```python
>>> diff = ndiff('one\ntwo\nthree\n'.splitlines(1),
...              'ore\ntree\nemu\n'.splitlines(1))
>>> diff = list(diff) # materialize the generated delta into a list
>>> print ''.join(restore(diff, 1)),
one
two
three
>>> print ''.join(restore(diff, 2)),
ore
tree
emu
```

- `difflib.unified_diff(a,b[,fromfile][,tofile][,fromfiledate][,tofiledate][,n][,lineterm])`
	- 比较a和b（字符串列表）；返回δ（通用行区别符），以统一diff格式表示。
	- 统一diff是一种紧凑的方式，用于显示那些有区别的行并附加一些前后上下文。区别显示成内联风格（取代before/after块的分离）。上下文行数由n设置，默认为3。
	- 默认情形，diff控制行数（那些---,+++或@@）由尾随的新行创建。这很有用因为输入是从file.readlines()差异结果中创建的，这对于使用file.writelines()很合适因为输入和输出都有尾随的新行。
	- 对输入来说如果想没有尾随的新行，则设置lineterm参数为""，以便于输出可以有统一的新行空闲。
	- 文本差异格式通常有一个头部，标志文件名和修改次数。任何或所有的这些都可以通过fromfile,tofile,fromfiledate,tofiledate来指定。修改次数通常用ISO 8601格式表示，字符串默认是空白。
	- 2.3新增

```python
>>> s1 = ['bacon\n', 'eggs\n', 'ham\n', 'guido\n']
>>> s2 = ['python\n', 'eggy\n', 'hamster\n', 'guido\n']
>>> for line in unified_diff(s1, s2, fromfile='before.py', tofile='after.py'):
...     sys.stdout.write(line)   
--- before.py
+++ after.py
@@ -1,4 +1,4 @@
-bacon
-eggs
-ham
+python
+eggy
+hamster
 guido
```

- difflib.IS_LINE_JUNK(line)
	- 对可忽略行返回true。line行是可忽略的，如果line是一个空白或包含单个字符'#'，否则就是不可忽略的。对2.3之前的ndiff()来说，这是其参数linejunk的默认值。
- difflib.IS_CHARACTER_JUNK(ch)
	- 对可忽略字符返回true。如果ch是空格或tab则ch可忽略，否则就是不可忽略的。作为ndiff()的默认charjunk参数。

## SequenceMatcher对象
SequenceMatcher类拥有一个构造器：
- class difflib.SequenceMatcher(isjunk=None,a='',b='',autojunk=True)
	- 可选参数isjunk必须为None(默认)或是一个单个参数的函数，携带一个序列元素返回true当且仅当元素是垃圾且应该被忽略。传递None给isjunk等价于传递lambda x:0；换句话说，没有元素会被忽略。例如，传递：`lambda x: x in " \t"`，如果你比较字符序列行，并不想同步空白和tab。
	- 可选参数a和b是比较的序列；默认都为空串。这两个参数都必须是可哈希的。
	- 可选参数autojunk可以用来关闭自动垃圾启发式。
	- 2.7.1新增：autojunk参数。
	- SequenceMatcher对象拥有下列方法。
	- set_seqs(a,b)
		- 设置两个序列用于比较
	- SequenceMatcher计算并缓存第二个序列的简要信息，所以如果你想要比较第一个序列很多次的话，使用set_seq2()来设置通用地使用序列一次并反复调用set_seq1()，每次对其他序列处理一次。
	- set_seq1(a)
		- 设置用于比较的第一个序列。第二个序列不会改变。
	- set_seq2(b)
		- 设置用于比较的第二个序列。第一个序列不会改变。
	- find_longest_match(alo, ahi, blo, bhi)
		- 查找a[alo:ahi]和b[blo:bhi]中的最长匹配块。
		- 如果isjunk被忽略或为None，find_longest_match()会返回(i,j,k)就像a[i:i+k]等同于b[j:j+k]，此时`alo <= i <= i+k <= ahi`且`blo <= j <= j+k <= bhi`。对所有的满足此情形的(i', j', k')，则 k >= k', i <= i'且如果 i == i', 则j <= j'也满足。换句话说，对所有匹配块， return one that starts earliest in a, and of all those maximal matching blocks that start earliest in a, return the one that starts earliest in b。
```python
>>> s = SequenceMatcher(None, " abcd", "abcd abcd")
>>> s.find_longest_match(0, 5, 0, 9)
Match(a=0, b=4, size=5)
```
	- 如果提供了isjunk，第一个最长的匹配块会像上面那样被决定，但有额外的约束那就是块中不能出现垃圾元素。然后该块会尽可能的扩展通过匹配两边的垃圾元素。所以结果块永远不会匹配到垃圾除非完全相同的垃圾刚好在一次有趣的匹配中是毗连的。
	- 这里有和之前一样相同的例子，但是会将空白当成垃圾。这直接阻止了' abcd'的匹配当其在第二个序列的尾处。取而代之的是只有'abcd'会匹配并且尽可能匹配第二个序列中'abcd'后剩余的部分。
```python
>>> s = SequenceMatcher(lambda x: x==" ", " abcd", "abcd abcd")
>>> s.find_longest_match(0, 5, 0, 9)
Match(a=1, b=0, size=4)
```
	- 如果没有块匹配，则返回(alo,blo,0)。
	- 2.6更改：该方法返回一个命名元组Match(a, b, size)。
- get_matching_blocks()
	- 返回描述匹配序列的“三元”列表。每个元都是(i,j,n)格式，表示a[i:i+n] == b[j:j+n]。每个元的i和j单调递增。
	- 最后一个元是哑的，拥有值(len(a),len(b),0)。它是n==0的唯一元。如果(i, j, n) 和 (i', j', n')在列表中是毗邻的元，且第二个在列表中不是最后一个元，那么i+n != i'或j+n != j'；换句话说，毗连的元永远描述非毗连的等同块。
	- 2.5更改：保证了毗连的元永远描述非毗连的元这一实现。
```python
>>> s = SequenceMatcher(None, "abxcd", "abcd")
>>> s.get_matching_blocks()
[Match(a=0, b=0, size=2), Match(a=3, b=2, size=2), Match(a=5, b=4, size=0)]
```
- get_opcodes()
	- 返回一个“五元”列表描述如何转换a到b。每个元都是(tag,i1,i2,j1,j2)的格式。第一个元i1==j1==0，剩下的元i1等同于i2，j1也等同于j2。
	- tag值是字符串，有这些意义：

| 值 | 意义 |
| -- | ---- |
| 'replace' | a[i1:i2]应被b[j1:j2]取代 |
| 'delete' | a[i1:i2]应被删除。注意到这里j1==j2 |
| 'insert' | b[j1:j2]应插入到a[i1:i1]。注意到这里i1==i2 |
| 'equal' | a[i1:i2]==b[j1:j2]（子序列是等价的） |
举个例子：
```python
>>> a = "qabxcd"
>>> b = "abycdf"
>>> s = SequenceMatcher(None, a, b)
>>> for tag, i1, i2, j1, j2 in s.get_opcodes():
...    print ("%7s a[%d:%d] (%s) b[%d:%d] (%s)" %
...           (tag, i1, i2, a[i1:i2], j1, j2, b[j1:j2]))
 delete a[0:1] (q) b[0:0] ()
  equal a[1:3] (ab) b[0:2] (ab)
replace a[3:4] (x) b[2:3] (y)
  equal a[4:6] (cd) b[3:5] (cd)
 insert a[6:6] () b[5:6] (f)
```
- get_grouped_opcodes([n])
	- 返回一个n行上下文分组的生成器。
	- 始于get_opcodes()返回的分组，该方法分离出更小的改变集群并且消除了没有改变的介入范围。
	- 分组的返回形式与get_opcodes()一致。
	- 2.3新增
- ratio()
	- 返回一个序列的相似性测量值，是[0,1]的浮点数。
	- T是两个序列的所有元素个数，M是匹配的个数，也就是2.0*M/T。注意到如果序列完全等同，则为1.0，完全无匹配则为0.0。
	- 如果get_matching_blocks()或get_opcodes()没有被调用过，则计算代价是昂贵的。你可能更希望尝试quick_ratio()或real_quick_ratio()先去拿一个上限值。
- quick_ratio()
	- 返回较之ratio()相对快速的一个上限值。
- real_quick_ratio()
	- 返回一个较之ratio()更为快速的一个上限值。

三个方法返回的所有字符匹配率会因不同等级的近似法而给出不同的结果，尽管quick_ratio()和real_quick_ratio()永远大于等于ratio()给出的结果：
```python
>>> s = SequenceMatcher(None, "abcd", "bcde")
>>> s.ratio()
0.75
>>> s.quick_ratio()
0.75
>>> s.real_quick_ratio()
1.0
```

## SequenceMatcher例子
该例比较两个字符串，考虑空格为垃圾。
```python
>>> s = SequenceMatcher(lambda x: x == " ",
...                     "private Thread currentThread;",
...                     "private volatile Thread currentThread;")
```

ratio()返回一个[0,1]的浮点数，测量两个序列的相似度。以一个笨拙的规则而论，超过0.6的值表明两个序列相近。
```python
>>> print round(s.ratio(), 3)
0.866
```

如果你仅仅对序列的匹配位置感兴趣，则get_matching_blocks()可用：
```python
>>> for block in s.get_matching_blocks():
...     print "a[%d] and b[%d] match for %d elements" % block
a[0] and b[0] match for 8 elements
a[8] and b[17] match for 21 elements
a[29] and b[38] match for 0 elements
```

注意到最后一个元组是哑的。

如果你想知道如何将第一个序列转为第二个，使用get_opcodes():
```python
>>> for opcode in s.get_opcodes():
...     print "%6s a[%d:%d] b[%d:%d]" % opcode
 equal a[0:8] b[0:8]
insert a[8:8] b[8:17]
 equal a[8:29] b[17:38]
```

## Differ对象
注意到Differ生成的差异没有声明为最小的差异。相反的，最小的差异经常是直觉计算的，因为它们可能在任何地方同步，有时会意外的匹配相距100页的地方。限制同步点为临近匹配保留了一些局部概念，偶然情况会产生一个较大的差异。

Differ类有一个构造器：
- class difflib.Differ([linejunk[, charjunk]])
	- linejunk和charjunk是为过滤函数（或None）准备的：
	- linejunk: 接收单个字符串参数，如果字符串是垃圾则返回true。默认为None，意味着没有行会被认为是垃圾。
	- charjunk: 接收单个字符参数（长度为1的字符串），如果字符是垃圾则返回true。默认为None，意味着没有字符被认为是垃圾。
	- Differ对象通过一个单独的方法使用（生成差异）:
	- compare(a,b)
		- 对比两个多行的序列，生成差异。
		- 每个序列必须包含独立的以新行结尾的单一行字符串。这些序列可以通过类文件对象的readlines()方法获取。生成的差异也包含新行结尾的字符串，类文件对象的writelines()方法可以打印出来。

## Differ例子
比较两个文本的例子。首先我们设置文本，以新行结尾的包含单行字符串的独立序列（通过类文件对象的readlines()方法拿到）：
```python
>>> text1 = '''  1. Beautiful is better than ugly.
...   2. Explicit is better than implicit.
...   3. Simple is better than complex.
...   4. Complex is better than complicated.
... '''.splitlines(1)
>>> len(text1)
4
>>> text1[0][-1]
'\n'
>>> text2 = '''  1. Beautiful is better than ugly.
...   3.   Simple is better than complex.
...   4. Complicated is better than complex.
...   5. Flat is better than nested.
... '''.splitlines(1)
```

下面我们实例化一个Differ对象：`>>> d = Differ()`

注意到当实例化Differ对象时我们可以传递过滤函数和垃圾字符。查看Differ()构造器了解详情。

最后，比较二者：`>>> result = list(d.compare(text1, text2))`

result是一个字符串列表，让我们漂亮的打印出来：
```python
>>> from pprint import pprint
>>> pprint(result)
['    1. Beautiful is better than ugly.\n',
 '-   2. Explicit is better than implicit.\n',
 '-   3. Simple is better than complex.\n',
 '+   3.   Simple is better than complex.\n',
 '?     ++\n',
 '-   4. Complex is better than complicated.\n',
 '?            ^                     ---- ^\n',
 '+   4. Complicated is better than complex.\n',
 '?           ++++ ^                      ^\n',
 '+   5. Flat is better than nested.\n']
```

作为单一多行字符串，看起来是这样的：
```python
>>> import sys
>>> sys.stdout.writelines(result)
    1. Beautiful is better than ugly.
-   2. Explicit is better than implicit.
-   3. Simple is better than complex.
+   3.   Simple is better than complex.
?     ++
-   4. Complex is better than complicated.
?            ^                     ---- ^
+   4. Complicated is better than complex.
?           ++++ ^                      ^
+   5. Flat is better than nested.
```

## difflib的命令行接口
该例子展示如何使用difflib去创建diff-like工具。这也在python的源码中包含，Tools/scripts/diff.py
```python
""" Command line interface to difflib.py providing diffs in four formats:

* ndiff:    lists every line and highlights interline changes.
* context:  highlights clusters of changes in a before/after format.
* unified:  highlights clusters of changes in an inline format.
* html:     generates side by side comparison with change highlights.

"""

import sys, os, time, difflib, optparse

def main():
     # Configure the option parser
    usage = "usage: %prog [options] fromfile tofile"
    parser = optparse.OptionParser(usage)
    parser.add_option("-c", action="store_true", default=False,
                      help='Produce a context format diff (default)')
    parser.add_option("-u", action="store_true", default=False,
                      help='Produce a unified format diff')
    hlp = 'Produce HTML side by side diff (can use -c and -l in conjunction)'
    parser.add_option("-m", action="store_true", default=False, help=hlp)
    parser.add_option("-n", action="store_true", default=False,
                      help='Produce a ndiff format diff')
    parser.add_option("-l", "--lines", type="int", default=3,
                      help='Set number of context lines (default 3)')
    (options, args) = parser.parse_args()

    if len(args) == 0:
        parser.print_help()
        sys.exit(1)
    if len(args) != 2:
        parser.error("need to specify both a fromfile and tofile")

    n = options.lines
    fromfile, tofile = args # as specified in the usage string

    # we're passing these as arguments to the diff function
    fromdate = time.ctime(os.stat(fromfile).st_mtime)
    todate = time.ctime(os.stat(tofile).st_mtime)
    fromlines = open(fromfile, 'U').readlines()
    tolines = open(tofile, 'U').readlines()

    if options.u:
        diff = difflib.unified_diff(fromlines, tolines, fromfile, tofile,
                                    fromdate, todate, n=n)
    elif options.n:
        diff = difflib.ndiff(fromlines, tolines)
    elif options.m:
        diff = difflib.HtmlDiff().make_file(fromlines, tolines, fromfile,
                                            tofile, context=options.c,
                                            numlines=n)
    else:
        diff = difflib.context_diff(fromlines, tolines, fromfile, tofile,
                                    fromdate, todate, n=n)

    # we're using writelines because diff is a generator
    sys.stdout.writelines(diff)

if __name__ == '__main__':
    main()
```