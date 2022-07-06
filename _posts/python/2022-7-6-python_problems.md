---
layout:     post
title:      "『Python』 使用 Python 过程中出现的一些问题"
subtitle:   "各种疑难杂症。"
date:       2022-07-6 15:29:00
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
catalog: true
tags:
    - Python
---

> 记录一下使用 Python 过程中出现的一些问题，持续更新。

**问题 1：** window 上安装 Python 时会提示 `python 运行异常`，安装过程就会中断。

解决办法：降低要安装的 python 版本，安装 python **3.8** 失败则试试 python **3.7** 和 python **3.6**。

---
**问题 2：** window 上使用 pip、django 等命令时，会出现 `不是内部或外部命令`。

解决办法：需要将 DLLs/Scripts 和 Scripts 路径也配置在系统环境变量里。

---
**问题 3：** window 上虽然将 /python.exe 和 /Scripts 的路径添加到系统环境变量，但是使用 pip、django 等其他命令时，会出现 `Fatal error in launcher`。

![image.png](/img/python/python_problems/0.png)

解决办法：经过各种搜索，大概是系统环境配置的问题。要么重装系统，要么使用 `python -m pip` 这种进行操作。

---
**问题 4：** windows 下 django 使用 `manage.py runserver` 启动 django 服务器时，在 *subprocess.py* 下报错提示 `文件不存在/参数错误`。

![QQ截图20220706120318.png](/img/python/python_problems/1.png)

解决办法：按照网上修改 *subprocess.py* 文件中的 `shell=True` 可能是没有用的。打印 `sys.executable`，发现为空，可以在 *manage.py* 的 `main()` 方法上直接手动设置 **python 解释器** 的路径。
```python
if __name__ == '__main__':
    # 设置 python 解释器的路径，单次有效
    sys.executable=("xxxx\python.exe")
    main()
```
---
**问题 5：** window 上想要在 `sys.path` 添加永久的路径。

解决办法：先输出 `sys.path`, 然后在其中已有的任一路径下，新建 *xxx.pth*（必须以 `pth` 为后缀，文件名随意） 的文件，在文件中写入需要添加的路径（存在多个路径需要换行），python 在搜索路径时就会检索 *.pth* 文件里的内容了。

![image.png](/img/python/python_problems/2.png)
 