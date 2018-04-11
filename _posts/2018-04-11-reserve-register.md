---
layout:     post
title:      "如何在编译器中保留特定寄存器"
subtitle:   "在LLVM中保留一个特殊寄存器作为专用寄存器，需要哪些改动。"
date:       2018-03-15 12:00:00
author:     "Readm"
header-img: "img/tech.jpg"
tags:
    - 技术
    - 编译器
    - LLVM
    - GLIBC
---

最近的工作中，设计中需要一个专用的寄存器。在现有编译器和库中当然没有预留，这需要我们更改编译器和库。

## 目的

在目前的X86寄存器中，选取若干寄存器作为专用寄存器，保留起来不被其他功能使用。

## 前置知识

+ 调用惯例
+ X86寄存器结构
+ 寄存器分配算法

### 调用惯例

在不同的操作系统中有不同的二进制接口（ABI），最主要就是调用惯例。调用惯例中规定了在函数调用中，哪些寄存器用于传递参数，哪些寄存器保持不变(callee saved registers/Preserved Registers)，哪些寄存器会被改变(caller saved registers/Scratch Registers)。这些信息对我们很重要，因为**传递参数的寄存器是无法预留的，除非你改动整个系统的ABI**。

对于其他寄存器，是否保持不变就看你的需求了：这取决于你如何计划库对你的寄存器的操作。

一般，你的编译器只编译你的源文件，而库文件直接使用共享库。这意味着你的程序会调用到共享库，如果你选定了Preserved Register，那么库文件就不会改动你的寄存器（这可以使你直接做到共享库的兼容而不需要重新编译库）；反之，你的寄存器会被库文件破坏。

但值得注意的是：并不总是从你的源文件调用库再返回的。类似于：

```
your_fun0() --call--> your_fun1() --call--> lib_fun0() --call--> lib_fun1()
```

一些情况下，也会出现交叉的调用：

```
your_fun0() --call--> lib_fun0() --call--> your_fun1() --call--> lib_fun1()
```

因为库不改动你的寄存器只是对于调用者说的，所以对于your_fun0来说，lib_fun0()会保持所有Preserved Registers，但在其调用your_func1()时，Preserved Registers仍然可能是被它更改的（在他退出时恢复之前）。

所以，**最保险的做法还是无论是否是Scratch Registers，都重新编译库来保证库不使用它们。**

查询这些信息，可以参考[Call Conventions](http://www.agner.org/optimize/calling_conventions.pdf) [(本站备份)](/docs/calling_conventions.pdf)。

### X86寄存器结构

X86的寄存器结构比较复杂，不同CPU也不同。这里只是提醒如果你的数据较大，通用寄存器可能难以保存，不妨尝试向量寄存器（XMM，YMM等）这在[Cryptographically enforced control flow integrity](https://arxiv.org/abs/1408.1451)中有过使用。

但向量寄存器的缺点在于，难于学习和使用，因为版本不同和用途各异，正确的使用是需要时间学习的。**混用不同版本或者作用域的指令可能得到正确的值，但极大影响性能。**

### 寄存器分配算法

编译器中（LLVM）在编译的前端使用虚拟寄存器来表示变量，只有在后端时才将虚拟寄存器映射到物理寄存器中。我们知道这个过程就足够了，我们需要的是在这个映射过程中，将我们需要的寄存器标记为保留寄存器，不被分配就可以了。

在[文档](https://llvm.org/docs/WritingAnLLVMBackend.html#defining-a-register-class)这一节的最后提到，只要将寄存器标记为reserved，一般情况下就不会被使用到，在我们的观察中也基本是这样的。即使使用到，也会马上帮你恢复回来。

## LLVM中预留

假设你通过上面的知识选定好了使用哪个寄存器来预留，我们就可以开始在LLVM中预留寄存器了。

