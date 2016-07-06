---
layout: post
title: 内存的优化
---

## 内存的优化

>内存优化是在所有的基础优化完成后再进行程序极限调优的时候使用，原理主要是集中在：CPU架构（NUMA）、CPU指令集、CPU Cache三个方面，CPU架构方面主要是针对多核CPU内存分页换入换出导致的CPU内核无法访问直接内存，CPU Cache方面主要是理解缓存的原理，最主要记住：64kb大小的数组和128kb倍数的数据会造成缓存Clliation问题，例如：512x512矩阵变换运算速度理论上慢于513x513矩阵变换运算速度大概3/4的样子，在数据一致的情况下。

### NUMA

```bash
numactl --interleave=all

```

>在进程启动前，使用sysctl -q -w vm.drop_caches=3清空文件缓存所占用的空间，启动时就完成整个buffer_pool的内存分配。解决方案只是减少了NUMA内存分配不均，导致的SWAP问题出现的可能性降低，如果当系统上其他进程，或者进程本身需要大量内存时，Buffer Pool的那些Page同样还是会被Swap到存储上。于是又在这基础上出现了另外几个进阶方案

>配置vm.zone_reclaim_mode = 0使得内存不足时去remote memory分配优先于swap out local page
echo -15 > /proc/<pid>/oom_adj调低pid对应进程被OOM_killer强制Kill的可能
>memlock
>参考Mysql的解决方案：

```bash
--memlock
```

```bash
Command-Line Format	--memlock
Permitted Values	Type	boolean
Default	FALSE
```
>Lock the mysqld process in memory. This option might help if you have a problem where the operating system is causing mysqld to swap to disk.

>--memlock works on systems that support the mlockall() system call; this includes Solaris, most Linux distributions that use a 2.4 or higher kernel, and perhaps other Unix systems. On Linux systems, you can tell whether or not mlockall() (and thus this option) is supported by checking to see whether or not it is defined in the system mman.h file, like this:
```bash
shell> grep mlockall /usr/include/sys/mman.h
```
>If mlockall() is supported, you should see in the output of the previous command something like the following:
```c
extern int mlockall (int __flags) __THROW;
```
> ***Important***
>Use of this option may require you to run the server as root, which, for reasons of security, is normally not a good idea. See Section 6.1.5, “How to Run MySQL as a Normal User”.

>On Linux and perhaps other systems, you can avoid the need to run the server as root by changing the limits.conf file. See the notes regarding the memlock limit in Section 8.12.5.2, “Enabling Large Page Support”.

>You must not try to use this option on a system that does not support the mlockall() system call; if you do so, mysqld will very likely crash as soon as you try to start it.

>对进程使用Huge Page（黑魔法，巧用了Huge Page不会被swap的特性），在编译时加上"gcc -D_FILE_OFFSET_BITS=64"

### CPU Cache

>总执行时间在数组大小超过64Bytes时有较为明显的拐点（当然，由于博主是在自己的Mac笔记本上测试的，会受到很多其他程序的干扰，因此会有波动）。原因是当数组小于64Bytes时数组极有可能落在一条Cache Line内，而一个元素的访问就会使得整条Cache Line被填充，因而值得后面的若干个元素受益于缓存带来的加速。而当数组大于64Bytes时，必然至少需要两条Cache Line，继而在循环访问时会出现两次Cache Line的填充，由于缓存填充的时间远高于数据访问的响应时间，因此多一次缓存填充对于总执行的影响会被放大。

#### 了解N-Way Set Associative的存储模式对我们有什么帮助

>了解N-Way Set的概念后，我们不难得出以下结论：2^(6Bits <Cache Line Offset> + 12Bits <Set Index>) = 2^18 = 256K。即在连续的内存地址中每256K都会出现一个处于同一个Cache Set中的缓存对象。也就是说这些对象都会争抢一个仅有16个空位的缓存池（16-Way Set）。而如果我们在程序中又使用了所谓优化神器的“内存对齐”的时候，这种争抢就会越发增多。效率上的损失也会变得非常明显。具体的实际测试我们可以参考： How Misaligning Data Can Increase Performance 12x by Reducing Cache Misses 一文。 这里我们引用一张Gallery of Processor Cache Effects 中的测试结果图，来解释下内存对齐在极端情况下带来的性能损失。 memory_align

![assoc_big1](http://aligitlab.oss-cn-hangzhou-zmf.aliyuncs.com/uploads/IDB-Insight/IDB-OpenRTB/5ab414f90fb78e239a09c648f95bd305/assoc_big1.png)

>该图实际上是我们上文中第一个测试的一个变种。纵轴表示了测试对象数组的大小。横轴表示了每次数组元素访问之间的index间隔。而图中的颜色表示了响应时间的长短，蓝色越明显的部分表示响应时间越长。从这个图我们可以得到很多结论。当然这里我们只对内存带来的性能损失感兴趣。有兴趣的读者也可以阅读原文分析理解其他从图中可以得到的结论。

>从图中我们不难看出图中每1024个步进，即每1024*4即4096Bytes，都有一条特别明显的蓝色竖线。也就是说，只要我们按照4K的步进去访问内存(内存根据4K对齐），无论热点数据多大它的实际效率都是非常低的！按照我们上文的分析，如果4KB的内存对齐，那么一个240MB的数组就含有61440个可以被访问到的数组元素；而对于一个每256K就会有set冲突的16Way二级缓存，总共有256K/4K=64个元素要去争抢16个空位，总共有61440/64=960个这样的元素。那么缓存命中率只有1%，自然效率也就低了。

### 内存分页

>为什么需要Huge Page 了解CPU Cache大致架构的话，一定听过TLB Cache。Linux系统中，对程序可见的，可使用的内存地址是Virtual Address。每个程序的内存地址都是从0开始的。而实际的数据访问是要通过Physical Address进行的。因此，每次内存操作，CPU都需要从page table中把Virtual Address翻译成对应的Physical Address，那么对于大量内存密集型程序来说page table的查找就会成为程序的瓶颈。所以现代CPU中就出现了TLB(Translation Lookaside Buffer) Cache用于缓存少量热点内存地址的mapping关系。然而由于制造成本和工艺的限制，响应时间需要控制在CPU Cycle级别的Cache容量只能存储几十个对象。那么TLB Cache在应对大量热点数据Virual Address转换的时候就显得捉襟见肘了。我们来算下按照标准的Linux页大小(page size) 4K，一个能缓存64元素的TLB Cache只能涵盖4K*64 = 256K的热点数据的内存地址，显然离理想非常遥远的。于是Huge Page就产生了。 Tips: 这里不要把Virutal Address和Windows上的虚拟内存搞混了。后者是为了应对物理内存不足，而将内容从内存换出到其他设备的技术（类似于Linux的SWAP机制）。

>Huge Page 既然改变不了TLB Cache的容量，那么只能从系统层面增加一个TLB Cache entry所能对应的物理内存大小，从而增加TLB Cache所能涵盖的热点内存数据量。假设我们把Linux Page Size增加到16M，那么同样一个容纳64个元素的TLB Cache就能顾及64*16M = 1G的内存热点数据，这样的大小相较上文的256K就显得非常适合实际应用了。像这种将Page Size加大的技术就是Huge Page。

>目前Linux基于多核CPU繁忙程度的线程调度机制，参看Chip Multi Processing aware Linux Kernel Scheduler论文
关于Huge Page

#### 在正式开始本文分析前，我们先大概介绍下Huge Page的历史背景和使用场景。

>为什么需要Huge Page 了解CPU Cache大致架构的话，一定听过TLB Cache。Linux系统中，对程序可见的，可使用的内存地址是Virtual Address。每个程序的内存地址都是从0开始的。而实际的数据访问是要通过Physical Address进行的。因此，每次内存操作，CPU都需要从page table中把Virtual Address翻译成对应的Physical Address，那么对于大量内存密集型程序来说page table的查找就会成为程序的瓶颈。所以现代CPU中就出现了TLB(Translation Lookaside Buffer) Cache用于缓存少量热点内存地址的mapping关系。然而由于制造成本和工艺的限制，响应时间需要控制在CPU Cycle级别的Cache容量只能存储几十个对象。那么TLB Cache在应对大量热点数据Virual Address转换的时候就显得捉襟见肘了。我们来算下按照标准的Linux页大小(page size) 4K，一个能缓存64元素的TLB Cache只能涵盖4K*64 = 256K的热点数据的内存地址，显然离理想非常遥远的。于是Huge Page就产生了。 Tips: 这里不要把Virutal Address和Windows上的虚拟内存搞混了。后者是为了应对物理内存不足，而将内容从内存换出到其他设备的技术（类似于Linux的SWAP机制）。

#### 什么是Huge Page
>既然改变不了TLB Cache的容量，那么只能从系统层面增加一个TLB Cache entry所能对应的物理内存大小，从而增加TLB Cache所能涵盖的热点内存数据量。假设我们把Linux Page Size增加到16M，那么同样一个容纳64个元素的TLB Cache就能顾及64*16M = 1G的内存热点数据，这样的大小相较上文的256K就显得非常适合实际应用了。像这种将Page Size加大的技术就是Huge Page。

#### Huge Page是万能的？

>了解了Huge Page的由来和原理后，我们不难总结出能从Huge Page受益的程序必然是那些热点数据分散且至少超过64个4K Page Size的程序。此外，如果程序的主要运行时间并不是消耗在TLB Cache Miss后的Page Table Lookup上，那么TLB再怎么大，Page Size再怎么增加都是徒劳。在LWN的一篇入门介绍中就提到了这个原理，并且给出了比较详细的估算方法。简单的说就是：先通过oprofile抓取到TLB Miss导致的运行时间占程序总运行时间的多少，来计算出Huge Page所能带来的预期性能提升。 简单的说，我们的程序如果热点数据只有256K，并且集中在连续的内存page上，那么一个64个entry的TLB Cache就足以应付了。说道这里，大家可能有个疑问了：既然我们比较难预测自己的程序访问逻辑是否能从开启Huge Page中受益。反正Huge Page看上去只改了一个Page Size，不会有什么性能损失。那么我们就索性对所有程序都是用Huge Page好啦。 其实这样的想法是完全错误的！也正是本文想要介绍的一个主要内容，在目前常见的NUMA体系下Huge Page也并非万能钥匙，使用不当甚至会使得程序或者数据库性能下降10%。

#### 理想状态

- 为了减少只读热点数据跨NUMA Zone的访问，可以将读写比非常高的Page，使用Replication的方式在每个NUMA Zone的Direct内存中都复制一个副本，降低响应时间。

- 为了减少False Sharing，监控造成大量Cache Miss的Page，并进行拆分重组。将同一CPU亲和的数据放在同一个Page中

#### 现实

>由于没有硬件级别的PMU(Performance Monitor Unit)支持，获取精准的Page访问和Cache Miss信息性能代价非常大。所以上面的理想仅仅停留在实验和论文阶段。那么在理想实现之前，我们现在该怎么办呢？ 答案只有一个就是***测试****

>***实际测试*** 实际测试的结果最具有说服力。所谓实际测试就是把优化对象给予真实环境的压力模拟。通过对比开启和关闭Huge Page时的性能差别来验证Huge Page是否会带来性能提升。当然大多数应用程序，要想模拟真实环境下的运行情况是非常困难的。那么我们就可以用下面这种理论测试

>***理论测试*** 理论测试可以通过profile预估出Huge Page能够带来的潜在提升。具体原理就是计算当前应用程序运行时TLB Miss导致的Page Walk成本占程序总执行时间的占比。当然这种测试方式没有把上文提到的那两种性能损失考虑进去，所以只能用于计算Huge Page所能带来的潜在性能提升的上限。如果计算出来这个值非常低，那么可以认为使用Huge Page则会带来额外的性能损失。具体方法见LWN上介绍的方法 具体的计算公式如下：
```bash
equation
protential benefits=TLB Miss Rate x Sigle Page Walk Cost / Application Cost x 100 %
```
>如果没有hardware的PMU支持的话，计算需要用到oprofile和calibrator。

>并不是所有的优化方案都是0性能损失的。充分的测试和对于优化原理的理解是一个成功优化的前提条件。
