### jmeter性能测试之windows篇

#### 一、jmeter在windows中的安装

jmeter就是传说中的绿色软件，无需安装，只需要在官网下载zip包后，解压即可。

#### 二、jmeter的运行

jmeter需要依托jdk，首先你的系统中得有jdk。在windows下，进入jmeter解压后的主目录（如apache-jmeter-2.13），进入bin目录，双击运行jmeter.bat即可启动jmeter。如果打开时cmd窗口报错，一般是给jmeter分配的空间太小，需要调整一些参数，我们可以在编辑器中打开jmeter.bat，调整两个参数的值，一个是HEAP，一个是NEW。我的jmeter中这两个值配置如下：

	set HEAP=-Xms512m -Xmx512m
	set NEW=-XX:NewSize=128m -XX:MaxNewSize=128m

目前运行正常，如果您的设置和我的设置相同但无法启动，可以再调整一下，这两个参数的设置和你电脑本身的内存大小有关。

如果正常运行，会打开jmeter的主界面，如下图：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/2564647465.png)

#### 三、压测一个普通的get接口

下面我们来进行一个普通的get接口压测。接口是我自己写的，部署在自己本地机器，你们是访问不了的^_^。
接口地址：http://127.0.0.1:5000/api/search/user
请求参数：term=xiaoming&type=1

好，下面我们打开jmeter的主界面，先在左侧测试计划上右键添加一个线程组，如下：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/702867952.png)

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/1806796478.png)

下面对几个重要的地方进行说明：

1.线程数
jmeter中的线程数相当于loadrunner中的虚拟用户（vuser），即你想要并发多少用户，我们这里并发10个用户，就是说10个用户同时不断请求这个接口

2.Ramp-Up Period
这个参数指你想要多长时间启动这10个线程。我们这边在1秒内启动，如果设置为0，则是瞬间启动10个线程；如果设置为5，则是大约每秒启动两个线程。

3.循环次数
指的是每个线程请求多少次，我们这里设置为10，即每个线程发送10个请求，那么10个线程就会总共发出100个接口请求。如果勾选了永远，则表示一直发送请求。

4.Delay Thread creation until needed
该参数勾选表示延迟创建线程，我一般不勾选。

5.调度器
我这里没有勾选，若勾选，则会出现下面的设置文本框：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/3549944133.png)

这个调度器常用于设置该次压测持续多长时间，比如，若持续时间填写为1800，则表示该次压测持续半小时后就会自动停止。

> Tips：如果设置了持续时间，则启动时间和结束时间的设置都将无效。并且如果你一旦勾选调度器，那么上面的循环次数最好勾选为永远。

刚才我们添加了线程组，现在让我们右键点击左侧的线程组，添加一个HTTP请求：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/324644707.png)

这个时候就要我们去设置要压测的url以及参数等等，我们已经设置好了，如下：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/1923593825.png)

关键位置已经用红框标注出来了，我们的服务器在本地，为127.0.0.1，我们的端口为5000，如果是http默认端口则不用填写。get请求的路径也已经填写进去了，还有我们的两个参数都添加进去了。接下来我们再添加一个查看结果数，以便我们先看下请求的结果是否正确。

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/3957689577.png)

Tips：当测试通过，正式进行性能测试时，一定记得把这个查看结果数去掉，好几次，就因为这个查看结果数的存在，把jmeter搞崩了。

目前我们左侧的目录树是这样的：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/2376186870.png)

好了，我们初步设置完了，下面保存一下这个测试计划（jmeter的测试计划后缀是jmx），然后执行一下看看：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/622651851.png)

没错，执行按钮就是这个绿色小三角。还记得我们刚才添加线程组的时候，起了10个线程，每个线程循环10次，那么查看结果数下面就是100个请求，我们随便点击一个请求看下：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/481954290.png)

没错，是预期的请求url，响应的json数据也正常。

##### 性能测试的目的是什么？
做一次性能测试，我们的目的肯定是要得到某个接口的性能数据，进而评估这个接口是否满足并发要求、时间要求、以及其他要求，如果不满足这些要求，那就要分析这个接口的性能瓶颈在哪，怎样优化等等。常见的性能数据是QPS以及响应时间。

QPS：指该接口在1s内能处理多少个请求
响应时间：该接口响应一次请求需要多少时间。

那么问题来了，我们怎么通过jmeter来查看这些数据呢？jmeter默认提供了一些图表，其中信息量最大的可能要数Aggregate Graph，下面我们添加一个看看：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/2914601014.png)

添加了聚合报告后，我们再跑一轮，看看这聚合报告里面有些什么数据。


![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/1725278346.png)

聚合报告的表格中有很多字段，我们一个一个解读一下：

	Samples：表示总共请求了几次接口
	Average：平均响应时间，单位ms
	Median：50%的请求的响应时间小于该值
	90% Line：90%的请求的响应时间小于该值
	95% Line：95%的请求的响应时间小于该值
	99% Line：99%的请求的响应时间小于该值
	Min：最小的响应时间
	Max：最大的响应时间
	Error %：错误率
	Throughput：吞吐率，即QPS（TPS）

> Tips：关于Median、90% Line等的含义，很多同学都有误解，以为是各个百分比请求数下的平均响应时间，其实不然，拿90%Line来说，他指的是90%请求的最大响应时间，即按照响应时间从小到大排序，有90%的请求的响应时间是在该值之下的。


从聚合报告我们可以看出我们想要的QPS和各个响应时间。

有同学可能会说，如果我想知道QPS和响应时间在我压测的过程中随着时间的变化曲线，这个需求怎么满足呢？
jmeter默认自带的图表包含的东西并不多，不过我们可以在jmeter的官网上找到这些图表的插件。我们可以下载JMeterPlugins-Extras-1.3.1.zip，解压后在解压包的lib/ext目录得到JMeterPlugins-Extras.jar，把这个jar包复制到你的jmeter的安装目录的相同位置，如：apache-jmeter-2.13\lib\ext\ 目录下，并重启jmeter。
重启后，我们添加一个QPS图表和一个响应时间图表，如下：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/3732705026.png)


![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/3514939857.png)

上图的Transactions Per Second就是QPS（TPS），下图就是接口响应时间随时间变化图。
现在我们左侧的目录树是这样的：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/2414477759.png)

我们再来进行一轮压测，观察下这两个图表：
先看QPS：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/167337359.png)

从上图我们可以看到目前我们的接口qps大约在430左右。

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/2937255878.png)

从响应时间图我们可以看到响应时间大概是11ms左右。大家可以对照聚合报告中的数据合并这连个图进行对比，看下是否一致。

#### 四、设置请求header

看到这里，您对基本的get接口压测就有一定了解了。有时候，我们的请求是需要带header的，对于这种情况该怎么办呢？jmeter照样提供了相应的方法，我们可以添加一个HTTP信息头管理器，如下：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/1358110718.png)

在信息头中，我们也可以设置Cookie，比如如下：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/4086235082.png)

这样的设置就可以满足需要cookie验证的接口。

更多的设置就等您来慢慢发掘吧！jmeter已经足够强大。

#### 五、压测一个post接口

对于post接口的压测，其实是完全一样的，只有在设置HTTP请求的时候，设置为POST方法，并且请求body放入“Body Data”就可以，如下：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/1973724565.png)

#### 六、参数化

以上我们讲到的接口测试，参数都是固定的，那如果我们有一个测试集，要在这次性能测试中对同一个接口发送不同的参数该怎么办呢？这种需求很多，比如要进行一个线上真实流量的的测试，我们需要把nginx日志拿到，并且把这些真实的参数作为压测的输入，就会涉及到参数化。常用的参数化方法大概分为两种，下面分别介绍。

我们先看下需求，还是刚才的接口：http://127.0.0.1:5000/api/search/user

现在我们不传固定的参数，我要从文件读取参数，文件格式如下：

	xiaoming,1
	xiaohong,1
	xiaolong,2

也就是说，我们想要每次请求的时候，把当前读取到的行的第一列赋值给term，第二列赋值给type，即每次请求的url如下：

	http://127.0.0.1:5000/api/search/user?term=xiaoming&type=1
	http://127.0.0.1:5000/api/search/user?term=xiaohong&type=1
	http://127.0.0.1:5000/api/search/user?term=xiaolong&type=2

当文件的三行取完后，再次从头开始依次取值。
我们先看第一种参数化方法。

##### 1.函数助手

在jmeter菜单点击选项->函数助手对话框，弹出如下：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/3214965067.png)

在标1处，我们选择__CSVRead，然后在标2处写入我们的参数文件的路径：E:\data.txt

> 此处与格式无关，且每列以半角逗号分隔

然后在文件列号中写0（此处为固定值，表示我们的参数文件第一列序号从0开始）。然后在3处点击生成，把4处的函数字符串复制，然后开始在HTTP请求中设置参数，，如下：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/592142899.png)

上图中的名称从上到下，依次就是参数文件中从左到右的每列，注意到在值的那列，我们的函数从上到下只有序号不同，依次从0往下排。

运行一下，并在查看结果树中查看每次请求的是否是不同的参数。

##### 2.CSV Data Set Config
这种方法下，我们需要添加一个配置原件，就叫CSV Data Set Config：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/115424753.png)

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/429298423.png)

每个字段含义如下：

* Filename：指定文件路径
* File encoding：UTF-8

> Tips：保存参数文件的时候，注意要以utf-8格式保存

* Variable Names：设置参数文件中每列的参数名
* Delimiter：分隔符，这里是半角逗号
* Recycle on EOF：为True，参数文件取值到结尾时再次从头取

设置了CSV Data Set Config后，我们还需要返回HTTP请求，去设置参数：

![](https://github.com/scalad/Note/blob/master/Java_jmeter_server_polling_pressure_test/image/4183093774.png)

注意这里使用“$”符号，来取参数文件的每列。

> Tips：这里的参数名一定要和CSV Data Set Config中设置的Variable Names相同。

设置完这些，就可以检验成果了。

经过实测，第一种参数化方法取值有时会发生行错乱。如果要精确取值的话，最好还是采用第二种参数化方法。

