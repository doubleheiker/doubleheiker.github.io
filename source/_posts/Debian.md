---
title: Install&beautify Debian 64-bit on computer
date: 2019-01-31 19:18:35
tags: [SystemInstall]
categories: [SystemInstall]
description: <center>双系统安装64位Debian系统</center>
copyright: true
photo: "http://r.photo.store.qq.com/psb?/V13BnVsC23MbK8/izIa10wmArjY*7Rcfl9g885BUfgmdwUnRZHhSZLBiXs!/r/dFMBAAAAAAAA"
---
---

# 制作U盘启动盘

首先使用格式化工具（我使用的是Diskgenius）将U盘彻底格式化，使用UltraISO将debian系统烧录进U盘
{% asset_img 烧录系统.png %}
{% asset_img 烧录系统2.png %}
{% asset_img 烧录系统3.png 选择USB-HDD+%}

# 划分硬盘空间

使用Diskgenius将硬盘划分出一个50.86G的未分配空间用于Debian系统的安装
{% asset_img 划分.png %}

# 在BIOS设置U盘启动优先

在电脑开机的时候进入BIOS界面，将开机启动选项选择到EFI启动优先，并将安全选项中security boot关闭，保存选项后重启电脑
{% asset_img IMG_20190213_161345.jpg %}

# 安装Debian 9.7

进入安装界面，选择图像化安装
{% asset_img IMG_20190213_161603.jpg %}

语言一路默认
{% asset_img IMG_20190213_161603.jpg %}
{% asset_img IMG_20190213_161623.jpg %}
{% asset_img IMG_20190213_161639.jpg %}
{% asset_img IMG_20190213_162345.jpg %}

主机名默认，设置root密码和用户密码
{% asset_img IMG_20190213_162411.jpg %}
{% asset_img IMG_20190213_162437.jpg %}
{% asset_img IMG_20190213_162513.jpg %}

磁盘分区，选择划分出来的的空闲空间
{% asset_img IMG_20190213_162923.jpg %}

选择`Automatically partion the free space`
{% asset_img IMG_20190213_163442.jpg %}

选择`All files in one partition`
{% asset_img IMG_20190213_163509.jpg %}
{% asset_img IMG_20190213_163528.jpg %}

等待安装，出现配置network时，选择`yes`
{% asset_img IMG_20190213_164145.jpg %}

我选择了科大源
{% asset_img IMG_20190213_164222.jpg %}

等待安装完成，电脑重新启动时拔出U盘，进入Debian系统
{% asset_img IMG_20190213_175838.jpg %}

# 美化系统

在这里超级推荐一个网站：
<https://www.gnome-look.org/>
我是参照一篇博主的文章对Debian系统进行主题美化，MacX样式，
以下是链接：
<https://blog.csdn.net/zyqblog/article/details/80152016>