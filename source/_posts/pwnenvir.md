---
title: pwn environment
date: 2019-08-25 16:14:26
tags: [pwn,IDA]
categories: [pwn]
copyright: true
description: <center>pwn环境配置</center>
photo: "http://r.photo.store.qq.com/psb?/V13BnVsC23MbK8/sDGWpZOjlyQ1ijd**.PFLsdd2khXFW3eorAdsnFgo2k!/r/dL4AAAAAAAAA"
---
---

> 国内网上关于pwn的知识零零散散，pwn门槛高、难入门，而且关于pwn环境的搭建以及IDA的使用更是没有系统的教程，或者是教程老旧过时。此次环境搭建，基于虚拟机上的Ubuntu18.04的32位和64位系统、pwntools，与Windows上的IDA7.0进行远程调试，是根据i春秋[Linux pwn入门教程系列](https://bbs.ichunqiu.com/forum.php?mod=collection&action=view&ctid=157)进行环境的配置和补漏。这期间踩了许多坑，原本尝试在docker上使用32位和64位系统来进行远程调试，但是IDA远程时出现：The file can‘t be loaded by the debugger plugin，尝试了许多解决方案，未能解决，于是只能使用两个虚拟环境来搭建，如果有大佬知道如何解决，请不吝赐教，感谢！

---

# IDA远程调试配置

在IDA所在的文件夹的`dbgsrv`文件夹下找到需要的调试服务器`linux_server(32位)`和`linux_serverx64(64位)`并复制到Ubuntu主目录中。如果没有可执行权限，则需要给这两个文件赋予可执行权限：

```bash
chmod 777 ./linux_server
chmod 777 ./linux_serverx64
```

然后执行命令：

```bash
./linux_server
```

此时我们可以看到Linux_server的版本为1.22、正在监听23946端口。

{% asset_img linux_server.png %}

接着打开32位的ida，载入heapTest_x86，在左侧的Functions window中找到main函数，随便挑一行代码按F2下一个断点。在Debugger中选择Remote Linux debugger，然后通过`Debugger->Process options...`打开选项窗口设置远程调试选项。Hostname就是Ubuntu的ip地址：

{% asset_img ip.png %}

{% asset_img ida.png %}

密码就是Ubuntu的密码，填写完成后点击OK，按F9快捷键运行程序。若连接正常可能提示Input file is missing:xxxxx，一路OK就行，IDA会将被调试的文件复制到服务器所在目录下，然后汇编代码所在窗口背景会变成浅蓝色并且窗口布局发生变化。

{% asset_img ida1.png %}

F8：单步跨入函数，F7：单步不跨入函数，F4：运行到指定位置。随着程序的调试执行，我们可以看到运行调试服务器的shell窗口会显示出新的内容：

{% asset_img ida2.png %}

# 使用pwntools和IDA调试程序

首先我们需要安装pwntools：

```bash
sudo pip install pwntools
```

其官方文档地址：<http://docs.pwntools.com/en/stable/ >

将heapTest_x86导入到Ubuntu主目录中，然后执行命令：

```
socat tcp-listen:10001,reuseaddr,fork EXEC:./heapTest_x86,pty,raw,echo=0
```

将heapTest_x86的IO转发到10001端口上。

{% asset_img socat.png %}

然后运行python，使用`from pwn import *`导入pwntools库。然后使用`io = remote("192.168.112.134", 10001)`与heapTest_x86连接。这个时候我们返回到IDA中设置断点。需要注意的是此时heapTest_x86已经开始运行，我们的目标是附加到其运行的进程上，所以我们需要把断点设置在`call    ___isoc99_scanf`等等待输入的指令运行顺序之后，否则由于计算机的运行速度，我们的断点将会因为已经目标指令已经执行完而失效，达不到断下来的效果。

{% asset_img ida3.png %}

选择`Debugger->Attach to process...`，附加到./heapTest_x86的进程上

{% asset_img ida4.png %}

然后IDA进入到调试模式。

{% asset_img ida5.png %}

这几行指令实际上是执行完sys_read后的指令，此处我们不需要关心它，直接按F9，选中标志会消失。回到python窗口，我们使用pwntools的recv/send函数族来与运行中的heapTest_x86进行交互。首先输入io.recv()，我们发现原先会在shell窗口出现的菜单被读出到python窗口里了。

{% asset_img pwntools.png %}

然后使用`io.sendline('1')`选择1，然后回到IDA，我们就发现到了断点处了。

{% asset_img ida6.png %}

当我们希望结束调试时，应该使用`io.close()`关闭掉这个io。否则下一次试图attach时会发现有两个`./heapTest_x86`进程。在IDA中按`Ctrl+F2`即可退出调试模式。

# 疑难解答

·如果在IDA中调试时，按下F9，出现`Incompatible debugging server:protocol version is 19,expected 22`类似错误，那么就是linux_sever的版本与IDA的版本不相符，可以重新下载一个IDA，然后将其linux_sever放入Ubuntu中运行查看版本，找到对应版本即可。

·pwntools下载到一半出错，检查下pip更新，下载出错多下载几次就可以了，玄学问题，估计就是网络原因。