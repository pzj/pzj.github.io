---
layout: post
title: Linux性能分析
category: 资源
tags: 性能
description: Linux性能分析
---


### 1. uptime

```
[pengzhangjie]$ uptime
 20:11:28 up 148 days,  8:09,  1 user,  load average: 0.08, 0.08, 0.03
```

这个命令显示了要运行的任务（进程）数，通过它能够快速了解系统的平均负载。在Linux上，这些数值既包括正在或准备运行在CPU上的进程，也包括阻塞在uninterruptible I/O（通常是磁盘I/O）上的进程。它展示了资源负载（或需求）的大致情况，不过进一步的解读还有待其它工具的协助。对它的具体数值不用太较真。

最右的三个数值分别是1分钟、5分钟、15分钟系统负载的移动平均值。

### 2. dmesg | tail

```
[pengzhangjie@centos217 ~]$ dmesg | tail
UDP: bad checksum. From 60.191.23.60:8312 to 183.36.122.217:1080 ulen 109
……
```

这个命令显示了最新的10个系统信息，如果有的话。注意会导致性能问题的错误信息。上面的例子里就包括对过多占用内存的某进程的死刑判决，还有丢弃TCP请求的公告。

### 3. vmstat 1

```
[pengzhangjie]$ vmstat 1
procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0 38261304 471000 5813820    0    0     0     2    0    0  1  0 99  0  0	
 0  0      0 38224276 471000 5813824    0    0     0     4 9365 14501  9  2 89  0  0	
```
vmstat(8)，是“virtual memory stat”的简称，几十年前就已经包括在BSD套件之中，一直以来都是居家常备的工具。它会逐行输出服务器关键数据的统计结果。

检查下面各列：

r：等待CPU的进程数。该指标能更好地判定CPU是否饱和，因为它不包括I/O。简单地说，r值高于CPU数时就意味着饱和。

free：空闲的内存千字节数。如果你数不清有多少位，就说明系统内存是充足的。接下来要讲到的第7个命令，free -m，能够更清楚地说明空闲内存的状态。

si，so：Swap-ins和Swap-outs。如果它们不为零，意味着内存已经不足，开始动用交换空间的存粮了。

us，sy，id，wa，st：它们是所有CPU的使用百分比。它们分别表示user time，system time（处于内核态的时间），idle，wait I/O和steal time（被其它租户，或者是租户自己的Xen隔离设备驱动域（isolated driver domain），所占用的时间）。

通过相加us和sy的百分比，你可以确定CPU是否处于忙碌状态。一个持续不变的wait I/O意味着瓶颈在硬盘上，这种情况往往伴随着CPU的空闲，因为任务都卡在磁盘I/O上了。你可以把wait I/O当作CPU空闲的另一种形式，它额外给出了CPU空闲的线索。

I/O处理同样会消耗系统时间。一个高于20%的平均系统时间，往往值得进一步发掘：也许系统花在I/O的时太长了。

在上面的例子中，CPU基本把时间花在用户态里面，意味着跑在上面的应用占用了大部分时间。此外，CPU平均使用率在90%之上。这不一定是个问题；检查下“r”列，看看是否饱和了。

### 4. mpstat -P ALL 1
```
[pengzhangjie@centos ~]$ mpstat -P ALL 1
Linux 2.6.32-504.16.2.el6.x86_64 (centos) 	09/19/2018 	_x86_64_	(16 CPU)

08:19:56 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
08:19:57 PM  all    0.44    0.00    0.13    0.00    0.00    0.00    0.00    0.00   99.44
08:19:57 PM    0    2.00    0.00    1.00    0.00    0.00    0.00    0.00    0.00   97.00
08:19:57 PM    1    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
08:19:57 PM    2    1.01    0.00    0.00    0.00    0.00    0.00    0.00    0.00   98.99
```
这个命令显示每个CPU的时间使用百分比，你可以用它来检查CPU是否存在负载不均衡。单个过于忙碌的CPU可能意味着整个应用只有单个线程在工作。

### 5. pidstat 1
```
[pengzhangjie]$ pidstat 1
Linux 2.6.32-504.16.2.el6.x86_64 (centos) 	09/19/2018 	_x86_64_	(16 CPU)

08:21:13 PM       PID    %usr %system  %guest    %CPU   CPU  Command
08:21:14 PM     14721    0.98    0.00    0.00    0.98     0  fts_task_redis_
08:21:14 PM     17931    0.98    0.00    0.00    0.98     1  python
08:21:14 PM     19907    0.98    0.00    0.00    0.98    10  yyms_agent_d
```
pidstat看上去就像top，不过top的输出会覆盖掉之前的输出，而pidstat的输出则添加在之前的输出的后面。这有利于观察数据随时间的变动情况，也便于把你看到的内容复制粘贴到调查报告中。

### 6. iostat -xz 1
```
[pengzhangjie]$ iostat -xz 1
Linux 2.6.32-504.16.2.el6.x86_64 (centos) 	09/19/2018 	_x86_64_	(16 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.33    0.00    0.20    0.01    0.00   99.46

Device:         rrqm/s   wrqm/s     r/s     w/s   rsec/s   wsec/s avgrq-sz avgqu-sz   await  svctm  %util
sda               0.00     6.49    0.03    3.57     0.90    80.49    22.60     0.00    0.54   0.39   0.14
sdb               0.00     0.00    0.00    0.07     0.00     0.55     8.24     0.00    0.73   0.73   0.00
```

这个命令可以弄清块设备（磁盘）的状况，包括工作负载和处理性能。注意以下各项：

r/s，w/s，rkB/s，wkB/s：分别表示每秒设备读次数，写次数，读的KB数，写的KB数。它们描述了磁盘的工作负载。也许性能问题就是由过高的负载所造成的。

await：I/O平均时间，以毫秒作单位。它是应用中I/O处理所实际消耗的时间，因为其中既包括排队用时也包括处理用时。如果它比预期的大，就意味着设备饱和了，或者设备出了问题。

svctm：平均每次设备I/O操作的服务时间 (毫秒).即 delta(use)/delta(rio+wio).

avgqu-sz：分配给设备的平均请求数。大于1表示设备已经饱和了。（不过有些设备可以并行处理请求，比如由多个磁盘组成的虚拟设备）

%util：设备使用率。这个值显示了设备每秒内工作时间的百分比，一般都处于高位。低于60%通常是低性能的表现（也可以从await中看出），不过这个得看设备的类型。接近100%通常意味着饱和。

cup使用率与负载区别？

如果某个存储设备是由多个物理磁盘组成的逻辑磁盘设备，100%的使用率可能只是意味着I/O占用

请牢记于心，disk I/O性能低不一定是个问题。应用的I/O往往是异步的（比如预读（read-ahead）和写缓冲（buffering for writes）），所以不一定会被阻塞并遭受延迟。

如果 %util 接近 100%，说明产生的I/O请求太多，I/O系统已经满负荷，该磁盘可能存在瓶颈。 idle小于70% IO压力就较大了，一般读取速度有较多的wait。 同时可以结合vmstat 查看查看b参数(等待资源的进程数)和wa参数(IO等待所占用的CPU时间的百分比，高过30%时IO压力高)。

另外 await 的参数也要多和 svctm 来参考。差的过高就一定有 IO 的问题。

avgqu-sz 也是个做 IO 调优时需要注意的地方，这个就是直接每次操作的数据的大小，如果次数多，但数据拿的小的话，其实 IO 也会很小。如果数据拿的大，才IO 的数据会高。也可以通过 avgqu-sz × ( r/s or w/s ) = rsec/s or wsec/s。也就是讲，读定速度是这个来决定的。

svctm 一般要小于 await (因为同时等待的请求的等待时间被重复计算了)，svctm 的大小一般和磁盘性能有关，CPU/内存的负荷也会对其有影响，请求过多也会间接导致 svctm 的增加。await 的大小一般取决于服务时间(svctm) 以及 I/O 队列的长度和 I/O 请求的发出模式。如果 svctm 比较接近 await，说明 I/O 几乎没有等待时间；如果 await 远大于 svctm，说明 I/O 队列太长，应用得到的响应时间变慢，如果响应时间超过了用户可以容许的范围，这时可以考虑更换更快的磁盘，调整内核 elevator 算法，优化应用，或者升级 CPU。

队列长度(avgqu-sz)也可作为衡量系统 I/O 负荷的指标，但由于 avgqu-sz 是按照单位时间的平均值，所以不能反映瞬间的 I/O 洪水。

形象的比喻：
r/s+w/s 类似于交款人的总数
平均队列长度(avgqu-sz)类似于单位时间里平均排队人的个数
平均服务时间(svctm)类似于收银员的收款速度
平均等待时间(await)类似于平均每人的等待时间
平均I/O数据(avgrq-sz)类似于平均每人所买的东西多少
I/O 操作率 (%util)类似于收款台前有人排队的时间比例
设备IO操作:总IO(io)/s = r/s(读) +w/s(写)

平均等待时间=单个I/O服务器时间*(1+2+...+请求总数-1)/请求总数

每秒发出的I/0请求很多,但是平均队列就4,表示这些请求比较均匀,大部分处理还是比较及时。

### 7. free -m
```
[pengzhangjie]$ free -m
             total       used       free     shared    buffers     cached
Mem:         48135      14972      33162          0        461       5622
-/+ buffers/cache:       8889      39246
Swap:         7629          0       7629

```
右边的两列显示：
buffers：用于块设备I/O的缓冲区缓存
cached：用于文件系统的页缓存
它们的值接近于0时，往往导致较高的磁盘I/O（可以通过iostat确认）和糟糕的性能。上面的例子里没有这个问题，每一列都有好几M呢。

比起第一行，-/+ buffers/cache提供的内存使用量会更加准确些。Linux会把暂时用不上的内存用作缓存，一旦应用需要的时候立刻重新分配给它。所以部分被用作缓存的内存其实也算是空闲内存，第二行以此修订了实际的内存使用量。为了解释这一点， 甚至有人专门建了个网站： linuxatemyram。

如果你在Linux上安装了ZFS，正如我们在一些服务上所做的，这一点会变得更加迷惑，因为ZFS它自己的文件系统缓存不算入free -m。有时系统看上去已经没有多少空闲内存可用了，其实内存都待在ZFS的缓存里呢。

### 8. sar -n DEV 1
```
[pengzhangjie]$ sar -n DEV 1
Linux 2.6.32-504.16.2.el6.x86_64 (centos) 	09/19/2018 	_x86_64_	(16 CPU)

08:25:37 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
08:25:38 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
08:25:38 PM      eth0   2107.07   1957.58    347.10   1035.85      0.00      0.00      0.00
08:25:38 PM      eth1      4.04      1.01      0.27      0.10      0.00      0.00      0.00
08:25:38 PM      eth2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
08:25:38 PM      eth3      0.00      0.00      0.00      0.00      0.00      0.00      0.00

```
这个命令显示一些关键TCP指标的汇总。其中包括：
active/s：本地每秒创建的TCP连接数（比如concept()创建的）
passive/s：远程每秒创建的TCP连接数（比如accept()创建的）
retrans/s：每秒TCP重传次数

主动连接数（active）和被动连接数（passive）通常可以用来粗略地描述系统负载。可以认为主动连接是对外的，而被动连接是对内的，虽然严格来说不完全是这个样子。（比如，一个从localhost到localhost的连接）

重传是网络或系统问题的一个信号；它可能是不可靠的网络（比如公网）所造成的，也有可能是服务器已经过载并开始丢包。在上面的例子中，每秒只创建一个新的TCP连接。

### 9. sar -n TCP,ETCP 1
```
[pengzhangjie]$ sar -n TCP,ETCP 1
Linux 2.6.32-504.16.2.el6.x86_64 (centos) 	09/19/2018 	_x86_64_	(16 CPU)

08:32:27 PM  active/s passive/s    iseg/s    oseg/s
08:32:28 PM      0.00      0.00   1602.02   1254.55
```
这个命令显示一些关键TCP指标的汇总。其中包括：
active/s：本地每秒创建的TCP连接数（比如concept()创建的）
passive/s：远程每秒创建的TCP连接数（比如accept()创建的）
retrans/s：每秒TCP重传次数

主动连接数（active）和被动连接数（passive）通常可以用来粗略地描述系统负载。可以认为主动连接是对外的，而被动连接是对内的，虽然严格来说不完全是这个样子。（比如，一个从localhost到localhost的连接）

重传是网络或系统问题的一个信号；它可能是不可靠的网络（比如公网）所造成的，也有可能是服务器已经过载并开始丢包。在上面的例子中，每秒只创建一个新的TCP连接。

### 10. top

```
[pengzhangjie]$ top
top - 20:11:53 up 148 days,  8:10,  1 user,  load average: 0.06, 0.08, 0.03
Tasks: 316 total,   1 running, 315 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.9%us,  0.3%sy,  0.0%ni, 98.8%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:  49290640k total, 11046268k used, 38244372k free,   471000k buffers
Swap:  7813116k total,        0k used,  7813116k free,  5813764k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                                                                                                  
23715 rabbitmq  20   0 9715m 259m 4212 S 113.4  0.5  29651:55 beam.smp                                                                                                                 
45098 root      20   0 4259m 3.1g 1244 S  3.9  6.6  32:10.98 fts_shared_redi                                                                                                           
23936 root      20   0 1915m  44m 4272 S  2.0  0.1 639:18.96 yyac_worker_ant 
```

- 指定显示进程 9836，每隔 5 秒的进程的资源的占用情况

```
# top –d 5 –p 9836 -c

pstack
```
