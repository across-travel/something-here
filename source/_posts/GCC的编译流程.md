---
title: GCC的编译流程
date: 2015-04-12 14:20
tags:
 - C++
---

**GCC的编译流程分为了4个步骤**

- 预处理（Pre-Processing）
- 编译（Compiling）
- 汇编（Assembling）
- 链接（Linking）

**GCC使用的基本语法为：**

    gcc [option | filename]

预处理阶段

    gcc –E –o [目标文件] [编译文件]

选项“-E”可以使编译器在预处理结束时就停止编译,选项“-o”是指定GCC输出的结果。

编译阶段

    gcc –S –o hello.s hello.i

 选项“-S”能使编译器在进行完编译之后就停止

汇编阶段

    gcc –c hello.s –o hello.
    
选项“-c”把编译阶段生成的“.s”文件生成目标文件 “.o”

链接阶段

    gcc hello.o –o hello

可以生成可执行文件