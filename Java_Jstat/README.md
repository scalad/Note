## JVM自带命令jstat命令 ##
Jstat是JDK自带的一个轻量级小工具。全称“Java Virtual Machine statistics monitoring tool”，它位于Java的bin目录下，主要利用JVM内建的指令对Java应用程序的资源和性能进行实时的命令行的监控，包括了对Heap size和垃圾回收状况的监控。可见，Jstat是轻量级的、专门针对JVM的工具，非常适用。

jstat工具特别强大，有众多的可选项，详细查看堆内各个部分的使用量，以及加载类的数量。使用时，需加上查看进程的进程id，和所选参数。

参考格式如下：jstat -options 

可以列出当前JVM版本支持的选项，常见的有

	class (类加载器) 
	compiler (JIT) 
	gc (GC堆状态) 
	gccapacity (各区大小) 
	gccause (最近一次GC统计和原因) 
	gcnew (新区统计)
	gcnewcapacity (新区大小)
	gcold (老区统计)
	gcoldcapacity (老区大小)
	gcpermcapacity (永久区大小)
	gcutil (GC统计汇总)
	printcompilation (HotSpot编译统计)

### 1、jstat –class<pid> : 显示加载class的数量，及所占空间等信息。 ###


	显示列名		具体描述
	Loaded		装载的类的数量
	Bytes   	装载类所占用的字节数
	Unloaded 	卸载类的数量
	Bytes		卸载类的字节数
	Time 		装载和卸载类所花费的时间

### 2、jstat -compiler <pid>显示VM实时编译的数量等信息。 ###


	显示列名			具体描述
	Compiled		编译任务执行数量
	Failed			编译任务执行失败数量
	Invalid			编译任务执行失效数量
	Time			编译任务消耗时间
	FailedType		最后一个编译失败任务的类型
	FailedMethod	最后一个编译失败任务所在的类及方法

### 3、jstat -gc <pid>: 可以显示gc的信息，查看gc的次数，及时间。 ###

	显示列名			具体描述
	S0C				年轻代中第一个survivor（幸存区）的容量 (字节)
	S1C				年轻代中第二个survivor（幸存区）的容量 (字节)
	S0U   			年轻代中第一个survivor（幸存区）目前已使用空间 (字节)
	S1U     		年轻代中第二个survivor（幸存区）目前已使用空间 (字节)
	EC      		年轻代中Eden（伊甸园）的容量 (字节)
	EU       		年轻代中Eden（伊甸园）目前已使用空间 (字节)
	OC        		Old代的容量 (字节)
	OU      		Old代目前已使用空间 (字节)
	PC    			Perm(持久代)的容量 (字节)
	PU				Perm(持久代)目前已使用空间 (字节)
	YGC    			从应用程序启动到采样时年轻代中gc次数
	YGCT   			从应用程序启动到采样时年轻代中gc所用时间(s)
	FGC   			从应用程序启动到采样时old代(全gc)gc次数
	FGCT    		从应用程序启动到采样时old代(全gc)gc所用时间(s)
	GCT				从应用程序启动到采样时gc用的总时间(s)

### 4、jstat -gccapacity <pid>:可以显示，VM内存中三代（young,old,perm）对象的使用和占用大小 ###
	
	显示列名				具体描述
	NGCMN				年轻代(young)中初始化(最小)的大小(字节)
	NGCMX    			年轻代(young)的最大容量 (字节)
	NGC    				年轻代(young)中当前的容量 (字节)
	S0C  				年轻代中第一个survivor（幸存区）的容量 (字节)
	S1C      			年轻代中第二个survivor（幸存区）的容量 (字节)
	EC     				年轻代中Eden（伊甸园）的容量 (字节)
	OGCMN     			old代中初始化(最小)的大小 (字节)
	OGCMX      			old代的最大容量(字节)
	OGC					old代当前新生成的容量 (字节)
	OC     				Old代的容量 (字节)
	PGCMN   			perm代中初始化(最小)的大小 (字节)
	PGCMX    			perm代的最大容量 (字节)  
	PGC      			perm代当前新生成的容量 (字节)
	PC    				Perm(持久代)的容量 (字节)
	YGC   				从应用程序启动到采样时年轻代中gc次数
	FGC					从应用程序启动到采样时old代(全gc)gc次数

### 5、jstat -gcutil <pid>:统计gc信息 ###

![](https://github.com/scalad/Note/blob/master/Java_Jstat/image/5.png)

### 6、jstat -gcnew <pid>:年轻代对象的信息。 ###

![](https://github.com/scalad/Note/blob/master/Java_Jstat/image/6.png)

### 7、jstat -gcnewcapacity<pid>: 年轻代对象的信息及其占用量。 ###

![](https://github.com/scalad/Note/blob/master/Java_Jstat/image/7.png)

### 8、jstat -gcold <pid>：old代对象的信息。 ###

![](https://github.com/scalad/Note/blob/master/Java_Jstat/image/8.png)

### 9、stat -gcoldcapacity <pid>: old代对象的信息及其占用量。 ###

![](https://github.com/scalad/Note/blob/master/Java_Jstat/image/9.png)

### 10、jstat -gcpermcapacity<pid>: perm对象的信息及其占用量。 ###

![](https://github.com/scalad/Note/blob/master/Java_Jstat/image/10.png)

### 11、jstat -printcompilation <pid>：当前VM执行的信息。 ###

![](https://github.com/scalad/Note/blob/master/Java_Jstat/image/11.png)