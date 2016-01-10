# JVM性能调优监控工具jps、jstack、jmap、jhat、jstat、hprof使用详解
## JPS(Java Virtual Machine Process Status Tool)  
 jps主要用来输出JVM中运行的进程状态信息。语法格式如下：

> jps [options] [hostid]


如果不指定hostid就默认为当前主机或服务器。
命令行参数选项说明如下：

```powershell
-q 不输出类名、Jar名和传入main方法的参数
-m 输出传入main方法的参数
-l 输出main类或Jar的全限名
-v 输出传入JVM的参数
```
例如：
```
13438 Bootstrap
29080 Bootstrap
14526 Bootstrap
27367 Jps
```
## jstack

jstack主要用来查看某个Java进程内的线程堆栈信息。语法格式如下：
```powershell
jstack [option] pid
jstack [option] executable core
jstack [option] [server-id@]remote-hostname-or-ip
```
    命令行参数选项说明如下：
```powershell
-l long listings，会打印出额外的锁信息，在发生死锁时可以用jstack -l pid来观察锁持有情况
-m mixed mode，不仅会输出Java堆栈信息，还会输出C/C++堆栈信息（比如Native方法）
```
jstack可以定位到线程堆栈，根据堆栈信息我们可以定位到具体代码，所以它在JVM性能调优中使用得非常多。
jstack可以检测出死锁等问题。

##  jmap(Memory Map)和jhat(Java Heap Analysis Tool)
jmap用来查看堆内存使用状况，一般结合jhat使用。
jmap语法格式如下：
> jmap [option] pid
> jmap [option] executable core
> jmap [option] [server-id@]remote-hostname-or-ip

如果运行在64位JVM上，可能需要指定**-J-d64**命令选项参数。
> jmap -permstat pid

打印进程的类加载器和类加载器加载的持久代对象信息，输出：类加载器名称、对象是否存活（不可靠）、对象地址、父类加载器、已加载的类大小等信息，如下图：
``` java
Attaching to process ID 13438, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 20.45-b01
12462 intern Strings occupying 1217888 bytes.
finding class loader instances ..Finding object size using Printezis bits and skipping over...
Finding object size using Printezis bits and skipping over...
Finding object size using Printezis bits and skipping over...
Finding object size using Printezis bits and skipping over...
done.
computing per loader stat ..done.
please wait.. computing liveness.....liveness analysis may be inaccurate ...
class_loader    classes bytes   parent_loader   alive?  type

<bootstrap>     1940    11755032          null          live    <internal>
0x000000070174e1d8      1       3104    0x000000070000a1f0      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x0000000701752500      1       1944    0x000000070000a1f0      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000b170      1       3088      null          dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000abf0      1       3120    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x0000000701752858      1       3104    0x000000070000a1f0      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000a9d0      1       3088      null          dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000b1e0      1       3088    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000a2f8      825     6398120 0x000000070000a1a8      live    org/apache/catalina/loader/StandardClassLoader@0x00000006f836cae0
0x000000070000ae28      1       3088    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000afb0      1       3128      null          dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x00000007017e81a8      1       3088    0x000000070000a1f0      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000ad20      1       3088    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000a1f0      1850    11229000        0x000000070000a2f8      live    org/apache/catalina/loader/WebappClassLoader@0x00000006f8d8b1c0
0x000000070000acb0      1       3128    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000a960      1       3128    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000b0c8      1       3120    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000ab58      1       3120    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000adb8      1       3120    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000a520      1       3112    0x000000070000a1f0      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x0000000700278b48      0       0       0x000000070000a1a8      dead    java/util/ResourceBundle$RBClassLoader@0x00000006f880d398
0x000000070000a788      1       1944      null          dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000b058      1       1944      null          dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x0000000701752f08      1       3104    0x000000070000a1f0      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070178a1a8      1       3088    0x000000070000a1f0      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000b138      1       3088    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000afe8      1       3120    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x0000000701752bb0      1       3104    0x000000070000a1f0      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000ac50      1       3144    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000a900      1       1944      null          dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000b020      1       3088      null          dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000ae60      1       3120    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070178a500      1       3088      null          dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070174e7e0      1       1944    0x000000070000a1f0      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000b218      1       3120    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000a998      1       3104      null          dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000b1a8      1       1944      null          dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000ad58      1       3120    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000abb8      1       3120    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070178a840      1       3088    0x000000070000a1f0      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x00000007017e8858      1       3088    0x000000070000a1f0      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070174e468      1       3104    0x000000070000a1f0      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000a5f8      19      123344    null          live    sun/misc/Launcher$ExtClassLoader@0x00000006f81d2878
0x00000007017521a8      1       1944    0x000000070000a1f0      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070176a190      1       3088      null          dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000b090      1       3120    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x00000007017e8500      1       3088    0x000000070000a1f0      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000ace8      1       3120    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000aa80      1       3120    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000b100      1       3120    0x000000070000a2f8      dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830
0x000000070000a1a8      100     968912  0x000000070000a5f8      live    sun/misc/Launcher$AppClassLoader@0x00000006f8242060
0x000000070000adf0      1       1944      null          dead    sun/reflect/DelegatingClassLoader@0x00000006f8067830

total = 52      4780    30608016            N/A         alive=5, dead=47            N/A    
```
使用**jmap -heap pid**查看进程堆内存使用情况，包括使用的GC算法、堆配置参数和各代中堆内存使用情况。比如下面的例子：
```java
Attaching to process ID 13438, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 20.45-b01

using thread-local object allocation.
Parallel GC with 8 thread(s)

Heap Configuration:
   MinHeapFreeRatio = 40
   MaxHeapFreeRatio = 70
   MaxHeapSize      = 4294967296 (4096.0MB)
   NewSize          = 1310720 (1.25MB)
   MaxNewSize       = 17592186044415 MB
   OldSize          = 5439488 (5.1875MB)
   NewRatio         = 2
   SurvivorRatio    = 8
   PermSize         = 134217728 (128.0MB)
   MaxPermSize      = 134217728 (128.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 1413087232 (1347.625MB)
   used     = 426731096 (406.96248626708984MB)
   free     = 986356136 (940.6625137329102MB)
   30.19849633741507% used
From Space:
   capacity = 9502720 (9.0625MB)
   used     = 2949120 (2.8125MB)
   free     = 6553600 (6.25MB)
   31.03448275862069% used
To Space:
   capacity = 9043968 (8.625MB)
   used     = 0 (0.0MB)
   free     = 9043968 (8.625MB)
   0.0% used
PS Old Generation
   capacity = 2863333376 (2730.6875MB)
   used     = 26222776 (25.00798797607422MB)
   free     = 2837110600 (2705.679512023926MB)
   0.9158128850728697% used
PS Perm Generation
   capacity = 134217728 (128.0MB)
   used     = 31856744 (30.380958557128906MB)
   free     = 102360984 (97.6190414428711MB)
   23.735123872756958% used
```
使用**jmap -histo[:live] pid**查看堆内存中的对象数目、大小统计直方图，如果带上live则只统计活对象，如下：
```java
[root@yq01-cm-m32-201502nova224.yq01.baidu.com work]# jmap -histo:live -J-d64  13438     

 num     #instances         #bytes  class name
----------------------------------------------
   1:         12041       11227400  [B
   2:         58720        7342112  [C
   3:         46016        6711032  <constMethodKlass>
   4:         46016        6268944  <methodKlass>
   5:          4062        4573968  <constantPoolKlass>
   6:         73875        4063264  <symbolKlass>
   7:          4062        3076840  <instanceKlassKlass>
   8:          3488        2750784  <constantPoolCacheKlass>
   9:         66740        2135680  java.lang.String
  10:         22519         900760  java.util.LinkedHashMap$Entry
  11:          1499         783440  <methodDataKlass>
  12:          2091         623024  [Ljava.util.HashMap$Entry;
  13:         20225         485400  org.apache.tomcat.util.bcel.classfile.ConstantUtf8
  14:          4479         465816  java.lang.Class
  15:         13177         421664  java.util.HashMap$Entry
  16:          6235         399040  java.net.URL
  17:          6057         370768  [S
  18:          4069         358072  java.lang.reflect.Method
  19:          6424         345984  [[I
  20:          7295         291800  java.lang.ref.Finalizer
  21:          4517         286440  [I
  22:          7190         230080  java.io.FileInputStream
  23:           388         226592  <objArrayKlassKlass>
  24:          4519         216912  org.apache.catalina.loader.ResourceEntry
  25:          7213         173112  java.io.FileDescriptor
  26:          5732         137568  com.baidu.fountain.event.TableMapEvent$ColumnInfo
```
class name是对象类型，说明如下：
```powershell
B  byte
C  char
D  double
F  float
I  int
J  long
Z  boolean
[  数组，如[I表示int[]
[L+类名 其他对象
```
还有一个很常用的情况是：用jmap把进程内存使用情况dump到文件中，再用jhat分析查看。jmap进行dump命令格式如下：

> jmap -dump:format=b,file=dumpFileName pid

 我一样地对上面进程ID为21711进行Dump：

> root@ubuntu:/# jmap -dump:format=b,file=/tmp/dump.dat 21711     
> Dumping heap to /tmp/dump.dat ...
> Heap dump file created

dump出来的文件可以用MAT、VisualVM等工具查看，这里用jhat查看：
> root@ubuntu:/# jhat -port 9998 /tmp/dump.dat
> Reading from /tmp/dump.dat...
> Dump file created Tue Jan 28 17:46:14 CST 2014
> Snapshot read, resolving...
> Resolving 132207 objects...
> Chasing references, expect 26 dots..........................
> Eliminating duplicate references..........................
> Snapshot resolved.
>Started HTTP server on port 9998
> Server is ready.

注意如果Dump文件太大，可能需要加上**-J-Xmx512m**这种参数指定最大堆内存，即jhat -J-Xmx512m -port 9998 /tmp/dump.dat。然后就可以在浏览器中输入主机地址:9998查看了：
## jstat (JVM统计监测工具)

    语法格式如下：
> jstat [ generalOption | outputOptions vmid [interval[s|ms] [count]] ]

vmid是Java虚拟机ID，在Linux/Unix系统上一般就是进程ID。interval是采样时间间隔。count是采样数目。比如下面输出的是GC信息，采样时间间隔为250ms，采样数为4：
``` java
[root@yq01-cm-m32-201502nova224.yq01.baidu.com work]# jstat -gc 13438 250  4
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       PC     PU    YGC     YGCT    FGC    FGCT     GCT   
8704.0 8832.0  0.0    0.0   1379968.0 172906.9 2796224.0   24650.2   131072.0 30996.6     43    0.720   1      0.125    0.845
8704.0 8832.0  0.0    0.0   1379968.0 172906.9 2796224.0   24650.2   131072.0 30996.6     43    0.720   1      0.125    0.845
8704.0 8832.0  0.0    0.0   1379968.0 172906.9 2796224.0   24650.2   131072.0 30996.6     43    0.720   1      0.125    0.845
8704.0 8832.0  0.0    0.0   1379968.0 172906.9 2796224.0   24650.2   131072.0 30996.6     43    0.720   1      0.125    0.845
```
现在来解释各列含义：

```
S0C、S1C、S0U、S1U：Survivor 0/1区容量（Capacity）和使用量（Used）
EC、EU：Eden区容量和使用量
OC、OU：年老代容量和使用量
PC、PU：永久代容量和使用量
YGC、YGT：年轻代GC次数和GC耗时
FGC、FGCT：Full GC次数和Full GC耗时
GCT：GC总耗时
```
## hprof(Heap/CPU Profiling Tool)

hprof能够展现CPU使用率，统计堆内存使用情况

语法格式如下：
> java -agentlib:hprof[=options] ToBeProfiledClass
> java -Xrunprof[:options] ToBeProfiledClass
> javac -J-agentlib:hprof[=options] ToBeProfiledClass

来几个官方指南上的实例。
CPU Usage Sampling Profiling(cpu=samples)的例子：
> java -agentlib:hprof=cpu=samples,interval=20,depth=3 Hello

上面每隔20毫秒采样CPU消耗信息，堆栈深度为3，生成的profile文件名称是java.hprof.txt，在当前目录。 
CPU Usage Times Profiling(cpu=times)的例子，它相对于CPU Usage Sampling Profile能够获得更加细粒度的CPU消耗信息，能够细到每个方法调用的开始和结束，它的实现使用了字节码注入技术(BCI):
> javac -J-agentlib:hprof=cpu=times Hello.java

Heap Allocation Profiling(heap=sites)的例子：
>  javac -J-agentlib:hprof=heap=sites Hello.java

Heap Dump(heap=dump)的例子，它比上面的Heap Allocation Profiling能生成更详细的Heap Dump信息：
> javac -J-agentlib:hprof=heap=dump Hello.java

虽然在JVM启动参数中加入-Xrunprof:heap=sites参数可以生成CPU/Heap Profile文件，但对JVM性能影响非常大，不建议在线上服务器环境使用。