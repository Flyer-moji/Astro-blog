---
title: 基于Hexo的轻量化博客搭建
description: 通过hexo再github上搭建自己的静态博客
pubDate: 05 17 2025
image: ""
categories:
  - tech
tags:
  - hexo
---

# 基于Hexo的轻量化博客搭建

在用Astro搭建博客之前，我先用Hexo搭建了一个静态博客，这篇博客记录了我搭建这个博客的过程，方便以后查看。

## 写在最前面

2025年5月16日，我搭建了这个博客。目的是记录自己的学习笔记，分享自己的心得体会，就不提能帮助到别人什么的了，我的水平太过肤浅，只希望这些博客能够见证自己的成长，当作一份水滴石穿的纪念吧。

## 搭建博客第一步--配置环境

首先需要下载的是Node.js，他的网址是[node.js](https://nodejs.org/en/)

这是网站运行需要的环境，下载安装一路点确认即可。

另外，需要安装Git，这是版本控制系统，用来管理博客的更新。

他的网址是[Git](https://git-scm.com/)

还有你也需要有一个github账号，用来存放博客的源代码。(如果你想把自己的博客放在github上的话)

## 搭建博客第二步--安装Hexo并操作

Hexo是一个基于Node.js的静态博客框架，他的网址是[Hexo](https://hexo.io/)

安装之后可以在cmd中输入`hexo -v`查看是否安装成功。

在cmd中，先在你想要放置博客文件的目录下创建一个文件夹，然后输入`hexo init`命令，Hexo会自动创建博客所需的目录结构。

接着需要下载部署插件，输入代码`npm install hexo-deployer-git --save`

然后打开自己的github账号，创建一个新的仓库，并把仓库的名字改为`yourname.github.io`，然后把仓库的clone地址复制下来。粘贴到下文的repo中。

然后在根目录中修改一个名为`_config.yml`的文件，并在最下面修改并写入以下内容：
```
type: git
repo: https://github.com/your_github_username/your_github_repo.git
branch: master
```

然后在cmd中输入`hexo d`命令，Hexo将其部署到github上。

## 搭建博客第三步--配置github pages页面

登录自己的github账号，找到刚才创建的仓库。

点击`Settings`按钮，然后点击`Pages`选项卡。

选择`Source`为`master branch /docs folder`，然后点击`Save`按钮。

等待几分钟，刷新页面，就可以看到自己的博客页面了。

## 搭建博客第四步--开始写博客

在根目录，用代码 `hexo new "博客标题"` 来新建一篇博客。

之后用代码 `code "博客标题"`打开刚才新建的博客文件，就可以写了。（这里用的是vscode，你也可以用记事本）

写完之后，用 `hexo g` 来生成静态页面，然后用 `hexo s` 来启动本地服务器，就可以看到刚才写的博客了。

最后用 `hexo d` 来部署博客，就可以看到自己的博客网页了。

## 写在最后

我一直觉得写博客是一件很酷的事情，就像在茫茫互联网的世界里圈了一块独属于自己的天地。希望以后还能写更多的博客，记录更多让自己有成就感的东西。

如果你并非是专职程序员，只是想用blog记录一些自己专业领域的学习知识，我觉得hexo已经能很好的完成任务了，但是为什么我还是切换了到Astro（导致我花费了很多时间学习）呢？我想就是爱瞎折腾吧：）

### 参考资料

[Hexo官方文档](https://hexo.io/zh-cn/docs/)

[Hexo部署到GitHub Pages](https://www.bilibili.com/video/BV1Yb411a7ty)