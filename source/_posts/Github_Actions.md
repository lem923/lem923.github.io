---

title: Hexo五分钟部署Github Action

tag: [随笔]

date: 1/23/2021

categories: [技术]

---

Github Action是Github提供的一种自动化服务，可以在触发一定条件时，自动执行此前设计好的脚本，从而自动完成一些重复性工作，最大程度地节省人力。就Hexo而言，如果只需要将写好的文章放进Github Repo的Sources文件夹，后面的一系列Hexo clean, Hexo generate, Hexo deploy操作都不再需要自己手动完成，是不是很香呢？

<!--more-->

网络上其实有不少Github Actions自动部署Hexo的教程，但往往需要配置ssh密钥，以及自己修改一些内容，对于更广泛的受众、尤其是零基础的受众来说多少有些令人望而却步。事实上，Hexo自己就提供了[一套简单易行的方案](https://hexo.io/docs/github-pages.html)，可以通过Github Action自动部署网站。

这一网页有提供[中文版](https://hexo.io/zh-cn/docs/github-pages)，但可惜的是，中文版长期没有更新，仍在使用过去的Travis CI，不如Github Action简易。因此，本文将结合Hexo最新的教程，介绍如何三步就简单第部署Github Action。

注：本文默认已安装Hexo并已通过Github Pages发布；否则，你可能并不需要Github Action。基于这一前提，现在你的电脑应已设置好git和git ssh所需密钥，下文将不再赘述。

## 第一步（针对部分人的前期准备）——将Hexo上传至user.github.io的非main分支
这一步只针对使用user.github.io作为Hexo博客repo的人群。由于Github的限制，使用这一域名的网站，内容必须储存在Main分支；同时，Hexo和网站内容（最好）不要放在同一个分支，因此，需要将Hexo上传至新的分支。

也就是说——
- 如果你使用普通repo创建网站，网站放置于gh-pages分支，可忽略该步；
- 如果你已经将Hexo全部上传至user.github.io的main分支，可以选择清空main分支或仅保留public/下的内容；
- 如果你只采用deploy方式将博客发布至user.github.io的main分支，可直接进行下述操作。

**（1）**选择喜欢的位置建立Hexo目录，作为本地仓库目录

**（2）**打开终端，通过cd命令移动到该仓库目录

**（3）**输入以下内容将本地仓库与远程仓库连接([username][repo name]需要根据自己情况替换，不保留中括号)：


```
$ git init
$ git add README.md
$ git commit -m "a"
$ git remote add origin git@github.com:[username]/[username].github.io.git
# 如你在其他repo建立网站，可套用下面这一行
$ git remote add origin git@github.com:[username]/[repo name].git
```


**（4）**继续输入以下命令，先完成首次上传，再建立Hexo分支：


```
$ git push -u origin main
$ git checkout -b Hexo
```


**（5）**将Hexo博客主目录**除public/外**的所有文件复制到你建立的本地仓库目录，再输入以下内容：

```
$ git add .
$ git commit -m "a"
$ git push origin Hexo
```

至此，准备工作已经完成，接下来将在Github网页端完成操作。

## 第二步——建立Github Action
**（1）**在自己Hexo相关Repo的主页（Codes），先将Branch（分支）调整到刚才上传的**Hexo**

**（2）**查看该分支的**Package.json**文件，看其中**scripts**项下是否包括"build": "hexo generate"

**（3）**如果没有，按以下格式添加并保存：

```
{
  "scripts": {
    ...
    "build": "hexo generate"
    ...
    ...
  },
  ...
```

**（4）**在Repo的菜单栏中选择Actions，进一步选择自己设置工作流，将文件命名为**Pages.yml**，并将以下内容复制进该文件（**注意核对目录是否为.github/workflows/pages.yml，一般来说这是默认设置**）：

```
name: Pages

on:
  push:
    branches:
      - Hexo  # 或你自己命名的Hexo储存分支

jobs:
  pages:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Use Node.js 12.x
        uses: actions/setup-node@v1
        with:
          node-version: '12.x'
      - name: Cache NPM dependencies
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          publish_branch: main  # deploying branch
```

**（5）**点击Commit，完成Github Action的创建

## 第三步——检验是否成功实现自动部署
其实自动部署的设置在第二步就可以宣告完成；在完成创建后，由于已经对Hexo分支做出改变，满足了这一Github Action的启动条件，事实上现在它已经开始工作了。

此时，可以在Action页面查看进度，如果中间出现任何问题，也可以在该页面查看具体是哪一步出现了错误，便于纠错。如果对这个Action有信心，也可以直接到Repo的main分支查看是否完成网站的发布。

## 后记
折腾这种事，果然只有0次和无数次。当初在搞Word Press的时候，就花了好几天整理插件、CSS样式、主题等等；到了Hexo，好在没什么太多东西可折腾（相比于Word Press），但就这我也折腾了半天主题和页面的设置以及Github Action。现在终于大功告成，总算能按捺住（或者说被迫按捺住）蠢蠢欲动的心，把心思都放在学习和写作上了。这或许就是Hexo被大家推崇的原因之一吧？
