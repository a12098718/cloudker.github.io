---
published: true
layout: post
title: python标准库-calendar
category: python
tags: 
  - python
time: 2017.07.16 14:21:00
excerpt: 余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

---

余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。
近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

<!--more-->

# python标准库-calendar-通用日历相关函数
源码：lib/calendar.py

----------
该模块允许你像Unix cal程序那样输出日历，并且提供额外的有用的日历相关函数。默认情况下，这些日历将Monday作为一周的第一天，Sunday作为最后一天（欧洲约定）。使用setfirstweekday()来设置一周的第一天为Sunday(6)或是其他的工作日。指定date的参数以整型给定。对相关函数，可以查看datetime和time模块。

这些函数和类的大部分都依赖于datetime模块，datetime使用了理想化的日历，当前Gregorian日历无限期的双向延展。这匹配“预期的Gregorian”日历的概念，该概念在Dershoitz和Reingold的书“Calendrical Calculations”有所说明，它是所有计算的基础日历。

- class calendar.Calendar([firstweekday])
	- 创建一个个Calendar对象。firstweekday是一个整型数用于指定一周的第一天。0表示Monday（默认），6表示Sunday。
	- 一个Calendar对象提供多个方法可以用来准备日历数据的格式化。这一类并不格式化本身。这是子类的工作。
	- 2.5新增。
	- Calendar实例有下列方法：
	- iterweekdays()
		- 返回一个周中日数的迭代器，可以用于一周。迭代器的第一个值应该和firstweekday属性一致。
	- itermonthdates(year,month)
		- 返回一个在year年month(1-12)月的迭代器。该迭代器返回这一年月的所有日（如同datetime.date对象），从月的起始到月的结束，需要获取整个周。
	- itermonthdays2(year,month)
		- 返回一个year年month月的迭代器，类似itermonthdates()。返回的日是由天数和周数组成的元组。
	- itermonthdays(year,month)
		- 返回一个year年month月的迭代器，类似itermonthdates()。返回的日是简单的天数。
	- monthdatescalendar(year,month)
		- 返回year年month月的完整周的列表。周是七个datetime.date对象的列表
	- monthday2calendar(year,month)
		- 返回一个year年month月的完整周的列表。周是7个天数和周数组成的元组的列表
	- monthdayscalendar(year[, width])
		- 返回一个year年month月的完整周的列表。周是七天数的列表。
	- yeardatescalendar(year[, width])
		- 返回指定格式化年的数据。返回值是月行数的列表。每个月行包含到width月（默认为3）。每个月包含4到6个周且每个周包含1-7天。天是datetime.date对象。
	- yeardays2calendar(year[, width])
		- 返回指定格式化年的数据（类似yeardatescalendar()）。周列表的条目是天数和周数的元组。在月之外的天数为0。
	- yeardayscalendar(year[, width])
		- 返回指定格式化年的数据（类似yeardatescalendar()）。周列表条目是天数。月之外的天数为0。
- class calendar.TextCalendar([firstweekday])
	- 该类可以用于生成平坦的文本日历。
	- 2.5新增。
	- TextCalendar实例有下列方法：
	- formatmonth(theyear,themonth[,w[,l]])
		- 返回一个月的日历以多行字符串表示。如果w提供，它指定天数列的宽度，且为居中。如果l给定，它指定每个周所用的行数。像在构造器中指定或通过setfirstweekday()方法设定的那样，依赖于第一个weekday。
	- prmonth(theyear,themonth[,w[,l]])
		- 打印一个月的日历，像formatmonth()返回的那样。
	- formatyear(theyear[,w[,l[,c[,m]]]])
		- 返回一个m列的整年的多行字符串日历。可选参数w,l,c是为了天数列的宽度，每周行数以及每个月之间列数的空格数。像在构造器中指定或通过setfirstweekday()方法设定的那样，依赖于第一个weekday。最早的年是平台相关所生成的。
	- pryear(theyear[, w[, l[, c[, m]]]])
		- 打印整年的日历，就像formatyear()返回的那样。
-  class calendar.HTMLCalendar([firstweekday])
	-  