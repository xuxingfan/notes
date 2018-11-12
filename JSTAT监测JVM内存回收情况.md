# JSTAT监测JVM内存回收情况

### 问题分析
作为开发人员来说，保证程序的稳定运行，是最基本的，也是最重要的。JVM的稳定运行是JAVA程序稳定运行的前提，作为JAVA开发人来说，学会分析JVM内存显得尤为重要。


### jstat-JDK自带的轻量级小工具
jstat(JVM Statistics Monitoring Tool)是用于监控虚拟机各种运行状态信息的命令工具。它可以显示本地或者是远程中的类装载、内存、垃圾回收、JIT编译等运行数据。在没有GUI图像桌面的服务器上，他是JVM虚拟机性能监测的首选工具。

jstat命令命令格式：
jstat [Options] vmid [interval] [count]
 
###### 命令参数说明:

Options，一般使用 -gcutil 或  -gc 查看gc 情况  
pid，当前运行的 java进程号  
interval，间隔时间，单位为秒或者毫秒  
count，打印次数，如果缺省则打印无数次  

###### Options 参数如下:

*-gc：统计 jdk gc时 heap信息，以使用空间字节数表示  
-gcutil：统计 gc时 heap情况，以使用空间的百分比表示  
-class：统计 class loader行为信息  
-compile：统计编译行为信息  
-gccapacity：统计不同   generations（新生代，老年代，持久代）的 heap容量情况   
-gccause：统计引起 gc的事件  
-gcnew：统计 gc时，新生代的情况  
-gcnewcapacity：统计 gc时，新生代 heap容量  
-gcold：统计 gc时，老年代的情况  
-gcoldcapacity：统计 gc时，老年代 heap容量  
-gcpermcapacity：统计 gc时， permanent区 heap容量*

实践：本次我们学习如何观察内存回收情况，避免内存溢出情况发生

内存正常回收的依据：
1. 内存回收频率是否正常，主要观察YGC和FGC次数
2. 每次YGC之后EU的释放情况，是否回到一个低值
查看内存回收代码如下
```
#跟踪中转数据处理Java进程GC情况
#@author 徐兴繁
pid=`ps -ef|grep TransferBasicData-Passenger.jar|grep -v grep|awk '{print $2}'`
/opt/app/java/jdk/jdk1.8.0_144/bin/jstat -gc $pid 10000
```
监测截图如下：

![image](https://note.youdao.com/yws/api/personal/file/0422E1A468434D0093B94BBFA21CAB97?method=download&shareKey=82936d799e8c719c6df65629f46fb162)

从上图中可以看出从JVM启动到现在，共执行1218次YGC,2次FGC(备注：很多时候内存溢出都是频繁发生FGC导致的),截图中红色标注位置为执行YGC，可以看到每次YGC之后，EU都释放90%以上的空间。所以可以基本确定程序内存回收没有什么问题，并且从YGC和FGC的次数以及OU大小可以得到结论是可以把年轻代 -Xmn 的大小适当调大。


列头 |  说明
---|---
S0C	| 年轻代中第一个survivor区的容量 (字节)
S1C	| 年轻代中第二个survivor区的容量 (字节)
S0U	| 年轻代中第一个survivor区目前已使用空间 (字节)
S1U	| 年轻代中第二个survivor区目前已使用空间 (字节)
EC	| 年轻代中Eden的容量 (字节)
EU	| 年轻代中Eden目前已使用空间 (字节)
OC	| Old代的容量 (字节)
OU	| Old代目前已使用空间 (字节)
PC	| Perm(持久代)的容量 (字节)
PU	| Perm(持久代)目前已使用空间 (字节)
YGC	| 从应用程序启动到采样时年轻代中gc次数
YGCT |	从应用程序启动到采样时年轻代中gc所用时间(s)
FGC	| 从应用程序启动到采样时old代(全gc)gc次数
FGCT | 从应用程序启动到采样时old代(全gc)gc所用时间(s)
GCT	| 从应用程序启动到采样时gc用的总时间(s)
NGCMN | 年轻代(young)中初始化(最小)的大小 (字节)
NGCMX | 年轻代(young)的最大容量 (字节)
NGC	| 年轻代(young)中当前的容量 (字节)
OGCMN | old代中初始化(最小)的大小 (字节)
OGCMX | old代的最大容量 (字节)
OGC	| old代当前新生成的容量 (字节)
PGCMN | perm代中初始化(最小)的大小 (字节)
PGCMX 	| perm代的最大容量 (字节)
PGC	| perm代当前新生成的容量 (字节)
S0 | 年轻代中第一个survivor区已使用的占当前容量百分比
S1 | 年轻代中第二个survivor区已使用的占当前容量百分比
E | 年轻代中Eden已使用的占当前容量百分比
O | old代已使用的占当前容量百分比
P | perm代已使用的占当前容量百分比
S0CMX | 年轻代中第一个survivor区的最大容量 (字节)
S1CMX | 年轻代中第二个survivor区的最大容量 (字节)
ECMX  | 年轻代中Eden的最大容量 (字节)
DSS	| 当前需要survivor区的容量 (字节)（Eden区已满）
TT	| 持有次数限制
MTT	| 最大持有次数限制

另外再介绍两个JDK自带的分析工具
1. jmap-分析dump，分析各代内存分配情况
2. jstack-查看虚拟机中各线程的运行情况
