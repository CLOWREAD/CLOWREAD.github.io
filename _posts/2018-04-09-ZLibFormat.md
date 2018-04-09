---
layout:     post
title:      "ZLib远古格式解析"
subtitle:   ""
date:       2018-04-09 18:48:00
author:     "SiyuanWang"
#header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - ZLIb
---
# ZLib
这是远古的文件格式了,最近想自己写一个png文件解析,然后发现data部分是压缩的(果然是压缩的).查看了一下压缩格式,deflate,pnglib也是用了zlib的库(deflate就是给png用的).网上一搜"zlib deflate",全是原理,没有一点代码解释.然后搜"deflate代码"结果全是zlib怎么用的.我自己看了看zlib源码,竟然用的状态机模型,里面的注释还有的没的.所以我直接放弃源码.还好给了文档,看了网上的文档和文档的翻译才半懂不懂地实验了起来. \
反正死就死吧,现时两个字节的0x789c,然后3bit的deflate头.