---
published: true
layout: post
title: Wannacrypt0r蠕虫样本分析
category: Bin-Security
tags: 
  - Security
time: 2017.05.21 13:12:00
excerpt: Wannacrypt0r在上周五爆发，周五的晚上在群里吹比，各种中了勒索软件的消息就刷爆了屏。于是乎，先美滋滋的睡一觉，周六一早拿到样本，操起屠龙刀和倚天剑，对勒索软件进行了初步的解剖。解剖中发现这是个利用了NSA此前泄露的ExternalBlue漏洞利用工具，将exp作为蠕虫扩散的payload，母体释放的dll完成一个复杂而又严谨的加密，感染一系列文件进行勒索。

---

Wannacrypt0r在上周五爆发，周五的晚上在群里吹比，各种中了勒索软件的消息就刷爆了屏。于是乎，先美滋滋的睡一觉，周六一早拿到样本，操起屠龙刀和倚天剑，对勒索软件进行了初步的解剖。解剖中发现这是个利用了NSA此前泄露的ExternalBlue漏洞利用工具，将exp作为蠕虫扩散的payload，母体释放的dll完成一个复杂而又严谨的加密，感染一系列文件进行勒索。


<!--more-->

# Wannacrypt0r蠕虫分析
## 暴力行为分析
对于恶意程序来说，分析之前最简单也最粗暴的做法，就是丢进虚拟机里断网，拍摄快照，然后用行为监控软件诸如MalwareDefender来看看都释放了哪些行为，对恶意程序的执行效果有了了解之后，对下一步的静态分析很有帮助。

MD的log记录了一系列的操作（前方高能，请做好防护）：
```
2017/5/21 13:13:42    创建新进程    允许
进程: c:\windows\explorer.exe
目标: c:\users\administrator\desktop\wannacrypt0r.exe
命令行: "C:\Users\Administrator\Desktop\wannacrypt0r.exe" 
规则: [应用程序]*

2017/5/21 13:14:02    修改其他进程的线程    允许
进程: c:\windows\system32\services.exe
目标: c:\users\administrator\desktop\wannacrypt0r.exe
规则: [应用程序]c:\windows\system32\services.exe

2017/5/21 13:14:12    创建注册表项    允许
进程: c:\windows\system32\services.exe
目标: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\mssecsvc2.0
规则: [注册表组]自动运行程序所在位置 -> [注册表]HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services

2017/5/21 13:14:25    修改注册表值    允许
进程: c:\windows\system32\services.exe
目标: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\mssecsvc2.0\Start
值: 0x00000002(2)
规则: [注册表组]自动运行程序所在位置 -> [注册表]HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\*; Start

2017/5/21 13:14:32    修改注册表值    允许
进程: c:\windows\system32\services.exe
目标: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\mssecsvc2.0\ImagePath
值: C:\Users\Administrator\Desktop\wannacrypt0r.exe -m security
规则: [注册表组]自动运行程序所在位置 -> [注册表]HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\*; ImagePath

2017/5/21 13:14:46    创建新进程    允许
进程: c:\windows\system32\services.exe
目标: c:\users\administrator\desktop\wannacrypt0r.exe
命令行: C:\Users\Administrator\Desktop\wannacrypt0r.exe -m security
规则: [应用程序]*

2017/5/21 13:15:00    修改其他进程的线程    允许
进程: c:\windows\system32\services.exe
目标: c:\users\administrator\desktop\wannacrypt0r.exe
规则: [应用程序]c:\windows\system32\services.exe

2017/5/21 13:15:03    创建新进程    允许
进程: c:\windows\system32\services.exe
目标: c:\windows\system32\taskhost.exe
命令行: taskhost.exe $(Arg0)
规则: [应用程序]*

2017/5/21 13:15:14    删除注册表值    允许
进程: c:\windows\system32\services.exe
目标: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\mssecsvc2.0\FailureCommand
规则: [注册表组]系统设置 -> [注册表]HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\*; FailureCommand

2017/5/21 13:15:17    创建文件    允许
进程: c:\users\administrator\desktop\wannacrypt0r.exe
目标: C:\WINDOWS\tasksche.exe
规则: [文件组]系统执行文件 -> [文件]c:\windows\*; *.exe

2017/5/21 13:15:27    创建新进程    允许
进程: c:\users\administrator\desktop\wannacrypt0r.exe
目标: c:\windows\tasksche.exe
命令行: C:\WINDOWS\tasksche.exe /i
规则: [应用程序]*

2017/5/21 13:15:40    创建文件    允许
进程: c:\windows\tasksche.exe
目标: C:\ProgramData\jdjxwsaumu506\tasksche.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:15:44    修改文件    允许
进程: c:\windows\tasksche.exe
目标: C:\ProgramData\jdjxwsaumu506\tasksche.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:15:50    创建注册表项    允许
进程: c:\windows\system32\services.exe
目标: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\jdjxwsaumu506
规则: [注册表组]自动运行程序所在位置 -> [注册表]HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services

2017/5/21 13:15:56    修改注册表值    允许
进程: c:\windows\system32\services.exe
目标: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\jdjxwsaumu506\Start
值: 0x00000002(2)
规则: [注册表组]自动运行程序所在位置 -> [注册表]HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\*; Start

2017/5/21 13:15:59    修改注册表值    允许
进程: c:\windows\system32\services.exe
目标: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\jdjxwsaumu506\ImagePath
值: cmd.exe /c "C:\ProgramData\jdjxwsaumu506\tasksche.exe"
规则: [注册表组]自动运行程序所在位置 -> [注册表]HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\*; ImagePath

2017/5/21 13:16:06    创建新进程    允许
进程: c:\windows\system32\services.exe
目标: c:\windows\system32\cmd.exe
命令行: cmd.exe /c "C:\ProgramData\jdjxwsaumu506\tasksche.exe"
规则: [应用程序]*

2017/5/21 13:16:09    创建新进程    允许
进程: c:\windows\system32\cmd.exe
目标: c:\programdata\jdjxwsaumu506\tasksche.exe
命令行: C:\ProgramData\jdjxwsaumu506\tasksche.exe
规则: [应用程序]*

2017/5/21 13:16:17    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\ProgramData\jdjxwsaumu506\taskdl.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:16:23    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\ProgramData\jdjxwsaumu506\taskse.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:16:25    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\windows\system32\attrib.exe
命令行: attrib +h .
规则: [应用程序]*

2017/5/21 13:16:29    创建新进程    允许
进程: c:\windows\system32\csrss.exe
目标: c:\windows\system32\conhost.exe
命令行: \??\C:\Windows\system32\conhost.exe "417251964-976739999217293812-1619964307-2134654239-16472526871960537049-1932885429"
规则: [应用程序]*

2017/5/21 13:16:33    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\windows\system32\icacls.exe
命令行: icacls . /grant Everyone:F /T /C /Q
规则: [应用程序]*

2017/5/21 13:16:37    向其他进程复制句柄    允许
进程: c:\windows\system32\conhost.exe
目标: c:\windows\system32\attrib.exe
句柄: (Event) 0x00000064
规则: [应用程序]*

2017/5/21 13:16:39    创建新进程    允许
进程: c:\windows\system32\csrss.exe
目标: c:\windows\system32\conhost.exe
命令行: \??\C:\Windows\system32\conhost.exe "4368683429103864072079618200190476583813685087751441246308961190961-1634784071"
规则: [应用程序]*

2017/5/21 13:16:43    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\programdata\jdjxwsaumu506\taskdl.exe
命令行: taskdl.exe
规则: [应用程序]*

2017/5/21 13:16:49    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\ProgramData\jdjxwsaumu506\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:16:50    结束其他进程    允许
进程: c:\windows\system32\services.exe
目标: c:\windows\system32\cmd.exe
规则: [应用程序]c:\windows\system32\services.exe

2017/5/21 13:16:54    向其他进程复制句柄    允许
进程: c:\windows\system32\conhost.exe
目标: c:\windows\system32\icacls.exe
句柄: (Event) 0x00000064
规则: [应用程序]*

2017/5/21 13:16:57    修改文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\ProgramData\jdjxwsaumu506\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:16:58    修改文件权限    允许
进程: c:\windows\system32\icacls.exe
目标: C:\ProgramData\jdjxwsaumu506\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:01    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\ProgramData\jdjxwsaumu506\277441495343817.bat
规则: [文件组]所有执行文件 -> [文件]*; *.bat

2017/5/21 13:17:02    修改文件权限    允许
进程: c:\windows\system32\icacls.exe
目标: C:\ProgramData\jdjxwsaumu506\taskdl.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:04    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\windows\system32\cmd.exe
命令行: cmd /c 277441495343817.bat
规则: [应用程序]*

2017/5/21 13:17:05    修改文件权限    允许
进程: c:\windows\system32\icacls.exe
目标: C:\ProgramData\jdjxwsaumu506\tasksche.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:06    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\~SD787.tmp
规则: [文件]?:\

2017/5/21 13:17:07    创建新进程    允许
进程: c:\windows\system32\csrss.exe
目标: c:\windows\system32\conhost.exe
命令行: \??\C:\Windows\system32\conhost.exe "-447381410-530109417-1550786138-1532135549-105676804810498169953338964741732086072"
规则: [应用程序]*

2017/5/21 13:17:08    修改文件权限    允许
进程: c:\windows\system32\icacls.exe
目标: C:\ProgramData\jdjxwsaumu506\taskse.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:10    修改文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\~SD787.tmp
规则: [文件]?:\

2017/5/21 13:17:11    向其他进程复制句柄    允许
进程: c:\windows\system32\conhost.exe
目标: c:\windows\system32\cmd.exe
句柄: (Event) 0x00000064
规则: [应用程序]*

2017/5/21 13:17:12    修改文件权限    允许
进程: c:\windows\system32\icacls.exe
目标: C:\ProgramData\jdjxwsaumu506\277441495343817.bat
规则: [文件组]所有执行文件 -> [文件]*; *.bat

2017/5/21 13:17:13    删除文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\~SD787.tmp
规则: [文件]?:\

2017/5/21 13:17:14    创建文件    允许
进程: c:\windows\system32\cmd.exe
目标: C:\ProgramData\jdjxwsaumu506\m.vbs
规则: [文件组]所有执行文件 -> [文件]*; *.vbs

2017/5/21 13:17:15    修改文件权限    允许
进程: c:\windows\system32\icacls.exe
目标: C:\ProgramData\jdjxwsaumu506\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:16    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\header.jpg.WNCRYT
规则: [文件]?:\

2017/5/21 13:17:17    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\programdata\jdjxwsaumu506\taskdl.exe
命令行: taskdl.exe
规则: [应用程序]*

2017/5/21 13:17:18    修改文件    允许
进程: c:\windows\system32\cmd.exe
目标: C:\ProgramData\jdjxwsaumu506\m.vbs
规则: [文件组]所有执行文件 -> [文件]*; *.vbs

2017/5/21 13:17:18    修改文件权限    允许
进程: c:\windows\system32\icacls.exe
目标: C:\ProgramData\jdjxwsaumu506\taskdl.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:19    修改文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\header.jpg.WNCRYT
规则: [文件]?:\

2017/5/21 13:17:20    修改文件    允许
进程: c:\windows\system32\cmd.exe
目标: C:\ProgramData\jdjxwsaumu506\m.vbs
规则: [文件组]所有执行文件 -> [文件]*; *.vbs

2017/5/21 13:17:21    修改文件权限    允许
进程: c:\windows\system32\icacls.exe
目标: C:\ProgramData\jdjxwsaumu506\tasksche.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:21    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\header.jpg.WNCRY
规则: [文件]?:\

2017/5/21 13:17:22    修改文件    允许
进程: c:\windows\system32\cmd.exe
目标: C:\ProgramData\jdjxwsaumu506\m.vbs
规则: [文件组]所有执行文件 -> [文件]*; *.vbs

2017/5/21 13:17:23    修改文件权限    允许
进程: c:\windows\system32\icacls.exe
目标: C:\ProgramData\jdjxwsaumu506\taskse.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:23    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\@Please_Read_Me@.txt
规则: [文件]?:\

2017/5/21 13:17:24    创建新进程    允许
进程: c:\windows\system32\cmd.exe
目标: c:\windows\system32\cscript.exe
命令行: cscript.exe  //nologo m.vbs
规则: [应用程序]*

2017/5/21 13:17:26    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\@WanaDecryptor@.exe
规则: [文件]?:\

2017/5/21 13:17:27    向其他进程复制句柄    允许
进程: c:\windows\system32\conhost.exe
目标: c:\windows\system32\cscript.exe
句柄: (Event) 0x00000064
规则: [应用程序]*

2017/5/21 13:17:28    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\IDA 6.8\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:28    删除文件    允许
进程: c:\windows\system32\cmd.exe
目标: C:\ProgramData\jdjxwsaumu506\m.vbs
规则: [文件组]所有执行文件 -> [文件]*; *.vbs

2017/5/21 13:17:29    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\IDA 6.8\ids\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:30    删除文件    允许
进程: c:\windows\system32\cmd.exe
目标: C:\ProgramData\jdjxwsaumu506\277441495343817.bat
规则: [文件组]所有执行文件 -> [文件]*; *.bat

2017/5/21 13:17:31    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\IDA 6.8\plugins\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:37    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\IDA 6.8\plugins\hexrays_sdk\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:38    修改文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\header.jpg
规则: [文件]?:\

2017/5/21 13:17:40    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\Python27\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:51    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\Python27\include\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:52    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\programdata\jdjxwsaumu506\taskdl.exe
命令行: taskdl.exe
规则: [应用程序]*

2017/5/21 13:17:53    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\Python27\Lib\idlelib\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:55    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\Python27\Lib\lib2to3\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:56    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\Python27\Lib\site-packages\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:58    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\Python27\Lib\test\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:17:59    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\Python27\tcl\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:18:03    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\Python27\Tools\pynche\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:18:04    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\Python27\Tools\Scripts\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:18:05    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\Python27\Tools\versioncheck\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:18:09    创建文件    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: C:\Python27\Tools\webchecker\@WanaDecryptor@.exe
规则: [文件组]所有执行文件 -> [文件]*; *.exe

2017/5/21 13:21:57    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\programdata\jdjxwsaumu506\taskdl.exe
命令行: taskdl.exe
规则: [应用程序]*

2017/5/21 13:22:09    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\programdata\jdjxwsaumu506\taskse.exe
命令行: taskse.exe C:\ProgramData\jdjxwsaumu506\@WanaDecryptor@.exe
规则: [应用程序]*

2017/5/21 13:22:12    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\taskse.exe
目标: c:\programdata\jdjxwsaumu506\@wanadecryptor@.exe
命令行: "C:\ProgramData\jdjxwsaumu506\@WanaDecryptor@.exe"
规则: [应用程序]*

2017/5/21 13:22:33    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\programdata\jdjxwsaumu506\taskdl.exe
命令行: taskdl.exe
规则: [应用程序]*

2017/5/21 13:22:42    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\programdata\jdjxwsaumu506\taskse.exe
命令行: taskse.exe C:\ProgramData\jdjxwsaumu506\@WanaDecryptor@.exe
规则: [应用程序]*

2017/5/21 13:22:43    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\taskse.exe
目标: c:\programdata\jdjxwsaumu506\@wanadecryptor@.exe
命令行: "C:\ProgramData\jdjxwsaumu506\@WanaDecryptor@.exe"
规则: [应用程序]*

2017/5/21 13:23:07    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\programdata\jdjxwsaumu506\taskdl.exe
命令行: taskdl.exe
规则: [应用程序]*

2017/5/21 13:23:28    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\programdata\jdjxwsaumu506\taskse.exe
命令行: taskse.exe C:\ProgramData\jdjxwsaumu506\@WanaDecryptor@.exe
规则: [应用程序]*

2017/5/21 13:23:30    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\taskse.exe
目标: c:\programdata\jdjxwsaumu506\@wanadecryptor@.exe
命令行: "C:\ProgramData\jdjxwsaumu506\@WanaDecryptor@.exe"
规则: [应用程序]*

2017/5/21 13:23:38    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\programdata\jdjxwsaumu506\taskdl.exe
命令行: taskdl.exe
规则: [应用程序]*

2017/5/21 13:24:02    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\programdata\jdjxwsaumu506\taskse.exe
命令行: taskse.exe C:\ProgramData\jdjxwsaumu506\@WanaDecryptor@.exe
规则: [应用程序]*

2017/5/21 13:24:07    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\taskse.exe
目标: c:\programdata\jdjxwsaumu506\@wanadecryptor@.exe
命令行: "C:\ProgramData\jdjxwsaumu506\@WanaDecryptor@.exe"
规则: [应用程序]*

2017/5/21 13:24:11    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\programdata\jdjxwsaumu506\taskdl.exe
命令行: taskdl.exe
规则: [应用程序]*

2017/5/21 13:24:34    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\programdata\jdjxwsaumu506\taskse.exe
命令行: taskse.exe C:\ProgramData\jdjxwsaumu506\@WanaDecryptor@.exe
规则: [应用程序]*

2017/5/21 13:24:36    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\taskse.exe
目标: c:\programdata\jdjxwsaumu506\@wanadecryptor@.exe
命令行: "C:\ProgramData\jdjxwsaumu506\@WanaDecryptor@.exe"
规则: [应用程序]*

2017/5/21 13:24:44    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\programdata\jdjxwsaumu506\taskdl.exe
命令行: taskdl.exe
规则: [应用程序]*

2017/5/21 13:25:01    安装全局消息钩子    允许
进程: c:\windows\explorer.exe
目标: c:\windows\explorer.exe
钩子类型: WH_KEYBOARD_LL
规则: [应用程序]c:\windows\explorer.exe

2017/5/21 13:25:07    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\programdata\jdjxwsaumu506\taskse.exe
命令行: taskse.exe C:\ProgramData\jdjxwsaumu506\@WanaDecryptor@.exe
规则: [应用程序]*

2017/5/21 13:25:09    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\taskse.exe
目标: c:\programdata\jdjxwsaumu506\@wanadecryptor@.exe
命令行: "C:\ProgramData\jdjxwsaumu506\@WanaDecryptor@.exe"
规则: [应用程序]*

2017/5/21 13:25:18    创建新进程    允许
进程: c:\programdata\jdjxwsaumu506\tasksche.exe
目标: c:\programdata\jdjxwsaumu506\taskdl.exe
命令行: taskdl.exe
规则: [应用程序]*
```

那么可以看到，wannacrpyt0r在执行时首先会注册一些服务，以较为经典的手法在注册表中加入自启动项，其后又以`-m security`的方式再次执行本体，这一次母体wannacrypt0r会首先释放子文件tasksche到C:\windows目录下。母体随后会启动放在windows目录下的tasksche.exe,参数为`/i`。

tasksche会复制自己到C:\ProgramData\jdjxwsaumu506目录下，该目录名看起来是随机生成的，其后tasksche对复制后的本体文件进行了修改操作，与此前用同样的手法，创建了对应的自启动的注册表项。随后复制的进程会再次通过services.exe调cmd启动，即`cmd.exe /c "C:\ProgramData\jdjxwsaumu506\tasksche.exe"`。此时tasksche会释放子体taskdl.exe以及taskse.exe放在同目录下。其后利用attrib.exe修改了文件属性为隐藏，达到隐蔽效果。此后看起来就是加密过程了，tasksche还会释放@WanaDecryptor@.exe程序，这个程序就是最后弹出那个勒索对话框的PE文件。tasksche还会释放一个277441495343817.bat批处理文件，后面通过cmd调用，应该是和后面调用icacls.exe有关，这个文件使用后会被删除。后面还有一个m.vbs脚本，看起来也是和修改文件权限有关。

此后一个运行中的@WanaDecryptor@.exe就会定时弹出烦人的勒索对话框，由于这里我关闭了网络，所以此前已经掌握的利用SMB协议发送ExternalBlue利用代码的行为没有监视到。最后的一些截图：

![](/img/posts/Security/Wannacry/20170521_1.jpg)
![](/img/posts/Security/Wannacry/20170521_2.jpg)
![](/img/posts/Security/Wannacry/20170521_3.jpg)

## 代码分析
实际上，勒索蠕虫通过上面的行为监控，其大体上的行为已经很清晰了，但对于其具体的加密方式以及此前屏蔽掉的扩散方式还不了解，所以，下一步我们需要对程序进行静态分析。

一般来说，静态分析之前，要先看程序是否有加壳，是否有混淆。该勒索软件的作者非常自信，没有加任何壳，也没有用LLVM这种恶心的混淆框架来混淆代码，综合所有的分析，其大概缘由如下：
1. 作者是小学生，学了点技术就来搞事情？经过对后续代码的分析，发现其编程非常规范，加密手法也非常符合密码学的标准，显然不是坐井观天的菜鸡。
2. 加壳会更易被各类防护软件查杀。
3. 至于混淆，毕竟作者也是个帽子，不想给同行增加徒劳的体力工作吧。

因此，对于该程序来说，直接拖进IDA里定位就可以了：

![](/img/posts/Security/Wannacry/20170521_4.jpg)

定位到WinMain入口，直接F5大法，发现一个非常奇怪的URL字符串，后面可以看到如果访问的到这一URL，则停止感染，如果没有则继续。所以这一URL是一个域名，作为作者自己保留的开关，当域名被注册时，扩散的蠕虫就不再进行感染工作了。后来听说这一域名被英国的一个小伙子无意中注册了，然而我是不信的，如此随机化的域名，想必小伙子也是同道中人。

![](/img/posts/Security/Wannacry/20170521_5.jpg)

根据条件判断：
```c
 if ( v5 )
  {
    InternetCloseHandle(v4);
    InternetCloseHandle(v5);
    result = 0;
  }
  else
  {
    InternetCloseHandle(v4);
    InternetCloseHandle(0);
    sub_408090();
    result = 0;
  }
```
函数sub_408090就是关键的call了。

![](/img/posts/Security/Wannacry/20170521_6.jpg)

这里可以看到和此前的行为监控一致，函数会通过argc参数的个数进行判断，小于2个也就是第一次执行的时候，会进入if分支，而此后的一次-m security的方式会进入else分支。我们先看if分支，这里是经典的SCManager注册服务的代码，其功能就对应了此前的log所记载的行为。我们再关注else分支，进入到sub_407F20()中查看：

![](/img/posts/Security/Wannacry/20170521_7.jpg)

可以看到函数内顺序调用了两个函数，先跟进第一个：

![](/img/posts/Security/Wannacry/20170521_8.jpg)

这里我将C代码copy到汇编，方便查看注册的服务名称，这个名称就是最终查看进程时其对应的服务描述。这里的行为就是创建服务名为'mssecsvc2.0'，描述为"Microsoft Security Center (2.0) Service"的服务，伪装成Microsoft的某个服务。在后续行为中会启动自身。

实际上，后面可以看到，这一服务的功能就是扫描互联网IP，利用SMB的内核漏洞（MS17-010）传播本体。传播的工具使用的是ExternalBlue，其具体的原理我还未作分析，日后会对该漏洞进行调试。感染的细节也很常规，就是简单的将payload释放到mssecsvc.exe中，然后通过启动线程向构造的IP地址进行发送。这里构造IP很有意思，LAN段则直接构造局域网IP，而WAN段则随机构造（也是醉了，看看谁倒霉。。。）。

我们接下来主要关心加密。

![](/img/posts/Security/Wannacry/20170521_9.jpg)

继续跟进，会发现是对资源的释放过程，其中对文件的操作API函数都采用GetProcAddr的方式获取，这是躲过防护软件追杀的一种方式。释放的文件是
tasksche.exe，是硬编码在.data段的。

![](/img/posts/Security/Wannacry/20170521_10.jpg)

这里的dword_431478就是此前通过GetProcAddr拿到的CreateProcessA函数，这里创建的进程`tasksche -i `

继续跟踪到tasksche.exe中，该程序此前在行为监控时，我copy了副本。

![](/img/posts/Security/Wannacry/20170521_11.jpg)

定位到WinMain，可以看到大部分的文件操作都在主函数中，此前行为分析已经看到了，所以就不再赘述，我们关心的是如何加密。

再继续跟代码，发现依然是创建服务，并以服务的方式再次启动自身。

这里说一下，执行的
```
attrib.exe +h
icacls.exe . /grant Everyone:F /T /C /Q
```
是用来修改文件的隐藏属性以及权限。我们跟到sub_4014A6中，前面的几个函数就不再展开了，和log中的行为相对应。

```c
int __thiscall sub_4014A6(void *this, LPCSTR lpFileName, int a3)
{
  void *v3; // esi@1
  int v4; // ebx@1
  HANDLE v5; // edi@1
  size_t Size; // [sp+14h] [bp-244h]@1
  int v8; // [sp+18h] [bp-240h]@1
  char Buf1; // [sp+1Ch] [bp-23Ch]@1
  int v10; // [sp+1Dh] [bp-23Bh]@1
  __int16 v11; // [sp+21h] [bp-237h]@1
  char v12; // [sp+23h] [bp-235h]@1
  __int64 dwBytes; // [sp+24h] [bp-234h]@9
  char Dst; // [sp+2Ch] [bp-22Ch]@11
  int v15; // [sp+22Ch] [bp-2Ch]@1
  int v16; // [sp+230h] [bp-28h]@12
  LARGE_INTEGER FileSize; // [sp+234h] [bp-24h]@2
  int v18; // [sp+23Ch] [bp-1Ch]@1
  CPPEH_RECORD ms_exc; // [sp+240h] [bp-18h]@1

  v3 = this;
  v4 = 0;
  v15 = 0;
  Size = 0;
  Buf1 = 0;
  v10 = 0;
  v11 = 0;
  v12 = 0;
  v8 = 0;
  v18 = 0;
  ms_exc.registration.TryLevel = 0;
  v5 = CreateFileA(lpFileName, 0x80000000, 1u, 0, 3u, 0, 0);
  if ( v5 != (HANDLE)-1 )
  {
    GetFileSizeEx(v5, &FileSize);
    if ( FileSize.QuadPart <= 104857600 )
    {
      if ( dword_40F880(v5, &Buf1, 8, &v18, 0) )
      {
        if ( !memcmp(&Buf1, aWanacry, 8u) )
        {
          if ( dword_40F880(v5, &Size, 4, &v18, 0) )
          {
            if ( Size == 256 )
            {
              if ( dword_40F880(v5, *((_DWORD *)v3 + 306), 256, &v18, 0) )
              {
                if ( dword_40F880(v5, &v8, 4, &v18, 0) )
                {
                  if ( dword_40F880(v5, &dwBytes, 8, &v18, 0) )
                  {
                    if ( dwBytes <= 104857600 )
                    {
                      if ( sub_4019E1(*((void **)v3 + 306), Size, &Dst, (int)&v15) )
                      {
                        sub_402A76((int)&Dst, Src, v15, 0x10u);
                        v16 = (int)GlobalAlloc(0, dwBytes);
                        if ( v16 )
                        {
                          if ( dword_40F880(v5, *((_DWORD *)v3 + 306), FileSize.LowPart, &v18, 0)
                            && v18
                            && (SHIDWORD(dwBytes) < 0 || SHIDWORD(dwBytes) <= 0 && v18 >= (unsigned int)dwBytes) )
                          {
                            v4 = v16;
                            sub_403A77(*((void **)v3 + 306), v16, v18, 1);
                            *(_DWORD *)a3 = dwBytes;
                          }
                        }
                      }
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
  local_unwind2(&ms_exc.registration, -1);
  return v4;
}
```
这里会释放一个t.wnry文件，对内容进行解密，解密后内容是一个dll，该dll不会写入磁盘，而是直接在内存中执行。而这个隐藏的如此好的dll显然是一个关键的作者不想示人的文件。

![](/img/posts/Security/Wannacry/20170521_12.jpg)

后面会直接跟进dll中的函数，TaskStart函数，TaskStart函数中会先通过GetProcAddress的方式拿到文件相关的API函数，然后连续创建了几个线程，几个线程各司其职。分别处理tasksc.exe注册表，检查驱动器，轮询taskdl.exe和执行m.vbs(动态调试后被我恢复快照了，OD分析的截图复原后就没了，有空再跟一次补上吧。。。我就简单的总结一下吧，有兴趣可以自己分析，没什么难度)。

而加密过程也很简单，就是简单的遍历文件，进行判断并对文件进行加密，其加密的流程非常严谨，也较为复杂，其所用的加密函数都是系统的API。

加密流程如下：
1. 首先生成本地的RSA2048一对秘钥A1/A2，导出公钥A1到00000000.pky中，用硬编码的公钥B1加密私钥A2保存到00000000.eky中。
2. 遍历文件，如果后缀满足要求（定义在dll的data段, .doc,.xlt,.ppt等茫茫多的后缀数组），使用CryptGenRandom()函数生成AES秘钥，128位。
3. 使用A1加密AES秘钥，并用AES秘钥加密文件，清除内存中的AES Key。
4. 将WANACRY!标记，加密后的AES Key和加密文件内容写入到新的同名文件中，后缀改为.wnry。这里的方式是删除旧文件，修改新文件属性和旧文件一致。

## 总结
1. 在我们了解了加密过程后，就会发现其加密手法相当严谨，由于私钥B2不公开，所以无法解密用B1加密的私钥A2，而私钥A2拿不到，也就无法解密被A1加密的AES key。加密流程环环相扣，没有弱点。
2. 之所以采用两次RSA加密的方式，是为了解密时便于分发RSA A2，而不用麻烦的给AES秘钥，更不会因为一个人交赎金而公开B2。另一方面，勒索软件为了让你信服，还会挑选一些文件可以免费恢复，跟踪时发现其A2没有被B1加密，所以可以直接解密。
3. 在AES加密后，立即清除了内存中的AES Context，可以最大限度降低AES key被曝光的周期，可以看出作者的思路非常的清奇、严谨。
4. 唯一的问题在于文件是新生成的，删除了旧文件而不是修改原有的旧文件，所以如果被加密的文件所在的磁盘文件不多，且运气好的话，有概率通过数据恢复软件找到被删除的原文件。

总之，这个恶意程序的逆向分析不具有什么难度，倒是在用F5大法时发现其代码非常规范。一般来说，帽子们的编程能力较之工程师稍逊一筹，写的代码往往也只追求简单有疗效，这份代码倒是安全人士少有的清流:)。

最后还是想说一句，用MS17-010做勒索软件，大材小用，有点可惜了。