# Hexo五分钟部署Github Actions
---

title: Hexo五分钟部署Github Action
tag: [随笔]
categories: [技术]

---
Github Action是Github提供的一种自动化服务，可以在触发一定条件时，自动执行此前设计好的脚本，从而自动完成一些重复性工作，最大程度地节省人力。就Hexo而言，如果只需要将写好的文章放进Github Repo的Sources文件夹，后面的一系列Hexo clean, Hexo generate, Hexo deploy操作都不再需要自己手动完成，是不是很香呢？

<!--more-->

网络上其实有不少Github Actions自动部署Hexo的教程，但往往需要配置ssh密钥，以及自己修改一些内容，对于更广泛的受众、尤其是零基础的受众来说多少有些令人望而却步。事实上，Hexo自己就提供了[一套简单易行的方案](https://hexo.io/docs/github-pages.html)，可以通过Github Action自动部署网站。

这一网页有提供[中文版](https://hexo.io/zh-cn/docs/github-pages)，但可惜的是，中文版长期没有更新，仍在使用过去的Travis CI，不如Github Action简易。因此，本文将结合Hexo最新的教程，一步步介绍如何更简单第部署Github Action。

注：本文默认已安装Hexo并已通过Github Pages发布；否则，你可能并不需要Github Action。基于这一前提，现在你的电脑应已设置好git和git ssh所需密钥，下文将不再赘述。

## 第一步（针对部分人的前期准备）——将Hexo上传至user.github.io的非main分支
这一步只针对使用user.github.io作为Hexo博客repo的人群。由于Github的限制，使用这一域名的网站，内容必须储存在Main分支；同时，Hexo和网站内容（最好）不要放在同一个分支，因此，需要将Hexo上传至新的分支。

也就是说——
- 如果你使用普通repo创建网站，网站放置于gh-pages分支，可忽略该步；
- 如果你已经将Hexo全部上传至user.github.io的main分支，可以选择清空main分支或仅保留public/下的内容；
- 如果你只采用deploy方式将博客发布至user.github.io的main分支，可直接进行下述操作。

1. 将Hexo主目录下的public/文件夹移动至外部文件夹储存
2. 打开终端，通过cd命令移动到Hexo主目录
3. 输入([username][repo name]需要根据自己情况替换，不保留中括号)：

```
$ git remote add origin git@github.com:[username]/[username].github.io.git
# 或
$ git remote add origin git@github.com:[username]/[repo name]
```

4. 继续输入：

```
$ git add .
$ git commit -m "a"
$ git push -u origin Hexo
```

