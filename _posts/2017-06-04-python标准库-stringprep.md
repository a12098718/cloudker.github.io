---
published: true
layout: post
title: python标准库-stringprep
category: python
tags: 
  - python
time: 2017.06.04 12:32:00
excerpt: 余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

---

余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。
近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

<!--more-->

# python标准库-stringprep-网络预备字符串
2.3新增。

当在网络中标记东西时（如host名称），经常需要去比较标识的“等同”。严格上比较如何执行取决于应用程序域，例如，是否应该大小写敏感。同样也需要限制标识的合法性，仅允许标识符由可打印字符组成。

RFC 3454为Unicode字符串在网络协议上定义了“准备”的步骤。在传递字符串到线路上前，他们会通过预备过程处理，此后它们有一个确定的标准化格式。RFC定义了一个表集合，可以合并到profiles中。每个profile都必须定义一个它用的表，其他可选的stringprep过程的部分是profile的一部分。stringprep profile的一个例子是nameprep，用于国际化域名。

模块stringprep仅仅披露RFC 3454的表。所有这些表都可能非常巨大以展示为字典或列表，模块内部使用Unicode字符数据库。模块源码本身是通过mkstringprep.py工具生成的。

作为一个结果，这些表被暴露为函数，而不是数据结构。RFC中有两种表：集合和映射。对一个集合，stringprep提供“特有的函数”，例如，一个当参数是集合的一部分时返回true的函数。对于映射，他提供该映射函数：给定一个键，它返回关联的值。下面是所有模块可用函数的列表：
- stringprep.in_table_a1(code)
	- 判断code是否在tableA.1（Unicode 3.2中未赋值的code points）
- stringprep.in_table_b1(code)
	- 判断code是否在tableB.1(通常映射为无)
- stringprep.map_table_b2(code)
	- 返回code根据tableB.2（使用NFKC对case-folding的映射）的映射值
- stringprep.map_table_b3(code)
	- 返回code根据tableB.3(使用无标准化对case-folding的映射)的映射值
- stringprep.in_table_c11(code)
	- 判断code是否在tableC.1.1(ASCII空格符)
- stringprep.in_table_c12(code)
	- 判断code是否在tableC.1.2(非ASCII空格符)
- stringprep.in_table_c11_c12(code)
	- 判断code是否在tableC.1(空格符，并C.1.1和C.1.2)
- stringprep.in_table_c21(code)
	- 判断code是否在tableC.2.1(ASCII控制符)
- stringprep.in_table_c22(code)
	- 判断code是否在tableC.2.2(非ASCII控制符)
- stringprep.in_table_c21_c22(code)
	- 判断code是否在tableC.2(控制字符，并C.2.1和C.2.2)
- stringprep.in_table_c3(code)
	- 判断code是否在tableC.3(私用)
- stringprep.in_table_c4(code)
	- 判断code是否在tableC.4(非字符code points)
- stringprep.in_table_c5(code)
	- 判断code是否在tableC.5(代理code)
- stringprep.in_table_c6(code)
	- 判断code是否在tableC.6(平坦文本中不合适的)
- stringprep.in_table_c7(code)
	- 判断code是否在tableC.7(权威表示中不合适的)
- stringprep.in_table_c8(code)
	- 判断code是否在tableC.8(改变显示属性或不推荐的)
- stringprep.in_table_c9(code)
	- 判断code是否在tableC.9(标记字符)
- stringprep.in_table_d1(code)
	- 判断code是否在tableD.1(具有双向属性"R"或"AL"的字符)
- stringprep.in_table_d2(code)
	- 判断code是否在tableD.2(具有双向属性"L"的字符)