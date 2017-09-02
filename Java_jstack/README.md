### JDK自带命令jstack通过JVM堆栈分析出现大量线程的原因 ###
最近服务器内存总是耗光，于是想要分析下问题的原因：

首先找到JVM进程

	[root@10-13-40-54 social-search]# ps -aux | grep tomcat
	root      7437  5.8  8.0 10049732 1303772 pts/0 Sl  09:31   
	1:12 /usr/bin/java -Djava.util.logging.config.file=/usr/local/5080SocialSearchTomcat/conf/
	logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.endorsed.dirs=
	/usr/local/5080SocialSearchTomcat/endorsed -classpath /usr/local/5080SocialSearchTomcat/bin/bootstrap.jar:
	/usr/local/5080SocialSearchTomcat/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/5080SocialSearchTomcat 
	 -Dcatalina.home=/usr/local/5080SocialSearchTomcat -Djava.io.tmpdir=/usr/local/5080SocialSearchTomcat/
	temp/org.apache.catalina.startup.Bootstrap start
	
得到JVM的tomcat进程是7437，然后把堆栈导出，下载到本地：

	jstack -l 7437 > log.txt

下载后，发现线程堆栈中，有大量的这样的日志：

	"pool-2918-thread-2" #14663 prio=5 os_prio=0 tid=0x00007fdc81759000 nid=0x157c 
	waiting on condition [0x00007fd8821ef000]
	   java.lang.Thread.State: WAITING (parking)
		at sun.misc.Unsafe.park(Native Method)
		- parking to wait for  <0x000000079329da40
			- > - (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
		at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
		at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
		at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
		at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1067)
		at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1127)
		at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
		at java.lang.Thread.run(Thread.java:745)
	
	"pool-2918-thread-1" #14662 prio=5 os_prio=0 tid=0x00007fdc81757000 nid=0x157b 
	waiting on condition [0x00007fd8822f0000]
	   java.lang.Thread.State: WAITING (parking)
		at sun.misc.Unsafe.park(Native Method)
		- parking to wait for  <0x000000079329da40> 
			- (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
		at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
		at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
		at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
		at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1067)
		at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1127)
		at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
		at java.lang.Thread.run(Thread.java:745)

可以看到，线程处于WAITING状态，阻塞在试图从任务队列中取任务(LinkedBlockingQueue.take)，这个任务队列指的是ThreadPoolExecutor的线程池启动的线程任务队列；

也就是说，这些线程都是空闲状态，在等着任务的到来呢！

补充下LinkedBlockingQueue的知识：

	>并发库中的BlockingQueue是一个比较好玩的类，顾名思义，就是阻塞队列。该类主要提供了两个方法put()和take()，
	>前者将一个对象放到队列尾部，
	>如果队列已经满了，就等待直到有空闲节点；后者从head取一个对象，如果没有对象，就等待直到有可取的对象。

定位到问题就简单了，查找代码，发现有个位置启动了线程池提交了任务，但是任务执行完返回后，线程池没有关闭导致的；

问题总结：

1、使用ExecutorService提交的线程任务，也要记得关闭；

2、启动新线程的时候，最好给线程起个名字，这样线程堆栈的问题排查更加容易； 

### 关于Jstack ###
#### 一、介绍 ####
jstack是java虚拟机自带的一种堆栈跟踪工具。jstack用于打印出给定的java进程ID或core file或远程调试服务的Java堆栈信息，如果是在64位机器上，需要指定选项"-J-d64"，Windows的jstack使用方式只支持以下的这种方式：

	jstack [-l] pid

jstack用于生成java虚拟机当前时刻的线程快照。线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。 线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。 如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程序是如何崩溃和在程序何处发生问题。另外，jstack工具还可以附属到正在运行的java程序中，看到当时运行的java程序的java stack和native stack的信息, 如果现在运行的java程序呈现hung的状态，jstack是非常有用的。

So,jstack命令主要用来查看Java线程的调用堆栈的，可以用来分析线程问题（如死锁）

#### 线程状态 ####
想要通过jstack命令来分析线程的情况的话，首先要知道线程都有哪些状态，下面这些状态是我们使用jstack命令查看线程堆栈信息时可能会看到的线程的几种状态：

* NEW,未启动的。不会出现在Dump中。
* RUNNABLE,在虚拟机内执行的。
* BLOCKED,受阻塞并等待监视器锁。
* WATING,无限期等待另一个线程执行特定操作。
* TIMED_WATING,有时限的等待另一个线程的特定操作。
* TERMINATED,已退出的。

#### Monitor ####
在多线程的 JAVA程序中，实现线程之间的同步，就要说说 Monitor。 Monitor是 Java中用以实现线程之间的互斥与协作的主要手段，它可以看成是对象或者 Class的锁。每一个对象都有，也仅有一个 monitor。下 面这个图，描述了线程和 Monitor之间关系，以 及线程的状态转换图：

![](https://github.com/scalad/Note/blob/master/Java_jstack/image/thread.png)

进入区(Entrt Set):表示线程通过synchronized要求获取对象的锁。如果对象未被锁住,则迚入拥有者;否则则在进入区等待。一旦对象锁被其他线程释放,立即参与竞争。

拥有者(The Owner):表示某一线程成功竞争到对象锁。

等待区(Wait Set):表示线程通过对象的wait方法,释放对象的锁,并在等待区等待被唤醒。

从图中可以看出，一个 Monitor在某个时刻，只能被一个线程拥有，该线程就是 “Active Thread”，而其它线程都是 “Waiting Thread”，分别在两个队列 “ Entry Set”和 “Wait Set”里面等候。在 “Entry Set”中等待的线程状态是 “Waiting for monitor entry”，而在“Wait Set”中等待的线程状态是 “in Object.wait()”。 先看 “Entry Set”里面的线程。我们称被 synchronized保护起来的代码段为临界区。当一个线程申请进入临界区时，它就进入了 “Entry Set”队列。对应的 code就像：

```java
	synchronized(obj) {
		...	
	}
```

#### 调用修饰 ####

* locked <地址> 目标：使用synchronized申请对象锁成功,监视器的拥有者。
* waiting to lock <地址> 目标：使用synchronized申请对象锁未成功,在迚入区等待。
* waiting on <地址> 目标：使用synchronized申请对象锁成功后,释放锁幵在等待区等待。
* parking to wait for <地址> 目标

#### 1、locked ####

	at oracle.jdbc.driver.PhysicalConnection.prepareStatement
	 - locked <0x00002aab63bf7f58> (a oracle.jdbc.driver.T4CConnection)
	at oracle.jdbc.driver.PhysicalConnection.prepareStatement
	 - locked <0x00002aab63bf7f58> (a oracle.jdbc.driver.T4CConnection)
	at com.jiuqi.dna.core.internal.db.datasource.PooledConnection.prepareStatement

通过synchronized关键字,成功获取到了对象的锁,成为监视器的拥有者,在临界区内操作。对象锁是可以线程重入的。

#### 2、waiting to lock ####

	at com.jiuqi.dna.core.impl.CacheHolder.isVisibleIn(CacheHolder.java:165)
	 - waiting to lock <0x0000000097ba9aa8> (a CacheHolder)
	at com.jiuqi.dna.core.impl.CacheGroup$Index.findHolder
	at com.jiuqi.dna.core.impl.ContextImpl.find
	at com.jiuqi.dna.bap.basedata.common.util.BaseDataCenter.findInfo

通过synchronized关键字,没有获取到了对象的锁,线程在监视器的进入区等待。在调用栈顶出现,线程状态为Blocked。

#### 3、waiting on ####
	at java.lang.Object.wait(Native Method)
	 - waiting on <0x00000000da2defb0> (a WorkingThread)
	at com.jiuqi.dna.core.impl.WorkingManager.getWorkToDo
	 - locked <0x00000000da2defb0> (a WorkingThread)
	at com.jiuqi.dna.core.impl.WorkingThread.run

通过synchronized关键字,成功获取到了对象的锁后,调用了wait方法,进入对象的等待区等待。在调用栈顶出现,线程状态为WAITING或TIMED_WATING。

#### 4、parking to wait for ####
park是基本的线程阻塞原语,不通过监视器在对象上阻塞。随concurrent包会出现的新的机制,不synchronized体系不同。

#### 线程动作 ####
线程状态产生的原因：

* runnable:状态一般为RUNNABLE。
* in Object.wait():等待区等待,状态为WAITING或TIMED_WAITING。
* waiting for monitor entry:进入区等待,状态为BLOCKED。
* waiting on condition:等待区等待、被park。
* sleeping:休眠的线程,调用了Thread.sleep()。

Wait on condition 该状态出现在线程等待某个条件的发生。具体是什么原因，可以结合 stacktrace来分析。 最常见的情况就是线程处于sleep状态，等待被唤醒。 常见的情况还有等待网络IO：在java引入nio之前，对于每个网络连接，都有一个对应的线程来处理网络的读写操作，即使没有可读写的数据，线程仍然阻塞在读写操作上，这样有可能造成资源浪费，而且给操作系统的线程调度也带来压力。在 NewIO里采用了新的机制，编写的服务器程序的性能和可扩展性都得到提高。 正等待网络读写，这可能是一个网络瓶颈的征兆。因为网络阻塞导致线程无法执行。一种情况是网络非常忙，几 乎消耗了所有的带宽，仍然有大量数据等待网络读 写；另一种情况也可能是网络空闲，但由于路由等问题，导致包无法正常的到达。所以要结合系统的一些性能观察工具来综合分析，比如 netstat统计单位时间的发送包的数目，如果很明显超过了所在网络带宽的限制 ; 观察 cpu的利用率，如果系统态的 CPU时间，相对于用户态的 CPU时间比例较高；如果程序运行在 Solaris 10平台上，可以用 dtrace工具看系统调用的情况，如果观察到 read/write的系统调用的次数或者运行时间遥遥领先；这些都指向由于网络带宽所限导致的网络瓶颈。（来自[http://www.blogjava.net/jzone/articles/303979.html](http://www.blogjava.net/jzone/articles/303979.html)）

### 二、命令格式 ###

	jstack [ option ] pid
	jstack [ option ] executable core
	jstack [ option ] [server-id@]remote-hostname-or-IP

常用参数说明

1）options： 

executable Java executable from which the core dump was produced.(可能是产生core dump的java可执行程序)

core 将被打印信息的core dump文件

remote-hostname-or-IP 远程debug服务的主机名或ip

server-id 唯一id,假如一台主机上多个远程debug服务 

2）基本参数：

-F 当’jstack [-l] pid’没有相应的时候强制打印栈信息

-l 长列表. 打印关于锁的附加信息,例如属于java.util.concurrent的ownable synchronizers列表.

-m 打印java和native c/c++框架的所有栈信息.

-h | -help打印帮助信息

pid 需要被打印配置信息的java进程id,可以用jps查询.

### 三、使用实例 ###
1、jstack pid  

	~$ jps -ml
	org.apache.catalina.startup.Bootstrap 
	~$ jstack 5661
	2013-04-16 21:09:27
	Full thread dump Java HotSpot(TM) Server VM (20.10-b01 mixed mode):
	
	"Attach Listener" daemon prio=10 tid=0x70e95400 nid=0x2265 waiting on condition [0x00000000]
	   java.lang.Thread.State: RUNNABLE
	
	"http-bio-8080-exec-20" daemon prio=10 tid=0x08a35800 nid=0x1d42 waiting on condition [0x70997000]
	   java.lang.Thread.State: WAITING (parking)
	    at sun.misc.Unsafe.park(Native Method)
	    - parking to wait for  <0x766a27b8> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:156)
	    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:1987)
	    at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:399)
	    at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
	    at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
	    at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:947)
	    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:907)
	    at java.lang.Thread.run(Thread.java:662)
	........

一般情况下，通过jstack输出的线程信息主要包括：jvm自身线程、用户线程等。其中jvm线程会在jvm启动时就会存在。对于用户线程则是在用户访问时才会生成。

2、jstack 查看线程具体在做什么，可看出哪些线程在长时间占用CPU，尽快定位问题和解决问题

[http://www.iteye.com/topic/1114219](http://www.iteye.com/topic/1114219)

1.top查找出哪个进程消耗的cpu高。执行top命令，默认是进程视图，其中PID是进程号

	21125 co_ad2    18   0 1817m 776m 9712 S  3.3  4.9  12:03.24 java                                                                                           
	5284 co_ad     21   0 3028m 2.5g 9432 S  1.0 16.3   6629:44 ja

这里我们分析21125这个java进程

2.top中shift+h 或“H”查找出哪个线程消耗的cpu高 

先输入top，然后再按shift+h 或“H”，此时打开的是线程视图，pid为线程号

	21233 co_ad2    15   0 1807m 630m 9492 S  1.3  4.0   0:05.12 java                                                                                           
	20503 co_ad2_s  15   0 1360m 560m 9176 S  0.3  3.6   0:46.72 java                                                                                           

这里我们分析21233这个线程，并且注意的是，这个线程是属于21125这个进程的。 

3.使用jstack命令输出这一时刻的线程栈，保存到文件，命名为jstack.log。注意：输出线程栈和保存top命令快照尽量同时进行。

由于jstack.log文件记录的线程ID是16进制，需要将top命令展示的线程号转换为16进制。

4. jstack查找这个线程的信息 
jstack [进程]|grep -A 10 [线程的16进制] 

	即： jstack 21125|grep -A 10 52f1  

-A 10表示查找到所在行的后10行。21233用计算器转换为16进制52f1，注意字母是小写。 结果： 
 
	"http-8081-11" daemon prio=10 tid=0x00002aab049a1800 nid=0x52bb in Object.wait() [0x0000000042c75000]  
	   java.lang.Thread.State: WAITING (on object monitor)  
	     at java.lang.Object.wait(Native Method)  
	     at java.lang.Object.wait(Object.java:485)  
	     at org.apache.tomcat.util.net.JIoEndpoint$Worker.await(JIoEndpoint.java:416)  

在结果中查找52f1，可看到当前线程在做什么。

3、代码示例

```java
/**
 * @author hollis
 */
public class JStackDemo1 {
    public static void main(String[] args) {
        while (true) {
            //Do Nothing
        }
    }
}
```

先是有jps查看进程号：

	hollis@hos:~$ jps
	29788 JStackDemo1
	29834 Jps
	22385 org.eclipse.equinox.launcher_1.3.0.v20130327-1440.jar

然后使用jstack 查看堆栈信息：

	hollis@hos:~$ jstack 29788
	2015-04-17 23:47:31
	...此处省略若干内容...
	"main" prio=10 tid=0x00007f197800a000 nid=0x7462 runnable [0x00007f197f7e1000]
	   java.lang.Thread.State: RUNNABLE
	    at javaCommand.JStackDemo1.main(JStackDemo1.java:7)

我们可以从这段堆栈信息中看出什么来呢？我们可以看到，当前一共有一条用户级别线程,线程处于runnable状态，执行到JStackDemo1.java的第七行。 看下面代码：

```java
/**
 * @author hollis
 */
public class JStackDemo1 {
    public static void main(String[] args) {
        Thread thread = new Thread(new Thread1());
        thread.start();
    }
}
class Thread1 implements Runnable{
    @Override
    public void run() {
        while(true){
            System.out.println(1);
        }
    }
}
```

线程堆栈信息如下：

	"Reference Handler" daemon prio=10 tid=0x00007fbbcc06e000 nid=0x286c in Object.wait() [0x00007fbbc8dfc000]
	   java.lang.Thread.State: WAITING (on object monitor)
	    at java.lang.Object.wait(Native Method)
	    - waiting on <0x0000000783e066e0> (a java.lang.ref.Reference$Lock)
	    at java.lang.Object.wait(Object.java:503)
	    at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:133)
	    - locked <0x0000000783e066e0> (a java.lang.ref.Reference$Lock)

我们能看到：

> 线程的状态： WAITING 线程的调用栈 线程的当前锁住的资源： <0x0000000783e066e0> 线程当前等待的资源：<0x0000000783e066e0>

为什么同时锁住的等待同一个资源：
> 线程的执行中，先获得了这个对象的 Monitor（对应于 locked <0x0000000783e066e0>）。当执行到 obj.wait(), 线程即放弃了 Monitor的所有权，进入 “wait set”队列（对应于 waiting on <0x0000000783e066e0> ）。

### 四、如何分析 ###
#### 1、线程Dump的分析 ####
原则
> 结合代码阅读的推理。需要线程Dump和源码的相互推导和印证。造成Bug的根源往往丌会在调用栈上直接体现,一定格外注意线程当前调用之前的所有调用。

入手点

进入区等待

	"d&a-3588" daemon waiting for monitor entry [0x000000006e5d5000]
	java.lang.Thread.State: BLOCKED (on object monitor)
	at com.jiuqi.dna.bap.authority.service.UserService$LoginHandler.handle()
	 - waiting to lock <0x0000000602f38e90> (a java.lang.Object)
	at com.jiuqi.dna.bap.authority.service.UserService$LoginHandler.handle()

线程状态BLOCKED,线程动作wait on monitor entry,调用修饰waiting to lock总是一起出现。表示在代码级别已经存在冲突的调用。必然有问题的代码,需要尽可能减少其发生。

同步块阻塞

一个线程锁住某对象,大量其他线程在该对象上等待。
	
	"blocker" runnable
	java.lang.Thread.State: RUNNABLE
	at com.jiuqi.hcl.javadump.Blocker$1.run(Blocker.java:23)
	 - locked <0x00000000eb8eff68> (a java.lang.Object)
	"blockee-11" waiting for monitor entry
	java.lang.Thread.State: BLOCKED (on object monitor)
	at com.jiuqi.hcl.javadump.Blocker$2.run(Blocker.java:41)
	 - waiting to lock <0x00000000eb8eff68> (a java.lang.Object)
	"blockee-86" waiting for monitor entry
	java.lang.Thread.State: BLOCKED (on object monitor)
	at com.jiuqi.hcl.javadump.Blocker$2.run(Blocker.java:41)
	 - waiting to lock <0x00000000eb8eff68> (a java.lang.Object)

持续运行的IO IO操作是可以以RUNNABLE状态达成阻塞。例如:数据库死锁、网络读写。 格外注意对IO线程的真实状态的分析。 一般来说,被捕捉到RUNNABLE的IO调用,都是有问题的。

以下堆栈显示： 线程状态为RUNNABLE。 调用栈在SocketInputStream或SocketImpl上,socketRead0等方法。 调用栈包含了jdbc相关的包。很可能发生了数据库死锁

	"d&a-614" daemon prio=6 tid=0x0000000022f1f000 nid=0x37c8 runnable
	[0x0000000027cbd000]
	java.lang.Thread.State: RUNNABLE
	at java.net.SocketInputStream.socketRead0(Native Method)
	at java.net.SocketInputStream.read(Unknown Source)
	at oracle.net.ns.Packet.receive(Packet.java:240)
	at oracle.net.ns.DataPacket.receive(DataPacket.java:92)
	at oracle.net.ns.NetInputStream.getNextPacket(NetInputStream.java:172)
	at oracle.net.ns.NetInputStream.read(NetInputStream.java:117)
	at oracle.jdbc.driver.T4CMAREngine.unmarshalUB1(T4CMAREngine.java:1034)
	at oracle.jdbc.driver.T4C8Oall.receive(T4C8Oall.java:588)

分线程调度的休眠

正常的线程池等待
	
	"d&a-131" in Object.wait()
	java.lang.Thread.State: TIMED_WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	at com.jiuqi.dna.core.impl.WorkingManager.getWorkToDo(WorkingManager.java:322)
	 - locked <0x0000000313f656f8> (a com.jiuqi.dna.core.impl.WorkingThread)
	at com.jiuqi.dna.core.impl.WorkingThread.run(WorkingThread.java:40)

可疑的线程等待

	"d&a-121" in Object.wait()
	java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	at java.lang.Object.wait(Object.java:485)
	at com.jiuqi.dna.core.impl.AcquirableAccessor.exclusive()
	 - locked <0x00000003011678d8> (a com.jiuqi.dna.core.impl.CacheGroup)
	at com.jiuqi.dna.core.impl.Transaction.lock()

入手点总结

wait on monitor entry： 被阻塞的,肯定有问题

runnable ： 注意IO线程

in Object.wait()： 注意非线程池等待

#### 2、死锁分析 ####
的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。 说白了，我现在想吃鸡蛋灌饼，桌子上放着鸡蛋和饼，但是我和我的朋友同时分别拿起了鸡蛋和病，我手里拿着鸡蛋，但是我需要他手里的饼。他手里拿着饼，但是他想要我手里的鸡蛋。就这样，如果不能同时拿到鸡蛋和饼，那我们就不能继续做后面的工作（做鸡蛋灌饼）。所以，这就造成了死锁。 看一段死锁的程序：

```java
package javaCommand;
/**
 * @author hollis
 */
public class JStackDemo {
    public static void main(String[] args) {
        Thread t1 = new Thread(new DeadLockclass(true));//建立一个线程
        Thread t2 = new Thread(new DeadLockclass(false));//建立另一个线程
        t1.start();//启动一个线程
        t2.start();//启动另一个线程
    }
}
class DeadLockclass implements Runnable {
    public boolean falg;// 控制线程
    DeadLockclass(boolean falg) {
        this.falg = falg;
    }
    public void run() {
        /**
         * 如果falg的值为true则调用t1线程
         */
        if (falg) {
            while (true) {
                synchronized (Suo.o1) {
                    System.out.println("o1 " + Thread.currentThread().getName());
                    synchronized (Suo.o2) {
                        System.out.println("o2 " + Thread.currentThread().getName());
                    }
                }
            }
        }
        /**
         * 如果falg的值为false则调用t2线程
         */
        else {
            while (true) {
                synchronized (Suo.o2) {
                    System.out.println("o2 " + Thread.currentThread().getName());
                    synchronized (Suo.o1) {
                        System.out.println("o1 " + Thread.currentThread().getName());
                    }
                }
            }
        }
    }
}

class Suo {
    static Object o1 = new Object();
    static Object o2 = new Object();
}
```

当我启动该程序时，我们发现，程序只输出了两行内容，然后程序就不再打印其它的东西了，但是程序并没有停止。这样就产生了死锁。 当线程1使用synchronized锁住了o1的同时，线程2也是用synchronized锁住了o2。当两个线程都执行完第一个打印任务的时候，线程1想锁住o2，线程2想锁住o1。但是，线程1当前锁着o1，线程2锁着o2。所以两个想成都无法继续执行下去，就造成了死锁。

然后，我们使用jstack来看一下线程堆栈信息：
	
	Found one Java-level deadlock:
	=============================
	"Thread-1":
	  waiting to lock monitor 0x00007f0134003ae8 (object 0x00000007d6aa2c98, a java.lang.Object),
	  which is held by "Thread-0"
	"Thread-0":
	  waiting to lock monitor 0x00007f0134006168 (object 0x00000007d6aa2ca8, a java.lang.Object),
	  which is held by "Thread-1"
	
	Java stack information for the threads listed above:
	===================================================
	"Thread-1":
	    at javaCommand.DeadLockclass.run(JStackDemo.java:40)
	    - waiting to lock <0x00000007d6aa2c98> (a java.lang.Object)
	    - locked <0x00000007d6aa2ca8> (a java.lang.Object)
	    at java.lang.Thread.run(Thread.java:745)
	"Thread-0":
	    at javaCommand.DeadLockclass.run(JStackDemo.java:27)
	    - waiting to lock <0x00000007d6aa2ca8> (a java.lang.Object)
	    - locked <0x00000007d6aa2c98> (a java.lang.Object)
	    at java.lang.Thread.run(Thread.java:745)
	
	Found 1 deadlock.

哈哈，堆栈写的很明显，它告诉我们 Found one Java-level deadlock，然后指出造成死锁的两个线程的内容。然后，又通过 Java stack information for the threads listed above来显示更详细的死锁的信息。 他说:
> Thread-1在想要执行第40行的时候，当前锁住了资源<0x00000007d6aa2ca8>,但是他在等待资源<0x00000007d6aa2c98>Thread-0在想要执行第27行的时候，当前锁住了资源<0x00000007d6aa2c98>,但是他在等待资源<0x00000007d6aa2ca8> 由于这两个线程都持有资源，并且都需要对方的资源，所以造成了死锁。 原因我们找到了，就可以具体问题具体分析，解决这个死锁了。

#### 其他 ####

虚拟机执行Full GC时,会阻塞所有的用户线程。因此,即时获取到同步锁的线程也有可能被阻塞。 在查看线程Dump时,首先查看内存使用情况。

#### 频繁GC问题或内存溢出问题 ####

一、使用jps查看线程ID

二、使用jstat -gc 3331 250 20 查看gc情况，一般比较关注PERM区的情况，查看GC的增长情况。

三、使用jstat -gccause：额外输出上次GC原因

四、使用jmap -dump:format=b,file=heapDump 3331生成堆转储文件

五、使用jhat或者可视化工具（Eclipse Memory Analyzer 、IBM HeapAnalyzer）分析堆情况。

六、结合代码解决内存溢出或泄露问题。

#### 死锁问题 ####

一、使用jps查看线程ID

二、使用jstack 3331：查看线程情况