---
published: true
layout: post
title: Linux kernel beginners - syscall篇
category: Bin-Security
tags: 
  - linux-kernel
time: 2017.05.30 19:08:00
excerpt: 对于linux内核exploit的调试，始终不得要领，终于挖到了muhe师傅和swing师傅的linux kernel beginners系列的几篇文章。后来了解到，最早是joker师傅指引，令我等后来人能够如是复盘，如获至宝。

---

对于linux内核exploit的调试，始终不得要领，终于挖到了muhe师傅和swing师傅的linux kernel beginners系列的几篇文章。后来了解到，最早是joker师傅指引，令我等后来人能够如是复盘，如获至宝。

<!--more-->

# Linux kernel beginners - syscall
这一篇主要讲环境的搭建与配置、syscall的添加与测试。内核版本选择2.6.32.1，采用qemu来调试内核，宿主机系统为Ubuntu 14.04 x86。

> 我在两个系统里尝试过，另一个是Ubuntu Kylin 14.04 x86。在Ubuntu 16.04 x64上由于内核版本过老，所以编译时问题太多而暂时没有继续下去。

## 环境搭建
第一步自然是需要获取内核源码，URL为：https://www.kernel.org/pub/linux/kernel/v2.6/linux-2.6.32.1.tar.gz

如果没有vpn或代理的话，可能下载速度很慢，由于我开了lantern vip，所以秒下，如果读者有需要的话，可以找我要一份。

解包：`tar -zvxf linux-2.6.32.1.tar.gz`

由于menuconfig需要libncurses5-dev，所以首先要安装libncurses5-dev，在ubuntu下可以：

```
sudo apt-get install libncurses5-dev
```

我在Ubuntu 14.04使用时，由于源的问题，存在依赖关系的问题，对此，可以通过如下方式尝试修正：

```
sudo apt-get -f install
sudo apt-get install --resinstall name=version
```
如果依赖耦合过度，则直接下载libncurses5-dev以及其依赖的libncurses5和ncurses5-bin。

此外，还需要qemu：

```
sudo apt-get install qemu qemu-system
```

这之后，可以进行对内核的编译：

```
make menuconfig
make 
make all
make modules
```

编译内核时，会有几个问题，这几个问题在swing的博文上都有说明，请参考附录链接。编译时产生的错误，都是因为新旧工具的兼容性问题，有兴趣的可以一一查阅。

## 增加syscall
对于内核来说，系统调用是尤为重要的，我们添加两个自定义的系统调用并测试一下。

对于linux内核来说，添加系统调用有3个位置需要处理：
1. arch/x86/kernel/syscall_table_32.S	在系统调用表中添加序号索引

![](/img/posts/Security/linux-kernel-beginners/syscall_table.jpg)

增加两个新的系统调用，序号顺延。

2. arch/x86/include/asm/unistd_32.h		在unistd的32位库中添加系统调用序列号的宏

![](/img/posts/Security/linux-kernel-beginners/unistd_32.jpg)

定义新系统调用宏，其值为在syscall_table.S中的索引号。注意要相应递增下面的NR_syscalls宏。

3. include/linux/syscalls.h				在syscalls.h中给出系统调用的声明

![](/img/posts/Security/linux-kernel-beginners/syscalls.jpg)

声明这两个新的系统调用。

4. 定义自己的系统调用，修改主Makefile，在linux根目录下增加r00tk1t_test目录，建r00tk1t_test.c文件以及Makefile：

![](/img/posts/Security/linux-kernel-beginners/r00tk1t_test.jpg)

![](/img/posts/Security/linux-kernel-beginners/linux-makefile.jpg)

重新Make内核。

## busybox编译
使用busybox是为了方便的添加文件到文件系统中，这东西大名鼎鼎，不知道的自己百度。

下载busybox-1.22.1：https://busybox.net/downloads/busybox-1.22.1.tar.bz2。老规矩：

```
make menuconfig
make
make install
```

在menuconfig中勾选上静态库，避免后续的各种动态库缺失的问题：

![](/img/posts/Security/linux-kernel-beginners/busybox-menuconfig.jpg)

此后，对busybox的文件系统做些配置。

```
$ cd _install
$ mkdir -pv {bin,sbin,etc,proc,sys,usr/{bin,sbin}}
$ cat init
#!/bin/sh
echo "INIT SCRIPT"
mount -t proc none /proc
mount -t sysfs none /sys
mount -t debugfs none /sys/kernel/debug
mkdir /tmp
mount -t tmpfs none /tmp
mdev -s # We need this to find /dev/sda later
echo -e "\nBoot took $(cut -d' ' -f1 /proc/uptime) seconds\n"
exec /bin/sh
$ chmod +x init
$ find . -print0 \
    | cpio --null -ov --format=newc \
    | gzip -9 > /tmp/initramfs-busybox-x86.cpio.gz
$ qemu-system-i386 -kernel arch/i386/boot/bzImage -initrd /tmp/initramfs-busybox-x86.cpio.gz
```

此时就可以用生成的文件系统测试linux内核了：`qemu-system-i386 -kernel arch/i386/boot/bzImage -initrd /tmp/initramfs-busybox-x86.cpio.gz`。

## 测试syscall
编写一个二进制程序，测试syscall：

![](/img/posts/Security/linux-kernel-beginners/syscall_test.jpg)

编译时要加-static参数静态编译，依然是绕开各种的动态库缺失问题。`gcc r00tk1t.c -o r00tk1t -static`

将编译好的二进制r00tk1t拷贝到busybox下的_install/usr/下，然后重新生成文件系统，依然是:

```
$ find . -print0 \
    | cpio --null -ov --format=newc \
    | gzip -9 > /tmp/initramfs-busybox-x86.cpio.gz
```

注意，每次有了文件系统的改动，initramfs-busybox-x86.cpio.gz都需要重新生成，并用新的文件系统重启qemu。

启动qemu后就可以执行usr下的r00tk1t了。

qemu出了点毛病，待续。。