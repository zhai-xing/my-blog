---
title: 异常排查
permalink: permalink
sticky: true
cover: ""
date: 2025-25-05
description: dexc
tags:
  - java
  - linux
  - 事故排查
categories:
  - Linux
---
## day1
### 工具：
1. java: jstack,jmap,arthas
2. jvm: gdb,strace,arthas
3. 操作系统: perf,bcc,bpftrace
4. 硬件资源：top,free iostat,iftop
5. 网络: ngrep  tcpdump
6. mysql: perf,gdb,bcc,bptrace,strace

### 排查思路：
资源占用分析视角：

自低向上：

硬件资源使用率如何？如果发现某个资源占用率高,逐步向上分析

1. 什么进程占用cpu多
2. 什么线程占用cpu多
3. 什么代码占用cpu多



系统负载分析视角：

接口的QPS多少，耗时怎么样，有多少报错，

1. controller层耗时高吗
2. mappe层处理耗时高吗
3. sql执行耗时高吗
4. 逐步向下分析时间占用

排查思路：

看日志->资源占用->排查工具 

tps：其实80%的问题都会直接在日志中呈现出来，所以有一套日志分析平台很重要



## day2
如何看日志：

命令：cat head,tail

cat：输出文件内容  cat /etc/t1.txt

head：查看文件前几行  示例：head -n2  /etc/t1.txt

tail 查看文件后几行，-f不断查看新内容 tail -n2  /etc/t1.txt



命令：vim

vim是文本编辑工具。基操



命令: less/more

分页文件查看器，因此可以查看大日志文件，因为是按需加载文件到内存。推荐使用less，相比more功能更多，



grep命令：指定搜索关键字，可以从日志中搜索出包含关键字的日志，-i是忽略大小写 示例： grep -i error app.log



awk命令：

awk其实是为文本处理的脚本语言，后面简单学一下



wc 用于对文本文件进行统计

sort 用于对文本行进行排序与

uniq 用于对文本去重

## 硬件资源观测
top命令

![](https://cdn.nlark.com/yuque/0/2025/png/39206930/1744816894743-43a8a503-1253-4648-b457-b6bda35425aa.png)

红色指标:  系统的负载，分别是1分钟 5分钟15分钟

黄色：系统任务状态，总任务，运行中，睡眠中，暂停，僵尸任务

绿色：cpu使用率

us：非niced进程(用户态)花费的cpu时间占比

sy：内核进程花费的cpu时间占比

ni niced进程花费的cpu时间占比

id：内核空闲进程花费的cpu时间占比

wa：等待磁盘io完成花费的cpu时间占比

hi ：处理硬件中断

si ：处理软件中断

st：被其他虚拟机偷取的cpu时间占比

![](https://cdn.nlark.com/yuque/0/2025/png/39206930/1744817426818-d5ea7060-68ec-4f7e-8fe9-cbfd027cc879.png)

mem

total 总内存大小

free：空闲内容大小

used：使用中的内存大小

buff/cache:用于文件缓存与系统缓存的内存大小



swap:

total 总swap文件大小

free：swap空闲内容大小

used：swap使用中的内存大小

avail Mem:可用内存大小，和swap无关

top是一个交互式的命令

也就是使用了top 可以按对应的键看不同信息

1 查看1号cpu的各核使用情况

M 进程按内存使用倒序

P 进程按cpu使用倒序

H 查看线程情况

c 查看进程的完整命令行

<,>,R 调整排序列

o,= o添加过滤条件，例如 command=java 

q 退出top



vmstat 命令

1s显示一次，第一行是系统启动以来的统计信息，一般可以忽略不看

![](https://cdn.nlark.com/yuque/0/2025/png/39206930/1744818599063-7561a10a-64b7-4630-8105-dbf23d0f0ac8.png)

r:cpu运行队列长度，也就是有多少线程等待操作系统调度运行。

b:不可中断阻塞的线程数量，一般就算阻塞于io访问的线程数量

swpd：内存交换到磁盘的内存大小

free：剩余内存大小 单位kb

buff：用于buff的内存大小，单位kb

cache：用于文件页面缓存的内存大小 单位kb

si: 磁盘换入到内存的当前速度，单位kb/s

so内存换出到磁盘的当前速度，单位kb/s

bi 每秒读取磁盘块数量，blocks/s

bo 每秒写入的磁盘块数量 block/s

in 每秒中断数量

cs每秒切换上下文次数

us cpu用户态使用率

sy cpu内核态使用率

id cpu空闲率

wa 等待io 线程被阻塞 等待磁盘io时的cpu空闲时间占比

st  steal偷取，cpu在虚拟化环境下在其他租户上的开销



free命令 查看内存使用 -m是mb为单位

![](https://cdn.nlark.com/yuque/0/2025/png/39206930/1744819579276-4c3c4406-fe92-4a5f-80c5-5d2c8a2ec4bf.png)

和top中内存部分类似，有一个经验知识：

1. free: Linux中随着使用时间越长，free会越来越小，因为linux会把访问的文件数据经可能缓存在内存中，
2. availble 系统真正的可用内存，约等于free+buff/cache



slabtop命令 

slab是linux基于对象的一种内存分配机制，类似于对象池的概念，当linux需要一个内核对象时，如果slab中有，就可以直接获取对象，而不用申请内存块



磁盘资源：

df命令:![](https://cdn.nlark.com/yuque/0/2025/png/39206930/1744820505844-3e42468d-c635-4e85-9f69-b4b14b6706ac.png)

iostat:命令

### sar命令
sar是应该全方位的命令，这个命令背熟了可以替代前面的命令 ，使用  yum install sysstat 安装

#### cpu观测
命令 sar -u  例如 sar -u 1 3 每间隔1秒统计1次 共统计3次

```java
[root@localhost ~]# sar -q 1 3
Linux 3.10.0-1160.el7.x86_64 (localhost.localdomain)    2025年04月26日  _x86_64_        (2 CPU)
11时46分49秒   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
11时46分50秒         0       187      0.01      0.04      0.05         0
11时46分51秒         0       187      0.01      0.04      0.05         0
11时46分52秒         0       187      0.01      0.04      0.05         0
平均时间:         0       187      0.01      0.04      0.05         0
```

<font style="color:rgb(0, 0, 0);">%user #用户空间的CPU使用</font>  
<font style="color:rgb(0, 0, 0);">%nice 改变过优先级的进程的CPU使用率</font>  
<font style="color:rgb(0, 0, 0);">%system 内核空间的CPU使用率</font>  
<font style="color:rgb(0, 0, 0);">%iowait CPU等待IO的百分比</font>  
<font style="color:rgb(0, 0, 0);">%steal 虚拟机的虚拟机CPU使用的CPU</font>  
<font style="color:rgb(0, 0, 0);">%idle 空闲的CPU</font>  
<font style="color:rgb(0, 0, 0);">在以上的显示当中，主要看%iowait和%idle，%iowait过高表示存在I/O瓶颈，即磁盘IO无法满足业务需求，如果%idle过低表示CPU使用率比较严重，需要结合内存使用等情况判断CPU是否瓶颈。</font>

#### <font style="color:rgb(0, 0, 0);">平均负载统计分析</font>
命令 sar -q 例如 sar -q 1 3  每间隔1秒统计1次 共统计3次

```java
[root@localhost ~]# sar -q 1 3
Linux 3.10.0-1160.el7.x86_64 (localhost.localdomain)    2025年04月26日  _x86_64_        (2 CPU)
11时46分49秒   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
11时46分50秒         0       187      0.01      0.04      0.05         0
11时46分51秒         0       187      0.01      0.04      0.05         0
11时46分52秒         0       187      0.01      0.04      0.05         0
平均时间:         0       187      0.01      0.04      0.05         0
```

<font style="color:rgb(0, 0, 0);">runq-sz 运行队列的长度（等待运行的进程数，每核的CP不能超过3个）</font>  
<font style="color:rgb(0, 0, 0);">plist-sz 进程列表中的进程（processes）和线程数（threads）的数量</font>  
<font style="color:rgb(0, 0, 0);">ldavg-1 最后1分钟的CPU平均负载，即将多核CPU过去一分钟的负载相加再除以核心数得出的平均值，5分钟和15分钟以此类推</font>  
<font style="color:rgb(0, 0, 0);">ldavg-5 最后5分钟的CPU平均负载</font>  
<font style="color:rgb(0, 0, 0);">ldavg-15 最后15分钟的CPU平均负载</font>

#### <font style="color:rgb(0, 0, 0);">内存统计分析</font>
sar -r 1 3     每间隔1秒统计1次 共统计3次

```java
[root@localhost ~]# sar -r 1 3
Linux 3.10.0-1160.el7.x86_64 (localhost.localdomain)    2025年04月26日  _x86_64_        (2 CPU)

11时50分04秒 kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
11时50分05秒   3262412    618100     15.93      2108    291468    306820      3.86    244056    125584        24
11时50分06秒   3262116    618396     15.94      2108    291472    307088      3.87    244136    125584        28
11时50分07秒   3262288    618224     15.93      2108    291476    306820      3.86    243952    125584        20
平均时间:   3262272    618240     15.93      2108    291472    306909      3.86    244048    125584        24
[root@localhost ~]# 
```

<font style="color:rgb(0, 0, 0);">kbmemfree 空闲的物理内存大小  
</font><font style="color:rgb(0, 0, 0);">kbmemused 使用中的物理内存大小  
</font><font style="color:rgb(0, 0, 0);">%memused 物理内存使用率  
</font><font style="color:rgb(0, 0, 0);">kbbuffers 内核中作为缓冲区使用的物理内存大小，kbbuffers和kbcached:这两个值就是free命令中的buffer和cache.  
</font><font style="color:rgb(0, 0, 0);">kbcached 缓存的文件大小  
</font><font style="color:rgb(0, 0, 0);">kbcommit 保证当前系统正常运行所需要的最小内存，即为了确保内存不溢出而需要的最少内存（物理内存+Swap分区）  
</font><font style="color:rgb(0, 0, 0);">commit 这个值是kbcommit与内存总量（物理内存+swap分区）的一个百分比的值</font>

#### <font style="color:rgb(0, 0, 0);">统计swap分区</font>
<font style="color:rgb(0, 0, 0);">查看系统swap分区的统计信息：每间隔1秒钟统计一次总共统计三次：#sar -W 1 3</font>

```java
[root@localhost ~]# sar -W 1 3
Linux 3.10.0-1160.el7.x86_64 (localhost.localdomain)    2025年04月26日  _x86_64_        (2 CPU)

11时53分42秒  pswpin/s pswpout/s
11时53分43秒      0.00      0.00
11时53分44秒      0.00      0.00
11时53分45秒      0.00      0.00
平均时间:      0.00      0.00

```

<font style="color:rgb(0, 0, 0);">pswpin/s 每秒从交换分区到系统的交换页面（swap page）数量</font>  
<font style="color:rgb(0, 0, 0);">pswpott/s 每秒从系统交换到swap的交换页面（swap page）的数量</font>

#### <font style="color:rgb(0, 0, 0);">查看磁盘IO</font>
<font style="color:rgb(0, 0, 0);">sar -b #查看I/O和传递速率的统计信息，</font>

```java
[root@localhost ~]# sar -b 1 3
Linux 3.10.0-1160.el7.x86_64 (localhost.localdomain)    2025年04月26日  _x86_64_        (2 CPU)

11时55分17秒       tps      rtps      wtps   bread/s   bwrtn/s
11时55分18秒      0.00      0.00      0.00      0.00      0.00
11时55分19秒      0.00      0.00      0.00      0.00      0.00
11时55分20秒      2.00      0.00      2.00      0.00     42.00
平均时间:      0.67      0.00      0.67      0.00     14.00
[root@localhost ~]# 

```

<font style="color:rgb(0, 0, 0);">tps 磁盘每秒钟的IO总数，等于iostat中的tps</font>  
<font style="color:rgb(0, 0, 0);">rtps 每秒钟从磁盘读取的IO总数</font>  
<font style="color:rgb(0, 0, 0);">wtps 每秒钟从写入到磁盘的IO总数</font>  
<font style="color:rgb(0, 0, 0);">bread/s 每秒钟从磁盘读取的块总数</font>  
<font style="color:rgb(0, 0, 0);">bwrtn/s 每秒钟此写入到磁盘的块总数</font>

#### <font style="color:rgb(0, 0, 0);">磁盘使用详情统计</font>
<font style="color:rgb(0, 0, 0);">sar -d #磁盘使用详情统计</font>

```java
[root@localhost ~]# sar -d
Linux 3.10.0-1160.el7.x86_64 (localhost.localdomain)    2025年04月26日  _x86_64_        (2 CPU)

11时40分02秒       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
11时50分01秒    dev8-0      0.80      0.00     21.61     27.14      0.00      3.87      1.82      0.14
11时50分01秒  dev253-0      0.83      0.00     21.55     25.97      0.00      3.83      1.75      0.15
11时50分01秒  dev253-1      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
平均时间:    dev8-0      0.80      0.00     21.61     27.14      0.00      3.87      1.82      0.14
平均时间:  dev253-0      0.83      0.00     21.55     25.97      0.00      3.83      1.75      0.15
平均时间:  dev253-1      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
[root@localhost ~]# 

```

<font style="color:rgb(0, 0, 0);">DEV 磁盘设备的名称，如果不加-p，会显示dev253-0类似的设备名称，因此加上-p显示的名称更直接</font>  
<font style="color:rgb(0, 0, 0);">tps：每秒I/O的传输总数</font>  
<font style="color:rgb(0, 0, 0);">rd_sec/s 每秒读取的扇区的总数</font>  
<font style="color:rgb(0, 0, 0);">wr_sec/s 每秒写入的扇区的总数</font>  
<font style="color:rgb(0, 0, 0);">avgrq-sz 平均每次次磁盘I/O操作的数据大小（扇区）</font>  
<font style="color:rgb(0, 0, 0);">avgqu-sz 磁盘请求队列的平均长度</font>  
<font style="color:rgb(0, 0, 0);">await 从请求磁盘操作到系统完成处理，每次请求的平均消耗时间，包括请求队列等待时间，单位是毫秒（1秒等于1000毫秒），等于寻道时间+队列时间+服务时间</font>  
<font style="color:rgb(0, 0, 0);">svctm I/O的服务处理时间，即不包括请求队列中的时间</font>  
<font style="color:rgb(0, 0, 0);">%util I/O请求占用的CPU百分比，值越高，说明I/O越慢</font>

#### 网络使用分析
<font style="color:rgb(0, 0, 0);">sar -n #统计网络信息</font>  
<font style="color:rgb(0, 0, 0);">sar -n选项使用6个不同的开关：DEV，EDEV，NFS，NFSD，SOCK，IP，EIP，ICMP，EICMP，TCP，ETCP，UDP，SOCK6，IP6，EIP6，ICMP6，EICMP6和UDP6 ，DEV显示网络接口信息，EDEV显示关于网络错误的统计数据，NFS统计活动的NFS客户端的信息，NFSD统计NFS服务器的信息，SOCK显示套接字信息，ALL显示所有5个开关。它们可以单独或者一起使用。</font>

```java
[root@localhost ~]# sar -n DEV 1 3
Linux 3.10.0-1160.el7.x86_64 (localhost.localdomain)    2025年04月26日  _x86_64_        (2 CPU)

11时58分50秒     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
11时58分51秒    ens192     21.00     24.00      3.23      5.07      0.00      0.00      0.00
11时58分51秒        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00

11时58分51秒     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
11时58分52秒    ens192     29.00     35.00      3.25      9.99      0.00      0.00      0.00
11时58分52秒        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
^C

11时58分52秒     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
11时58分53秒    ens192     15.22     21.74      1.53      2.74      0.00      0.00      0.00
11时58分53秒        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00

平均时间:     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
平均时间:    ens192     21.92     27.05      2.70      6.02      0.00      0.00      0.00
平均时间:        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
[root@localhost ~]# 

```

<font style="color:rgb(0, 0, 0);">IFACE 网卡名称  
</font><font style="color:rgb(0, 0, 0);">rxerr/s 每秒钟接收到的损坏的数据包  
</font><font style="color:rgb(0, 0, 0);">txerr/s 每秒钟发送的数据包错误数  
</font><font style="color:rgb(0, 0, 0);">coll/s 当发送数据包时候，每秒钟发生的冲撞（collisions）数，这个是在半双工模式下才有  
</font><font style="color:rgb(0, 0, 0);">rxdrop/s 当由于缓冲区满的时候，网卡设备接收端每秒钟丢掉的网络包的数目  
</font><font style="color:rgb(0, 0, 0);">txdrop/s 当由于缓冲区满的时候，网络设备发送端每秒钟丢掉的网络包的数目  
</font><font style="color:rgb(0, 0, 0);">txcarr/s 当发送数据包的时候，每秒钟载波错误发生的次数  
</font><font style="color:rgb(0, 0, 0);">rxfram 在接收数据包的时候，每秒钟发生的帧对其错误的次数  
</font><font style="color:rgb(0, 0, 0);">rxfifo 在接收数据包的时候，每秒钟缓冲区溢出的错误发生的次数  
</font><font style="color:rgb(0, 0, 0);">txfifo 在发生数据包 的时候，每秒钟缓冲区溢出的错误发生的次数  
</font><font style="color:rgb(0, 0, 0);">1.10.3：sar -n SOCK 1 1 #统计socket连接信息</font>

<font style="color:rgb(0, 0, 0);">sar -n SOCK 1 1 #统计socket连接信息  
</font><font style="color:rgb(0, 0, 0);">totsck 当前被使用的socket总数  
</font><font style="color:rgb(0, 0, 0);">tcpsck 当前正在被使用的TCP的socket总数  
</font><font style="color:rgb(0, 0, 0);">udpsck 当前正在被使用的UDP的socket总数  
</font><font style="color:rgb(0, 0, 0);">rawsck 当前正在被使用于RAW的skcket总数  
</font><font style="color:rgb(0, 0, 0);">if-frag 当前的IP分片的数目  
</font><font style="color:rgb(0, 0, 0);">tcp-tw TCP套接字中处于TIME-WAIT状态的连接数量  
</font><font style="color:rgb(0, 0, 0);">########如果你使用FULL关键字，相当于上述DEV、EDEV和SOCK三者的综合</font>

<font style="color:rgb(0, 0, 0);">sar -n TCP 1 3 #TCP连接的统计  
</font><font style="color:rgb(0, 0, 0);">active/s 新的主动连接  
</font><font style="color:rgb(0, 0, 0);">passive/s 新的被动连接  
</font><font style="color:rgb(0, 0, 0);">iseg/s 接受的段  
</font><font style="color:rgb(0, 0, 0);">oseg/s 输出的段</font>

<font style="color:rgb(0, 0, 0);">sar -n 使用总结</font>

<font style="color:rgb(0, 0, 0);">-n DEV ： 网络接口统计信息。  
</font><font style="color:rgb(0, 0, 0);">-n EDEV ： 网络接口错误。  
</font><font style="color:rgb(0, 0, 0);">-n IP ： IP数据报统计信息。  
</font><font style="color:rgb(0, 0, 0);">-n EIP ： IP错误统计信息。  
</font><font style="color:rgb(0, 0, 0);">-n TCP ： TCP统计信息。  
</font><font style="color:rgb(0, 0, 0);">-n ETCP ： TCP错误统计信息。  
</font><font style="color:rgb(0, 0, 0);">-n SOCK ： 套接字使用。</font>

#### <font style="color:rgb(0, 0, 0);">进程，文件状态</font>
<font style="color:rgb(0, 0, 0);">sar -v #进程、inode、文件和锁表状态 ，每间隔1秒钟统计一次总共统计三次：#sar -v 1 3</font>

```java
[root@localhost ~]# sar -v 1 3
Linux 3.10.0-1160.el7.x86_64 (localhost.localdomain)    2025年04月26日  _x86_64_        (2 CPU)

11时59分59秒 dentunusd   file-nr  inode-nr    pty-nr
12时00分00秒     22736      1280     26803         2
12时00分01秒     22747      1280     26810         2
12时00分02秒     22747      1280     26810         2
平均时间:     22743      1280     26808         2
```

<font style="color:rgb(0, 0, 0);">dentunusd 在缓冲目录条目中没有使用的条目数量  
</font><font style="color:rgb(0, 0, 0);">file-nr 被系统使用的文件句柄数量  
</font><font style="color:rgb(0, 0, 0);">inode-nr 已经使用的索引数量  
</font><font style="color:rgb(0, 0, 0);">pty-nr 使用的pty数量</font>

#### 扩展
```java
默认监控: sar 5 5     //  CPU和IOWAIT统计状态 
(1) sar -b 5 5        // IO传送速率
(2) sar -B 5 5        // 页交换速率
(3) sar -c 5 5        // 进程创建的速率
(4) sar -d 5 5        // 块设备的活跃信息
(5) sar -n DEV 5 5    // 网路设备的状态信息
(6) sar -n SOCK 5 5   // SOCK的使用情况
(7) sar -n ALL 5 5    // 所有的网络状态信息
(8) sar -P ALL 5 5    // 每颗CPU的使用状态信息和IOWAIT统计状态 
(9) sar -q 5 5        // 队列的长度（等待运行的进程数）和负载的状态
(10) sar -r 5 5       // 内存和swap空间使用情况
(11) sar -R 5 5       // 内存的统计信息（内存页的分配和释放、系统每秒作为BUFFER使用内存页、每秒被cache到的内存页）
(12) sar -u 5 5       // CPU的使用情况和IOWAIT信息（同默认监控）
(13) sar -v 5 5       // inode, file and other kernel tablesd的状态信息
(14) sar -w 5 5       // 每秒上下文交换的数目
(15) sar -W 5 5       // SWAP交换的统计信息(监控状态同iostat 的si so)
(16) sar -x 2906 5 5  // 显示指定进程(2906)的统计信息，信息包括：进程造成的错误、用户级和系统级用户CPU的占用情况、运行在哪颗CPU上
(17) sar -y 5 5       // TTY设备的活动状态
```



### USE观测法
使用率

饱和度

错误

以Java线程池为例，

1. 线程池是一个软件资源，
2. 线程池的使用率：正在运行线程数占线程池总线程数百分比
3. 线程池饱和度：目前正在线程池任务队列中排队的任务数量
4. 线程池的错误：触发线程池拒绝策略的次数





## 场景问题分析
CPU占用高：

适合自下向上去分析:

1. 用户态还是内核态高 
2. 什么进程占用cpu多  
3. 什么线程占用多 然
4. 什么调用栈占用cpu多
5. 什么请求占用cpu多
6. ![](https://cdn.nlark.com/yuque/0/2025/png/39206930/1745642060360-37129ef5-fb6f-45c4-a59d-5a90db48c421.png)



示例：通过top命令可以识别出是进程(用户态)占用高  还是内核占用高



示例 就可以使用 top发现是那个进程占用高， 然后使用top -h -p pid 就可以发现是什么线程占用高

示例 可以使用jstack 查看线程栈  jstack pid >stack.log 然后可以使用awk过滤指定线程栈

找线程和找线程栈是分两步进行的，但是可能来不及， 可以使用arthas的命令去 它可以将两步合并为一步

arthas是基于Java的，对于非Java进程，可以使用perf record或者bcc中的命令

### 系统态（内核态）cpu高：
系统态主要是 系统调用，中断、内核线程 这三部分组成

#### 系统调用：
系统调用是操作系统提供给用户空间程序的一种接口，允许程序请求操作系统的服务，例如访问磁盘(open,read,write) 网络(socket/recvfrom/sendto）等 内核收到系统调用请求后会先切换到内核态，然后执行相应处理逻辑

系统调用：由于系统调用是由用户进程触发，可以通过bcc的syscount命令来检查系统调用情况，以及那些进程的系统调用多，

#### 中断：
是指计算机在执行程序过程中，由于某些紧急事件或外部设备的请求，暂时停止当前程序执行，转而去处理更优先的任务，处理完这个紧急任务后，计算机系统会返回到被中断地方，继续执行，比如磁盘中断、网络中断

1. 硬中断：由硬件之间触发，由于硬中断执行过程中，一般需要禁用中断后再处理，因此硬中断需要能快速执行完以避免其他中断信号丢失
2. 软中断：为了尽可能块执行完硬中断，减少禁用时间，linux设计实现了软中断，用于处理中断中可以延迟处理的部分

中断可以使用bcc工具 hardirqs和softirqs来诊断

#### 内核线程：
linux内核管理着1批内核线程，用来执行特定的任务，如kswapd处理内存回收，ksoftirqd处理软中断等。

内核线程： 内核线程可以通过pidstat的%system来确认，内核线程一般是以k开头

### 空闲cpu使用率高
情况1 id%使用率高

这种是空闲cpu使用 

1. 系统负载不大，可能是没有业务流量，或者是网络丢包导致流量未进系统，例如tcp连接队列或者socket接受缓存配置过小
2. 线程都被阻塞，无线程占用cpu资源，这种情况下请求耗时会增加，

情况2 wa%使用率高

1. 线程阻塞再磁盘访问上，检查磁盘性能指标 使用iostat或者ioping命令检查磁盘性能是否退化
2. 使用了nfs之类的网络存储 由于网络较慢 wa高可能也是正常的



![](https://cdn.nlark.com/yuque/0/2025/png/39206930/1745642348328-357af96e-a70b-48b7-b3a3-bc2e158b577b.png)

![](https://cdn.nlark.com/yuque/0/2025/png/39206930/1745642233132-ff5ae61a-e6ba-49da-a82f-ddd235b86621.png)

![](https://cdn.nlark.com/yuque/0/2025/png/39206930/1745642262206-d910e0ee-251c-412e-a239-4300858dc932.png)







