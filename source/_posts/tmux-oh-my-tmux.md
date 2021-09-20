---
title: tmux+oh-my-tmux
date: 2019-03-13 19:17:39
tags: [Linux,bash]
categories: [Linux]
copyright: true
description: <center>使用tmux让你爱上终端吧！</center>
photo: "http://r.photo.store.qq.com/psb?/V13BnVsC23MbK8/PmiQwtN48wPrbQ*zWv6mbzH*jHl4635ONGQFWloSnGM!/r/dL4AAAAAAAAA"
---
---

> tmux的官方介绍：tmux is a terminal multiplexer: it enables a number of terminals to be created, accessed, and controlled from a single screen. tmux may be detached from a screen and continue running in the background, then later reattached. 总之，tmux可以使你的终端使用体验有极大的提升，还不赶紧来试试！

# 安装使用

首先给出githuib地址：[tmux](https://github.com/tmux/tmux)
各个系统下的安装命令：

```bash
brew install tmux       # OSX
pacman -S tmux          # archlinux
apt-get install tmux    # Ubuntu
yum install tmux        # Centos
```

安装完成后，赶紧试试tmux吧：

```bash
tmux                    #默认名字启用tmux
tmux new -s name        #指定session的名字
```
在Tmux Session中，`<prefix>`是tmux的前缀键，所有tmux快捷键都需要先按前缀键。它的默认值是Ctrl+b。
比如我们想做到把终端分成两半，如下图：

{% asset_img tmux.png %}

使用如下命令：

```bash
<prefix>%               #即：先按下ctrl+b(默认前缀)，再按下shift+5(%)
```

是不是很舒服？赶紧试试其他命令吧！
tmux更多命令详见：[Tmux使用手册](http://louiszhai.github.io/2017/09/30/tmux/)

# 安装Oh My Tmux

虽然安装了tmux，但是是不是感觉`ctrl + b`不太好按？我们可以更改tmux的配置文件来修改默认前置，但在这篇博客里不会分享，喜欢折腾的玩家请自行google解决。在这里我介绍一个已经配置好的tmux：oh my tmux。这里是它的github链接：[Oh My Tmux](https://github.com/gpakosz/.tmux)。可以看到这个tmux界面很炫酷有木有。我们赶紧来试试吧！

安装命令：

```bash
cd
git clone https://github.com/gpakosz/.tmux.git
ln -s -f .tmux/.tmux.conf
cp .tmux/.tmux.conf.local .
```

重启你的终端，在使用tmux，是不是已经看到效果了？当然，如果你不满足于此，可以通过查阅官方文档来自己打造一个适合自己的tmux配置。另外tmux配合zsh也可以让你的终端看起来更漂亮，tmux还可以与vim结合起来做一个IDE。多去尝试吧！