---
layout:     post
title:      "Linux下ASLR"
subtitle:   " \"Hello World, Hello Blog\""
date:       2016-09-25 12:00:00
author:     "Readm"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Linux
    - 安全
---


SLR在系统层面上分为3中情况：

+ `0` 不开启
+ `1` 保守开启(Conservative)，除了堆以外全部开启。
+ `2` 全部开启

在1和2中，程序本身编译关系到程序段是否随机化。即是否开启了PIE，gcc缺省并不开启。

linux在x86下开启pie会有明显的性能损耗，而在x64下则不会。