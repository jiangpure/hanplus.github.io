---
layout:     post
title:      "『Github』命令行上传项目到 GitHub"
subtitle:   "Git命令行上传。"
date:       2019-02-12 09:18:00
author:     "purejiang"
header-img: "img/post-bg-1.jpg"
catalog: true
tags:
    - Github
---

```
下载并安装 git，打开 GitBash。
```
###### 第一步：在目录下建立 *git* 本地仓库。
*cd* 到你的本地项目根目录下，执行 *git* 命令

`git init`

###### 第二步：将项目的所有文件添加到仓库中。

`git add .`

如果想添加某个特定的文件，只需把.换成特定的文件名即可，如果是想添加文件夹，只需用把 . 换成 xxx/ 即可。

###### 第三步：将 `add` 的文件 `commit` 到仓库。

`git commit -m "注释语句"`

###### 第四步：去 *github* 上创建自己的 *Repository*。

点击 `Create repository`，创建完成之后，拿到创建的仓库的 `https` 地址


###### 第五步：重点来了，将本地的仓库关联到 *github* 上。

`git remote add origin https://xxx/xxxx.git`

后面的 `https` 链接地址换成你自己的仓库 `url` 地址。

###### 第六步：上传 *github* 之前，要先 `pull` 分支，执行如下命令：

`git pull origin master`

###### 第七步：上传代码到 *github* 远程仓库。

`git push -u origin master`

###### 第八步：输入 *github* 账号和密码。

等待执行完成就上传成功了，这时就可以看到代码已上传至 *github* 上。
