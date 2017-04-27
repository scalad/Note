## JVM监控和诊断概述jps、jstak、jstack、jinfo、jmap ##
程序运行中经常会遇到各种问题，定位问题时通常需要综合各种信息，如系统日志、堆dump文件、线程dump文件、GC日志等。通过虚拟机监控和诊断工具可以帮忙我们快速获取、分析需要的数据，进而提高问题解决速度。可以使用图形化工具如:jconsole,jvisualvm,或者命令进行监控和诊断命令。 本文将介绍虚拟机常用监控和问题诊断命令工具的使用方法，主要包含以下工具: 

	jps		显示系统中所有Hotspot虚拟机进程
	jstat	收集Hotspot虚拟机各方面运行数据
	jstack	显示虚拟机的线程栈信息
	jinfo	显示虚拟机的配置信息
	jmap	用于生成虚拟机的内存快照信息

### 命令介绍  ###

#### 1).jps 命令 ####
JVM Process Status Tool，该命令用于列出正在运行的虚拟机进程，显示main类的名称和虚拟机进程id。该命令受当前用户的访问权限影响，比如linux下非root用户只列出当前用户启动的虚拟机进程。 

命令格式: 

	jps  [options]  [hostid]

	常用参数: 
	 -l	输出主类全名
	 -v	输出虚拟机进程启动的jvm参数
	 -m	输出启动时传递给main函数的参数

#### 2).jstack 命令 ####
Stack Trace for Java，用于生成虚拟机当前的线程快照信息，包含每一条线程的堆栈信息。该命令通常用于定位线程停顿原因，当出现线程停顿时，可通过stack查看每个线程的堆栈信息，进而分析停顿原因。 

命令格式: 
	jstack [ option ] pid
	
	常用参数: 
	 -l	除堆栈外，显示锁的附加信息
	 -F	当请求不被响应时，强制输出线程堆栈
	 -m	混合模式，打印java和本地C++调用的堆栈信息

#### 3) jstat命令  ####
JVM Statistics Monitoring Tool，用于监控各种运行状态信息的命令。在只有文本控制台的环境中(如企业中的生产环境)，该工具非常有用。 可以用来显示系统中类装载、垃圾回收、运行期编译状况等运行数据。

命令格式: 
	jstat [ generalOption | outputOptions vmid [interval[s|ms] [count]] ] 
	vmid表示虚拟机唯一标识符，如果是本地虚拟机进程，与LVMID一致，通常为本地虚拟机进程号。 
	interval表示查询间隔时间，count表示查询次数。如果省略interval和count参数，表示查询一次。

	常用参数: 
	class		类装载相关信息.
	compiler	JIT编译器编译过的方法、耗时等.
	gc			java堆信息和垃圾回收状况.
	gccapacity	关注java堆各个区的最大和最小空间.
	gccause		类似gcutil，额外输出导致上一次gc的原因.
	gcnew		新生代gc状况.
	gcnewcapacity	关注新生代gc使用的最大和最小空间.
	gcold		老年代gc状况.
	gcoldcapacity	关注老年代gc使用的最大和最小空间.
	gcpermcapacity	关注持久代gc使用的最大和最小空间.
	gcutil		关注已使用空间占总空间比例.
	printcompilation	输出已经被JIT编译的方法.

#### 4) jinfo命令 ####
Configuration Info for Java，用于查看和修改虚拟机的各项参数信息。 

命令格式： 

	jinfo [ option ] pid 
	常用参数: 
	 -flag name	打印虚拟机该参数对应的值.
	 -flag [+\-]name	使该参数生效或失效.
	 -flag name=value	修改相应参数的值.
	 -flags	打印传给jvm的参数值.
	 -sysprops	打印System.getProperties()信息.

#### 5) jmap命令 ####
Memory Map for Java，可以产生堆dump文件，查询堆和持久代的详细信息等。 

命令格式: 

	jmap [ option ] pid 
	常用参数:
	 -dump	生成堆dump文件，格式为: -dump:[live,]format=b,file=<filename>
	 -heap	显示java堆的详细信息，包括垃圾回收期、堆配置和分代信息等
	 -histo	显示堆中对象的统计信息，包括类名称，对应的实例数量和总容量
	 -permstat	统计持久代中各ClassLoader的统计信息。

####  6) jhatk命令 ####
Jvm Heap Analysis Tool， 与jmap一起使用

示例： 

	jhat dump.tmp  
	Reading from dump.tmp...  
	Dump file created Tue Jun 28 13:55:09 CST 2016    

