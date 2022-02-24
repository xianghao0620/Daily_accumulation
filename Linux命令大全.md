监控命令

Top

3>$ top
top - 12:46:04 up 860 days,  1:31,  1 user,  load average: 0.18, 0.31, 0.38
Tasks: 204 total,   1 running, 202 sleeping,   1 stopped,   0 zombie
%Cpu(s):  0.8 us,  0.8 sy,  0.0 ni, 98.5 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 13186284+total, 62092372 free, 38641888 used, 31128584 buff/cache
KiB Swap:        0 total,        0 free,        0 used. 91412856 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 5924 tstadmin  20   0   10.3g   1.1g   6784 S  12.5  0.9  14086:53 java
 9552 tstadmin  20   0   11.0g 992892   6632 S  12.5  0.8  24097:43 java
  527 tstadmin  20   0   10.2g   1.1g   6868 S   6.2  0.8   6193:09 java
 3128 root      20   0  319296  78800   3124 S   6.2  0.1  10673:05 ilogtail
第一行 -- 任务队列信息

12:46:04 当前时间
up 860 days 系统运行时间
1 user 当前登录用户数
load average: 0.18, 0.31, 0.38 系统负载（任务队列的平均长度），分别是1分钟、5分钟、15分钟到现在的平均值
正常情况下，系统负载应该在 0.7*n 以下（n 为 CPU 核心数）

第二行 -- 进程信息

204 total 进程总数
1 running 正在运行的进程数
202 sleeping 睡眠进程数
0 stopped 停止进程数
0 zombie 僵尸进程数
第三行 -- CPU信息

0.8%us 用户空间占CPU百分比
0.8%sy 内核空间占CPU百分比
0.0%ni 用户进程空间内改变过优先级的进程占用CPU百分比
98.5%id 空闲CPU百分比
0.0%wa 等待输入输出的CPU时间百分比
0.0%hi 硬件中断占CPU时间百分比
0.0%si 软件终端占CPU时间百分比
0.0%st 提供给虚拟化环境执行占CPU时间百分比
第四行 -- 内存信息

13186284 total 物理内存总量
62092372 used 使用的物理内存总量
38641888 free 空闲内存总量
31128584 buffers 用作内核缓存的内存量
第五行 -- 内存交换区信息

0 total 交换区总容量
0 used 使用交换区的总量
0 free 空闲交换区总量
进程信息

PID 进程ID
PPID 父进程ID
RUSER real user name
UID 进程所有者用户ID
USER 进程所有者用户名
GROUP 进程所有者组名
TTY 启动进程的终端名
PR 优先级
NI nice值，负数表示高优先级，正数表示低优先级
P 最后使用的CPU，仅用于多 CPU 环境
%CPU 上次更新到现在的 CPU 时间占用百分比
TIME 进程使用的CPU时间总计（以秒为单位）
TIME+ 进程使用的CPU时间总计（以1/100秒为单位）
%MEM 进程使用物理内存百分比
VIRT 进程使用虚拟内存总量（以KB为单位） VIRT=SWAP+RES
SWAP 进程使用的虚拟内存中，被换出的大小
RES 进程使用的未被换出的物理内存大小（以KB为单位） RES=CODE+DATA
CODE 可执行代码占用物理内存总大小
DATA 数据段+栈占用的物理内存总大小
SHR 共享内存总大小
nFLT 页面错误次数
nDRT 最后一次写入到现在，被修改过的页面数
S 进程状态
COMMAND 命令名/命令行
WCHAN 若进程在睡眠，则显示睡眠中的系统函数名
Flags 任务标志，参考 sched.h
快捷键

输入c，按cpu使用率排序
输入m，按内存使用率排序
vmstat

vmstat 是一个标准的工具，它会报告 Linux 系统的虚拟内存统计。vmstat 会报告有关进程、内存、分页、块 IO、陷阱（中断）和 cpu 活动的信息。它可以帮助 Linux 管理员在解决问题时识别系统瓶颈。

vmstat [间隔秒数] [次数]

-S m 参数以获取以兆字节为单位的统计

8>$ vmstat 1 5 -S m
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0  63580    191  31732    0    0     0    22    0    0  2  1 96  0  0
 0  0      0  63580    191  31732    0    0     0     4 16075 29643  1  1 98  0  0
 0  0      0  63579    191  31732    0    0     0     0 23028 36191  3  1 95  0  0
 1  0      0  63579    191  31732    0    0     0   112 18460 31581  2  1 97  0  0
 1  0      0  63578    191  31733    0    0     0     0 21290 34749  3  2 95  0  0
r 表示运行队列(就是说多少个进程真的分配到CPU)，我测试的服务器目前CPU比较空闲，没什么程序在跑，当这个值超过了CPU数目，就会出现CPU瓶颈了。
b 表示阻塞的进程。
swpd 虚拟内存已使用的大小，如果大于0，表示你的机器物理内存不足了，如果不是程序内存泄露的原因，那么你该升级内存了或者把耗内存的任务迁移到其他机器。
free 空闲的物理内存的大小。
buff Linux/Unix系统是用来存储，目录里面有什么内容，权限等的缓存
cache cache直接用来记忆我们打开的文件,给文件做缓冲，(这里是Linux/Unix的聪明之处，把空闲的物理内存的一部分拿来做文件和目录的缓存，是为了提高 程序执行的性能，当程序使用内存时，buffer/cached会很快地被使用。)
si 从磁盘交换的内存量（换入，从 swap 移到实际内存的内存）。如果这个值大于0，表示物理内存不够用或者内存泄露了，要查找耗内存进程解决掉。
so 交换到磁盘的内存量（换出，从实际内存移动到 swap 的内存）。
bi 块设备每秒接收的块数量，这里的块设备是指系统上所有的磁盘和其他块设备，默认块大小是1024byte
bo 块设备每秒发送的块数量，例如我们读取文件，bo就要大于0。bi和bo一般都要接近0，不然就是IO过于频繁，需要调整。
in 每秒CPU的中断次数，包括时间中断。这两个值越大，会看到由内核消耗的cpu时间sy会越多
cs 每秒上下文切换次数，例如我们调用系统函数，就要进行上下文切换，线程的切换，也要进程上下文切换，这个值要越小越好，太大了，要考虑调低线程或者进程的数目,例如在apache和nginx这种web服务器中，我们一般做性能测试时会进行几千并发甚至几万并发的测试，选择web服务器的进程可以由进程或者线程的峰值一直下调，压测，直到cs到一个比较小的值，这个进程和线程数就是比较合适的值了。系统调用也是，每次调用系统函数，我们的代码就会进入内核空间，导致上下文切换，这个是很耗资源，也要尽量避免频繁调用系统函数。上下文切换次数过多表示你的CPU大部分浪费在上下文切换，导致CPU干正经事的时间少了，CPU没有充分利用，是不可取的。
us us的值比较高时，说明用户进程消耗的cpu时间多，但是如果长期超过50%的使用，那么我们就该考虑优化程序算法或其他措施了
sy 值过高时，说明系统内核消耗的cpu资源多，这个不是良性的表现，我们应该检查原因。这里us + sy的参考值为80%，如果us+sy 大于 80%说明可能存在CPU不足
id 空闲 CPU时间，一般来说，id + us + sy = 100,一般我认为id是空闲CPU使用率，us是用户CPU使用率，sy是系统CPU使用率。
wa wa过高时，说明io等待比较严重，这可能是由于磁盘大量随机访问造成的，也有可能是磁盘的带宽出现瓶颈。
CPU 利用率高 &&平均负载高

vmstat 查看详细的 CPU 利用率。用户态 CPU 利用率（us）较高，说明用户态进程占用了较多的 CPU，如果这个值长期大于 50%，应该着重排查应用本身的性能问题。内核态 CPU 利用率（sy）较高，说明内核态占用了较多的 CPU，所以应该着重排查内核线程或者系统调用的性能问题。如果 us + sy 的值大于 80%，说明 CPU 可能不足。

CPU 利用率低 &&平均负载高

如果 CPU 利用率不高，说明我们的应用并没有忙于计算，而是在干其他的事。CPU 利用率低而平均负载高，常见于 I/O 密集型进程，这很容易理解，毕竟平均负载就是 R 状态进程和 D 状态进程的和，除掉了第一种，就只剩下 D 状态进程了（产生 D 状态的原因一般是因为在等待 I/O，例如磁盘 I/O、网络 I/O 等）。

排查 &&验证思路：使用 vmstat 1 定时输出系统资源使用，观察 %wa(iowait) 列的值，该列标识了磁盘 I/O 等待时间在 CPU 时间片中的百分比，如果这个值超过 30%，说明磁盘 I/O 等待严重，这可能是大量的磁盘随机访问或直接的磁盘访问（没有使用系统缓存）造成的，也可能磁盘本身存在瓶颈，可以结合 iostat 或 dstat 的输出加以验证，如 %wa(iowait) 升高同时观察到磁盘的读请求很大，说明可能是磁盘读导致的问题。

此外，耗时较长的网络请求（即网络 I/O）也会导致 CPU 平均负载升高，如 MySQL 慢查询、使用 RPC 接口获取接口数据等。这种情况的排查一般需要结合应用本身的上下游依赖关系以及中间件埋点的 trace 日志，进行综合分析。

CPU 上下文切换次数变高

先用 vmstat 查看系统的上下文切换次数，然后通过 pidstat 观察进程的自愿上下文切换（cswch）和非自愿上下文切换（nvcswch）情况。自愿上下文切换，是因为应用内部线程状态发生转换所致，譬如调用 sleep()、join()、wait()等方法，或使用了 Lock 或 synchronized 锁结构；非自愿上下文切换，是因为线程由于被分配的时间片用完或由于执行优先级被调度器调度所致。

如果自愿上下文切换次数较高，意味着 CPU 存在资源获取等待，比如说，I/O、内存等系统资源不足等。如果是非自愿上下文切换次数较高，可能的原因是应用内线程数过多，导致 CPU 时间片竞争激烈，频频被系统强制调度，此时可以结合 jstack 统计的线程数和线程状态分布加以佐证。

iostat

iostat 主要用于输出磁盘IO 和 CPU的统计信息。

用法：iostat [选项] [<时间间隔>] [<次数>]

-c： 显示CPU使用情况
-d： 显示磁盘使用情况
-N： 显示磁盘阵列(LVM) 信息
-n： 显示NFS 使用情况
-k： 以 KB 为单位显示
-m： 以 M 为单位显示
-t： 报告每秒向终端读取和写入的字符数和CPU的信息
-V： 显示版本信息
-x： 显示详细信息
-p：[磁盘] 显示磁盘和分区的情况
CPU属性

1>$ iostat
Linux 3.10.0-957.27.2.el7.x86_64 (tst-center-achitask1) 	2022年01月05日 	_x86_64_	(16 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           2.39    0.00    1.21    0.02    0.00   96.38

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda              11.29         2.79       351.05  207142965 26086788620
%user：CPU处在用户模式下的时间百分比。
%nice：CPU处在带NICE值的用户模式下的时间百分比。
%system：CPU处在系统模式下的时间百分比。
%iowait：CPU等待输入输出完成时间的百分比。
%steal：管理程序维护另一个虚拟处理器时，虚拟CPU的无意识等待时间百分比。
%idle：CPU空闲时间百分比。
如果%iowait的值过高，表示硬盘存在I/O瓶颈，%idle值高，表示CPU较空闲，如果%idle值高但系统响应慢时，有可能是CPU等待分配内存，此时应加大内存容量。%idle值如果持续低于10，那么系统的CPU处理能力相对较低，表明系统中最需要解决的资源是CPU。

磁盘属性

1>$ iostat -d -x -k 1
Linux 3.10.0-957.27.2.el7.x86_64 (tst-center-achitask1) 	2022年01月05日 	_x86_64_	(16 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     9.53    0.21   11.08     2.79   351.04    62.68     0.03    2.89    3.13    2.89   0.24   0.28
rrqm/s 每秒进行 merge 的读操作数目。即 rmerge/s
wrqm/s 每秒进行 merge 的写操作数目。即 wmerge/s
r/s 每秒完成的读 I/O 设备次数。即 rio/s
w/s 每秒完成的写 I/O 设备次数。即 wio/s
rsec/s 每秒读扇区数。即 rsect/s
wsec/s 每秒写扇区数。即 wsect/s
rkB/s 每秒读 K 字节数。是 rsect/s 的一半，因为每扇区大小为 512 字节。
wkB/s 每秒写 K 字节数。是 wsect/s 的一半。
avgrq-sz 平均每次设备 I/O 操作的数据大小 (扇区)。
avgqu-sz 平均 I/O 队列长度。
await 平均每次设备 I/O 操作的等待时间 (毫秒)。
svctm 平均每次设备 I/O 操作的服务时间 (毫秒)。
%util 一秒中有百分之多少的时间用于 I/O 操作，即被 io 消耗的 cpu 百分比。
如果 %util 接近 100%，说明产生的 I/O 请求太多，I/O 系统已经满负荷，该磁盘可能存在瓶颈。如果 svctm 比较接近 await，说明 I/O 几乎没有等待时间；如果 await 远大于 svctm，说明 I/O 队列太长，io 响应太慢，则需要进行必要优化。如果 avgqu-sz 比较大，也表示有当量 io 在等待。

举一个例子，我们在超市排队 checkout 时，怎么决定该去哪个交款台呢? 首当是看排的队人数，5 个 人总比 20 人要快吧? 除了数人头，我们也常常看看前面人购买的东西多少，如果前面有个采购了1星 期食品的大妈，那么可以考虑换个队排了。还有就是收银员的速度了，如果碰上了连钱都点不清楚的新 手，那就有的 等了。另外，时机也很重要，可能 5 分钟前还人满为患的收款台，现在已是人去楼空，这时候交款可 是很爽啊，当然，前提是那过去的 5 分钟里所做的事情比排队要有意义 (不过我还没发现什么事情比 排队还无聊的)。 I/O 系统也和超市排队有很多类似之处: r/s+w/s 类似于交款人的总数 平均队列长度(avgqu-sz)类似于单位时间里平均排队人的个数 平均服务时间(svctm)类似于收银员的收款速度 平均等待时间(await)类似于平均每人的等待时间 平均 I/O 数据(avgrq-sz)类似于平均每人所买的东西多少 I/O 操作率 (%util)类似于收款台前有人排队的时间比例。 我们可以根据这些数据分析出 I/O 请求的模式，以及 I/O 的速度和响应时间。

pidstat

pidstat：是一个常用的进程性能分析工具，用来实时查看进程的 CPU、内存、I/O 以及上下文切换等性能指标。

用法：pidstat [ 选项 ] [ <时间间隔> ] [ <次数> ]

-u：默认的参数，显示各个进程的cpu使用统计
-r：显示各个进程的内存使用统计
-d：显示各个进程的IO使用情况
-p：指定进程号
-w：显示每个进程的上下文切换情况
-t：显示选择任务的线程的统计信息外的额外信息
-T { TASK | CHILD | ALL }
 这个选项指定了pidstat监控的。TASK表示报告独立的task，CHILD关键字表示报告进程下所有线程统计信息。ALL表示报告独立的task和task下面的所有线程。
 注意：task和子线程的全局的统计信息和pidstat选项无关。这些统计信息不会对应到当前的统计间隔，这些统计信息只有在子线程kill或者完成的时候才会被收集。
-V：版本号
-h：在一行上显示了所有活动，这样其他程序可以容易解析。
-I：在SMP环境，表示任务的CPU使用率/内核数量
-l：显示命令名和所有参数
7>$ pidstat -w -p 19993
Linux 3.10.0-1127.18.2.el7.x86_64 (tst-center-achievement1) 	2022年01月05日 	_x86_64_	(8 CPU)

15时03分23秒   UID       PID   cswch/s nvcswch/s  Command
PID:进程id
Cswch/s:每秒主动任务上下文切换数量
Nvcswch/s:每秒被动任务上下文切换数量
Command:命令名
4>$ pidstat -t -p 3366
Linux 3.10.0-957.27.2.el7.x86_64 (tst-center-achitask1) 	2022年01月05日 	_x86_64_	(16 CPU)

15时05分02秒   UID      TGID       TID    %usr %system  %guest    %CPU   CPU  Command
15时05分02秒  1004      3366         -    0.17    0.04    0.00    0.21     7  java
15时05分02秒  1004         -      3366    0.00    0.00    0.00    0.00     7  |__java
15时05分02秒  1004         -      3367    0.00    0.00    0.00    0.00     1  |__java
15时05分02秒  1004         -      3368    0.00    0.00    0.00    0.00     8  |__java
TGID:主线程的表示
TID:线程id
%usr：进程在用户空间占用cpu的百分比
%system：进程在内核空间占用cpu的百分比
%guest：进程在虚拟机占用cpu的百分比
%CPU：进程占用cpu的百分比
CPU：处理进程的cpu编号
Command：当前进程对应的命令
strace

starce 命令，可以看到系统调用方法的踪迹，方便对代码的分析

-c 统计每一系统调用的所执行的时间,次数和出错的次数等. 
-d 输出strace关于标准错误的调试信息. 
-f 跟踪由fork调用所产生的子进程. 
-ff 如果提供-o filename,则所有进程的跟踪结果输出到相应的filename.pid中,pid是各进程的进程号. 
-F 尝试跟踪vfork调用.在-f时,vfork不被跟踪. 
-h 输出简要的帮助信息. 
-i 输出系统调用的入口指针. 
-q 禁止输出关于脱离的消息. 
-r 打印出相对时间关于,,每一个系统调用. 
-t 在输出中的每一行前加上时间信息. 
-tt 在输出中的每一行前加上时间信息,微秒级. 
-ttt 微秒级输出,以秒了表示时间. 
-T 显示每一调用所耗的时间. 
-v 输出所有的系统调用.一些调用关于环境变量,状态,输入输出等调用由于使用频繁,默认不输出. 
-V 输出strace的版本信息. 
-o filename 将strace的输出写入文件filename 
-p pid 跟踪指定的进程pid. 

strace -tt -f -o /tmp/output.log -p {pid}

strace -c -p {pid}
网络

ifconfig

2>$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.16.2.7  netmask 255.255.255.0  broadcast 172.16.2.255
        ether 00:16:3e:0f:ac:95  txqueuelen 1000  (Ethernet)
        RX packets 142846989031  bytes 49750276969587 (45.2 TiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 102849935058  bytes 43306466819952 (39.3 TiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 101463  bytes 4665960 (4.4 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 101463  bytes 4665960 (4.4 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
UP：表示“接口已启用”。BROADCAST ：表示“主机支持广播”。RUNNING：表示“接口在工作中”。MULTICAST：表示“主机支持多播”。MTU:1500（最大传输单元）：1500字节
inet ：网卡的IP地址。netmask ：网络掩码。nbroadcast ：广播地址。
连接类型：Ethernet (以太网) HWaddr (硬件mac地址)txqueuelen (网卡设置的传送队列长度)
RX packets 接收时，正确的数据包数。RX bytes 接收的数据量。RX errors 接收时，产生错误的数据包数。RX dropped 接收时，丢弃的数据包数。RX overruns 接收时，由于速度过快而丢失的数据包数。RX frame 接收时，发生frame错误而丢失的数据包数。
TX packets 发送时，正确的数据包数。TX bytes 发送的数据量。TX errors 发送时，产生错误的数据包数。TX dropped 发送时，丢弃的数据包数。TX overruns 发送时，由于速度过快而丢失的数据包数。TX carrier 发送时，发生carrier错误而丢失的数据包数。collisions 冲突信息包的数目
netstat

netstat统计TCP连接状态

4>$ netstat -n | grep "^tcp" | awk '{print $6}' | sort | uniq -c | sort -n
      1 SYN_SENT
     21 CLOSE_WAIT
    435 TIME_WAIT
   1714 ESTABLISHED
netstat -s 统计TCP半连接和全连接队列溢出

>$ netstat -s | egrep "listen|LISTEN"
    14160060 times the listen queue of a socket overflowed // 全连接溢出数，不断增量
    
    14160060 SYNs to LISTEN sockets dropped //半连接丢弃数，不断增量
ss 命令，来查看 TCP 全连接队列

Recv-Q：当前全连接队列的大小，也就是当前已完成三次握手并等待服务端 accept() 的 TCP 连接；
Send-Q：当前全连接最大队列长度，上面的输出结果说明监听 8088 端口的 TCP 服务，最大全连接长度为 128
全连接队列上限=min(somaxconn, backlog),backlog在tomcat中体现在server.tomcat.accept-count=800
2>$ ss -lnt
State      Recv-Q Send-Q                                                                                       Local Address:Port                                                                                         Peer Address:Port
LISTEN     0      128                                                                                                      *:10050                                                                                                   *:*
LISTEN     0      5                                                                                               172.16.2.7:873                                                                                                     *:*
LISTEN     0      128                                                                                                      *:40022                                                                                                   *:*

>$ cat /proc/sys/net/core/somaxconn
32768 // 阿里云ECS默认为32768


检查端口占用

netstat -apn | grep 80
TCP 的连接状态查看

netstat -napt
route

查看路由表
route -n
arp

查看arp缓存
arp -a
文件查找 find

查找txt和pdf文件: find . ( -name ".txt" -o -name ".pdf" ) -print
正则方式查找.txt和pdf: find . -regex ".*(.txt|.pdf)$"
否定参数 ,查找所有非txt文本: find . ! -name "*.txt" -print
指定搜索深度,打印出当前目录的文件（深度为1）: find . -maxdepth 1 -type f
最近第7天被访问过的所有文件: find . -atime -7 -type f -print
按大小搜索大于512M的文件: find . -type f -size +512M
删除当前目录下所有的swp文件: find . -type f -name "*.swp" -delete
将当前目录下的所有权变更为weber: find . -type f -user root -exec chown weber {} ;
找到的文件全都copy到另一个目录: find . -type f -mtime +10 -name "*.txt" -exec cp {} OLD ;
递归当前目录及子目录删除所有.o文件: find ./ -name "*.o" -exec rm {} ;
查找/server目录下占用磁盘空间最大的top10文件: find /server -type f -print0 | xargs -0 du -h | sort -rh | head -n 10
文本搜索 grep

在文件中搜索一个单词: grep match_pattern file_name
在多个文件中查找: grep "match_pattern" file_1 file_2 file_3 ...
统计文件或者文本中包含匹配字符串的行数: grep -c "text" file_name
搜索多个文件并查找匹配文本在哪些文件中: grep -l "text" file1 file2 file3...
在多级目录中对文本进行递归搜索(.表示当前目录): grep "text" . -r -n
查找关键字上下文n行信息：cat /task/logs/task-thirdparty-order-pull.log|grep -C 200 'registering service'
命名行参数转换 xargs

将单行转化为多行输出(-n：指定每行显示的字段数)：cat single.txt | xargs -n 3
复制文件到多个目录: echo /home/user/1/ /home/user/2/ /home/user/3/ | xargs -n 1 cp /home/user/my_file.txt
排序 sort

查看一下你最常用的10个命令: cat .bash_history | sort | uniq -c | sort -rn | head -n 10
列出当前目录里的文件(由大到小排序): ls -Slh
列出头十个最耗内存的进程: ps aux | sort -nk +4 | tail
去除重复的行:
sort -u your_file > sorted_deduplicated_file
# 保留原来行顺序
cat -n your_file | sort -uk2 | sort -nk1 | cut -f2-
消除重复行 uniq

消除重复行: sort unsort.txt | uniq
统计各行在文件中出现的次数: sort unsort.txt | uniq -c
找出重复行: sort unsort.txt | uniq -d
统计行和字符 wc

统计行数: wc -l file
统计单词数: wc -w file
统计字符数: wc -c file
文本替换 sed

文本分析工具 awk

快速杀死所有的 java 进程：ps aux | grep java | awk '{ print $2 }' | xargs kill -9
内置参数说明：

$0：表示整个当前行
$1：每行的第一个字段
$2：每行的第二个字段
$3…：以此类推
NR：每行的记录号（即行号）
NF：字段数量变量（即列号）
-F：分隔符选项，默认为空格
输出文件 time.txt 的第一行的内容： cat time.txt | awk 'NR==1'
输出文件最后一行的内容: cat time.txt | awk '{print $NF}'
以特定字符分隔字符串: cat time.txt | awk -F “:” '{print $2}'
文件上传下载 scp

从服务器下载文件（scp 你在服务器上的用户名@服务器ip:服务器上的目录 本地路径）
scp -P 9998 lRb612kp_tstadmin@120.55.58.173:/home/tstadmin/jstack8731.log /Users/xiongying/
将本地文件上传服务器(scp 本地路径 你在服务器上的用户名@服务器ip:服务器上的目录)
scp -P 9998 /Users/xiongying/a.txt lRb612kp_tstadmin@120.55.58.173:home/tstadmin
磁盘管理

查看磁盘空间利用大小: df -h
查看当前目录所占空间大小： du -sh
寻找当前目录，哪个文件夹占用空间最大：du -h --max-depth=1
查询/home目录下占用空间较大的文件 du -sh /home/*|sort -nr|head -3
进程管理

查看端口占用的进程状态：lsof -i:3306
查询指定的进程ID(23295)打开的文件：lsof -p 23295
查询正在运行的进程信息：ps -ef
其他

查看历史执行过的命令：history
参考资料

vmstat：一个标准的报告虚拟内存统计工具
90% 的人会遇到性能问题，如何用 1 行代码快速定位？
Linux iostat命令详解
pidstat 命令详解
pidstat-进程性能分析工具
