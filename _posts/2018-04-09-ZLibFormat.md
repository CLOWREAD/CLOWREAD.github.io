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
这是远古的文件格式了,最近想自己写一个png文件解析,然后发现data部分是压缩的(果然是压缩的).查看了一下压缩格式,deflate,pnglib也是用了zlib的库(deflate就是给png用的).网上一搜"zlib deflate",全是原理,没有一点代码解释.然后搜"deflate代码"结果全是zlib怎么用的.我自己看了看zlib源码,里面用的状态机模型,代码的注释还有的没的.所以我直接放弃源码.还好给了文档,看了网上的文档和文档的翻译才半懂不懂地实验了起来.

反正死就死吧,先是两个字节的0x789c,然后3bit的deflate头.3个Bit含义如下:
+ first bit       BFINAL
+  next 2 bits     BTYPE

头结构之后是数据流.数据视为Bit流,其中Bit流数据的可以跨字节.Bit流从低位到高位,从低字节到高字节;

|Byte0  | Byte1 | Byte2 |Byte3|
|:--:|:--:|:--:|:--:|
|01234567->|01234567->|01234567->|01234567->|

注意:

+ 低位的Bit位在取出的数据的低位记为正序,反之为逆序;
+ Huffman编码为逆序;
+ 扩展位为正序;
+ distance的5个Bit为逆序;



正序:

    offset = 0;
    for (int i = 0; i < eb; i++)
    {			
      res = ReadBit(&bit);
      if (res == 1)
      {
        goto FIN;
      }
      offset += bit<<i;
    }


逆序:

    distcode = 0;
    for (int i = 1; i <= 5; i++)
    {
      distcode <<= 1;
      res = ReadBit(&bit);
      if (res == 1)
      {
        goto FIN;
      }
      distcode += bit;
    }



Inflate流程:

     从Bit流中读取一个Huffman编码Value;
     if(Value<=255)
     {
        直接输出;
     }
     if(Value>=257)
     {
       根据Value获得Length扩展位个数;
       正序读取Length扩展位;
       根据Lit/Len Code表和扩展位ExtraBit得到length;
       逆序读取5Bit Distance Code;
       根据Distance Code 得到Distance 的扩展位个数;
       正序读取Distance 的扩展位;
       根据Distance表和扩展位得到Distance;
       从当前输出位置向前移动Distance字节,由此处向后复制Length个字节到输出位置;
     }
     if(Value==256)
     {
       当前块结束;
     }
     更新滑动窗口;



Lit/Len Code:

      Extra               Extra               Extra
      Code Bits Length(s) Code Bits Lengths   Code Bits Length(s)
      ---- ---- ------     ---- ---- -------   ---- ---- -------
       257   0     3       267   1   15,16     277   4   67-82
       258   0     4       268   1   17,18     278   4   83-98
       259   0     5       269   2   19-22     279   4   99-114
       260   0     6       270   2   23-26     280   4  115-130
       261   0     7       271   2   27-30     281   5  131-162
       262   0     8       272   2   31-34     282   5  163-194
       263   0     9       273   3   35-42     283   5  195-226
       264   0    10       274   3   43-50     284   5  227-257
       265   1  11,12      275   3   51-58     285   0    258
       266   1  13,14      276   3   59-66

Distance Code:

          Extra           Extra               Extra
       Code Bits Dist  Code Bits   Dist     Code Bits Distance
       ---- ---- ----  ---- ----  ------    ---- ---- --------
         0   0    1     10   4     33-48    20    9   1025-1536
         1   0    2     11   4     49-64    21    9   1537-2048
         2   0    3     12   5     65-96    22   10   2049-3072
         3   0    4     13   5     97-128   23   10   3073-4096
         4   1   5,6    14   6    129-192   24   11   4097-6144
         5   1   7,8    15   6    193-256   25   11   6145-8192
         6   2   9-12   16   7    257-384   26   12  8193-12288
         7   2  13-16   17   7    385-512   27   12 12289-16384
         8   3  17-24   18   8    513-768   28   13 16385-24576
         9   3  25-32   19   8   769-1024   29   13 24577-32768


Fixed Huffman codes (BTYPE=01):

      Lit Value    Bits        Codes
      ---------    ----        -----
        0 - 143     8          00110000 through
                               10111111
      144 - 255     9          110010000 through
                               111111111
      256 - 279     7          0000000 through
                               0010111
      280 - 287     8          11000000 through
                               11000111

