---
published: true
layout: post
title: python标准库-datetime
category: python
tags: 
  - python
time: 2017.07.03 19:42:00
excerpt: 余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

---

余弦:我没看过python的书，但我通读过python文档。文档不过一遍，视野会局限。
近日读sqlmap源码，发现自己对于标准库所知甚少，想到knownsec技能表中余弦的一番话，加之最近有意提升English能力，遂意通读python2.7.13官方文档，去粗取精，译成中文，国内虽有一译站点已有翻译，但为参考。然本钱平平，词不达意，技止于此，望看客海涵。

<!--more-->

# python标准库-datetime-基础日期和时间类型
2.3新增。
datetime模块提供简单和复杂的方法来精心操控日期和时间的类。日期和时间提供了算数操作，实现的焦点在于输出格式和操纵时高效的属性抽取。对相关的函数来说，也可以查看time和calendar模块。

有两种date和time对象类型："naive"和"aware"。

aware对象拥有对应用算法和策略时间调整充分的认知，诸如时区和令时信息，定位自身和其他aware对象相关联。一个aware对象用于表示特定的一个时刻，该时刻对解释器不开放。

一个naive对象并不包含充分的信息来精确定位自身和其他日期/时间对象相关联。一个naive对象是否表示成UTC，当地时间，或是其他时区的时间这都纯粹取决于程序，就像一个特别的数字用于表示米数，公里数或是群众数也都取决于程序一样。Naive对象易于理解、使用，这也是忽略了一些现实方面的代价。

对需要aware对象的应用来说，datetime和time对象有一个可选的时区信息属性，tzinfo，可以设置到一个抽象类tzinfo的子类的实例，这些tzinfo对象薄或关于UTF时间偏移的信息，时区名字，是否有令时生效。注意到datetime模块么有提供具体的tzinfo类。关于支持时区的任意层级的详细是否需要取决于应用程序。关于跨越世界的时间调整规则更为政策性而非合理性，对于每个应用程序来说没有一个标准的套装。

datetime模块导出了下列的常量：
- datetime.MINYEAR
	- date或datetime对象中最小的年数。MINYEAR是1。
- datetime.MAXYEAR
	- date或datetime对象中最大的年数。MAXYEAR是9999。

同时查阅：
- Module calendar
	- 通用日历相关函数
- Module time
	- 时间访问和转换

## 可用类型
- class datetime.date
	- 理想化naive数据，假定当前总是Gregorian日历，且永远如此。属性：year，month和day。
- class datetime.time
	- 理想时间，独立于任意特殊日，假定每一天都是精确的24*60*60秒（没有“跳跃秒数”的概念）。属性：hour，minute，second，microsecond和tzinfo。
- class datetime.datetime
	- date和time的组合。属性：year,month,day,hour,minute,second,microsecond和tzinfo。
- class datetime.timedelta
	- 一个duration，表示两个date，time或datetime实例间的差异为微秒分辨率。
- class datetime.tzinfo
	- 时区信息对象的抽象基类。被datetime和time类使用来提供定制的时间调整概念（例如，对时区和令时记账）。

这些类型的对象时不可变的。

date类型的对象永远是naive。

time或datetime类型的对象可以是naive或aware。datetime对象d是aware，如果d.tzinfo不是None且d.tzinfo.utcoffset(d)不返回None。如果d.tzinfo是None，或者d.tzinfo不是None但d.tzinfo.utcoffset(d)返回None，则d是naive。一个time对象t当t.tzinfo不是None且t.tzinfo.utcoffset(None)不返回None则是aware。否则，t是naive。

naive和aware的区别并不应用于timedelta对象。

子类关联：
```
object
    timedelta
    tzinfo
    time
    date
        datetime
```

## timedelta对象
timedelta对象表示一个duration，两个dates或times的差值。

- class datetime.timedelta([days[, seconds[, microseconds[, milliseconds[, minutes[, hours[, weeks]]]]]]])
	- 所有参数都是可选的且默认为0。参数可以是int,long,float，可正可负。
	- 只有days，seconds和microseconds会存储在内部。参数被转换成那些单位：
		- 1毫秒转换为1000微秒
		- 1分钟转换成60秒
		- 1小时转成3600秒
		- 1周转成7天
	- days，seconds和microseconds会被标准化以便于独特的表示形式，有：
		- 0 <= microseconds < 1000000
		- 0 <= seconds < 3600*24(一天的秒数)
		- -999999999 <= days <= 999999999
	- 如果任意参数是float且存在有小数的微秒，小数微秒会在所有参数合并时留下，且合并的和会四舍五入到微秒数。如果参数没有float，转换以及标准化过程都是严格精确的（没有信息丢失）。
	- 如果标准化后的值超出了预定的范围，抛出OverflowError。
	- 注意到标准化负数值可能会在一开始很惊奇。例如：
```python
>>> from datetime import timedelta
>>> d = timedelta(microseconds=-1)
>>> (d.days, d.seconds, d.microseconds)
(-1, 86399, 999999)
```

类属性：
- timedelta.min
	- 最小数timedelta对象，timedelta(-999999999)
- timedelta.max
	- 最大数timedelta对象，timedelta(days=999999999, hours=23, minutes=59, seconds=59, microseconds=999999)
- timedelta.resolution
	- 不等同的timedelta对象间最小的可能差值，timedelta(microseconds=1)

注意到，因为标准化，timedelta.max > -timedelta.min。-timedelta.max不可表示为一个timedelta对象。

实例属性（只读）：

| 属性 |	值 |
| ---- | - |
|days |	Between -999999999 and 999999999 inclusive|
|seconds |	Between 0 and 86399 inclusive|
|microseconds 	|Between 0 and 999999 inclusive|

支持的操作：

| 操作 | 结果 |
| --- | ----- |
| t1 = t2 + t3 | t2和t3的和。此后t1-t2==t3和t1-t3==t2为真 （1）|
| t1 = t2 - t3 | t2和t3的差。此后t1 == t2-t3,t2==t1+t3为真 （1）|
| t1 = t2 * i 或 t1 = i * t2 | 整型数相乘。此后t1//i == t2为真，这里i!=0 |
| - | 通常来说, t1 * i == t1 * (i-1) + t1 为真. (1) |
| t1 = t2 // i | 地板除，余数被丢弃（3） |
| +t1 | 返回一个想通知的timedelta对象(2) |
| -t1 | 等同于timedelta(-t1.days,-t1.seconds,-t1.microseconds)（1）（4）|
| abs(t) | 等同于+t当t.days>=0，-t当t.days<0（2） |
| str(t) | 返回一个格式为`[D day[s]][H]H:MM:SS[.UUUUUU]`的字符串，t为负数时D为负(5) |
| repr(t) | 返回一个格式为`datetime.timedelta(D[,S[,U]])`的字符串，t为负时D为负(5) |

注意：
1. 这是严格的，但可能溢出。
2. 这是严格的，不可能溢出。
3. 除0抛ZeroDivisionError。
4. -timedelta.max不可表示为timedelta对象。
5. 字符串表示timedelta对象会标准简单化成它们的内部表示。这会引导对负数timedeltas的不寻常结果。例如：

```python
>>> timedelta(hours=-5)
datetime.timedelta(-1, 68400)
>>> print(_)
-1 day, 19:00:00
```

除了上面的timedelta对象支持的确定的date和datetime对象的加减法（看下面）。

timedelta对象的比较被表示成更小的duration的timedelta对象所支持，它被认为是一个更小的timedelta。当一个timedelta对象和另一个类型的对象比较时，为了阻止通过对象地址回退到默认的混合类型比较，会抛出TypeError除非比较类型是`==`或`!=`。后种情况返回False或True。

timedelta对象是可哈希的（可以用于字典键），支持有效的pickling，在布尔上下文中，一个timedelta对象被认为是true当且仅当它不等于timedelta(0)。

实例方法：
- timedelta.total_seconds()
	-返回duration中包含的秒的总数。等价于`(td.microseconds + (td.seconds + td.days * 24 *3600) * 10**6) / 10**6`。
	- 注意到非常大的时间间隔（最大平台上270年以上）该方法会丢失微妙的精度。
	- 2.7新增。

例子用法：
```python
>>> from datetime import timedelta
>>> year = timedelta(days=365)
>>> another_year = timedelta(weeks=40, days=84, hours=23,
...                          minutes=50, seconds=600)  # adds up to 365 days
>>> year.total_seconds()
31536000.0
>>> year == another_year
True
>>> ten_years = 10 * year
>>> ten_years, ten_years.days // 365
(datetime.timedelta(3650), 10)
>>> nine_years = ten_years - year
>>> nine_years, nine_years.days // 365
(datetime.timedelta(3285), 9)
>>> three_years = nine_years // 3;
>>> three_years, three_years.days // 365
(datetime.timedelta(1095), 3)
>>> abs(three_years - ten_years) == 2 * three_years + year
True
``` 

## date对象
date对象以理想化日历的形式表示日期（年月日），Gregorian日历无限期双向扩展。首年的一月一日被称为日号1，首年的1月2日被称为日号2，以此类推。This matches the definition of the “proleptic Gregorian” calendar in Dershowitz and Reingold’s book Calendrical Calculations, where it’s the base calendar for all computations. 查阅书中的算法来转换预期的Gregorian序数与其他多种日历系统。

- class datetime.date(year, month, day)
	- 所有参数都是必须的。参数可以是int或long，范围如下：
		- MINYEAR <= year <= MAXYEAR
		- 1 <= month <= 12
		- 1 <= day <= 给定年月的日数
	- 如果给定的数超出了范围，抛ValueError。

其他构造器，所有的类方法：
- classmethod date.today()
	- 返回当前本地日期。等价于date.fromtimestamp(time.time())
- classmethod date.fromtimestamp(timestamp)
	- 返回和POSIX时间戳一致的本地日期，就好像time.time()返回的一样。这可能会抛ValueError，如果时间戳超出了平台C localtime()函数的范围。限制年数为1970到2038是合理的。注意到非POSIX系统在时间戳概念中包含跳跃秒数，跳跃秒数会被fromtimestamp()忽略。
- classmethod date.fromordinal(ordinal)
	- 返回和预期的Gregorian序数一致的日期，年1的1月1日序数为1。如果`1<=ordinal<=date.max.toordinal()`不满足则抛ValueError。对任意日期d来说，`date.fromordinal(d.toordinal()) == d`。

类属性：
- date.min
	- 表示最早的日期，date(MINYEAR, 1, 1)
- date.max
	- 表示最晚的日期，date(MAXYEAR, 12, 31)
- date.resolution
	- 两个不等date对象间最小的可能差值，timedelta(days=1)

实例属性（只读）：
- date.year
	- MINYEAR和MAXYEAR之间
- date.month
	- 1到12之间
- date.day
	- 1和给定年月的日期数之间

支持的操作：

| 操作 | 结果 |
| ---- | --- |
| date2 = date1 + timedelta | date2是将timedelta.days从date1中移除(1) |
| date2 = date1 - timedelta | 计算date2，date2 + timedelta == date1（2） |
| timedelta = date1 - date2 | （3） |
| date1 < date2 | date1被认为小于date2，当date1的日期早于date2 |

注意到：
1. 如果timedelta.days > 0则date2沿时间线递进，如果timedelta.days<0则回退。此后date2-date1==timedelta.days。timedelta.seconds和timedelta.microseconds被忽略。如果date2.year比MINYEAR小或比MAXYEAR大则抛OverflowError。
2. 这并不完全等同于date1+（-timedelta），因为孤立的-timedelta可能移除而date1-timedelta不会。timedelta.seconds和timedelta.microseconds被忽略。
3. 这是严格的不会溢出的。timedelta.seconds和timedelta.microseconds为0，此后date2+timedelta==date1。
4. 换句话说，date1<date2当且仅当date1.toordinal()<date2.toordinal()。为了阻止回退到默认比较对象地址机制的比较，日期比较通常抛出TypeError如果其他比较提并非一个date对象。然而，NotImplemented会被返回取代如果其他比较体有一个timetuple()属性。该hook给其他类型日期对象一个机会去实现混合类型比较。如果不是，当date对象在和其他对象比较时，除非比较方法是==或!=，否则会抛出TypeError。后者返回False或True。

日期可以用于字典键。在布尔上下文中，所有date对象都被认为是True。

实例方法：
- date.replace(year, month, day)
	- 返回等值的date，除了给那些通过关键字参数指定的参数给定的新值。例如，如果d == date(2002,12,31)则d.replace(day=26)==date(2002,12,26)。
- date.timetuple()
	- 返回一个time.struct_time如同time.localtime()一样。hours，minutes和seconds都是0，DST标记为-1。d.timetuple()等同于`time.struct_time((d.year,d.month,d.day,0,0,0,d.weekday(),yday,-1))`，`yday = d.toordinal()-date(d.year,1,1).toordinal()+1`是当前年以1开始的1月1日。
- date.toordinal()
	- 返回预期的Gregorian日期序，1年1月1日为1。对任意date对象d，`date.fromordinal(d.toordinal())==d`。
- date.weekday()
	- 返回周里的日，作为整型返回，Monday是0，Sunday是6。例如，`date(2002,12,4).weekday() == 2`，也就是星期三。也可以看看isoweekday()。
- date.isoweekday()
	- 返回周里的日，作为整型返回。Monday是1，Sunday是7。例如，`date(2002,12,4).isoweekday() == 3`，是星期三。可以看看weekday(),isocalendar()。
- date.isocalendar()
	- 返回三元组，（ISO year， ISO week number， ISO weekday）
	- ISO日历会被宽泛的使用于Gregorian日历的变体种。查看https://www.staff.science.uu.nl/~gent0113/calendar/isocalendar.htm获取更好的解释。
	- ISO年包含52或53整周，周始于Monday结束于Sunday。ISO年的首周是包含一个周四的年的第一个（Gregorian）日历周。这被称为周数1，ISO年的那个星期四也等同于它的Gregorian年。
	- 例如，2004开始于周四，所以ISO年2004的第一周开始于Monday，29 Dec 2003结束于Sunday，4 Jan 2004，因此`date(2003,12,29).isocalendar() == (2004,1,1)`且`date(2004,1,4).isocalendar() == (2004,1,7)`。
- date.isoformat()
	- 返回一个表示为ISO 8601格式的日期，‘YYYY-MM-DD’。例如，date(2002,12,4).isoformat() == '2002-12-04'。
- date.__str__()
	- 对date d来说，str(d)等同于d.isoformat()。
- date.ctime()
	- 返回表示date的字符串，例如`date(2002,12,4).ctime() == 'Wed Dec 4 00：00：00 2002'`。`d.ctime()`等价于`time.ctime(time.mktime(d.timetuple()))`在那些本地C 遵循C标准的ctime()函数平台上。
- date.strftime(format)
	- 返回一个表示date的字符串，由显示的个是字符串控制。格式代码引用小时、分钟或秒数可以看0值。对一个完整的格式化指令列表，查看strftime()和strptime()行为。
- date.__format__(format)
	- 等同于date.strftime()。这使得为date对象在用str.format()函数时指定格式化字符串是可能的。查看节strftime()和strptime()行为。

Counting days to an event示例：
```python
>>> import time
>>> from datetime import date
>>> today = date.today()
>>> today
datetime.date(2007, 12, 5)
>>> today == date.fromtimestamp(time.time())
True
>>> my_birthday = date(today.year, 6, 24)
>>> if my_birthday < today:
...     my_birthday = my_birthday.replace(year=today.year + 1)
>>> my_birthday
datetime.date(2008, 6, 24)
>>> time_to_birthday = abs(my_birthday - today)
>>> time_to_birthday.days
202
```

date工作示例：
```python
>>> from datetime import date
>>> d = date.fromordinal(730920) # 730920th day after 1. 1. 0001
>>> d
datetime.date(2002, 3, 11)
>>> t = d.timetuple()
>>> for i in t:     
...     print i
2002                # year
3                   # month
11                  # day
0
0
0
0                   # weekday (0 = Monday)
70                  # 70th day in the year
-1
>>> ic = d.isocalendar()
>>> for i in ic:    
...     print i
2002                # ISO year
11                  # ISO week number
1                   # ISO day number ( 1 = Monday )
>>> d.isoformat()
'2002-03-11'
>>> d.strftime("%d/%m/%y")
'11/03/02'
>>> d.strftime("%A %d. %B %Y")
'Monday 11. March 2002'
>>> 'The {1} is {0:%d}, the {2} is {0:%B}.'.format(d, "day", "month")
'The day is 11, the month is March.'
```

## datetime对象
datetime对象是一个包含从date对象和time对象所有信息的单一对象。类似date对象，datetime假定当前Gregorian日历双向扩展；类似time对象，datetime假定每天有3600*24秒。

构造器：
- class datetime.datetime(year,month,day[,hour[,minute[,second[,microsecond[,tzinfo]]]]])
	- 年，月和日参数是需要的。tzinfo可以是None，或是一个tzinfo子类的实例。保留参数可以是int或long，范围如下：
		- MINYEAR <= year <= MAXYEAR
    	- 1 <= month <= 12
		- 1 <= day <= number of days in the given month and year
		- 0 <= hour < 24
		- 0 <= minute < 60
		- 0 <= second < 60
		- 0 <= microsecond < 1000000
	- 如果参数值超过了范围，抛ValueError。

其他构造器，所有类方法：
- classmethod datetime.today()
	- 返回当前本地datetime，tzinfo为None。这等价于`datetime.fromtimestamp(time.time())`。可查看now(),fromtimestamp()。
- classmethod datetime.now([tz])
	- 返回当前本地日期和时间。如果可选参数tz是None或没有指定，这类似与today()，但是，如果可能，比time.time()时间戳所获取可以支持更高精度（例如，支持C gettimeofday()函数的平台上这是可能的）。
	- 如果tz不是None，则必须是tzinfo子类的实例，当前date和time转换成tz的时间域。在这种情况，结果等价于`tz.fromutc(datetime.utcnow().replace(tzinfo=tz))`。可查看today()，utcnow()。
- classmethod datetime.utcnow()
	- 返回当前UTC日期和时间，tzinfo为None。类似now()，但是返回当前UTC日期和时间，一个naive datetime对象。亦可查看now()。
- classmethod datetime.fromtimestamp(timestamp[,tz])
	- 返回本地日期和时间，遵循POSIX时间戳标准，就像time.time()返回一样。如果可选参数tz是None或没有指定，timestamp转换为平台的本地日期和时间，返回的datetime对象是naive的。
	- 如果tz非None，则必须为tzinfo子类的实例，时间戳转换为tz的时间域。这种情况结果等价于`tz.fromutc(datetime.utcfromtimestamp(timestamp).replace(tzinfo=tz))`
	- fromtimestamp()可能抛ValueError，如果timestamp超出了平台C localtime()或gmtime()函数范围。约束年为1970到2038是很正常的。注意到非POSIX系统包含跳跃时间在其时间戳的概念时，跳跃时间会被fromtimestamp()忽略。此后两个timestamps差值通过datetime对象的second域表示是可能的。亦可查看utcfromtimestamp()。
- classmethod datetime.utcfromtimestamp(timestamp)
	- 返回UTC datetime，和POSIX时间戳一致，tzinfo为None。如果timestamp越界则抛ValueError。约束年为1970到2038很是平常。可以查看fromtimestamp()。
- classmethod datetime.fromordinal(ordinal)
	- 返回和预期Gregorian序一致的datetime，年1的1月1日为1。如果 `1 <= ordinal <= datetime.max.toordinal()`不满足，则抛ValueError。结果中hour，minute，second和microsecond都为0，tzinfo是None。
- classmethod datetime.combine(date,time)
	- 返回一个新datetime对象，date组件等同于给定的date对象，时间组件和tzinfo属性等同于给定的time对象。对任意datetime对象d，`d == datetime.combine(d.date(),d.timetz())`。如果date是datetime对象，它的time组件和tzinfo属性被忽略。
- classmethod datetime.strptime(date_string, format)
	- 返回datetime，和date_string一致，根据format解析。这等价于`datetime(*(time.strptime(date_string, format)[0:6]))`。如果date_string和格式不能够由time.strptime()解析，则抛ValueError。如果返回的值不是time tuple格式，也抛ValueError。对一个完整的格式化指令列表，查看节“strftime()和strptime()行为”。
	- 2.5新增。

类属性：
- datetime.min
	- 最早的datetime表示，`datetime(MINYEAR, 1, 1, tzinfo=None）`。
- datetime.max
	- 最晚的datetime表示，`datetime(MAXYEAR, 12, 31, 23, 59, 59, 999999, tzinfo=None)`。
- datetime.resolution
	- 最小的可能差值在不等同datetime对象间。`timedelta(microseconds=1)`。

实例属性（只读）：
- datetime.year
	- MINYEAR和MAXYEAR之前
- datetime.month
	- 1到12之间
- datetime.day
	- 1到给定年月的日期天数
- datetime.hour
	- range(24)
- datetime.minute
	- range(60)
- datetime.second
	- range(60)
- datetime.microsecond
	- range(1000000)
- datetime.tzinfo
	- 作为tzinfo参数传递给datetime构造器的对象，如果未传递则为None。

支持的操作：

| 操作 | 结果 |
| --- | ---- |
| datetime2 = datetime1 + timedelta | (1) |
| datetime2 = datetime1 - timedelta | (2) |
| timedelta = datetime1 - datetime2 | (3) |
| datetime1 < datetime2 | 比较datetime和datetime(4) |

1. datetime2是一个timedelta的duration，移除自datetime1，如果timedelta.days>0则沿时间线向前，如果timedelta.days<0则回退。作为输入的datetime，结果具有相同的tzinfo属性，此后datetime2-datetime1==timedelta。OverflowError会在datetime2.year小于MINYEAR或大于MAXYEAR的时候抛出。注意到如果输入是个aware对象，则不会进行时间域调整。
2. 计算datetime2如同datetime2+timedelta==datetime1。额外的，结果具有等同的tzinfo属性作为input的datetime，如果输入时aware则不会进行时间域调整。这并不完全等同于datetime1+(-timedelta)，因为独立的-timedelta可能会溢出而datetime1-timedelta不会。
3. datetime的减法仅在两个操作数都是naive或都是aware时才有定义。如果仅有一个是aware且另一个是naive，抛TypeError。如果两个都是naive，或两个都是aware且具有等同的tzinfo属性，tzinfo会被忽略，结果是一个timedelta对象t，有`datetime2+t==datetime1`。此时不会进行时间域调整。如果两个都是aware且具有不同的tzinfo属性，a-b表现的如同a和b都是首先转换到naive UTC datetime。结果是`(a.replace(tzinfo=None) - a.utcoffset()) - (b.replace(tzinfo=None) - b.utcoffset())`，实现体永不溢出。
4. datetime1被认为小于datetime2当datetime1早先于datetime2在时序上。如果一个比较数是naive而另一个是aware，会抛TypeError。如果二者都是aware，tzinfo属性等同，则tzinfo被忽略，比较的就是基本的datetimes。如果二者都是aware且tzinfo不同，则比较会首先调整到UTC偏移再做比较（self.htcoffset()获取）。

> 注意到：为了阻止回退到比较对象地址的默认体制中，datetime比较通常会在其他操作数不是datetime对象时抛出TypeError。然而，NotImplemented会被返回做取代如果其他比较数有一个timetuple()属性。该hook给定了其他的date对象类型一个机会去实现混合类型比较。若非如此，当datetime对象和其他类型对象比较时，抛TypeError，除非比较操作时==或!=。后者返回False或True。

datetime对象可以用于字典键。在布尔上下文中，所有datetime对象都被认为是True。

实例方法：
- datetime.date()
	- 返回同年、月、日的date对象。
- datetime.time()
	- 返回同小时、分钟、秒、微秒的time对象。tzinfo是None。可查看方法timetz()。
- datetime.timetz()
	- 返回同小时、分钟、秒、微秒的time对象，以及tzinfo属性。查看方法time()。
- datetime.replace([year[,month[,day[,hour[,minute[,second[,microsecond[,tzinfo]]]]]]])
	- 返回同属性的datetime，除了那些指定的关键字参数所给定新值的属性。注意到tzinfo=None可以用于指定去通过一个aware datetime来创建一个naive datetime，无需date和time数据的转换。
- datetime.astimezone(tz)
	- 返回datetime对象，携带新的tzinfo属性tz，调整日期和时间数据使得结果和self是相同的UTC时间，而不是tz的本地时间。
	- tz必须是一个tzinfo子类实例，它的utcoffset()和dst()方法不能返回None。self可以必须是aware（self.tzinfo必须不能是None，self.utcoffset()必须不能返回None）。
	- 如果self.tzinfo是tz，self.astimezone(tz)等价于self：不执行对日期或时间的调整。结果是时间域tz的本地时间，表示和self相同的UTC时间：`astz=dt.astimezone(tz),astz-astz.utcoffset()`后通常具有和`dt-dt.utcoffset()`等同的日期和时间数据。tzinfo类的讨论解释了令时的转换分界线。
	- 如果你仅仅想要附加时间域对象tz到datetime dt而不调整日期和时间，可以使用`dt.replace(tzinfo=tz)`。如果你仅仅想要从一个aware datetime dt中移除时间域对象，使用`dt.replace(tzinfo=None)`。
	- 注意到默认的tzinfo.fromutc()方法可以在tzinfo子类中覆盖，以影响astimezone()返回的结果。忽略错误情况，astimezone()表现如同：
```python
def astimezone(self, tz):
    if self.tzinfo is tz:
        return self
    # Convert self to UTC, and attach the new time zone object.
    utc = (self - self.utcoffset()).replace(tzinfo=tz)
    # Convert from UTC to tz's local time.
    return tz.fromutc(utc)
```
- datetime.utcoffset()
	- 如果tzinfo是None，返回None，否则返回`self.tzinfo.utcoffset(self)`，如果后者不返回None则抛出异常，或是一个timedelta对象表示整个分钟数，其量级上小于一天。
- datetime.dst()
	- 如果tzinfo是None，返回None，否则返回`self.tzinfo.dst(self)`，如果后者不返回None则抛出异常，或是一个timedelta对象表示整个分钟数，量级小于一天。
- datetime.tzname()
	- 如果tzinfo是None，返回None，否则返回`self.tzinfo.tzname(self)`，抛出一个异常如果后者不返回None或是一个字符串对象。
- datetime.timetuple()
	- 返回一个像time.localtime()返回的time.struct_time。`d.timetuple()`等价于`time.struct_time((d.year, d.month, d.day, d.hour, d.minute, d.second, d.weekday(), yday, dst))`，当`yday = d.toordinal() - date(d.year, 1, 1).toordinal() + 1`是日数在当前年以1年1月一日开始计算。结果中tm_isdst标志根据dst()方法设置：tzinfo是None或者dst()返回None，tm_isdst设置为-1；否则如果dst()返回非0值，tm_isdst设置为1；否则tm_isdst设置为0。
- datetime.utctimetuple()
	- 如果datetime实例d是naive，这等同于d.timetuple()除了tm_isdst强制为0不管d.dst()返回什么。DST永远不会为UTC时间生效。
	- 如果d是aware，d标准化成UTC时间，通过d.utcoffset()减法运算，以及一个返回的time.struct_time标准化时间。tm_isdst强制为0。注意到结果的tm_year成员可能是MINYEAR-1或MAXYEAR+1，当d.year是MINYEAR或MAXYEAR且UTC调整溢出了一年的边界时。
- datetime.toordinal()
	- 返回预期的Gregorian日期序。等同于`self.date().toordinal()`。
- datetime.weekday()
	- 返回周中的日期整型数，Monday是0，Sunday是6。等同于`self.date().weekday()`。也可查看isoweekday()。
- datetime.isoweekday()
	- 返回周中的日期整型数，Monday是1，Sunday是7。等同于`self.date().isoweekday()`。也可查看weekday(),isocalendar()。
- datetime.isocalendar()
	- 返回一个3元组，（ISO year，ISO week number，ISO weekday）。等同于`self.date().isocalendar()`。
- datetime.isoformat([sep])
	- 返回一个表示日期和时间的基于ISO 8601格式的字符串，YYYY-MM-DDTHH:MM:SS.mmmmmm或者如果microsecond是0，YYYY-MM-DDTHH:MM:SS
	- 如果utcoffset()不返回None，附加一个6字符字符串，给定UTC偏移为有符号的小时和分钟数：YYYY-MM-DDTHH:MM:SS.mmmmmm+HH:MM或者，如果microsecond是0的话，YYYY-MM-DDTHH:MM:SS+HH:MM
	- 可选参数sep（默认为'T'）是一个分割字符，放置在date和time之间。例如：
```python
>>> from datetime import tzinfo, timedelta, datetime
>>> class TZ(tzinfo):
...     def utcoffset(self, dt): return timedelta(minutes=-399)
...
>>> datetime(2002, 12, 25, tzinfo=TZ()).isoformat(' ')
'2002-12-25 00:00:00-06:39'
```
- datetime.__str__()
	- 对datetime实例d，`str(d)`等价于`d.isoformat(' ')`。
- datetime.ctime()
	- 返回一个表示日期和时间的字符串，例如`datetime(2002, 12, 4, 20, 30, 40).ctime() == 'Wed Dec  4 20:30:40 2002'`。`d.ctime()`等价于`time.ctime(time.mktime(d.timetuple())) `在平台上当本地C的ctime()函数遵循C标准（time.ctime()调用，但是datetime.ctime()不会调用）。
- datetime.strftime(format)
	- 返回一个表示日期和时间的字符串，由显示的个是字符串控制。对于完整的格式化指令列表，查看节"strftime()和strptime()行为"。
- datetime.__format__(format)
	- 等同于`datetime.strftime()`。这使得为一个datetime对象使用str.format()指定一个格式化串是可能的。查看节"strftime()和strptime()行为"。

datetime对象工作的例子：

```python
>>> from datetime import datetime, date, time
>>> # Using datetime.combine()
>>> d = date(2005, 7, 14)
>>> t = time(12, 30)
>>> datetime.combine(d, t)
datetime.datetime(2005, 7, 14, 12, 30)
>>> # Using datetime.now() or datetime.utcnow()
>>> datetime.now()   
datetime.datetime(2007, 12, 6, 16, 29, 43, 79043)   # GMT +1
>>> datetime.utcnow()   
datetime.datetime(2007, 12, 6, 15, 29, 43, 79060)
>>> # Using datetime.strptime()
>>> dt = datetime.strptime("21/11/06 16:30", "%d/%m/%y %H:%M")
>>> dt
datetime.datetime(2006, 11, 21, 16, 30)
>>> # Using datetime.timetuple() to get tuple of all attributes
>>> tt = dt.timetuple()
>>> for it in tt:   
...     print it
...
2006    # year
11      # month
21      # day
16      # hour
30      # minute
0       # second
1       # weekday (0 = Monday)
325     # number of days since 1st January
-1      # dst - method tzinfo.dst() returned None
>>> # Date in ISO format
>>> ic = dt.isocalendar()
>>> for it in ic:   
...     print it
...
2006    # ISO year
47      # ISO week
2       # ISO weekday
>>> # Formatting datetime
>>> dt.strftime("%A, %d. %B %Y %I:%M%p")
'Tuesday, 21. November 2006 04:30PM'
>>> 'The {1} is {0:%d}, the {2} is {0:%B}, the {3} is {0:%I:%M%p}.'.format(dt, "day", "month", "time")
'The day is 21, the month is November, the time is 04:30PM.'
```

使用携带tzinfo的datetime：

```python
>>> from datetime import timedelta, datetime, tzinfo
>>> class GMT1(tzinfo):
...     def utcoffset(self, dt):
...         return timedelta(hours=1) + self.dst(dt)
...     def dst(self, dt):
...         # DST starts last Sunday in March
...         d = datetime(dt.year, 4, 1)   # ends last Sunday in October
...         self.dston = d - timedelta(days=d.weekday() + 1)
...         d = datetime(dt.year, 11, 1)
...         self.dstoff = d - timedelta(days=d.weekday() + 1)
...         if self.dston <=  dt.replace(tzinfo=None) < self.dstoff:
...             return timedelta(hours=1)
...         else:
...             return timedelta(0)
...     def tzname(self,dt):
...          return "GMT +1"
...
>>> class GMT2(tzinfo):
...     def utcoffset(self, dt):
...         return timedelta(hours=2) + self.dst(dt)
...     def dst(self, dt):
...         d = datetime(dt.year, 4, 1)
...         self.dston = d - timedelta(days=d.weekday() + 1)
...         d = datetime(dt.year, 11, 1)
...         self.dstoff = d - timedelta(days=d.weekday() + 1)
...         if self.dston <=  dt.replace(tzinfo=None) < self.dstoff:
...             return timedelta(hours=1)
...         else:
...             return timedelta(0)
...     def tzname(self,dt):
...         return "GMT +2"
...
>>> gmt1 = GMT1()
>>> # Daylight Saving Time
>>> dt1 = datetime(2006, 11, 21, 16, 30, tzinfo=gmt1)
>>> dt1.dst()
datetime.timedelta(0)
>>> dt1.utcoffset()
datetime.timedelta(0, 3600)
>>> dt2 = datetime(2006, 6, 14, 13, 0, tzinfo=gmt1)
>>> dt2.dst()
datetime.timedelta(0, 3600)
>>> dt2.utcoffset()
datetime.timedelta(0, 7200)
>>> # Convert datetime to another time zone
>>> dt3 = dt2.astimezone(GMT2())
>>> dt3     
datetime.datetime(2006, 6, 14, 14, 0, tzinfo=<GMT2 object at 0x...>)
>>> dt2     
datetime.datetime(2006, 6, 14, 13, 0, tzinfo=<GMT1 object at 0x...>)
>>> dt2.utctimetuple() == dt3.utctimetuple()
True
```

## time对象
time对象表示当地日的时间，独立于任意特殊的日期，服从tzinfo对象的调整。
- class datetime.time([hour[,minute[,second[,microsecond[,tzinfo]]]])
	- 所有参数是可选的。tzinfo可能是None，或是一个tzinfo子类的实例。保留参数可以是int或long，在下列范围：	
	    - 0 <= hour < 24
	    - 0 <= minute < 60
	    - 0 <= second < 60
	    - 0 <= microsecond < 1000000.
	- 如果一个参数越界，抛ValueError。所有默认为0除了tzinfo，tzinfo默认为None。

类属性：
- time.min
	- 最早的时间表示，`time(0,0,0,0)`。
- time.max
	- 最晚的时间表示，`time(23, 59, 59, 999999)`。
- time.resolution
	- 最小的可能差值在两个不等的time对象之间，`timedelta(microseconds=1)`，尽管注意到time对象的算术运算是不支持的。

实例属性（只读）：
- time.hour
	- range(24)
- time.minute
	- range(60)
- time.second
	- range(60)
- time.microsecond
	- range(1000000)
- time.tzinfo
	- 给time构造器传递的tzinfo参数，如果不传递则为None。

支持操作：
- time之间的比较，a被认为小于b当a日期早先于b。如果一个操作数是naive另一个是aware，抛TypeError。当两个操作数都是aware，具有相同的tzinfo属性，tzinfo被忽略且基本的时间做比较。如果两个比较对象都是aware且tzinfo不同，则首先要调整到UTC时间再比较。为了阻止回退到基本对象地址比较的混合类型比较，当time对象和另外类型对象比较时，抛TypeError，除非比较操作是==或!=。后者返回False或True。
- 哈希，用于字典键
- efficient pickling
- 在布尔上下文，time对象被认为是true当且仅当，在转换成分钟后并减去utcoffset()（0如果是None）后，结果非零。

实例方法：
- time.replace([hour[,minute[,second[,microsecond[,tzinfo]]]]])
	- 返回一个值相等的time，除了那些通过特定关键字参数给定新值的属性。注意到tzinfo=None可以被指定去通过一个aware time创建一个naive time，无需时间数据的转换。
- time.isoformat()
	- 返回一个ISO 8601格式的时间字符串，HH:MM:SS.mmmmmm或者，如果self.microsecond为0，HH:MM:SS。如果utcoffset()不返回None，一个6字符字符串被附加，给定UTC偏移为有符号的小时和分钟数：HH:MM:SS.mmmmmm+HH:MM或者，如果self.microsecond为0，HH:MM:SS+HH:MM
- time.__str__()
	- 对一个time 他来说，str(t)等同于t.isoformat()。
- time.strftime(format)
	- 返回一个表示时间的字符串，由显示的格式化串控制。一个完整的格式化指令列表可以参阅节"strftime()和strptime()行为"。
- time.__format__(format)
	- 等同于time.strftime()。这使得用str.format()为一个time对象指定一个格式化字符串变为可能。查看节strftime()以及strptime()行为。
- time.utcoffset()
	- 如果tzinfo是None，返回None，否则返回`self.tzinfo.utcoffset(None)`，如果后者并不返回None或一个表示完整分钟数的小于一天量级的timedelta对象，抛出一个异常。
- time.dst()
	- 如果tzinfo是None，返回None，否则返回`self.tzinfo.dst(None)`，如果后者没有返回None，或是一个量级小于一天表示完整分钟数的timedelta对象，则抛出异常。
- time.tzname()
	- 如果tzinfo是None，返回None，否则返回`self.tzinfo.tzname(None)`。如果后者没有返回None或是一个字符串对象，则抛出异常。

例子：
```python
>>> from datetime import time, tzinfo, timedelta
>>> class GMT1(tzinfo):
...     def utcoffset(self, dt):
...         return timedelta(hours=1)
...     def dst(self, dt):
...         return timedelta(0)
...     def tzname(self,dt):
...         return "Europe/Prague"
...
>>> t = time(12, 10, 30, tzinfo=GMT1())
>>> t                               
datetime.time(12, 10, 30, tzinfo=<GMT1 object at 0x...>)
>>> gmt = GMT1()
>>> t.isoformat()
'12:10:30+01:00'
>>> t.dst()
datetime.timedelta(0)
>>> t.tzname()
'Europe/Prague'
>>> t.strftime("%H:%M:%S %Z")
'12:10:30 Europe/Prague'
>>> 'The {} is {:%H:%M}.'.format("time", t)
'The time is 12:10.'
```

## tzinfo对象
- class datetime.tzinfo
	- 这是个抽象基类，意味着该类不应该被直接实例化。你需要继承一个具体的子类，至少提供你使用的datetime方法所需要的标准tzinfo方法实现。datetime模块不支持任何具体tzinfo子类。
	- tzinfo的一个实例（具体子类）可以传递给datetime和time的构造器。后者对象视其属性为当地时间，且tzinfo对象支持方法显示到本地时间到UTC时间的偏移，时间域的名字，DST偏移，所有时间或日期对象的相关都会传递给它们。
	- pickling特殊的需求：一个tzinfo子类必须有一个__init__()方法，可以以无参数调用，否则他可以被pickled但是可能不会再次被unpickled。这是一个技术需求可能在未来得到缓解。
	- 一个具体tzinfo的子类可能需要实现下列方法。严格来说需要的方法取决于aware datetime对象的使用。如果有所怀疑，单纯的实现所有方法。
- tzinfo.utcoffset(self,dt)
	- 返回当地时间到UTC的偏移，UTC东区，分钟为单位。如果当地时间是西区，这应该是负数。注意到这打算称为整个UTC偏移；例如，如果一个tzinfo对象表示时间域和DST调整，utoffset()应该返回它们的和。如果UTC偏移不知道，则返回None。否则返回的值必须是一个timedelta对象，其指定了整个分钟数在-1439到1439范围内（1440=24*60；偏移量级小于一天）。大部分utcoffset()的实现可能看起来像下面两个中的一个：
```python
return CONSTANT                 # fixed-offset class
return CONSTANT + self.dst(dt)  # daylight-aware class
```
	- 如果utcoffset()不返回None，dst()应该也不返回None。
	- 默认的utcoffset()实现抛出NotImplementedError。
- tzinfo.dst(self, dt)
	- 返回令时（DST）调整，UTC东区，分钟为单位，如果DST不清楚则为None。返回timedelta(0)如果DST不生效。如果DST生效，返回偏移作为一个timedelta对象（看utcoffset()获取详细信息）。注意到DST偏移，如果可以应用，则已经通过utcoffset()加到UTC偏移，因此没有必要去咨询dst()除非你对获取DST信息感兴趣。例如，datetime.timetuple()调用它的tzinfo属性的dst()方法来决定如何设置tm_isdst标记，且tzinfo.fromutc()调用dst()来计数DST改变当跨越时区时。
	- 一个tzinfo子类的实例tz模范了标准和白日时间必须为连续的：`tz.utcoffset(dt)-tz.dst(dt)`
	- 对每个datetime dt，如有`dt.tzinfo==tz`，则必须返回相同的结果。对健全的tzinfo子类，表达式域、时区的“标准偏移”，不应该依赖于时间或日期，而仅应该依赖于地理位置。datetime.astimezone()的实现依赖于此，但是不能检测妨碍；这是程序员的职责来确保它。如果一个tzinfo子类不能保证，他应该可以覆盖默认的tzinfo.fromutc()实现来正确的工作不管astimezone()如何。
	- 大部分dst()实现可能看起来像下面两个中的一个：

```python
def dst(self, dt):
    # a fixed-offset class:  doesn't account for DST
    return timedelta(0)

def dst(self, dt):
    # Code to set dston and dstoff to the time zone's DST
    # transition times based on the input dt.year, and expressed
    # in standard local time.  Then

    if dston <= dt.replace(tzinfo=None) < dstoff:
        return timedelta(hours=1)
    else:
        return timedelta(0)
```

- tzinfo.tzname(self,dt)
	- 返回时间域的字符串，名称和datetime对象dt一致。string名称相关的任何事物都没有在datetime模块定义，也没有需求如此这意味着任何东西都是特别的。例如，"GMT","UTC","-500","-5：00","EDT","US/Eastern","America/New York"都是有效的答复。当string名称不知道时，返回None。注意到比起一个修正的字符串这更是一个方法，因为一些tzinfo子类可能希望返回依赖特定dt传递值的不同名字，特别是tzinfo类计数白天的时间。
	- tzname()的默认实现抛NotImplementedError。

这些方法都被datetime或time对象调用，响应它们的相同名字的方法。一个datetime对象传递自身为参数，一个time对象传递None为参数。一个tzinfo子类方法应该准备去接收一个None的dt参数，或是类datetime。

当传递None时，他取决于类设计者来决定最好的响应。例如，返回None是合适的当类希望声明time对象不参与tzinfo协议。utcoffset(None)返回一个标准UTC偏移可能更为有用，因为没有其他转换来发现标准的偏移。

当一个datetime对象传递来响应datetime方法，dt.tzinfo和self是同一对象。tzinfo方法可以依赖与此，除非用户代码直接调用tzinfo方法。其目的在于tzinfo方法解释dt为当地时间，无需担心其他时区的对象。

有一个多出来的tzinfo方法，子类可能希望去覆盖：
- tzinfo.fromutc(self,dt)
	- 默认的datetime.astimezone()实现会调用它。当调用它时，dt.tzinfo是self，dt的date和time数据视为表示成UTC时间。fromutc()的目的在于调整日期和时间数据，返回一个等价的datetime在self的本地时间。
	- 大多数tzinfo子类应该可以无问题继承默认的fromutc()实现。它足够强大去处理修正偏移的时区，以及时区的标准和白日时间计数，后者甚至可以在不同年份进行DST转换时间。一个时区的例子是默认的fromutc()实现并不会处理的正确在所有情况下，当标准偏移（UTC获取）依赖于传递的特定日期和时间，可能由于政策发生。默认astimezone()和fromutc()的实现可能产生你想要的结果如果结果是跨跃在标准偏移变更的时刻的那个小时。
	- 忽略code的错误情形，默认fromutc()实现可能像这样：
```python
def fromutc(self, dt):
    # raise ValueError error if dt.tzinfo is not self
    dtoff = dt.utcoffset()
    dtdst = dt.dst()
    # raise ValueError if dtoff is None or dtdst is None
    delta = dtoff - dtdst  # this is self's standard offset
    if delta:
        dt += delta   # convert to standard local time
        dtdst = dt.dst()
        # raise ValueError if dtdst is None
    if dtdst:
        return dt + dtdst
    else:
        return dt
```

tzinfo类例子：
```python
from datetime import tzinfo, timedelta, datetime

ZERO = timedelta(0)
HOUR = timedelta(hours=1)

# A UTC class.

class UTC(tzinfo):
    """UTC"""

    def utcoffset(self, dt):
        return ZERO

    def tzname(self, dt):
        return "UTC"

    def dst(self, dt):
        return ZERO

utc = UTC()

# A class building tzinfo objects for fixed-offset time zones.
# Note that FixedOffset(0, "UTC") is a different way to build a
# UTC tzinfo object.

class FixedOffset(tzinfo):
    """Fixed offset in minutes east from UTC."""

    def __init__(self, offset, name):
        self.__offset = timedelta(minutes = offset)
        self.__name = name

    def utcoffset(self, dt):
        return self.__offset

    def tzname(self, dt):
        return self.__name

    def dst(self, dt):
        return ZERO

# A class capturing the platform's idea of local time.

import time as _time

STDOFFSET = timedelta(seconds = -_time.timezone)
if _time.daylight:
    DSTOFFSET = timedelta(seconds = -_time.altzone)
else:
    DSTOFFSET = STDOFFSET

DSTDIFF = DSTOFFSET - STDOFFSET

class LocalTimezone(tzinfo):

    def utcoffset(self, dt):
        if self._isdst(dt):
            return DSTOFFSET
        else:
            return STDOFFSET

    def dst(self, dt):
        if self._isdst(dt):
            return DSTDIFF
        else:
            return ZERO

    def tzname(self, dt):
        return _time.tzname[self._isdst(dt)]

    def _isdst(self, dt):
        tt = (dt.year, dt.month, dt.day,
              dt.hour, dt.minute, dt.second,
              dt.weekday(), 0, 0)
        stamp = _time.mktime(tt)
        tt = _time.localtime(stamp)
        return tt.tm_isdst > 0

Local = LocalTimezone()


# A complete implementation of current DST rules for major US time zones.

def first_sunday_on_or_after(dt):
    days_to_go = 6 - dt.weekday()
    if days_to_go:
        dt += timedelta(days_to_go)
    return dt


# US DST Rules
#
# This is a simplified (i.e., wrong for a few cases) set of rules for US
# DST start and end times. For a complete and up-to-date set of DST rules
# and timezone definitions, visit the Olson Database (or try pytz):
# http://www.twinsun.com/tz/tz-link.htm
# http://sourceforge.net/projects/pytz/ (might not be up-to-date)
#
# In the US, since 2007, DST starts at 2am (standard time) on the second
# Sunday in March, which is the first Sunday on or after Mar 8.
DSTSTART_2007 = datetime(1, 3, 8, 2)
# and ends at 2am (DST time; 1am standard time) on the first Sunday of Nov.
DSTEND_2007 = datetime(1, 11, 1, 1)
# From 1987 to 2006, DST used to start at 2am (standard time) on the first
# Sunday in April and to end at 2am (DST time; 1am standard time) on the last
# Sunday of October, which is the first Sunday on or after Oct 25.
DSTSTART_1987_2006 = datetime(1, 4, 1, 2)
DSTEND_1987_2006 = datetime(1, 10, 25, 1)
# From 1967 to 1986, DST used to start at 2am (standard time) on the last
# Sunday in April (the one on or after April 24) and to end at 2am (DST time;
# 1am standard time) on the last Sunday of October, which is the first Sunday
# on or after Oct 25.
DSTSTART_1967_1986 = datetime(1, 4, 24, 2)
DSTEND_1967_1986 = DSTEND_1987_2006

class USTimeZone(tzinfo):

    def __init__(self, hours, reprname, stdname, dstname):
        self.stdoffset = timedelta(hours=hours)
        self.reprname = reprname
        self.stdname = stdname
        self.dstname = dstname

    def __repr__(self):
        return self.reprname

    def tzname(self, dt):
        if self.dst(dt):
            return self.dstname
        else:
            return self.stdname

    def utcoffset(self, dt):
        return self.stdoffset + self.dst(dt)

    def dst(self, dt):
        if dt is None or dt.tzinfo is None:
            # An exception may be sensible here, in one or both cases.
            # It depends on how you want to treat them.  The default
            # fromutc() implementation (called by the default astimezone()
            # implementation) passes a datetime with dt.tzinfo is self.
            return ZERO
        assert dt.tzinfo is self

        # Find start and end times for US DST. For years before 1967, return
        # ZERO for no DST.
        if 2006 < dt.year:
            dststart, dstend = DSTSTART_2007, DSTEND_2007
        elif 1986 < dt.year < 2007:
            dststart, dstend = DSTSTART_1987_2006, DSTEND_1987_2006
        elif 1966 < dt.year < 1987:
            dststart, dstend = DSTSTART_1967_1986, DSTEND_1967_1986
        else:
            return ZERO

        start = first_sunday_on_or_after(dststart.replace(year=dt.year))
        end = first_sunday_on_or_after(dstend.replace(year=dt.year))

        # Can't compare naive to aware objects, so strip the timezone from
        # dt first.
        if start <= dt.replace(tzinfo=None) < end:
            return HOUR
        else:
            return ZERO

Eastern  = USTimeZone(-5, "Eastern",  "EST", "EDT")
Central  = USTimeZone(-6, "Central",  "CST", "CDT")
Mountain = USTimeZone(-7, "Mountain", "MST", "MDT")
Pacific  = USTimeZone(-8, "Pacific",  "PST", "PDT")
```

注意到存在不可避免的微秒，每年两次在tzinfo子类做标准和白日计数时，在DST转换点。具体来说，认为US 东(UTC -0500)，EDT始于3月的第二个星期日的1:59(DST)之后，结束于11月的第一个星期日的1:59(EDT)之后。

```python
  UTC   3:MM  4:MM  5:MM  6:MM  7:MM  8:MM
  EST  22:MM 23:MM  0:MM  1:MM  2:MM  3:MM
  EDT  23:MM  0:MM  1:MM  2:MM  3:MM  4:MM

start  22:MM 23:MM  0:MM  1:MM  3:MM  4:MM

  end  23:MM  0:MM  1:MM  1:MM  2:MM  3:MM
```

当DST开始时，本地钟表在从1:59跳跃到3:00。格式2:MM的墙上时间并不真正对当天起作用，因此astimezone(Eastern)不会传递一个hour==2的结果在DST开始的那天。为了astimezone()可以有所保证，rzinfo.dst()方法必须认为时间在白天是“丢失了小时”（2:MM for Eastern）。

当DST结束时，有一个潜在的更糟问题：有一个小时不能够以本地墙上时间拼写的不清不楚：白天的最后一个小时。在东区，是格式为5:MM UTC的时间在白日结束时。本地墙上时间从1:59（白日时间）跳回到1:00（标准时间）。1:MM格式的当地时间是模糊的。astimezone()模拟本地钟表的行为通过映射两个相邻的UTC小时到同一个本地小时中。在东区例子中，格式为5:MM的UTC时间以及6:MM都会映射到1:MM当转换成东区。为了astimezone()能够有所保证，tzinfo.dst()方法必须认为时间存在“重复的小时”在标准时间中。这是很简单的安排，就像这个例子中，在时区的标准本地时间表示DST切换时间。

应用不能忍受这些含糊应当避免使用混合的tzinfo子类；当使用UTC时不会含糊不清，或者其他任何修正偏移的tzinfo子类（比如仅表示EST（-5小时偏移）或仅表示EDT（-4小时偏移）的类）。


```python
See also

pytz

    The standard library has no tzinfo instances, but there exists a third-party library which brings the IANA timezone database (also known as the Olson database) to Python: pytz.

    pytz contains up-to-date information and its usage is recommended.
IANA timezone database
    The Time Zone Database (often called tz or zoneinfo) contains code and data that represent the history of local time for many representative locations around the globe. It is updated periodically to reflect changes made by political bodies to time zone boundaries, UTC offsets, and daylight-saving rules.
```

## strftime()和strptime()行为
date,datetime和time对象都支持`strftime(format)`方法，创建一个表示time的字符串在显示的格式化字符串控制下。宽泛来讲，`d.strftime(fmt)`表现如同time模块的`time.strftime(fmt, d.timetuple())`尽管不是所有对象都支持一个timetuple()方法。

相反的，`datetime.strptime()`类方法从一个表示日期和时间的字符串创建一个datetime对象，字符串遵守格式化字符串。`datetime.strptime(date_string, format)` 等同于 `datetime(*(time.strptime(date_string, format)[0:6]))`。

对time对象，年月日的格式化代码应该不被使用，time对象没有这些值。如果他们使用了，1900的1月1日会作为替代品。

对date对象，小时分钟秒和微秒的格式化代码不应该被使用，date对象没有这些值。如果date对象使用了，则全部为0。

完整的format代码集合提供多样的跨平台性，因为python调用平台的C库的strftime()函数，平台变种是常见的。想要查看平台支持的完整格式化代码集合，咨询strftime(3)文档。

下方是所有格式化代码的列表，C标准（1989版本）所需要的，它们工作在标准C实现的平台上。注意到C的1999版本增加了额外的格式化代码。

year的严格范围在strftime()工作时也是跟平台有关。不管平台如何，1900年以前的year不会被使用。
