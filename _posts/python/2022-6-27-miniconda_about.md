---
layout:     post
title:      "『Python』 Miniconda 的安装与使用"
subtitle:   "Anaconda 的精简版：Miniconda。"
date:       2022-06-27 11:00:00
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
catalog: true
tags:
    - Python
---

> 以前都是使用 **Anaconda** 做 python 的开发和学习，长期使用下来，感觉稍微有点臃肿，很多东西都用不到，后面发现了简洁版本的 **Miniconda**。

## 对比
一般来说，**Anaconda** 用的人多，**Miniconda** 相对较少，在这里也做下简单的对比：

|| Anconda | Miniconda |
| ---| --- | --- |
| 大小|  > 580 mb| < 80 mb |
| 包|  > 1500 个| 需单独安装包|
| 便捷性|  高| 低|
| 新手友好度|  高| 低|

其实两者的差别不是很大，主要来说就是占用空间大小和导包的问题。

- 对于没有空间大小顾虑的，对于 conda 不是很熟悉的来说，当然选择 **Anaconda** 最好，基本上要用到的包都一键导入了，对于初学者或者不喜欢频繁导包的人也是想当的方便。
- 对于想要更简洁的环境，多学习 conda 和 python 命令的，可以选择 **Miniconda**。

## 安装
在 [**Miniconda**](https://docs.conda.io/en/latest/miniconda.html)下载系统适用的版本，因为用的是 windows + python3, 所以下载的也是 windows 系统对应的 **Miniconda3**。

![Miniconda download](/img/python/miniconda_about/0.png)

### 配置
和 **Anaconda** 一样，也需要在系统的环境变量中配置。

通过 我的电脑->属性->高级系统设置 -> 高级 -> 环境变量 -> 新建环境变量，将新建的环境变量加入到 path 中，然后就大功告成了。

环境变量的值如下：

`Miniconda 安装路径];[Miniconda 安装路径]\Scripts;[Miniconda 安装路径]\Library\bin`

## 使用

**Miniconda** 的命令行和 **Anaconda** 是一样的，常用的命令如下：

```
    conda list  【查看当前环境安装了哪些包】

    conda env list  【查看当前存在哪些虚拟环境】
    
    conda create -n [env_name] python=[py_version]  【创建新的虚拟环境】

    conda env remove -n [env_name]  【移除环境】

    conda activate [env_name]  【进入虚拟环境】

    conda deactivate  【退出当前虚拟环境】

    conda upgrade --all  【更新当前所有的包】

    conda --version  【查看 anaconda 版本】

    conda install [package_name]  【安装包】
    
    conda remove [package_name]  【移除包】

    conda -h  【查询 conda 的命令使用帮助】
    
    conda env -h 【环境的命令使用帮助】

    conda update conda 【升级 conda】

```