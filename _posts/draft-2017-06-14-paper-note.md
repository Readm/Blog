---
layout: post
title: 论文简记
subtitle: 论文简记，规范过的部分。
date: 2016-06-14T12:00:00.000Z
author: Readm
header-img: img/post-bg-2015.jpg
tags:
  - 技术
  - Paper
---

# 论文简记

ROP三连：

## The geometry of innocent flesh on the bone: return-into-libc without function calls (on the x86)

** CCS2007 **
** tag: ROP **

第一次提出ROP概念，之前貌似有ret-to-libc但没有正式论文？

## When good instructions go bad: generalizing return-oriented programming to RISC

** CCS2008 **
** tag: ROP **

ROP on RISC，介绍ROP的基本原理。

*Note：在这之前就已经有Stack-Smashing Protection技术和ASLR技术了。*

## Return-oriented programming without returns

** CCS2010 **
** tag： ROP **

ROP的第二篇，在ARM和x86上实现了不使用Return指令的ROP攻击，主要思想是用与Retrun指令等价的指令序列来代替Return。

*Note：文章开头总结了当时的部分防御方法。*

ROP防御

## ROPdefender: a detection tool to defend against return-oriented programming attacks

** ASIACCS2011 **
** tag: ROP **






