---
layout: post
title: 论文简记
subtitle: 论文简记，规范过的部分。(写笔记真是拖慢看论文速度……
date: 2016-06-14T12:00:00.000Z
author: Readm
header-img: img/post-bg-2015.jpg
tags:
  - 技术
  - Paper
---

# 论文简记

## ROP三连：

### The geometry of innocent flesh on the bone: return-into-libc without function calls (on the x86)

**CCS2007**

**tag: ROP**

第一次提出ROP概念，之前貌似有ret-to-libc但没有正式论文？

### When good instructions go bad: generalizing return-oriented programming to RISC

**CCS2008**

**tag: ROP**

ROP on RISC，介绍ROP的基本原理。

*Note：在这之前就已经有Stack-Smashing Protection技术和ASLR技术了。*

### Return-oriented programming without returns

**CCS2010**

**tag： ROP**

ROP的第二篇，在ARM和x86上实现了不使用Return指令的ROP攻击，主要思想是用与Retrun指令等价的指令序列来代替Return。

*Note：文章开头总结了当时的部分防御方法。*

## ROP防御

防御的方式总共有如下几种思路（互有交叉）：

+ Randomization
+ Function Epilogue（prologue）
+ CFI
+ Return 计数
+ Taint Tracking
+ Shadow Stacks
+ JIT 主要限制了RET指令的具体目标等。

### ROPdefender: a detection tool to defend against return-oriented programming attacks

**ASIACCS2011**

**tag: ROP Pin**

在Pin上实现的，不修改硬件，不需要Side information，针对RET-based配件，支持多线程，效率为2x。主要工作就是使用了影子栈，对各种影子栈的异常情况作了处理。

### Smashing the Gadgets: Hindering Return-Oriented Programming Using In-place Code Randomization

**S&P2012**

**tag: ROP**

原地替换指令来消除/随机化配件，总处理率大概在70+%，但依据文章给出的结果，足以使自动的payload生成失败（Q or Mona）。人工排配（针对性自动）仍有可能。

---

## 污迹追踪与DFI 简记

## Common

+ Frames：
    + Temu, TaintCheck
    + Binary Analysis Plantform
    + **DTA++** for control flow
+ theory->practice: heuristics

### Current Chanllenge

+ Software-only schemes -> large performance overheads and some other problem, while hardware-assisted schemes can avoid theses drawbacks.[^1] [^2] [^3] [^4]
+ Over taint and under taint:
    + under taint:  miss introduce
    + over taint: introduce when not necessary
+ Sanitization problem
    + how to remove taint
+ Raise an alert too late

---

### Secure Program Execution via Dynamic Information Flow Tracking

year: 2004

Information Flow Tracking

#### Implementation of policy:

+ PCR->progration rules
+ TCR->trap rules
+ Trap->software check

#### Implementation on chip:

see fig, nothing novel

#### Efficient tag management:

Page individual.

#### Simulation:

Correct: sim-fast SPEC2000
Efficiency: Bochs open-source x86 emulator

#### Init Tag:

hard drive: tagged
I/O, network: spurious
??? When boot: Only BIOS, ROM is authentic

#### overhead:

low: time & space <5%

#### false positive and false negative

none(20attack and simple linux bash commands: ls etc.)

---

### A Critical Review of Dynamic Taint Analysis and Forward Symbolic Execution

year: 2012

a note for *All you wanted to know about dynamics taint analysis and forward symbolic execution (but may have been afraid to ask)*, and some review

+ *All* provide a new **SimpLE** language as a framework to describe and discuss existing dynamic taint analysis and forward symbolic execution algorithms.
+ not very fit for me the language itself.

---

### FlexiTaint: A Programmable Accelerator for Dynamic Taint Propagation

year: 2008

#### main idea: 

+ Programmed at run time to efficiently follow a desired tainting policy.
+ No hardward change when policy change.
+ Decouples taint storage and processing from data.
+ Without modifying the system bus or main memory.
+ Overhead:
    + use TPC(Taint Propagation Cache) to memoize resulting tains for recently seen combinations of opcode and input taints.
    + use a filter TPT(Taint Propagation Table) to accelerate:
| Value | Meaning |
|:-----:|---------|
|00|Use Taint Propagation Cache (TPC) for this opcode.|
|01|If all source taints are zeros, destination taint is zero. Otherwise, like 00 (use TPC).|
|10|If only one non-zero source taint, copy it to destination taint. Otherwise, like 01.|

#### overhead: 

+ ave:1% worst: 8.4% for SPEC2000  
+ ave:3.7% worst: 8.7% for Splash-2
+ details(Figure 6 in the paper):
    + Most: No Taint
    + Some: One Sourec Taint
    + Rare: TPC hit
    + Very Rare: TPC Miss

---

### Pointless Tainting? Evaluating the Practicality of Pointer Tainting

year: 2009

#### A review of point taint

#### history

+ basic tainting: 
    + only for control divert attack
    + raise when jmp/call
+ pointer tainting
    1. for non-control divert attack, LPT
        + if p is tainted, raise an alert on any dereference of p
    2. for privacy breache, FPT
        + if p is tainted, any dereference of p taints the destination
        
        
#### Pointer tainting is very popular because:

+ it can be applied to unmodified software without recompilation
+ according to its advocates, it incurs hardly (if any) false positives
+ it is assumed to be one of the only (if not the only) reliable techniques capable of detecting both control-diverting and non-control-diverting attacks without requiring recompilation.

#### minor issues:

+ discuss: *Deconstructing hardware architectures for security* 2006
+ address: *Realworld buffer overflow protection for userspace and kernelspace* 2008
+ same author: *Raksha: a flexible information flow architecture for software security* 2007


Basic taint analysis has been successfully applied in numerous systems [Crandall 2004, Newsome 2005, Costa 2005, Ho 2006, Portokalidis 2006, Slowinska 2007, Portokalidis 2008]. The drawback is that it protects against controldiverting attacks, but not against non-control-diverting attacks as shown presently






[^1]: S. Chen, J. Xu, N. Nakka, Z. Kalbarczyk, and R. Iyer. Defeating Memory Corruption Attacks via Pointer Taintedness Detection. Proc. 2005 International Conference on Dependable Systems and Networks, 2005.

[^2]: J. Kong, C. Zou, and H. Zhou. Improving Software Security via Runtime Instruction Level Taint Checking. Proc. 1st Workshop on Architectural and System Support for Improving Software Dependability, 2006.

[^3]: M.Dalton, H.Kannan, and C.Kozyrakis. Raksha: A Flexible Information Flow Architecture for Software Security. Proc. 34th International Symposium on Computer Architecture, 2007.

[^4]: G. Suh, J. Lee, and S. Devadas. Secure Program Execution via Dynamic Information Flow Tracking. Proc. 11th International Conference on Architectural Support for Programming Languages and Operating Systems, 2004.

[^SPEC]: SPEC. Standard performance evaluation corporation. http://www.spec.org, 2004.

---

### Eudaemon: Involuntary and On-Demand Emulation Against Zero-Day Exploits

year: 2008

#### Aim: full-time deployment(what I want)

#### main idea:

+ **attach/detach at runtime**
+ use a DLL/DSO
+ based on *Argos*
+ strange shortcoming: place in the user space.

#### Mentioned System:

+ Argos
+ Minos
+ Vigilante
+ TaintCheck

#### Speed up to full-time

+ [4],use Qemu[6]

#### Useful information:

+ the syscall that need to consider is in the section 4.1.1, which I may implement in the same way.

#### details:

+ handle signals aswell

**Read, not useful** Another interesting way of coping with the slowdown (and indeed, a way of slicing in the temporal domain for servers) is known as shadow honeypots [5]. A fast method is used to crudely classify certain traffic as suspect with few false negative but some false positives. Such traffic is then handled by a slow honeypot. -->*Detecting Targeted Attacks Using Shadow Honeypots*

---

### libdft: Practical Dynamic Data Flow Tracking for Commodity Systems

year: 2012

Main: 

+ fast, reuseable and applicable to commodity hardware and software
+ **use Pin framework**: a meta-tool in form of a shared library.
+ 1.14x~6.03x (large applications like Apache, MySQL: 1.25x~4.83x)

+ Minemu: The World's Fastest Taint Tracker 2011
    + 32-bit
    + focus on a single problem domain

Features:

+ 64-bit only
+ not support mutithreaded applications
+ fast: Dytan reported a 30x slowdown when compressing with gzip, LIFT reports less than 10x(both sucks...)
+ 

Optimizations：

+ 4.2.2 Analysis Routines: optimizations for pin.
+ 4.4.1 vcpu optimizations in pin, use *scratch registers* in pin.
+ **4.4.2 fast_rep: propagation outside of the loop.**
+ 4.4.3 Huge TLB: use multiple page size feature on x86 architectures
nothing else novel

---

### Securing software by enforcing data-flow integrity(DFI)

year: 2006

Need static analysis to generate a DFG.

Use the Phoenix compiler infrastructure[29]. The implementation uses reaching definitions analysis[7] to compute a static data-flow graph.

Overhead:

+ Mem: 50%
+ Runtime 44~103% in Spec 2000 benchmarks
+ web server response time 0.1%, throughput -23%.

**Has false negatives but no false positives.

Three Step:

1. uses static analysis to compute a data-flow graph for the vulnerable program
1. instruments the program to ensure that the data-flow at runtime is allowed by this graph
3. runs the instrumented program and raises an exception if data-flow integrity is violated

To compute reaching definitions at runtime, we maintain the runtime definitions
table (RDT) that records the identifier of the last instruction to write to each memory position. Every write is instrumented to update the RDT.

---

### HDFI: Hardware-Assisted Data-flow Isolation

year: 2016 S&P

HDFI’s tags are associated with memory units’ physical addresses, (on FPGA)

Tag: Word Granularity, 1bit

developed and ported six representative security mechanisms to leverage HDFI:

+ stack protection
+ standard library enhancement (protection for setjmp/longjmp, heap metadata, GOT, and the exit handler)
+ virtual function table protection
+ code pointer separation
+ kernel data protection
+ information leak prevention

Implement DFI of ONE bit tag.

Physical memory tag and data are separated. Between cache and physical memory, there is a DFITagger, to bind tag and cache together in cache. 






---

### PASS

+ LIFT: software base
+ TaintDroid：For Private Information



