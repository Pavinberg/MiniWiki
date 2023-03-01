---
layout: post
title: 给 Python 新手的一点建议
subtitle: 
author: Pavinberg
tags: [Python]
comments: true
---


## 背景

随着 Python 的流行，网上关于 Python 帖子层出不穷，但质量参差不齐。我见过许许多多的新手，因为轻信了网上的帖子而把环境越搞越乱。根据我的观察，总结几点常见的问题，希望能够给不熟悉 Python 的人一点帮助。这些问题不限于直接用 Python 编程的，也包括别的工具间接使用 Python 的。

很多问题基本都归结于两点根本原因，而这是新手没有深刻理解的：

1. Python 是一个对版本很敏感的语言。
2. 在 Unix 系统中，sudo 是一个很危险的操作。

## 不要动系统自带的 Python

Linux、macOS 等类 Unix 操作系统都会自带 Python，因为有很多工具是用 Python 写出来的，而 Python 是一门解释型语言，意味着总是需要 Python 这个解释器来运行 Python 程序（实践中一般不会去打包 Python 程序的）。所以说，这些操作系统都会直接包括一个 Python，来给操作系统提供支持。系统的 Python 一般在 `/usr/bin/python3.x` 这种位置存放。

操作系统自带的 Python 往往版本比较旧，缺少很多新特性，于是新手要么会想要安装新版本的 Python 学习新功能，要么是在使用一些其它工具时被要求更高版本的 Python 支持。

于是网上的帖子开始教大家如何用包管理安装一个新的 Python 等等，然后为了能够直接使用新版本的 Python，会说要你更改系统软链接。会让你执行诸如如下的命令：

```bash
$ sudo ln -s /usr/bin/python3.10 /usr/bin/python3
```

在 macOS 则可能在安装过程中弹窗要求你输入账户密码。

千万别执行！！

事实上， `/usr/bin/python3` 是一个软链接文件。读者可以输入：

```bash
$ ls -l /usr/bin/python3
lrwxrwxrwx 1 root root 9 Oct 25  2018 /usr/bin/python3 -> python3.6
```

这里就显示了它实际指向了 `/usr/bin/python3.6` 这个文件 。也就是说操作系统本身安装了 Python3.6 用于系统工具，然后弄了一个 `python3` 作为 `python3.6` 的别名，方便使用。

那么如果你按照一些帖子所说，使用 `sudo ln -s` 命令更改了这个软链接，就会让 `python3` 指向 `python3.10`。那么这有什么问题呢？那就是 Python 是一个很看重版本的语言，版本高了低了都有可能有问题。 擅自更改系统的 `python3` 会直接破坏一些系统工具的正常使用。

破坏作用不是说百分之百，但是遇到了往往都很坑。例如 Ubuntu 的 bash 有一个 `command-not-found` 功能，在用户输入错误命令时推测用户实际想要输入的命令进行提示。这个功能实际上是 Python 写的，它在运行时需要检查系统的版本，会调用 `lsb_release`，而这也是 Python 写的程序。 它们都使用 `/usr/bin/python3` 指向的 Python 解释器。当你更改了这个软链接时，一旦版本不匹配程序会报错，于是就会惊讶的发现一旦输入了错误的 Linux 命令，就会出现一长串 Python 的 Traceback 报错，不仔细分析根本想不到是系统软链接的问题，还以为是自己的 `~/.bashrc` 出错了呢。

## 不要轻易使用 sudo

这个不仅是对使用 Python 而言，在用 Linux、macOS 时都应该注意。`sudo` 是一个非常强大而危险的最高权限，不应当在你不知道这句命令做什么的情况下就轻易使用。我见到很多新晋 Linux 用户输命令，一遇到 "Permission denied"，想都不想，直接 sudo 再来一遍。我看到这种操作真是心里一惊。正确的做法是思考片刻，为什么权限不够，文件需要什么权限，我有什么权限，是文件权限设置错了，还是我确实不应该碰它，这个命令根本就不该运行。都思考过了，确实需要 sudo，那才应当 sudo。

## 如何管理好 Python 的环境：conda

Python 每个版本都关联着自己的标准库，同时不同的第三方库也有着各自的版本，一个项目需要和这些库的版本对应好才能正确运行。不同的项目却有各自对版本的需求。

Python 本身提供了一个 venv 模块可以提供环境管理，其原理就是平常用户使用 pip 安装包时或者会安到系统全局，或者会安装到用户目录 `~/.local` 中。`venv` 则在可以在项目目录里建一个环境目录， 把项目需要用到的包都安装进去，跟项目绑定了起来。但是 `venv` 并不能指定 Python 版本，而且 `venv` 实际上是分散在各个项目中的，不利于日常零散使用。

我强烈推荐使用 `conda` 来做环境管理。`conda` 被包含在 [Anaconda](https://www.anaconda.com/) 和 [Miniconda](https://docs.conda.io/en/latest/miniconda.html) 中。Anaconda 是一个包含了关于科学计算、数据分析、机器学习库的整合，外加一个 `conda` 环境管理工具。而 Miniconda 只包含了最基本的 Python 标准库和 `conda`。如果没有那么多数据分析需求，可以安装 Miniconda 节省空间，那些数据分析的库完全可以后期用 `pip` 或者 `conda` 手动安装。

安装好后，`conda` 可以为用户建立虚拟环境，环境之间相互隔离，默认是 “base“ 环境。例如，如果希望一个用于 `foo` 项目的 Python3.10 环境，就可以输入：

```bash
$ conda create -n fooenv python=3.10 # 创建虚拟环境 fooenv 包含 Python3.10
$ conda activate fooenv              # 进入虚拟环境 fooenv
$ conda deactivate                   # 退出虚拟环境
$ conda env list                     # 列出所有虚拟环境
$ conda env remove -n fooenv         # 删除虚拟环境 fooenv
```

`conda` 的本质是替用户修改环境变量，让用户输入 `python3` 时，指向 `conda` 虚拟环境内的 Python。同时 `pip` 命令也会被一起更改。

事实上，即使从来不去自己新建虚拟环境，用 `conda` 也是很推荐的。这样可以和系统自带的 Python 区分开，你的所有操作都在自己的 `conda` 环境中，不干扰系统全局，也不怕不小心搞坏了，搞坏了只需要重新建一个 `conda` 环境就好了。

但在极少数情况，环境变量的更改可能会没有生效，需要退出环境再重新激活。
确认环境

新手往往会遇到明明安装了某个包，但是运行的时候还是提示找不到，这往往就是虚拟环境没有切换对导致的。因此，新手应当养成良好习惯，学会检查 Python 的环境。一般来说有如下几个角度：

1. 检查 Python 解释器位置是否在虚拟环境 fooenv 中，
   ```shell
   (fooenv) $ which python3
   /home/username/miniconda3/envs/fooenv/bin/python3
   ```
  
可以看到，路径中包含了 `miniconda3/envs/fooenv/` ，那就没问题了。

2. 不直接使用 `pip` 命令安装库，而是使用如下命令，可以确保 Python3 和 `pip` 是一致的环境：

```bash
$ python3 -m pip install xxx
```

3. 也可以用如下代码查看一个 module 到底被安装到了哪里，例如 `numpy`：

```shell
(fooenv) $ python3 -c "import numpy; print(numpy.__file__)"
/home/username/miniconda3/envs/fooenv/lib/python3.10/site-packages/numpy/__init__.py
```

也有可能不小心安装到了 `~/.local/lib/` 中。

## conda 也可以安装非 Python 相关程序

`conda` 的虚拟环境其实和 Python 不是绑定的，事实上，`conda` 也可以用来管理其它工具的虚拟环境，包括 `gcc`、`nodejs` 等等。 因此，如果用户在服务器上，不方便更改系统中的工具，都可以使用 `conda` 来进行安装。安装前可以先 `conda search xxx` 查看一下是否提供、提供了什么版本，再用 `conda install xxx` 来安装。

## 别动库的源码

虽然这个听起来有些离谱，但真的有帖子这么教别人……

我曾遇到了一个报错不知道如何解决，搜索的过程中，发现 CSDN 上的一篇帖子教大家把源码中这段报错的 `if` 判断注释掉。我内心：？？？

## 多看英文资源和官方文档

国内中文社区真的质量不是很高，错误非常多，即使没错也往往不是最优解。多用用 Bing 等搜索引擎，看不懂英文就多尝试理解，实在不行还有自动翻译嘛。

另外，慢慢练习看官方文档，也有中文文档，尽量能看英文看英文。文档虽然会写的稍微晦涩一点，但绝对是准确严谨的，比个人帖子的可靠性高太多，看多了也就不觉得晦涩了。

## Python 为什么慢

我相信很多人会说，因为 Python 是解释型语言，每一句代码都要进行解释，所以慢。那有没有想过，Java 也是解释执行的，为什么 Java 不慢呢？因为 Java 事先编译好了源码成字节码，在编译的过程中进行了大量的优化。那么 Python 像 Java 一样先编译优化再执行，不就速度快了吗？

答案是，Python 不能被编译优化，因为 Python 是动态类型的，其变量类型要在运行时刻才能确定，不能事先编译确定，这阻止了很多优化的可能。因此，Python 慢的关键不是简单一句解释执行的原因，是动态类型 + 解释执行共同决定的，而动态类型其实是主导因素。
