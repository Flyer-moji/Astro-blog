---
title: Matlab App Designer 使用笔记 / 单片机串口调试
description: Matlab App Designer 使用笔记 / 单片机与上位机的交互配置
pubDate: 2025-07-15
categories:
  - tech
tags:
  - 单片机
  - matlab
---

# Matlab App Designer 使用笔记

最近在做关于单片机的上位机交互实验，想这如果能把代码可视化会非常有意思。所以研究了一下matlab app designer，正好水一篇，防止自己三个月之后什么也记不起来。

## App Designer 设计思路

![我设计的图形页面](https://blog-picbed-1317989631.cos.ap-hongkong.myqcloud.com/20250911142127.png)
### 概述

Designer的功能很强，而且比较安全。打开界面之后在左侧选择控件，这些控件的功能相当字如其意。

设置好界面之后可以右键选择添加回调，不同控件有对应的选择。比如pushdown回调，或者在为点击之前设置回调等。

还有一部分控件用于在可视化界面中设置初始化数据。比如说图中的波特率设置，端口等等。

![选项控件定义方法](https://blog-picbed-1317989631.cos.ap-hongkong.myqcloud.com/20250911143032.png)

点击3点框，可以设置需要的信息。也可以设置默认信息位。

另外也可以设置按钮名称，这样在代码回调书写时更方便。
![按钮名称](https://blog-picbed-1317989631.cos.ap-hongkong.myqcloud.com/20250911143731.png)

### 回调设置

右键控件之后可以设置回调，在代码视图中选择。
![代码图床在右上](https://blog-picbed-1317989631.cos.ap-hongkong.myqcloud.com/20250911143241.png)
代码视图的左上可以添加函数、全局变量。
![函数、变量选择](https://blog-picbed-1317989631.cos.ap-hongkong.myqcloud.com/20250911143415.png)
这里要注意，public和private的区别。在app中，前者指的是公用，即在命令行里也可以调用，而后者指的是私用（范围为app内部），也就是说他在app内部是全局的，而在外部不可访问。

回调设置完成之后，点击运行按钮，可以看到程序运行的结果。

### 使用

![app运行](https://blog-picbed-1317989631.cos.ap-hongkong.myqcloud.com/20250911143912.png)

app最好用的就是一点代码都看不到，完全是图形化的界面。

## 单片机串口调试

这次吸收的教训主要有这几点：

1. ST-link的作用是烧录程序，并不用于和上位机的信息传递。
2. 串口通信的配置需要注意波特率、数据位、校验位、停止位等。
3. 串口通信使用的是另一根线，长得像旧版的安卓usb-mini线。
4. 想要识别到设备一定要检查对应的驱动安装问题。

### 如何检查串口识别

1.win+x找到设备管理器，，在ports下可以看到串口设备。
![](https://blog-picbed-1317989631.cos.ap-hongkong.myqcloud.com/20250911144550.png)
2.如果没找到，需要根据连接形式和芯片，寻找对应的驱动并安装。这里是ch340。[驱动下载](https://www.wch.cn/downloads/CH341SER_EXE.html)
3.使用串口助手，设定好正确的接收格式（如波特率，校验）最后就能看到单片机的输出了。
![串口精灵](https://blog-picbed-1317989631.cos.ap-hongkong.myqcloud.com/20250911144831.png)




















