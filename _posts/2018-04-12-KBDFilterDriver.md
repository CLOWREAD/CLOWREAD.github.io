---
layout:     post
title:      "键盘过滤驱动"
subtitle:   ""
date:       2018-04-26 08:00:00
author:     "SiyuanWang"
#header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Driver
---
# HelloWord:
- 注意,测试机不能是预览版.
- 可能会导致无法开机(比较常见).
- 所以用虚拟机,然后备份一下虚拟磁盘或者创建还原点.
- 虚拟机比实体机测试速度快(16GB RAM).
# KBDFilter:

- 认真读完ReadMe.md
- 把设备名换成真正的设备名.
- 虽然设备名必须是写死的,但是可以写很多很多设备名.
- 如果不是测试模式,会强制驱动签名.但是安装驱动的时候不会报错,只是没有任何作用.
- 框架不要动,只有一个函数要改