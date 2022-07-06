---
layout:     post
title:      "『Python』 Anaconda 的安装以及 PyCharm 中的配置"
subtitle:   "学Python的第一步，Anaconda。"
date:       2018-04-29 12:00:00
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
catalog: true
tags:
    - Python
---


## 前言

本文是基于 Windows 系统的，Linux 以及其他系统请参考其他文章哦~ 

安装以及配置遇到的问题以及常用命令在文末给出


##  Anaconda 的简介

Anaconda 是 python 的一个开源发行版本，它里面包含了大量的科学包以及各种工具，所以它的下载文件比较大。详细介绍就免了，直接百度...

##  Anaconda 的下载

由于墙的问题，所以 Anaconda 的下载可能会很慢甚至无法下载，下面贴一下清华大学的镜像使用帮助，当然有梯子就可以用官网的下载...

**官方下载地址**（选择自己需要的版本即可）：
[https://www.anaconda.com/distribution/](https://www.anaconda.com/distribution/)

**Anaconda 镜像使用帮助**（windows 根据 32位/ 64位，选择最新的版本即可，在中间位置）：[https://mirror.tuna.tsinghua.edu.cn/help/anaconda/](https://mirror.tuna.tsinghua.edu.cn/help/anaconda/)

![windows 64位系统下载示例](/img/python/anconda_install/0.png)

## Anaconda 的安装

下载了 Anaconda 之后，安装时一直点下一步，到 Selecte Installation Type 这里：
![下面是所有用户都可以使用，但是有管理员](/img/python/anconda_install/1.jpg)

再到 Advanced Options，这里默认不选上面这个加入环境变量，下面的则需要勾选，然后一路安装到底就 OK 了。
![环境变量加入以及默认的Python版本](/img/python/anconda_install/2.jpg)


## Anaconda 的环境配置

上面又说到没有勾选配置环境变量，这里手动配置，我的电脑->属性->高级系统设置->高级->环境变量->新建环境变量，然后就大功告成了。

*例如：*（变量名：ANACONDA_PATH，值：[Anaconda 的安装路径]）：

![例子](/img/python/anconda_install/3.jpg)


## Anconda 创建新的虚拟环境

下载安装并配好环境变量之后，现在使用的只是名为 base 的基础环境，开发项目时需要创建新的虚拟环境（PS：因为不同项目的开发可能包以及 python 的版本都不同，所以虚拟环境能够最大程度提供适合的开发环境）,可以通过以下命令查看当前已有的虚拟环境。

`conda env list` 或 `conda info -e `

创建新的虚拟环境， python=...如果不指定即为默认的，Anaconda 如果不写即不会将当前 base 中常用的包都加入新环境：

```conda create -n [虚拟环境的名称] python=[Anaconda 已有的 Python 的版本] anaconda```

*例如：* ```conda create -n my_env python=3.7 anaconda```

创建好之后可以在再通过命令行查看是否已有即可。

## 使用新的虚拟环境

在命令行中或者在 Anaconda Prompt 中输入，就可以切换环境了，如需切换回 base，直接不加虚拟环境名即可。

```activate [虚拟环境名] ```

*例如：* ```activate my_env```

## Pycharm 中的 Anaconda 配置

新建一个 project，就以新建一个 Django 项目为例吧：

*File* > *New Project* > *Django*，然后会有下面两个选择，如果没有创建虚拟环境就选择上面那个，如果已经有了虚拟环境就选择下面这个。
![Django 环境配置](/img/python/anconda_install/4.jpg)

这里选择以后，如果为空则点击右边的...，进入如下界面，选择需要的虚拟环境，创建完成。

![虚拟环境的选择](/img/python/anconda_install/5.jpg)

#### *过程中出现的问题*

- 如果创建项目之后出现 SSLError 或者是 DLL 的问题，则将下面的路径也添加到环境变量中，并不需要那些在网上说的要重装加 OpenSSL 的，如果这样不行的话OpenSSL也可以加上试试...

```[Anacond的路径]\Script;      [Anacond的路径]\Library\bin```

- 如果没有 Django 模块，则通过 PyCharm 安装：File->Settings->Project:[项目名]->Poject Interpreter，然后添加包安装即可。

![通过 PyCharm 安装 Django](/img/python/anconda_install/6.jpg)


- 如果不能通过 PyCharm 安装 Django 的模块，可以通过命令行切换到指定的虚拟环境下，然后通过 Anaconda 安装 Django。

```conda install django=[可选的版本]```

- 如果通过 conda 命令安装报错，出现 TypeError: LoadLibrary() ... 则在命令行通过 pip 安装。

```pip install django==[可选的版本]```

#### *Anaconda 的基本命令*

Anaconda 常用的命令。

```
    conda list  【查看当前环境安装了哪些包】

    conda env list  【查看当前存在哪些虚拟环境】
    
    conda create -n [env_name] python=[py_version] anaconda  【创建新的虚拟环境】

    conda env remove -n [env_name]  【移除环境】

    activate [env_name]  【进入虚拟环境】

    deactivate  【退出当前虚拟环境】

    conda upgrade --all  【更新当前所有的包】

    conda --version  【查看 anaconda 版本】

    conda install [package_name]  【安装包】
    
    conda remove [package_name]  【移除包】

    conda -h  【查询 conda 的命令使用帮助】
    
    conda env -h 【环境的命令使用帮助】
```