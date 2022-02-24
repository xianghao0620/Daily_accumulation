
什么是 Arthas
摘录一段官方 Github 上的简介
Arthas 是Alibaba开源的Java诊断工具，深受开发者喜爱。
当你遇到以下类似问题而束手无策时，Arthas 可以帮助你解决：

这个类从哪个 jar 包加载的？为什么会报各种类相关的 Exception？
我改的代码为什么没有执行到？难道是我没 commit？分支搞错了？
遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？
线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！
是否有一个全局视角来查看系统的运行状况？
有什么办法可以监控到JVM的实时运行状态？
Arthas 支持JDK 6+，支持Linux/Mac/Windows，采用命令行交互模式，同时提供丰富的 Tab 自动补全功能，进一步方便进行问题的定位和诊断。

Arthas 基于哪些工具开发而来
greys-anatomy: Arthas代码基于Greys二次开发而来，非常感谢Greys之前所有的工作，以及Greys原作者对Arthas提出的意见和建议！
termd: Arthas的命令行实现基于termd开发，是一款优秀的命令行程序开发框架，感谢termd提供了优秀的框架。
crash: Arthas的文本渲染功能基于crash中的文本渲染功能开发，可以从这里看到源码，感谢crash在这方面所做的优秀工作。
cli: Arthas的命令行界面基于vert.x提供的cli库进行开发，感谢vert.x在这方面做的优秀工作。
compiler: Arthas里的内存编绎器代码来源
Apache Commons Net: Arthas里的Telnet Client代码来源
JavaAgent：运行在 main方法之前的拦截器，它内定的方法名叫 premain ，也就是说先执行 premain 方法然后再执行 main 方法
ASM：一个通用的Java字节码操作和分析框架。它可以用于修改现有的类或直接以二进制形式动态生成类。ASM提供了一些常见的字节码转换和分析算法，可以从它们构建定制的复杂转换和代码分析工具。ASM提供了与其他Java字节码框架类似的功能，但是主要关注性能。因为它被设计和实现得尽可能小和快，所以非常适合在动态系统中使用(当然也可以以静态方式使用，例如在编译器中)
唯一可惜的是，arthas没有jmap -histo功能，此功能可以在线打印所有对象的数量和占用内存大小

启动
启动命令

java -jar arthas-boot.jar
1
不用arthas时一定要正常退出，命令如下：

quit：只是退出当前的连接。Attach到目标进程上的arthas还会继续运行，端口会保持开放，下次连接时执行java -jar arthas-boot.jar可以直接连接上。
exit：和quit命令一样的功能；
stop：完全退出arthas，
如果是非正常退出，会报下面的错误，提示端口占用。原因是上次连接了一个进程，未正常退出。

[ERROR] The telnet port 3658 is used by process 3804 instead of target process 15043, you will connect to an unexpected process.
[ERROR] 1. Try to restart arthas-boot, select process 3804, shutdown it first with running the 'stop' command.
[ERROR] 2. Or try to stop the existing arthas instance: java -jar arthas-client.jar 127.0.0.1 3658 -c "stop"
[ERROR] 3. Or try to use different telnet port, for example: java -jar arthas-boot.jar --telnet-port 9998 --http-port -1
1
2
3
4
注意： arthas依赖jdk的环境变量，也依赖一些jdk自带的工具，比如 jps，如果服务器上只有jre环境而没有jdk环境的话，是没有jps的，所以arthas也会报错；

arthas所有命令
启动arthas后，在命令行输入help，即可查看命令帮助信息、当前arthas版本支持的指令，或者查看具体指令的使用说明。所有的命令如下：

基础命令
命令	说明
cls	清空当前屏幕区域。
session	查看当前会话的信息，显示当前绑定的pid以及会话id。
reset	重置增强类，将被 Arthas 增强过的类全部还原，Arthas 服务端stop时会重置所有增强过的类
version	输出当前目标 Java 进程所加载的 Arthas 版本号
history	打印历史命令，打印出你在使用arthas过程中输入了哪些命令。
keymap	输出arthas的快捷键映射表：
进阶命令
命令	说明
dashboard	查看当前系统的实时数据面板，例如：服务器thread信息、内存memory、GC回收等情况
thread	查看线程的堆栈信息
jvm	打印出jvm的信息，包括参数和变量，以及用的jvm名字、系统等等
sysprop	查看当前JVM的系统属性(System Property)
sysenv	查看当前JVM的环境属性(System Environment Variables)
vmoption	查看，更新VM诊断相关的参数
perfcounter	查看当前JVM的 Perf Counter信息
logger	查看logger信息，更新logger level（级别）
mbean	这个命令可以便捷的查看或监控 Mbean 的属性信息。
getstatic	通过getstatic命令可以方便的查看类的静态属性。使用方法为getstatic class_name field_name,推荐直接使用ognl命令，更加灵活。
ognl	执行ognl表达式，此命令可动态执行代码
sc	查看JVM已加载的类信息，可以查看类在哪个jar包里面
sm	查看已加载类的方法信息
dump	dump 已加载类的 bytecode 到特定目录
heapdump	dump java heap, 类似jmap命令的heap dump功能。
vmtool	vmtool 利用JVMTI接口，实现查询内存对象，强制GC等功能。
jad	反编译指定已加载类的源码
classloader	查看classloader的继承树，urls，类加载信息
mc	Memory Compiler/内存编译器，编译.java文件生成.class。
retransform	加载外部的.class文件，动态重新加载 jvm已加载的类。
redefine	加载外部的.class文件，redefine jvm已加载的类。这个方式只是修改运行时内存，class文件并没有改变，服务重启就失效了，推荐使用 retransform 命令
monitor	方法执行监控
watch	方法执行数据观测，让你能方便的观察到指定方法的调用情况。能观察到的范围为：返回值、抛出异常、入参，通过编写 OGNL 表达式进行对应变量的查看。
trace	方法内部调用路径，并输出方法路径上的每个节点上耗时，trace 命令能主动搜索 class-pattern／method-pattern 对应的方法调用路径，渲染和统计整个调用链路上的所有性能开销和追踪调用链路。
stack	输出当前方法被调用的调用路径
tt	方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测
profiler	profiler 命令支持生成应用热点的火焰图。本质上是通过不断的采样，然后把收集到的采样结果生成火焰图。
cat	打印文件内容，和linux里的cat命令类似。
echo	打印参数，和linux里的echo命令类似
grep	类似传统的grep命令
base64	base64编码转换，和linux里的 base64 命令类似。
tee	类似传统的tee命令, 用于读取标准输入的数据，并将其内容输出成文件。
pwd	返回当前的工作目录，和linux命令类似
auth	验证当前会话
options	系统的配置开关，可开启和关闭
命令介绍
这里主要介绍常用的arthas命令，不会每个命令都介绍一遍，有些命令是linux继承过来的，也没必要介绍。

session 查看当前会话的信息
这个命令非常简单，就只是打印进程的pid和会话id，注意是java的进程id，不是arthas的进程id；

[arthas@12771]$ session
 Name        Value                                                                                                                                  
--------------------------------------------------                                                                                                  
 JAVA_PID    12771                                                                                                                                  
 SESSION_ID  41d19baa-2064-442a-acf1-b8ac11801265 
1
2
3
4
5
history 打印命令历史
启动arthas后，你输入过的命令都会被记录起来，history则会打印出你的输入历史

history 5 ：查看最近执行的5条指令
history -c：清空输入的历史指令
quit 退出当前arthas客户端
只是退出当前 Arthas 客户端，其他 Arthas 客户端不受影响。等同于exit、logout、q三个指令。

stop 关闭 Arthas 服务端
执行stop指令后，所有 Arthas 客户端全部退出。关闭Arthas服务器之前，会重置掉所有做过的增强类。但是用redefine重加载的类内容不会被重置。

dashboard命令 查看当前系统的实时数据面板
查看当前系统的实时数据面板，例如：服务器thread（线程）信息、内存memory、GC回收等情况


数据列说明
ID: Java级别的线程ID，注意这个ID不能跟jstack中的nativeID一一对应。
NAME: 线程名
GROUP: 线程组名
PRIORITY: 线程优先级, 1~10之间的数字，越大表示优先级越高
STATE: 线程的状态
CPU%: 线程的cpu使用率。比如采样间隔1000ms，某个线程的增量cpu时间为100ms，则cpu使用率=100/1000=10%
DELTA_TIME: 上次采样之后线程运行增量CPU时间，数据格式为秒
TIME: 线程运行总CPU时间，数据格式为分:秒
INTERRUPTED: 线程当前的中断位状态
DAEMON: 是否是daemon线程
JVM内部线程
Java 8之后支持获取JVM内部线程CPU时间，这些线程只有名称和CPU时间，没有ID及状态等信息（显示ID为-1）。 通过内部线程可以观测到JVM活动，如GC、JIT编译等占用CPU情况，方便了解JVM整体运行状况。

设置刷新间隔和次数
另外，面板会默认会每5秒刷新一次，并且会一直刷新下去，如果想要指定刷新次数和间隔时间，可以这么写：

dashboard dashboard -i 1000 -n 2
1
-i表示刷新的间隔时间，单位（毫秒），-n 表表示查询的次数，到达指定次数后，自动退出dashboard面板；

thread 查看线程的堆栈信息
当没有参数时，默认显示第一页的线程信息：thread


查看某个线程的堆栈：thread 13，13为线程的id

[arthas@12771]$ thread 13
"ContainerBackgroundProcessor[StandardEngine[Tomcat]]" Id=13 TIMED_WAITING
    at java.lang.Thread.sleep(Native Method)
    at org.apache.catalina.core.ContainerBase$ContainerBackgroundProcessor.run(ContainerBase.java:1368)
    at java.lang.Thread.run(Thread.java:748)
1
2
3
4
5
除此之外，thread还有其他的用法

thread -n 5：打印前5个最忙的线程并打印堆栈
thread -all ：显示所有匹配的线程
thread -n 3 -i 1000 ：列出1000ms内最忙的3个线程栈
thread –state WAITING，查看指定状态的线程,（TIMED_WAITI、WAITING、RUNNABLE等等）
thread -b：找出阻塞其他线程的线程,当出现死锁后，会提示你出现死锁的位置，代码如下
    static Object obj = new Object();
    public static void main(String[] args) throws InterruptedException, IOException {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    synchronized (obj) {
                        System.out.println("我是第一个线程");
                        Thread.sleep(100000);
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"yexindong-one").start();

        Thread.sleep(1000);
        new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (obj) {
                    System.out.println("我是第二个线程");
                }
            }
        },"yexindong-two").start();
    }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26


jvm 查看当前JVM信息
jvm显示jvm的数据汇总,具体内容分为以下几块

runtime： 包括jvm开始时间,启动参数,class_path等
class-loading ：已加载类的数量,总共加载类数量,已卸载类的数量
garbage-collectors： 显示使用的垃圾收集器及垃圾收集次数
memory ：堆内存空间使用情况
thread： 线程总数,守护线程数,死锁数量
thread线程相关信息如下
COUNT: JVM当前活跃的线程数
DAEMON-COUNT: JVM当前活跃的守护线程数
PEAK-COUNT: 从JVM启动开始曾经活着的最大线程数
STARTED-COUNT: 从JVM启动开始总共启动过的线程次数
DEADLOCK-COUNT: JVM当前死锁的线程数
vmoption 查看/更新虚拟机参数
这个命令可以看到我们的java项目在运行时设置了哪些参数，命令没有参数时会打印所有的vm参数

[arthas@12771]$ vmoption
 KEY                                  VALUE                                ORIGIN                               WRITEABLE                           
----------------------------------------------------------------------------------------------------------------------------------------------------
 HeapDumpBeforeFullGC                 false                                DEFAULT                              true                                
 HeapDumpAfterFullGC                  false                                DEFAULT                              true                                
 HeapDumpOnOutOfMemoryError           false                                DEFAULT                              true                                
 HeapDumpPath                                                              DEFAULT                              true                                
 CMSAbortablePrecleanWaitMillis       100                                  DEFAULT                              true                                
 CMSWaitDuration                      2000                                 DEFAULT                              true                                
 CMSTriggerInterval                   -1                                   DEFAULT                              true                                
 PrintGC                              false                                DEFAULT                              true                                
 PrintGCDetails                       false                                DEFAULT                              true                                
 PrintGCDateStamps                    false                                DEFAULT                              true                                
 PrintGCTimeStamps                    false                                DEFAULT                              true                                
 PrintGCID                            false                                DEFAULT                              true                                
 PrintClassHistogramBeforeFullGC      false                                DEFAULT                              true                                
 PrintClassHistogramAfterFullGC       false                                DEFAULT                              true                                
 PrintClassHistogram                  false                                DEFAULT                              true                                
 MinHeapFreeRatio                     40                                   DEFAULT                              true                                
 MaxHeapFreeRatio                     70                                   DEFAULT                              true                                
 PrintConcurrentLocks                 false                                DEFAULT                              true         
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
除此之外，还可查看单个参数和动态设置参数功能，这里可以动态更新参数，不需要重启java进程；

vmoption PrintGCDetails 查看指定的vm参数
vmoption PrintGCDetails true 更新指定的vm参数
ognl 动态执行代码
调用静态方法：ognl '@java.lang.System@out.println("hello yexindong")'，执行后，可以看到控制台打印了hello yexindong；


sc 查看jvm已加载的类信息
简单用法sc com.test.Test，如果jvm已加载这个类，则会打印出该类的全类名，

[arthas@3568]$ sc com.test.Test
com.test.Test
Affect(row-cnt:1) cost in 4 ms.
1
2
3
但是这样的信息太过于简陋了， 加上-d参数可以打印类的详细信息，并且还可以查看你这个类在哪个jar包里面，命令如下

  sc -d com.test.Test
1
打印结果如下

[arthas@3568]$ sc -d *.Test
 class-info        com.test.Test                                                
 code-source       /Users/yexindong/Documents/java/java_project/Test/target/cla 
                   sses/                                                        
 name              com.test.Test                                                
 isInterface       false                                                        
 isAnnotation      false                                                        
 isEnum            false                                                        
 isAnonymousClass  false                                                        
 isArray           false                                                        
 isLocalClass      false                                                        
 isMemberClass     false                                                        
 isPrimitive       false                                                        
 isSynthetic       false                                                        
 simple-name       Test                                                         
 modifier          public                                                       
 annotation                                                                     
 interfaces                                                                     
 super-class       +-java.lang.Object                                           
 class-loader      +-sun.misc.Launcher$AppClassLoader@18b4aac2                  
                     +-sun.misc.Launcher$ExtClassLoader@3df8ac7c                
 classLoaderHash   18b4aac2                                                     

Affect(row-cnt:1) cost in 17 ms.
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
除此之外，还可以使用模糊查询

sc -d *.Test ：查询所有类名为Test的类
sc -d com.*：查询com包下面所有的类，会往所有的子包遍历
sm 查看已加载的类方法信息
查询某个类下所有的方法

[arthas@3568]$ sm com.test.Test
com.test.Test <init>()V
com.test.Test main([Ljava/lang/String;)V
Affect(row-cnt:2) cost in 4 ms.
1
2
3
4
通过结果可以看到，有2个方法，一个是main方法，另一个为构造方法，如果你未使用构造方法，在编译的时候会自动生成构造方法；
除此之外，还可以加上-d打印方法的详细信息 ：sm -d com.test.Test

[arthas@3568]$ sm -d com.test.Test
 declaring-class   com.test.Test                                                
 constructor-name  <init>                                                       
 modifier          public                                                       
 annotation                                                                     
 parameters                                                                     
 exceptions                                                                     
 classLoaderHash   18b4aac2                                                     

 declaring-class  com.test.Test                                                 
 method-name      main                                                          
 modifier         public,static                                                 
 annotation                                                                     
 parameters       java.lang.String[]                                            
 return           void                                                          
 exceptions       java.lang.InterruptedException                                
                  java.io.IOException                                           
 classLoaderHash  18b4aac2                                                      

Affect(row-cnt:2) cost in 6 ms.
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
另外，还可以打印类中的某一个方法： sm -d com.test.Test main

# 这里打印的是main方法信息
[arthas@3568]$ sm -d com.test.Test main
 declaring-class  com.test.Test                                                 
 method-name      main                                                          
 modifier         public,static                                                 
 annotation                                                                     
 parameters       java.lang.String[]                                            
 return           void                                                          
 exceptions       java.lang.InterruptedException                                
                  java.io.IOException                                           
 classLoaderHash  18b4aac2                                                      

Affect(row-cnt:1) cost in 4 ms.
1
2
3
4
5
6
7
8
9
10
11
12
13
dump 将某个类的class字节码文件导出到指定目录
这个功能最大的好处就是可以导出某个指定的类，比如我想要导出Test.class文件到 指定目录。可以这样写

dump -d /Users/yexindong/Downloads com.test.Test
1

然后在指定的目录下就可以看到刚刚导出的class文件

经过反编译后的class文件是这样的

另外，也可以使用-c或者--classLoaderClass来导出指定hashcode 的类加载器加载的类
dump -c 3d4eac69 demo.* 或者 dump --classLoaderClass sun.misc.Launcher$AppClassLoader demo.*

heapdump 导出堆转储文件
这个命令和jdk自带的jmap类似，导出的文件是一个二进制文件，导出命令：heapdump，未指定参数时会导出所有的类，并且会在磁盘的某个地方自动创建一个临时目录，二进制文件就在那个地方；

[arthas@28747]$ heapdump
Dumping heap to /var/folders/yq/mz0mh34s2s58zpj3sf7c9d800000gn/T/heapdump2021-07-19-16-321079309703341992219.hprof ...
Heap dump file created
1
2
3
如果要只导出存活的对象，并且存储到指定的目录，可以这么写

heapdump --live /tmp/dump.hprof
1
--live 表示只导出存活的对象
/tmp/dump.hprof 表示导出到哪个目录，并制定文件名称；
jad 反编译class文件
这里的反编译用起来比较简单，只需要输入全类名即可反编译源码：jad com.test.Test，执行jad命令后还会打印出类加载器和class文件的所在目录

[arthas@3568]$ jad com.test.Test

ClassLoader:                                                                        
+-sun.misc.Launcher$AppClassLoader@18b4aac2                                         
  +-sun.misc.Launcher$ExtClassLoader@3df8ac7c                                       

Location:                                                                           
/Users/yexindong/Documents/java/java_project/Test/target/classes/                   

       /*
        * Decompiled with CFR.
        */
       package com.test;
       
       import java.io.IOException;
       
       public class Test {
       
           public static void main(String[] args) throws InterruptedException, IOException {
               System.out.println("hello would");
           }
       }

Affect(row-cnt:3) cost in 643 ms.
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
除此之外jad还可以反编译类中的某个方法，下面的代码只编译main方法，并且不显示行号，lineNumber默认的值是true，现在我们把它设为false就不显示行号了；

jad com.test.Test main --lineNumber false
1
mc 内存编译器，编译.java文件生成.class。
使用也非常简单，mc后面加上需要编译的java文件即可

mc /tmp/Test.java
1
也可以使用--classLoaderClass参数来指定类加载器，加上-d命令来指定输出目录

mc --classLoaderClass org.springframework.boot.loader.LaunchedURLClassLoader /tmp/UserController.java -d /tmp
1
编译生成.class文件之后，可以结合retransform命令实现热更新代码。

retransform 加载外部的.class文件
以下命令即可加载指定的class文件

retransform /tmp/Test.class
1
monitor 方法执行监控
执行以下命令既可以启动对方法的监控，

monitor -c 1 -n 2 com.test.Test show
1
其中-c 1 表示监控周期，每一秒监控一次，如果不指定，默认是120秒周期
-n 2 表示一共监控2次，监控的方法为Test类中的show()方法，如果不指定，将会一直监控下去
监控结果如下

监控结果说明

监控项	说明
timestamp	时间戳
class	Java类
method	方法（构造方法、普通方法）
total	调用次数
success	成功次数
fail	失败次数
rt	平均RT
fail-rate	失败率
watch 方法执行数据观测
watch命令可以让用户能方便的观察到指定方法的调用情况。能观察到的范围为：返回值、抛出异常、入参，通过编写 OGNL 表达式进行对应变量的查看。
1、以下命令用于观察方法出参和返回值，

watch com.test.Test show "{params,returnObj}" -x 2 -b -e -s -f 
1
观察的方法为Test类下的show()方法，
"{params,returnObj}"是观察表达式，是一个ognl表达式，
-x 2是指定输出结果的属性遍历深度，默认值为1，为1时看不到参数的具体值，只能看到类型；
-b：在方法调用之前观察，用此命令可查看方法的入参
-e：在方法异常之后观察。用此命令可查看方法抛出的异常
-s：在方法返回之后观察，可查看方法的返回值
-f：在方法结束之后(正常返回和异常返回)观察，可查看方法的返回值和异常信息，默认打开-f


trace 方法调用链
trace 命令能主动搜索方法调用路径，，并输出方法路径上的每个节点上耗时，渲染和统计整个调用链路上的所有性能开销和追踪调用链路。

监听Test类下show方法的调用链：trace com.test.Test show

[arthas@4805]$ trace com.test.Test show
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 28 ms, listenerId: 2
`---ts=2021-07-18 21:41:52;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@18b4aac2
    `---[0.884064ms] com.test.Test:show()
        `---[0.135646ms] com.test.Test:showChild() #38
1
2
3
4
5
6
通过结果可以看到，在main线程中show()方法调用了showChild()方法，前面的ms数是调用方法所花费的时间，

调用链属性说明

thread_name ：线程名称
id：内部线程id
is_daemon ： 是否为守护线程
priority：线程优先级
TCCL：类加载器
stack 输出当前方法被调用的调用路径
和trace命令类似，不同的是stack只输出调用路径，且stack可以通过表达式来过滤，就像下面这个命令，只过滤show方法第一个参数值大于320的方法，-n 2表示过滤2次

stack com.test.Test show 'params[0]>320' -n 2
1
执行结果

stack com.test.Test show 'params[0]>320' -n 2
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 17 ms, listenerId: 5
ts=2021-07-18 21:52:01;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@18b4aac2
    @com.test.Test.show()
        at com.test.Test.main(Test.java:27)

....此处省略一次打印信息

Command execution times exceed limit: 2, so command will exit. You can set it with -n option.
1
2
3
4
5
6
7
8
9
10
tt 记录下指定方法每次调用的入参和返回信息
watch 虽然很方便和灵活，但需要提前想清楚观察表达式的拼写，这对排查问题而言要求太高，因为很多时候我们并不清楚问题出自于何方，只能靠蛛丝马迹进行猜测。

这个时候如果能记录下当时方法调用的所有入参和返回值、抛出的异常会对整个问题的思考与判断非常有帮助。

于是乎，TimeTunnel 命令就诞生了。执行方法如下

tt  -t -n 3 com.test.Test show
1
-t 表示这个参数的表明希望记录下类 Test 的 show() 方法的每次执行情况。
-n 3 ：当你执行一个调用量不高的方法时可能你还能有足够的时间用 CTRL+C 中断 tt 命令记录的过程，但如果遇到调用量非常大的方法，瞬间就能将你的 JVM 内存撑爆。此时你可以通过 -n 参数指定你需要记录的次数，当达到记录次数时 Arthas 会主动中断tt命令的记录过程，避免人工操作无法停止的情况。
命令执行结果如下

[arthas@4805]$ tt  -t -n 3 com.test.Test show 
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 16 ms, listenerId: 8
 INDE  TIMESTAMP      COST(  IS-R  IS-E  OBJECT     CLASS                  METHOD               
 X                    ms)    ET    XP                                                           
------------------------------------------------------------------------------------------------
 1045  2021-07-18 21  0.327  true  fals  NULL       Test                   show                 
       :57:15         033          e                                                            
 1046  2021-07-18 21  0.100  true  fals  NULL       Test                   show                 
       :57:17         478          e                                                            
 1047  2021-07-18 21  0.160  true  fals  NULL       Test                   show                 
       :57:19         673          e                                                                                      
1
2
3
4
5
6
7
8
9
10
11
12
结果字段说明
表格字段	字段解释
INDEX	时间片段记录编号，每一个编号代表着一次调用，后续tt还有很多命令都是基于此编号指定记录操作，非常重要。
TIMESTAMP	方法执行的本机时间，记录了这个时间片段所发生的本机时间
COST(ms)	方法执行的耗时
IS-RET	方法是否以正常返回的形式结束
IS-EXP	方法是否以抛异常的形式结束
OBJECT	执行对象的hashCode()，注意，曾经有人误认为是对象在JVM中的内存地址，但很遗憾他不是。但他能帮助你简单的标记当前执行方法的类实体
CLASS	执行的类名
METHOD	执行的方法名
profiler 生成火焰图
启动profiler

 $ profiler start
 Started [cpu] profiling
1
2
获取已采集的sample的数量

$ profiler getSamples
23
1
2
查看profiler状态，可以查看当前profiler在采样哪种event和采样时间

$ profiler status
[cpu] profiling is running for 4 seconds
1
2
停止profiler—生成svg格式结果

$ profiler stop
profiler output file: /tmp/demo/arthas-output/20191125-135546.svg
OK
1
2
3
默认情况下，生成的结果保存到应用的工作目录下的arthas-output目录。可以通过 --file参数来指定输出结果路径。比如：

$ profiler stop --file /tmp/output.svg
profiler output file: /tmp/output.svg
OK
1
2
3
生成html格式结果
默认情况下，结果文件是svg格式，如果想生成html格式，可以用–format参数指定：

$ profiler stop --format html
profiler output file: /tmp/test/arthas-output/20191125-143329.html
OK
1
2
3
或者在–file参数里用文件名指名格式。比如–file /tmp/result.html 。
通过浏览器查看arthas-output下面的profiler结果
默认情况下，arthas使用3658端口，则可以打开： http://localhost:3658/arthas-output/ 查看到arthas-output目录下面的profiler结果：

点击可以查看具体的结果：


options 全局配置开关
查看所有的options

[arthas@4805]$ options
 LEVEL  TYPE   NAME         VALUE  SUMMARY              DESCRIPTION                             
------------------------------------------------------------------------------------------------
 0      boole  unsafe       false  Option to support s  This option enables to proxy functional 
        an                         ystem-level class    ity of JVM classes. Due to serious secu 
                                                        rity risk a JVM crash is possibly be in 
                                                        troduced. Do not activate it unless you 
                                                         are able to manage.                    
 1      boole  dump         false  Option to dump the   This option enables the enhanced classe 
        an                         enhanced classes     s to be dumped to external file for fur 
                                                        ther de-compilation and analysis.       
 1      boole  batch-re-tr  true   Option to support b  This options enables to reTransform cla 
        an     ansform             atch reTransform Cl  sses with batch mode.                   
                                   ass                                                          
 2      boole  json-format  false  Option to support J  This option enables to format object ou 
        an                         SON format of objec  tput with JSON when -x option selected. 
                                   t output                                                     
 1      boole  disable-sub  false  Option to control i  This option disable to include sub clas 
        an     -class              nclude sub class wh  s when matching class.                  
                                   en class matching                                            
 1      boole  support-def  true   Option to control i  This option disable to include default  
        an     ault-method         nclude default meth  method in interface when matching class 
                                   od in interface whe  .                                       
                                   n class matching                                             
 1      boole  save-result  false  Option to print com  This option enables to save each comman 
        an                         mand's result to lo  d's result to log file, which path is $ 
                                   g file               {user.home}/logs/arthas-cache/result.lo 
                                                        g.                                      
 2      Strin  job-timeout  1d     Option to job timeo  This option setting job timeout,The uni 
        g                          ut                   t can be d, h, m, s for day, hour, minu 
                                                        te, second. 1d is one day in default    
 1      boole  print-paren  true   Option to print all  This option enables print files in pare 
        an     t-fields             fileds in parent c  nt class, default value true.           
                                   lass                                                         
 1      boole  verbose      false  Option to print ver  This option enables print verbose infor 
        an                         bose information     mation, default value false.            
[arthas@4805]$ 
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
结果字段说明

名称	默认值	描述
unsafe	false	是否支持对系统级别的类进行增强，打开该开关可能导致把JVM搞挂，请慎重选择！
dump	false	是否支持被增强了的类dump到外部文件中，如果打开开关，class文件会被dump到/${application working dir}/arthas-class-dump/目录下，具体位置详见控制台输出
batch-re-transform	true	是否支持批量对匹配到的类执行retransform操作
json-format	false	是否支持json化的输出
disable-sub-class	false	是否禁用子类匹配，默认在匹配目标类的时候会默认匹配到其子类，如果想精确匹配，可以关闭此开关
support-default-method	true	是否支持匹配到default method，默认会查找interface，匹配里面的default method。参考 #1105
save-result	false	是否打开执行结果存日志功能，打开之后所有命令的运行结果都将保存到~/logs/arthas-cache/result.log中
job-timeout	1d	异步后台任务的默认超时时间，超过这个时间，任务自动停止；比如设置 1d, 2h, 3m, 25s，分别代表天、小时、分、秒
print-parent-fields	true	是否打印在parent class里的filed
获取option的值
[arthas@4805]$ options json-format
 LEVEL  TYPE   NAME         VALUE  SUMMARY              DESCRIPTION                             
------------------------------------------------------------------------------------------------
 2      boole  json-format  false  Option to support J  This option enables to format object ou 
        an                         SON format of objec  tput with JSON when -x option selected. 
                                   t output                                                     
1
2
3
4
5
6
设置指定的option
[arthas@4805]$ options save-result true
 NAME         BEFORE-VALUE  AFTER-VALUE                                                         
----------------------------------------                                                        
 save-result  false         true    
1
2
3
4
完
arthas目前来说用的人还是非常多的，据说阿里内部的人都在使用这个工具来查找错误，目前来说大部分的公司都还在使用 jinfo、jmap、jstack、jhat等jdk自带的工具来找错，工具没有好坏，看你如何使用，只要能达到目的就可以！arthas这个工具的出现， 帮助我们查找故障又多了一种方法；
