###JDK自带命令jstack通过JVM堆栈分析出现大量线程的原因
最近服务器内存总是耗光，于是想要分析下问题的原因：

首先找到JVM进程

	[root@10-13-40-54 social-search]# ps -aux | grep tomcat
	root      7437  5.8  8.0 10049732 1303772 pts/0 Sl  09:31   1:12 /usr/bin/java -Djava.util.logging.config.file=/usr/local/5080SocialSearchTomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.endorsed.dirs=/usr/local/5080SocialSearchTomcat/endorsed -classpath /usr/local/5080SocialSearchTomcat/bin/bootstrap.jar:/usr/local/5080SocialSearchTomcat/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/5080SocialSearchTomcat -Dcatalina.home=/usr/local/5080SocialSearchTomcat -Djava.io.tmpdir=/usr/local/5080SocialSearchTomcat/temp org.apache.catalina.startup.Bootstrap start
	
得到JVM的tomcat进程是7437，然后把堆栈导出，下载到本地：

	jstack -l 7437 > log.txt

下载后，发现线程堆栈中，有大量的这样的日志：

	"pool-2918-thread-2" #14663 prio=5 os_prio=0 tid=0x00007fdc81759000 nid=0x157c waiting on condition [0x00007fd8821ef000]
	   java.lang.Thread.State: WAITING (parking)
		at sun.misc.Unsafe.park(Native Method)
		- parking to wait for  <0x000000079329da40> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
		at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
		at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
		at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
		at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1067)
		at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1127)
		at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
		at java.lang.Thread.run(Thread.java:745)
	
	"pool-2918-thread-1" #14662 prio=5 os_prio=0 tid=0x00007fdc81757000 nid=0x157b waiting on condition [0x00007fd8822f0000]
	   java.lang.Thread.State: WAITING (parking)
		at sun.misc.Unsafe.park(Native Method)
		- parking to wait for  <0x000000079329da40> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
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

	>并发库中的BlockingQueue是一个比较好玩的类，顾名思义，就是阻塞队列。该类主要提供了两个方法put()和take()，前者将一个对象放到队列尾部，
	>如果队列已经满了，就等待直到有空闲节点；后者从head取一个对象，如果没有对象，就等待直到有可取的对象。

定位到问题就简单了，查找代码，发现有个位置启动了线程池提交了任务，但是任务执行完返回后，线程池没有关闭导致的；

问题总结：

1、使用ExecutorService提交的线程任务，也要记得关闭；

2、启动新线程的时候，最好给线程起个名字，这样线程堆栈的问题排查更加容易； 