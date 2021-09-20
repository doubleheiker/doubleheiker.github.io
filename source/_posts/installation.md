---
title: TensorFlow+CUDA installation
date: 2019-08-21 16:34:17
tags: [Deep Learning,AI,TensorFlow,CUDA]
categories: [AI]
copyright: true
description: <center>Window下超详细的TensorFlow-GPU安装教程！</center>
photo: "http://r.photo.store.qq.com/psb?/V13BnVsC23MbK8/mD.NHcRzs.MalFed2S2VYmiD27mKAdvo6tLNuG6CfSo!/r/dLYAAAAAAAAA"
---
---

> 因为我的电脑TensorFlow-CPU版本感觉运算速度不足，于是乎开始安装TensorFlow-GPU版本。这期间经历了许多坑，也因为各种原因尝试了安装TensorFlow和CUDA的许多版本，因此打算记录下来，作为分享。

---

# 安装Anaconda Navigator

Anaconda Navigator下载地址：[**Anaconda**](https://www.anaconda.com/distribution/)
选择python3.7版本安装

{% asset_img anaconda1.png %}

安装过程比较简单，需要注意的是在`install`这一步前，需要勾选第一项，否则需要手添加环境变量

{% asset_img anaconda2.png %}

安装完成后，验证是否安装成功：
在命令窗口输入：

```
conda --version
```

如果显示了版本，即表明安装成功。
然后为了以后在Anaconda中安装其他插件和环境的方便，我们需要修改下载的镜像地址，我们打开刚刚安装好的Anaconda中的 `Anaconda Prompt`，然后输入:

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --set show_channel_urls yes
```

---

# 安装CUDA® Toolkit+cuDNN

安装TensorFlow-GPU前，需要查看其相应版本所需要的CUDA版本，相应的网址：
https://www.tensorflow.org/install/source_windows

版本|Python 版本|编译器|编译工具|cuDNN|CUDA
---|-----------|------|-------|-----|----
tensorflow_gpu-2.0.0-alpha0|2.7、3.3-3.6|GCC 4.8|Bazel 0.19.2|7.4.1以及更高版本|CUDA 10.0 (需要 410.x 或更高版本)
tensorflow_gpu-1.14.0|2.7、3.3-3.6|GCC 4.8|Bazel 0.19.2|7.4.1以及更高版本|CUDA 10.0 (需要 410.x 或更高版本)
tensorflow_gpu-1.13.0|2.7、3.3-3.6|GCC 4.8|Bazel 0.19.2|7.4|10.0
tensorflow_gpu-1.12.0|2.7、3.3-3.6|GCC 4.8|Bazel 0.15.0|7|9
tensorflow_gpu-1.11.0|2.7、3.3-3.6|GCC 4.8|Bazel 0.15.0|7|9
tensorflow_gpu-1.10.0|2.7、3.3-3.6|GCC 4.8|Bazel 0.15.0|7|9
tensorflow_gpu-1.9.0|2.7、3.3-3.6|GCC 4.8|Bazel 0.11.0|7|9
tensorflow_gpu-1.8.0|2.7、3.3-3.6|GCC 4.8|Bazel 0.10.0|7|9
tensorflow_gpu-1.7.0|2.7、3.3-3.6|GCC 4.8|Bazel 0.9.0|7|9
tensorflow_gpu-1.6.0|2.7、3.3-3.6|GCC 4.8|Bazel 0.9.0|7|9
tensorflow_gpu-1.5.0|2.7、3.3-3.6|GCC 4.8|Bazel 0.8.0|7|9
tensorflow_gpu-1.4.0|2.7、3.3-3.6|GCC 4.8|Bazel 0.5.4|6|8
tensorflow_gpu-1.3.0|2.7、3.3-3.6|GCC 4.8|Bazel 0.4.5|6|8
tensorflow_gpu-1.2.0|2.7、3.3-3.6|GCC 4.8|Bazel 0.4.5|5.1|8
tensorflow_gpu-1.1.0|2.7、3.3-3.6|GCC 4.8|Bazel 0.4.2|5.1|8
tensorflow_gpu-1.0.0|2.7、3.3-3.6|GCC 4.8|Bazel 0.4.2|5.1|8

然后我们需要查看NVIDIA驱动版本，才能安装合适的CUDA版本。在`C:\Program Files\NVIDIA Corporation\NVSMI`目录下，打开命令行窗口，执行`nvidia-smi.exe`：

{% asset_img nvidia1.png %}

如果电脑上没有`NVSMI`文件夹和`nvidia-smi.exe`文件，可以参照这里：
[**Windows NVIDIA Corporation下没有NVSMI文件夹解决方法**](https://blog.csdn.net/Kelly_Young/article/details/96848042)
然后需要看CUDA对应的NVIDIA驱动版本，这里有一个对照表，参照表来安装相应的CUDA：

{% asset_img cuda3.png %}

这个网址对应了官方的版本要求说明：https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html
确定好了需要安装的CUDA和cuDNN版本后，在以下网址下载CUDA和cuDNN:
CUDA：https://developer.nvidia.com/cuda-toolkit-archive
cuDNN：https://developer.nvidia.com/cudnn
下载cuDNN需要NVIDIA账号，注册一个即可。
下载完成后，开始安装CUDA，打开下载好的安装程序，刚开始的安装程序临时存放位置，默认就好：

{% asset_img cuda1.png %}

然后会检测系统兼容性，有些显卡是不支持GPU的，自己需要先查清楚。下一步接受协议，然后选择安装模式，选择自定义模式，程序默认的精简模式应该可以理解为安装所有东西，其中包括VS以及显卡驱动，所以我选择的是自定义模式。在自定义模式中，如果电脑上有VS，那么就去掉VS的安装；另外Driver Component和NVIDIA GeForce Experience也不用勾选。

{% asset_img cuda2.png %}

然后会让你选择安装路径，建议C盘空间足够的同学就直接按照默认路径在C盘中安装了，安装在其他盘有可能出问题。
安装完成后，还需要配置环境变量。系统中会多出两个环境变量：

{% asset_img cuda4.png %}

然后添加如下环境变量：

```
CUDA_SDK_PATH = C:\ProgramData\NVIDIA Corporation\CUDA Samples\v9.0(这是默认安装位置的路径，如果自己路径设置安装成功的话就用自己的路径)

CUDA_LIB_PATH = %CUDA_PATH%\lib\x64 

CUDA_BIN_PATH = %CUDA_PATH%\bin 

CUDA_SDK_BIN_PATH = %CUDA_SDK_PATH%\bin\win64 

CUDA_SDK_LIB_PATH = %CUDA_SDK_PATH%\common\lib\x64
```

{% asset_img cuda5.png %}

添加完之后CUDA就算安装完成了。检验是否安装成功可以到`C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.0\extras\demo_suite(这是默认路径)`中分别执行：

```
bandwidthTest.exe
deviceQuery.exe
```

如果分别返回：

{% asset_img cuda6.png %}

{% asset_img cuda7.png %}

则表示CUDA安装成功。接下来需要解压下载好的cuDNN，将里面的`bin、include、lib/x64`中的文件分别拷到安装好的CUDA文件夹里对应的`bin、include、lib/x64`文件夹中。至此就安装好了CUDA和对应的cuDNN。

---

# 切换CUDA和cuDNN版本

如果安装了错误的CUDA版本或者在之后需要更换CUDA和cuDNN相应的版本，其实方法比较简单，还是在CUDA和cuDNN的官网下载相应的版本，按照上文的方法安装后，只需要将相应环境变量（`CUDA_SDK_PATH`和`CUDA_PATH`）修改为对应的版本即可。

---

# 安装TensorFlow-GPU

我们选择在Anaconda上安装TensorFlow-GPU，因为Anaconda可以独立的配置多个python环境，而互不影响，因此想换想改成其他版本都十分便捷。
首先通过

```bash
conda create -n tf python=3.6
```

创建一个专门用于TensorFlow的环境，然后

```bash
activate tf #进入tensorflow环境
deactivate  #退出tensorflow环境
```
进入这个环境。为了保证安装没有什么问题，建议更新pip和setuptool工具

```bash
python -m pip install --upgrade pip
pip install -–upgrade setuptools
```

然后就可以安装TensorFlow了：

```bash
pip install tensorflow-gpu  # stable
pip install tf-nightly-gpu  # preview
pip install tensorflow-gpu==2.0.0-beta1  #tensorflow2.0
```

安装完成后，进入python解释器，导入TensorFlow，如果导入成功即安装成功。

---

# 将Anaconda的TensorFlow环境导入到PyCharm

现在TensorFlow的环境已经搭好了，为了方便快捷的码代码，建议可以将Anaconda的TensorFlow环境导入到PyCharm（虽然Anaconda下的spider也可以用，但是建议PyCharm）。
打开PyCharm，在`File->Setting`中搜索`Project Interpreter`:

{% asset_img py1.png %}

选择`Add Local`

{% asset_img py2.png %}

然后添加刚刚建立的TensorFlow的环境的`python.exe`的地址：`\Anaconda\envs\tf\python.exe`。然后`OK`、`Apply`即可。