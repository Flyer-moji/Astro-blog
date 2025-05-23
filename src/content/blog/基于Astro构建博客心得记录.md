---
title: 基于Astro构建博客心得记录
description: 通过Astro构建博客的途中，碰到了许多问题，记录一下解决方法和心得。
pubDate: 05 18 2025
image: ""
categories:
  - tech
tags:
  - Astro
  - blog

---
## 想要把东西搞成，就不要怕折腾

昨晚觉得Hexo的各种更能都有点过时了，而且没有炫酷的动效，~~是不是有点太注重外貌了：）~~，遂开始折腾新的架构。

[Astro的官方文档](https://docs.astro.build/zh-cn/getting-started/)是可以用于学习的，但是如果github上有别人的轮子，自己从头开始确实效率偏低，（我毕竟是个菜鸡）。

遂直接把别人的“车”给拿下了，就是现在这个主题[Frosti](https://github.com/EveSunMaple/Frosti/)

## 遇到的问题

#### 1. 部署到netlify时，复制指令的错误


白嫖虽好，也不是没有代价的，比如说各种基础的知识，从零开始的我属实花了一些时间搞明白自己在干什么。而许多代码的配置问题，由于并不了解作者整体的部署，也是走了许许多多弯路。

比如说这个关于跨平台复制的功能：
package.json
```
### 已经修复了

"scripts": {
    "dev": "astro dev",
    "start": "astro dev",
    "build": "astro check && astro build && pagefind --site dist && cp -r dist/pagefind public/",
    "build:win": "astro check && astro build && pagefind --site dist && powershell Copy-Item -Recurse -Force dist/pagefind public/",
    "preview": "astro preview",
    "astro": "astro"
  },
```
这个脚本的作用是，在build的时候，复制pagefind文件夹到public文件夹下，这样就可以在本地预览的时候，访问到pagefind的页面。

但是，在windows系统下，这个脚本并不能正常工作，因为windows系统下，复制文件夹的命令与linux系统不同。

参见build 和build：win两个命令的区别，可以看到，在windows系统下，使用的是powershell的命令，而在linux系统下，使用的是cp命令。

我第一遍部署的时候对全局不了解，别AI忽悠瘸了，直接把这个代码改成win下能运行的，本地build之后发现没什么问题，就提交了。

但是在netlify中问题就出现了，这超出我的能力范围，压根不知道怎么处理一个网络服务器。

在愤然关机睡觉之后，我把本地文件干掉又重新部署了一次。

这次对全局有了一定的了解，在本地部署的时候出现cp命令不能识别，我就开始处理跨平台的问题了。最终是解决了，在linux系统下用默认的build，我在本地部署的时候用Windows的专属命令。

最后重新用·Astro·部署博客，发现部署成功，并且可以正常访问，心情大好，问题解决了，又水了一篇博客：）

#### 2. 部署到github时，报错

```

pages build and deployment / build (dynamic) Failing after 16s

```

这是个蠢问题，在本地更新md文件时新建命名后忘记写后缀，导致无法识别，报错。