---
description: 评估在内核中引入了签名验证机制后所带来的各类性能开销。
---

# Chapter 9 - 签名验证机制开销评估

## 9.1 开销分析

我们的解决方案在内核为执行 ELF 文件进行准备的过程中，额外引入了签名验证机制，从而引入了 CPU、内存、I/O 上的开销：

* CPU 的开销来源于摘要算法和 _RSA_ 算法解密所带来的计算开销
* 内存开销主要来源于将 ELF 中的被保护 section 和签名 section 分别保存在内存缓冲区中用于签名验证
* I/O 开销主要来源于将 ELF 文件的不同 section 载入内存的开销

这里的开销主要分为两个维度：**时间** 上的开销，以及 **空间** 上的开销。

## 9.2 时间开销

我们评估了签名验证机制所带来的时间开销。具体地，评估带有签名验证机制的内核，在执行一个 ELF 文件前的准备工作中，比不进行签名验证的内核慢多少。

我们选用的测试对象是 `/bin` 下的一些系统内置命令，它们实际上都是 ELF 文件，如 `ls` `mv` 等。我们对它们进行签名后，分别在带有签名验证机制的内核与不带签名验证机制的内核上，以空参数执行这些 ELF 程序足够多次。虽然以默认参数执行一些 ELF 文件会导致程序错误退出，但我们只关心 **内核为执行二进制文件而进行准备工作** 的时间，而并不关心二进制文件真正被执行后的状态。即，只关心用户下达执行二进制文件的命令后，到该文件真正开始执行之前的这段时间。

我们对 `fs/exec.c` 文件中实现 exec 系统调用逻辑的 `do_execveat_common()` 函数作为评估对象。该内核函数主要负责为可执行文件准备运行环境，包括在进程控制块中设置入口地址、内存等相关工作。最重要的是，该函数中包括了对 [二进制文件处理模块](../group-1-kernel-signature-verification/chapter-1-binary-execution-procedure.md#13-er-jin-zhi-wen-jian-ge-shi-chu-li-cheng-xu-binary-format-handler) 的遍历和使用。

将每个待测试的 ELF 文件分别在两个内核上运行足够多的次数后，统计 ELF 签名验证机制引入开销所占的百分比。统计结果如下：



## 9.3 空间开销

这里主要关注内核为被验证的 ELF section 所动态分配的内存空间。由于内存空间会在使用完毕后被回收，这里只关心动态分配内存空间的极限值。即，内核最大能够支持分配多大的内存将一个 section 装入内存进行验证。
