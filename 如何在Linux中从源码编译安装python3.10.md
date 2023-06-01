---
title: "如何在Linux中从源码编译安装python3.10"
date: 2023-03-30T00:07:22+09:00
draft: false
categories:
- IT
tags:
- Linux
- Debian
- python
- 安装
thumbnailImage: "https://d1u9biwaxjngwg.cloudfront.net/video-integration-showcase/peak-140.jpg"
thumbnailImagePosition: left
autoThumbnailImage: yes
metaAlignment: center
summary: "One runs the risk of crying a bit if one allows oneself to be tamed."
---



[Python](https://www.python.org/) 是世界上最流行的编程语言之一。它作为一种通用语言，可以用来构建各种应用程序，例如从简单的脚本到复杂的机器学习算法。它凭借着其简单易学的语法，受到很多初学者以及经验丰富的开发人员的喜爱。

Python 3.10是 Python 的比较新版本的版本之一，本文将介绍如何在基于Debian的Linux发行版中安装Python 3.10 。

目前，[Debian 10](https://linux265.com/distro/10.html) 默认附带 Python 3.7，如果你并不需要更新的版本，完全可以可以使用 apt 工具安装或更新 Python 3.7。注意，如果用以下方法编译安装python，所需时间会比较长。



#### 在基于Debian的Linux发行版中手动编译安装Python3.10的版本

----

从源代码编译 Python 允许安装最新的 Python 版本并自定义构建选项。 但是，这样将无法通过 apt 包管理器维护 Python。所以如果不是对 Python 版本有特殊要求，一般建议默认使用系统仓库自带的Python版本。



01、构建Python编译安装需要的依赖库

```shell
sudo apt update
sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libsqlite3-dev libreadline-dev libffi-dev curl libbz2-dev
```

02、使用wget从Python官网下载对应的版本源代码

```shell
# 进入存放安装包的位置并下载：
cd /usr/local/package
sudo wget https://www.python.org/ftp/python/3.10.10/Python-3.10.10.tgz
```

03、下载完成后，解压gzip压缩包：

```shell
sudo tar -xf Python-3.10.10.tgz
```

04、通过cd命令进入Python源码目录，并执行如下脚本:

```shell
cd Python-3.10.10
./configure --enable-optimizations
```

`--enable-optimizations` 选项通过运行多个测试来优化Python二进制文件。该脚本运行许多检查以确保系统上的所有依赖项都存在。

05、开始构建Python，执行如下命令:

```shell
make -j 2
```

为了加快构建过程，可以通过 -j 指定对应处理器中的内核数。如果你不知道设备处理其中的内核数，可以通过输入`nproc`命令查看内核数。

06、当构建完成后，通过如下命令安装Python二进制文件：

```shell
sudo make altinstall
```

使用 altinstall 而不是 install，因为这样可以覆盖默认的系统 python3 二进制文件。

至此，Python 3.10 已经安装完成，你可以在终端中键入如下命令来验证:

```shell
python3.10 --version
```

你会看到类似如下的输出：

```shell
Python 3.10.10
```

安装成功。
